> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fTAarvPvswGIXMz7hzQkzg)

作者：Dennis Brinkrolf  
译者：豌豆花下猫 @Python 猫  
原题：10 Unknown Security Pitfalls for Python  
英文：https://blog.sonarsource.com/10-unknown-security-pitfalls-for-python  
声明：本翻译是出于交流学习的目的，基于 CC BY-NC-SA 4.0 授权协议。为便于阅读，内容略有改动。转载时请保留作者及译者信息。

Python 开发者们在使用标准库和通用框架时，都以为自己的程序具有可靠的安全性。然而，在 Python 中，就像在任何其它编程语言中一样，有一些特性可能会被开发者们误解或误用。通常而言，只有极少的微妙之处或细节会使开发者们疏忽大意，从而在代码中引入严重的安全漏洞。

在这篇博文中，我们将分享在实际 Python 项目中遇到的 10 个安全陷阱。我们选择了一些在技术圈中不太为人所知的陷阱。通过介绍每个问题及其造成的影响，我们希望提高人们对这些问题的感知，并提高大家的安全意识。如果你正在使用这些特性，请一定要排查你的 Python 代码！

# 1. 被优化掉的断言
------------

Python 支持以优化的方式执行代码。这使代码运行得更快，内存用得更少。当程序被大规模使用，或者可用的资源很少时，这种方法尤其有效。一些预打包的 Python 程序提供了优化的字节码。

然而，当代码被优化时，所有的 assert 语句都会被忽略。开发者有时会使用它们来判断代码中的某些条件。例如，如果使用断言来作身份验证检查，则可能导致安全绕过。

```
def superuser_action(request, user):
    assert user.is_super_user
    # execute action as super user


```

在这个例子中，第 2 行中的 assert 语句将被忽略，导致非超级用户也可以运行到下一行代码。不推荐使用 assert 语句进行安全相关的检查，但我们确实在实际的项目中看到过它们。

# 2. MakeDirs 权限
----------------

`os.makdirs` 函数可以在操作系统中创建一个或多个文件夹。它的第二个参数 mode 用于指定创建的文件夹的默认权限。在下面代码的第 2 行中，文件夹 A/B/C 是用 rwx------ (0o700) 权限创建的。这意味着只有当前用户（所有者）拥有这些文件夹的读、写和执行权限。

```
def init_directories(request):
    os.makedirs("A/B/C", mode=0o700)
    return HttpResponse("Done!")


```

在 Python <3.6 版本中，创建出的文件夹 A、B 和 C 的权限都是 700。但是，在 Python> 3.6 版本中，只有最后一个文件夹 C 的权限为 700，其它文件夹 A 和 B 的权限为默认的 755。

因此，在 Python > 3.6 中，`os.makdirs` 函数等价于 Linux 的这条命令：`mkdir -m 700 -p A/B/C`。有些开发者没有意识到版本之间的差异，这已经在 Django 中造成了一个权限越级漏洞（cve - 2022 -24583），无独有偶，这在 WordPress 中也造成了一个加固绕过问题。

# 3. 绝对路径拼接
-----------

`os.path.join(path, *paths)` 函数用于将多个文件路径连接成一个组合的路径。第一个参数通常包含了基础路径，而之后的每个参数都被当做组件拼接到基础路径后。

然而，这个函数有一个少有人知的特性。如果拼接的某个路径以 / 开头，那么包括基础路径在内的所有前缀路径都将被删除，该路径将被视为绝对路径。下面的示例揭示了开发者可能遇到的这个陷阱。

```
def read_file(request):
    filename = request.POST['filename']
    file_path = os.path.join("var", "lib", filename)
    if file_path.find(".") != -1:
        return HttpResponse("Failed!")
    with open(file_path) as f:
        return HttpResponse(f.read(), content_type='text/plain')


```

在第 3 行中，我们使用 os.path.join 函数将用户输入的文件名构造出目标路径。在第 4 行中，检查生成的路径是否包含”.“，防止出现路径遍历漏洞。

