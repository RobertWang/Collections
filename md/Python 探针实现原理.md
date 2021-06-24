> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkxNDE5NTQwMQ==&mid=2247490213&idx=3&sn=fd55a0966c6f0ae617085175e9600550&chksm=c1734c03f604c51511f666c6feb89e51d7227d9818d11cba0e5bac3176042f649b2197ad2f90&mpshare=1&scene=1&srcid=0624eMEaNOeIlMdKeyxEcfWd&sharer_sharetime=1624521740329&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> https://segmentfault.com/a/1190000004889212

关于 Python 的导入机制，我以前写过一篇文章，非常详细，[深入探讨 Python 的 import 机制：实现远程导入模块] 另外，今天再给你推荐这篇文章，同样是介绍 Python 的导入机制，和上面的文章一起食用更佳。本文呢，将简单讲述一下 Python 探针的实现原理。同时为了验证这个原理，我们也会一起来实现一个简单的统计指定函数执行时间的探针程序。探针的实现主要涉及以下几个知识点:

*   sys.meta_path
    
*   sitecustomize.py
    

### sys.meta_path

`sys.meta_path` 这个简单的来说就是可以实现 `import hook` 的功能， 当执行 `import` 相关的操作时，会触发 `sys.meta_path` 列表中定义的对象。关于 `sys.meta_path` 更详细的资料请查阅 python 文档中 sys.meta_path 相关内容以及 PEP 0302 。  
`sys.meta_path` 中的对象需要实现一个 `find_module` 方法， 这个 `find_module` 方法返回 `None` 或一个实现了 `load_module` 方法的对象 (代码可以从 github 上下载 part1_) :

```
import sysclass MetaPathFinder:    def find_module(self, fullname, path=None):        print('find_module {}'.format(fullname))        return MetaPathLoader()class MetaPathLoader:    def load_module(self, fullname):        print('load_module {}'.format(fullname))        sys.modules[fullname] = sys        return syssys.meta_path.insert(0, MetaPathFinder())if __name__ == '__main__':    import http    print(http)    print(http.version_info)


```

`load_module` 方法返回一个 `module` 对象，这个对象就是 import 的 module 对象了。比如我上面那样就把 `http` 替换为 `sys` 这个 module 了。

```
$ python meta_path1.pyfind_module httpload_module http<module 'sys' (built-in)>sys.version_info(major=3, minor=5, micro=1, releaselevel='final', serial=0)


```

通过 `sys.meta_path` 我们就可以实现 import hook 的功能：当 import 预定的 module 时，对这个 module 里的对象来个狸猫换太子， 从而实现获取函数或方法的执行时间等探测信息。上面说到了狸猫换太子，那么怎么对一个对象进行狸猫换太子的操作呢？对于函数对象，我们可以使用装饰器的方式来替换函数对象 (代码可以从 github 上下载 part2) :

```
import functoolsimport timedef func_wrapper(func):    @functools.wraps(func)    def wrapper(*args, **kwargs):        print('start func')        start = time.time()        result = func(*args, **kwargs)        end = time.time()        print('spent {}s'.format(end - start))        return result    return wrapperdef sleep(n):    time.sleep(n)    return nif __name__ == '__main__':    func = func_wrapper(sleep)    print(func(3))


```

执行结果:

```
$ python func_wrapper.pystart funcspent 3.004966974258423s3


```

下面我们来实现一个计算指定模块的指定函数的执行时间的功能 (代码可以从 github 上下载 part3) 。假设我们的模块文件是 hello.py:

```
import timedef sleep(n):    time.sleep(n)    return n


```

我们的 import hook 是 hook.py:

```
import functoolsimport importlibimport sysimport time_hook_modules = {'hello'}class MetaPathFinder:    def find_module(self, fullname, path=None):        print('find_module {}'.format(fullname))        if fullname in _hook_modules:            return MetaPathLoader()class MetaPathLoader:    def load_module(self, fullname):        print('load_module {}'.format(fullname))        # ``sys.modules`` 中保存的是已经导入过的 module        if fullname in sys.modules:            return sys.modules[fullname]        # 先从 sys.meta_path 中删除自定义的 finder        # 防止下面执行 import_module 的时候再次触发此 finder        # 从而出现递归调用的问题        finder = sys.meta_path.pop(0)        # 导入 module        module = importlib.import_module(fullname)        module_hook(fullname, module)        sys.meta_path.insert(0, finder)        return modulesys.meta_path.insert(0, MetaPathFinder())def module_hook(fullname, module):    if fullname == 'hello':        module.sleep = func_wrapper(module.sleep)def func_wrapper(func):    @functools.wraps(func)    def wrapper(*args, **kwargs):        print('start func')        start = time.time()        result = func(*args, **kwargs)        end = time.time()        print('spent {}s'.format(end - start))        return result    return wrapper


```

