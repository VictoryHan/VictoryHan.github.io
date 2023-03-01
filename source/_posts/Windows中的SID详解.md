---
title: Windows中的SID详解
date: 2023-02-18 21:31:46
categories:
            - [运维]
            - [网络安全]
tags:
    - Windows
---
# SID详解
## 前言
SID也就是安全标识符（Security Identifiers），是标识用户、组和计算机帐户的唯一的号码。在第一次创建该帐户时，将给网络上的每一个帐户发布一个唯一的 SID。Windows 2000 中的内部进程将引用帐户的 SID 而不是帐户的用户或组名。如果创建帐户，再删除帐户，然后使用相同的用户名创建另一个帐户，则新帐户将不具有授权给前一个帐户的权力或权限，原因是该帐户具有不同的 SID 号。安全标识符也被称为安全 ID 或 SID。 
## SID的作用
用户通过验证后，登陆进程会给用户一个访问令牌，该令牌相当于用户访问系统资源的票证，当用户试图访问系统资源时，将访问令牌提供给 Windows NT，然后 Windows NT 检查用户试图访问对象上的访问控制列表。如果用户被允许访问该对象，Windows NT将会分配给用户适当的访问权限。
访问令牌是用户在通过验证的时候有登陆进程所提供的，所以改变用户的权限需要注销后重新登陆，重新获取访问令牌。
## SID号码的组成
如果存在两个同样SID的用户，这两个帐户将被鉴别为同一个帐户，原理上如果帐户无限制增加的时候，会产生同样的SID，在通常的情况下SID是唯一的，他由计算机名、当前时间、当前用户态线程的CPU耗费时间的总和三个参数决定以保证它的唯一性。

一个完整的SID包括：
• 用户和组的安全描述
• 48-bit的ID authority
• 修订版本
• 可变的验证值Variable sub-authority values
例：S-1-5-21-310440588-250036847-580389505-500
我们来先分析这个重要的SID。第一项S表示该字符串是SID；第二项是SID的版本号，对于2000来说，这个就是1；然后是标志符的颁发机构（identifier authority），对于2000内的帐户，颁发机构就是NT，值是5。然后表示一系列的子颁发机构，前面几项是标志域的，最后一个标志着域内的帐户和组。
## SID的获得
开始－运行－regedt32－HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Builtin\Aliases\Members，找到本地的域的代码，展开后，得到的就是本地帐号的所有SID列表。
其中很多值都是固定的，比如第一个000001F4（16进制），换算成十进制是500，说明是系统建立的内置管理员帐号administrator，000001F5换算成10进制是501，也就是GUEST帐号了，详细的参照后面的列表。
这一项默认是system可以完全控制，这也就是为什么要获得这个需要一个System的Cmd的Shell的原因了，当然如果权限足够的话你可以把你要添加的帐号添加进去。
或者使用Support Tools的Reg工具：
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList

还有一种方法可以获得SID和用户名称的对应关系：
1. Regedt32:
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion \ProfileList
2. 这个时候可以在左侧的窗口看到SID的值，可以在右侧的窗口中ProfileImagePath看到不同的SID关联的用户名，
比如%SystemDrive%\Documents and Settings\Administrator.momo这个对应的就是本地机器的管理员SID
%SystemDrive%\Documents and Settings\Administrator.domain这个就是对应域的管理员的帐户

另外微软的ResourceKit里面也提供了工具getsid，sysinternals的工具包里面也有Psgetsid，其实感觉原理都是读取注册表的值罢了，就是省了一些事情。
## SID重复问题的产生
安装NT／2000系统的时候，产生了一个唯一的SID，但是当你使用类似Ghost的软件克隆机器的时候，就会产生不同的机器使用一个SID的问题。产生了很严重的安全问题。
同样，如果是重复的SID对于对等网来说也会产生很多安全方面的问题。在对等网中帐号的基础是SID加上一个相关的标识符（RID），如果所有的工作站都拥有一样的SID，每个工作站上产生的第一个帐号都是一样的，这样就对用户本身的文件夹和文件的安全产生了隐患。
这个时候某个人在自己的NTFS分区建立了共享，并且设置了自己可以访问，但是实际上另外一台机器的SID号码和这个一样的用户此时也是可以访问这个共享的。
## SID重复问题的解决
下面的几个试验带有高危险性，慎用，我已经付出了惨痛的代价！
微软在ResourceKit里面提供了一个工具，叫做SYSPREP,这个可以用在克隆一台工作站以前产生一个新的SID号码。 下图是他的参数

这个工具在DC上是不能运行这个命令的，否则会提示

但是这个工具并不是把所有的帐户完全的产生新的SID，而是针对两个主要的帐户Administrator和Guest，其他的帐号仍然使用原有的SID。
下面做一个试验，先获得目前帐号的SID：
S-1-5-21-2000478354-688789844-839522115
然后运行Sysprep，出现提示窗口：

