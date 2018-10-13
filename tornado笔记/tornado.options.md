# `tornado.options` — Command-line parsing

## tornado.options.define()函数

用来定义options选项变量的方法，定义的变量**可以在全局的tornado.options.options中获取使用**，传入参数：

- name 选项变量名，须保证全局唯一性，否则会报“Option 'xxx' already defined in ...”的错误；
- default　选项变量的默认值，如不传默认为None；
- type 选项变量的类型，从命令行或配置文件导入参数的时候tornado会根据这个类型转换输入的值，转换不成功时会报错，可以是str、float、int、datetime、timedelta中的某个，若未设置则根据default的值自动推断，若default也未设置，那么不再进行转换。可以通过利用设置type类型字段来过滤不正确的输入。
- multiple 选项变量的值是否可以为多个，布尔类型，默认值为False，如果multiple为True，那么设置选项变量时值与值之间用英文逗号分隔，而选项变量则是一个list列表（若默认值和输入均未设置，则为空列表[]）。
- help 选项变量的帮助提示信息，在命令行启动tornado时，通过加入命令行参数 --help　可以查看所有选项变量的信息（注意，代码中需要加入tornado.options.parse_command_line()）。

## tornado.options.parse_command_line()

转换命令行参数，并将转换后的值对应的设置到全局options对象相关属性上。追加命令行参数的方式是--myoption=myvalue

## tornado.options.parse_config_file(path)

从配置文件导入option，配置文件中的选项格式如下：
```
myoption = "myvalue"
myotheroption = "myothervalue"
```
我们用代码来看一下如何使用，新建配置文件config，注意字符串和列表按照python的语法格式：

port = 8000
itcast = ["python","c++","java","php","ios"]
