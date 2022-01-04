> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [umaar.com](https://umaar.com/dev-tips/242-considerate-javascript/)

> JavaScript - Conditional JavaScript, only download when it is appropriate to do so

![](https://umaar.com/assets/images/dev-tips/considerate-javascript.gif)

Intro
-----

In this post, I'd like to share a few techniques which you can use for **selectively downloading**/executing resources such as JavaScript.

As an example, if the users device does not have a lot a RAM, you could decide to skip the downloading (and in turn, the parse + **execution costs**) of a particular JavaScript resource.

Note: I won't be specifying the exact times you should minimise usage of JavaScript. It ultimately depends on your use case.

*   If I were building a game in WebGL, I'd probably load files regardless of `navigator.deviceMemory`.
*   If I were building Wikipedia, I might indeed skip loading of some JavaScript based on `navigator.deviceMemory` + `navigator.getBattery()` + `navigator.connection.downlink`.

Load JavaScript if the device has enough RAM
--------------------------------------------

Let's start out with this:

```
if (navigator.deviceMemory > 1) {
    await import('./costly-module.js');
}
```

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/device-memory-js-142fa065f6.png)

Browser support is limited to Chromium based browsers, which according to [caniuse](https://caniuse.com/mdn-api_navigator_devicememory) is 70% of the global usage stats. If `navigator.deviceMemory` is `undefined`, maybe consider loading the JavaScript regardless.

### Detecting device memory on the server

The Web Platform is full of hidden features. Turns out you can get this information on your server!

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/device-memory-header-47c8894672.png)

The browser can send a **network request header** which means your server can decide on the most appropriate resource to send back down. Use this `<meta>` tag in your HTML to activate this feature:

```
<meta http-equiv="Accept-CH" content="Device-Memory">
```

Load JavaScript when the device has enough CPU
----------------------------------------------

You can get the number of logical processors on the users device with `hardwareConcurrency`.

```
if (navigator.hardwareConcurrency > 4) {
    await import('./costly-module.js');
}
```

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorConcurrentHardware/hardwareConcurrency) has an explanation:

> The number of logical processor cores can be used to measure the number of threads which can effectively be run at once without them having to context switch.

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/logical-processors-e3d4ca9d75.png)

This feature can be especially useful when creating new **Workers** - part of the Web Workers API.

Browser support is [very good](https://caniuse.com/hardwareconcurrency).

Load JavaScript when the user has enough battery left
-----------------------------------------------------

We've all been there. Low battery 🔋️, no charger, and you start clearing apps in the hope that your 4% lasts on the journey home.

A page _with JavaScript_ will utilise more **battery** than a page _without JavaScript_ - and yes, the same applies to CSS and images.

The Network Download, Parse and Execution costs will defintely use battery, but don't forget the variety of tasks which JavaScript is capable of e.g. creating long-running timers.

```
const {level, charging} = await navigator.getBattery();



if (charging || level > 0.2) {
        await import('./costly-module.js');
}
```

You also get charging times and discharging times which in some scenarios could be even more valuable than what the current level is.

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/battery-js-f2083c9b54.png)

Browser [support](https://caniuse.com/battery-status) seems to be limited to Chromium based browsers. Interestingly it was removed in Firefox due to privacy concerns (a.k.a. fingerprinting 🖐🏼️).

Load JavaScript when the device has enough storage
--------------------------------------------------

This one really depends on what the JavaScript is doing. For example if the JavaScript is:

*   Downloading lots of maps as part of a game.
*   Downloading + Saving website resources via a Service Worker (e.g. for offline capabilities).
*   Downloading large textures/assets part of a visualisation.

Then maybe consider the idea of using storage availability to influence downloading of resources.

```
const {quota} = await navigator.storage.estimate();
const fiftyMegabytesInBytes = 50 * 1e+6;

if (quota > fiftyMegabytesInBytes) {
    await import('./costly-module.js');
}
```

Browser [support](https://caniuse.com/mdn-api_storageestimate) is decent

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/storage-quota-bed662b8eb.png)

Load JavaScript when the device has a good network connection
-------------------------------------------------------------

**Tip:** Network Connection types can be simulated in DevTools. Learn DevTools in my [ModernDevTools.com course](https://moderndevtools.com/).

At this point, you can argue it's also effective to skip the download of _any_ optional resource - not just JavaScript.

```
if (navigator.connection.effectiveType === '4g') {
    await import('./costly-module.js');
}
```

The `navigator.connection` object gives some information on the network. E.g. the `downlink` property gives you the bandwidth in megabits per second. The round-trip time is in milliseconds

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/network-info-c451ffd2ba.png)

How you use this API is highly dependant on your use case. But as an example, if `navigator.connection.downlink` is low, you can download smaller resources instead. In an ideal scenario, this setting would be customisable however (via your webpage UI).

**Note:** Being on WiFi doesn't mean fast, and being on 3G doesn't mean slow. For example, think about tethering!

Browser [support](https://caniuse.com/mdn-api_networkinformation) seems to be limited to Chromium based browsers.

### Getting the network information on the server

You can also get some of that network information on your server, in the form of a network request header.

![](https://umaar.com/assets/images/dev-tips/considerate-javascript/downlink-network-header-7c7d1bddd7.png)

To achieve this, use the following `<meta>` tag in your main HTML file:

```
<meta http-equiv="Accept-CH" content="Downlink">
```

Subsequent requests now have the request header: `downlink: 6` for example. You can also use `content="ECT"` to get the Effective Connection Type as a network request header.

### Save-data

While you can **infer** context based on network information like roundtrip times and connection types, the `saveData` property states a device preference for reduced data usage.

```
if (navigator.connection.saveData === false) {
    await import('./costly-module.js');
}
```

This is available server side (as a client hint), and at some point it should be available as a [CSS Media Feature](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-data).

Conclusion
----------

It's not all about JavaScript, you can also use these various device details to:

*   Reduce heavy/complex animations - which can be a challenge for devices with low memory.
*   Reduce the download of heavily encoded images (e.g. [AVIF](https://caniuse.com/avif)) - which can be a battery drain.
*   Reduce download of network resources of any type - which can struggle on a congested network connection.

More:

*   If you want to stay up to date with content like this, go and subscribe to this [Dev Tips mailing list](https://umaar.com/dev-tips/).
*   I've also got a [video course](https://moderndevtools.com/) all on DevTools.
*   If you're interested in more in-depth writing from me, go and check out [my blog](https://umaar.com/blog/).
*   For quick tips, I'm on Twitter [@umaar](https://twitter.com/umaar).

Thank you for taking the time to read this! Hope you learnt something new!