确定以后需要重启，然后安装程序需要重新设置计算机名称、管理员口令等，但是登陆的时候还是需要输入原帐号的口令。
进入2000以后，再次查询SID，得到：
S-1-5-21-759461550-145307086-515799519，发现SID号已经得到了改变，查询注册表，发现注册表已经全部修改了，当然全部修改了?。

另外sysinternals公司也提供了类似的工具NTSID，这个到后来才发现是针对NT4的产品，界面如下：

他可不会提示什么再DC上不能用，接受了就开始，结果导致我的一台DC崩溃，重启后提示“安全账号管理器初始化失败，提供给识别代号颁发机构的值为无效值，错误状态0XC0000084，请按确定，重启到目录服务还原模式...”，即使切换到目录服务还原模式也再也进不去了！
想想自己胆子也够大的啊，好在是一台额外DC，但是自己用的机器，导致重装系统半天，重装软件N天?，所以再次提醒大家，做以上试验的时候一定要慎重，最好在一台无关紧要的机器上试验，否则出现问题我不负责哦?。另外在Ghost的新版企业版本中的控制台已经加入了修改SID的功能，自己还没有尝试，有兴趣的朋友可以自己试验一下，不过从原理上应该都是一样的。
文章发表之前，又发现了微软自己提供的一个工具“Riprep”，这个工具主要用做在远程安装的过程中，想要同时安装上应用程序。管理员安装了一个标准的公司桌面操作系统，并配置好应用软件和一些桌面设置之后，可以使用Riprep从这个标准的公司桌面系统制作一个Image文件。这个Image文件既包括了客户化的应用软件，又把每个桌面系统必须独占的安全ID、计算机账号等删除了。管理员可以它放到远程安装服务器上，供客户端远程启动进行安装时选用。但是要注意的是这个工具只能在单硬盘、单分区而且是Professional的机器上面用。


下面是SID末尾RID值的列表，括号内为16进制：

Built-In Users
DOMAINNAME\ADMINISTRATOR
S-1-5-21-917267712-1342860078-1792151419-500 (=0x1F4)

DOMAINNAME\GUEST
S-1-5-21-917267712-1342860078-1792151419-501 (=0x1F5)
Built-In Global Groups
DOMAINNAME\DOMAIN ADMINS
S-1-5-21-917267712-1342860078-1792151419-512 (=0x200)

DOMAINNAME\DOMAIN USERS
S-1-5-21-917267712-1342860078-1792151419-513 (=0x201)

DOMAINNAME\DOMAIN GUESTS
S-1-5-21-917267712-1342860078-1792151419-514 (=0x202)
Built-In Local Groups
BUILTIN\ADMINISTRATORS S-1-5-32-544 (=0x220)
BUILTIN\USERS S-1-5-32-545 (=0x221)
BUILTIN\GUESTS S-1-5-32-546 (=0x222)
BUILTIN\ACCOUNT OPERATORS S-1-5-32-548 (=0x224)
BUILTIN\SERVER OPERATORS S-1-5-32-549 (=0x225)
BUILTIN\PRINT OPERATORS S-1-5-32-550 (=0x226)
BUILTIN\BACKUP OPERATORS S-1-5-32-551 (=0x227)
BUILTIN\REPLICATOR S-1-5-32-552 (=0x228)
Special Groups
\CREATOR OWNER S-1-3-0
\EVERYONE S-1-1-0
NT AUTHORITY\NETWORK S-1-5-2
NT AUTHORITY\INTERACTIVE S-1-5-4
NT AUTHORITY\SYSTEM S-1-5-18
NT AUTHORITY\authenticated users S-1-5-11 *.(over)

## 如何修改镜像操作系统的SID
SID也就是安全标识符（Security Identifiers），是标识用户、组和计算机帐户的唯一的号码。在第一次创建该帐户时，将给网络上的每一个帐户发布一个唯一的 SID。Windows 2000 中的内部进程将引用帐户的 SID 而不是帐户的用户或组名。如果创建帐户，再删除帐户，然后使用相同的用户名创建另一个帐户，则新帐户将不具有授权给前一个帐户的权力或权限，原因是该帐户具有不同的 SID 号。安全标识符也被称为安全 ID 或 SID。如果我们要想查看我们的SID有很多方法可以看见比如通过注册表就可以找到,另外我们也可以通过CMD中的命令来查看SID,用命令“whoami /user”来查看用户的SID的

SID我们也看见了，那到底该如何来修改操作系统SID呢？其实很简单，这里依windows server 2003操作系统为例，在windows server 2003操作系统的安装盘中有一个SUPPORT文件夹打开该文件夹找到TOOLS子文件夹,然后找到DEPLOY.CAB文件包进行解压, 解压完成之后，直接点击sysprep.exe文件会出现图二窗口，点击“工厂”按钮，计算机会自动关闭，然后打开计算机，再查看SID就可以看见SID已经被修改了。

