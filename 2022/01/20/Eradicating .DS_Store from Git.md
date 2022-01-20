> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [0xmachos.com](https://0xmachos.com/2020-01-22-Eradicating-.DS_Store-From-Git/)

> A Simple Solution

Every directory on macOS gets a [`.DS_Store`](https://eclecticlight.co/2021/11/27/explainer-ds_store-files/) file to tell Finder how it should display it. 99% of the time we don’t want to include `.DS_Store` files in our repositories so we [add](https://github.com/0xmachos/0xmachos.github.io/commit/2be455f8fb94adc4f8394fd749f1c4ac9da2cc17) `.DS_Store` [to our projects](https://github.com/0xmachos/OSINT/commit/548e4de7f3fbe5f670d4fbc0ac8a59c8dd08951a) `.gitignore` file.

What if we could globally ignore `.DS_Store` rather than adding it to every repository’s `.gitignore` file?

`core.excludesfile`
-------------------

Git [allows us to specify](https://git-scm.com/docs/gitignore) a global gitignore file via the variable `core.excludesfile`.

Global `.gitignore`
-------------------

Create ([and populate](https://gist.github.com/octocat/9257657)) your global gitignore file. In my [dotfiles repo](https://github.com/0xmachos/dotfiles) I’ve [called mine](https://github.com/0xmachos/dotfiles/blob/master/.gitignore_global) `.gitignore_global` then symlinked it into my home directory as `.gitignore`.

```
ln -s /Users/0xmachos/Documents/Projects/dotfiles/.gitignore_global /Users/0xmachos/.gitignore
```

Tell Git About It
-----------------

Tell Git about it via **one of** the following methods:

Execute the following:

```
git config --global core.excludesfile ~/.gitignore
```

Or add the following to your `.gitconfig`:

```
excludesfile = ~/.gitignore
```

You no longer have to add `.DS_Store` to every project’s `.gitignore` file.

![](https://0xmachos.com/img/dogs/group0.jpg) Photo by Jametlene Reskp on Unsplash