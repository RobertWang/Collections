> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jackfranklin.co.uk](https://www.jackfranklin.co.uk/blog/check-in-your-node-dependencies/)

> On every team at every company I've been at prior to my current role, the advice was simple: don't ch......

December 6, 2021

On every team at every company I've been at prior to my current role, the advice was simple: don't check your `node_modules` folder into your version control system (which I'll refer to as “Git” for the rest of this article…). This seemed like solid advice for multiple reasons:

*   The code within `node_modules` isn't authored by the team directly.
*   The code within `node_modules` is often quite large and would cause a lot of noise in git diffs and pull requests.
*   The code within `node_modules` can easily be replicated with an `npm` install.

I currently work at Google on the Chrome DevTools team and we check our `node_modules` folder into source control. At first this struck me as unusual, but I've come to believe that there are some major benefits to this approach that I think more people should consider.

No need for npm installs
------------------------

Once you check your `node_modules` in, there's no need to run an install step before you can get up and running on the codebase. This isn't just useful for developers locally, but a big boost for any bots you might have running on a Continuous Integration platform (e.g. CircleCI, GitHub Actions, and so on). That's now a step that the bots can miss out entirely. I've seen projects easily need at least 1-2 minutes to run a complete `npm install` from scratch - and on bots that could be even longer. If you think that a bot runs on every pull request and deploy, you could easily have 50+ bots run each day. That's a lot of minutes (and bandwidth!) saved.

Guaranteed replicated builds
----------------------------

Having your `node_modules` checked in guarantees that two developers running the code are running the exact same code with the exact same set of dependencies. Yes, this can be managed by a package-lock.json file, or other tools, but I've seen all of them slip up rarely or allow a slight variation in a minor version number that causes issues. Once the dependencies are in git, you cannot possibly run with anything other than those and each developer will be running the exact codebase.

Better awareness of the code you're shipping
--------------------------------------------

I've been surprised at how more aware I am of adding dependencies when the git diff shows me the entirety of the code that is being added to the project. This has lead us to make contributions to tools to help reduce their file size on disk and have a better awareness of the impact a dependency will have on our bundle size.

More consideration to adding a dependency because it's not invisible
--------------------------------------------------------------------

I mentioned earlier that people see the noise in a git diff as a downside to adding dependencies to version control, and I do acknowledge that it can be a downside to this approach, but I've found that noise to often be a useful signal. Adding that one extra dependency because I don't want to write a few lines of code myself is something I used to do frequently - but now I'm much more considered because I can see the code that's being added and can reflect on if it's worth it.

_Note: this doesn't mean that we don't have dependencies!_ There are times where it is worth it to add a dependency - but seeing the code in version control has made me more considered about doing it - the cost is no longer invisible.

You can manage the large diffs
------------------------------

There is no shying away from the fact that if a developer works on a change that adds a new dependency, there could be a lot of noise in the diff. One of our dependencies that we check in is TypeScript, and every time we update that, the git diff is huge and frankly not worth looking at (beyond the CHANGELOG). We've come up with a rule that helps us here: a change that updates `node_modules` may not touch any other code in the codebase. So if I update `node_modules/typescript` with its latest version, I will be warned by our tooling if any other folder outside of `node_modules` is changed.

This rule serves us well the majority of the time, because any work that relies on a new or updated dependency can be split into two changes:

1.  Update the dependency
2.  Use the dependency in the code

There are times where this doesn't work; updating TypeScript may require us to update some code to fix errors that the new version of TypeScript is now detecting. In that case we have the ability to override the rule.

> As with anything in software engineering, most "rules" are guidelines, and we're able to side-step them when required.

Protection from another left pad
--------------------------------

The [now infamous left_pad incident](https://qz.com/646467/how-one-programmer-broke-the-internet-by-deleting-a-tiny-piece-of-code/), where a popular npm package was removed from the repository all of a sudden, causing builds everywhere to break, would not have impacted a team who checked all their dependencies into git. They would still have to deal with the long term impact of "what do we do with this now unsupported dependency", but in the short term their builds wouldn't break and they wouldn't be blocked on shipping new features.

Conclusion
----------

If I was starting a new codebase this week, or joining a small start-up just getting their first version off the ground, I would advocate strongly for checking `node_modules` into version control. It absolutely takes some getting used to, but in my experience over the last two years of working this way the benefits I've listed above strongly outweigh the additional git noise and slight overhead.