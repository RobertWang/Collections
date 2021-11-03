> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [shkspr.mobi](https://shkspr.mobi/blog/2020/10/please-stop-using-cdns-for-external-javascript-libraries/)

> I want to discuss a (minor) antipattern that I think is (slightly) harmful. Lots of websites use larg......

I want to discuss a (minor) antipattern that I think is (slightly) harmful.

Lots of websites use large Javascript libraries. They often include them by using a 3rd party Content Delivery Network like so:

```
<script src="https://cdn.example.com/js/library-v1.2.3.js"></script>
```

There are, supposedly, a couple of advantages to doing things this way.

1.  Users may already have the JS library in their cache from visiting another site.
2.  Faster download speeds of large libraries from CDNs.
3.  Latest software versions automatically when the CDN updates.

I think these advantages are overstated and lead to some significant disadvantages.

Cacheing
--------

I get the superficial appeal of this. But there are dozens of popular Javascript CDNs available. What are the chances that your user has visited a site which uses the _exact_ same CDN as your site?

How much of an advantage does that really give you?

Speed
-----

You probably shouldn't be using multi-megabyte libraries. Have some respect for your users' download limits. But if you are truly worried about speed, surely your whole site should be behind a CDN - not just a few JS libraries?

Versioning
----------

There are some CDN's which let you include the latest version of a library. But then you have to deal with breaking changes with little warning.

So most people only include a specific version of the JS they want. And, of course, if you're using v1.2 and another site is using v1.2.1 the browser can't take advantage of cacheing.

Reliability
-----------

Is your CDN reliable? You hope so! But if a user's network blocks a CDN or interrupts the download, you're now serving your site without Javascript. That isn't necessarily a bad thing - you do progressive enhancement, right? But it isn't ideal.

If you serve your JS from the same source as your main site, there is less chance of a user getting a broken experience.

Privacy
-------

What's your CDN's privacy policy? Do you need to tell your user that their browsing data are being sent to a shadowy corporation in a different legal jurisdiction?

What is your CDN doing with all that data?

Security
--------

British Airways' payments page was [hacked by compromised 3rd party Javascript](https://www.riskiq.com/blog/labs/magecart-british-airways-breach/). A malicious user changed the code on site which wasn't in BA's control - then BA served it up to its customers.

What happens if someone hacks your CDN?

You gain extra security by using [SubResource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). That lets you write code like:

```
<script src="https://cdn.example.com/js/library-v1.2.3.js"
   integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
   crossorigin="anonymous"></script>
```

If even a single byte of that JS file is changed, the hash won't match and the browser should refuse to run the code.

Of course, that means that you could end up with a broken experience on your site. So just serve the JS from your own site.

So what?
--------

This isn't the biggest issue on the web. And I'm certainly guilty of misusing CDNs like this.

Back when there were only a few CDNs, and their libraries didn't change rapidly, there was an advantage to using them.

Nowadays, in an era of rampant privacy and security violations, I think using 3rd party sources for Javascript should be treated as an anti-pattern.