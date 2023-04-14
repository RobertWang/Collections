> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [fullystacked.net](https://fullystacked.net/posts/modern-html-email/)

> Coding email like it's 2023

The first point in MailChimp’s email development [best practices guidelines](https://templates.mailchimp.com/getting-started/html-email-basics/#:~:text=Best%20Practices%3A%20Development,to%20build%20complex%20structures.) is “code all structure using the table element”. The best practices [listed by Cerberus](https://www.cerberusemail.com/best-practices), perhaps the most popular HTML email template, instructs “when in doubt, nest another table”. Things haven’t even progressed as far as float-based layouts. Coding for email is a collection of workarounds, hacks, and obsolete deprecated HTML elements.

It’s long been a Wild West of inconsistency that requires arcane knowledge to cater to. Taking a look at the support table on [Can I Email](https://www.caniemail.com/clients/) (the email equivalent of [Can I Use](https://caniuse.com/)) makes clear the wide divergence between email clients. Writing an email for Apple Mail, Hey, Fastmail, Outlook for Mac, Proton Mail or Mail.ru isn’t so different from coding a modern web page. Yahoo Mail, by contrast, doesn’t even support background gradients. But it’s been Outlook on Windows, far more than any other client, that has trapped email in the past.

Outlook on Windows has very much been [the Internet Explorer](https://css-tricks.com/a-business-case-for-dropping-internet-explorer/) of email clients. The Outlook desktop app on Windows, along with the Windows Mail app, were the only reason developers had to continue building emails with HTML tables. (Outlook apps on macOS, iOS, and Android are unproblematic.)

Because of Outlook, contemporary email code typically looks [something like this](https://emaillove.com/lemon-squeezy-launch-hangover):

```
    <tr>
      <td class="pt-50 mpt-40 px-80 mpx-15" style="padding-top:5 0px;padding-left:80px;padding-right:80px;">
        <table width="100%" border="0" cellspacing="0" cellpadding="0">
          <tr>
            <td class="pb-80 mpb-50" style="padding-bottom:80px;">
              <table width="100%" border="0" cellspacing="0" cellpadding="0">
                <tr>
                  <td class="text-18 lh-28 a-center mfz-16 mlh-26" style="color:#24242c;font-family:Inter, Arial, sans-serif;font-size:20px;line-height:34px;" align="left">
                    <br>
                    <strong>MacBook Pro Winner</strong>
                    <br>
                    <br>First things first. During our pre-launch, we promised one lucky winner a brand-spanking-new MacBook Pro. M1 Chip and all.
                    <br>
                    <br>
                    Well, now that Lemon Squeezy has officially launched it's time to announce the winner.
                    <br>
                    <br>
                    <a href="#" >Drum roll, please →</a>
                    <br>
                    <br>
                    <br>
                  </td>
                </tr>
              </table>
            </td>
          </tr>
        </table>
      </td>
    </tr>
    <!-- etc. -->

```

Tables within tables within tables…

It’s well past time that Outlook got a proper update, and [it’s finally here](https://insider.office.com/en-us/blog/new-outlook-for-windows-available-to-all-office-insiders). The new Outlook switches rendering engines from Microsoft Word to Edge. Support for CSS features in the new Outlook application appears to be identical to that of [outlook.com](https://www.caniemail.com/clients/), which is a great leap forward.

Outlook on Windows didn’t support `background-image`, one of the oldest and [most useful CSS properties](https://www.campaignmonitor.com/css/email-client/outlook-2007-16/). It didn’t support `border-radius`, flexbox, `float`, `opacity`, `outline`, `z-index`, animated GIFs and a whole host of other features. Even support for `width`, `margin`, and `display: none` was partial and buggy. Usage of Outlook has been decreasing over time but remains the third most popular email client. Outlook was truly terrible yet too big to ignore.

![](https://fullystacked.net/assets/emailstats.png.webp) [Email Client Market Share](https://www.litmus.com/email-client-market-share) as of February 2023 calculated from over 1.3 billion opens in Litmus Email Analytics

While some users will continue using older versions of Outlook for [years to come](https://learn.microsoft.com/en-us/lifecycle/products/?terms=Outlook), there are some compelling reasons for developers to move on as quickly as possible.

> In my opinion the majority of [#A11y](https://twitter.com/hashtag/A11y?src=hash&ref_src=twsrc%5Etfw) issues and rendering bugs across all email clients are caused by our need to support Word rendering in Outlook.
> 
> ⁰⁰If we remove that need for developers, it could improve the experience for a huge number of users.
> 
> — Mark Robbins (@M_J_Robbins) [January 5, 2021](https://twitter.com/M_J_Robbins/status/1346387076430835712?ref_src=twsrc%5Etfw)

Gmail has a 102KB size-limit for HTML. After that it will truncate the email with the words “[Message clipped]” and a link to view the rest of the email. Needless to say, it’s not a great user experience and most users probably won’t click the link, so some of your email’s content won’t be seen by many recipients. The markup for an HTML table layout is massively bulky, and moving beyond them will help avoid messages getting clipped in Gmail. Meanwhile, some contemporary email code is such a dense mess of nested tables it can [crash Outlook](https://www.theverge.com/2022/8/2/23288589/microsoft-outlook-app-crash-uber-email-receipts).

While email speed has never been a particular focus within the email community the way it is for front-end web developers, it’s bound to have some impact on click-through rates and the general perception of your brand. Legacy Outlook on Windows is the only client that lacks support for WebP images (even AVIF is supported in most clients), and given the general lack of support for the `<picture>` element, catering to Outlook inhibits performance.

The biggest email headaches now
-------------------------------

Every month, Litmus (a popular email testing tool) release stats showing email client market share.

Apple Mail is by far the most used client. Included on all Apple devices by default, it thankfully has the best support for CSS of any email client. Of the 253 HTML and CSS properties covered by Can I Email, Apple Mail supports 234 (or 92%) of them. Gmail on iOS, by contrast, only supports 92 (36%) of them.

Here’s an abridged version of the [Can I Email scoreboard](https://www.caniemail.com/scoreboard/), showing only the more popular clients:

![](https://fullystacked.net/assets/caniemailstats.png.webp)

The death of table-based layouts doesn’t fully bring email into the modern world. There are still plenty of CSS properties and values that continue to lack full cross-client support, including:

*   linear, radial and conic gradients for `background-image`
*   `box-shadow`
*   `gap`/`column-gap`/`row-gap` for spacing items in grid and flexbox
*   `grid`
*   `align-items` and `justify-content`
*   `object-fit`
*   relative and absolute positioning

The most challenging client to deal with is now Gmail. The Gmail app supports a different level of CSS depending on whether you’re using an `@gmail.com` email address versus a third-party address (`@outlook.com`, `@yahoo.com`, etc). The level of styling support for third party accounts is minimal. These users are a minority, but they’re still worth catering to, even if with only a progressive enhancement mindset.

How should we build emails now?
-------------------------------

That’s the question. If we were to move beyond HTML tables, then we need to see what HTML and CSS is capable of doing in email today.

### First off, use `div`s

There are going to be no issues working with a standard `<div>` element, but that doesn’t mean email supports all other HTML as well. The following HTML5 elements are not supported by all email clients so you’ll need to use a `<div>` instead.

*   `<article>`
*   `<aside>`
*   `<details>`
*   `<figcaption>`
*   `<figure>`
*   `<footer>`
*   `<header>`
*   `<main>`
*   `<mark>`
*   `<nav>`
*   `<section>`
*   `<summary>`
*   `<time>`

Other HTML elements like heading elements (`<h1>`, `<h2>`, etc.), paragraphs (`<p>`), and lists (`<ul>`, `<li>`) are supported everywhere.

### We still (sorta) need inline CSS

The vast majority of email clients support `<style>` tags but there are some edge cases. If you forward an email in the desktop webmail version of Gmail, all `<style>` tags are removed in the forwarded copy. Third-party email accounts used with the Gmail app don’t support the `<style>` tag either.

The `<style>` tag is necessary to define things like [media queries](https://css-tricks.com/a-complete-guide-to-css-media-queries/), [`:hover`](https://css-tricks.com/almanac/selectors/h/hover/) [styles](https://css-tricks.com/almanac/selectors/h/hover/), and [`@font-face`](https://css-tricks.com/snippets/css/using-font-face-in-css/) [declarations](https://css-tricks.com/snippets/css/using-font-face-in-css/) because they can’t be defined inline. For all other styles, it’s a good idea to continue using inline CSS to cater to the widest possible audience.

What about AMP for Email?
-------------------------

[AMP was one of the worst things to happen to the web.](https://css-tricks.com/need-catch-amp-debate/) In 2018, hundreds of big names from the world of web standards and front-end development [signed a letter criticizing Google](http://ampletter.org/) for using their vast power to coerce companies to adopt the technology.

Terence Eden reported from an AMP Advisory Committee meeting:

> We heard, several times, that publishers don’t like AMP. They feel forced to use it because otherwise they don’t get into Google’s news carousel — right at the top of the search results.

While AMP for the web rightly garnered a bad reputation, AMP for email has far more to justify its existence.

### Dynamic email

All email clients block JavaScript in HTML email. Instead of running any arbitrary script, AMP offers a limited alternative. This opens up new possibilities that previously weren’t possible.

Ever RSVP’d to an event without leaving the email? That was made possible by AMP.

> 📢 [@gmail](https://twitter.com/gmail?ref_src=twsrc%5Etfw) update alert! Now you can RSVP to an event, respond to a comment & browse a catalog → directly in your inbox 👨‍💻 [https://t.co/GM779XRvtC](https://t.co/GM779XRvtC) [#ampforemail](https://twitter.com/hashtag/ampforemail?src=hash&ref_src=twsrc%5Etfw) [pic.twitter.com/NQUj2wMfws](https://t.co/NQUj2wMfws)
> 
> — Google Design (@GoogleDesign) [March 27, 2019](https://twitter.com/GoogleDesign/status/1110964743643242496?ref_src=twsrc%5Etfw)

With AMP, users can take action within an email rather than needing to click a link.

### Sending an AMP email

In order to actually send an AMP email, you’re email service provider needs to support AMP. [A fair amount of them do](https://www.caniemail.com/features/amp/) — but certainly not all. You’ll also need to register with Yahoo and Google.

In order to receive an AMP email, the recipient needs to be using either Yahoo, Gmail, AOL, or Mail.ru. You need to send both an AMP version and an HTML version of the email. The HTML version is necessary for people using Apple Mail, Outlook, and any other inbox that doesn’t support AMP.

### AMP enables modern styling

While the interactive abilities of AMP have been presented as its main selling point, perhaps AMP’s best feature is its consistency. AMP for email implements a clearly specified set of CSS features. You can use modern CSS and markup in your AMP email and be relatively sure it’ll render identically in Gmail, Yahoo, and any other client that supports AMP. For anybody that’s had to QA an email, that’s a big deal. There’s a whole industry built around testing HTML emails necessitated by the huge array of inconsistencies across different clients. Foregoing that is one of AMP’s main appeals.

Email clients use a sanitizer to strip out certain code as a security precaution. Allowing any arbitrary CSS for things like absolute positioning and `z-index` could allow a sender to overlay their content on top of the Gmail UI. Most email clients don’t render emails in an iframe, whereas an AMP email is always rendered within an iframe. This changes the security considerations, so Google and other clients can be more permissive in what they strip out of the email (although it’s also the case that the CSS sanitizer in Gmail hasn’t been updated for quiet some time, whereas AMP got a fresh start). That’s why two of the worst email clients for HTML and CSS, Yahoo and Gmail, are able to render modern markup and styles in AMP.

Yahoo and Gmail are the only popular clients that lack support for `flex-wrap`, `align-items` and `justify-content`, for example. Regardless of the interactive features of AMP, we could use AMP as a way to adopt modern CSS. Coding two different kind of emails might seem onerous, but most valid HTML email code is valid AMP code, with a few minor differences (use of `!important` is not allowed, AMP uses `<amp-img>` instead of the `<img>` tag).

[Parcel](https://parcel.io/) — a code editor specifically designed for email — has a [“Generate AMP from your HTML”](https://parcel.io/docs/workspace/emails#generate-amp) button that automates the process. This works well, albeit some manual editing is required. When working on a HTML email, I inline CSS by hand so that I know what I need to override with `!important` inside of a media query (`!important` is required to override an inline style). Parcel removes all `!important` declarations because they aren’t valid in AMP which means none of the code inside the media query works anymore. You’ll need to manually take any inline styles and move them to a `<style>` block in the `<head>`.

### AMP does have some shortcomings

It’s a step in the right direction, but is still far from perfect:

*   AMP emails only survive for thirty days. After that they will fallback to displaying a regular HTML email. Most people will read your email either immediately or never, but it’s still not ideal.
*   If you forward an AMP email, even to another inbox that supports AMP (like Gmail or Yahoo), the recipient will receive only the standard HTML version.
*   AMP does not support inline SVG.
*   AMP emails do not support `@font-face`, so you are restricted to using fonts on the recipient’s system.
*   AMP supports CSS transitions but not CSS animations (`@keyframes`).
*   There’s no support for the `::before` or `::after` pseudos.

AMP for email does seem to have stalled somewhat but you shouldn’t write it off. Google is notorious for killing off its own products, but I’m not worried about AMP for email. The user experience of Google’s own products (Google Calendar, etc) have benefited from AMP — deprecating it would adversely effect their own services.

Conclusion
----------

The world of email moves slowly but it’s heading in the right direction. The [Email Markup Consortium](https://emailmarkup.org/) was established recently to push for more consistency between clients — hopefully they’ll have some success as there’s still a way to go.

Thanks to Geoff Graham for editing this article.