> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mcla.ug](https://mcla.ug/blog/emulating-stm32-qemu.html)

> Sat 04 April 2020

Sat 04 April 2020

I recently published a blog post titled [How to flash an LED](https://mcla.ug/blog/how-to-flash-an-led.html) about writing ARM assembly for an STM32. I was running my code on a [1bitsy](https://1bitsy.org/) development board, but I wanted anyone to be able to have a go at writing the assembly and testing it – even if they didn't have the right hardware. You can find [the repo on my Github](https://github.com/lochsh/gpio-toggle-exercise).

Finding the specific QEMU tools I needed took a while, so I wanted to document it in this blog post.

The quest for a QEMU STM32F4 machine
------------------------------------

[QEMU](https://www.qemu.org/) is a "generic and open source machine emulator and virtualizer". An example where emulation is useful: if you are writing software for an embedded target, reliable automated tests can be a challenge. Emulating your embedded target on your host computer makes allows for easier testing, and for isolating problems to do with the real hardware from problems to do with the software.

The 1bitsy has a STM32F415 microcontroller, so I was looking for emulation for a development board with a similar MCU. QEMU comes with a set of built-in machines, and you can write your own machines to emulate the hardware you desire.

Unfortunately, none of the built-in machines suited my purposes. There were multiple Cortex M3 machines, but none of them were STM32s, and there were no Cortex M4 machines. I wanted to use an STM32F4 so as to avoid confusion for people going from my assembly blog post to the emulation. I also gave the blog post as a talk at work, and didn't want my coworkers to be confused by the inconsistency. Creating my own QEMU machine was not remotely feasible on the timescales I was working on for my work talk, and I believe it is a lot of work! So, I searched the internet for an available STM32F4 QEMU machine.

### Enter GNU MCU Eclipse...maybe

During this search, I came across many mentions of [GNU MCU Eclipse](https://gnu-mcu-eclipse.github.io/), a suite of tools for the Eclipse IDE which included an STM32F4 Discovery board.

I am a [vim](https://www.vim.org/) die-hard with no desire to use Eclipse, so I was hopeful to find a way to be able to extract these tools and easily use them outside of Eclipse.

It turns out the fork of QEMU that this toolsuite is using is available separately as [xPack QEMU ARM](https://xpack.github.io/qemu-arm/).

### xPack QEMU ARM has an STM32F4 Discovery machine

xPack [recommend](https://xpack.github.io/qemu-arm/install/) using [xpm](https://www.npmjs.com/package/xpm) to install their QEMU ARM fork, but I found the manual installation instructions worked fine on Ubuntu. This gives us, among other things, the `qemu-system-gnuarmeclipse` binary.

Connecting to the emulator via gdb
----------------------------------

We can run our target executable on the emulator like this, with the `gdb` switch telling QEMU to wait for a connection from gdb before continuing the program's execution. Here, we tell it listen on TCP port 3333.

```
1 qemu-system-gnuarmeclipse \
2     -cpu cortex-m4 \
3     -machine STM32F4-Discovery \
4     -gdb tcp::3333 \
5     -nographic \
6     -kernel "${TARGET}"


```

Then we can connect from gdb:

```
1 gdb-multiarch \
2     -q "${TARGET}" \
3     -ex "target remote :3333"


```

Automated testing of ARM assembly on the STM32F4 emulator
---------------------------------------------------------

I put all this together into a bash script so I could test that assembly for toggling a GPIO pin was doing what it should, by reading register values from gdb at breakpoints set in the assembly file:

```
 1 
 2 set -euo pipefail
 3 TARGET=$1
 4 
 5 qemu-system-gnuarmeclipse \
 6     -cpu cortex-m4 \
 7     -machine STM32F4-Discovery \
 8     -gdb tcp::3333 \
 9     -nographic \
10     -kernel "${TARGET}" > /dev/null &
11 QEMU_PID=$!
12 
13 if [ ! -d "/proc/${QEMU_PID}" ]
14 then
15     echo -ne "\033[31m Failed to start QEMU"
16     echo -e "\033[0m"
17     exit 1
18 fi
19 
20 function read_address() {
21     local ADDRESS=$1
22     VALUE=$(gdb-multiarch \
23         -q "${TARGET}" \
24         -ex "target remote :3333" \
25         -ex "x/1xw ${ADDRESS}" \
26         --batch | tail -1 | cut -f 2)
27 }
28 
29 function test_address() {
30     local ADDRESS=$1
31     local REGISTER_NAME=$2
32     local EX_VALUE=$3
33 
34     read_address "${ADDRESS}"
35     if [ "$VALUE" = "${EX_VALUE}" ]
36     then
37         echo -ne "\033[32m✓ ${REGISTER_NAME} correctly set to ${EX_VALUE}"
38     else
39         echo -ne "\033[31m✘ ${REGISTER_NAME} was ${VALUE}, want ${EX_VALUE}"
40     fi
41 
42     echo -e "\033[0m"
43 }
44 
45 test_address "0x40023830" "AHB1ENR" "0x00000001"
46 test_address "0x40020000" "GPIOA_MODER" "0xa8010000"
47 test_address "0x40020014" "GPIOA_ODR" "0x00000100"
48 
49 kill ${QEMU_PID} &> /dev/null


```

The output looks like this for correct assembly, which you can view [here](https://github.com/lochsh/gpio-toggle-exercise/blob/master/sneak-peek/working-gpio-toggle.s).

![](https://mcla.ug/blog/images/qemu-output.png)

This testing is limited, but I'm pleased with it! I put everything in a Dockerfile in the Github repo to make life easier for people trying this out.

Bonus 1337 screenshot
---------------------

This is a screenshot from when I finally got all this working after a couple evenings of trials:

![](https://mcla.ug/blog/images/qemu-all-screenshot.png)

Clockwise from left is the assembly, the bash script, a gdb session where I'd been poking around, and me running the bash script.

I hope this is post helpful to anyone wanting to emulate an STM32F4 without having to do it all themselves, and that the tools in my [GPIO toggling exercise](https://github.com/lochsh/gpio-toggle-exercise) are useful to anyone who wants to play with ARM assembly but doesn't have the hardware.