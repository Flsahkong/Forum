# tornado总体概述和深入

## tornado和Django框架

以Django为代表的python web应用部署时采用wsgi协议与服务器对接（被服务器托管），而**这类服务器通常都是基于多线程的**，也就是说**每一个网络请求服务器都会有一个对应的线程**来用web应用（如Django）进行处理。

## 考虑两类应用场景

- 用户量大，高并发

  如秒杀抢购、双十一某宝购物、春节抢火车票

- 大量的HTTP持久连接

  使用同一个TCP连接来发送和接收多个HTTP请求/应答，而不是为每一个新的请求/应答打开新的连接的方法。

  对于HTTP 1.0，可以在请求的包头（Header）中添加Connection: Keep-Alive。

  对于HTTP 1.1，所有的连接默认都是持久连接。

**对于这两种场景，通常基于多线程的服务器很难应对。**

## C10K问题

对于前文提出的这种高并发问题，我们通常用C10K这一概念来描述。C10K—— Concurrently handling ten thousandconnections，即并发10000个连接。对于单台服务器而言，根本无法承担，而采用多台服务器分布式又意味着高昂的成本。如何解决C10K问题？——tornado

## Tornado

Tornado在设计之初就考虑到了性能因素，旨在解决C10K问题，这样的设计使得其成为一个拥有非常高性能的解决方案（服务器与框架的集合体）。

**特点：**

