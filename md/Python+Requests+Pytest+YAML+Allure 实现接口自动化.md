> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI5OTI4OTY5NQ==&mid=2247495899&idx=3&sn=1914e5804801804fd84771e44a23df9d&chksm=ec9a6debdbede4fd8adb24bdd3d6b4950bf20eebc2cfd8c96d2287d1652dc7884dced7188e5b&mpshare=1&scene=1&srcid=0727jXKiE1scsQuuWkz0gGDA&sharer_sharetime=1627372958295&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **作者：wintest**  
> 
> **链接：https://www.cnblogs.com/wintest/p/13423231.html**

本项目实现接口自动化的技术选型：**Python+Requests+Pytest+YAML+Allure** ，主要是针对之前开发的一个接口项目来进行学习，通过 Python+Requests 来发送和处理 HTTP 协议的请求接口，使用 Pytest 作为测试执行器，使用 YAML 来管理测试数据，使用 Allure 来生成测试报告。  

> **接口项目开发学习：**
> 
> 使用 Flask 开发简单接口 (1)--GET 请求接口 (https://www.cnblogs.com/wintest/p/12728095.html) 
> 
> 使用 Flask 开发简单接口 (2)--POST 请求接口 (https://www.cnblogs.com/wintest/p/12731508.html) 
> 
> 使用 Flask 开发简单接口 (3)-- 引入 MySQL (https://www.cnblogs.com/wintest/p/12741772.html) 
> 
> 使用 Flask 开发简单接口 (4)-- 借助 Redis 实现 token 验证 (https://www.cnblogs.com/wintest/p/12777340.html)
> 
> 使用 Flask 开发简单接口 (5)-- 数据加密处理 (https://www.cnblogs.com/wintest/p/12780090.html)

### 项目说明

本项目在实现过程中，把整个项目拆分成请求方法封装、HTTP 接口封装、关键字封装、测试用例等模块。

首先利用 Python 把 HTTP 接口封装成 Python 接口，接着把这些 Python 接口组装成一个个的关键字，再把关键字组装成测试用例，而测试数据则通过 YAML 文件进行统一管理，然后再通过 Pytest 测试执行器来运行这些脚本，并结合 Allure 输出测试报告。

当然，如果感兴趣的话，还可以再对接口自动化进行 Jenkins 持续集成。

> GitHub 项目源码地址：https://github.com/wintests/pytestDemo

### 项目结构

*   api ====>> 接口封装层，如封装 HTTP 接口为 Python 接口
    
*   common ====>> 各种工具类
    
*   core ====>> requests 请求方法封装、关键字返回结果类
    
*   config ====>> 配置文件
    
*   data ====>> 测试数据文件管理
    
*   operation ====>> 关键字封装层，如把多个 Python 接口封装为关键字
    
*   pytest.ini ====>> pytest 配置文件
    
*   requirements.txt ====>> 相关依赖包文件
    
*   testcases ====>> 测试用例
    

### 请求方法封装

在 `core/rest_client.py` 文件中，对 `Requests` 库下一些常见的请求方法进行了简单封装，以便调用起来更加方便。

```
class RestClient():

    def __init__(self, api_root_url):
        self.api_root_url = api_root_url
        self.session = requests.session()

    def get(self, url, **kwargs):
        return self.request(url, "GET", **kwargs)

    def post(self, url, data=None, json=None, **kwargs):
        return self.request(url, "POST", data, json, **kwargs)

    def put(self, url, data=None, **kwargs):
        return self.request(url, "PUT", data, **kwargs)

    def delete(self, url, **kwargs):
        return self.request(url, "DELETE", **kwargs)

    def patch(self, url, data=None, **kwargs):
        return self.request(url, "PATCH", data, **kwargs)

    def request(self, url, method, data=None, json=None, **kwargs):
        url = self.api_root_url + url
        headers = dict(**kwargs).get("headers")
        params = dict(**kwargs).get("params")
        files = dict(**kwargs).get("params")
        cookies = dict(**kwargs).get("params")
        self.request_log(url, method, data, json, params, headers, files, cookies)
        if method == "GET":
            return self.session.get(url, **kwargs)
        if method == "POST":
            return requests.post(url, data, json, **kwargs)
        if method == "PUT":
            if json:
                # PUT 和 PATCH 中没有提供直接使用json参数的方法，因此需要用data来传入
                data = complexjson.dumps(json)
            return self.session.put(url, data, **kwargs)
        if method == "DELETE":
            return self.session.delete(url, **kwargs)
        if method == "PATCH":
            if json:
                data = complexjson.dumps(json)
            return self.session.patch(url, data, **kwargs)
```

### HTTP 接口 封装为 Python 接口

在 `api/user.py` 文件中，将上面封装好的 HTTP 接口，再次封装为不同的 Python 接口。不同的 Python 接口，会处理不同 URL 下的请求。

```
class User(RestClient):

    def __init__(self, api_root_url, **kwargs):
        super(User, self).__init__(api_root_url, **kwargs)

    def list_all_users(self, **kwargs):
        return self.get("/users", **kwargs)

    def list_one_user(self, username, **kwargs):
        return self.get("/users/{}".format(username), **kwargs)

    def register(self, **kwargs):
        return self.post("/register", **kwargs)

    def login(self, **kwargs):
        return self.post("/login", **kwargs)

    def update(self, user_id, **kwargs):
        return self.put("/update/user/{}".format(user_id), **kwargs)

    def delete(self, name, **kwargs):
        return self.post("/delete/user/{}".format(name), **kwargs)
```

### 关键字返回结果类

在 `core/result_base.py` 下，定义了一个空类 `ResultBase` ，该类主要用于自定义关键字返回结果。

```
class ResultBase():
    pass

"""
自定义示例：
result = ResultBase()
result.success = False
result.msg = res.json()["msg"]
result.response = res
"""` 
```

在多流程的业务场景测试下，通过自定义期望保存的返回数据值，以便更好的进行断言。

### 关键字封装

关键字应该是具有一定业务意义的，在封装关键字的时候，可以通过调用多个 Python 接口来完成。在某些情况下，比如测试一个充值接口的时候，在充值后可能需要调用查询接口得到最新账户余额，来判断查询结果与预期结果是否一致，那么可以这样来进行测试：

*   1, 首先，可以把 `充值-查询` 的操作封装为一个关键字，在这个关键字中依次调用充值和查询的接口，并可以自定义关键字的返回结果。
    
*   2, 接着，在编写测试用例的时候，直接调用关键字来进行测试，这时就可以拿到关键字返回的结果，那么断言的时候，就可以直接对关键字返回结果进行断言。
    

### 测试用例层

**根据用例名分配测试数据**

测试数据位于 `data` 文件夹下，在这里使用 `YAML` 来管理测试数据，同时要求测试数据中第一层的名称，需要与测试用例的方法名保持一致，如 `test_get_all_user_info` 、`test_delete_user`。

```
test_get_all_user_info:
  # 期望结果,期望返回码,期望返回信息
  # except_result, except_code, except_msg
  - [True, 0, "查询成功"]
省略
test_delete_user:
  # 删除的用户名,期望结果,期望返回码,期望返回信息
  # username, except_result, except_code, except_msg
  - ["测试test", True, 0, "删除用户信息成功"]
  - ["wintest3", False, 3006, "该用户不允许删除"]
```

这里借助 `fixture` 方法，我们就能够通过 `request.function.__name__` 自动获取到当前执行用例的函数名 `testcase_name` ，当我们传入测试数据 `api_data` 之后，接着便可以使用 `api_data.get(testcase_name)` 来获取到对应用例的测试数据。

```
import pytest
from testcases.conftest import api_data

@pytest.fixture(scope="function")
def testcase_data(request):
    testcase_name = request.function.__name__
    return api_data.get(testcase_name)`
```

**数据准备和清理**

在接口自动化中，为了保证用例可稳定、重复地执行，我们还需要有测试前置操作和后置操作，即数据准备和数据清理工作。

```
@pytest.fixture(scope="function")
def delete_register_user():
    """注册用户前，先删除数据，用例执行之后，再次删除以清理数据"""
    del_sql = base_data["init_sql"]["delete_register_user"]
    db.execute_db(del_sql)
    logger.info("注册用户操作：清理用户--准备注册新用户")
    logger.info("执行前置SQL：{}".format(del_sql))
    yield # 用于唤醒 teardown 操作
    db.execute_db(del_sql)
    logger.info("注册用户操作：删除注册的用户")
    logger.info("执行后置SQL：{}".format(del_sql))
```

在这里，以用户注册用例为例。对于前置操作，我们应该准备一条删除 SQL，用于将数据库中已存在的相同用户删除，对于后置操作，我们应该再执行删除 SQL，确保该测试数据正常完成清理工作。

在测试用例中，我们只需要在用例上传入 `fixture` 的函数参数名 `delete_register_user` ，这样就可以调用 `fixture` 实现测试前置及后置操作。当然，也可以使用 pytest 装饰器 `@pytest.mark.usefixtures()` 来完成，如：

```
@pytest.mark.usefixtures("delete_register_user")` 
```

**Allure 用例描述**

在这里，我们结合 Allure 来实现输出测试报告，同时我们可以使用其装饰器来添加一些用例描述并显示到测试报告中，以便报告内容更加清晰、直观、可读。如使用 `@allure.title()` 自定义报告中显示的用例标题，使用 `@allure.description()` 自定义用例的描述内容，使用 `@allure.step()` 可在报告中显示操作步骤，使用 `@allure.issue()` 可在报告中显示缺陷及其链接等。

```
@allure.step("步骤1 ==>> 注册用户")
def step_1(username, password, telephone, sex, address):
    logger.info("步骤1 ==>> 注册用户 ==>> {}, {}, {}, {}, {}".format(username, password, telephone, sex, address))

@allure.severity(allure.severity_level.NORMAL)
@allure.epic("针对单个接口的测试")
@allure.feature("用户注册模块")
class TestUserRegister():
    """用户注册"""
    @allure.story("用例--注册用户信息")
    @allure.description("该用例是针对获取用户注册接口的测试")
    @allure.issue("https://www.cnblogs.com/wintest", name="点击，跳转到对应BUG的链接地址")
    @allure.testcase("https://www.cnblogs.com/wintest", name="点击，跳转到对应用例的链接地址")
    @allure.title(
        "测试数据：【 {username}，{password}，{telephone}，{sex}，{address}，{except_result}，{except_code}，{except_msg}】")
    @pytest.mark.single
    @pytest.mark.parametrize("username, password, telephone, sex, address, except_result, except_code, except_msg",
                             api_data["test_register_user"])
    @pytest.mark.usefixtures("delete_register_user")
    def test_delete_user(self, login_fixture, username, except_result, except_code, except_msg):
省略` 
```

### 项目部署

首先，下载项目源码后，在根目录下找到 `requirements.txt` 文件，然后通过 pip 工具安装 requirements.txt 依赖，执行命令：

```
pip3 install -r requirements.txt` 
```

接着，修改 `config/setting.ini` 配置文件，在 Windows 环境下，安装相应依赖之后，在命令行窗口执行命令：

```
pytest
```

**注意**：因为我这里是针对自己的接口项目进行测试，如果想直接执行我的测试用例来查看效果，需要提前部署上面提到的接口项目。

### 测试报告效果展示

在命令行执行命令：`pytest` 运行用例后，会得到一个测试报告的原始文件，但这个时候还不能打开成 HTML 的报告，还需要在项目根目录下，执行命令启动 `allure` 服务：

```
# 需要提前配置allure环境，才可以直接使用命令行
allure serve ./report`
```

最终，可以看到测试报告的效果图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/HZW0wwFxbQAdvSRUD0RbOPSqvib8IXWsry8lquAw5rSUicIebht2YicVusDFib5F1IrlQuKlofzV1bqg7bwBy22zZg/640?wx_fmt=png)

image.png

来源：https://www.cnblogs.com/wintesl