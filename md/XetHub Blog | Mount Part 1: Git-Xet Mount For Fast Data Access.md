> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [about.xethub.com](https://about.xethub.com/blog/mount-part-1)

> Scaling Git to support very large datasets bumps up against a big problem: I do not want to download ......

Scaling Git to support very large datasets bumps up against a big problem: I do not want to download the entire repository just to do anything with it. While lazy “no-smudge” checkouts + selective use of git xet checkout works, it is a painful and unnatural way to work with a repository.

Instead, we believe that a file system mount provides the easiest way to explore a large dataset without needing to download all of it. Watch this one minute video to see mount in action.

Try our git xet mount command yourself with the 4.2 GB Flickr30k dataset right now by installing the client and running:

```
  git xet mount https://xethub.com/XetHub/Flickr30k.git



```

This will create a directory called Flickr30k that you can explore within seconds! You do not even need to create a XetHub account to mount public repos. Just [install the client](https://xethub.com/assets/docs/getting-started/install#git-xet-installation)[](https://xethub.com/assets/docs/getting-started/installation).

Once mounted, you can use any local tools to read the files. When you are done, you can simply unmount with:

Note that you will need to close everything that is accessing the folder for the unmount to succeed. You can also force unmount with umount -f, or by ejecting the mount from the UI.

‍

How does it work?
-----------------

If you are on Mac OS, you might have noticed that unlike every other virtual filesystem implementation out there (S3Fuse or SSHFuse for instance), we do not require the installation of the MacFuse Kernel driver. So how on earth are we making a virtual filesystem?

The hint is that we are depending on an almost 30 year old piece of technology that now has an implementation embedded in almost every operating system.

### **NFSv3.**

Specifically, we wrote a custom NFSv3 server implementation in Rust (~2.5 KLOC for the read-only subset of the protocol). It was implemented by basically following [RFC 1813](https://www.rfc-editor.org/rfc/rfc1813) closely (and cross referencing RFC 1057, RFC 1014 and RFC 5531).

To create a virtual filesystem, we start a daemonized localhost NFSv3 server that serves the repository contents, and ask the operating system to mount that into the target directory.

### **Advantages**

NFS also confers several advantages over Fuse.

1.  Robustness: The NFS implementations have been hardened over time and are highly resilient to errors. This was very useful in testing and debugging. Perhaps FUSE might be more stable today, but when I was toying with FUSE years ago, it was easy to get a mount into a really bad state when the server process dies abruptly.
2.  Cache-aware: The NFS protocol is designed with OS caching in mind allowing for very simple cache management.
3.  OS Support: Pretty much every operating system has an NFS client implementation. We do not have to need a MacFuse driver on MacOS, or WinFuse on Windows. Nor do we have to support a variety of Fuse versions on Linux.
4.  Simplicity: Now having implemented an NFS Server as well as a FUSE Server, I found the NFSv3 API to be significantly easier to work with compared to the FUSE Low Level API. (Apart from the NFSPROC3_READDIRPLUS method which is rather annoying).

### **Performance**

The final and the most important question is that of performance. Is the NFS server fast enough? We have not ran any formal benchmarks, but for bulk sequential access we are comfortably in the couple gigabytes per second range. We have not performed much optimization work apart from rudimentary tuning of mount parameters and there is certainly a lot of room for improvement.

‍

**Rust Reflections**
--------------------

I have worked with C++ for 20 years, having worked on high performance distributed messaging systems and having spent countless days designing and redesigning synchronization models: Where do I use locks? What kind of locks? Can I make this lock-free? Making a synchronous messaging implementation (one request at a time) to a concurrent implementation (multiple requests simultaneously) is **supposed** to be hard.

My initial NFS server implementation was synchronous (the RFC was slightly ambiguous). Converting it to support concurrency required < 100 LOC and no thinking about safety, locking or synchronization.

In pseudocode, the change is from:

```
    loop {
    
        # simple message loop. 
        # read a message, handle it and reply
        message = read_message(socket)
        reply = handle_message(message)
        write_reply(reply)
    }



```

To:

```
    sender, receiver = new MPSCQueue()
    loop {
       # Executes whichever is ready.
       # In this case, either a message is read into message
       # or there is something in the MPSC Queue
       select! { 
         read_message(socket) into message {
            Spawn a new task to do the following {
                # instead of just sending the reply immediately
                # we write it into the MPSCqueue for the main loop 
                # to pick up
                reply = handle_message(message)
                sender.send(reply)
            }
         }
         # If there is a reply in the queue to pick up
         receiver into reply {
           write_reply(reply)
         }
       }
    }



```

Rust not only makes it easy to implement this main loop, but also provides confidence that the handle_message method is safe to call in parallel.

While I have no doubts that I can make a C++ implementation that is faster, it would have needed a significantly more elaborate design and I would have to work through all the threading details right from the start. Rust allowed me to smoothly convert an initial toy implementation to something pretty performant with very little additional effort. And most importantly, if I _really_ want to, I can still pull out all the stops with unsafe and implement bits in C-style to manage all the safety issues myself.

Is Rust perfect? Absolutely not. But no language is perfect. I believe that as an alternative to C++, Rust definitely hits a sweet spot and should be considered for any systems project.

In Part 2 of this series, we will show an example of why mount is so useful.

‍