- Tornado与大多数Python Web框架不同。它不基于[WSGI](https://wsgi.readthedocs.io/en/latest/)，并且通常每个进程只运行一个线程。有关 Tornado异步编程方法的更多信息，请参阅[用户指南](https://www.tornadoweb.org/en/stable/guide.html)。
- 通常，**Tornado代码不是线程安全的**。Tornado中唯一可以安全地从其他线程调用的方法是[`IOLoop.add_callback`](https://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.add_callback)。您还可以使用[`IOLoop.run_in_executor`](https://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.run_in_executor)异步在另一个线程上运行阻塞函数，但请注意，传递给它的函数`run_in_executor`应该避免引用任何Tornado对象。`run_in_executor`是阻止代码交互的推荐方法。
  - **线程安全**：线程安全是多线程领域的问题，线程安全可以简单理解为一个方法或者一个实例可以在多线程环境中使用而不会出现问题。
- 作为Web框架，是一个轻量级的Web框架，类似于另一个Python web框架Web.py，其拥有异步非阻塞IO的处理方式。
- 作为Web服务器，Tornado有较为出色的抗负载能力，官方用nginx反向代理的方式部署Tornado和其它Python web应用框架进行对比，结果最大浏览量超过第二名近40%。

Tornado框架和服务器一起组成一个WSGI的全栈替代品。单独在WSGI容器中使用tornado网络框架或者tornaod http服务器，有一定的局限性，为了最大化的利用tornado的性能，**推荐同时使用tornaod的网络框架和HTTP服务器。**



### tornado的官方文档

#### 用户指导

-------------

##### 介绍

[Tornado](http://www.tornadoweb.org/)是一个Python Web框架和异步网络库，通过使用非阻塞网络I / O，Tornado可以扩展到数万个开放连接，使其成为 [long polling](http://en.wikipedia.org/wiki/Push_technology#Long_polling)， [WebSockets](http://en.wikipedia.org/wiki/WebSocket)和其他需要与每个用户建立长期连接的应用程序的理想选择 。

 Tornado大致可分为四个主要部分：

- Web框架（包括[`RequestHandler`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler)子类，用于创建Web应用程序和各种支持类）。
- HTTP（[`HTTPServer`](https://www.tornadoweb.org/en/stable/httpserver.html#tornado.httpserver.HTTPServer)和 [`AsyncHTTPClient`](https://www.tornadoweb.org/en/stable/httpclient.html#tornado.httpclient.AsyncHTTPClient)）的**客户端**和**服务器端**实现。客户端可能对应的就是httpclient，就像是爬虫一样，可以当做一个客户机来发送请求。服务器端就是当做了服务器，返回请求的结果。
- 一个异步网络库，包括类[`IOLoop`](https://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop) 和[`IOStream`](https://www.tornadoweb.org/en/stable/iostream.html#tornado.iostream.IOStream)，它们作为HTTP组件的构建块，也可用于实现其他协议。
- 一个协程库（[`tornado.gen`](https://www.tornadoweb.org/en/stable/gen.html#module-tornado.gen)），它允许以比链接回调更直接的方式编写异步代码。这类似于Python 3.5（）中引入的本机协同程序功能。如果可用，建议使用本机协程代替模块。`async def`[`tornado.gen`](https://www.tornadoweb.org/en/stable/gen.html#module-tornado.gen)

##### 异步和非阻塞 I/O

实时Web功能需要每个用户长期保持大部分空闲连接。在传统的**同步Web服务器中，这意味着将一个线程投入到每个用户**，这可能非常昂贵。

为了最小化**并发连接的成本**，Tornado**使用单线程事件循环**。这意味着所有应用程序代码都应该是异步和非阻塞的，因为**一次只能有一个操作处于活动状态**。

术语异步和非阻塞是密切相关的，并且通常可以互换使用，但它们并不完全相同。

 > **阻塞**

函数在返回之前等待某事发生时会**阻塞**。一个函数可能由于多种原因而阻塞：网络I / O，磁盘I / O，互斥体等。事实上，**每个函数**在运行和使用CPU时都会至少有一点阻塞（对于一个极端的例子来说明为什么CPU阻塞必须像其他类型的阻塞一样严肃，考虑密码散列函数，如 [bcrypt](http://bcrypt.sourceforge.net/)，它设计使用数百毫秒的CPU时间，远远超过典型的网络或磁盘访问）。

函数在某些方面可以是阻塞的，在其他方面可以使非阻塞的。在Tornado的上下文中，我们通常谈论在网络I / O的上下文中阻塞，尽管要最小化所有类型的阻塞。

> **异步**

一个**异步**函数在他执行完成之前就返回了，通常会导致一些工作在后台进行。这里就像是Android和ajax里面的一样，当完成之后会调用回调函数callback，这其实就是异步。异步接口有很多种风格：

- 回调参数  Callback argument
- 返回一个占位符（[`Future`](https://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)，`Promise`，`Deferred`） Return a placeholder 
- 交付队列  Deliver to a queue
- 回调注册表（例如POSIX信号） Callback registry (e.g. POSIX signals)

无论使用哪种类型的接口，定义的异步函数 与其调用者的交互方式都不同; 没有一种自由的方法可以使同步函数以对其调用者透明的方式异步（像[gevent](http://www.gevent.org/)这样的系统使用轻量级线程来提供与异步系统相当的性能，但它们实际上并不会使事情异步）。

Tornado中的异步操作通常返回占位符对象（`Futures`），但某些低级组件除外，例如[`IOLoop`](https://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop)使用回调函数callback。`Futures`通常使用`await`或`yield` 关键字将其转换为结果。

##  Tornado与Django

### Django

Django是走大而全的方向，注重的是高效开发，它最出名的是其全自动化的管理后台：只需要使用起ORM，做简单的对象定义，它就能自动生成数据库结构、以及全功能的管理后台。

Django提供的方便，也意味着Django内置的ORM跟框架内的其他模块耦合程度高，应用程序必须使用Django内置的ORM，否则就不能享受到框架内提供的种种基于其ORM的便利。

- session功能
- 后台管理
- ORM

### Tornado

Tornado走的是**少而精**的方向，**注重的是性能优越**，它最出名的是**异步非阻塞的设计方式**。

- HTTP服务器
- 异步编程
- WebSockets

## tornado的深入学习

[tornado的深入学习](https://blog.csdn.net/fengge02/article/details/80045353)

