---
link: https://blog.csdn.net/horizon_rain/article/details/155097615
title: Vulnhub靶机----- 64Base:1.0.1-----WriteUp
description: 文章浏览阅读821次，点赞13次，收藏17次。这个靶机难度还是适中的，关键是要紧扣主题base64编码方式。打靶流程也较为规范，对我这种初学者还有帮助较大。靶机中权限的设置较为有趣。_vulnhub 64base
keywords: linux,新人首发,安全
author: Horizon_rain 博客等级 码龄2年
date: 2026-04-14T08:58:20.436Z
publisher: · AI 阅读助手· AI 阅读助手
stats: paragraph=155 sentences=101, words=178
---
**Base64 编码用处非常广泛**，核心是将二进制数据转成 ASCII 可见字符，解决 "二进制数据在文本场景中传输 / 存储乱码" 的问题，是计算机领域的 "二进制文本转换器"。该靶机以base64编码为主题，以星球大战为背景，趣味性强，且上手容易。该靶机的主要任务是找到6个flag。

## 下载

直接前往Vulnhub官网，搜索64base，找到64base：1.0.1

![](https://i-blog.csdnimg.cn/direct/6a2596d0038d4805bd11c690ccef18ed.png)

![](https://i-blog.csdnimg.cn/direct/d001329b5e4b4aa0834a58078150a50a.png)

下载后得到.ova文件

![](https://i-blog.csdnimg.cn/direct/0d115f2fb59a41a4b01eaf3b167ee051.png)

前往虚拟机打开该文件

![](https://i-blog.csdnimg.cn/direct/180d8180df9d4ed0b73afa13a5ded888.png)

网络适配器方面注意：

1. 作者提示靶机使用仅主机模式，但该模式下主机打开网页速度较慢
2. kali和靶机都设置为同一网段下的NAT模式

最后，打开kali和靶机

靶机如下：

![](https://i-blog.csdnimg.cn/direct/8e37917b051d47b88596c5187b66d562.png)

## flag1

### 1.网络探查

首先侦察一下，靶机的网络情况

使用arp-scan -l命令探查本机所在局域网的活跃主机（这里一般不会出现本机IP），

![](https://i-blog.csdnimg.cn/direct/42838b1f89324e57841bfd66b9471927.png)

netdiscover也可以（同样不出现主机IP），

![](https://i-blog.csdnimg.cn/direct/23980958a64f453a82aa410d2bf7253e.png)

我们发现了4个IP地址，其中以1，2，254结尾的IP地址一般是路由器和网关，我们也可以使用nmap进一步探查这些IP地址的信息

![](https://i-blog.csdnimg.cn/direct/bcbbaacc90384da89fb3d90e32d53e4b.png)

### 2.网站内容查看

其中192.168.153.128是本机IP地址，由此可见只有192.168.153.134主机有开放端口，我们可以假设这既是靶机IP地址，不妨在浏览器中输入其IP，一探究竟

![](https://i-blog.csdnimg.cn/direct/1abf63e246c848a8a07b5d4b22edf800.png)

果然是靶机界面，先从主页搜集信息，主页标题下面，一串长的字符串很可疑，收录

点进HOME的所有文章以及POST板块，都是同一篇文章

![](https://i-blog.csdnimg.cn/direct/c4bc020ca07b46b5996990ee108decdf.png)

在最底部，有两段特殊的话（标黄部分，小字部分），收录

![](https://i-blog.csdnimg.cn/direct/2fc15fae156043449b935e4d318f8aff.png)

在ABOUT板块，是有关天行者阿斯纳的故事，有趣

所以我们至今搜集到的信息如下：

![](https://i-blog.csdnimg.cn/direct/579f2735189d47f5b0bb7c1c75d61dc7.png)

### 3.全端口扫描

三个端口只有80能进（网站主页），这时候我们可以进行全端口扫描，看看有哪些漏网之鱼

![](https://i-blog.csdnimg.cn/direct/c36f81aef4284f72ba5603da17a95ba5.png)

发现一个62964端口，但是发现无法访问

![](https://i-blog.csdnimg.cn/direct/2d267e3115ea4030bb5bf237b67bbec7.png)

可以使用nc或者telnet进行详细的端口扫描。

### 4.目录爆破

端口和网页信息都侦察过了，还可以进行目录爆破，

![](https://i-blog.csdnimg.cn/direct/e4b92f580a5042ddae5b612b39b4e091.png)

可以发现有很多子网页，这些都是作者设置的陷阱

### 5.回顾信息

我们回到搜集到的信息本身64base-dmlldyBzb3VyY2UgO0QK，有什么用？我们注意到64和base总是出现，于是搜索

![](https://i-blog.csdnimg.cn/direct/5693feda9b5941af80c39f99ed14cd30.png)

原来是一种编码方式，我们将得到的信息解码，发现是看源代码

![](https://i-blog.csdnimg.cn/direct/1a374cc1844444f09a36b1ba57392596.png)

找到一串长字符，发现是16进制

![](https://i-blog.csdnimg.cn/direct/799568f683ce4e9b9196a15937e1f00b.png)

转成字符串后，再进行base64解码，得到flag1。

![](https://i-blog.csdnimg.cn/direct/dcde48249c824f86b782674d38d9a0cf.png)

## flag2

### 1.结果分析

思考flag1结果，发现是base64编码，继续解码

![](https://i-blog.csdnimg.cn/direct/e5ab52bb240f47758773ab03b60fc88e.png)

### 2.目录（401）爆破

发现是一个用户名：密码，什么地方能用到？需要登陆的网站。怎么找需要登陆的网站？利用dirb

![](https://i-blog.csdnimg.cn/direct/47a582430bfe4f79b3605eabcfa66bf9.png)

### 3.定制字典

但是只发现了一个admin，而且还登陆不上去，这时我们必须考虑字典的正确性了，因为直接使用dirb会自动使用默认字典，或许我们应该用网页里的内容做一个定制字典，这样可以找到更多子网页。

![](https://i-blog.csdnimg.cn/direct/f3aa4652c7c14a07a1ea0903a4dde522.png)

再使用dirb，配置定制字典，搜寻返回值为401的网页，

![](https://i-blog.csdnimg.cn/direct/252d0760f4294cd1bedba4cd707c08d2.png)

果然，我们找到了一个默认字典不可能找到的子网页，打开后，发现需要用户名密码，

![](https://i-blog.csdnimg.cn/direct/bf1fbba376c342e3a36475fac51ad510.png)

![](https://i-blog.csdnimg.cn/direct/cc29e38fe8e0456caa44fce7aed1211a.png)

根据提示，我们知道网址路径不对，怎么改呢？这时候想起Only respond if you are a real Imperial-Class BountyHunter,构造[http://192.168.153.134/Imperial-Class/BountyHunter/](http://192.168.153.134/Imperial-Class/BountyHunter/ "http://192.168.153.134/Imperial-Class/BountyHunter/")

![](https://i-blog.csdnimg.cn/direct/e1a030f295884acc9ca7252642afaae0.png)

### 4.源代码

发现我们登陆到了其php网页，查看源代码，发现三串编码，好像还都是16进制的

![](https://i-blog.csdnimg.cn/direct/a875918ea9484f76a8b9d7165365ff6e.png)

三串合并转字符串，又是一个base64

![](https://i-blog.csdnimg.cn/direct/1114118b044d4e3c8110d837b198026f.png)

得到flag2

![](https://i-blog.csdnimg.cn/direct/7cc6901463504cb19641d423065e58df.png)

## flag3

### 1.结果分析

![](https://i-blog.csdnimg.cn/direct/9cc00f92f3644823989b1e5bbb563368.png)

显示一个网址，跳转，发现要我们使用burp

![](https://i-blog.csdnimg.cn/direct/655d6e5e835c4bbfa6e3abb5d6c0ed81.png)

### 2.burp抓取

在刚才的网页再使用一次64base:Th353@r3N0TdaDr01DzU@reL00K1ing4登录，用burp抓取request，在response中出现flag3

![](https://i-blog.csdnimg.cn/direct/795614c7d2264735aaa307e0a32c2f96.png)

## flag4

### 1.结果分析

解码得一串URL，在浏览器登陆，记得改成主机IP地址

![](https://i-blog.csdnimg.cn/direct/2a424c2877204cba946a9c22d7d4d631.png)

发现什么都没有

![](https://i-blog.csdnimg.cn/direct/000858808c8d4a5194f5442b90b1f9c7.png)

这是我们想到一个熟悉的单词"exec" ,USE SYSTEM INSTEAD OF EXEC TO RUN THE SECRET 5H377,将URL改为[http://192.168.153.134/Imperial-Class/BountyHunter/login.php?f=system&c=id](http://192.168.153.134/Imperial-Class/BountyHunter/login.php?f=system&c=id "http://192.168.153.134/Imperial-Class/BountyHunter/login.php?f=system&c=id")

![](https://i-blog.csdnimg.cn/direct/410c61d752b541049c68763ddfb066f7.png)

## flag5

### 1.结果分析

![](https://i-blog.csdnimg.cn/direct/3aebb1dfd9ee4188b8a73b9ab918bcb5.png)

### 2.寻找登陆口

又是用户名：密码。这次我们选择/admin登录

![](https://i-blog.csdnimg.cn/direct/745b8d618d154168a570d12d1373bc6f.png)

登陆不进去，还有其他的登录入口吗，有的，还有ssh登录，不过貌似不太行

![](https://i-blog.csdnimg.cn/direct/903fcd297cde4b709b6797c9d7bdf4cb.png)

那我们将64base5h377，编码吧

![](https://i-blog.csdnimg.cn/direct/347d0bdd449349e8b942dfedbb7e3937.png)

成功了！

![](https://i-blog.csdnimg.cn/direct/4874953b74f942ddaed60392ab76ef6d.png)

### 3.探索64base

输入了一些命令行，发现无效

![](https://i-blog.csdnimg.cn/direct/d23496e4f6f84990b61ecce09fcd0bba.png)

这时候我们只有well_done_:D这一个text文件，联想到整个靶机是以base64为主题，我们想到使用base64对这个文件进行编码解码

![](https://i-blog.csdnimg.cn/direct/d4da4f268e3d4762923e8c7215707bf0.png)

这其中的droids吸引了我们的注意力，输入droids，似乎pop root成功了

![](https://i-blog.csdnimg.cn/direct/6304c5f4c6a4486985619aab94733366.png)

![](https://i-blog.csdnimg.cn/direct/3d32453128db4937bd95e72ed9f8b77d.png)

### 4.寻找flag5

我们直接使用find命令找flag5，这里存在一个问题，靶机中的find路径被作者修改为/var/alt-bin/find，展示ascll图，并非标准find，我们应该使用/var/bin/find

![](https://i-blog.csdnimg.cn/direct/e16fe4e6d35b4f258feb6518642659fe.png)

## flag6

### 1.结果分析

我们发现这是一个图片文件

![](https://i-blog.csdnimg.cn/direct/a62f7ae152114bd29230418d3db1b9be.png)

查看图片信息（直接用strings上下文长度很大，因此用/usr/bin/head只展示前几段信息），发现一大段16进制，

![](https://i-blog.csdnimg.cn/direct/cac1591a874e4089b67df30e3a04b4b0.png)

先转成字符，再使用base64解码，得到一份密钥

![](https://i-blog.csdnimg.cn/direct/8a50d237582c4f04b6383b01b6dbea44.png)

在后面使用重定向。得到ssh.key文件（放在/tmp目录下，因为这个文件下所有用户都有写入权限）

![](https://i-blog.csdnimg.cn/direct/9247ee1988ef45e7b799e910e8654fe2.png)

### 2.登录root用户

尝试通过该密钥登录root用户，但还需要密码，我们想到flag5是一张图片，难道你就不想看看长啥样吗？

![](https://i-blog.csdnimg.cn/direct/1977aae2877e4900a06b85beb6e09350.png)

通过scp命令，将flag5传输到kali上，这一步稍微麻烦，从kali上使用scp都会提示Connection

Closed，只有从64base使用scp才能成功传输，这里要注意图片文件的权限要设置成777才能传输

![](https://i-blog.csdnimg.cn/direct/3895651c6b6c4843bd6d39d7f214450c.png)

![](https://i-blog.csdnimg.cn/direct/32dca8c7ae204f74b4abfb90fff53cb1.png)

![](https://i-blog.csdnimg.cn/direct/723c574c8b4746329d8be134422363ce.png)

在kali上查看图片，想到密码可能是usetheforce

![](https://i-blog.csdnimg.cn/direct/b110a4f3eaa74cd9900e2baa3f8f19e1.png)

登录本机root用户，获得flag6，经过多次16进制转字符，以及base64解码，得到base64 -d /var/local/.luke|less.real

得到：

![](https://i-blog.csdnimg.cn/direct/afcf1a7a92934ea7a4e7e1523b22905a.png)

完结！

## 总结

这个靶机难度还是适中的，关键是要紧扣主题base64编码方式。打靶流程也较为规范，对我这种初学者还有帮助较大。靶机中权限的设置较为有趣。

## 后记补充

### 2026-4-11

最近再回来发现当初还是有一些地方没有说清楚，还有一些命令的现代化版本扩展

目录爆破时:

使用更现代化的ffuf,格式：ffuf -u http://ip_address/FUZZ -w dic_address，速度更快

flag5寻找登陆点时：

我们将64base5h377进行base64编码，但这里存在一个问题

![](https://i-blog.csdnimg.cn/direct/80e500be5d6f43af9aae90242e196613.png)

你在网页上编码是上面这种情况，忽略了换行符，得到的密码是错误的，必须像下面这样直接编码

flag6登陆root用户时：

要使用ssh -p 62964 root@192.168.0.159 -i ssh.key

**ssh.key的权限设置为700，因为私钥文件的权限如果太低会报错**

**-i绝对要带上，因为如果你的私钥文件名不是默认的 `id_rsa` ，或者不在 `~/.ssh/` 目录下，你必须显式地用 `-i` 指定路径。**

要先把flag5文件复制到/tmp文件夹再scp传输
