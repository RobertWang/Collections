> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [dev.to](https://dev.to/mokiat/proper-http-shutdown-in-go-3fji)

> Up to this day, I continue to come across code that has problems with shutting down an HTTP server.........

Up to this day, I continue to come across code that has problems with shutting down an HTTP server gracefully in Go. This is why I decided to write a post about this.

Background
----------

We should first talk about graceful HTTP shutdown and when this might be important. If you already have experience with horizontally scaled microservices and rolling updates then you can skip this section.

When you have multiple instances of your HTTP application deployed and you would like to update these instances with a newer version, in general, you want to do it in a way that avoids downtime or failed HTTP requests.

The most common practice is to do a rolling update. Simply put, you start a new instance with the new version of the application and you configure your ingress to include this new instance when routing HTTP requests. Next, you shutdown one of the old instances. But before you can shut it down, you first need to configure your ingress to stop routing HTTP requests to that particular instance. You repeat this process until all old instances are replaced by new ones.

_Side note: The actual algorithms for rolling update might differ a bit from what I explained above, depending on the platform you are using, but the general idea is the same._

If you are using some type of PaaS or IaaS infrastructure (e.g. Cloud Foundry or Kubernetes) that whole process is somewhat automated or at least made easy to implement.

However, there is one important aspect to consider. While you may have configured new requests to no longer be routed to a particular instance you plan to shut down, you may have active connections still in progress. If you were to shut the instance down, connected clients would get `connection reset` errors or similar. If those calls originate from a browser (e.g. the endpoints are meant to serve front-end requests and not microservice-to-microservice requests) and there isn’t some retry mechanism in place, it will result in poor user experience for the customers whenever you perform updates.

Usually, when a platform shutdowns your instance, it sends a `SIGTERM` or `SIGINT` signal to inform your application that it is time to shutdown and it is up to your application to ensure that all connections have completed processing before exiting. And this is where gracefully shutting down your HTTP server comes to play. It ensures that connections are properly drained.

Problematic Implementations
---------------------------

Now that we understand why graceful shutdown is important, let us explore some of the most common Go HTTP server implementations and how they fail at proper HTTP shutdown.

### The Hello World approach

This is probably the most common code you would come across when getting started with Go and HTTP.  

```
package main

import "net/http"

func main() {
    http.Handle("/", http.FileServer(http.Dir("./public")))
    http.ListenAndServe(":8080", nil)
}


```

To be honest, for a Hello World it is probably fine. The problem is that it creates a lot of wrong assumptions for new developers that can be hard to unlearn.

The first problem is that `http.ListenAndServe` returns an error. So one should probably handle it. We get to the following code.  

```
package main

import (
    "log"
    "net/http"
)

func main() {
    http.Handle("/", http.FileServer(http.Dir("./public")))
    if err := http.ListenAndServe(":8080", nil); err != nil {
        log.Fatalf("HTTP server error: %v", err)
    }
}


```

Better, right? Well, wrong. It turns out that when `http.ListenAndServe` returns normally (note that it is a blocking call), it actually returns an `http.ErrServerClosed` error.

_Side note: My personal opinion is that this was a mistake on the Go team’s side. I see no reason why returning_ `nil` wouldn’t have been better and more properly aligned with Go convetions. If anyone knows the answer to this, please write a comment.

So we make another iteration and fix the problem from above.  

```
package main

import (
    "errors"
    "log"
    "net/http"
)

func main() {
    http.Handle("/", http.FileServer(http.Dir("./public")))
    if err := http.ListenAndServe(":8080", nil); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server error: %v", err)
    }
}


```

We have even decided to be fancy and have used the `errors.Is` function to compare the error. Surely this should be enough.

Unfortunately, it isn’t.

See, as I mentioned in the previous section, when the application is being stopped it gets sent a `SIGINT` (`CTRL+C` on some platforms) or `SIGTERM` signal.

If you check the documentation of the [signal](https://pkg.go.dev/os/signal) package, you will see that the default behavior of a Go program when it receives one of these two signals is to exit. What this means is that the program is abruptly stopped. It never gets to return from the `http.ListenAndServe` call and to perform the error check. It is as though `os.Exit` was called.

Let’s extend the code above with the following log statements to see what happens.  

```
package main

import (
    "errors"
    "log"
    "net/http"
)

func main() {
    log.Println("Starting...")
    http.Handle("/", http.FileServer(http.Dir("./public")))
    if err := http.ListenAndServe(":8080", nil); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server error: %v", err)
    }
    log.Println("Stopped.")
}


```

If we run this code, followed by `CTRL+C` in the terminal, we get the following output.  

```
$ go build -o experiment .; ./experiment
2022/01/14 00:19:51 Starting...
^C
$ echo $?
130


```

_Side note: I am using_ `go build` instead of `go run` since `go run` always returns an exit code equal to `1`, even when the app is properly written.

As you can see, we never got the `Stopped.` log statement. Instead, `signal: interrupt` was printed and checking the exit code of the program shows that it exited with `130` (non-zero) exit code.

### The signal handling approach

After some digging around, we realize that in order for our Go application not to exit so abruptly, we need to handle incoming signals. We soon end up using the [signal](https://pkg.go.dev/os/signal) package. We also find out that we need to create a dedicated `http.Server` instance, since there is no way to tell `http.ListenAndServe` to unblock. We end up with the following code.  

```
package main

import (
    "errors"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
)

func main() {
    server := &http.Server{
        Addr: ":8080",
    }

    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan

        if err := server.Close(); err != nil {
            log.Fatalf("HTTP close error: %v", err)
        }
    }()

    http.Handle("/", http.FileServer(http.Dir("./public")))
    if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server error: %v", err)
    }
}


```

What we have done is to spawn a goroutine that starts listening for signals and whenever a `SIGINT` or `SIGTERM` is received we close the server.

While one step closer to the truth, this code still does not achieve a graceful shutdown, since [Close](https://pkg.go.dev/net/http#Server.Close) immediatelly terminates all active connections without waiting for them to be processed.

We do some more reading and we change our implementation to use [Shutdown](https://pkg.go.dev/net/http#Server.Shutdown) with some timeout (through the usage of a timeout context).  

```
package main

import (
    "context"
    "errors"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    server := &http.Server{
        Addr: ":8080",
    }

    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan

        shutdownCtx, shutdownRelease := context.WithTimeout(context.Background(), 10*time.Second)
        defer shutdownRelease()

        if err := server.Shutdown(shutdownCtx); err != nil {
            log.Fatalf("HTTP shutdown error: %v", err)
        }
    }()

    http.Handle("/", http.FileServer(http.Dir("./public")))
    if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server error: %v", err)
    }
}


```

We run the program and we see that we no longer get any signal exit errors and we are using `Shutdown` so we should be ok to go. We deploy the application and feel happy with a work well done.

Except that, after some time has passed, and once we have applied this pattern to many of our microservice applications, we start to notice `connection reset` errors in our logging dashboard. We start troubleshooting what is going on and eventually we realize that our applications are not shutting down properly after all.

So what went wrong, we are using `Shutdown` after all, aren't we?

The problem is that a lot of developers don’t actually read the documentation thoroughly and end up in this pitfall. For those of you that know what I am talking about, the code above may seem silly and unlikely to occur, but I have seen it a number of times in practice. Sometimes it is not so trivial to spot, since there is some custom framework in place or the code has been split into multiple functions (maybe across files; often having channels and contexts involved).

If we read the documentation for [Shutdown](https://pkg.go.dev/net/http#Server.Shutdown) carefully, we will notice the following warning:

> When Shutdown is called, Serve, ListenAndServe, and ListenAndServeTLS immediately return ErrServerClosed. Make sure the program doesn’t exit and waits instead for Shutdown to return.

In our code above, we call `Shutdown` from within a goroutine. This immediately unblocks the `server.ListenAndServe` call in the `main` function, running on the main goroutine.

And the second problem is that Go has one very specific rule about the main function that is often forgotten or is overlooked by even more senior Go developers — If the main function returns, then the program is immediatelly terminated. All other goroutines are killed, without even running any `defer` statements.

So, the moment the `server.ListenAndServe` unblocks, the program exists and the `server.Shutdown` call never gets a chance to drain the connections and to properly release resources. We can verify this easily by adding some logging statements.  

```
package main

import (
    "context"
    "errors"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    server := &http.Server{
        Addr: ":8080",
    }

    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan

        shutdownCtx, shutdownRelease := context.WithTimeout(context.Background(), 10*time.Second)
        defer shutdownRelease()

        if err := server.Shutdown(shutdownCtx); err != nil {
            log.Fatalf("HTTP shutdown error: %v", err)
        }
        log.Println("Graceful shutdown complete.")
    }()

    http.Handle("/", http.FileServer(http.Dir("./public")))
    if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server error: %v", err)
    }
    log.Println("Stopped serving new connections.")
}


```

We get the following output:  

```
$ go run main.go
^C
2022/01/13 23:44:54 Stopped serving new connections.


```

We never get to see the `Graceful shutdown complete.` message.

### A working graceful shutdown

There is a very simple solution to the above problem. You can just swap the locations of the `Shutdown` and `ListenAndServe calls`, where the former can be called from the main function and the latter from the goroutine.  

```
package main

import (
    "context"
    "errors"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    server := &http.Server{
        Addr: ":8080",
    }

    http.Handle("/", http.FileServer(http.Dir("./public")))

    go func() {
        if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
            log.Fatalf("HTTP server error: %v", err)
        }
        log.Println("Stopped serving new connections.")
    }()

    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    <-sigChan

    shutdownCtx, shutdownRelease := context.WithTimeout(context.Background(), 10*time.Second)
    defer shutdownRelease()

    if err := server.Shutdown(shutdownCtx); err != nil {
        log.Fatalf("HTTP shutdown error: %v", err)
    }
    log.Println("Graceful shutdown complete.")
}


```

If we run this program and then stop it, we can see the following output.  

```
$ go run main.go
^C
2022/01/14 20:49:25 Stopped serving new connections.
2022/01/14 20:49:29 Graceful shutdown complete.


```

Finally, the outcome we wanted to achieve.

The example above can be further improved. You may decide to add a `server.Close` call inside the error branch of the `server.Shutdown` call. That way if the graceful shutdown is unable to complete within the specified timeout, you can still force the server to close.

Summary
-------

The main key points to take away from this post are as follows:

*   Always read the Go documentation for methods you are using. There are often hints to important corner cases or unexpected outcomes.
    
*   The `httpServer.ListenAndServe` method unblocks immediatelly when `httpServer.Shutdown` is called.
    
*   Make sure you never return from the main function until you are actually ready to quit. Either structure your code in such a way (as in the example above) or use synchronization primitives (e.g. wait groups, channels).
    

Final remarks
-------------

I hope that the examples above were useful to you and that this helps you avoid a common pitfall. I am sorry about the long post but I wanted this to be useful to both beginner and advanced Go developers.

I should mention that in a more complicated application it might not be possible to have the `Shutdown` call block on the main goroutine, since you may have multiple HTTP Servers that you want to run and stop concurrently. Most often you would use some type of framework for starting and stopping concurrent sub-processes (for a lack of a better term). Even so, make sure that all `Shutdown` calls have unblocked before you exit the main function. Use `WaitGroup`s or other synchronization mechanisms to your advantage.

Something to keep in mind when using Kubernetes is that even if your code is well written you may still end up with connection problems during rollout. The problem is that it takes time for Kubernetes to adjust its ingress routing and prevent new connections from reaching your instance (pod/container in this case) that is being stopped. In such cases, instead of `connection reset` errors you may observe `bad gateway` errors or similar, depending on your ingress implementation.

To solve this, you could use a [preStop](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#container-hooks) hook on your container to ensure that it is given a number of seconds between being detached (or rather, requested for detachment) from the ingress and receiving a `SIGTERM` signal.

I should also point out that in the microservice and cloud world there is a common understanding that cloud applications should be resilient to failure and as such they should be implemented in a way that they can handle abrupt application shutdowns.

While I agree with that and I strongly advice that you employ tactics like retrying failed requests, circuit breakers and the like, keep in mind that all of these mechanisms result in error logs, connections that might not be immediatelly cleaned up, extra processing overhead, message queues that need time to figure out what is going on and start resending messages to working instances, etc.

As such, my personal opinion is that applications should try and stop as gracefully as possible (properly releasing all resources and disconnecting from services) but should also have mechanisms in place to handle failures. After all, it happens that applications crash, VMs on which they are running disappear, the network starts acting out, and so on. If you really want to test your environment and ensure that it is resilient, you may consider using something like [Chaos Monkey](https://netflix.github.io/chaosmonkey/).

* * *

**Note:** This article has been ported from another platform as I gradually migrate here. Sorry if you have seen this before.