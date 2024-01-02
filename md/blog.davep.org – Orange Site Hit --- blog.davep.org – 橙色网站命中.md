> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.davep.org](https://blog.davep.org/2024/01/01/oshit.html)

> I know I'm not alone in having a relationship with the orange site that is... weird. I generally disl......

I know I'm not alone in having a relationship with [the orange site](https://news.ycombinator.com/) that is... weird. I generally dislike the culture there, it's almost impossible to read any of the comments without being frustrated about the industry I work in or am adjacent to and some of the people who inhabit it; but as a link aggregator of stuff I might find interesting... I honestly can't think of anywhere better. So, yes, I've been a fairly avid reader of HackerNews for many years, and have even had an account there for over 4 years.

Given this, for a wee while now, I've been meaning to knock up a terminal-based client for it using [Textual](https://textual.textualize.io/).

So after work on [Tinboard](https://blog.davep.org/2023/12/19/tinboard.html) settled down I got the urge to start a new pet project (not abandoning Tinboard, I'm still going to be tweaking and extending it of course) and finally knocking up that client seemed like the one.

_Orange Site Hit_ is the result.

![](https://blog.davep.org/attachments/2024/01/01/oshit-index.png)

It's worth making clear from the very start: this is a read-only reader. There is no logging in, there is no voting, there is no posting of things. This is a client built with [their own API](https://github.com/HackerNews/API) and it doesn't provide such a thing; at least not now and despite me seeing past promises that this will change, there's no API for doing that sort of thing.[1](#fn:1)

The idea of this application is you can run it up in the terminal, check the top, best and latest from the categories provided by the API, perhaps dive off into your web browser if needed, and then got on with other things.

It's there for when you're in the terminal you just _need_ your hit of the orange site.

The main screen of the app revolves around the index of items, most of which are going to be stories. You can see an example of that above. For people who prefer things to be slightly less cramped, I've also added a"relaxed layout" mode too:

![](https://blog.davep.org/attachments/2024/01/01/oshit-index-relaxed.png)

From the index you can head off into your web-browser by hitting Enter on any item; if the item is a story that links to somewhere that link will be opened; if it's something more like _AskHN_, or a job, it'll open the related page on HackerNews itself.

Pressing u with an item selected will let you view the details for the user who posted the item:

![](https://blog.davep.org/attachments/2024/01/01/oshit-user-dialog.png)

If you're the sort of person who wants to torture themselves by reading the comments (oh come on we all do it!), there's a comment reader/navigator too. With an item selected press c and the comment dialog will open:

![](https://blog.davep.org/attachments/2024/01/01/oshit-comments.png)

I _think_ the navigation within that dialog is fine; although I can see some scope for improvement. At the moment it uses a widget-per-comment (actually, it's at least 4 widgets per comment), which is fine and Textual handles that without an issue, even on items with lots of comments, but longer-term I can see me having some fun using [the line API](https://textual.textualize.io/guide/widgets/#line-api) to build a super-efficient comment presentation and navigation widget.

That's it for now; it feels like a good v0.1.0 spot to be in. There are a bunch of things I still want to do with it (better cleaning up of the text, perhaps with some markup support so links get handled, etc; plus lots of ways of searching for stuff), but I felt it was in a place where I could start using it.

Anyway, if this sounds like your sort of thing, it can be installed with `pip` or (ideally) `pipx` [from PyPi](https://pypi.org/project/oshit/). The [source is available over on GitHub](https://github.com/davep/oshit).