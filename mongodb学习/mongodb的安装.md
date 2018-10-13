# mongodb的安装

#### 下载、解压、移动

从[这个链接](https://www.mongodb.com/download-center#community)l里面下载对应版本的mongodb版本，然后使用命令：

`tar -zxvf mongodb-linux-x86_64-3.0.6.tgz`来解压。

`mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb`将解压好的目录移动到**/usr/local/mongodb**目录下，注意使用**sudo**命令。



#### 创建数据库文件夹，改编权限

然后进入**/usr/local/mongodb**目录下面，执行 `mkdir -p /data/db`命令创建文件夹，之后使用 `sudo chmod -R go+w /data/db`来改变root用户的权限。



#### 添加环境变量

之后通过 `gedit ~/.profile`在打开的文件末尾添加一句 `PATH="/usr/local/mongodb/bin:$PATH"` ，保存关闭。

然后使用命令 `source ~/.profile`刷新。



#### 安装依赖

输入 ` sudo apt-get install libcurl4-openssl-dev`命令来安装依赖的库，之后就可以在命令行通过 `mongod`命令来启动数据库啦。



#### 使用shell后台管理

**启动数据库**之后，可以进入数据库所在文件夹下面的bin目录，执行 `./mongo`就可以进入mongodb的后台管理的shell。**MongoDB Shell是MongoDB自带的交互式Javascript shell**,用来对MongoDB进行操作和管理的交互式环境。当你进mongoDB后台后，它默认会链接到 test 文档（数据库	）



