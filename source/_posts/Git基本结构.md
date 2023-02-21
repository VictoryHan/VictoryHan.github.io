---
title: Git基本结构
date: 2023-02-18 19:38:13
categories:
            - [数据结构]
            - [前端]
            - [后端]
            - [运维]
            - [网络安全]
            - [AI]
            - [大数据&数据分析]
            - [云计算]
tags:
    - Git
comment: true
---
<center>

![Git基本结构](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/jiagou.png)

Git基本结构

</center>



# Git托管中心
### 作用：维护远程库
### 局域网下
* Gitlab服务器
### 互联网环境下
* Github
* 码云
# 代码托管中心和本地库的交互
### 团队内协作

<center>

![git团队协作](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/jiegou.png)
</center>

### 跨团队协作

<center>

![git跨团队协作](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/jiegou2.png)

</center>

# 实操
### 本地库初始化
进入开发文件夹，进入此命令行的终端界面

`
git init
`
**效果**：出现隐藏的.git/文件夹。此目录中存放的是本地库相关的子目录和文件，不要删除，也不要胡乱修改。
### 设置签名
**作用**：区分不同的开发人员的身份
**辨析**：这里设置的签名和登录远程库（例如：码云和github）的账号、密码没有任何关系。
**项目/仓库级别**：仅在当前本地库范围内生效
`
git config user.name [用户名]
`
`
git config user.email [开发者邮箱]
`
**信息保存位置：**.git/config
例：

<center>

![用户文件信息](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info.png)

</center>

**系统用户级别**：登陆当前操作系统的用户范围
`
git config --global user.name [用户名]
`
`
git config --global user.email [开发者邮箱]
`
**信息保存位置：**~/.gitconfig
例：

<center>

![git全局用户信息](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info1.png)

</center>

**级别优先级**
*就近原则*：项目级别优先于系统用户级别，二者都有时采用项目级别的签名。如果只有系统用户级别的签名，就以系统用户级别的签名为准。**二者都没有是不允许的**
### 添加提交以及查看状态
**状态查看操作**
`
git status
`
查看工作区、暂存区状态
**添加操作**
`
git add [filename]
`
或者
`
git add --all
`
将工作区的"新建/修建"添加到暂存区
**提交操作**
`
git commit -m "commit  message" [filename]
`
或者
`
git commit -m "commit  message" --all
`
将暂存区的内容提交到本地库
### 版本前进和后退
**查看历史纪录**
方法1：
`
git log
`
多屏显示控制方式：
空格向下翻页
b向上翻页
q退出

<center>

![git log 效果图](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info2.png)

</center>
方法2：

`
git log --pretty=oneline
`

<center>

![git log](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info3.png)

</center>
方法3：

`
git log --oneline
`

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info4.png)



</center>
方法4：

`
git reflog
`

<center>

![git reflog](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info5.png)



</center>

HEAD@{移动到当前版本需要多少步}

**版本前进和后退**

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info6.png)



</center>

HEAD指针指向的是当前版本

**基于索引值操作（推荐）**
`
git reset --hard [索引值]
`

<center>

![](/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info7.png)



</center>

**使用^符号**
此方法只能后退版本，一个^符号退一步，三步以上用下个方法

`
git reset --hard HEAD^^^
`

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info8.png)



</center>

**使用~符号**
`
git reset --hard HEAD~3
`
~后面的数字是后退的步数,此方法也是只能后退。

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info9.png)



</center>

**reset命令的三个参数对比**
`
git reset --soft  [索引值]
`
此参数仅仅在本地库移动HEAD指针。
`
git reset --mixed [索引值]
`
此参数在本地库移动HEAD指针，重置暂存区。
`
git reset --hard [索引值]
`
此参数在本地库移动HEAD指针，重置暂存区，重置工作区。
**删除文件并找回**
git只会增加版本而不会删除任何一个版本。  所以，我们可以通过恢复到没有删除文件的版本恢复文件。
`
git reset --hard [指针位置]
`
**比较文件差异**
`
git diff [文件名]
`
将工作区中的文件和暂存区进行比较
`
git diff [本地库中历史版本] [文件名]
`
将工作区中的文件和本地库历史记录比较
`
git diff
`
不带文件名比较多个文件
### 分支管理
**什么是分支**
在版本控制过程中，使用多条线同时推进多个任务。

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/jiagou1.png)



