> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [stackoverflow.com](https://stackoverflow.com/questions/22054578/how-to-run-a-program-without-an-operating-system/32483545#32483545) 

[[simhumileco]]

How do you run a program all by itself without an operating system running? Can you create assembly programs that the computer can load and run at startup, e.g. boot the computer from a flash drive and it runs the program that is on the CPU?

[[user2320609]]

> How do you run a program all by itself without an operating system running?

You place your binary code to a place where processor looks for after rebooting (e.g. address 0 on ARM).

> Can you create assembly programs that the computer can load and run at startup (e.g. boot the computer from a flash drive and it runs the program that is on the drive)?

General answer to the question: it can be done. It's often referred to as"bare metal programming". To read from flash drive, you want to know what's USB, and you want to have some driver to work with this USB. The program on this drive would also have to be in some particular format, on some particular filesystem... This is something that boot loaders usually do, but your program could include its own bootloader so it's self-contained, if the firmware will only load a small block of code.

Many ARM boards let you do some of those things. Some have boot loaders to help you with basic setup.

[Here](https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/) you may find a great tutorial on how to do a basic operating system on a Raspberry Pi.

Edit: This article, and the whole wiki.osdev.org will anwer most of your questions [http://wiki.osdev.org/Introduction](http://wiki.osdev.org/Introduction)

Also, if you don't want to experiment directly on hardware, you can run it as a virtual machine using hypervisors like qemu. See how to run"hello world" directly on virtualized ARM hardware [here](http://balau82.wordpress.com/2010/02/28/hello-world-for-bare-metal-arm-using-qemu/).

[[Peter Cordes]]

**Runnable examples**

Let's create and run some minuscule bare metal hello world programs that run without an OS on:

*   an x86 [Lenovo Thinkpad T430](https://www.lenovo.com/gb/en/laptops/thinkpad/t-series/t430/) laptop with UEFI BIOS 1.16 firmware
*   an ARM-based [Raspberry Pi 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

We will also try them out on the QEMU emulator as much as possible, as that is safer and more convenient for development. The QEMU tests have been on an Ubuntu 18.04 host with the pre-packaged QEMU 2.11.1.

The code of all x86 examples below and more is present on [this GitHub repo](https://github.com/cirosantilli/x86-bare-metal-examples).

**How to run the examples on x86 real hardware**

Remember that running examples on real hardware can be dangerous, e.g. you could wipe your disk or brick the hardware by mistake: only do this on old machines that don't contain critical data! Or even better, use cheap semi-disposable devboards such as the Raspberry Pi, see the ARM example below.

For a typical x86 laptop, you have to do something like:

1.  Burn the image to an USB stick (will destroy your data!):
    
    ```
    sudo dd if=main.img of=/dev/sdX
    ```
    
2.  plug the USB on a computer
    
3.  turn it on
    
4.  tell it to boot from the USB.
    
    This means making the firmware pick USB before hard disk.
    
    If that is not the default behavior of your machine, keep hitting Enter, F12, ESC or other such weird keys after power-on until you get a boot menu where you can select to boot from the USB.
    
    It is often possible to configure the search order in those menus.
    

For example, on my T430 I see the following.

After turning on, this is when I have to press Enter to enter the boot menu:

[![](https://i.stack.imgur.com/WfLBl.jpg)](https://i.stack.imgur.com/WfLBl.jpg)

Then, here I have to press F12 to select the USB as the boot device:

[![](https://i.stack.imgur.com/S9zBv.jpg)](https://i.stack.imgur.com/S9zBv.jpg)

From there, I can select the USB as the boot device like this:

[![](https://i.stack.imgur.com/fxoDn.jpg)](https://i.stack.imgur.com/fxoDn.jpg)

Alternatively, to change the boot order and choose the USB to have higher precedence so I don't have to manually select it every time, I would hit F1 on the"Startup Interrupt Menu" screen, and then navigate to:

[![](https://i.stack.imgur.com/8pIGP.jpg)](https://i.stack.imgur.com/8pIGP.jpg)

**Boot sector**

On x86, the simplest and lowest level thing you can do is to create a [Master Boot Sector (MBR)](https://en.wikipedia.org/wiki/Master_boot_record), which is a type of [boot sector](https://en.wikipedia.org/wiki/Boot_sector), and then install it to a disk.

Here we create one with a single `printf` call:

```
printf '\364%509s\125\252' > main.img
sudo apt-get install qemu-system-x86
qemu-system-x86_64 -hda main.img
```

Outcome:

[![](https://i.stack.imgur.com/CvZ7r.png)](https://i.stack.imgur.com/CvZ7r.png)

Note that even without doing anything, a few characters are already printed on the screen. Those are printed by the firmware, and serve to identify the system.

And on the T430 we just get a blank screen with a blinking cursor:

[![](https://i.stack.imgur.com/Gn8nH.jpg)](https://i.stack.imgur.com/Gn8nH.jpg)

`main.img` contains the following:

*   `\364` in octal == `0xf4` in hex: the encoding for a `hlt` instruction, which tells the CPU to stop working.
    
    Therefore our program will not do anything: only start and stop.
    
    We use octal because `\x` hex numbers are not specified by POSIX.
    
    We could obtain this encoding easily with:
    
    ```
    echo hlt > a.S
    as -o a.o a.S
    objdump -S a.o
    ```
    
    which outputs:
    
    ```
    a.o:     file format elf64-x86-64
    
    
    Disassembly of section .text:
    
    0000000000000000 <.text>:
       0:   f4                      hlt
    ```
    
    but it is also documented in the Intel manual of course.
    
*   `%509s` produce 509 spaces. Needed to fill in the file until byte 510.
    
*   `\125\252` in octal == `0x55` followed by `0xaa`.
    
    These are 2 required magic bytes which must be bytes 511 and 512.
    
    The BIOS goes through all our disks looking for bootable ones, and it only considers bootable those that have those two magic bytes.
    
    If not present, the hardware will not treat this as a bootable disk.
    

If you are not a `printf` master, you can confirm the contents of `main.img` with:

```
hd main.img
```

which shows the expected:

```
00000000  f4 20 20 20 20 20 20 20  20 20 20 20 20 20 20 20  |.               |
00000010  20 20 20 20 20 20 20 20  20 20 20 20 20 20 20 20  |                |
*
000001f0  20 20 20 20 20 20 20 20  20 20 20 20 20 20 55 aa  |              U.|
```

where `20` is a space in ASCII.

The BIOS firmware reads those 512 bytes from the disk, puts them into memory, and sets the PC to the first byte to start executing them.

**Hello world boot sector**

Now that we have made a minimal program, let's move to a hello world.

The obvious question is: how to do IO? A few options:

*   ask the firmware, e.g. BIOS or UEFI, to do it for us
    
*   VGA: special memory region that gets printed to the screen if written to. Can be used in Protected Mode.
    
*   write a driver and talk directly to the display hardware. This is the "proper" way to do it: more powerful, but more complex.
    
*   [serial port](https://en.wikipedia.org/wiki/Serial_port). This is a very simple standardized protocol that sends and receives characters from a host terminal.
    
    On desktops, it looks like this:
    
    [![](https://i.stack.imgur.com/z0kR7.jpg)](https://i.stack.imgur.com/z0kR7.jpg)
    
    [Source](https://unix.stackexchange.com/questions/307390/what-is-the-difference-between-ttys0-ttyusb0-and-ttyama0-in-linux/367882#367882).
    
    It is unfortunately not exposed on most modern laptops, but is the common way to go for development boards, see the ARM examples below.
    
    This is really a shame, since such interfaces are really useful [to debug the Linux kernel for example](https://askubuntu.com/questions/104771/where-are-kernel-panic-logs/932380#932380).
    
*   use debug features of chips. ARM calls theirs [semihosting](https://developer.arm.com/docs/dui0471/k/what-is-semihosting/what-is-semihosting) for example. On real hardware, it requires some extra hardware and software support, but on emulators it can be a free convenient option. [Example](https://stackoverflow.com/questions/31990487/how-to-cleanly-exit-qemu-after-executing-bare-metal-program-without-user-interve/40957928#40957928).
    

Here we will do a BIOS example as it is simpler on x86. But note that it is not the most robust method.

main.S

```
.code16
    mov $msg, %si
    mov $0x0e, %ah
loop:
    lodsb
    or %al, %al
    jz halt
    int $0x10
    jmp loop
halt:
    hlt
msg:
    .asciz "hello world"
```

[GitHub upstream](https://github.com/cirosantilli/x86-bare-metal-examples/blob/a2ca298afe704b0b9e94cd39edb924094034cc54/bios_hello_world.S).

link.ld

```
SECTIONS
{
    /* The BIOS loads the code from the disk to this location.
     * We must tell that to the linker so that it can properly
     * calculate the addresses of symbols we might jump to.
     */
    . = 0x7c00;
    .text :
    {
        __start = .;
        *(.text)
        /* Place the magic boot bytes at the end of the first 512 sector. */
        . = 0x1FE;
        SHORT(0xAA55)
    }
}
```

Assemble and link with:

```
as -g -o main.o main.S
ld --oformat binary -o main.img -T link.ld main.o
qemu-system-x86_64 -hda main.img
```

Outcome:

[![](https://i.stack.imgur.com/qRPUW.png)](https://i.stack.imgur.com/qRPUW.png)

And on the T430:

[![](https://i.stack.imgur.com/auien.jpg)](https://i.stack.imgur.com/auien.jpg)

Tested on: Lenovo Thinkpad T430, UEFI BIOS 1.16. Disk generated on an Ubuntu 18.04 host.

Besides the standard userland assembly instructions, we have:

*   `.code16`: tells GAS to output 16-bit code
    
*   `cli`: disable software interrupts. Those could make the processor start running again after the `hlt`
    
*   `int $0x10`: does a BIOS call. This is what prints the characters one by one.
    

The important link flags are:

*   `--oformat binary`: output raw binary assembly code, don't wrap it inside an ELF file as is the case for regular userland executables.

To better understand the linker script part, familiarize yourself with the relocation step of linking: [What do linkers do?](https://stackoverflow.com/questions/3322911/what-do-linkers-do/33690144#33690144)

**Cooler x86 bare metal programs**

Here are a few more complex bare metal setups that I've achieved:

*   multicore: [What does multicore assembly language look like?](https://stackoverflow.com/questions/980999/what-does-multicore-assembly-language-look-like/33651438#33651438)
*   paging: [How does x86 paging work?](https://stackoverflow.com/questions/18431261/how-does-x86-paging-work/18431262#18431262)

**Use C instead of assembly**

Summary: use GRUB multiboot, which will solve a lot of annoying problems you never thought about. See the section below.

The main difficulty on x86 is that the BIOS only loads 512 bytes from the disk to memory, and you are likely to blow up those 512 bytes when using C!

To solve that, we can use a [two-stage bootloader](https://wiki.osdev.org/Bootloader#Two-Stage_Bootloader). This makes further BIOS calls, which load more bytes from the disk into memory. Here is a minimal stage 2 assembly example from scratch using the [int 0x13 BIOS calls](https://github.com/cirosantilli/x86-bare-metal-examples/blob/893fdba02c795917c40de17d77220db0115f23fa/bios_disk_load.S):

Alternatively:

*   if you only need it to work in QEMU but not real hardware, use the `-kernel` option, which loads an entire ELF file into memory. [Here is an ARM example I've created with that method](https://github.com/cirosantilli/linux-kernel-module-cheat/tree/54e15e04338c0fecc0be139a0da2d0d972c21419#baremetal-setup-getting-started).
*   for the Raspberry Pi, the default firmware takes care of the image loading for us from an ELF file named `kernel7.img`, much like QEMU `-kernel` does.

For educational purposes only, here is a [one stage minimal C example](https://github.com/cirosantilli/x86-bare-metal-examples/tree/b4e4c124a3c3c329dcf09a5697237ed3b216a318#c-hello-world):

main.c

```
void main(void) {
    int i;
    char s[] = {'h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd'};
    for (i = 0; i < sizeof(s); ++i) {
        __asm__ (
            "int $0x10" : : "a" ((0x0e << 8) | s[i])
        );
    }
    while (1) {
        __asm__ ("hlt");
    };
}
```

entry.S

```
.code16
.text
.global mystart
mystart:
    ljmp $0, $.setcs
.setcs:
    xor %ax, %ax
    mov %ax, %ds
    mov %ax, %es
    mov %ax, %ss
    mov $__stack_top, %esp
    cld
    call main
```

linker.ld

```
ENTRY(mystart)
SECTIONS
{
  . = 0x7c00;
  .text : {
    entry.o(.text)
    *(.text)
    *(.data)
    *(.rodata)
    __bss_start = .;
    /* COMMON vs BSS: https://stackoverflow.com/questions/16835716/bss-vs-common-what-goes-where */
    *(.bss)
    *(COMMON)
    __bss_end = .;
  }
  /* https://stackoverflow.com/questions/53584666/why-does-gnu-ld-include-a-section-that-does-not-appear-in-the-linker-script */
  .sig : AT(ADDR(.text) + 512 - 2)
  {
      SHORT(0xaa55);
  }
  /DISCARD/ : {
    *(.eh_frame)
  }
  __stack_bottom = .;
  . = . + 0x1000;
  __stack_top = .;
}
```

run

```
set -eux
as -ggdb3 --32 -o entry.o entry.S
gcc -c -ggdb3 -m16 -ffreestanding -fno-PIE -nostartfiles -nostdlib -o main.o -std=c99 main.c
ld -m elf_i386 -o main.elf -T linker.ld entry.o main.o
objcopy -O binary main.elf main.img
qemu-system-x86_64 -drive file=main.img,format=raw
```

**C standard library**

Things get more fun if you also want to use the C standard library however, since we don't have the Linux kernel, which implements much of the C standard library functionality [through POSIX](https://stackoverflow.com/questions/1780599/what-is-the-meaning-of-posix/31865755#31865755).

A few possibilities, without going to a full-blown OS like Linux, include:

*   Write your own. It's just a bunch of headers and C files in the end, right? Right??
    
*   [Newlib](https://en.wikipedia.org/wiki/Newlib)
    
    Detailed example at: [https://electronics.stackexchange.com/questions/223929/c-standard-libraries-on-bare-metal/223931](https://electronics.stackexchange.com/questions/223929/c-standard-libraries-on-bare-metal/223931)
    
    Newlib implements all the boring non-OS specific things for you, e.g. `memcmp`, `memcpy`, etc.
    
    Then, it provides some stubs for you to implement the syscalls that you need yourself.
    
    For example, we can implement `exit()` on ARM through semihosting with:
    
    ```
    void _exit(int status) {
        __asm__ __volatile__ ("mov r0, #0x18; ldr r1, =#0x20026; svc 0x00123456");
    }
    ```
    
    as shown at [in this example](https://github.com/cirosantilli/linux-kernel-module-cheat/blob/54e15e04338c0fecc0be139a0da2d0d972c21419/baremetal/lib/syscalls.c#L73).
    
    For example, you could redirect `printf` to the UART or ARM systems, or implement `exit()` with [semihosting](https://stackoverflow.com/questions/31990487/how-to-cleanly-exit-qemu-after-executing-bare-metal-program-without-user-interve/49930361#49930361).
    
*   embedded operating systems like [FreeRTOS](https://en.wikipedia.org/wiki/FreeRTOS) and [Zephyr](https://en.wikipedia.org/wiki/Zephyr_(operating_system)).
    
    Such operating systems typically allow you to turn off pre-emptive scheduling, therefore giving you full control over the runtime of the program.
    
    They can be seen as a sort of pre-implemented Newlib.
    

**GNU GRUB Multiboot**

Boot sectors are simple, but they are not very convenient:

*   you can only have one OS per disk
*   the load code has to be really small and fit into 512 bytes
*   you have to do a lot of startup yourself, like moving into protected mode

It is for those reasons that [GNU GRUB](https://en.wikipedia.org/wiki/GNU_GRUB) created a more convenient file format called multiboot.

Minimal working example: [https://github.com/cirosantilli/x86-bare-metal-examples/tree/d217b180be4220a0b4a453f31275d38e697a99e0/multiboot/hello-world](https://github.com/cirosantilli/x86-bare-metal-examples/tree/d217b180be4220a0b4a453f31275d38e697a99e0/multiboot/hello-world)

I also use it on my [GitHub examples repo](https://github.com/cirosantilli/x86-bare-metal-examples/tree/a2ca298afe704b0b9e94cd39edb924094034cc54#getting-started-with-the-big-image) to be able to easily run all examples on real hardware without burning the USB a million times.

QEMU outcome:

[![](https://i.stack.imgur.com/zWfWl.png)](https://i.stack.imgur.com/zWfWl.png)

T430:

[![](https://i.stack.imgur.com/q7Bfx.jpg)](https://i.stack.imgur.com/q7Bfx.jpg)

If you prepare your OS as a multiboot file, GRUB is then able to find it inside a regular filesystem.

This is what most distros do, putting OS images under `/boot`.

Multiboot files are basically an ELF file with a special header. They are specified by GRUB at: [https://www.gnu.org/software/grub/manual/multiboot/multiboot.html](https://www.gnu.org/software/grub/manual/multiboot/multiboot.html)

You can turn a multiboot file into a bootable disk with `grub-mkrescue`.

**Firmware**

In truth, your boot sector is not the first software that runs on the system's CPU.

What actually runs first is the so-called _firmware_, which is a software:

*   made by the hardware manufacturers
*   typically closed source but likely C-based
*   stored in read-only memory, and therefore harder / impossible to modify without the vendor's consent.

Well known firmwares include:

*   [BIOS](https://en.wikipedia.org/wiki/BIOS): old all-present x86 firmware. SeaBIOS is the default open source implementation used by QEMU.
*   [UEFI](http://wiki.osdev.org/UEFI): BIOS successor, better standardized, but more capable, and incredibly bloated.
*   [Coreboot](https://en.wikipedia.org/wiki/Coreboot): the noble cross arch open source attempt

The firmware does things like:

*   loop over each hard disk, USB, network, etc. until you find something bootable.
    
    When we run QEMU, `-hda` says that `main.img` is a hard disk connected to the hardware, and `hda` is the first one to be tried, and it is used.
    
*   load the first 512 bytes to RAM memory address `0x7c00`, put the CPU's RIP there, and let it run
    
*   show things like the boot menu or BIOS print calls on the display
    

Firmware offers OS-like functionality on which most OS-es depend. E.g. a Python subset has been ported to run on BIOS / UEFI: [https://www.youtube.com/watch?v=bYQ_lq5dcvM](https://www.youtube.com/watch?v=bYQ_lq5dcvM)

It can be argued that firmwares are indistinguishable from OSes, and that firmware is the only "true" bare metal programming one can do.

As this [CoreOS dev puts it](https://lennartb.home.xs4all.nl/coreboot/col3.html):

> The hard part
> 
> When you power up a PC, the chips that make up the chipset (northbridge, southbridge and SuperIO) are not yet initialized properly. Even though the BIOS ROM is as far removed from the CPU as it could be, this is accessible by the CPU, because it has to be, otherwise the CPU would have no instructions to execute. This does not mean that BIOS ROM is completely mapped, usually not. But just enough is mapped to get the boot process going. Any other devices, just forget it.
> 
> When you run Coreboot under QEMU, you can experiment with the higher layers of Coreboot and with payloads, but QEMU offers little opportunity to experiment with the low level startup code. For one thing, RAM just works right from the start.

**Post BIOS initial state**

Like many things in hardware, standardization is weak, and one of the things you should _not_ rely on is the initial state of registers when your code starts running after BIOS.

So do yourself a favor and use some initialization code like the following: [https://stackoverflow.com/a/32509555/895245](https://stackoverflow.com/a/32509555/895245)

Registers like `%ds` and `%es` have important side effects, so you should zero them out even if you are not using them explicitly.

Note that some emulators are nicer than real hardware and give you a nice initial state. Then when you go run on real hardware, everything breaks.

**El Torito**

Format that can be burnt to CDs: [https://en.wikipedia.org/wiki/El_Torito_%28CD-ROM_standard%29](https://en.wikipedia.org/wiki/El_Torito_%28CD-ROM_standard%29)

It is also possible to produce a hybrid image that works on either ISO or USB. This is can be done with `grub-mkrescue` ([example](https://github.com/cirosantilli/x86-bare-metal-examples/blob/d217b180be4220a0b4a453f31275d38e697a99e0/multiboot/hello-world/Makefile#L6)), and is also done by the Linux kernel on `make isoimage` using `isohybrid`.

**ARM**

In ARM, the general ideas are the same.

There is no widely available semi-standardized pre-installed firmware like BIOS for us to use for the IO, so the two simplest types of IO that we can do are:

*   serial, which is widely available on devboards
*   blink the LED

I have uploaded:

*   a few simple QEMU C + Newlib and raw assembly examples [here on GitHub](https://github.com/cirosantilli/linux-kernel-module-cheat/tree/54e15e04338c0fecc0be139a0da2d0d972c21419#baremetal-setup-getting-started).
    
    The [prompt.c example](https://github.com/cirosantilli/linux-kernel-module-cheat/blob/54e15e04338c0fecc0be139a0da2d0d972c21419/baremetal/interactive/prompt.c) for example takes input from your host terminal and gives back output all through the simulated UART:
    
    ```
    enter a character
    got: a
    new alloc of 1 bytes at address 0x0x4000a1c0
    enter a character
    got: b
    new alloc of 2 bytes at address 0x0x4000a1c0
    enter a character
    ```
    
    See also: [How to make bare metal ARM programs and run them on QEMU?](https://stackoverflow.com/questions/38914019/how-to-make-bare-metal-arm-programs-and-run-them-on-qemu/50981397#50981397)
    
*   a fully automated Raspberry Pi blinker setup at: [https://github.com/cirosantilli/raspberry-pi-bare-metal-blinker](https://github.com/cirosantilli/raspberry-pi-bare-metal-blinker)
    
    [![](https://i.stack.imgur.com/eLiee.gif)](https://i.stack.imgur.com/eLiee.gif)
    
    See also: [How to run a C program with no OS on the Raspberry Pi?](https://stackoverflow.com/questions/29837892/how-to-run-a-c-program-with-no-os-on-the-raspberry-pi/40063032#40063032)
    
    To "see" the LEDs on QEMU you have to compile QEMU from source with a debug flag: [https://raspberrypi.stackexchange.com/questions/56373/is-it-possible-to-get-the-state-of-the-leds-and-gpios-in-a-qemu-emulation-like-t](https://raspberrypi.stackexchange.com/questions/56373/is-it-possible-to-get-the-state-of-the-leds-and-gpios-in-a-qemu-emulation-like-t)
    
    Next, you should try a UART hello world. You can start from the blinker example, and replace the kernel with this one: [https://github.com/dwelch67/raspberrypi/tree/bce377230c2cdd8ff1e40919fdedbc2533ef5a00/uart01](https://github.com/dwelch67/raspberrypi/tree/bce377230c2cdd8ff1e40919fdedbc2533ef5a00/uart01)
    
    First get the UART working with Raspbian as I've explained at: [https://raspberrypi.stackexchange.com/questions/38/prepare-for-ssh-without-a-screen/54394#54394](https://raspberrypi.stackexchange.com/questions/38/prepare-for-ssh-without-a-screen/54394#54394) It will look something like this:
    
    [![](https://i.stack.imgur.com/L0XyU.jpg)](https://i.stack.imgur.com/L0XyU.jpg)
    
    Make sure to use the right pins, or else you can burn your UART to USB converter, I've done it twice already by short circuiting ground and 5V...
    
    Finally connect to the serial from the host with:
    
    ```
    screen /dev/ttyUSB0 115200
    ```
    
    For the Raspberry Pi, we use a Micro SD card instead of an USB stick to contain our executable, for which you normally need an adapter to connect to your computer:
    
    [![](https://i.stack.imgur.com/ZbUFX.jpg)](https://i.stack.imgur.com/ZbUFX.jpg)
    
    Don't forget to unlock the SD adapter as shown at: [https://askubuntu.com/questions/213889/microsd-card-is-set-to-read-only-state-how-can-i-write-data-on-it/814585#814585](https://askubuntu.com/questions/213889/microsd-card-is-set-to-read-only-state-how-can-i-write-data-on-it/814585#814585)
    
    [https://github.com/dwelch67/raspberrypi](https://github.com/dwelch67/raspberrypi) looks like the most popular bare metal Raspberry Pi tutorial available today.
    

Some differences from x86 include:

*   IO is done by writing to magic addresses directly, there is no `in` and `out` instructions.
    
    This is called [memory mapped IO](https://stackoverflow.com/questions/3890484/what-is-the-difference-between-memory-mapped-io-and-io-mapped-io).
    
*   for some real hardware, like the Raspberry Pi, you can add the firmware (BIOS) yourself to the disk image.
    
    That is a good thing, as it makes updating that firmware more transparent.
    

**Resources**

*   [http://wiki.osdev.org](http://wiki.osdev.org) is a _great_ source for those matters.
*   [https://github.com/scanlime/metalkit](https://github.com/scanlime/metalkit) is a more automated / general bare metal compilation system, that provides a tiny custom API



Operating System as the inspiration
-----------------------------------

**The operating system is also a program**, so we can also **create our own program by creating from scratch or changing** (limiting or adding) features of one of the **small operating systems**, and then **run it during the boot process** (using an **ISO image**).

For example, this page can be used as a starting point:

[How to write a simple operating system](http://mikeos.sourceforge.net/write-your-own-os.html)

Here, the **entire Operating System fit entirely in a 512-byte boot sector ([MBR](https://en.wikipedia.org/wiki/Master_boot_record))!**

Such or similar simple OS can be used to **create a simple framework that will allow us:**

> make **the bootloader load subsequent sectors on the disk into RAM, and jump to that point to continue execution**. Or you could **read up on FAT12, the filesystem used on floppy drives, and implement that**.

**There are many possibilities, however.** For for example to see a **bigger x86 assembly language OS** we can explore the [MykeOS](http://mikeos.sourceforge.net/), x86 operating system which is a **learning tool** to show the simple 16-bit, real-mode OSes work, with **well-commented code** and **extensive documentation**.

Boot Loader as the inspiration
------------------------------

Other common type of **programs that run without the operating system are also Boot Loaders**. We can create a program inspired by such a concept for example using this site:

[How to develop your own Boot Loader](https://www.codeproject.com/articles/36907/how-to-develop-your-own-boot-loader)

The above article presents also the **basic architecture of such a programs**:

> 1.  **Correct loading to the memory by 0000:7C00 address.**
> 2.  **Calling the BootMain function** that is developed in the high-level language.
> 3.  Show “”Hello, world…”, from low-level” message on the display.

As we can see, **this architecture is very flexible and allows us to implement any program**, not necessarily a boot loader.

In particular, it shows how to use the **"mixed code" technique** thanks to which it is possible **to combine high-level constructions** (from **C** or **C++**) **with low-level commands** (from **Assembler**). This is a very useful method, but we have to remember that:

> **to build the program and obtain executable file** you will need **the compiler and linker of Assembler for 16-bit mode**. **For C/C++** you will need only the **compiler that can create object files for 16-bit mode**.

The article shows also how to see the created program in action and how to perform its testing and debug.

UEFI applications as the inspiration
------------------------------------

The above examples used the fact of loading the sector MBR on the data medium. **However, we can go deeper into the depths** by plaing for example with the **[UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) applications**:

> Beyond loading an OS, UEFI can run UEFI applications, which reside as files on the EFI System Partition. They can be executed from the UEFI command shell, by the firmware's boot manager, or by other UEFI applications. **UEFI applications can be developed and installed independently of the system manufacturer.**
> 
> A type of UEFI application is an **OS loader** such as GRUB, rEFInd, Gummiboot, and Windows Boot Manager; which loads an OS file into memory and executes it. Also, an OS loader can provide a user interface to allow the selection of another UEFI application to run. **Utilities like the UEFI shell are also UEFI applications.**

If we would like **to start creating such programs**, we can, for example, start with these websites:

[Programming for EFI: Creating a "Hello, World" Program](https://www.rodsbooks.com/efi-programming/hello.html) / [UEFI Programming - First Steps](http://x86asm.net/articles/uefi-programming-first-steps/)

Exploring security issues as the inspiration
--------------------------------------------

It is well known that there is a whole group of **malicious software** (which are programs) **that are running before the operating system starts**.

A huge group of them operate on the MBR sector or UEFI applications, just like the all above solutions, but there are also those that use another entry point such as the [Volume Boot Record](https://en.wikipedia.org/wiki/Volume_boot_record) (VBR) or the [BIOS](https://en.wikipedia.org/wiki/BIOS):

> **There are at least four known BIOS attack viruses**, two of which were for demonstration purposes.

or perhaps another one too.

[Attacks before system startup](https://securelist.com/attacks-before-system-startup/63725/)

> **Bootkits** have evolved from Proof-of-Concept development to mass distribution and **have now effectively become open-source software**.

Different ways to boot
----------------------

I also think that in this context it is also worth mentioning that **there are [various forms of booting](https://en.wikipedia.org/wiki/Booting) the operating system (or the executable program intended for this)**. There are many, but I would like to pay attention **to loading the code from the network** using Network Boot option (_PXE_), which allows us to run the program on the computer **regardless of its operating system and even regardless of any storage medium** that is directly connected to the computer:

[What Is Network Booting (PXE) and How Can You Use It?](https://www.howtogeek.com/63226/how-to-setup-network-bootable-utility-discs-using-pxe/)

![](https://www.gravatar.com/avatar/a007be5a61f6aa8f3e85ae2fc18dd66e?s=64&d=identicon&r=PG)Kissiel

I wrote a c++ program based on Win32 to write an assembly to the boot sector of a pen-drive. When the computer is booted from the pen-drive it executes the code successfully - have a look here [C++ Program to write to the boot sector of a USB Pendrive](https://blog-hoven.appspot.com/cpp-projects/write-boot-sector-pendrive-cpp.html)

This program is a few lines that should be compiled on a compiler with windows compilation configured - such as a visual studio compiler - any available version.