测试代码:

```
>>> import hook>>> import hellofind_module helloload_module hello>>>>>> hello.sleep(3)start funcspent 3.0029919147491455s3>>>


```

其实上面的代码已经实现了探针的基本功能。不过有一个问题就是上面的代码需要显示的 执行 `import hook` 操作才会注册上我们定义的 hook。那么有没有办法在启动 python 解释器的时候自动执行 `import hook` 的操作呢？答案就是可以通过定义 `sitecustomize.py` 的方式来实现这个功能。

### sitecustomize.py

简单的说就是，python 解释器初始化的时候会自动 `import PYTHONPATH` 下存在的 `sitecustomize` 和 `usercustomize` 模块: 实验项目的目录结构如下 (代码可以从 github 上下载 part4) :

```
$ tree.├── sitecustomize.py└── usercustomize.py


```

sitecustomize.py:

```
$ cat sitecustomize.pyprint('this is sitecustomize')


```

usercustomize.py:

```
$ cat usercustomize.pyprint('this is usercustomize')


```

把当前目录加到 `PYTHONPATH` 中，然后看看效果:

```
$ export PYTHONPATH=.$ pythonthis is sitecustomize       <----this is usercustomize       <----Python 3.5.1 (default, Dec 24 2015, 17:20:27)[GCC 4.2.1 Compatible Apple LLVM 7.0.2 (clang-700.1.81)] on darwinType "help", "copyright", "credits" or "license" for more information.>>>


```

可以看到确实自动导入了。所以我们可以把之前的探测程序改为支持自动执行 import hook (代码可以从 github 上下载 part5) 。目录结构:

```
$ tree.├── hello.py├── hook.py├── sitecustomize.py


```

sitecustomize.py:

```
$ cat sitecustomize.pyimport hook


```

结果:

```
$ export PYTHONPATH=.$ pythonfind_module usercustomizePython 3.5.1 (default, Dec 24 2015, 17:20:27)[GCC 4.2.1 Compatible Apple LLVM 7.0.2 (clang-700.1.81)] on darwinType "help", "copyright", "credits" or "license" for more information.find_module readlinefind_module atexitfind_module rlcompleter>>>>>> import hellofind_module helloload_module hello>>>>>> hello.sleep(3)start funcspent 3.005002021789551s3


```

不过上面的探测程序其实还有一个问题，那就是需要手动修改 `PYTHONPATH` 。用过探针程序的朋友应该会记得， 使用 newrelic 之类的探针只需要执行一条命令就 可以了：`newrelic-admin run-program python hello.py` 实际上修改 `PYTHONPATH` 的操作是在`newrelic-admin` 这个程序里完成的。下面我们也要来实现一个类似的命令行程序，就叫 `agent.py` 吧。

### agent

还是在上一个程序的基础上修改。先调整一个目录结构，把 `hook` 操作放到一个单独的目录下， 方便设置 `PYTHONPATH` 后不会有其他的干扰（代码可以从 github 上下载 part6 ）。

```
$ mkdir bootstrap$ mv hook.py bootstrap/_hook.py$ touch bootstrap/__init__.py$ touch agent.py$ tree.├── bootstrap│   ├── __init__.py│   ├── _hook.py│   └── sitecustomize.py├── hello.py├── test.py├── agent.py


```

`bootstrap/sitecustomize.py` 的内容修改为:

```
$ cat bootstrap/sitecustomize.pyimport _hook


```

agent.py 的内容如下:

```
import osimport syscurrent_dir = os.path.dirname(os.path.realpath(__file__))boot_dir = os.path.join(current_dir, 'bootstrap')def main():    args = sys.argv[1:]    os.environ['PYTHONPATH'] = boot_dir    # 执行后面的 python 程序命令    # sys.executable 是 python 解释器程序的绝对路径 ``which python``    # >>> sys.executable    # '/usr/local/var/pyenv/versions/3.5.1/bin/python3.5'    os.execl(sys.executable, sys.executable, *args)if __name__ == '__main__':    main()


```

`test.py` 的内容为:

```
$ cat test.pyimport sysimport helloprint(sys.argv)print(hello.sleep(3))


```

使用方法:

```
$ python agent.py test.py arg1 arg2find_module usercustomizefind_module helloload_module hello['test.py', 'arg1', 'arg2']start funcspent 3.005035161972046s3


```

至此，我们就实现了一个简单的 python 探针程序。当然，跟实际使用的探针程序相比肯定是有 很大的差距的，这篇文章主要是讲解一下探针背后的实现原理。文中的代码：_https://github.com/mozillazg/apm-python-agent-principle_