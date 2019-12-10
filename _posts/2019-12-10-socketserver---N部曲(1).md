## 曲一

`socketserver` 是为了简化服务器端开发而产生的，是一个高级的标准库。（背景介绍完毕，开始干）

### 一些概念

来自源码的一张图片，简洁哟`FengSao`

```
 		+------------+
        | BaseServer |
        +------------+
              |
              v
        +-----------+        +------------------+
        | TCPServer |------->| UnixStreamServer |
        +-----------+        +------------------+
              |
              v
        +-----------+        +--------------------+
        | UDPServer |------->| UnixDatagramServer |
        +-----------+        +--------------------+
```

这是`socketserver`的一个基本框架，`BaseServer` 是基类，是不可以初始化的。然后我们可以利用`BaseServer`的子类就是下面的四个，来实例化我们需要的服务器类型。（本人太菜只知道一点TCP的相关知识，所以从`TCPserver`入手学习）。

看到这个图开始还是很开心的，以为就这图上的5个类，结果大神写的源码把我一个英语不好的“澜”差点看哭了。我开始发现没那么简单，有好多的类，好多好多的类！

一行一行的看绝对懵逼，所以我决定不看其他的内容只看关于`TCPserver`的（好机智，好吧其实还是一脸懵逼），如果你也和我一样看到源码不知从何下手，下面给出一个我的查看路线吧（一行一行爬出来的一条路，当然只是`TCPserver`相关的内容）。

我用的`pycharm`，鼠标放在想查看源码的对象上，`ctrl + b` 就可以查看源码，还是非常方便的，其他的编辑器我就不知道了，下面开始。

不要一行一行的看了（想想当初憨憨的我...）先搜关键词`BaseRequestHandler` ，你会找到一个独立的类，看下面的说明有这样一段话

This class is instantiated for each request to be handled.  The
constructor sets the instance variables request, client_address
and server, and then calls the handle() method.  **To implement a**
**specific service, all you need to do is to derive a class which**
**defines a handle() method.**

The handle() method can find the request as self.request, the
client address as self.client_address, and the server (in case it
needs access to per-server information) as self.server.  Since a
separate instance is created for each request, the handle() method
can define other arbitrary instance variables.

注意加粗的句子，貌似告诉我们只需要派生一个`BaseRequestHandler`的子类，并且重写一下`handle()`方法就可以了，怎么样很简单的样子有没有。好写一个这个类的子类先上代码

```python
# Bianry_socketserver_v1
import socketserver
class my_tcphandler(socketserver.BaseRequestHandler):
    def handle(self):
        pass    # 不知道这个函数干嘛的的，先占个坑
```

继续看源码（心情沉重没有办法），继续搜关键字`BaseRequestHandler`，让我们把视线转到源码里的第一个关键词位置

To implement a service, you must derive a class from BaseRequestHandler and redefine its handle() method.  You can then run various versions of the service by combining one of the **server classes** with your **request handler class**.

看到还是让我们重写一个`BaseRequestHandler`的`handle()`方法，然后注意有一个关键词`server classes`，嗯！我们不是要一个`TCPserver`吗？貌似看到希望了啊。提示告诉我们要把自己写的`request handler class`和`server class`结合一下，怎么结合啊，别慌，关键字搜索`TCPServer`目光再次转移...

直接看`TCPServer`的构造函数（为什么？因为我在它的参数里看到了`BaseRequestHandlerClass`）

```python
# constractor of TCPServer
def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True):
        """Constructor.  May be extended, do not override."""   # 别人的函数叫你不要随便该
        BaseServer.__init__(self, server_address, RequestHandlerClass)
        self.socket = socket.socket(self.address_family,
                                    self.socket_type)
        if bind_and_activate:
            try:
                self.server_bind()
                self.server_activate()
            except:
                self.server_close()
                raise
```

看到他又调用了父类的构造函数，而且把我们想要的东西传了进去，那就查看它父类的构造函数，走起...

```python
# BaseServer constractor
def __init__(self, server_address, RequestHandlerClass):
        """Constructor.  May be extended, do not override."""
        self.server_address = server_address
        self.RequestHandlerClass = RequestHandlerClass
        self.__is_shut_down = threading.Event()
        self.__shutdown_request = False
```

看到父类就把它存了起来...就没了吗？不行继续搜`RequestHandlerClass`找到一个`finish_request`函数

```python
def finish_request(self, request, client_address):
        """Finish one request by instantiating RequestHandlerClass."""
        self.RequestHandlerClass(request, client_address, self)
```

终于看到一点实际的了，看到这个函数把我们传入的我们自己写的类`my_tcphandler`（继承的`BaseRequestHandler`）实例化了，好激动赶紧回去看看`BaseRequestHandler`构造函数有什么，因为自己写的`my_tcphandler`类没有构造函数，所以实例化会执行父类的构造函数先。

```python
# BaseRequestHandler constractor
def __init__(self, request, client_address, server):
        self.request = request
        self.client_address = client_address
        self.server = server
        self.setup()
        try:
            self.handle()
        finally:
            self.finish()
```

