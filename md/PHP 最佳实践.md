> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.ruanyifeng.com](http://www.ruanyifeng.com/blog/2010/12/php_best_practices.html)

今天下午，我在读下面这篇文章。

虽然名字叫[《PHP 最佳实践》](http://www.odi.ch/prog/design/php/guide.php)，但是它主要谈的不是编程规则，而是 PHP 应用程序的合理架构。

它提供了一种逻辑和数据分离的架构模式，属于 [MVC 模式](https://www.ruanyifeng.com/blog/2007/11/mvc.html)的一种实践。我觉得，这是很有参考价值的学习资料，类似的文章网上并不多，所以一边学习，一边就把它翻译了出来。

根据自己的理解，我总结了它的 MVC 模式的实现方式（详细解释见译文）：

> 　　* **视图层（View）**：前端网页；
> 
> 　　* **逻辑层（Controller）**：先是页逻辑（Page Controller），负责处理页面请求；然后，调用业务逻辑（Business Controller），实现具体功能；
> 
> 　　* **数据层（Model）**：数据保存在数据库之中，上面有一个数据库抽象层，再上面则是一个 "数据访问对象"（DAO），它生成 "值对象"（Value Object）。业务逻辑通过 DAO，操作值对象。

=======================================

**PHP 最佳实践**

原载：[http://www.odi.ch/prog/design/php/guide.php](http://www.odi.ch/prog/design/php/guide.php)

译者：阮一峰

  
本文给出了 PHP 程序设计常见问题的解决方法，同时简单描述了 PHP 应用程序的架构。

**1. php.ini 设置**

php.ini 控制了解释器的行为，下面的一些设置保证了你的程序有最大的可移植性。

i. short_open_tag

设为 0，即永远使用 PHP 的长标签形式：<?php echo "hello world"; ?>，不用短标签形式 <?= "hello world" ?>。

ii. asp_tags

设为 0，不使用 ASP 标签 <% echo "hello world"; %>。

iii. magic_quotes_gpc

建议在脚本中包含一个全局文件，负责在读取 $_GET、$_POST、$_COOKIE 变量之前，首先检查这个设置是否打开，如果打开了，这对这些变量应用 stripslashes 函数。（注：该设置已经在 PHP 5.3 中被废除。）

iv. register_globals

不要依赖这个设置，永远通过全局变量 $_GET、$_POST、$_COOKIE 去读取 GET、POST 和 COOKIE 的值。为了方便起见，建议声明 $PHP_SELF = $_SERVER['PHP_SELF']。

v. file_uploads

上传文件的最大大小，由下面的设置决定：

> 　　* file_uploads 必须设为 1（默认值），表示允许上传。
> 
> 　　* memory_limit 必须略大于 post_max_size 和 upload_max_filesize。
> 
> 　　* post_max_size 和 upload_max_filesize 要足够大，能满足上传的需要。

**2. 配置文件（configuration file）**

你应该把与应用程序相关的所有配置，写在一个文件里。这样你就能很方便地适应开发环境的变化。配置文件通常包含以下信息：数据库参数、email 地址、各类选项、debug 和 logging 输出开关、应用程序常数。

**3. 名称空间（namespace）**

选择类和函数名的时候，必须很小心，避免出现重名。尽可能不要在类以外，放置全局性函数，类对内部的属性和方法，相当于有一层名称空间保护。如果你确实有必要声明全局性函数，那么使用一个前缀，比如 dao_factory()、 db_getConnection()、text_parseDate() 等等。

**4. 数据库抽象层**

PHP 不提供数据库操作的通用函数，每种数据库都有一套自己的函数。你不应该直接使用这些函数，否则一旦改用其他数据库（比如从 MySQL 转为 Oracle），你就有大麻烦了。而且，数据库抽象层通常比系统本身的数据库函数，更易用一些。

**5. "值对象"（Value Object, VO）**

值对象（VO）在形式上，就像 C 语言的 struct 结构。它是一个只包含属性、不包含任何方法（或只包含很少方法）的类。一个值对象，就对应一个实体。它的属性，通常应该与数据库的字段名保持相同。此外，还应该有一个 ID 属性。

> 　　class Person {
> 
> 　　　　var $id, $first_name, $last_name, $email;
> 
> 　　}

**6. 数据访问对象（Data Access Object, DAO）**

数据访问对象（DAO）的作用，主要是将数据库访问与其他代码相隔离。DAO 应该是可以叠加（stacked）的，这样就有利于将来你再添加数据库缓存。每一个值对象的类，都应该有自己的 DAO。

> 　　class PersonDAO {  
> 　　　　var $conn;
> 
> 　　　　function PersonDAO(&$conn) {  
> 　　　　　　$this->conn =& $conn;  
> 　　　　}
> 
> 　　　　function save(&$vo) {  
> 　　　　　　if ($v->id == 0) {  
> 　　　　　　　　$this->insert($vo);  
> 　　　　　　} else {  
> 　　　　　　　　$this->update($vo);  
> 　　　　　　}  
> 　　　　}
> 
>   
> 　　　　function get($id) {  
> 　　　　　　#execute select statement  
> 　　　　　　#create new vo and call getFromResult  
> 　　　　　　#return vo  
> 　　　　}
> 
> 　　　　function delete(&$vo) {  
> 　　　　　　#execute delete statement  
> 　　　　　　#set id on vo to 0  
> 　　　　}
> 
> 　　　　#-- private functions
> 
> 　　　　function getFromResult(&vo, $result) {  
> 　　　　　　#fill vo from the database result set  
> 　　　　}
> 
> 　　　　function update(&$vo) {  
> 　　　　　　#execute update statement here  
> 　　　　}
> 
> 　　　　function insert(&$vo) {  
> 　　　　　　#generate id (from Oracle sequence or automatically)  
> 　　　　　　#insert record into db  
> 　　　　　　#set id on vo  
> 　　　　}  
> 　　}

DAO 通常应该部署以下方法：

> 　　* save：插入或更新一条记录  
> 　　* get：取出一条记录  
> 　　* delete：删除一条记录

你可以根据自己的需要，添加其他 DAO 方法，常见的例子有 isUsed()、getTop($n)、find($criteria)。

但是，所有的 DAO 方法都应该与数据库操作有关，不应该执行其他操作。DAO 只应该对一张表进行基本的 select / insert / update，不应该包含业务逻辑。举例来说，PersonDAO 就不应该包含向某人发送 Email 的代码。

你可以写一个工厂函数，根据不同的类名，返回相应的 DAO。

> 　　function dao_getDAO($vo_class) {
> 
> 　　　　$conn = db_conn('default'); #get a connection from the pool
> 
> 　　　　switch ($vo_class) {
> 
> 　　　　　　case "person": return new PersonDAO($conn);
> 
> 　　　　　　case "newsletter": return new NewsletterDAO($conn);
> 
> 　　　　　　...
> 
> 　　　　}
> 
> 　　}  

**7. 自动生成代码**

99% 的值对象和 DAO 代码，可以根据数据库模式（schema）自动生成，前提是你的表和列使用约定的方式进行命名。如果你修改数据库模式，一个自动生成代码的脚本将大大节省你的时间。

**8. 业务逻辑**

业务逻辑直接反映使用者的需要。它们处理值对象，根据业务需要修改值对象的属性，使用 DAO 与数据库层交互。

> 　　class NewsletterLogic {  
> 　　　　function NewsletterLogic() {  
> 　　　　　　...  
> 　　　　}
> 
> 　　　　function subscribePerson(&$person) {  
> 　　　　　　...  
> 　　　　}
> 
> 　　　　function unsubscribePerson(&$person) {  
> 　　　　　　...  
> 　　　　}
> 
> 　　　　function sendNewsletter(&$newsletter) {  
> 　　　　　　...  
> 　　　　}  
> 　　}  

**9. 页逻辑（控制器）**

当一个网页被请求时，页控制器（page controller）就会运行，然后产生输出。控制器的任务，就是将 HTTP 请求转化成业务对象（business object），然后调用相应的业务逻辑，最后生成一个 "展示输出" 的对象。

页逻辑依次执行以下步骤（请参照后面的 PageController 类的代码）：

> 　　i. 假定页面请求之中，包含一个 cmd 参数。
> 
> 　　ii. 根据 cmd 参数的值，执行相应的动作。
> 
> 　　iii. 验证页面返回的值，生成一个值对象。
> 
> 　　iv. 针对值对象，执行业务逻辑。
> 
> 　　v. 如果有必要，可以导向另一个页面。
> 
> 　　vi. 收集必要的数据，输出结果。

注意：可以编写一个工具函数（utility function），处理 GET 或 POST 值，当有的变量没有赋值时，提供一个默认值。页逻辑不包含 HTML 代码。

> 　　class PageController {  
> 　　　　var $person; #$person is used by the HTML page  
> 　　　　var $errs;
> 
> 　　　　function PageController() {  
> 　　　　　　$action = Form::getParameter('cmd');  
> 　　　　　　$this->person = new Person();  
> 　　　　　　$this->errs = array();
> 
> 　　　　　　if ($action == 'save') {  
> 　　　　　　　　$this->parseForm();  
> 　　　　　　　　if (!this->validate()) return;
> 
> 　　　　　　　　NewsletterLogic::subscribe($this->person);
> 
> 　　　　　　　　header('Location: confirmation.php');  
> 　　　　　　　　exit;  
> 　　　　　　}  
> 　　　　}
> 
>   
> 　　　　function parseForm() {  
> 　　　　　　$this->person->name = Form::getParameter('name');  
> 　　　　　　$this->person->birthdate = Util::parseDate(Form::getParameter('birthdate');  
> 　　　　　　...  
> 　　　　}
> 
> 　　　　function validate() {  
> 　　　　　　if ($this->person->name == '') $this->errs['name'] = FORM_MISSING;  
> 　　　　　　#FORM_MISSING is a constant  
> 　　　　　　...  
> 　　　　　　return (sizeof($this->errs) == 0);  
> 　　　　}  
> 　　}

**10. 表现层（Presentation Layer）**

最顶层的页面包含实际的 HTML 代码。这个页面需要的所有业务对象（business object），由页逻辑提供。

这个页面先读取业务对象的属性，然后将它们转换成 HTML 格式。

> 　　<?php  
> 　　　　require_once('control/ctl_person.inc.php'); #the page controller  
> 　　　　$c =& new PageController();  
> 　　?>
> 
> 　　<html>  
> 　　<body>  
> 　　<form action="<?php echo htmlspecialchars($PHP_SELF) ?>" method="POST">  
> 　　　　<input type="hidden" >  
> 　　　　<input type="text"  
> value="<?php echo htmlspecialchars($c->person->name); ?>">  
> 　　　　<button type="submit">Subscribe</button>  
> 　　</form>  
> 　　</body>  
> 　　</html>

**11. 本地化（Localization）**

本地化意味着要支持多种语言，这个比较麻烦，你无非有两种方法可以选择：

> 　　A) 准备多重页面。
> 
> 　　B) HTML 页面中去除特定语言相关的内容。

一般来说，A 方法用得比较多，因为 B 方法会使得 HTML 页面的可读性很差。

所以，你可以先写完一种语言的页面，然后把它们进行拷贝，用某种命名法区别不同语言的版本，比如 index_fr.php 表示 index.php 的法语版。

为了保存用户的语言选择，你有几种方法：

> 　　A) 将语言设定保存在一个 session 变量或 cookie 之中；
> 
> 　　B) 从 HTTP 头中读取 locale 值；
> 
> 　　C) 把语言设定作为一个参数，追加在每个 URL 后面。

