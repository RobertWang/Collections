> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [k3tan.com](https://k3tan.com/starting-a-new-digital-identity)

> How I would start my digital identity again.

I've often wondered how I would start my digital identity again. This wouldn't be applicable to me but rather for anyone just starting out down the path. Knowing what I know now, I'd definitely do things a little differently.

If I was to start over again, here's what I would do. I think I could do it rather cheaply as well.

Earning money
-------------

Firstly, I would need some cash. Good old fashioned government issued paper notes. I'd do whatever it took to earn cash. That could be from completing chores, baby sitting, washing cars, mowing lawns/gardening, catering, walking/washing dogs, selling goods at the local market, tutoring, whatever. I'd look to save up $1,000AUD in cash but I wouldn't stop there. I'd also keep cash rolling in as best I could.

Acquiring devices
-----------------

I would go to a public library and use a free computer to create an email address. I would not sign up for a gmail account, but instead opt for a free [Protonmail](https://protonmail.com/) account. I would not tie the email address to my identity. Something generic - a pseudonym. The password I pick would be 3 or 4 words put together. I'd try to memorise it and also write it down on a piece of paper for now. We'll look into password managers later.

Next, I'd search for a Google Pixel 3a and a Lenovo Thinkpad laptop on Gumtree. I would look to budget $250-$300 for the phone and $450-500 on the laptop. I'd make sure the devices are in working order and with decent specs. I'd arrange a suitable time and public location to make the trade with the sellers, paying with cash for the items.

Once I have the devices, it's time to start working on them.

The laptop
----------

In all likelihood, the laptop will come with Windows. We're going to remove that. I'd go to the nearest shopping mall and purchase an 8GB USB drive. Shopping malls will usually also have free WiFi, so head over to the food court, open your laptop and download [Pop!OS](https://pop.system76.com/) and [Balena Etcher](https://www.balena.io/etcher/). Flash the USB with Pop!OS using Etcher. Install Pop!OS on to the laptop, overwriting the Windows installation.

The phone
---------

Once you've completed the installation of Pop!OS on your computer, get back on the WiFi and download [CalyxOS](https://calyxos.org/get/) and the [flashing tool](https://calyxos.org/get/install/). Use the flashing tool to flash the Pixel 3a with CalyxOS using the instructions provided. On first boot, enable MicroG. This allows me to use a 'spoofed' google account for some background services.

Get bitcoin
-----------

Now that you've got your mobile phone flashed, it's time to download a Bitcoin wallet on your phone. [Samourai Wallet](https://samouraiwallet.com/) would be my choice of wallet. I'd write the passphrase and 12 recovery words down on a piece of paper. I would go to a local Bitcoin meetup and see if there would be anyone interested in selling me $300 worth of Bitcoin. Deposit the Bitcoin directly into Samourai wallet mobile app. For $300, it's probably not worth setting up a Samourai Wallet Dojo, but we will start a fresh wallet that connects to one later.

Getting an eSIM
---------------

Next step is to get access to a mobile phone carrier and get mobile credit in an anonymous way. The best way would be through [silent.link](https://silent.link/). Purchase an eSIM and add credit to the account paying with bitcoin. Import the eSIM into your Pixel 3a device and you now have access to a mobile network on your phone without KYC, worldwide.

Upgrading existing home setup
-----------------------------

This part assumes you have a home internet connection. Unfortunately, home internet connections require you to sign up using your identity and pay through traditional banking systems, not bitcoin.

In Australia, your internet service provider is required to keep 'metadata' on your internet connection for up to 2 years. 'Metadata' is loosely defined, but I assume it means timestamps of when you visited certain websites.

I would look to purchase a VPN (I like [Mullvad](https://mullvad.net/) for now) and pay with bitcoin for a yearly subscription. I would also purchase hardware for a pfsense router using [my article](https://k3tan.com/pfsense). I'd setup the VPN on the pfsense router such that all devices on the home network are protected by the VPN.

With the pfsense router, I would also look to install a VPN server on it such that I am able to access my network from anywhere in the world.

Incremental upgrades
--------------------

The next thing I would focus on is self hosting my data. I would purchase an old desktop computer (including monitor) off gumtree again, install Pop!_OS on to that. I would create a nextcloud virtual machine and a bitwardenrs virtual machine to host these services. [Nextcloud](https://nextcloud.com/) would store my calendar, contacts, photos and other documents. BitwardenRS would be my password manager. I'd learn how to back all this information up, storing the virtual machine images periodically on a spare SSD for safe keeping. I'd consider learning more about backups such as RAID configurations, but for the short term, periodic manual data backups would suffice.

The next thing I would look into is configuring a bitcoin node box. I would purchase an old computer, run Ubuntu server on it and go through [my tutorials](https://youtube.com/playlist?list=PLCRbH-IWlcW290O0N0lQV6efxuCA5Ja8c) on setting up a bitcoin node. I'd install bitcoin, a block explorer, samourai wallet dojo and whirlpool, lightning and btcpayserver if I wanted to earn bitcoin online. At this point, I'd create a new samourai wallet, pair it with my dojo and mix coins through whirlpool.

If I was pressed for time and wanted a quick and easy solution, I would look into one of the node packages available for raspberry pi - [myNode](http://mynodebtc.com/), [Umbrel](https://getumbrel.com/), [Raspiblitz](http://raspiblitz.org/) or [Ronin](https://ronindojo.io/).

My non-negotiables
------------------

Here is what I wouldn't do. I wouldn't sign up for google, facebook, tiktok, whatsapp or instagram accounts. The only social media I would have is a nym [twitter](https://twitter.com/) account and perhaps self hosting a mastodon/pleroma instance.

Instead of giving out my real email address, I would look to cloak my email address to sign up for services using [simplelogin.io](https://simplelogin.io/). One time use disposable emails I would consider using [guerrillamail](https://www.guerrillamail.com/) or [getnada](https://www.getnada.com/).

I would look to install and use as much [free and open source software](https://k3tan.com/foss) as I possibly could.

Conclusion
----------

Building out self sovereignty takes time and patience. If you want to upgrade your digital life, I would recommend watching the free [Go Incognito](https://youtube.com/playlist?list=PL3KeV6Ui_4CayDGHw64OFXEPHgXLkrtJO) course by Techlore.