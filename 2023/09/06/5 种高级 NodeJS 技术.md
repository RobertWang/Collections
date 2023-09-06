> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651616550&idx=2&sn=fed0c44d18b0dde22d116b685c5be850&chksm=8022a2e7b7552bf1ec96679e0cf9740f4bd4255fe527f20b6eba3ac137de940ee9cc9eb7b322&scene=21#wechat_redirect)

作为开发人员，我们都致力于打造高效、健壮且易于理解、修改和扩展的代码库。通过采用最佳实践和探索先进技术，我们可以释放 NodeJS 的真正潜力并显着提高应用程序的质量。在这篇文章中，我们将重点介绍 NodeJS 的五种高级技术。所以，系好安全带，我们要开车了，准备好探索它们吧。

**1. 添加中间件**

不要将中间件添加到每个路由，而是使用 use 方法将其添加到路由列表的顶部。这样，中间件下面定义的任何路由都会在到达各自的路由处理程序之前自动通过中间件。

```
const route = express.Router();
const {login} = require("../controllers/auth");
route.get('/login', login)
// isAuthenticated is middleware that checks whether 
// you are authenticated or not
// // ❌ Avoid this: middleware on each route
route.get('/products', isAuthenticated, fetchAllProducts);
route.get('/product/:id', isAuthenticated, getProductById)

```

```
// ✅ Instead, do this
// Route without middleware
route.get('/login', login)
// Middleware function: isAuthenticated
// This will be applied to all routes defined after this point
route.use(isAuthenticated);
// Routes that will automatically check the middleware
route.get('/products', fetchAllProducts);
route.get('/product/:id', getProductById);

```

这种方法有助于保持代码的组织性，并避免为每个路由单独重复中间件。

**2. 使用全局错误处理**

我们可以使用 NodeJS 全局错误处理功能，而不是在每个控制器上构建错误响应。首先，创建一个派生自内置 Error 类的自定义 AppError 类。此自定义类允许您使用 statusCode 和 status 等附加属性来自定义错误对象。

```
// Custom Error class
module.exports = class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = statusCode < 500 ? "error" : "fail";
    Error.captureStackTrace(this, this.constructor);
  }
};

```

创建自定义错误类后，请在根路由器文件中添加全局错误处理程序中间件。该中间件函数采用四个参数（err、req、res、next）并处理整个应用程序中的错误。 

在全局错误处理程序中，您可以根据错误对象的 statusCode、status 和 message 属性来格式化错误响应。 

您可以自定义此响应格式以满足您的需求。此外，还包括用于开发环境的堆栈属性。

```
// Express setup
const express = require('express');
const app = express();
app.use('/', (req, res) => {
  res.status(200).json({ message: "it works" });
});
app.use('*', (req, res) => {
    res.status(404).json({
        message: `Can't find ${req.originalUrl} this route`,
    });
});
// 👇 add a global error handler after all the routes.
app.use((err, req, res, next) => {
  err.status = err.status || "fail";
  err.statusCode = err.statusCode || 500;
  res.status(err.statusCode).json({
    status: err.status,
    message: transformMessage(err.message),
    stack: process.env.NODE_ENV === "development" ? err.stack : undefined,
  });
});

```

添加后，您可以使用 next(new AppError(message, statusCode)) 抛出错误。下一个函数会自动将错误传递给全局错误处理程序中间件。

```
// inside controllers
// route.get('/login', login);
exports.login = async (req, res, next) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email }).select("+password +lastLoginAt");
    if (!user || !(await user.correctPassword(password, user.password))) {
      // 👇 like this
      return next(new AppError("Invalid Email / Password / Method", 404));
    }
     // Custom logic for generating a token
    const token = 'generated_token';
    res.status(200).json({ token });
  } catch(error) {
      next(error
  }
});

```

总体而言，这种方法通过将错误处理集中在一个位置来简化错误处理，从而更轻松地在应用程序中维护和自定义错误响应。

**3. 使用自定义 Try-Catch 函数**

我们可以使用实现相同目的的自定义函数，而不是使用 try-catch 块手动包装每个控制器函数。

```
// ❌ Avoid this
// Using try-catch block each controllers
exports.login = async (req, res, next) => {
  try {
    // logic here
  } catch(error) {
      res.status(400).json({ message: 'You error message'}
  }
});

```

tryCatchFn 函数接受函数 (fn) 作为输入，并返回一个用 try-catch 块包装原始函数的新函数。 

如果在包装函数内发生错误，则使用 catch 方法捕获错误，并将错误传递到下一个函数以由全局错误处理程序处理。

```
// ✅ Instead, do this
const tryCatchFn = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
}
// To use this custom function, you can wrap your controller 
// functions with tryCatchFn:
exports.login = tryCatchFn(async (req, res, next) => {
  // logic here
});

```

通过使用 tryCatchFn 包装控制器函数，您可以确保自动捕获这些函数中引发的任何错误并将其传递给全局错误处理程序，从而无需单独添加 try-catch 块。

这种方法有助于以更清晰、更简洁的方式集中错误处理，使代码更易于维护并减少重复的错误处理代码。

**4. 将主文件分成两部分。**

使用 Express 开发 NodeJS 应用程序时，通常有一个包含所有业务逻辑、路由定义和服务器设置的主文件。 

然而，随着应用程序的增长，管理和维护处理所有事情的单个文件可能会变得困难。

解决此问题并保持代码库更干净、更有条理的一种推荐技术是将主文件分为两部分：一个用于路由，另一个用于服务器设置或配置。 

这是一个例子：

```
// app.js
const express = require('express');
const app = express();
/* Middlewares */
app.get('/', (req, res) => {
  res.status(200).json({ message: "it works" });
})
app.use(/* Global Error Handler */);
module.exports = app;
// server.js
const app = require('./app');
const port = process.env.PORT || 5001;
app.listen(port, () => console.log('Server running at', port));

```

**5. 将路由与控制器分开**

为了实现更有组织性和模块化的代码库，我建议将路由与控制器分开。这种做法有助于保持清晰的关注点分离，并提高代码的可读性和可维护性。 

这是一个演示路由和控制器分离的示例。

```
// ❌ Avoid this
const route = express.Router();
route.get('/login', tryCatchFn(req, res, next) => {
  // logic here
}))
// ✅ Do this
const route = express.Router();
const {login} = require("../controllers/auth");
route.get('/login', login);

```

**结论**

在本文中，我们讨论了编写干净且易于维护的 NodeJS 代码的不同高级技术。有许多最佳实践可以显着提高应用程序代码的质量。 

最后，希望这篇内容对你有用，感谢你的阅读。