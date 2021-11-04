> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ryangjchandler.co.uk](https://ryangjchandler.co.uk/posts/running-github-actions-for-certain-commit-messages)

> A quick look at how you can configure your GitHub Actions workflows to only run when a certain phrase......

Published 29 Sep, 2020 in [GitHub Actions](http://ryangjchandler.co.uk/posts?category=github-actions)

I'm going to be honest with you all for a second. I write a lot of `wip` commits. These commits are normally small changes that I want to push up to GitHub so that:

1.  I don't lose things if anything goes wrong and my backup hasn't picked it up.
2.  If I can't describe the change I have just made.
3.  If I'm demonstrating something to somebody on a pull-request.

The problem is, my actions are setup to run on `push`, so every single `wip` commit gets run through the CI process, whether it be running tests, linting or formatting.

After doing some research, I found a way of preventing these from running on every single commit.

Now, whenever I push a `wip` commit or any commit that contains the word `wip`, it will be marked as skipped inside of GitHub actions.

You could also flip the logic and perhaps do something like:

Any commit that contains `[build]` will now trigger these jobs, everything else will be skipped.

You can thank me later! 😉