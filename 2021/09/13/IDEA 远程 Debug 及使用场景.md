> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247522658&idx=2&sn=921871e3720b37338760793c58da0932&chksm=ce0e2ee1f979a7f7954bcc48a571a99272f5ccc1d508b1e11f592a5918fdc9649f328fc70fd2&mpshare=1&scene=1&srcid=09134gpChg6NstpH58j6iQGQ&sharer_sharetime=1631530251562&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

说到远程 Debug 这个功能，基本上大多 IDE 都会自带，笔者切换到 IDEA 之后，还真再就没用过远程 Debug，直到昨天发现了一个非常基础的错误...

### **坑从何来**

### 坑来自于我的开源小工具， V-Mock. 笔者本意是打造一个，简单，轻巧，一键运行的**接口模拟系统**，用来方便**等待他人接口**的前端后端同学.

基于以上目的，我使用了嵌入式数据库 **sqlite**，来配合 Springboot，构造了无须配置，一行启动的小 jar 包. 目录结构如下，数据库直接扔在了 Resource 中：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSuWjUpaiaNqyFicPbFbADRWB0tAicSribXmiaXKpS5dLoS8rJofFD9A2MKqQ/640)

升级版本的同学，发现数据没了，笔者暂时给出了方案，**嵌入式数据库嘛，把旧 jar 中的 DB 文件，覆盖到新 Jar 中就好了**

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSM2SsiaJLmLGlRnhD8c7Iic9vFkUQsPUle3picaYDPxd6EmB1NpQQzULFQ/640)

说出这句话的时候，也不能完全赖🧠瓦特了，毕竟 Springboot+Sqlite 这种奇葩组合也是为了工具的小巧性，偶尔尝试的产物.

事实上稍微想想，db 文件和其他资源不一样，是要频繁改写的，当然改动的不是 jar 包中的原始文件.

直到收到了一个 Issues，告诉了笔者 DB 文件复制到新 jar 中并没有生效.

笔者也迅速反应过来，怎么可能用的 jar 内的 DB 文件，真实文件不出意外是放在 java.io.tmpdir 下了.

java.io.tmpdir 的路径，一般情况下，macos 是在 **$TMPDIR**，win 则在 **%temp%**.

笔者也切换到了对应的目录，终于看到了 jar 运行时真实使用的 DB 文件：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSxRX4qMeonSmFYvvpIaeyHo8z5z9a2uyG9ZVyDbkXAmzaqBlnQeOGjQ/640)

但是这个命名方式很奇怪啊，和原本的 v-mock.sqlite 并不沾边.

一路追随 sqlite 的 jdbc 驱动源码，找到了`org.sqlite.SQLiteConnection`的`extractResource`方法，看到了命名代码：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSgHsa3XUQQDffyMwrGPYa2icaibu1Yhz2Qp65z4Na3WjeysCcISm4oy0Q/640)

其实看到这已经清晰了，源码中使用了 **sqlite-jdbc-tmp** 拼接了原始 jar 中 DB 文件的 URL 类的 hashcode 作为文件名.

之所以笔者**开发的时候没注意到**，看看这个方法**第一个 if** 判断就知道了。

笔者习惯用 IDE 中的 Springboot 或者 Application 模式直接启动项目，并不是打包后的启动方式

所以当`Protocol`是`file`而不是`jar`的情况，直接就使用了`target/classes/db/v-mock.sqlite`文件，不用生成临时文件.

开发时，DB 可视化工具也连接的是`target/classes/db/v-mock.sqlite`，所以当时并没发现疑点.

事实上这是很正常的操作，很多地方的源码都有判断是**普通 web 环境**还是以 **jar** 运行的，如果有这方面的调试，要思考你的启动方式了.

那么想把断点打在**第一个 if 之后**，看到效果，**选择之一**就是可以使用**远程 Debug** 的方式.

### **IDEA 的远程 Debug**

IDEA 的远程 Debug 模块真的是设计十分贴心，傻瓜操作，命令都生成好了，不知道现在的 eclipse 版本有没有这么贴心.

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSHiboarBVLDd7oaTYYt6giadwKaPfialwjKLiagDeQF0GwibiclqljKLT9sWA/640)

从 configuration 中搜索 **remote** 模版，点击右上角的 **create configuration**，就创建好了一个**远程 debug** 启动方式.

**Debugger mode** 选择 **Attach to remote JVM** 即可，它还有一个选项是 **Listen to remote JVM**，意如其名嘛，一个是主动附着到启动的程序，一个是被动监听程序。

**ip 和端口**不用多说，笔者直接用的本地 jar 包，所以填了 **localhost**，右边 jdk 版本如果使用其他版本的，需要调一下。

中间的文本框就是生成好的 jvm 参数了，非常人性化了，直接加入启动命令即可  
`java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 v-mock.jar`

可以完全不用管命令什么意思，如果你想知道，笔者也大概解释一下：

*   -agentlib:jdwp  最重要的参数，启动 JDWP 代理，JDWP 全称就是 Java Debug Wire Protocol，官方给的方便调试的工具.
    
*   transport=dt_socket  通过 socket 方式传输数据，dt 八成就是 data transfer 的缩写了.
    
*   server=y  开启调试 server 端，注意，因为笔者上文选择的是 Attach to remote JVM，所以这里才是 y，等待有调试器 Attach 过来，如果你选了 Listen 模式，那么就是反过来的，调试器是 server，这里就是 n 了.
    
*   suspend=n 是否挂起，这里设置为 n，也就是说程序正常跑，什么时候需要 Attach 就去 Attach 即可，如果设置为 y，程序将会等待调试器 Attach 上才会继续执行，比如启动源码的调试场景.
    
*   address=5005  调试端口设置为 5005，当然其它端口也可以.
    

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSTv0LIASah6tQia4tkc6PCahXXgAhNmjRiadCH8XmKBpu0HLLNN5020qA/640)

启动 jar 包，再以刚才创建的方式进行 debug，期待的断点位置已经成功到达了.

**结尾**
------

找到原因后，笔者也顺着 sqliteJDBC 源码中命名的方式，开发了一种数据迁移的方式，即：自动或手动找到之前版本的数据库文件，复制一份，以新版本的 hashcode 命名即可.

Spring Boot + 嵌入式数据库，是个非常 “**不正经**” 的组合，但是开发**小工具**还是蛮不错的，如果有遇到同样数据迁移问题的，不妨参考一下笔者的解决方案😄