出处：http://lotus-3hao.blog.sohu.com/51996952.html

=================================================================

使用用Ghost制作的win2k3和winxp文件具有相同的SID的解决办法

特别声明:这篇文档是从我的朋友发给我的邮件中整理的。我觉得非常好就把它共享出来，让大家分享。非常感谢他为我们作的贡献。
### 问题描述
在Windows内部，每个账号具有一个惟一的Security ID，可以在HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion \ProfileList看到。SID是用来识别账户的惟一标志，而不是通常以为的机器名\用户名。而现有的Ghost文件是把整个安装好的系统分区直接备份下来，这样，当把这个Ghost文件恢复到别的机器上时，别的机器也具有了同样的SID（欲知SID更多详情，请参见http://www.winmag.com.cn/html/2005/03/20050324170613-1.shtml）。
带来的问题：
1.潜在的安全问题。如果某台机器设置了一个共享目录，限制为只有本机Administrator才能完全访问，但实际上来自别的机器的Administrator也有可能访问。
2.在域中，涉及组策略等操作的话，也是以SID为机器或用户的惟一标志。如果LAN中存在重复的SID，组策略将不能正常工作。
### 解决方案
使用Windows XP和2003安装光盘中提供的附加工具Sysprep，它是专为在多台计算机上部署Windows而设计的工具，可以擦除并重新生成SID。Sysprep工具在安装光盘的support\tools\deploy.cab中，解压将得到10个文件。你可以使用任意语言的deploy.cab。
### 额外发现
经过一天的研究，Sysprep除了重新生成SID之外，还可以为我们做这样两件非常好的事：
1.将Windows激活期限展期。
我们的测试环境中所使用的Windows XP和2003都是未激活的。受到激活期限的限制，我们不能使用真实时间，而是要根据Ghost文件的日期相应调整机器日期。比如Ghost文件日期不幸为2001年1月1日，测试机的日期就必须使用2001年1月的某天。这样的机器一旦加入其它时间的域中，将被同步时间而导致Windows过期。而经过Sysprep，Windows重启时将重新执行所有系统设置。重启之后Windows XP就可以重新恢复30天的激活日期，2003类推。想无限Sysprep吗？忘掉这个念头吧，早就试过了，Sysprep的展期功能只允许使用三次-_-b..正确的解决方法见后。
2.重新扫描硬件并安装驱动。
我们之前的Ghost文件中是已经安装有驱动程序的，所以这些Ghost文件只能对应于特定机型。如果将Dell 170L的Ghost恢复到了Dell 3000上，有时会出现驱动不能正常工作，但是又无法安装新驱动的情况（虽然他们都是Intel 865G芯片组，板载显卡网卡都一样……奇怪中）。
同样经过Sysprep，可以擦除HAL（硬件抽象层）信息，Windows重启时将重新扫描硬件。如果系统已经安装某些驱动文件，这些驱动会被自动识别。即使不能自动识别，也不会影响安装新的驱动。惟一的问题是暂时无法将若干种显卡、网卡、芯片组的驱动集成到一起，如果可以做到，今后我们使用Ghost备份就再也不用区分机型了，仅仅需要区分语言。但是实际上的Windows部署从来都是不带驱动的，驱动需要自行安装:(

Sysprep简解：
将WinXP\pro\support\tools\delpoy.cab解压到本地文件夹中，将得到10个文件。其中的Sysprep.exe就是我们主要需要的。
下图是运行界面：
![在这里插入图片描述](/2023/02/18/Windows%E4%B8%AD%E7%9A%84SID%E8%AF%A6%E8%A7%A3/0.png)
### 说明
第一个按钮Reseal就是我们所需要的“重新封装”，意即Windows将像重新安装一样去进行配置。
在点击Reseal之前，下方的Flags区域是可选项。Minisetup是我们惟一需要的，“最小化安装”，意即Windows仅保留文件，而不使用任何驱动和设置。如果不选此项，Reseal之后不会重新扫描硬件，而是仅仅重新生成SID。
其它选项，PnP指在重启后扫描非即插即用设备，这年头……似乎不用了吧；NoSIDGen指不生成SID，这是为了便于OEM商在单台计算机上恢复出厂设置用的，不需要重新生成SID；Pre-actived指不使用“展期”功能。

Reseal重启之后会和使用光盘第一次安装一样，要求我们输入各种用户、网络、区域等信息。
系统当前时间等。



<center>

如果您乐意，感谢支持~

![微信赞赏码](/2023/02/18/Windows%E4%B8%AD%E7%9A%84SID%E8%AF%A6%E8%A7%A3/1.png)

![支付宝收款码](/2023/02/18/Windows%E4%B8%AD%E7%9A%84SID%E8%AF%A6%E8%A7%A3/2.png)

</center>
