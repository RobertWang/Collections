> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [frozenfractal.com](https://frozenfractal.com/blog/2019/7/2/blokjes-the-beginning/)

> Here’s a thing I’ve been working on since January: Blokjes! Sorry, your browser does not support embe......

Here’s a thing I’ve been working on since January: Blokjes!

<video src="" control></video>

In case you can’t tell from the video, the idea is very simple: you get a sequence of blocks (polyominoes) that you have to place on the board. Each of them has to fit entirely on black, or entirely on white squares, and the squares that you place it on will change colour. As the game progresses, the blocks increase in size, so you have to look and plan ahead to make room for the bigger ones.

You can [play the latest version in your browser, right here](https://blokjes.frozenfractal.com/). Check it out, I’ll wait.

Blokjes started as a half-baked idea late one evening when I couldn’t sleep. The idea was to have a grid, where you could tap each tile to toggle it from black to white or vice versa, and you’d have to make shapes or patterns using as few moves as possible.

Not long after, I realized that this would mean lots of tedious tapping, so the idea was born of placing the shape into the grid directly.

### Nonomino in Godot

I took some days to create a prototype of this concept in the Godot engine. I named it Nonomino, after the largest polyomino that you can encounter in the game (9 squares). It looked like this:

![](https://frozenfractal.com/static/2019-07-02-nonomino-godot.png)

As you can see, the basics were there: the 5×5 board, the blocks arranged in a circle around it, and the font. But of course it looked nowhere as polished as today.

Still, it was fun to play, so I decided to upload a web build and send it to my friends for testing. Unfortunately, this was in the days of Godot 3.0, whose “HTML5” output was rather useless: out of six people who tried it, only one could run it, and that was me! Clearly, Godot wasn’t a viable engine for web games yet. (Things got a lot better with the release of Godot 3.1.)

It was clear that this game was a perfect fit for mobile: easy to learn, playable in short rounds, and even somewhat addictive. Yet I wanted it to run on the web as well, because of previous good experiences with HTML5 games publishers (upload game, do nothing, regularly get money). So I decided to rewrite Nonomino in TypeScript, which (unlike the use of some other engine) would keep the web build lean and mean.

### Nonomino in TypeScript

The initial TypeScript rewrite looked a lot like the game we have today:

![](https://frozenfractal.com/static/2019-07-02-nonomino-typescript.png)

This shows the debut of the “Stash” option, which lets you temporarily park one block if you can’t place it immediately. This adds a lot of freedom and strategic depth to the game. It’s equivalent to being able to choose one of the first two blocks off the queue, but the UI is easier to understand.

On a whim, I [submitted](https://alakajam.com/6th-kajam/549/nonomino/) this to the [6th Kajam](https://alakajam.com/6th-kajam/results) game jam, which was run in January and was themed “hyper-casual”. It ended up in 2nd place out of 11 entries, confirming to me that I was onto something here.

Moreover, people fervently competed for top scores and easily beat my own best score. They were beating me at my own game, they were improving day by day, _and_ they were not getting tired of it! Clearly, an element of skill was involved, and also perhaps some addictiveness.

This encouraged me to continue development, with the aim of a commercial release in… oh, how hard can it be… about a month, so maybe early March?

### Further developments

It turned out that “Stash” was a difficult word to understand for non-native speakers. I renamed it to “Hold”, but then people started holding down their finger on it. I would never have predicted that – this is why you do usability testing! So I ended up with “Park”, and the problem disappeared.

Each block was worth the square of its number of tiles. For example, a block of 4 tiles would give you 16 points. I later realized that this was pointless: you only have very limited choice over what blocks you place, so there’s no choice to be made between high-risk, high-reward blocks and low-risk, low-reward ones. More moves made would always result in higher scores and conversely. So I simplified it, by awarding just 1 point per block placed.

There also wasn’t a “Points per move” thing yet. Rather, clearing the board would just give you 100 points. This gave experienced players something to strive for during the relatively easy early game, but wasn’t impactful enough. That’s why I changed it to increase your “Multiplier” instead. And the word “Multiplier” eventually got replaced by the more verbose, but much clearer “Points per move”.

Another change that you might have noticed by now is that the game is no longer called Nonomino, but Blokjes. I had to change the name because people found Nonomino difficult to remember, sometimes spelling it “Monomino” or “Nononimo” or whatever. Moreover, “nonomino” is also the name of a Sudoku variant, which might make my game hard to search for in the app stores (although it might also lead to accidental traffic… it could go either way). I had a hard time coming up with a good alternative name, but my mind was in English mode all the time. When my girlfriend suggested “Blokjes”, Dutch for (little) blocks, it just seemed to fit perfectly. But I had to teach non-Dutch people how to pronounce it, which is why I asked her to do the voice-over!

Behind the scenes, I also overhauled the block generator, which is the piece of code that decides which blocks you’re going to get and when, and thus also determines the difficulty curve of the game. That’s something I could (and probably will) write an entire other blog post about.

There were also some issues with low framerates, caused by the fact that the game doesn’t use WebGL or even the canvas 2D API, but just raw DOM nodes like any old web page does. To optimize it, I had to learn a thing or two about how browsers render the page. [Animating only compositor properties](https://developers.google.com/web/fundamentals/performance/rendering/stick-to-compositor-only-properties-and-manage-layer-count) and promoting animated elements to a separate layer were the key here.

I also [recruited](https://www.reddit.com/r/gameDevClassifieds/comments/bxtc3t/need_some_relaxed_nondistracting_possibly_jazzy/) a professional composer, because I wanted to give this game a unique flavour that cannot be achieved by using free Creative Commons music. This was a first for me, and so far it’s been a great experience. I may write another blog post about that too.

### And now…

I wrapped the game up in Electron so it could be played on the PC (Windows and Linux), and uploaded this build to [itch.io](https://frozenfractal.itch.io/blokjes). The goal here wasn’t to make a lot of money, but to gather some more feedback from a small group of friendly people, before exposing Blokjes to the harsh world of mobile app store commenters.

The build for the web is almost ready for release, and I’m just waiting for the final mix of the music track to be delivered. Then I can start spamming the game to HTML5 web games publishers, who will hopefully put it online on their portals and give me a revenue-share deal.

The mobile port, using Cordova to wrap it into an app for Android and iOS, is still in the works. If all goes as planned, this will be another stream of revenue. But I’m still on the fence about how exactly to monetize the mobile version; that will be the subject of a future blog post.

Here’s to future blog posts!