> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blogsystem5.substack.com](https://blogsystem5.substack.com/p/ssh-agent-forwarding-and-tmux-done)

> The SSH agent is a little daemon that holds your private keys in memory. This is particularly handy w......

The [SSH agent](https://man.openbsd.org/ssh-agent) is a little daemon that holds your private keys in memory. This is particularly handy when your keys are protected by a passphrase: you can unlock and add your keys to the agent once and, from then on, any SSH client such as `ssh(1)` can interact with the keys without asking you for the passphrase again.

The SSH agent becomes even handier when you primarily work on a remote workstation over SSH. Under these circumstances, you will often need the remote workstation to establish SSH connections to _other_ remote machines (e.g. to contact GitHub). In those situations, you can: copy your private keys to the remote workstation; generate different private keys on the remote workstation; or forward your SSH agent so that the remote workstation can leverage the keys from your client machine without them ever traveling over the network.

Anyway. This article isn’t a tutorial on what the SSH agent _is_ or how to _use_ it; that’s already [well-documented elsewhere](https://www.ssh.com/academy/ssh/agent). What this article does is dig deeper into how the forwarding feature works, how it becomes problematic for long-lived processes such as tmux, and what you can do about it.

Agent forwarding 101


======================

When you connect to an SSH host, the `sshd` server process receives the connection, forks itself, lowers its privileges to the user requesting the connection, creates a pseudo-terminal to host the requested program (typically a shell), and spawns such program “inside” the pseudo-terminal.

Take a look at this process table, which shows the `sshd` processes running on the server I’m typing this on:

```
$ ps -ax -o user,pid,command | grep ssh[d]
root   3617 sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups (sshd)
root  82202 sshd: jmmv [priv] (sshd)
jmmv  82204 sshd: jmmv@pts/1 (sshd)
$ ps -ax -o user,pid,command | grep ssh[d]
root   3617 sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups (sshd)
root  82202 sshd: jmmv [priv] (sshd)
jmmv  82204 sshd: jmmv@pts/1 (sshd)

```

What you can see in the output above is: the listener process with PID 3617 accepting connections on port 22; a “privsep” controlling process owned by root for my session with PID 82202, [responsible for doing authentication](https://security.stackexchange.com/questions/115896/can-someone-explain-how-sshd-does-privilege-separation); and the process serving my session on the pseudo-terminal `pts/1` and owned by myself with PID 82204.

The user-owned `sshd` instance on PID 82204 is what hosts the shell that you interact with on the client when connecting to an SSH server. This process is in charge of receiving your keystrokes from the network, sending them to the pseudo-terminal, and then ferrying back the pseudo-terminal’s “output” to the terminal window where your client runs. Pseudo-terminals are fascinating by the way; go read [openpty(3)](https://man.netbsd.org/openpty.3).

But when you enable SSH agent forwarding with an `ssh -A` invocation, something else happens. The user-owned `sshd` server process creates a Unix domain socket on the remote machine and starts _serving_ on it. Whenever an SSH process on the remote machine connects to this local socket and sends a request to it, the `sshd` process proxies the request _back_ to the client machine and, in turn, the client machine’s `ssh` process contacts the client machine’s `ssh-agent` to perform key-related operations.

This path to the socket on the remote machine is exposed by the `SSH_AUTH_SOCK` environment variable, and all SSH commands know how to access it. Here, look:

```
client$ ssh -A server

server$ env | grep SSH_AUTH_SOCK
SSH_AUTH_SOCK=/tmp/ssh-jewlWIz1jC/agent.82204

server$ ls -l "${SSH_AUTH_SOCK}"
srwxr-xr-x  1 jmmv  wheel  0 Nov 17 05:30 /tmp/ssh-jewlWIz1jC/agent.82204
client$ ssh -A server

server$ env | grep SSH_AUTH_SOCK
SSH_AUTH_SOCK=/tmp/ssh-jewlWIz1jC/agent.82204

server$ ls -l "${SSH_AUTH_SOCK}"
srwxr-xr-x  1 jmmv  wheel  0 Nov 17 05:30 /tmp/ssh-jewlWIz1jC/agent.82204

```

Because a picture is worth a thousand words, here is the same idea in a diagram… and this picture starts hinting at the problem I want to analyze:

[![](https://substackcdn.com/image/fetch/w_2400,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe000e95e-952b-42a5-9261-42cbb6cbdce2_882x288.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe000e95e-952b-42a5-9261-42cbb6cbdce2_882x288.png)Representation of a client machine (left) connecting to an SSH server. The SSH server (large box on the right) contains an sshd process and a tmux instance. The tmux instance shows the value of SSH_AUTH_SOCK.

Pay close attention to what I described above: the path to the local socket created by the remote `sshd` instance is specific to the (unprivileged) `sshd` process handling the connection. The fact that the socket’s name contains the PID gives this away, but if you want to double-check:

```
server$ sudo lsof | grep "${SSH_AUTH_SOCK}"
sshd      82204       jmmv    7u     unix    0xfffff80f6a50fb10                0t0         /tmp/ssh-jewlWIz1jC/agent.82204
server$ sudo lsof | grep "${SSH_AUTH_SOCK}"
sshd      82204       jmmv    7u     unix    0xfffff80f6a50fb10                0t0         /tmp/ssh-jewlWIz1jC/agent.82204

```

I did not say it explicitly, but you can guess what happens: when you disconnect from the remote session, the `sshd` instance for the connection terminates and deletes its local socket on the way out. Which is normal because that process was the one in charge of proxying SSH agent connections back to the SSH client, so with the process gone, the socket becomes useless.

Unfortunately, the fact that this is normal doesn’t make it less problematic. The path to the forwarding socket is exposed to processes via the `SSH_AUTH_SOCK` environment variable, so every process started within the remote SSH session gets and _caches_ a copy of this path in its environment. If you have long-lived process such as `tmux` that can survive _across_ sessions, then all of a sudden the value of `SSH_AUTH_SOCK` that they cached at startup time ends up pointing to a non-existing file, breaking all future SSH agent interactions. Look:

[![](https://substackcdn.com/image/fetch/w_2400,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3c7aa68b-7751-4b1f-b0e2-9268924b87a6_882x472.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3c7aa68b-7751-4b1f-b0e2-9268924b87a6_882x472.png)Representation of a client machine (left) re-connecting to an SSH server after an earlier connection drops. The SSH server (large box on the right) shows how the tmux instance from the first connection "moves" to the second connection, and how the value that tmux displayed in SSH_AUTH_SOCK now points to an invalid file.

The specific sequence of events represented in the diagram above and that leads to the problem is this:

1.  Connect to an SSH server with SSH agent forwarding.
    
2.  Start a tmux session in the SSH server.
    
3.  Detach the tmux session.
    
4.  Log out of the SSH server.
    
5.  Reconnect to the SSH server with SSH agent forwarding.
    
6.  Attach to the existing tmux session.
    
7.  Run an SSH command.
    
8.  See the command fail to communicate with the forwarded agent.
    

So, what can we do about it?

Usual broken solutions


========================

[Most](https://blog.testdouble.com/posts/2016-11-18-reconciling-tmux-and-ssh-agent-forwarding/) [“solutions”](https://werat.dev/blog/happy-ssh-agent-forwarding/) [I](https://without-brains.net/2020/08/05/tmux-and-ssh-agent-forwarding/) [find](https://stackoverflow.com/questions/21378569/how-to-auto-update-ssh-agent-environment-variables-when-attaching-to-existing-tm) [online](https://www.jwon.me/ssh-agent-forwarding-with-tmux/) to this problem are hyper-focused on propagating new values of `SSH_AUTH_SOCK` to impacted processes. These articles present how to re-inject a fresh value of this variable into tmux. Some of them go to a greater extent by showing you how to propagate the variable’s value to the _shells_ already running inside tmux sessions, because obviously the shells outlast SSH sessions too and they _also_ have their own copy of `SSH_AUTH_SOCK`.

This mostly works but it has two problems:

*   It’s a pile of hacks and a whack-a-mole battle you will not win. You can patch tmux. You can patch the shell. But what else do you run inside tmux that outlives SSH sessions? Emacs? Vim? A “transitive” outbound SSH connection? These all have their own copies of `SSH_AUTH_SOCK`. You just can’t monkey-patch all possible long-lived processes that cached an ephemeral path in their environment when they started.
    
*   It’s insufficient. Some of the solutions linked to above leverage a symlink to keep the socket’s name stable. Those solutions refresh the symlink every time you log into the remote machine and then point `SSH_AUTH_SOCK` to the symlink. This is good because it solves the problem of having to monkey-patch all possible long-lived processes. Unfortunately, this breaks as soon as the _latest_ connection to the server disconnects: once that happens, the symlink will be dangling until you establish a new session to update it.
    

Can we do better?

Better working solutions


==========================

_Of course_ we can do better, and in various different ways actually. I’m surprised I haven’t found any of them online with ease…

Here are some ideas:

1.  We can refine the symlinks approach described earlier by adding a _logout_ script to accompany the login script. The goal of this pair would be to keep track of all possible valid sockets and, when a session disconnects, refresh the symlink to point to a still-valid one. This can work in the common case but breaks if SSH sessions die suddenly, which is pretty normal: think putting your laptop to sleep with an open SSH session and resuming it after commuting. So… this is better, but not awesome.
    
2.  We can use `LD_PRELOAD` to hijack calls that open a socket, check if they seem to refer to an SSH agent socket, and then redirect those to any _other_ alive SSH agent socket we can find on the machine. This possibly works well, but leveraging `LD_PRELOAD` is a pretty heavy thing to do because it impacts _all_ processes. Furthermore, this variable is pretty sensitive so it is probably cleared by many programs at startup time… which means it can be ineffective.
    
3.  We can supplant the SSH agent with our own daemon that serves a local socket on a well-known path. This daemon then redirects all SSH agent requests to a working SSH agent socket. The “working” socket is determined by searching `/tmp/` for _any_ agent socket that the calling user has permissions to open, and using the first one that works. We can start this daemon when the user firsts connects to the remote server, and we can fix up the `SSH_AUTH_SOCK` value at login time—well before any process can cache the wrong value.
    

This last solution is what I prototyped in **[ssh-agent-switcher](https://github.com/jmmv/ssh-agent-switcher/)**. It took me about an hour of evening coding to get this to work, an extra hour to write integration tests and refine the implementation, and a final half hour to set up the GitHub project. It works for me, but don’t expect it to be perfect.

As a side note, I wrote this program in Go… not because I wanted to, but because it is a necessity due to the context in which I’ll probably use this. And you know what, writing Go for this was really fun due to easy APIs, quick iteration, and simple tooling… But I could only tolerate Go because I had no need to model any kind of structured data: I still find Go to be a bad choice for anything larger in scope where correctness matters.

Now the question is: is this safe? And the answer is: I think so? The ssh-agent-switcher daemon runs in the context of your user account so it can only access sockets you already have access to. The only risk comes from the creation of the daemon’s own Unix socket: if the socket is accessible to anyone else on the remote machine, then that person would be able to access your forwarded agent and leverage the credentials that live on your client machine. I’ve taken care to create the socket with tight permissions, but the question remains: is the default location under `/tmp/` good-enough? Creating temporary files with predictable names [is generally unsound](https://www.netmeister.org/blog/mktemp.html) because that leaves them subject to hijacking via symlinks… but in this case, the daemon is creating a Unix domain socket, not a regular file, and I’ve not succeeded at stealing it.

Oh, and by the way, this all explains the warning in the [sshd(1) manual page](https://man.openbsd.org/ssh) about the use of agent forwarding:

> Agent forwarding should be enabled with caution. Users with the ability to bypass file permissions on the remote host (for the agent’s UNIX-domain socket) can access the local agent through the forwarded connection. An attacker cannot obtain key material from the agent, however they can perform operations on the keys that enable them to authenticate using the identities loaded into the agent. A safer alternative may be to use a jump host (see -J).

If you do not trust the administrator of a remote machine, SSHing into it and forwarding your agent will allow the remote administrator to send requests to your local agent. Be careful out there.