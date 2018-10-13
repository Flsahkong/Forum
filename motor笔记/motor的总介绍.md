# motor的总介绍

Motor 为 [Tornado](http://www.oschina.net/p/tornado) 提供了一个基于回调和 Future 机制的非堵塞的 [MongoDB](https://www.oschina.net/p/mongodb) 驱动程序。Motor 封装了 [PyMongo](http://www.oschina.net/p/pymongo)，Motor是一个异步的操作MongoDB的驱动。

## Motor和PyMongo之间的区别

### 主要差异

#### 连接到

Motor提供单个客户端类[`MotorClient`](https://motor.readthedocs.io/en/stable/api-tornado/motor_client.html#motor.motor_tornado.MotorClient)。与PyMongo不同 [`MongoClient`](http://api.mongodb.com/python/3.7.0/api/pymongo/mongo_client.html#pymongo.mongo_client.MongoClient)，Motor的客户端类在实例化时不会在后台开始连接。相反，当您第一次尝试操作时，它会按需连接。

就是说**Motor每次连接都会重新建立新的连接，而PyMongo不会**，他会使用同一个连接。但是Motor可以优化高并发带来的问题。

#### 线程和分叉(Threading and forking)

Motor不支持多线程和分叉。Motor旨在用于**单线程**Tornado应用程序。请参阅Tornado关于[running Tornado in production](http://www.tornadoweb.org/en/stable/guide/running.html) Tornado的文档 ，以利用多个内核。 