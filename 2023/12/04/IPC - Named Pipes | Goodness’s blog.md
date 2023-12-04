> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [goodyduru.github.io](https://goodyduru.github.io/os/2023/09/26/ipc-named-pipes.html)

> In the previous article, We introduced Inter-Process Communication and its different mechanisms. We’l......

In the previous article, We introduced Inter-Process Communication and its different mechanisms. We’ll start with the first, Named Pipes or FIFO file!

Named pipes are a mechanism that builds upon the structure of anonymous pipes. Your regular Unix pipes are actually Anonymous pipes. To understand named pipes, we need to understand anonymous pipes.

### Anonymous Pipes

Anonymous pipes are memory buffers created and maintained by the kernel. This buffer has two file descriptors for referencing it, one for reading and the other for writing. You can read and write data to this buffer with the `read` and `write` system calls with the proper descriptors. **Data written to this buffer do not appear on disk**.

Anonymous pipes are unidirectional, meaning you can only write to one end and read from one end. You can create an anonymous pipe using the [`pipe()`](https://man7.org/linux/man-pages/man2/pipe.2.html) function. Whenever you use the `|` symbol in your shell, it calls the `pipe()` function to create a buffer that your programs can use (along with some redirection tricks that we will not cover here).

One important fact about an anonymous pipe is that **data is erased from the buffer once it’s read**.

Imagine you want two processes to communicate without using the shell. How will you do that with anonymous pipes? The simple answer is that you can’t. When you call the `pipe()` function, the kernel creates the buffer that only the **caller process and all its child processes know about**. No other application process can reference that buffer. If you were to call the `pipe()` function in each application process, two separate pipe buffers would be created without any connection between them, defeating the purpose of your processes communicating. You might ask, how does the shell do it? Every process executed in your shell program is a child of that shell process. Remember what I said about how an anonymous pipe buffer created by a process can only be known by the creating application process and all its child processes :-). That’s how your shell can get separate application processes to communicate.

We’ve seen that as cool as anonymous pipes are, it has a limitation. That limitation is knowledge, not sharing. If one process were to know about another’s pipe buffer, it could read and write to that buffer. But that knowledge is hidden by the kernel. It’s isolation by obfuscation! What if we want other application processes to read our pipe buffer without sharing process ancestry? That’s what Named Pipes are for!

### Named Pipes

In computing, creating a reference is the first step in allowing access to data. That is what Named Pipes is, Anonymous pipes plus reference. This reference is simply a file name. This file name is stored on disk and will appear as a file in your Explorer or directory listing. We can test this out by running these commands in your shell.

```
mkfifo example-pipes
ls -l 


```

The [`mkfifo`](https://man7.org/linux/man-pages/man1/mkfifo.1.html) command[[1]](#footer-note-1) creates a named pipe called _example-pipes_; Your output should be similar to this:

```
prw-r--r--  1 user  group    0 Sep 21 16:36 example-pipes


```

It’s just a file! That’s neat. You will notice that the first column starts with `p`. That means its type is a FIFO file.

Because it’s a file, any process can interact with it like every other regular file. That means you can `open` it, `read` from it, `write` to it, `unlink` it, `close` it, and so on. A bonus about the reference being a file is that it can have permissions. What this means is you can restrict who has access to it.

The difference between a regular file and a FIFO file is that it can never contain data. That will violate the purpose of a pipe, which is to act as a communication buffer between processes. The only reason for the file name is to be a reference, nothing more!

One important detail about named pipes is no bytes are written to a named pipe buffer until there is at least one concurrent reader.

Named pipes can be bidirectional. An application process can read from it and write to it with a single file descriptor. Once an application process has the correct permissions to open a named pipe, all it has to do is open the named pipe using its name in `O_RDWR` mode. This is tricky because a process can immediately read what it has written, causing the written data not to be visible to other application processes.

### Show me the code

Our example will demonstrate two Python processes; a server and a client. The client will send a “ping” message to the server, and the server will print it out.

Here’s the client

```
    import os

    ROUNDS = 100

    def run():
        pipe_path = '/tmp/ping'
        fd = os.open(pipe_path, os.O_WRONLY)
        i = 0
        while i != ROUNDS:
            os.write(fd, b'ping')
            print("Client: Sent ping")
            i += 1
        os.write(fd, b'end')
        os.close(fd)


```

The client opens the named pipe referenced by the name `/tmp/ping` in write-only mode. It sends a “ping” message a hundred times to the server using the named pipe. It signifies that it’s finished by sending “end”. Once it’s finished sending, it closes the file. The write will block until at least one other process tries to read from the pipe.

Note that data is sent as a byte string, not as a regular string.

Here’s the server

```
    import os

    def run():
        pipe_path = '/tmp/ping'
        os.mkfifo(pipe_path)
        fd = os.open(pipe_path, os.O_RDONLY)
        data = os.read(fd, 4).decode()
        while data != 'end':
            print(f"Server: Received {data}")
            data = os.read(fd, 4).decode()
        os.close(fd)
        os.unlink(pipe_path)


```

Here, the [`mkfifo()`](https://man7.org/linux/man-pages/man3/mkfifo.3.html) function creates a named pipe called `/tmp/ping`. After creation, the server opens it in read-only mode. Data is read from the pipe buffer and printed to the console in a loop. The loop ends when the “end” message is received. The file is closed and deleted. Note that the read is blocked until there’s data in the pipe.

Note that data has to be decoded to a utf-8 string because it’s originally received as a byte string.

### Performance

Named pipes are pretty fast. [IPC-Bench](https://github.com/goldsborough/ipc-bench#benchmarked-on-intelr-coretm-i5-4590s-cpu--300ghz-running-ubuntu-20041-lts) benchmarked 254,880 1KB messages per second on an Intel(R) Core(TM) i5-4590S CPU @ 3.00GHz running Ubuntu 20.04.1 LTS. That’s fast enough to serve most processes’ communication needs.

### Demo Code

You can find my code that demonstrates unidirectional and bidirectional communication using Named Pipes on [GitHub](https://github.com/goodyduru/ipc-demos).

### Conclusion

Named Pipes is a simple and powerful IPC mechanism. As with all powerful tools, you have to use it with caution. I wouldn’t recommend using it for bi-directional communication unless you aren’t worried about losing data.

This article was filesystem-related. The next one will be networking-related :-). I’m referring to [Unix Domain Sockets](https://goodyduru.github.io/os/2023/10/03/ipc-unix-domain-sockets.html). Till then, take care of yourself and stay hydrated! ✌🏾

* * *