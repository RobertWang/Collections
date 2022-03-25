> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.trickster.dev](https://www.trickster.dev/post/setting-up-mitmproxy-with-ios15/)

> Code level discussion of web scraping, gray hat automation, growth hacking and bounty hunting

[mitmproxy](https://mitmproxy.org/) is an open source proxy server developed for launching man-in-the-middle attacks against network communications (primarily HTTP(S). mitmproxy enables passive sniffing, active modification and replaying of HTTP messages. It is meant to be used for troubleshooting, reverse engineering and penetration testing of networked software.

We will be setting up mitmproxy with iOS 15 device for the scraping and gray hat automation purposes. One use case of mitmproxy-iPhone setup is discussed in my previous post about [scraping private API of mobile app](https://www.trickster.dev/post/using-python-and-mitmproxy-to-scrape-private-api-of-mobile-app/).

At this point, the only prerequisite for the following procedure is that iOS device and your computer is connected to the same IP network (it’s okay if computer is using Ethernet connection and iPhone is using WiFi, but the IP subnet has to match). You should be good to go if both are connected to the same WiFi network.

First we need to install mitmproxy on our local machine. Simple and safe way to do this is to follow what the official site tells you to do:

*   On Linux, download the binary tarball and install it. Also some Linux distribution provide a mitmproxy package in the package managers.
*   On macOS, use Homebrew command `brew install mitmproxy`
*   On Windows, download the installer executable and run it.

You can also use the official [Docker image](https://hub.docker.com/r/mitmproxy/mitmproxy/) that is available on Docker Hub.

mitmproxy installation can be checked as follows by running `mitmproxy --version`. Now you can use mitmproxy TUI by running `mitmproxy` command or web interface by running `mitmweb` command. There is also `mitmdump` command that works bit like tcpdump and provides no interactivity. Have mitmproxy running for the next steps.

Find out your IP address of your local computer _within the local subnet_. Depending on your operating system, there will be a platform-specific way to do this:

*   On macOS and some Linux systems, run `ifconfig` and note the IP address for your primary network interface that you are using (something like `eth0` or `en0`). You can also check GUI settings app (on macOS, IP address is available at Settings -> Network -> Your primary network).
*   On Windows, it will be available in the output of `ipconfig` command and also in the network settings GUI.
*   On some Linux systems, it will be available in the output of `ip addr` command.

Now open the Settings app of your iOS device and go to Wi-Fi part. Press the (i) button next to the name of your WiFi network. In next page, scroll down to bottom and tap on Configure Proxy cell. Next, choose Manual. Three new cells will appear. In the Server field, enter IP address of your computer. In the port field, enter 8080.

Now we need to install the root X.509 certificate to intercept HTTP traffic within TLS connections. Open Safari on your iPhone and go to [mitm.it](http://mitm.it/). Press the green button below “iOS” label. You will be asked if you want to download configuration profile - press Allow button. Now go back to Settings app. You will see “Profile Downloaded” cell on the top. Pressing it will open the “Install Profile” dialog with Install button at the top - press it and verify the installation with your passcode.

There is one step left on the iOS device side. Go back to the home screen of Settings app and go “General” -> “About” -> “Certificate Trust Settings”. Make sure the switch next to “mitmproxy” text is on. At this point, your mitmproxy instance should be showing HTTP requests that iOS and various apps are making in the background.

If this does not work right away - double check your settings and try switching your iOS device to airplane mode and back. Note that some apps are using certificate pinning and will not work if traffic is being intercepted like this - more on this later.

Code level discussion of web scraping, gray hat automation, growth hacking and bounty hunting

* * *

By rl1987, 2022-01-23