但是，如果攻击者传入的文件名参数为”/a/b/c.txt“，那么第 3 行得到的变量 file_path 会是一个绝对路径（/a/b/c.txt）。即 os.path.join 会忽略掉”var/lib“部分，攻击者可以不使用 “.” 字符就读取到任何文件。尽管 os.path.join 的文档中描述了这种行为，但这还是导致了许多漏洞（Cuckoo Sandbox Evasion， CVE-2020-35736）。

# 4. 任意的临时文件
------------

`tempfile.NamedTemporaryFile` 函数用于创建具有特定名称的临时文件。但是，prefix（前缀）和 suffix（后缀）参数很容易受到路径遍历攻击（Issue 35278）。如果攻击者控制了这些参数之一，他就可以在文件系统中的任意位置创建出一个临时文件。下面的示例揭示了开发者可能遇到的一个陷阱。

```
def touch_tmp_file(request):
    id = request.GET['id']
    tmp_file = tempfile.NamedTemporaryFile(prefix=id)
    return HttpResponse(f"tmp file: {tmp_file} created!", content_type='text/plain')


```

在第 3 行中，用户输入的 id 被当作临时文件的前缀。如果攻击者传入的 id 参数是 “/../var/www/test”，则会创建出这样的临时文件：/var/www/test_zdllj17。粗看起来，这可能是无害的，但它会为攻击者创造出挖掘更复杂的漏洞的基础。

# 5. 扩展的 Zip Slip
-----------------

在 Web 应用中，通常需要解压上传后的压缩文件。在 Python 中，很多人都知道 TarFile.extractall 与 TarFile.extract 函数容易受到 Zip Slip 攻击。攻击者通过篡改压缩包中的文件名，使其包含路径遍历（../）字符，从而发起攻击。

这就是为什么压缩文件应该始终被视为不受信来源的原因。zipfile.extractall 与 zipfile.extract 函数可以对 zip 内容进行清洗，从而防止这类路径遍历漏洞。

但是，这并不意味着在 ZipFile 库中不会出现路径遍历漏洞。下面是一段解压缩文件的代码。

```
def extract_html(request):
    filename = request.FILES['filename']
    zf = zipfile.ZipFile(filename.temporary_file_path(), "r")
    for entry in zf.namelist():
        if entry.endswith(".html"):
            file_content = zf.read(entry)
            with open(entry, "wb") as fp:
                fp.write(file_content)
    zf.close()
    return HttpResponse("HTML files extracted!")


```

第 3 行代码根据用户上传文件的临时路径，创建出一个 ZipFile 处理器。第 4 - 8 行代码将所有以 “.html” 结尾的压缩项提取出来。第 4 行中的 zf.namelist 函数会取到 zip 内压缩项的名称。注意，只有 zipfile.extract 与 zipfile.extractall 函数会对压缩项进行清洗，其它任何函数都不会。

在这种情况下，攻击者可以创建一个文件名，例如 “../../../var/www/html”，内容随意填。该恶意文件的内容会在第 6 行被读取，并在第 7-8 行写入被攻击者控制的路径。因此，攻击者可以在整个服务器上创建任意的 HTML 文件。

如上所述，压缩包中的文件应该被看作是不受信任的。如果你不使用 zipfile.extractall 或者 zipfile.extract，你就必须对 zip 内文件的名称进行 “消毒”，例如使用 os.path.basename。否则，它可能导致严重的安全漏洞，就像在 NLTK Downloader （CVE-2019-14751）中发现的那样。

# 6. 不完整的正则表达式匹配
----------------

正则表达式（regex）是大多数 Web 程序不可或缺的一部分。我们经常能看到它被自定义的 Web 应用防火墙（WAF，Web Application Firewalls）用来作输入验证，例如检测恶意字符串。在 Python 中，re.match 和 re.search 之间有着细微的区别，我们将在下面的代码片段中演示。

```
def is_sql_injection(request):
    pattern = re.compile(r".*(union)|(select).*")
    name_to_test = request.GET['name']
    if re.search(pattern, name_to_test):
        return True
    return False


```