</center>

**分支的好处**
* 同时并行推进多个功能开发，提高开发效率。
* 各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响。失败的分支删除重新开始即可

**分支操作**
* 创建分支
`
git branch [分支名]
`
* 查看分支
`
git branch -v
`
* 切换分支
`
git checkout [分支名]
`
* 合并分支
1、切换到接受修改的分支（被合并，增加新内容）上。
`
git checkout [接受修改的分支名]
`
2、执行merge命令
`
git merge [有新内容分支名]
`
* 解决冲突
当前分支和合并分支同一位置内容不同，就产生了冲突，不知道采用哪一个，需要人为干预。
分支的表现
*

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/jiegou3.png)

</center>

解决
1、编辑文件，删除特殊符号。特殊符号如下：

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/jiegou4.png)



</center>

2、把文件修改到满意的程度，保存退出
3、`git add [文件名]`
4、`git commit -m "日志信息"`(注：此时commit不一定带具体的文件名)
### 本地库和远程库交互
**创建远程库**

<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step.png)

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step2.png)

</center>

**创建远程库地址别名**
`
git remote -v
`
查看当前所有远程地址别名
`
git remote add 远程库地址别名 [远程地址]
`
添加远程库

`
git remote remove 远程库地址别名
`
删除远程库


<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info10.png)

</center>

**推送**
`git push 远程库地址别名 远程分支名`

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/info11.png)

</center>

**克隆**

`git clone [远程地址]`(默认master分支)

`git clone [远程库地址] -b [分支名]`（指定分支）

效果：
1、完整的把远程库下载到本地
2、创建远程地址别名
3、初始化本地库
### 团队成员邀请

<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step3.png)

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step4.png)

</center>

被邀请者登录自己的Github账号，访问邀请链接。

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step5.png)



</center>

**拉取**

pull=fetch+merge

`git fetch 远程库地址别名 远程分支名`

`git merge 远程库地址别名 远程分支名`

`git pull  远程库地址别名 远程分支名`

fetch 是将远程库的内容仅仅下载到本地

merge 是将远程库的内容与本地库的内容合并

**解决远程库和本地库冲突**

如果不是基于Github远程库的最新版本所做的修改，不能推送，必须先拉取。
拉取下来后如果进入冲突状态，则按照上面"解决冲突"操作解决即可。

**设置git push和pull的默认远程分支**

`git branch --set-upstream-to=远程库地址别名/远程分支名 本地分知名`

示例：

`git branch --set-upstream-to=origin/master master`

### 跨团队协作
团队外成员：
1、找到项目，Fork项目

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step6.png)

</center>

2、本地修改，然后推送到远程库
3、Pull Request

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step7.png)
![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step8.png)
![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step9.png)

</center>
团队内成员
1、审核代码

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step10.png)

</center>
2、合并代码

<center>

![](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step11.png)



</center>
3、将远程库修改拉去到本地

### 配置表SSH免密登录
1、进入当前用户的家目录
`cd ~`[linux]
`C:
`
`
cd User/[当前系统用户名]`[Windows]

2、运行命令生成ssh密钥目录及文件
`ssh-keygen -t rsa -C [远程库邮件地址]`
3、进入.ssh目录查看文件列表
`cd .ssh/`[通用]
`cat id_rsa.pub`[Linux]
`notepad id_rsa.pub`[Windows]
4、复制id_rsa.pub文件内容，登录Github，点击用户头像→Settings→SSH and GPG keys
<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step12.png)

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step13.png)

</center>
5、New SSH Key

<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step14.png)

</center>

6、输入复制的密钥信息
<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step15.png)

</center>
7、复制项目SSH地址，回到 本地终端 创建远程库地址别名。

`git remote add [远程库别名] [远程库SSH地址]`

<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step16.png)

</center>



8、测试

`ssh -T git@github.com`

<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step17.png)

</center>

附：如果有以下错误，可以选择忽略或注释掉配置文件（/etc/ssh/ssh_config）中的 `RSAAuthentication yes`行

<center>

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step18.png)

![img](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step19.png)

</center>

<center>

如果您乐意，感谢支持~

![微信赞赏码](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step20.png)

![支付宝收款码](/2023/02/18/Git%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84/step21.png)

</center>
