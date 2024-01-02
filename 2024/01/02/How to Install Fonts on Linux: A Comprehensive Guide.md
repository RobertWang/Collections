> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [linuxiac.com](https://linuxiac.com/how-to-install-fonts-on-linux/)

> This guide walks you through installing fonts on Linux, using the command line, or utilizing the dedi......

Fonts play a crucial role in enhancing text’s visual appeal and readability on your Linux system. Known for its flexibility and customization, Linux offers multiple ways to install fonts, catering to various user preferences and skill levels.

This guide will explore two main approaches: the command line and the dedicated graphical user interface (GUI) applications available in popular desktop environments like GNOME and KDE Plasma.

But first, let’s look over the various font type formats.

Types of Font Formats
---------------------

In Linux, several font formats are commonly used to store and manage fonts. Each has its characteristics and advantages, catering to different needs and use cases.

Here are some of the most common font formats you’ll encounter on Linux systems:

*   **TrueType Font (TTF):** TrueType is one of the most popular and widely supported font formats. TTF fonts are scalable and display well at various sizes. They are commonly used for both on-screen and print purposes, making them suitable for web design, graphic design, and general text rendering.
*   **OpenType Font (OTF):** OpenType is an extension of the TrueType font format. OTF fonts support advanced typography features, such as ligatures, alternative characters, and stylistic sets. They can include TrueType and PostScript font data, making them versatile for different platforms and applications.
*   **PostScript Font (Type 1):** PostScript fonts, often called Type 1 fonts, were one of the earliest digital font formats. While they have been largely replaced by TrueType and OpenType fonts, they still find use in specific legacy applications and printing workflows.
*   **Bitmap Font (BDF):** Bitmap fonts are created using a grid of pixels, where each pixel corresponds to a specific glyph. These fonts are not scalable and are best suited for particular screen resolutions. They were commonly used in older systems and terminals with limited screen space.
*   **X11 Font Format (XLFD):** The X Logical Font Description (XLFD) format was used in the X Window System to describe fonts. It included various attributes like font family, style, size, weight, etc. However, XLFD has become less relevant as newer font technologies and formats have emerged.
*   **Web Font Formats (WOFF, WOFF2):** While not exclusive to Linux, web font formats like WOFF (Web Open Font Format) and WOFF2 are optimized for web usage, providing efficient compression and faster loading times. They allow web designers to use custom fonts on their websites while ensuring cross-browser compatibility.
*   **SVG Fonts:** Scalable Vector Graphics (SVG) fonts use XML-based descriptions to define font outlines. While they offer scalability without losing quality, they are less common than TrueType or OpenType fonts for general text usage.

These font formats vary in their capabilities, compatibility, and use cases. When installing fonts on a Linux system, choosing a format that suits your intended purpose, whether for general text rendering, design work, or web development, is essential.

In this guide, we will focus on the OTF and TTF formats as they are the most used.

Installing Fonts on Linux: A Command-Line Approach
--------------------------------------------------

Adding new fonts to a Linux system through the command line offers a swift and efficient method for enhancing typographic choices. By utilizing simple commands, users can easily manage font caches and refresh font lists. Let me show you how.

Step 1: Obtain the Font File
----------------------------

The first step is to download the font file we want to install on our Linux system. For this guide, we will use the [Fira Code](https://github.com/tonsky/FiraCode) font, available for free download from GitHub.

We download it with the following [wget command](https://linuxiac.com/wget-command-examples/):

```
wget https://github.com/tonsky/FiraCode/releases/download/6.2/Fira_Code_v6.2.zip

```

[After unzipping it](https://linuxiac.com/how-to-unzip-in-linux/), we will get several folders with different font types – TTF, WOFF, and WOFF2. We will use TTF.

Step 2: Install Font
--------------------

Here, we must decide how we want the font to be available on our system. Linux provides two main ways to install fonts: **system-wide** and **per-user**.

System-wide font installation involves installing fonts that will be available to all users on the system. This typically requires administrative privileges and affects the entire system’s appearance.

Per-user font installation allows users to install fonts only available to them. This can be useful if you want to customize your environment without affecting other users on the system.

Below, we will show you how to install the font with both approaches.

### Step 2.1: System-Wide Font Installation

Most Linux distributions store the system-wide fonts in the “_/usr/share/fonts_” directory.

To install the font system-wide, you need to copy or move the font files (“_.ttf_ “) into that directory, and to make them easier to manage, it is good practice to put them in the one you create in advance. So let’s do it.

First, we will create a new sub-directory under “_/usr/share/fonts_” with the name “_fira-code_” and then copy or move the font files to it.

```
sudo mkdir /usr/share/fonts/fira-code
sudo mv *.ttf /usr/share/fonts/fira-code/

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-system-wide-linux.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-system-wide-linux.jpg)Install font on Linux system-wide.

Finally, you’ll need to update the system’s font cache so that applications can recognize the new fonts. Running the below command will regenerate the font cache.

```
sudo fc-cache -f -v

```

After regenerating the cache, the command’s output shows your Linux system’s font folders, including our newly created directory containing the Fira Code font.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-system-wide-linux2.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-system-wide-linux2.jpg)Install font on Linux system-wide.

Using the `fc-list` command, you can confirm that the fonts are installed by displaying the paths and style definitions:

```
fc-list | grep "Fira"

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-system-wide-linux6-1024x274.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-system-wide-linux6.jpg)Verifying font installation.

And that’s it. The font is successfully installed and ready to be used by applications on our Linux system.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-system-wide-linux4-1024x564.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-system-wide-linux4.jpg)The font is successfully installed and available on our Linux system.

Need more details? Check [fc-cache](https://linux.die.net/man/1/fc-cache) and [fc-list](https://linux.die.net/man/1/fc-list) commands man pages.

### Step 2.1: Per-User Font Installation

Per-user font installation on Linux allows users to install fonts only accessible within their own user accounts. This approach offers a personalized way to customize the appearance of text in applications without affecting other users or the system as a whole.

The procedure is identical to the system-wide installation; the only difference is that this time, the directory where we will install the fonts is in our home Linux directory. More appropriately, on most Linux systems, this is “_~/.local/share/fonts_.”

The directory may not exist, so let’s first make it with the sub-directory where we will install the font.

```
mkdir -p ~/.local/share/fonts/fira-code

```

Then, familiarly, copy or move the font files there and regenerate the cache.

```
mv *.ttf ~/.local/share/fonts/fira-code
sudo fc-cache -f -v

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-system-wide-linux5.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-system-wide-linux5.jpg)Install font on Linux per user.

That all! Enjoy your new font.

Install Fonts on Linux with GNOME’s Fonts Manager
-------------------------------------------------

The GNOME Fonts Manager provides a user-friendly and graphical way to manage and install fonts on Linux systems running the GNOME desktop environment. This method simplifies the font installation process and is especially helpful for users who prefer a more intuitive approach.

Go to the font file you want to install, right-click, and select “_Open With Fonts_” from the context menu.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-gnome1.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-gnome1.jpg)Install fonts on Linux with GNOME’s Fonts Manager.

The font will open in the default GNOME font management application. Click the “_Install_” button to install the font.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-gnome2.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-gnome2.jpg)Install fonts on Linux with GNOME’s Fonts Manager.

You are ready. The font is installed. How much easier could it be? Repeat the procedure for all the other files if more than one.

Finally, we’ll point out that the fonts are thus installed just for the current user and can be located in the “_~/.local/share/fonts_” directory.

Install Fonts on Linux with KDE’s Fonts Manager
-----------------------------------------------

KDE Plasma, one of the popular desktop environments for Linux, provides a user-friendly Fonts Manager that simplifies installing and managing fonts on your system.

Go to “_System Settings_” > “_Appearance_” > “_Font Management_” and click on the “_Install from File_” button.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-kde-1024x739.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-kde.jpg)KDE’s Fonts Manager

Then select the font files you want to install and click the “_Open_” button.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-kde2-1024x739.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-kde2.jpg)Install fonts on Linux with KDE’s Fonts Manager.

You’ll be asked whether you want the fonts installed system-wide or per-user. My advice is to choose “_System_.”

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-kde3-1024x739.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-kde3.jpg)Install system-wide fonts on Linux with KDE’s Fonts Manager.

You will be prompted to enter your [password for the user](https://linuxiac.com/how-to-change-remove-disable-user-password-in-linux/) with administrative privileges to execute sudo commands. So, enter it and confirm with “_OK_.”

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-kde4-1024x739.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-kde4.jpg)User authentication.

The fonts will be installed; you can see them on the list immediately.

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/install-font-linux-kde5-1024x739.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/install-font-linux-kde5.jpg)Install system-wide fonts on Linux with KDE’s Fonts Manager.

Uninstall Fonts on Linux
------------------------

Uninstalling fonts on Linux can be necessary to clean up your font collection or remove fonts you no longer need. Whether you’ve installed fonts system-wide or for a specific user, removing them is a straightforward process. Here’s how to do it.

### Step 1: Locate Fonts Directory

Open a terminal window and search for specific fonts using the `fc-list` command by filtering its output and pipelining it to the grep command with the font name you want to remove.

```
fc-list | grep -i "fira"

```

[![](https://cdn.shortpixel.ai/spai/q_glossy+ret_img+to_auto/linuxiac.com/wp-content/uploads/2023/08/uninstall-fonts-on-linux2-1024x273.jpg)](https://linuxiac.b-cdn.net/wp-content/uploads/2023/08/uninstall-fonts-on-linux2.jpg)Locate the fonts directory.

### Step 2: Delete Font Files/Directory

Once we have found where the font files are located in the filesystem, we delete them, or as in our case, delete the entire sub-directory they are in.

```
sudo rm -rf /usr/share/fonts/fira-code/

```

### Step 3: **Update** Fonts Cache

After removing the fonts, update the system’s font cache to reflect the changes.

```
sudo fc-cache -f -v

```

Remember: When uninstalling fonts, make sure you are removing the correct fonts. Removing system fonts improperly could affect system stability.

Additionally, in some cases, you might need to log out and then log back in for applications to recognize the changes.

Conclusion
----------

This guide is a comprehensive resource for users looking to enhance their font selection and customization on Linux by installing additional fonts on their systems.

So, whether you’re a seasoned Linux user comfortable with the command line or a newer user seeking a more straightforward GUI approach, this guide equips you with the knowledge to enhance your system’s aesthetics through font installation.

Thanks for your time! Your feedback and comments are most welcome.