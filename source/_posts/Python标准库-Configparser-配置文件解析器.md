---
title: Python标准库-Configparser-配置文件解析器
date: 2023-02-24 09:35:17
categories:
            - [后端]
            - [运维]
tags:
    - Python
    - 配置文件解析器
---
# 简介
[官方文档](https://docs.python.org/zh-cn/3.10/library/configparser.html)
开发中INI格式的配置文件使用还是有必要的。如果某些配置项需要在运行时由用户来修改指定，比如数据库用户信息等等，这种配置项如果使用INI格式的配置文件来操作的话就会方便很多。Python中操作配置文件的模块为configparser，这个模块可以用来解析与Windows上INI文件结构类似的文件。
****ps：****这个库 并不 能够解析或写入在 Windows Registry 扩展版本 INI 语法中所使用的值-类型前缀。

# 实操
### 受支持的INI文件结构
配置文件是由小节组成的，每个小节都有一个[section] 标头，加上多个由特定字符串 (默认为 = 或 : 1) 分隔的键/值条目。 在默认情况下，****小节名对大小写敏感而键对大小写不敏感**** 。键和值开头和末尾的空格会被移除。 在解释器配置允许时值可以被省略 ，在此情况下键/值分隔符也可以被省略。 值还可以跨越多行，只要值的其他行带有比第一行更深的缩进。 依据解析器的具体模式，空白行可能会被视为多行值的组成部分或是被忽略。
* section：表示一个区块，由方括号及方括号中的名称组成，section的范围为当前方括号到下一个方括号的内容。如“Simple Values”、“All Values Are Strings“......特性如下：
（1）大小写和空格检查：section中的名称在保存和获取的时候是原样保存和获取的，即大小写不一样或者空格不一样等都是不同的section；
（2）重复性检查：同一个配置文件中section名称是不允许重复的。
* option：表示section中的配置项，由key、分隔符和value组成的键值对，如：[Simple Values]下的”key=value“。特性如下：
（1）大小写检查：key是大小写不敏感的，保存进文件的时候会自动将key小写保存，但value是大小写敏感的；
（2）空格检查：通过key获取value时，会自动将文件中的key和value前后空格去掉再进行匹配。
（3）跨多行检查：key是不能跨行的，但是value可以跨行，只要第二行即之后的行的缩进与第一行不同即可，一直到下一个option为止；
（4）重复性检查：和section一样，同一section下的key是不允许重复的；
（5）分隔符：可以是等号“=”或者冒号“:”。
* 注释：行注释用井号“#”或者分号“;”表示，特别需要注意的是必须得是行开头（前面可以有空格），用在行中间的就不会算作是注释了。
* DEFAULT：这是一个特殊的section，会用作其他section的option取不到值时的备用值，或者可以理解为它是一个root，其他的section都是它的子section，但不是必须提供的。
格式示例：
```
[Simple Values]
key=value
spaces in keys=allowed
spaces in values=allowed as well
spaces around the delimiter = obviously
you can also use : to delimit keys from values

[All Values Are Strings]
values like this: 1000000
or this: 3.14159265359
are they treated as numbers? : no
integers, floats and booleans are held as: strings
can use the API to get converted values directly: true

[Multiline Values]
chorus: I'm a lumberjack, and I'm okay
    I sleep all night and I work all day

[No Values]
key_without_value
empty string value here =

[You can use comments]
# like this
; or this

# By default only in an empty line.
# Inline comments can be harmful because they prevent users
# from using the delimiting characters as parts of values.
# That being said, this can be customized.

    [Sections Can Be Indented]
        can_values_be_as_well = True
        does_that_mean_anything_special = False
        purpose = formatting for readability
        multiline_values = are
            handled just fine as
            long as they are indented
            deeper than the first line
            of a value
        # Did I mention we can indent comments, too?
```
### 支持的数据类型
 配置解析器并不会猜测配置文件中值的类型，而总是将它们在内部存储为字符串。 这意味着如果你需要其他数据类型，你应当自己来转换

### 写入数据
```
>>> import configparser
>>> config = configparser.ConfigParser()
>>> config['DEFAULT'] = {'ServerAliveInterval': '45',
...                      'Compression': 'yes',
...                      'CompressionLevel': '9'}
>>> config['bitbucket.org'] = {}
>>> config['bitbucket.org']['User'] = 'hg'
>>> config['topsecret.server.com'] = {}
>>> topsecret = config['topsecret.server.com']
>>> topsecret['Port'] = '50022'     # mutates the parser
>>> topsecret['ForwardX11'] = 'no'  # same here
>>> config['DEFAULT']['ForwardX11'] = 'yes'
>>> with open('example.ini', 'w') as configfile:
...   config.write(configfile)
```
### 读取数据
```
>>> config = configparser.ConfigParser()
>>> config.sections()
[]
>>> config.read('example.ini')
['example.ini']
>>> config.sections()
['bitbucket.org', 'topsecret.server.com']
>>> 'bitbucket.org' in config
True
>>> 'bytebong.com' in config
False
>>> config['bitbucket.org']['User']
'hg'
>>> config['DEFAULT']['Compression']
'yes'
>>> topsecret = config['topsecret.server.com']
>>> topsecret['ForwardX11']
'no'
>>> topsecret['Port']
'50022'
>>> for key in config['bitbucket.org']:  
...     print(key)
user
compressionlevel
serveraliveinterval
compression
forwardx11
>>> config['bitbucket.org']['ForwardX11']
'yes'
```
### RawConfigParser 与 ConfigParser



一般情况都是使用ConfigParser这个方法，但是当我们配置中有%(filename)s这种格式的配置的时候，可能会出现以下问题：

 
```

configparser.InterpolationMissingOptionError: Bad value substitution: option 'output_fmt' in section 'output' contains an interpolation key 'asctime' which is not a valid option name.

Raw value: '"%(asctime)s-%(levelname)s-%(filename)s-%(name)s-日志信息:%(message)s"'

```
 

解决方法：

将ConfigParser方法换成RawConfigParser方法，上面的问题即可解决。


<center>

如果您乐意，感谢支持~

![微信赞赏码](Python标准库-Configparser-配置文件解析器/01.jpg)

![支付宝收款码](Python标准库-Configparser-配置文件解析器/02.jpg)

</center>