在第 2 行中，我们定义了一个匹配 union 或者 select 的模式，以检测可能的 SQL 注入。这是一个糟糕的写法，因为你可以轻易地绕过这些黑名单，但我们已经在线上的程序中见过它。在第 4 行中，函数 re.match 使用前面定义好的模式，检查第 3 行中的用户输入内容是否包含这些恶意的值。

然而，与 re.search 函数不同的是，re.match 函数不匹配新行。例如，如果攻击者提交了值 aaaaaa \n union select，这个输入就匹配不上正则表达式。因此，检查可以被绕过，失去保护作用。

总而言之，我们不建议使用正则表达式黑名单进行任何安全检查。

# 7. Unicode 清洗器绕过
------------------

Unicode 支持用多种形式来表示字符，并将这些字符映射到码点。在 Unicode 标准中，不同的 Unicode 字符有四种归一化方案。程序可以使用这些归一化方法，以独立于人类语言的标准方式来存储数据，例如用户名。

然而，攻击者可以利用这些归一化，这已经导致了 Python 的 urllib 出现漏洞（CVE-2019-9636）。下面的代码片段演示了一个基于 NFKC 归一化的跨站点脚本漏洞（XSS,Cross-Site Scripting）。

```
import unicodedata
from django.shortcuts import render
from django.utils.html import escape

def render_input(request):
    user_input = escape(request.GET['p'])
    normalized_user_input = unicodedata.normalize("NFKC", user_input)
    context = {'my_input': normalized_user_input}
    return render(request, 'test.html', context)


```

在第 6 行中，用户输入的内容被 Django 的 escape 函数处理了，以防止 XSS 漏洞。在第 7 行中，经过清洗的输入被 NFKC 算法归一化，以便在第 8-9 行中通过 test.html 模板正确地渲染。

**templates/test.html**

```
<!DOCTYPE html>
<html lang="en">
<body>
{{ my_input | safe}}
</body>
</html>


```

在模板 test.html 中，第 4 行的变量 my_input 被标记为安全的，因为开发人员预期有特殊字符，并且认为该变量已经被 escape 函数清洗了。通过标记关键字 safe, Django 不会再次对变量进行清洗。

但是，由于第 7 行（view.py）的归一化，字符 “%EF%B9%A4” 会被转换为 “<”，“%EF%B9%A5” 被转换为“>”。这导致攻击者可以注入任意的 HTML 标记，进而触发 XSS 漏洞。为了防止这个漏洞，就应该在把用户输入做完归一化之后，再进行清洗。

# 8. Unicode 编码碰撞
-----------------

前文说过，Unicode 字符会被映射成码点。然而，有许多不同的人类语言，Unicode 试图将它们统一起来。这就意味着不同的字符很有可能拥有相同的 “layout”。例如，小写的土耳其语 ı（没有点）的字符是英语中大写的 I。在拉丁字母中，字符 i 也是用大写的 I 表示。在 Unicode 标准中，这两个不同的字符都以大写形式映射到同一个码点。

这种行为是可以被利用的，实际上已经在 Django 中导致了一个严重的漏洞（CVE-2019-19844）。下面的代码是一个重置密码的示例。

```
from django.core.mail import send_mail
from django.http import HttpResponse
from vuln.models import User

def reset_pw(request):
    email = request.GET['email']
    result = User.objects.filter(email__exact=email.upper()).first()
    if not result:
        return HttpResponse("User not found!")
    send_mail('Reset Password','Your new pw: 123456.', 'from@example.com', [email], fail_silently=False)
    return HttpResponse("Password reset email send!")


```

第 6 行代码获取了用户输入的 email，第 7-9 行代码检查这个 email 值，查找是否存在具有该 email 的用户。如果用户存在，则第 10 行代码依据第 6 行中输入的 email 地址，给用户发送邮件。需要指出的是，第 7-9 行中对邮件地址的检查是不区分大小写的，使用了 upper 函数。

