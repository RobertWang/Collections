> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/1aa14a6b6ae1)

邮件发送函数
======

send_email_smtp 函数
------------------

通过查看 incubator-airflow/airflow/utils/email.py 源码我们可以得出以下结论

*   email 发送功能最终由 send_email_smtp 函数完成
*   send_email 函数读取 airflow 的配置文件决定邮件的发送函数，默认是 send_email_stmp 函数

```
def send_email(to, subject, html_content, files=None, dryrun=False, cc=None, bcc=None, mime_subtype='mixed'):
    """
    Send email using backend specified in EMAIL_BACKEND.
    """
    path, attr = configuration.get('email', 'EMAIL_BACKEND').rsplit('.', 1)
    module = importlib.import_module(path)
    backend = getattr(module, attr)
    return backend(to, subject, html_content, files=files, dryrun=dryrun, cc=cc, bcc=bcc, mime_subtype=mime_subtype)


```

因此，airflow 自定义邮件标题的实现就是在发送邮件时，能够使用 “自定义的邮件标题”，即可以自定义 subject 的值。

实现方式
====

通过调研，有三种实现方式：

1.  直接改变源码。可以修改其调用者 models.py 中的 email_alert 方法，这种方法最直观，也最简单实现，缺点是需要修改 airflow 的源码，适合于几乎不会更改并且修改源码较容易的场景。
2.  添加 controller。在 send_email 和 send_email_smtp 函数中间添加 email_smtp_controller 函数，此函数用于转发邮件的发送任务，此方法，可以添加多个自定义类似与 send_email_smtp 的函数，这种场景下，实现一个自定义的邮件发送者即可完成功能。这种方法，不需要修改 airflow 的源码，只需要简单修改配置文件，扩展性较强，相对第一种方法复杂度要高一些。适合于不修改源码，对扩展性有部分要求的场景。
3.  对 airflow 进行 adapter。通过封装 airflow 的来满足工作需求。此方法不需要修改 airflow 源码，需要额外维护 airflow 的 adapter，复杂度高，但扩展性强。适合于有较多修改，并对扩展性要求较高的场景。

直接修改源码
------

这种方式比较简单，并且可在互联网上搜索到源码，此处就不再重复介绍。

添加 controller
-------------

此方法的本质是，在 send_email_smtp 的某个调用层次改变 subject 的值。需要查看 send_email_smtp 的整个调用层次，通过源码分析，其调用层次如下：

![](http://upload-images.jianshu.io/upload_images/2414051-ee63c7c09df11ad3) 调用层次

send_email_smtp 函数的调用者涉及三个文件 job.py,email_operator.py 和 models.py; 其中 send_email 对 send_email_smtp 的调用通过 airflow.cfg 配置的。

```
email_backend = airflow.utils.email.send_email_smtp 


```

airflow 报警邮件功能在 models.py 中的 email_alert 函数中实现

```
    def email_alert(self, exception, is_retry=False):
        task = self.task
        title = "Airflow alert: {self}".format(**locals())
        exception = str(exception).replace('\n', '<br>')
        # For reporting purposes, we report based on 1-indexed,
        # not 0-indexed lists (i.e. Try 1 instead of
        # Try 0 for the first attempt).
        body = (
            "Try {try_number} out of {max_tries}<br>"
            "Exception:<br>{exception}<br>"
            "Log: <a href='{self.log_url}'>Link</a><br>"
            "Host: {self.hostname}<br>"
            "Log file: {self.log_filepath}<br>"
            "Mark success: <a href='{self.mark_success_url}'>Link</a><br>"
        ).format(try_number=self.try_number + 1, max_tries=self.max_tries + 1, **locals())
        send_email(task.email, title, body)


```

根据上面的调用层次，在不影响其它邮件调用的前提下，具体实现自定义报警邮件标题方法可采用：

1.  复制 send_email_smtp，使 models.py 与其它文件的调用分开
2.  添加 rpt_email_smtp_controller.py，做为控制器以选择合适的 send_email_smtp  
    第二种方法比第一种方法要高级一些，并且容易扩展，实现方式如下：

```
def send_email_smtp_controller(to, subject, html_content, files=None, dryrun=False, cc=None, bcc=None,
                               mime_subtype='mixed'):
    log = LoggingMixin().log

    custom_alerts = get_custom_alerters()
    email_context = EmailContext(subject, html_content)
    for custom_alert in custom_alerts:
        if custom_alert.sure_called_by_me(email_context):
            log.info("Sent an alert email by custom %s", custom_alert.__class__)
            custom_alert.send_email(to, subject, html_content, files, dryrun, cc, bcc, mime_subtype)
            return

    log.info("Sent an alert email by default %s", "send_email_smtp")
    send_email_smtp(to, subject, html_content, files, dryrun, cc, bcc, mime_subtype)


```

其中，get_custom_alerters 是工厂函数，用于产生所有的 alerters，每个 alerter 通过 sure_called_by_me 检查调用者来向和自己的处理范围，通过 send_email 自定义邮件的发送。

Airflow adapter
---------------

这种方法的实现原理，是基于 python 的导入操作的原理，python 的倒入模块会进行 3 步处理，即：找到模块对应的文件、编译此文件、运行此文件；默认情况下，一个模块文件仅第一倒入时，执行上述 3 个步骤。再次导入此模块，不会执行上述步骤，转而重用已经被倒入到内存中的模块。  
文件导入时，所有的名称赋值操作都会被执行，函数定义也算一种名称赋值操作，即函数名和相应的函数实现相绑定。  
以下面的测试用例为例:  
a.py 内容如下：

```
class TaskInstance:
    def test(self):
        print("a.TaskInstance.test()")
    def sum(self):
        print("a.TaskInstance.sum()")


t = TaskInstance()
t.test()


```

b.py 内容如下：

```
from a import *

def test(self):
    print("b.text()")

TaskInstance.test = test

TaskInstance().test()


```

b 模块中定义了一个函数签名和 a 模块中成员函数完全相同的一个函数，此函数的目的就是为了覆盖 a 模块中的 test 函数。当执行`python b.py`之后其输出如下：

```
a.TaskInstance.test()
b.text()


```

由此，可证明 a 模块中的函数 test 被 b 模块中的函数 test 覆盖了。  
同样，若想改变 models.py 中 email_alert 函数的作用，我们仍然可以添加一个覆盖函数，只需要保证我们新添加的覆盖函数在 models.py 中，email_alert 之后即可。  
这样，接下来就是通过什么方法，能够保证你所添加的功能模块能够在 airflow 原有功能模块之后！！  
其中，可以实现覆盖的方式之一是，重写 airflow 的入口，在函数入口时，首先 import 你想覆盖的函数，之后，再导入覆盖函数覆盖之前导入的同名函数。

源码
==

[https://github.com/ggchangan/custom_alert_email_subject](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fggchangan%2Fcustom_alert_email_subject)