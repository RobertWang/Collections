> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/leokale-zz/p/11957768.html)

**下面我们使用 Python 来实现并发的 Web Server，其中采用了多进程、多线程、协程、单进程单线程非阻塞的方式。**

**一、使用子进程来实现并发 Web Server**
---------------------------

参照 [https://www.cnblogs.com/leokale-zz/p/11949208.html](https://www.cnblogs.com/leokale-zz/p/11949208.html) 中的代码，我们将其修改为支持并发的简单 Web Server：

```
import socket
import re
import multiprocessing


def handle_request(new_socket):
    # 接收请求
    recv_msg = ""
    recv_msg = new_socket.recv(1024).decode("utf-8")
    if recv_msg == "":
        print("recv null")
        new_socket.close()
        return

    # 从请求中解析出URI
    recv_lines = recv_msg.splitlines()
    print(recv_lines.__len__())
    # 使用正则表达式提取出URI
    ret = re.match(r"[^/]+(/[^ ]*)", recv_lines[0])
    if ret:
        # 获取URI字符串
        file_name = ret.group(1)
        # 如果URI是/，则默认返回index.html的内容
        if file_name == "/":
            file_name = "/index.html"

    try:
        # 根据请求的URI，读取相应的文件
        fp = open("." + file_name, "rb")
    except:
        # 找不到文件，响应404
        response_msg = "HTTP/1.1 404 NOT FOUND\r\n"
        response_msg += "\r\n"
        response_msg += "<h1>----file not found----</h1>"
        new_socket.send(response_msg.encode("utf-8"))
    else:
        html_content = fp.read()
        fp.close()
        # 响应正确 200 OK
        response_msg = "HTTP/1.1 200 OK\r\n"
        response_msg += "\r\n"

        # 返回响应头
        new_socket.send(response_msg.encode("utf-8"))
        # 返回响应体
        new_socket.send(html_content)

    # 关闭该次socket连接
    new_socket.close()


def main():
    # 创建TCP SOCKET实例
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # # 设置重用地址
    # tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定地址（默认本机IP）和端口
    tcp_server_socket.bind(("", 7890))
    # 监听
    tcp_server_socket.listen(128)
    # 循环接收客户端连接
    while True:
        new_socket, client_addr = tcp_server_socket.accept()
        # 启动一个子进程来处理客户端的请求
        sub_p = multiprocessing.Process(target=handle_request, args=(new_socket,))
        sub_p.start()
        # 这里要关闭父进程中的new_socket，因为创建子进程会复制一份new_socket给子进程
        new_socket.close()

    # 关闭整个SOCKET
    tcp_server_socket.close()


if __name__ == "__main__":
    main()
```

我们使用进程来实现并发的 Web Server，也就是将 Accept 到 new_socket 传递给子进程去处理，处理函数还是 handle_request。

但是这里注意，子进程会从父进程中将所有的变量**进行拷贝**，也就是说父进程和子进程中**各有一份 new_socket**，而在 Linux 下，socket 对应的也是一个文件描述符，而这两个 new_socket 实际上是指向同一个 fd 的。所以我们将 new_socket 交给子进程后，父进程就可以马上关闭自己的 new_socket 了，当子进程服务完毕后，关闭子进程中的 new_socket，这样**对应的 FD 才会正真关闭，此时才会触发四次挥手。所以父进程代码中蓝色部分 new_socket.close() 非常重要。**

**二、使用线程来实现并发 Web Server**
--------------------------

在第一节中，我们使用进程来实现并发，但是进程对资源消耗很大，一般不推荐使用。所以这里我们使用线程来实现并发，很简单，我们将 multiprocessing.Process 替换为 threaing.Thread 就可以了：

```
import socket
import re
import threading


def handle_request(new_socket):
    # 接收请求
    recv_msg = ""
    recv_msg = new_socket.recv(1024).decode("utf-8")
    if recv_msg == "":
        print("recv null")
        new_socket.close()
        return

    # 从请求中解析出URI
    recv_lines = recv_msg.splitlines()
    print(recv_lines.__len__())
    # 使用正则表达式提取出URI
    ret = re.match(r"[^/]+(/[^ ]*)", recv_lines[0])
    if ret:
        # 获取URI字符串
        file_name = ret.group(1)
        # 如果URI是/，则默认返回index.html的内容
        if file_name == "/":
            file_name = "/index.html"

    try:
        # 根据请求的URI，读取相应的文件
        fp = open("." + file_name, "rb")
    except:
        # 找不到文件，响应404
        response_msg = "HTTP/1.1 404 NOT FOUND\r\n"
        response_msg += "\r\n"
        response_msg += "<h1>----file not found----</h1>"
        new_socket.send(response_msg.encode("utf-8"))
    else:
        html_content = fp.read()
        fp.close()
        # 响应正确 200 OK
        response_msg = "HTTP/1.1 200 OK\r\n"
        response_msg += "\r\n"

        # 返回响应头
        new_socket.send(response_msg.encode("utf-8"))
        # 返回响应体
        new_socket.send(html_content)

    # 关闭该次socket连接
    new_socket.close()


def main():
    # 创建TCP SOCKET实例
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # # 设置重用地址
    # tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定地址（默认本机IP）和端口
    tcp_server_socket.bind(("", 7890))
    # 监听
    tcp_server_socket.listen(128)
    # 循环接收客户端连接
    while True:
        new_socket, client_addr = tcp_server_socket.accept()
        # 启动一个线程来处理客户端的请求
        t = threading.Thread(target=handle_request, args=(new_socket,))
        t.start()

    # 关闭整个SOCKET
    tcp_server_socket.close()


if __name__ == "__main__":
    main()
```

我们发现，除了将子进程的创建过程替换成了线程的创建过程，后面的 new_socket.close() 也被删除了，这是因为线程是公用进程资源的，new_socket 不会被复制，所以 socket 对应的 FD，只有一个 new_socket 指向他。

如果此时我们仍然在这里关闭 new_socket，那么在线程再使用 new_socket 就会报错。如下信息：

```
Exception in thread Thread-5:
Traceback (most recent call last):
  File "D:\Anaconda3_530\lib\threading.py", line 917, in _bootstrap_inner
    self.run()
  File "D:\Anaconda3_530\lib\threading.py", line 865, in run
    self._target(*self._args, **self._kwargs)
  File "D:/pycharm_project/leo1127/thread_web_server.py", line 44, in handle_request
    new_socket.send(response_msg.encode("utf-8"))
OSError: [WinError 10038] 在一个非套接字上尝试了一个操作。
```

****三、使用协程来实现并发 Web Server****
------------------------------

使用进程和线程来实现的并发 Web Server，当并发访问量很大时，资源消耗都很高。所以这里使用协程来实现并发服务器。

```
import socket
import re
import gevent
from gevent import monkey
monkey.patch_all()


def handle_request(new_socket):
    # 接收请求
    recv_msg = ""
    recv_msg = new_socket.recv(1024).decode("utf-8")
    if recv_msg == "":
        print("recv null")
        new_socket.close()
        return

    # 从请求中解析出URI
    recv_lines = recv_msg.splitlines()
    print(recv_lines.__len__())
    # 使用正则表达式提取出URI
    ret = re.match(r"[^/]+(/[^ ]*)", recv_lines[0])
    if ret:
        # 获取URI字符串
        file_name = ret.group(1)
        # 如果URI是/，则默认返回index.html的内容
        if file_name == "/":
            file_name = "/index.html"

    try:
        # 根据请求的URI，读取相应的文件
        fp = open("." + file_name, "rb")
    except:
        # 找不到文件，响应404
        response_msg = "HTTP/1.1 404 NOT FOUND\r\n"
        response_msg += "\r\n"
        response_msg += "<h1>----file not found----</h1>"
        new_socket.send(response_msg.encode("utf-8"))
    else:
        html_content = fp.read()
        fp.close()
        # 响应正确 200 OK
        response_msg = "HTTP/1.1 200 OK\r\n"
        response_msg += "\r\n"

        # 返回响应头
        new_socket.send(response_msg.encode("utf-8"))
        # 返回响应体
        new_socket.send(html_content)

    # 关闭该次socket连接
    new_socket.close()


def main():
    # 创建TCP SOCKET实例
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # # 设置重用地址
    # tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定地址（默认本机IP）和端口
    tcp_server_socket.bind(("", 7890))
    # 监听
    tcp_server_socket.listen(128)
    # 循环接收客户端连接
    while True:
        new_socket, client_addr = tcp_server_socket.accept()
        # 启动一个协程来处理客户端的请求
        gevent.spawn(handle_request, new_socket)

    # 关闭整个SOCKET
    tcp_server_socket.close()


if __name__ == "__main__":
    main()
```

使用 gevent 来实现协程，并发处理请求。

**四、使用单进程单线程非阻塞模拟并发（非并行）**
--------------------------

前面我们使用的多进程和多线程来处理并发，是因为 socket.recv() 是阻塞的，每次 accept 一个连接，就需要交给一个新的进程或线程去处理，从而不影响下一个 socket 连接。

但是我们可以通过单进程单线程和非阻塞的方式来完成并发 socket 的处理：

```
import socket


def main():
    # 创建TCP SOCKET实例
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # # 设置重用地址
    # tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定地址（默认本机IP）和端口
    tcp_server_socket.bind(("", 7890))
    # 监听
    tcp_server_socket.listen(128)

    # 将accept设置为非阻塞,这里设置一次，后面不管调多少次accept都是非阻塞的
    tcp_server_socket.setblocking(False)
    # 定义一个列表，将每次连接的socket加入该列表
    client_socket_list = list()

    # 循环接收客户端连接
    while True:
        try:
            new_socket, client_addr = tcp_server_socket.accept()
        except Exception as ret:
            # 当没有客户端链接的时候，抛出异常
            pass
        else:
            # 当有客户端链接的时候
            # 将new_socket.recv()设置为非阻塞的
            new_socket.setblocking(False)
            # 将new_socket加入列表
            client_socket_list.append(new_socket)

        # 遍历socket列表，检查每一个socket是否有数据到达，或者客户端是否断开
        for client_socket in client_socket_list:
            try:
                recv_content = client_socket.recv(1024)
            except Exception as ret:
                # 异常，表示该客户端没有发数据过来
                pass
            else:
                # 正常，表示客户端发了数据，或者客户端断开连接（断开连接会导致recv正常返回）
                if recv_content:
                    # 有数据，调用请求处理代码
                    print("处理请求")
                else:
                    # 客户端断开时，服务器也关闭连接
                    client_socket.close()
                    # 将已关闭的链接提出列表
                    client_socket_list.remove(client_socket)

    # 关闭整个SOCKET
    tcp_server_socket.close()
```

上面代码只有主干部分（省略了请求处理部分），主要是说明在单进程单线程情况下，如何将 accept 和 recv 分开，并且都用非阻塞的方式来处理，这样每次查看是否有客户端链接进来的时候，都会去检查所有已链接的 socket 是否有数据发送过来。

**注意：**socket.recv(1024) 一定要给参数，读取的字节数。否则会一直报异常。

在这种方式中，我们使用单进程单线程模拟了并发处理 socket 连接的功能，但这些 socket 连接的处理不是并行的。当一个 socket 处理数据时间比较长时，也会**造成整个程序的等待。**

**五、短连接和长连接**
-------------

在第四节中，我们使用单进程单线程非阻塞的形式实现了并发处理 socket 连接。其中省略了实际处理的部分，我们将其补充上：

```
import socket
import time
import re


def handle_request(new_socket, recv_msg):
    # 从请求中解析出URI
    recv_lines = recv_msg.splitlines()

    # 使用正则表达式提取出URI
    ret = re.match(r"[^/]+(/[^ ]*)", recv_lines[0])

    if ret:
        # 获取URI字符串
        file_name = ret.group(1)
        # 如果URI是/，则默认返回index.html的内容
        if file_name == "/":
            file_name = "/index.html"

    try:
        # 根据请求的URI，读取相应的文件
        fp = open("." + file_name, "rb")
    except:
        # 找不到文件，响应404
        response_msg = "HTTP/1.1 404 NOT FOUND\r\n"
        response_msg += "\r\n"
        response_msg += "<h1>----file not found----</h1>"
        new_socket.send(response_msg.encode("utf-8"))
    else:
        html_content = fp.read()
        fp.close()

        response_body = html_content

        # 响应正确 200 OK
        response_header = "HTTP/1.1 200 OK\r\n"
        response_header += "Content-Length:%d\r\n" % len(response_body)
        response_header += "\r\n"

        response = response_header.encode("utf-8") + response_body

        # 返回响应数据
        new_socket.send(response)


def main():
    # 创建TCP SOCKET实例
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # # 设置重用地址
    # tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定地址（默认本机IP）和端口
    tcp_server_socket.bind(("", 7890))
    # 监听
    tcp_server_socket.listen(128)

    # 将accept设置为非阻塞,这里设置一次，后面不管调多少次accept都是非阻塞的
    tcp_server_socket.setblocking(False)
    # 定义一个列表，将每次连接的socket加入该列表
    client_socket_list = list()

    # 循环接收客户端连接
    while True:
        time.sleep(0.5)

        try:
            new_socket, client_addr = tcp_server_socket.accept()
        except Exception as ret:
            # 当没有客户端链接的时候，抛出异常
            pass
        else:
            print("一个新的客户端连接。。。。")
            # 当有客户端链接的时候
            # 将new_socket.recv()设置为非阻塞的
            new_socket.setblocking(False)
            # 将new_socket加入列表
            client_socket_list.append(new_socket)
            print(client_socket_list.__len__())

        # 遍历socket列表，检查每一个socket是否有数据到达，或者客户端是否断开
        for client_socket in client_socket_list:
            try:
                recv_content = client_socket.recv(1024).decode("utf-8")
            except Exception as ret:
                # 异常，表示该客户端没有发数据过来
                pass
            else:
                # 正常，表示客户端发了数据，或者客户端断开连接（断开连接会导致recv正常返回）
                if recv_content:
                    # 有数据，调用请求处理代码
                    handle_request(client_socket, recv_content)
                else:
                    # recv正常返回，且数据为空，表示客户端断开了链接
                    # 将该socket踢出列表
                    client_socket_list.remove(client_socket)
                    # 服务器也关闭连接
                    client_socket.close()
    # 关闭整个SOCKET
    tcp_server_socket.close()


if __name__ == "__main__":
    main()
```

特别注意的是，在请求处理函数 handle_request 中，我们将请求内容作为参数一并传递进去。然后在返回 200 OK 的时候，在响应头中添加了 Content-Length 字段，这个字段用于告诉客户端，此次发送的响应体有多大。当客户端收完指定大小的数据，就认为这次服务器发送的数据已经发送完毕。他就可以继续发送下一个新的请求。

在 handle_request 中可以看到，new_socket.close() 已经被删除，也就是说服务器不会自动关闭连接，而直到客户端断开连接之前，服务器都保持长连接。断开连接由客户端来发起。