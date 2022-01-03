> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [nystudio107.com](https://nystudio107.com/blog/dock-life-using-docker-for-all-the-things)

> Embracing Docker for All The Things gives you a more flexible, robust, and transportable way to use t......

Andrew Welch · [#devops](https://nystudio107.com/blog/tagged/devops) [#docker](https://nystudio107.com/blog/tagged/docker) [#dev](https://nystudio107.com/blog/tagged/dev)

Mak­ing the web bet­ter one site at a time, with a focus on per­for­mance, usabil­i­ty & SEO

Published 2021.11.02, updated 2021.12.20 · 5 min read · [RSS Feed](https://nystudio107.com/blog/feed)

  
Please consider [🎗 sponsoring me 🎗](https://github.com/sponsors/khalwat) to keep writing articles like this.

Embrac­ing Dock­er for All The Things gives you a more flex­i­ble, robust, and trans­portable way to use tools on your com­put­er with­out messy setup
------------------------------------------------------------------------------------------------------------------------------------------------------

[Dock­er](https://www.docker.com/) is a devops tool that some peo­ple find intim­i­dat­ing because there is much to learn in order to build things with it.

And while that’s true, it’s actu­al­ly quite sim­ple to start using Dock­er for some very prac­ti­cal and use­ful things, by lever­ag­ing what oth­er peo­ple have created.

> While writing an iPhone app might be complex, using one isn’t. Same with Docker!

In this arti­cle, we’re going to talk a bit about the phi­los­o­phy behind using Dock­er for ​“All The Things”, and show a num­ber of prac­ti­cal exam­ples you can start with today.

All you will need to have is [Dock­er Desk­top](https://www.docker.com/products/docker-desktop) installed on your com­put­er to start using it imme­di­ate­ly for some very use­ful things.

So let’s get going!

### [Link](#dock-life) Dock Life

Embrac­ing Dock­er means that we don’t install tools or pack­ages on our actu­al com­put­er using some­thing like [Home­brew](https://brew.sh/).  

We don’t install node. We don’t install composer. We don’t install a whole host of things that you might be used to using every day.

But with a lit­tle mag­ic, you’ll nev­er notice the difference.

You still will just type npm install <package> or composer install <package> and every­thing will just work.

### [Link](#why-bother) Why bother?

If every­thing is going to work the same, why should we both­er using Docker?

There are a num­ber of advan­tages to the ​“con­tainer­ized” approach that Dock­er uses:

*   **Dock­er images are dis­pos­able**. If some­thing goes wrong, you throw them away, with­out them ever affect­ing your actu­al computer
*   **You can run spe­cif­ic ver­sions**. You’re not locked into the ver­sion of the tool you have installed on your com­put­er, you can spin up any ver­sion of the tool you need
*   **Switch­ing to a new com­put­er is easy**. You don’t have to spend hours metic­u­lous­ly recon­fig­ur­ing your shiny new Mac­Book Pro with all the inter­con­nect­ed tools & pack­ages you need
*   **You can exper­i­ment with aplomb**. If you’re curi­ous about a new tool or tech­nol­o­gy, you can just give it a whirl. If it does­n’t work out, just dis­card the image
*   **Dock­er images are self-con­tained**. You’re not going to have to scram­ble to down­load a set of inter­con­nect­ed depen­den­cies in order to get them to work

…and there are many oth­ers, too.

### [Link](#docker-images-containers) Docker images & containers

Instead of installing all of the tools & pack­ages you’re used to using, we use Dock­er images that some­one else has cre­at­ed that con­tain these tools & packages.

Think of a Dock­er image as a bun­dle of all of the files, exe­cuta­bles, etc. need­ed to run some tool we want to use.

There is a cen­tral reg­istry called [Dock­er Hub](https://hub.docker.com/) where orga­ni­za­tions and peo­ple can pub­lish their Dock­er images, and we get to use them!

Images are used as a fac­to­ry to spin up a Dock­er con­tain­er, which is just a run­ning instance based on that image.

These run­ning Dock­er con­tain­ers are what do the actu­al work, and they are where the term ​“con­tainer­iza­tion” comes from.  

The first time you run a Dock­er con­tain­er, it will down­load the image. From then on, it’ll just run the con­tain­er, do the work, then exit.

### [Link](#shell-aliases) Shell Aliases

In order to seam­less­ly pro­vide access to var­i­ous tools run via Dock­er, we’re going to use shell aliases.

A [shell](https://en.wikipedia.org/wiki/Unix_shell) is just the pro­gram that pro­vides the com­mand-line inter­face in your terminal.  

Shell alias­es are user-defined com­mands that expand into some­thing else when you type them into your terminal.

> Think of aliases as fancy macros

If you’re using recent ver­sions of MacOS, you’re prob­a­bly using the [zsh](https://en.wikipedia.org/wiki/Unix_shell) as your shell. Oth­er­wise, you are most like­ly using [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) as your shell.

You can find out what shell you’re using by typ­ing the fol­low­ing into your terminal:

Either way, both shells allow you to cre­ate shell alias­es, which is what we’re after.

For zsh, we’ll be putting our alias­es in the ~/.zshrc file. For bash, we’ll be putting our alias­es in ~/.bashrc

The ~/ pre­fix means ​“in my home direc­to­ry”, and for the curi­ous, rc stands for ​“run commands”.

Both files are lit­er­al­ly just a list of com­mands that are run when the shell is start­ed up, and we can lever­age that to add our alias­es in.

After you change some­thing in one of the rc files (such as to add an alias), you need to re-run the com­mands in the file to exe­cute them:

For zsh: source ~/.zshrc

For bash: source ~/.bashrc

For more on alias­es, check out the [How to Cre­ate Bash Alias­es](https://linuxize.com/post/how-to-create-bash-aliases/) and [How to Con­fig­ure and use Alias­es in Zsh](https://linuxhint.com/configure-use-aliases-zsh/) articles.  

### [Link](#anatomy-of-a-docker-alias) Anatomy of a Docker Alias

While you don’t need to ful­ly grok the Dock­er alias­es we describe here to use them, let’s have a quick look at the com­po­nents in the alias­es we’re using.

Here’s a break­down of an exam­ple alias we might use to run node in a Dock­er container:

Anato­my of a Dock­er Alias

*   alias — tells our shell we’re defin­ing an alias
*   node — the name of our alias (what we type in ter­mi­nal to expand the alias)
*   docker — the Dock­er CLI command
*   run — tells Dock­er to [run the container](https://docs.docker.com/engine/reference/commandline/run/)
*   --rm — auto­mat­i­cal­ly remove the con­tain­er when it exits
*   -it — cre­ate an inter­ac­tive Bash shell in the con­tain­er, to let us poten­tial­ly type fur­ther input
*   -v "${CURDIR}":/app — mount a [vol­ume](https://docs.docker.com/storage/volumes/) from the cur­rent direc­to­ry (out­put by ${CURDIR}) to the /app direc­to­ry in the con­tain­er. This lets the con­tain­er read & write to the cur­rent direc­to­ry on our com­put­er. We put quotes around it to han­dle file paths with spaces in them properly
*   -w /app — set the work­ing direc­to­ry to /app in the container
*   node:16-alpine — use the [node](https://hub.docker.com/_/node) Dock­er image with the [tag](https://docs.docker.com/engine/reference/commandline/tag/) 16-alpine to cre­ate the container

Again, this does­n’t all need to make sense to you imme­di­ate­ly, you can still use the alias­es list­ed below.

But it’s here for you if you decide to mod­i­fy the alias­es, or cre­ate your own.

Now let’s get on to the alias­es we actu­al­ly used to run var­i­ous Dock­er images!

### [Link](#composer-alias) Composer alias

[Com­pos­er](https://getcomposer.org/) is a pack­age man­ag­er for PHP, to run it via Dock­er con­tain­er, just add this alias to the rc file for your shell:

```
alias composer='docker run --rm -it -v "${CURDIR}":/app -v ${COMPOSER_HOME:-$HOME/.composer}:/tmp composer '
```

Then source your rc file (see above) to reload it, and you’ll have composer avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

What it does is it runs the offi­cial [com­pos­er Dock­er con­tain­er](https://hub.docker.com/_/composer), mounts the cur­rent direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then pass­es the composer com­mands into the con­tain­er to run them.

It also cre­ates a sec­ond shared vol­ume for Com­poser’s cache mount­ed to /​tmp on your com­put­er, so it can 

So you can do things like:  

```
composer create-project craftcms/cms craft-test
```

But what if for some rea­son we need to run the old­er composer 1.x? No prob­lem, we can set up an alias for that too:

```
alias composer1='docker run --rm -it -v "${CURDIR}":/app -v ${COMPOSER_HOME:-$HOME/.composer}:/tmp composer:1 '
```

Now if we type composer1 in our ter­mi­nal, it’ll use Com­pos­er v1.x.

In the composer:1 in the alias above, composer is the name of the Dock­er Hub image, and 1 is the tag, which lets you spec­i­fy which ver­sion of the image you want to use.

### [Link](#node-alias) Node alias

[Node.js](https://nodejs.org/en/) is a JavaScript run­time that runs in your ter­mi­nal. To run it via Dock­er con­tain­er, just add this alias to the rc file for your shell:

```
alias node='docker run --rm -it -v "${CURDIR}":/app -w /app node:16-alpine '
```

Then source your rc file (see above) to reload it, and you’ll have node avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

What it does is it runs the offi­cial [node Dock­er con­tain­er](https://hub.docker.com/_/node), mounts the cur­rent direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then pass­es the node com­mands into the con­tain­er to run them.

Then you can do things like:  

This uses Node 16 by default (as indi­cat­ed by the 16-alpine tag), but what if you want­ed to be able to run old­er ver­sions of Node? No prob­lem, add these aliases:

```
alias node14='docker run --rm -it -v "${CURDIR}":/app -w /app node:14-alpine '
alias node12='docker run --rm -it -v "${CURDIR}":/app -w /app node:12-alpine '
alias node10='docker run --rm -it -v "${CURDIR}":/app -w /app node:10-alpine '
```

Then you can instant­ly run com­mands using any ver­sion of node by using node14, node12, and node10 respectively!

### [Link](#npm-alias) npm alias

But what if you use [npm](https://www.npmjs.com/) so much that you want an alias for that as well, so you can just type npm install, and not have to type node npm install? No prob­lem, just add this alias to the rc file for your shell:

```
alias npm='docker run --rm -it -v "${CURDIR}":/app -w /app node:16-alpine npm '
```

Then source your rc file (see above) to reload it, and you’ll have npm avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

Note that if we make the alias npm here, we can’t use node npm any­more, because it will expand both the node and the npm alias­es. Either then always use npm on its own, or rename the alias to npmx or some­thing else.

What it does is it runs the same offi­cial [node Dock­er con­tain­er](https://hub.docker.com/_/node), mounts the cur­rent direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then runs npm and also pass­es your npm com­mands into the con­tain­er to run them.

Then you can do things like:

### [Link](#deno-alias) Deno alias

[Deno](https://deno.land/) is a mod­ern run­time for JavaScript and Type­Script that runs in your ter­mi­nal. To run it via Dock­er con­tain­er, just add this alias to the rc file for your shell:

```
alias deno='docker run --rm -it -v "${CURDIR}":/app -w /app denoland/deno '
```

Then source your rc file (see above) to reload it, and you’ll have deno avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

What it does is it runs the offi­cial [deno Dock­er con­tain­er](https://hub.docker.com/r/denoland/deno), mounts the cur­rent direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then pass­es the deno com­mands into the con­tain­er to run them.

Then you can do things like:

### [Link](#aws-alias) AWS Alias

The [AWS Com­mand Line Inter­face](https://aws.amazon.com/cli/) allows you to man­age your AWS ser­vices from your ter­mi­nal. To run it via Dock­er con­tain­er, just add this alias to the rc file for your shell:

```
alias aws='docker run --rm -it -v ~/.aws:/root/.aws amazon/aws-cli '
```

Then source your rc file (see above) to reload it, and you’ll have aws avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

What it does is it runs the offi­cial [aws-cli Dock­er con­tain­er](https://hub.docker.com/r/amazon/aws-cli), mounts the hid­den ~/.aws direc­to­ry your home direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then pass­es the aws com­mands into the con­tain­er to run them.

So you can do things like:

```
aws ec2 describe-instances
```

### [Link](#ffmpeg-alias) ffmpeg alias

[ffm­peg](https://www.ffmpeg.org/) is a cross-plat­form solu­tion to record, con­vert, and stream audio and video. To run it via Dock­er con­tain­er, just add this alias to the rc file for your shell:

```
alias ffmpeg='docker run --rm -it -v "${CURDIR}":/app -w /app jrottenberg/ffmpeg '
```

Then source your rc file (see above) to reload it, and you’ll have ffmpeg avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

What it does is it runs the offi­cial [ffm­peg Dock­er con­tain­er](https://hub.docker.com/r/jrottenberg/ffmpeg), mounts the cur­rent direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then pass­es the ffmpeg com­mands into the con­tain­er to run them.

So you can do things like:

```
ffmpeg -i input.mp4 output.avi
```

### [Link](#yeoman-alias) yeoman alias

[yeo­man](https://yeoman.io/) is a gener­ic scaf­fold­ing sys­tem allow­ing the cre­ation of any kind of app. To run it via Dock­er con­tain­er, just add this alias to the rc file for your shell:

```
alias yo='docker run --rm -it -v "${CURDIR}":/app nystudio107/node-yeoman:16-alpine '
```

Then source your rc file (see above) to reload it, and you’ll have yo avail­able glob­al­ly in your ter­mi­nal, with­out ever hav­ing installed it.

What it does is it runs a [yeo­man Dock­er con­tain­er](https://hub.docker.com/repository/docker/nystudio107/node-yeoman), mounts the cur­rent direc­to­ry as a shared vol­ume so the Dock­er con­tain­er can write to it, and then pass­es the yo com­mands into the con­tain­er to run them.

So you can do things like:

### [Link](#making-your-own-docker-aliases) Making your own Docker Aliases

Hope­ful­ly, you can see a pat­tern emerg­ing in terms of how to con­struct an alias that runs a Dock­er con­tain­er for you, so you can cre­ate your own, too!

A typ­i­cal pat­tern for me is going to [Dock­er Hub](https://hub.docker.com/), search­ing for the tool I’m inter­est­ed in, and then cre­at­ing a shell alias for it… and away we go!

Usu­al­ly, the Dock­er Images will have exam­ple usage com­mands list­ed along with them that makes it pret­ty pain­less to do.

### [Link](#making-your-own-docker-images) Making your own Docker Images

Mak­ing your own Dock­er images is a bit of a leap from just using images that oth­ers have cre­at­ed, and is beyond the scope of this article.

In order to build Dock­er images your­self, it will require learn­ing a bit more about the ins & outs of Dock­er, and how it works.

I can high­ly rec­om­mend the [Mas­ter­ing Dock­er](https://www.udemy.com/course/docker-mastery/) video series from Bret ​“The Cap­tain” Fish­er if your inter­est is piqued.

You can also look at the [nys­tu­dio107/­dock­er-yeo­man](https://github.com/nystudio107/docker-yeoman) repos­i­to­ry for a sim­ple Dock­er­file exam­ple, with [GitHub actions](https://docs.github.com/en/actions) that build & push the images to [Dock­er Hub](https://hub.docker.com/).

### [Link](#tying-off-at-the-dock) Tying Off at the Dock

There’s a rea­son why Dock­er has such a strong fol­low­ing in the devops and enter­prise worlds, but we can dip our toe gen­tly into the water, and still reap the benefits.

If you want all of the alias­es list­ed above in one place for a sin­gle copy & paste, here they are:  

```
# Docker aliases
alias composer='docker run --rm -it -v "${CURDIR}":/app -v ${COMPOSER_HOME:-$HOME/.composer}:/tmp composer '
alias composer1='docker run --rm -it -v "${CURDIR}":/app -v ${COMPOSER_HOME:-$HOME/.composer}:/tmp composer:1 '
alias node='docker run --rm -it -v "${CURDIR}":/app -w /app node:16-alpine '
alias node14='docker run --rm -it -v "${CURDIR}":/app -w /app node:14-alpine '
alias node12='docker run --rm -it -v "${CURDIR}":/app -w /app node:12-alpine '
alias node10='docker run --rm -it -v "${CURDIR}":/app -w /app node:10-alpine '
alias npm='docker run --rm -it -v "${CURDIR}":/app -w /app node:16-alpine npm '
alias deno='docker run --rm -it -v "${CURDIR}":/app -w /app denoland/deno '
alias aws='docker run --rm -it -v ~/.aws:/root/.aws amazon/aws-cli '
alias ffmpeg='docker run --rm -it -v "${CURDIR}":/app -w /app jrottenberg/ffmpeg '
alias yo='docker run --rm -it -v "${CURDIR}":/app nystudio107/node-yeoman:16-alpine '
```