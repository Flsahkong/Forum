# tornado 关于多进程方面

# 运行和部署

由于Tornado提供自己的HTTPServer，因此运行和部署它与其他Python Web框架略有不同。您可以编写一个`main()`启动服务器的函数，而不是配置WSGI容器来查找应用程序 ：

```
def main():
    app = make_app()
    app.listen(8888)
    IOLoop.current().start()

if __name__ == '__main__':
    main()
```

配置操作系统或进程管理器以运行此程序以启动服务器。请注意，可能需要增加每个进程的打开文件数（以避免“打开太多文件”-Error）。要提高此限制（例如将其设置为50000），您可以使用ulimit命令，修改/etc/security/limits.conf或`minfds`在supervisord配置中进行设置。

## 进程和端口

由于Python GIL（全局解释器锁），有必要运行多个Python进程以充分利用多CPU机器。通常，每个CPU最好运行一个进程。

**Tornado包含一个内置的多进程（注意是进程不是线程）模式，可以同时启动多个进程。**这需要对标准主要功能稍作改动：

```
def main():
    app = make_app()
    server = tornado.httpserver.HTTPServer(app) #这一句是需要添加的
    server.bind(8888)
    server.start(0)  # forks one process per cpu
    IOLoop.current().start()
```

> http_server.bind(port)方法是将服务器绑定到指定端口。

> http_server.start(num_processes=1)方法指定开启几个进程，参数num_processes默认值为1，即默认仅开启一个进程；如果num_processes为None或者<=0，则自动根据机器硬件的cpu核芯数创建同等数目的子进程；如果num_processes>0，则创建num_processes个子进程。

**虽然tornado给我们提供了一次开启多个进程的方法，但是由于：**

- 每个子进程都会从父进程中复制一份IOLoop实例，如过在创建子进程前我们的代码动了IOLoop实例，那么会影响到每一个子进程，势必会干扰到子进程IOLoop的工作；
- 所有进程是由一个命令一次开启的，也就无法做到在不停服务的情况下更新代码；
- 所有进程共享同一个端口，想要分别单独监控每一个进程就很困难。
- **不建议使用这种多进程的方式，而是手动开启多个进程，并且绑定不同的端口。**

