> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [dev.to](https://dev.to/khangnd/build-an-android-app-with-web-techs-gji)

> In this article, I'm going to share with you how I built an Android app with web techs using WebView.......

In this article, I'm going to share with you how I built an **Android app with web techs using [WebView](https://developer.android.com/reference/android/webkit/WebView)**.

Yes, there are other alternative approaches, like [PWA](https://web.dev/progressive-web-apps/) or [React Native](https://reactnative.dev/) where web techs or the likes are involved to possibly achieve the same outcome, but due to the fact that a solution or approach may have the upperhand over the other depending on the context, it's always good to know more than just one solution. In this case, WebView provides us more interaction with our Android through the [Javascript Interface](https://developer.android.com/reference/android/webkit/JavascriptInterface) and the learning path for this solution might be flatter if you already have some basic knowledge in Java/Kotlin and proficient in web development. Now let's get started:

1.1. The Android app
--------------------

Download and install [Android Studio](https://developer.android.com/studio).

Start Android Studio and create a new project with an empty activity:  
[![](https://res.cloudinary.com/practicaldev/image/fetch/s--W3QNnJFy--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ns94v0kq2yeukvmhoocj.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--W3QNnJFy--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ns94v0kq2yeukvmhoocj.png)

Select _Java_ as the development language (or Kotlin if you are familiar with it)

Once done, the main structure of our app is as follows:  
[![](https://res.cloudinary.com/practicaldev/image/fetch/s--W2JHQtf8--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1eamu00q7v8ygwposvus.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--W2JHQtf8--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1eamu00q7v8ygwposvus.png)

A lot of things, but let's just put our focus on the **`MainActivity`** class, where we write our code and the **`activity_main.xml`**, where we construct the app's layout.

In **`activity_main.xml`**, remove everything under the root and add:  

```
<WebView
    android:id="@+id/webapp"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

This creates a `WebView` container which will display a web page in our app. The width and height are set to `match_parent` which fills up the whole screen.

In the **`MainActivity`** class, add the following lines to `onCreate`:  

```
WebView webView = (WebView) findViewById(R.id.webapp);
webView.getSettings().setJavaScriptEnabled(true);
```

This searches for the WebView with the id `webapp` from our layout that we defined and enables its JavaScript.

Now create an `assets` folder under the root `app` as shown in the screenshots below:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--rY4sMre---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dbt0zsfmqp6b05aa42pk.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--rY4sMre---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dbt0zsfmqp6b05aa42pk.png)

The `assets` folder will contain the web resources we use in our app.

> If the web resources are composed of plain HTML, CSS and JS, you can add them directly here, if they have a more complex composition such as some frameworks and bundlers, you might need to put the **built version** here instead, so the app will be of an optimal size.

1.2. The web page
-----------------

Now that we had our Android app ready to display a web page, we need a web page. I keep this guide simple and easy by just adding an HTML file directly in the `assets` folder with the following content:  

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta >
</head>
<body>
    <h1>Good day!</h1>
    <p>You are viewing this web page from Android!</p>
</body>
</html>
```

Then add the following line to your **`MainActivity`** class to load this page locally:  

```
webView.loadUrl("file:///android_asset/index.html");
```

If you are going to serve the web page on a host, you may then replace the URL to point to the target host:  

```
webView.loadUrl("https://mydomain.com");
```

Now try running your app on a simulator or connected device, you should be able to see the result:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--167Ci5ep--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/802h6fwe701okeg4seac.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--167Ci5ep--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/802h6fwe701okeg4seac.png)

And that's it, we have just finished creating our Android app which is mainly constructed with web contents.

I did mention that using this WebView approach, our web page has the possibility to interact with our Android through the JavaScript Interface, correct? Let's see how it works:

Under the same package as the `MainActivity`, create a new class called `WebAppInterface` with the following contents:  

```
package com.example.androidwebapp;

import android.os.Build;
import android.webkit.JavascriptInterface;

public class WebAppInterface {
  @JavascriptInterface
  public String getAndroidVersion() {
    return Build.VERSION.RELEASE;
  }
}
```

Our function to get the Android version is marked with the `@JavaScriptInterface` annotation and will be available to our web page in a few more steps.

In our `MainActivity` class, add:  

```
webView.addJavascriptInterface(new WebAppInterface(), "Android");
```

This exposes any function we defined in the `WebAppInterface` class to the web page's JavaScript under the"Android" alias.

And finally, in our HTML page, add the following script:  

```
<script>
document.body.append("Your device is using Android " + Android.getAndroidVersion());
</script>
```

Try running again and you should be able to see the Android version of your simulator or device.

> A thing to note about this technique is that, due to the differences between Java/Kotlin and JavaScript, only **primitive** data types may be shared through the interface. For instance, exposing the function that returns a Java array as follows will resolve to `undefined` in JavaScript:  

```
@JavascriptInterface
public int[] getJavaArray() {
  return new int[] { 0, 1, 2 };
}
```

A complete repository you can find at: [https://github.com/khang-nd/android-web](https://github.com/khang-nd/android-web)

And this concludes my sharing article, thank you for reading, see you in the next one.