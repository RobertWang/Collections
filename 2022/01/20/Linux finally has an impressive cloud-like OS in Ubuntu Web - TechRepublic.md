> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.techrepublic.com](https://www.techrepublic.com/article/linux-finally-has-an-impressive-cloud-like-os-in-ubuntu-web/) [![](https://www.techrepublic.com/a/hub/i/r/2021/11/10/eaede8c5-4e5a-4274-839e-77ab0279846c/resize/770x/cdfc78590063cccbe1f78281eb6798e1/ubuntuweb.jpg)](https://www.techrepublic.com/a/hub/i/r/2021/11/10/eaede8c5-4e5a-4274-839e-77ab0279846c/resize/770x/cdfc78590063cccbe1f78281eb6798e1/ubuntuweb.jpg)

Image: Jack Wallen

Linux powers the cloud. But for the longest time, the operating system that single-handily makes the cloud possible didn't really have a desktop distribution that offered much in the way of applications that interacted well with the cloud. Yes, there's a Dropbox app and a few third-party tools that can be installed to sync your desktop to cloud storage accounts ... but not much more.

**SEE:** [**40+ open source and Linux terms you need to know**](https://www.techrepublic.com/resource-library/downloads/40-open-source-and-linux-terms-you-need-to-know/) **(TechRepublic Premium)**

### More about open source

*   [Log4j vulnerability: Why your hot take on it is wrong](https://www.techrepublic.com/article/log4j-vulnerability-your-hot-take-is-wrong/)
*   [Open source year in review: 2021](https://www.techrepublic.com/article/open-source-year-in-review-2021/)
*   [Android in 2021 year in review: The highs and the lows](https://www.techrepublic.com/article/android-2021-a-year-in-review/)
*   [Linux turns 30: Celebrating the open source operating system (free PDF)](https://www.techrepublic.com/resource-library/downloads/linux-turns-30-celebrating-the-open-source-operating-system-free-pdf/)

Although GNOME makes it possible to connect your desktop to your Google Drive account, that solution is far from viable for most users. In fact, the Linux desktop and the cloud have been really hit or miss for the longest time.

And then comes [Ubuntu Web](https://discourse.ubuntu.com/t/ubuntu-web-remix/19394). This new-ish distribution promises to be the Chrome OS for Linux and, wow, does it achieve just that. To be honest, when I first heard about the remix I was doubtful. I'd seen so many distributions attempt them and, for the most part, fail. So, with trepidation, I downloaded the latest version of Ubuntu Web, spun up a VM, and gave it the test.

Upon completing the installation, I logged in and was greeted by a window I'd never seen before in a Linux distribution. Said window required I log in. But to what account? It didn't take me long to realize it was requesting I log into an [/e/ foundation account](https://e.foundation/e-email-invite/) (which I already had). Logging into the /e/ account makes it possible for you to take advantage of a rather nifty trick Ubuntu Web has up its sleeve. Said trick is WayDroid, a port of Anbox which allows users to install Android apps from the /e/ store.

That's when doubt started to trickle away. 

These Android apps perform as if they were native to the OS, and there are plenty of apps to choose from (**Figure A**).

**Figure A**

[![](https://www.techrepublic.com/a/hub/i/r/2021/11/10/4b8e52de-05af-400c-af1d-e43988810677/resize/770x/19aa0682ba2903fa452bc64ed91fc635/ubuntuweba.jpg)![ubuntuweba.jpg](https://www.techrepublic.com/a/hub/i/r/2021/11/10/4b8e52de-05af-400c-af1d-e43988810677/resize/770x/19aa0682ba2903fa452bc64ed91fc635/ubuntuweba.jpg)](https://www.techrepublic.com/a/hub/i/r/2021/11/10/4b8e52de-05af-400c-af1d-e43988810677/resize/770x/19aa0682ba2903fa452bc64ed91fc635/ubuntuweba.jpg)

Installing Android apps from the /e/ app store on Ubuntu Web.

One caveat to WayDroid is that it doesn't function as expected when Ubuntu Web is running as a virtual machine. you can use the app, but the installer only downloads the .wapp file for the app (instead of installing it). According to the developer, when Ubuntu Web is installed on bare metal, the app performs as expected.

The fun didn't stop there with WayDroid. Once you've logged into your /e/ account, you'll find a special file manager has some special features in store. Open /e/ Files (from the Application Overview) and you'll find tabs for files, email, contacts, calendar, notes, tasks and photos (**Figure B**).

**Figure B**

[![](https://www.techrepublic.com/a/hub/i/r/2021/11/10/2311128d-a126-4f9b-9284-a8bfa8455394/resize/770x/6715840520ceb9353872aaa53d91238f/ubuntuwebb.jpg)![ubuntuwebb.jpg](https://www.techrepublic.com/a/hub/i/r/2021/11/10/2311128d-a126-4f9b-9284-a8bfa8455394/resize/770x/6715840520ceb9353872aaa53d91238f/ubuntuwebb.jpg)](https://www.techrepublic.com/a/hub/i/r/2021/11/10/2311128d-a126-4f9b-9284-a8bfa8455394/resize/770x/6715840520ceb9353872aaa53d91238f/ubuntuwebb.jpg)

The /e/ Files app with the added /e/ goodness.

This is truly something no other Linux "cloudified" desktop has ever achieved and it's pretty amazing. All of a sudden the file manager is like having an alternative to Google Workspace installed, dashing my doubts that Linux would never have a solid relationship with the cloud on the desktop. For example, click on the Files section of the file manager, click the New Document button, and ONLYOFFICE opens so you can create your new document. Save that file and then point your browser to your [/e/ cloud account](https://ecloud.global). Guess what? That file is available for you to use from anywhere.

**SEE:** [**Linux turns 30: Celebrating the open source operating system (free PDF)**](https://www.techrepublic.com/resource-library/downloads/linux-turns-30-celebrating-the-open-source-operating-system-free-pdf/) **(TechRepublic)**

It should be noted that there are really two file managers installed, there's /e/ Files, for working with your cloud account and GNOME Files, for working with local files. Also, from the Application Overview, you can launch /e/ Apps, /e/ Calendar, /e/ Contacts, /e/ Email, /e/ Notes, /e/ WayDroid, /e/ Photoes, and /e/ Tasks. Each of these launchers will open the associated /e/ cloud application.

Who is Ubuntu Web for?
----------------------

This is a tricky question to answer. To try to simplify the answer looks something like this: If you're looking for an operating system that functions similarly to Chrome OS but want to cut ties with Google, Ubuntu Web might well be the ideal platform. This Linux distribution is easy to use, and works seamlessly with the /e/ ecosystem. All the while, Ubuntu Web can function as a full-blown Linux operating system, so it's like getting the best of both worlds.

Ubuntu Web has crashed through that barrier and makes working with the cloud on Linux a pretty impressive experience. Give this OS a try and see if it doesn't make you realize the [Linux love/hate relationship with the cloud](https://www.techrepublic.com/article/the-lovehate-relationship-the-cloud-has-with-linux/) can be mostly love.

**_Subscribe to TechRepublic's [How To Make Tech Work on YouTube](https://www.youtube.com/channel/UCKyMiy1zmJ7aZ8aP6DLZLIA) for all the latest tech advice for business pros from Jack Wallen._**

![](https://www.techrepublic.com/a/fly/bundles/techrepubliccss/images/article-NLSthumb.jpg)

### Open Source Weekly Newsletter

You don't want to miss our tips, tutorials, and commentary on the Linux OS and open source applications. Delivered Tuesdays

Sign up today

Also see
--------

*   [Linux 101: How to execute commands from within the nano text editor](https://www.techrepublic.com/article/linux-101-how-to-execute-commands-from-within-the-nano-text-editor/) (TechRepublic)
*   [Linux 5.14 kernel: New and exciting features coming to the release](https://www.techrepublic.com/article/new-and-exciting-features-coming-to-the-linux-5-14-kernel/) (TechRepublic)
*   [How to become a developer: A cheat sheet](https://www.techrepublic.com/article/how-to-become-a-developer-a-cheat-sheet/) (TechRepublic)
*   [How-to guide for Linux administrators (free PDF)](https://www.techrepublic.com/resource-library/whitepapers/how-to-guide-for-linux-administrators-free-pdf/?r=1913753406/) (TechRepublic)
*   [Linux 101: What tech pros need to know](https://www.techrepublic.com/resource-library/whitepapers/linux-101-what-tech-pros-need-to-know/?r=2027107033) (TechRepublic Premium)
*   [Linux, Android, and more open source tech coverage](https://flipboard.com/@techrepublic/linux-android-and-more-open-source-tech-lg1d5t1ky) (TechRepublic on Flipboard)