至于攻击，我们假设数据库中存在一个邮箱地址为 foo@mix.com 的用户。那么，攻击者可以简单地传入 foo@mıx.com 作为第 6 行中的 email，其中 i 被替换为土耳其语 ı。第 7 行代码将邮箱转换成大写，结果是 FOO@MIX.COM。这意味着找到了一个用户，因此会发送一封重置密码的邮件。

然而，邮件被发送到第 6 行未转换的邮件地址，也就是包含了土耳其语的 ı。换句话说，其他用户的密码被发送到了攻击者控制的邮件地址。为了防止这个漏洞，可以将第 10 行替换成使用数据库中的用户邮箱。即使发生编码冲突，攻击者在这种情况下也得不到任何好处。

# 9. IP 地址归一化
-------------

在 Python < 3.8 中，IP 地址会被 ipaddress 库归一化，因此前缀的零会被删除。这种行为乍一看可能是无害的，但它已经在 Django 中导致了一个高严重性的漏洞（CVE-2021-33571）。攻击者可以利用归一化绕过校验程序，发起服务端请求伪造攻击（SSRF，Server-Side Request Forgery）。

下面的代码展示了如何绕过这样的校验器。

```
import requests
import ipaddress

def send_request(request):
    ip = request.GET['ip']
    try:
        if ip in ["127.0.0.1", "0.0.0.0"]:
            return HttpResponse("Not allowed!")
        ip = str(ipaddress.IPv4Address(ip))
    except ipaddress.AddressValueError:
        return HttpResponse("Error at validation!")
    requests.get('https://' + ip)
    return HttpResponse("Request send!")


```

第 5 行代码获取用户传入的一个 IP 地址，第 7 行代码使用一个黑名单来检查该 IP 是否为本地地址，以防止可能的 SSRF 漏洞。这份黑名单并不完整，仅作为示例。

第 9 行代码检查该 IP 是否为 IPv4 地址，同时将 IP 归一化。在完成验证后，第 12 行代码会对该 IP 发起实际的请求。

但是，攻击者可以传入 127.0.001 这样的 IP 地址，在第 7 行的黑名单列表中找不到。然后，第 9 行代码使用 ipaddress.IPv4Address 将 IP 归一化为 127.0.0.1。因此，攻击者就能够绕过 SSRF 校验器，并向本地网络地址发送请求。

# 10. URL 查询参数解析
----------------

在 Python <3.7 中，urllib.parse.parse_qsl 函数允许使用 “;” 和“&”字符作为 URL 的查询变量的分隔符。有趣的是 “;” 字符不能被其它语言识别为分隔符。

在下面的例子中，我们将展示为什么这种行为会导致漏洞。假设我们正在运行一个基础设施，其中前端是一个 PHP 程序，后端则是一个 Python 程序。

攻击者向 PHP 前端发送以下的 GET 请求:

```
GET https://victim.com/?a=1;b=2


```

PHP 前端只识别出一个查询参数 “a”，其内容为“1;b=2”。PHP 不把“;” 字符作为查询参数的分隔符。现在，前端会将攻击者的请求直接转发给内部的 Python 程序:

```
GET https://internal.backend/?a=1;b=2


```

如果使用了 urllib.parse.parse_qsl，Python 程序会处理成两个查询参数，即 “a=1” 和“b=2”。这种查询参数解析的差异可能会导致致命的安全漏洞，比如 Django 中的 Web 缓存投毒漏洞（CVE-2021-23336）。

# 总结
----

在这篇博文中，我们介绍了 10 个 Python 安全陷阱，我们认为开发者不太了解它们。每个细微的陷阱都很容易被忽视，并在过去导致了线上程序的安全漏洞。

正如前文所述，安全陷阱可能出现在各种操作中，从处理文件、目录、压缩文件、URL、IP 到简单的字符串。一种常见的情况是库函数的使用，这些函数可能有意想不到的行为。这提醒我们一定要升级到最新版本，并仔细阅读文档。在 SonarSource 中，我们正在研究这些缺陷，以便将来不断改进我们的代码分析器。