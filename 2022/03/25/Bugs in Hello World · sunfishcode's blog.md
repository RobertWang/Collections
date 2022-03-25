> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.sunfishcode.online](https://blog.sunfishcode.online/bugs-in-hello-world/)

> A blog by sunfishcode

Posted on March 08, 2022

Hello World might be the most frequently written computer program. For decades, it's been the first program many people write, when getting started in a new programming language.

Surely, this humble starting-point program should be bug free, right?

[![](https://www.monkeyuser.com/assets/images/2019/131-bug-free.png)](https://www.monkeyuser.com/2019/bug-free/)

After all, hello world programs only do one thing. How could there be a bug?

Hello world in C
----------------

There are a lot of different ways to write hello world in C. There's the [Wikipedia version](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program#C), [the hello world in the K&R book](https://riptutorial.com/c/example/3675/original--hello--world---in-k-r-c), and there's even [the oldest known C hello world program from 1974](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program#History).

[![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Hello_World_Brian_Kernighan_1978.jpg/512px-Hello_World_Brian_Kernighan_1978.jpg)](https://commons.wikimedia.org/wiki/File:Hello_World_Brian_Kernighan_1978.jpg "Brian Kernighan, CC BY-SA 3.0 <https://creativecommons.org/licenses/by-sa/3.0>, via Wikimedia Commons")

Here's another, this one [in "ANSI C"](http://helloworldcollection.de/#C%C2%A0(ANSI)):

```
/* Hello World in C, Ansi-style */

#include <stdio.h>
#include <stdlib.h>

int main(void)
{
  puts("Hello World!");
  return EXIT_SUCCESS;
}
```

This is the most careful version of the bunch. It uses `(void)` to ensure that `main` is a new-style declaration. It uses the `EXIT_SUCCESS` macro instead of just assuming that the platform uses 0 to indicate success, which isn't necessary, according to the C standard, but we're not taking any chances here. And it uses the appropriate headers to avoid implicitly declaring `puts`. This version attempts to do _everything_ right.

And yet, it still has a bug.

All the versions linked above have a bug.

A bug?
------

Linux has this fun device file called "/dev/full", which is like its more famous cousin "/dev/null", but when you write to "/dev/full", instead of throwing away the data, it fails. It acts like a file on a filesystem that has just run out of space:

```
$ echo "Hello World!" > /dev/full
bash: echo: write error: No space left on device
$ echo $?
```

This is a great little tool for testing that programs handle I/O errors correctly. It's inconvenient to create actual filesystems with no space left, or disks that actually fail, but it's really easy to ask a program to write its output to "/dev/full" and see what happens.

So let's test the C example above:

```
$ gcc hello.c -o hello
$ ./hello > /dev/full
$ echo $?
```

Unlike when we used `echo` in the shell above, here, we got no output, and the exit status was zero. That means the `hello` program reported successful execution. However, it didn't actually succeed. We can confirm that it encounters a failure using [strace](https://man7.org/linux/man-pages/man1/strace.1.html):

```
$ strace -etrace=write ./hello > /dev/full
write(1, "Hello World!\n", 13)          = -1 ENOSPC (No space left on device)
+++ exited with 0 +++
```

There's our"No space" error getting reported by the OS, but no matter, the program silently swallows it and returns 0, the code for success. That's a bug!

How severe is this bug? Arguably, hello world isn't going to be safety-critical anywhere. However, hello world does do something that programs in the real world do: print to standard output, which might be redirected to a file. And files in the real world can run out of space. If a program doesn't detect this kind of error and report it through its return code, its parent process won't know that the child failed, and will continue running as if nothing was wrong, even though the output it expected to have been produced has silently lost data.

For example, consider a program that prints a [yaml](https://yaml.org/) file to standard output. If standard output runs out of space, the output may be truncated at some arbitrary point, though it [may still be valid yaml](https://www.sqlservercentral.com/editorials/do-you-have-all-the-yaml). So we should expect programs to detect and report this kind of situation.

What about other languages?
---------------------------

We looked at bash and C above; what about Python, which tells us that ["Errors should never pass silently"](https://www.python.org/dev/peps/pep-0020/#id2)? Here's Python 2:

```
$ python2 hello.py > /dev/full
close failed in file object destructor:
sys.excepthook is missing
lost sys.stderr
$ echo $?
```

It did print a message to stderr, though it's a confusing message. However, it also returned 0, which means it's telling whoever ran it that it exited succeesfully.

Fortunately, Python 3 properly reports the error, and prints a nicer error message too:

```
$ python3 hello.py > /dev/full
Exception ignored in: <_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
OSError: [Errno 28] No space left on device
$ echo $?
```

Using hello world programs from common tutorial sites in a few languages that I happened to try, here are the results:

```
Yes
```

A more complete and current list is maintained [here](https://github.com/sunfishcode/hello-world-vs-io-errors).