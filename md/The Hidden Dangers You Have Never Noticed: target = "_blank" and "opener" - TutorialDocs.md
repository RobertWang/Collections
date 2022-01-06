> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.tutorialdocs.com](https://www.tutorialdocs.com/article/html-opener-blank.html)

> Usually, when using a link in a web page, you'll add the attribute target ="_blank" on the a tag if ......

Usually, when using a link in a web page, you'll add the attribute `target = "_blank"` on the `a` tag if you want the browser to open the specified URL in a new tab automatically.

However, It is this attribute that becomes the chance which can be sought by phishing attackers.

![](https://www.tutorialdocs.com/upload/2018/11/blank0.jpg)

Basics
------

### `parent` and `opener`

Before talking about the `opener`, let's take a look at the `parent` in the `<iframe>` first.

We know that an object used for parent-child page interaction is provided in `<iframe>`, and it's called `window.parent`. We can get access to the `window` of the parent page from the page in the frame through the `window.parent` object.

The `opener` is the same as the `parent`, but it's just used for the `<a target="_blank">` to open a page in a new tab. You can use `window.opener` directly to get access to the `window` object of the source page through the page opened by `<a target="_blank">`.

### Same-domain and cross-domain

The browser provides full cross-domain protection. When the domain name is the same, actually, the `parent` object and `opener` object are the `window` objects of the parent. When the domain name is different, the `parent` and `opener` are the wrapped `global` objects. This `global` object only provides very limited access to properties, and most of the only a few properties are not allowed to be accessed to (it will throw a `DOMException` directly if you access).

![](https://www.tutorialdocs.com/upload/2018/11/blank1.png)

![](https://www.tutorialdocs.com/upload/2018/11/blank2.png)

In `<iframe>`, a `sandbox` property is provided to control the permissions of the pages in the frame, so even in the same domain, you can also control the security of `<iframe>`.

Be Exploited
------------

If you have a link which uses `target="_blank"` on your site, once the user clicks on the link and gets into a new tab, your site will be navigated to a fake website directly if there exists malicious code on the page in the new tab. Thus at this point if the user goes back to your tab-page, what they will see is the page which has been replaced.

### Steps

1. There is a link on your website `https://example.com`:

```
<a href="https://an.evil.site" target="_blank">Enter an "evil" website</a>
```

2. The user clicks on the link and opens the site on a new tab. The website can determine the source of the user through the `Referer` property in the HTTP Header.

And the site contains JavaScript code similar to this:

```
const url = encodeURIComponent('{{header.referer}}');
window.opener.location.replace('https://a.fake.site/?' + url);
```

3. Now, the user continues to browse the new tab, while the tab of the original website has been navigated to `https://a.fake.site/?https%3A%2F%2Fexample.com%2F`.

4. According to the Query String, the malicious website `https://a.fake.site` forges a page which is used to trick the user, and displays it (you can also make another jump during the period, making the browser's address bar more confusing).

5. The user closes the tab of `https://an.evil.site` and goes back to the original website..................Oh ,no, you can't go back now.

> The above attacking steps are in the scene of crossing-domain. Because when crossing-domain, the `opener` object is the same as the `parent`, both are limited and provide only very limited access properties. And among the only few properties, most of them are not allowed to be accessed (it will throw a `DOMException` directly if you access).
> 
> But unlike `parent`, in the scene of crossing-domain, the `opener` still can call the `location.replace` method while the `parent` can't.
> 
> If it is in the same domain (for example, a page on a website has been embedded with malicious code), the situation will be much more serious.

Defense
-------

The `` `<iframe>` `` has the `sandbox` property, so you use the following ways for a link:

### 1. Referrer Policy and noreferrer

In the above attacking steps, the `Referer` property which is in the HTTP Header has been used. In fact, you can add the `Referrer Policy` header to the response header of the HTTP to ensure the privacy of the source.

You need to modify the backend code to implement the `Referrer Policy`. And on the front end, you can also use the `rel` property of the `<a>` tag to specify `rel="noreferrer"` in order to ensure source privacy.

```
<a href="https://an.evil.site" target="_blank" rel="noreferrer">Enter an "evil" website</a>
```

> However, it should be noted that even if you have limited the `referer`'s delivery, the original page still cannot be prevented from being maliciously redirected.

### 2. noopener

For security reasons, modern browsers support specifying `rel="noopener` in the `rel` property of the `<a>` tag, so that in an opened new tab, the `opener` object will no longer be available, as it has been set to `null`.

```
<a href="https://an.evil.site" target="_blank" rel="noopener">Enter an "evil" website</a>
```

### 3. JavaScript

The `noopener` property seems to solve all the problems, but...you still should take browser compatibility issues into consideration.

![](https://www.tutorialdocs.com/upload/2018/11/blank3.png)

As you can see, most browsers have been compatible with the `rel="noopener"` property now. However, in order to protect those slightly older modern browsers and ancient browsers, only the `noopener` property is not enough.

So you have to turn to the following JavaScript code for help.

```
"use strict";
function openUrl(url) {
  var newTab = window.open();
  newTab.opener = null;
  newTab.location = url;
}
```

### Best Advice

First, you should add `rel="noopener"` onto the link in the website (you are recommended to add `rel="noreferrer"` too), if `target="_blank"` is used. Similar to this:

```
<a href="https://an.evil.site" target="_blank" rel="noopener noreferrer">Enter an "evil" website</a>
```

Of course, when jumping to a third-party website, it is recommended to add `rel="nofollow"` for SEO weights, so it ends up like this:

```
<a href="https://an.evil.site" target="_blank" rel="noopener noreferrer nofollow">Enter an "evil" website</a>
```

Performance
-----------

Finally, let's talk about performance issues.

If the site uses `<a target="_blank">`, the performance of the newly opened tab will affect the current page. At this point, if a very bloated JavaScript script is executed in the newly opened page, the original tab will be affected, and the phenomenon of stagnation will occur (of course, it will not be stuck).

If the `noopener` is added to the link, then the two tabs won't interfere with each other, so that the performance of the original page won't be affected by the new page.