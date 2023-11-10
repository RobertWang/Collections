> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [linuxiac.com](https://linuxiac.com/how-to-find-linux-os-installation-date/)

> Want to know when your Linux system was installed? Don't wonder anymore! Here's how to do it quickly ......

**Want to know when your Linux system was installed? Don’t wonder anymore! Here’s how to do it quickly with a single simple command.**

Have you ever found yourself pondering the age of your Linux system? Perhaps you’ve inherited a computer or are curious about when you first set up your trusty Linux machine.

In this article, we’ll show you a straightforward and efficient method to uncover the installation date of your Linux system using just a single command.

So, let’s embark on this short journey, and you’ll soon have a clear answer to the question, “When was my Linux system installed?”

Find Date and Time Linux OS Was Installed
-----------------------------------------

Finding this information can be done in many different ways, and below, we present you with the best ones, both valid for any Linux distribution and specific to a particular one.

### A Universal Method

First, we must clarify that there is no dedicated setting, variable, log file entry, or similar in Linux designed to expressly store information about the actual day and time when the system was installed.

However, it can be easily retrieved by determining when the root (“/”) file system was created.

This can be accomplished with the help of the `stat` command, which displays detailed information about a file or directory, including metadata such as create, access, and modification timestamps, file type, permissions, and more.

So, to find the exact day and time when your Linux system was installed, run the following:

```
stat / | awk '/Birth: /{print $2 " " substr($3,1,5)}'

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+w_942+h_193+to_auto+ret_img/linuxiac.com/wp-content/uploads/2023/09/linux-install-date01.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/linux-install-date01.jpg)Find the exact date and time when Linux was installed.

And that’s all! We get the information we need right away. According to the command’s output above, the system was installed on **December 22, 2020, at 2:14 AM**.

In the above example, to display in a convenient way only the information we need, we have piped the `stat` command’s output to the `awk` and further formatted it using the `substr` function.

However, if you want to keep things as simple as possible, you can run the `stat` command below and look at the “_Birth_” row, indicating the moment the item was created on the file system.

```
stat /

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+w_942+h_366+to_auto+ret_img/linuxiac.com/wp-content/uploads/2023/09/linux-install-date02.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/linux-install-date02.jpg)Find the exact date and time when Linux was installed using the stat command.

Need more details? Check the `stat` command [man page](https://man7.org/linux/man-pages/man1/stat.1.html).

Another alternative approach to get information about when the file system was created, which is the Linux system’s installation time, is to use the command below. Remember that you must switch to the root user before running it.

```
fsname=$(df / | tail -1 | cut -f1 -d' ') && tune2fs -l $fsname | grep 'created'

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+w_1026+h_247+to_auto+ret_img/linuxiac.com/wp-content/uploads/2023/09/linux-install-date03.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/linux-install-date03.jpg)Get information about when the file system was created.

Of course, other ways exist to discover when the [Linux operating system](https://linuxiac.com/what-is-linux-operating-system/) was installed. Still, unlike the distro-agnostic ones shown above, they are tied to the particular distribution used. Here they are.

### Debian / Ubuntu

On Debian and Ubuntu systems and all their derivatives like Linux Mint and others, you can see the exact day and time they were installed by displaying the first line of the installer syslog file.

```
sudo head -n1 /var/log/installer/syslog

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+w_926+h_203+to_auto+ret_img/linuxiac.com/wp-content/uploads/2023/09/linux-install-date06.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/linux-install-date06.jpg)Find the installation date and time on Debian / Ubuntu / Linux Mint.

### Fedora / Rocky Linux / AlmaLinux

In Fedora, Red Hat Enterprise Linux and all its forks, such as Rocky Linux, AlmaLinux, Oracle Linux, etc., we can use it as a reliable marker of when the operating system was installed by checking the installation date of the “_basesystem_” package.

```
sudo rpm -qi basesystem | grep -i "install date"

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+w_887+h_182+to_auto+ret_img/linuxiac.com/wp-content/uploads/2023/09/linux-install-date05.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/linux-install-date05.jpg)Find the exact OS’s installation date and time on Fedora / RHEL / Rocky Linux / AlmaLinux / Oracle Linux.

Keep in mind, however, that if you were performing an in-place upgrade, say from Fedora 38 to Fedora 39, Rocky 9.1 to Rocky 9.2, etc., the date that will be displayed when retrieving the “_basesystem_” package information will be that of the time of the upgrade.

In this case, to get the actual date of the initial installation, use some of the first two universal methods based on the `stat` command mentioned at the beginning of the article.

### Arch Linux

If you’re running Arch or one of its derivatives like Manjaro or EndeavourOS, the first line of the “_pacman.log_” file will tell you when your Linux system was installed.

```
head -n1 /var/log/pacman.log

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+w_937+h_203+to_auto+ret_img/linuxiac.com/wp-content/uploads/2023/09/linux-install-date07.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/linux-install-date07.jpg)Find the exact OS’s installation date and time on Arch Linux / Manjaro / EndeavourOS.

Bottom Line
-----------

The quest to determine the installation date of your Linux system need not be shrouded in uncertainty any longer. Using the methods outlined in this guide, you can swiftly and effortlessly unveil your Linux system’s birthdate.

So, when is your Linux system installed? We can’t wait for you to let us know in the comments below.