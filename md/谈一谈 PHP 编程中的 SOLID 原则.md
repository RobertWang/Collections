> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FAwaZIgDazXY6aapkKguXQ)

> image.png 什么是 SOLID 原则软件设计有很多原则，比如软件设计上的 SOLID principle，

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQcKNIRiafVAkwFmFE8OD9v6X6sOIWQhRbVmw2Gibsiab4Lgotg2XACgQbGd8A2EhOqJFbVlrH0WMl3Gw/640?wx_fmt=png)

什么是 SOLID 原则
============

软件设计有很多原则，比如软件设计上的 SOLID principle，单元测试中的 FIRST 和 AAA，代码实现上的 DRY principle 等。熟悉这些原则，可以把我们的经验上升到理论高度，有利于程序员的成长，也便于团队带头人和组员控制软件质量。其中面向对象设计的 **SOLID 原则**是创建易于维护、扩展和测试的软件的基本指南。 SOLID 是下面几个英文词组的缩写

1.  1. **S**ingle-Responsibility principle (单一职责原则)
    
2.  2. **O**pen-Close principle (开放扩展，封闭修改)
    
3.  3. **L**iskov substitution principle (Liskov 替换原则)
    
4.  4. **I**nterface **s**egregation principle (接口隔离原则)
    
5.  5. **D**ependency inversion principle （依赖反转原则)
    

在本文中，我们将提供代码示例来演示如何在 PHP 编程中应用 SOLID 原则。

单一职责原则 (SRP)
============

单一职责原则 (SRP) 指出一个类应该只有一个职责。 这意味着一个类应该只有一个改变的理由。 让我们看一下如何在 PHP 编程中应用 SRP 的示例。

```
// Example of a class with multiple responsibilities
class User
{
 public function login($username, $password)
 {
 // Authenticate user
 }
public function updateProfile($userId, $data)
 {
 // Update user profile
 }
public function sendEmail($userId, $subject, $body)
 {
 // Send email to user
 }
}
// Refactored class with single responsibility
class Authenticator
{
 public function login($username, $password)
 {
 // Authenticate user
 }
}
class ProfileUpdater
{
 public function update($userId, $data)
 {
 // Update user profile
 }
}
class EmailSender
{
 public function send($userId, $subject, $body)
 {
 // Send email to user
 }
}

```

在上面的示例中，我们从一个名为 “User” 的类开始，该类具有多项职责：身份验证、配置文件更新和电子邮件发送。 然后我们将该类重构为三个独立的类，每个类都有一个职责：身份验证、配置文件更新和电子邮件发送。

开闭原则（OCP）
=========

开闭原则 (OCP) 指出软件实体应该对扩展开放，对修改关闭。 这意味着您应该能够在不修改其源代码的情况下扩展类的行为。 让我们看一下如何在 PHP 编程中应用 OCP 的示例。

```
// Example of a class without OCP
class Car
{
 public function drive()
 {
 // Drive the car
 }
}
// Extended class with OCP
class CarWithNavigation extends Car
{
 public function navigate()
 {
 // Navigate using GPS
 }
}

```

在上面的示例中，我们从一个名为 “Car” 的类开始，该类只有一个驾驶汽车的方法。 然后我们扩展该类以创建一个名为 “CarWithNavigation” 的新类，该类具有使用 GPS 进行导航的附加方法。 通过扩展原始类，我们能够在不修改原始类源代码的情况下添加新功能。

里氏替换原则 (LSP)
============

Liskov 替换原则 (LSP) 指出超类的对象应该能够被其子类的对象替换而不影响程序的正确性。 让我们看一下如何在 PHP 编程中应用 LSP 的示例。

```
// Example of a class that violates LSP
class Rectangle
{
    public function setWidth($width)
    {
        $this->width = $width;
    }

    public function setHeight($height)
    {
        $this->height = $height;
    }

    public function getArea()
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth($width)
    {
        $this->width = $width;
        $this->height = $width;
    }

    public function setHeight($height)
    {
        $this->height = $height;

```

接口隔离原则（ISP）
===========

“I” 代表面向对象设计 SOLID 原则中的 Interface Segregation Principle（ISP）。 ISP 声明不应强迫客户端依赖于它们不使用的接口。 换句话说，拥有许多小的、特定的接口比拥有几个大的、通用的接口要好。  
让我们看一个如何在 PHP 编程中应用 ISP 的示例。

```
// Example of a class that violates ISP
interface Document
{
    public function open();
    public function save();
    public function close();
}

class TextDocument implements Document
{
    public function open()
    {
        // Open text document
    }

    public function save()
    {
        // Save text document
    }

    public function close()
    {
        // Close text document
    }
}

class PdfDocument implements Document
{
    public function open()
    {
        // Open PDF document
    }

    public function save()
    {
        // Save PDF document
    }

    public function close()
    {
        // Close PDF document
    }
}

```

在上面的示例中，我们从一个名为 “Document” 的接口开始，该接口具有三个方法：打开、保存和关闭。 然后我们创建了两个实现 Document 接口的类：TextDocument 和 PdfDocument。 但是，TextDocument 类不需要 close 方法，因为它只处理文本文档。 同样，PdfDocument 类不需要保存方法，因为 PDF 文档通常是只读的。 通过将 Document 接口分解为更小、更具体的接口，我们可以避免强制客户端依赖于它们不使用的方法。  
要在 PHP 编程中应用 ISP，您的目标应该是创建特定于客户需求的接口。 这将使您的代码更加模块化，更易于维护，并且更不容易出错。

依赖反转原则（DIP）
===========

D” 代表面向对象设计 SOLID 原则中的 Dependency Inversion Principle（DIP）。 DIP 指出高级模块不应该依赖于低级模块，但两者都应该依赖于抽象。 另外，抽象不应该依赖于细节，而细节应该依赖于抽象。  
让我们看一下如何在 PHP 编程中应用 DIP 的示例。

```
// Example of a class that applies DIP
interface PaymentGateway
{
    public function charge($amount);
}

class PaymentProcessor
{
    private $gateway;

    public function __construct(PaymentGateway $gateway)
    {
        $this->gateway = $gateway;
    }

    public function processPayment($amount)
    {
        $this->gateway->charge($amount);
    }
}

class PaypalGateway implements PaymentGateway
{
    public function charge($amount)
    {
        // Charge payment using Paypal API
    }
}

```

在更新的示例中，我们引入了 PaypalGateway 类实现的 PaymentGateway 接口。 PaymentProcessor 类现在依赖于 PaymentGateway 接口而不是 PaypalGateway 类，使其更灵活，将来更容易更改。 我们现在可以轻松地将 PaypalGateway 类换成另一个实现 PaymentGateway 接口的支付网关。  
要在 PHP 编程中应用 DIP，您的目标应该是创建将高级模块与低级模块隔离开来的抽象。 这将使您的代码更加模块化，更易于维护，并且更不容易出错。