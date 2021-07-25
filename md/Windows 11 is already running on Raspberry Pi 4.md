> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.windowslatest.com](https://www.windowslatest.com/2021/06/30/windows-11-is-already-running-on-raspberry-pi-4/)

> Windows 11 is up and running on the Rasberry Pi 4 after the same team managed to install Windows 10 o......

[![](https://www.windowslatest.com/wp-content/uploads/2021/06/Windows-11-Raspberry-Pi-4-696x392.jpg)](https://www.windowslatest.com/wp-content/uploads/2021/06/Windows-11-Raspberry-Pi-4.jpg)

Windows 11 is up and running on the Rasberry Pi 4 after the same team managed to install Windows 10 on the Raspberry Pi 4, as we saw last year.

If you want to run Windows on your Raspberry PI, then previously, you were required to use Windows 10 IoT Core version, which offers a basic Windows experience without desktop apps support as it is designed for low-powered devices.

Thanks to the [open-source project](https://github.com/worproject/) started by developers like [Mario](https://github.com/mariobalanica), we can now easily install Windows on ARM onto the Pi. Unlike the IoT Core version of Windows, Windows 11 on ARM or Windows 10 on ARM offer full-desktop experience on the Raspberry Pi, and emulation is supported as well.

Microsoft recently published [Windows 11 Build 22000.51](https://www.windowslatest.com/2021/06/28/windows-11-build-22000-51-is-now-available-for-insiders/) and developer Mario managed to get Windows 10 on ARM successfully running on the latest version of the Pi 4 Model B. As expected, the new version of Windows offers better performance than Windows 10 on the Pi and it can run the OS more smoothly.

The installation process of Windows 11 on ARM on Raspberry is the same as Windows 10 and [instructions can be found in our previous article](https://www.windowslatest.com/2020/07/24/install-windows-10-on-raspberry-pi/). You’ll need Raspberry Pi 4 with 4GB of RAM and a Windows 11 ARM64 image (which is can be created by converting the UUP image from the UUPDump platform).

It’s also recommended to use GPT for the new operating system. Likewise, you may need to [bypass the TPM 2.0 requirement](https://www.windowslatest.com/2021/06/28/youll-be-able-to-bypass-windows-11-tpm-2-0-requirement/) by modifying Windows Registry during the installation process if you see the error “This PC can’t run Windows”.

Of course, there’ll be some limitations in terms of the power of the Pi 4, especially when compared to Snapdragon-powered Windows on ARM devices, which Windows on ARM is designed for. However, one of the users who tested the operating system confirmed that the performance is somewhat better than Windows 10.

The project may not be perfect, but Windows on ARM is slowly getting better and it’s able to do important computing tasks on low-end devices like Pi too. Remember that Windows 11 itself is in the early stages with several known issues, including a bug where Windows Search may not work properly.

In addition to Raspberry Pi, some developers have also [managed to install the desktop OS on Lumia 950 XL](https://www.windowslatest.com/2021/06/30/developer-boots-windows-11-on-a-phone-and-it-surprisingly-scales-well/), which was released back in 2015.

[Windows 11 will begin rolling out to existing devices later this year](https://www.windowslatest.com/2021/06/28/microsoft-hints-at-windows-11-release-date/), with a wider rollout now expected in early 2022. Initially, flagship products like Surface devices will be offered the update via Windows Update. Based on the telemetry data, Microsoft will expand the rollout to include more configurations.