看上去 A 方法比 C 方法容易得多（虽然 session 和 cookie 都有过期的问题），而 B 方法只能作为 A 或 C 的补充。

最后不要忘了，数据库中的字段也必须进行本地化。

**12. 安装位置**

有时候你需要知道程序的根目录在哪里，但是 $_SERVER['DOCUMENT_ROOT'] 只是 web 服务器的根目录，如果你的程序安装在它的某个子目录之中，PHP 没法自动知道。

你可以定义一个全局变量 $ROOT，它的值就是程序的根目录，然后把它包含在每一个脚本文件中。那么，你要包含某个文件，就这样写 require_once("$ROOT/lib/base.inc.php");。

**13. 目录结构**

首先，每个类都应该有自己的独立文件，还必须有一套文件名的命名规则（naming convention）。

软件的目录结构可以采用如下形式：

> 　　/ 根目录。浏览器从这个页面开始访问。
> 
> 　　/lib/ 包含全局变量（base.inc.php）和配置文件（config.inc.php）。
> 
> 　　/lib/common/ 包含其他项目也可以共用的库，比如数据库抽象层。
> 
> 　　/lib/model/ 包含值对象类。
> 
> 　　/lib/dao/ 包含数据访问对象（DAO）类，以及 DAO 工厂函数。
> 
> 　　/lib/logic/ 包含业务逻辑类。
> 
> 　　/parts/ 包含 HTML 模板文件。
> 
> 　　/control/ 包含页逻辑。对于大型程序来说，这个目录下面可能还有子目录（比如 admin/, /pub/）。

base.inc.php 文件中，应该按照以下顺序添加包含文件：

> 　　* /lib/common 之中经常使用的类（比如数据库层）。
> 
> 　　* 配置文件；
> 
> 　　* /lib/model 之中所有类；
> 
> 　　* /lib/dao 的之中所有类。

至于那些存放图片、上传文件的目录，这里就省略了。