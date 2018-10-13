# MongoDB 创建集合

## 方法一

MongoDB 中使用 **createCollection()** 方法来创建集合。

语法格式：

```
db.createCollection(name, options)
```

参数说明：

- name: 要创建的集合名称
- options: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段        | 类型 | 描述                                                         |
| ----------- | ---- | ------------------------------------------------------------ |
| capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔 | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。   |
| size        | 数值 | （可选）为固定集合指定一个最大值（以字节计）。 **如果 capped 为 true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

在插入文档时，MongoDB **首先检查固定集合的 size 字段，然后检查 max 字段**。

### 实例

在 test 数据库中创建 runoob 集合：

```
> use test
switched to db test
> db.createCollection("runoob")
{ "ok" : 1 }
>
```

如果要查看已有集合，可以使用 show collections 命令：

```
> show collections
runoob
```

下面是带有几个关键参数的 createCollection() 的用法：

创建固定集合 mycol，整个集合空间大小 6142800 KB, 文档最大个数为 10000 个。

```
> db.createCollection("mycol", { capped : true, autoIndexId : true, size : 
   6142800, max : 10000 } )
{ "ok" : 1 }
>
```

 在 MongoDB 中，**你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。**下面的例子会自动创建mycol2这个集合。

```
> db.mycol2.insert({"name" : "菜鸟教程"})
> show collections
mycol2
...
```

## 方法二

**insert()方法自动创建集合**

MongoDB可以在集合未创建的情况下，直接使用`insert()`方法插入数据。插入数据时，如果集合还未创建，会自动创建该集合：

```
> db.sites.insert({name:"itbilu.com"})
WriteResult({ "nInserted" : 1 })
> show collections
sites
student
teams
users
……
```

# MongoDB 删除集合

本章节我们为大家介绍如何使用 MongoDB 来删除集合。

MongoDB 中使用 drop() 方法来删除集合。

**语法格式：**

```
db.collection.drop()
```

**返回值**

如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。

### 实例

在数据库 mydb 中，我们可以先通过 show collections 命令查看已存在的集合：

```
>use mydb
switched to db mydb
>show collections
mycol
mycol2
system.indexes
runoob
>
```

接着删除集合 mycol2 :

```
>db.mycol2.drop()
true
>
```

通过 show collections 再次查看数据库 mydb 中的集合：

```
>show collections
mycol
system.indexes
runoob
>
```

从结果中可以看出 mycol2 集合已被删除。



# 查看数据库中的集合

查看数据库的集合使用`show collections`命令。

```
>show collections
teams
users
……
```

某些关系型数据库中，查询所有表的命令`show tables`，在MongoDB也可以用来查看集合。

```
>show tables
teams
users
……
```

`db.getCollectionNames()`用查看文档中所有集合的名称，其返回值是一个数组：

```
> db.getCollectionNames()
[
	"teams",
	"users",
	……
]
```

如果需要查看单个集合，可以使用` db.getCollection("NAME")`命令：

```
> db.getCollection("users")
newDB.users
```

 

# 集合的重命名

集合的重命名使用`renameCollection`方法。

```
db.COLLECTION_NAME.renameCollection("NEW_NAME")
```

### 实例

将`student`重命名为`students`

```
> db.student.renameCollection("students")
{ "ok" : 1 }
> show collections
sites
students
teams
users
……
```

 