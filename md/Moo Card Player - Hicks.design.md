> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [hicks.design](https://hicks.design/journal/moo-card-player)

> A journal post about my @MOO card music player setup, using square business cards and NFC tags to pla......

27 Sep 2021

![](https://hicks.design/media/pages/journal/moo-card-player/7da78ee01c-1632738477/IMG_0962.jpg)

I've been looking for ways to restore some kind of physicality to my digital music collection, and this has led me to create a setup using a [square Moo](https://www.moo.com/uk/business-cards/square) 'album' card with NFC tag sticker, to trigger an iOS shortcut and play an album to an airplay speaker with the least amount of friction.

There's nothing original here, people have been doing this for years, most notably [Brendan Dawes with his Plastic Players 1 & 2](https://brendandawes.com/projects/plasticplayer2). I'd considered using a Raspberry Pi + NFC reader add-on, but as iPhone's since the XS/XR can read and write NFC tags in the background, I went that route. Using a device that's always with me (rather than maintaining a screen-less Pi) is the most frictionless way for me to do this.

![](https://hicks.design/media/pages/journal/moo-card-player/d34e61a790-1632738478/front.jpg)It's a square Moo card on the front… ![](https://hicks.design/media/pages/journal/moo-card-player/b7d0dab511-1632738477/Back.jpg)…and an NFC sticker on the back

[Leigh videoed me demonstrating it to her](https://twitter.com/hicksleigh/status/1430823574013726724), but sadly the airplay connection didn't work then (more on that bit later). What follows is an explanation of how I've done this using iOS, the different approaches you can use, and the pros and cons of each.

If you want to have a go, you'll need:

*   An iPhone XS/XR or above. These have the ability to read NFC in the background, which is key. This means as long as the iPhone screen is on (it doesn't have to be unlocked) the NFC tag will be read.
*   NFC Tags. These are cheap and come in many forms - I picked up round stickers from [ZipNFC](https://zipnfc.com/) for around 20p each.
*   Some form of physical album artwork to stick the tag to - I opted for square Moo cards (more on that below).

You can then either write the information to the NFC tag, or use it to trigger an iOS Personal Automation. The former works for everyone, but you have to OK a dialog prompt. The latter can run immediately, but only works on that particular device. I went with Personal Automations, but here's a run down of both methods:

Writing to the Tag
------------------

I used the [NFC Tools](https://apps.apple.com/us/app/nfc-tools/id1252962749) app to write the data to the tag, which worked fine, and I haven't try any others.

The terminology might seem alien at first, but you need to **Add a Record**. You can write all sorts of data as a record such as contact details, initiate a phone call, pass on Wi-fi details and more. The ones I used were **Open URL** and **Run Shortcut**. They can be written onto the NFC tag and launched when placed close to the phone's NFC reader (for iPhone's it's top centre).

**Spotify**

1.  Find the album and select **…** on the first track → **Share** → **Copy Link**. If you do this at the album level, the link will just take you to the album, but choosing the first track will play that, and then carry on to finish the album. The link will be a standard web URL ([https://open/spotify.com/](https://open/spotify.com/)…) which is the preferred format for iOS, but on Android the short code version is preferred.
2.  Then in NFC tools, **Write** → **Add a record** → **URL** (paste the URL you copied).
3.  Select **OK** and then **Write**
4.  Bring the tag up to the top centre of the phone, and you'll get a satisfying PING to confirm that the data is written.

**Apple Music**  
You can use a share URL from Apple Music to open an album, but it won't play automatically. Here you'll need to combine NFC data with a shortcut input. I haven't found a totally satisfactory way to do this, but this method works in most cases:

Create a new Shortcut (I'd recommend naming it something short and simple like'Play') then add the following actions to it:

![](https://hicks.design/media/pages/journal/moo-card-player/54e93c4316-1632738478/nfc.png)

(Recent changes in iOS 15 mean the 'Receive Any input' action gets added when you choose the 'Find Music' action)

Then in NFC Tools, instead of writing a URL record to the tag, we need a _Shortcut_ record. Add the exact name of the shortcut (which is why I suggested short and sweet!), and put the album title in the input field. This single shortcut can then be triggered by many different NFC tags, passing the album title to the action to find and play that album. It works fine when the album title is unique, but it'll presumably break when it isn't. Ideally, we'd be able to use a unique ID as a way of finding a specific album, but that doesn't seem to be possible at the moment.

**Plex**  
Plex on iOS apparently has a [way of deep linking](https://forums.plex.tv/t/deep-links/205583), which _should_ mean it's possible, but I gave up trying to get this to work. If anyone has had success with this, I'd love to hear about it!

**Pros:** Anyone with a NFC reader enabled device can pick up a card and play, and it can work cross-platform too.

**Cons:** iOS prompts you to OK the action. It's obviously a sensible security feature that prevents anyone from running actions without you knowing, but it does mean it's not quite as slick as I'd like it to be. Also, some URLs might be too long to fit on the tag, depending on what capacity you buy.

### Personal Automation

Rather than write data to the tag, this method uses the tag to trigger a personal automation on your phone. This means that you have to repeat the following process for each album:

1.  In the Shortcuts app, choose 'Automation'.
2.  Select the add button and then 'Create Personal Automation'.
3.  Select 'NFC' from the list of triggers.
4.  Tap 'Scan' and bring your tag up to the top of the phone until it registers.
5.  Give it a name. (I do a one-word version of the album title) and tap 'Next'.
6.  Add 'Open URLs' action if you're using Spotify, or'Play Music'action if it's Apple Music. Then you can add either the URL from Spotify, or select 'Music' and browse your Apple Music library. When you're viewing the album you want, select the plus button top right.
7.  Important final step: switch off 'Run without asking' and confirm. This is the key part which lets you avoid being asked whether the run the shortcut!

![](https://hicks.design/media/pages/journal/moo-card-player/88be40ec0e-1632738478/automation.png)

**Pros:** This method means you can avoid the prompt, and the music plays automatically. Frictionless! It doesn't matter how much data the tag can store, as no data is ever written to it.

**Cons:** Personal automations are _specific_ to a device. They can't be shared with others or synced to your other devices (although they are carried across when migrating to a new device). Not everyone can pick up a card and just play it. I originally thought this was a huge issue, until I realised that no one else in the family cares! Lol :)

Cards
-----

Getting the feel of the card right was a crucial part. I could've just stuck printouts from an inkjet printer onto card, or wrote the album name onto an NFC card, but they wouldn't be particularly enticing to pick up.

My solution was to order a batch of ['Super' square business cards from Moo on 400gsm card](https://www.moo.com/uk/business-cards/square). There is a minimum order of 50, and they allow up to 50 different designs, so this was an ideal way to get a set of nicely printed album cards. At 65x65mm they feel the right size for using with an iPhone. They even come in a nice card box that's ideal for browsing!

The only obstacle is getting decent high-res versions of the album artwork. Purchases from [Bandcamp](https://bandcamp.com/) come with a high res file automatically, but others required a bit of hunting to find something near decent (Tori Amos''Under the Pink' was the hardest to find). In general, 1500x1500px images looked great in print at this size.

In all, this solution cost me around 50p each (22p for the tag, and 27p for the Moo card). For more speed and flexibility, I'll probably need to find a way to do these homemade eventually. Otherwise I have to wait until I have another 50 covers that I want to print.

Speakers
--------

The final part of the setup was the audio output: I wanted the flexibility of playing to one of _three_ possible airplay speakers, and while it's possible to build a chooser dialog into the iOS shortcut, I didn't want that friction. Instead, I have shortcuts saved to my Home Screen for connecting to each speaker:

![](https://hicks.design/media/pages/journal/moo-card-player/6d2396d205-1632738478/shortcuts.png)

This ensures that playback happens where I want it, and at a decent starting volume (no sudden BOOM or too quiet to hear). Worth noting: at first, I made the mistake of using the _'Handoff playback to device'_ shortcut action, instead of _'Set playback destination'_, which got messy quickly with different audio playing on my phone and the speaker.

Then when I want to play music, I choose the speaker shortcut first, and then I don't have to think about it again for the rest of the session. Job done!

While the initial setup of these 'Mini-Albums' takes a few steps, thereafter it's a joyfully fast interface to play albums!

Hicks Design  
Wenrisc House  
Meadow Court  
Witney, OX28 6ER  
United Kingdom