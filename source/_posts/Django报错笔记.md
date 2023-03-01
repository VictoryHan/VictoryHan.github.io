---
title: Django报错笔记
date: 2023-02-24 09:32:02
categories:
            - [后端]
tags:
    - Python
    - Django
---
##  一、  `(1054, "Unknown column 'xxxxx' in 'field list'")`
更改数据库字段名称后同步数据包如上错误
原因：可能是数据库没清理干净，字段干扰造成的。
解决办法：
在数据库中清理掉相应模型的app，然后重新迁移文件。
```
python  manage.py  migrate  应用名称    zero
python manage.py makemigrations 
python manage.py migrate 
```
未完待续······

<center>

如果您乐意，感谢支持~

![微信赞赏码](Django报错笔记/01.jpg)

![支付宝收款码](Django报错笔记/02.jpg)

</center>