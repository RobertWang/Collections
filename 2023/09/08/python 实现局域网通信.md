> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/swWxSQlVThpSpyuktIAQpQ)

> 局域网中创建一个简单的客户端 - 服务器通信系统。实现可以在局域网内的多个设备之间进行消息传递，并实现双向通信。

**局域网中创建一个简单的客户端 - 服务器通信系统。实现可以在局域网内的多个设备之间进行消息传递，并实现双向通信。**

**服务器端：**

创建一个服务器 Socket 对象，绑定到指定的主机和端口。

监听来自客户端的连接请求。

一旦有客户端连接，就创建一个新的线程来处理该客户端的连接。

在处理客户端连接的线程中，接收客户端发送的消息，并打印出来。

从服务器端获取回复消息，并将其发送给客户端。

server.py  

```
import socket
import threading
def handle_client(client_socket):
    while True:
        try:
            data = client_socket.recv(1024).decode()
            if not data:
                print("客户端断开连接")
                break
            print("收到消息：", data)
            response = input("输入回复：")  # 从服务器端获取回复消息
            client_socket.send(response.encode())
        except Exception as e:
            print("发生异常：", e)
            break
    client_socket.close()
# 创建socket对象
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 获取本地主机名和端口号
host = socket.gethostname()
port = 12345
# 绑定地址和端口
server_socket.bind((host, port))
# 监听连接
server_socket.listen(5)
print("等待客户端连接...")
while True:
    client_socket, addr = server_socket.accept()
    print("连接地址：", addr)
    client_thread = threading.Thread(target=handle_client, args=(client_socket,))
    client_thread.start()

```

**客户端：**

创建一个客户端 Socket 对象，连接到指定的服务器主机和端口。  

循环等待用户输入消息。

将用户输入的消息发送给服务器。

接收服务器的回复消息，并打印出来。

通过这个脚本，你可以在局域网中的不同设备之间建立简单的通信连接。例如，你可以在一个设备上运行服务器端代码，然后在另一个设备上运行客户端代码。这样，你就可以通过客户端向服务器发送消息，并接收服务器的回复消息。

client.py

```
import socket
def send_message(client_socket, message):
    client_socket.send(message.encode())
    response = client_socket.recv(1024).decode()
    print("收到响应：", response)
# 创建socket对象
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 获取服务器的主机名和端口号
host = "192.168.2.13"
port = 12345
# 连接服务器
client_socket.connect((host, port))
while True:
    message = input("输入消息：")
    send_message(client_socket, message)

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/REwWx1OBrpjCbdu7htj1enbKYR9ZWaeRTibibdRmwtp16aFxKIaxUo6Vz7sb3FDUbMyV8DjdPCaSicX34wZMKI0TQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/REwWx1OBrpjCbdu7htj1enbKYR9ZWaeRLY9ZuenSsVt0A44DUHjgwmDbG6GyHhjQEwiaF17TibYB7vwYJnHx9Bicg/640?wx_fmt=png)