> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51cto.com](https://www.51cto.com/article/442756.html)

> 随着网络上免费的可用 DDoS 工具增多，DoS 攻击日益增长，本文介绍几款 Hacker 常用的 DoS 攻击工具。

DoS(Denial Of Service) 攻击是指故意的攻击网络协议实现的缺陷或直接通过野蛮手段残忍地耗尽被攻击对象的资源，目的是让目标计算机或网络无法提供正常的服务或资源访问，使目标系统服务系统停止响应甚至崩溃 (关于 DDoS 更多认识请点击这里)。然而随着网络上免费的可用 DDoS 工具增多，DoS 攻击也日益增长，下面介绍几款 Hacker 常用的 DoS 攻击工具。

**特别提示：仅用于攻防演练及教学测试用途，禁止非法使用。**

**1、卢瓦 (LOIC) (Low Orbit Ion Canon)**

LOTC 是一个最受欢迎的 DOS 攻击工具。 这个工具被去年流行的黑客集团匿名者用于对许多大公司的网络攻击。

它可以通过使用单个用户执行 DOS 攻击小型服务器，工具非常易于使用，即便你是一个初学者。 这个工具执行 DOS 攻击通过发送 UDP,TCP 或 HTTP 请求到受害者服务器。 你只需要知道服务器的 IP 地址或 URL，其他的就交给这个工具吧。

[![](http://s1.51cto.com/wyfs02/M00/2F/51/wKioL1OfoB-Dw3KgAADLZq_sQ7I372.jpg)](http://s1.51cto.com/wyfs02/M00/2F/51/wKioL1OfoB-Dw3KgAADLZq_sQ7I372.jpg)

下载卢瓦 LOIC: http://sourceforge.net/projects/loic/

**2、XOIC**

XOIC 是另一个不错的 DOS 攻击工具。它根据用户选择的端口与协议执行 DOS 攻击任何服务器。XOIC 开发者还声称 XOIC 比上面的 LOIC 在很多方面更强大呢。

一般来说, 该工具有三种攻击模式, 第一个被称为测试模式，是非常基本的; 第二个是正常的 DOS 攻击模式; 最后一个是带有 HTTP / TCP / UDP / ICMP 消息的 DOS 攻击模式,。

对付小型网站来说，这是一个很有效的 DDOS 工具。 但是从来没有尝试的要小心点，你可能最终会撞自己的网站的服务器。

[![](http://s2.51cto.com/wyfs02/M01/2F/51/wKioL1OfoC7xO03dAAC2tcT16B4052.jpg)](http://s2.51cto.com/wyfs02/M01/2F/51/wKioL1OfoC7xO03dAAC2tcT16B4052.jpg)

下载 XOIC: http://sourceforge.net/projects/xoic/

**3、HULK (HTTP Unbearable Load King)**

HULK 是另一个不错的 DOS 攻击工具，这个工具使用某些其他技术来避免通过攻击来检测。它有一个已知的用户代理列表，且使用的是随机请求。

在这里下载 HULK: http://packetstormsecurity.com/files/112856/HULK-Http-Unbearable-Load-King.html

**4、 DDOSIM-Layer**

DDOSIM 是另一种流行的 DOS 攻击工具。 顾名思义, 它是通过模拟控制几个僵尸主机执行 DDOS 攻击。所有僵尸主机创建完整的 TCP 连接到目标服务器。

这个工具是用 c++ 写的, 并且在 Linux 系统上运行。

这些是 DDOSIM 的主要特点：

模拟几个僵尸攻击

随机的 IP 地址

TCP-connection-based 攻击

应用程序层 DDOS 攻击

HTTP DDos 等有效的请求

与无效请求 HTTP DDoS(类似于直流 + + 攻击)

SMTP DDoS

TCP 洪水连接随机端口

在这里下载 DDOSIM: http://sourceforge.net/projects/ddosim/

阅读更多关于此工具: http://stormsecurity.wordpress.com/2009/03/03/application-layer-ddos-simulator/

**5、R-U-Dead-Yet**

R-U-Dead-Yet 是一个 HTTP post DOS 攻击工具。它执行一个 DOS 攻击长表单字段，通过 POST 方法提交。 这个工具提供了一个交互式控制台菜单，检测给定的 URL, 并允许用户选择哪些表格和字段应用于 POST-based DOS 攻击。

下载: https://code.google.com/p/r-u-dead-yet/

**6、 Tor's hammer**

Tor'hammer 是另一个不错的 DOS 测试工具。 它是用 Python 编写的。 这个工具有一个额外的优势: 它可以通过 TOR 匿名网络执行攻击。 这是一个非常有效的工具, 它可以在几秒内杀了 Apache 和 IIS 服务器。

下载 TOR'Hummer: http://packetstormsecurity.com/files/98831/

**7、 PyLoris**

据说 PyLoris 是服务器的测试工具。它也可以用来执行 DOS 攻击。 这个工具可以利用 SOCKS 代理和 SSL 连接服务器上执行 DOS 攻击。它可以针对各种协议, 包括 HTTP、FTP、SMTP、IMAP,Telnet。不像其他传统 DOS 攻击工具一样，其最新版本的软件添加了一个简单易用的 GUI。

下载 PyLoris: http://sourceforge.net/projects/pyloris/

**8、OWASP DOS HTTP POST**

这是另外一个很好的工具。您可以使用这个工具来检查您的 web 服务器能否够捍卫得住别人的 DOS 攻击。当然，不仅对防御，它也可以用来执行 DOS 攻击哦。

下载: https://code.google.com/p/owasp-dos-http-post/

**9、DAVOSET**

DAVOSET 是另一个很好的执行 DDOS 攻击工具。 最新版本的工具新增支持 cookie 以及许多其他功能。 您可以从 Packetstormsecurity DAVOSET 免费下载。

下载 DavoSET: http://packetstormsecurity.com/files/123084/DAVOSET-1.1.3.html

**10、黄金眼 (GoldenEye)HTTP 拒绝服务工具**

黄金眼也是一个简单但有效的 DOS 攻击工具。 这是在 Python 测试开发的 DOS 攻击, 但是人们也把它当做一种黑客工具了。

下载: http://packetstormsecurity.com/files/120966/GoldenEye-HTTP-Denial-Of-Service-Tool.html

欢迎补充，如有不对之处欢迎指正:)

原文地址：http://resources.infosecinstitute.com/dos-attacks-free-dos-attacking-tools/