看到它加了几个变量进去`request`,`clinet_address`,`server`，而且先调用`setup()`函数，然后再`handle()`'（还占着一个坑的函数，等下去填上），最后你呢回去执行`finish()`，这三个函数都是没有写任何内容的，是给我们发挥的呀，刚刚只写了一个`handle()`，那怎么行，待会儿把这两个也加进去。现在的问题是`request`,`clinet_address`,`server`这三个变量是用来干嘛的呀，我们的三个函数怎么重构啊，我们要的是一个`TCPServer`啊。

上面的问题先不管了（船到桥头自然直，安慰一下自己），下面来尝试搭建我们的第一个`TCPServer`，看能不能行，先缕一缕思路

首先，我们要写一个`BaseRequestHandler`的子类并重写`handle()`方法，其他两个函数不是必须写的先不管。

然后，我们要实例化一个`TCPServer`的对象，并且把我们写的类作为参数传进去。（而实例化的过程非常的坎坷就是我们上面一步一步分析得到那样子。）

最后...没了耶，就两部这么简单

好开始写

```python
# Bianry_socketserver_v1
import socketserver
class my_tcphandler(socketserver.BaseRequestHandler):
    def handle(self):
        print('request:',self.request)
        print('clinet_address:',self.client_address)
        print('server:',self.server)
        # 不是不晓得这是三个什么东西吗，打印它，不解释
if __name__ == '__main__':
    addr,port = 'localhost',9999
    tcp_server = socketserver.TCPServer((addr,port),my_tcphandler)
    tcp_server.serve_forever()  # keep running...
```

运行程序发现没反应，没打印我们想要内容，这东西就像个憨憨一样，憨憨的在等一个人的到来。于是我们找了一个人（客户端）来尝试去告诉他一些真相（连接`TCPserver`），客户端程序

```python
import socket
clinet = socket.socket()
clinet.connect(('localhost',9999))
clinet.close()
```

只是连接一下，就断开了，就是个过客...

回首看看服务器端，打印了我们想要的东西

```
request: <socket.socket fd=424, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 9999), raddr=('127.0.0.1', 61933)>
clinet_address: ('127.0.0.1', 61933)
server: <socketserver.TCPServer object at 0x000001DE0F2DF5C0>
```

看到`request`原来是一个套接字，`clinet_address`原来是客户端留下的`ip`和`port`，`srever`原来就是自己（服务端）啊。

现在貌似知道`handle()`里应该写什么内容了，应该是和客户端交互的内容，因为我们有一个套接字（`request`），有了套接字就可以和和客户端交互了。而且`handle()`方法在客户端建立应该连接时才会执行，是不是想到了`accept()`... ，个人理解就是差不多的（当然对单线程服务器，多线程的原理还不懂，菜哭...），源码上是这么说的

**Since a separate instance is created for each request, the handle() method can define other arbitrary instance variables.**

是差不多吧，不多废话了，上代码来。

```python
# Bianry_socketTCPserver_v1
import socketserver
from time import ctime
class my_tcphandler(socketserver.BaseRequestHandler):
    def handle(self):
       try:
           while True:
               clinet_data = self.request.recv(1024)
               if not clinet_data:
                   break
               print('clinet:',clinet_data.decode())
               self.request.sendall(b'[%s] %s' % (clinet_data,ctime().encode('utf-8')))
       except Exception as e:
           exit(e)
       finally:
           self.request.close()
if __name__ == '__main__':
    addr,port = 'localhost',9999
    tcp_server = socketserver.TCPServer((addr,port),my_tcphandler)     # TCPServer
    tcp_server.serve_forever()  # keep running...
```

客户端

```python
# Bianry_socketTCPclinet_v1
import socket
clinet = socket.socket()
clinet.connect(('localhost',9999))
while True:
    data = input()
    if not data:
        break
    clinet.send(data.encode('utf-8'))
    server_data = clinet.recv(1024)
    print('server:',server_data.decode())
clinet.close()
```

这是单线程的，不能同时处理多个连接请求，想要处理多个怎么办，加几个字母就可以了

```python
# Bianry_socketThreadingTCPserver_v1
import socketserver
from time import ctime
class my_tcphandler(socketserver.BaseRequestHandler):
    def handle(self):
       try:
           while True:
               clinet_data = self.request.recv(1024)
               if not clinet_data:
                   break
               print('clinet:',clinet_data.decode())
               self.request.sendall(b'[%s] %s' % (clinet_data,ctime().encode('utf-8')))
       except Exception as e:
           exit(e)
       finally:
           self.request.close()
if __name__ == '__main__':
    addr,port = 'localhost',9999
    tcp_server = socketserver.ThreadingTCPServer((addr,port),my_tcphandler)     																			#ThreadingTCPServer
    tcp_server.serve_forever()  # keep running...
```

多线程还不会...菜鸡去学习了...

url:[一些关于socket编程有趣的小栗子](https://github.com/Binary-ZRJ/python/tree/master/socket)

url:[关于我和多线程socketserver的故事](菜鸡学习中....)