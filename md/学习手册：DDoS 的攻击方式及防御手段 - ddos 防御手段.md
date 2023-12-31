> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51cto.com](https://www.51cto.com/article/503799.html)

> 最早的时候，黑客们都是大都是为了炫耀个人技能，所以攻击目标选择都很随意，娱乐性比较强。

最早的时候，黑客们都是大都是为了炫耀个人技能，所以攻击目标选择都很随意，娱乐性比较强。后来，有一些宗教组织和商业组织发现了这个攻击的效果，就以勒索、报复等方式为目的，对特定目标进行攻击，并开发一些相应的工具，保证攻击成本降低。当国家级政治势力意识到这个价值的时候，DDoS 就开始被武器化了，很容易就被用于精确目标的网络战争中。

现如今，信息技术的发展为人们带来了诸多便利，无论是个人社交行为，还是商业活动都开始离不开网络了。但是网际空间带来了机遇的同时，也带来了威胁，其中 DDoS 就是最具破坏力的攻击，通过这些年的不断发展，它已经成为不同组织和个人的攻击，用于网络中的勒索、报复，甚至网络战争。

​[​](http://s4.51cto.com/wyfs02/M02/79/89/wKiom1aUXU-RS54pAAGenwoyU0M285.jpg)

[​](http://s4.51cto.com/wyfs02/M02/79/89/wKiom1aUXU-RS54pAAGenwoyU0M285.jpg)​

**先聊聊 DDoS 的概念和发展**

在说发展之前，咱先得对分布式拒绝服务 (DDoS) 的基本概念有个大体了解。

**啥叫 “拒绝服务” 攻击呢?**

其实可以简单理解为：让一个公开网站无法访问。要达到这个目的的方法也很简单：不断地提出服务请求，让合法用户的请求无法及时处理。

**啥叫 “分布式” 呢?**

其实随着网络发展，很多大型企业具备较强的服务提供能力，所以应付单个请求的攻击已经不是问题。道高一尺，魔高一丈，于是乎攻击者就组织很多同伙，同时提出服务请求，直到服务无法访问，这就叫 “分布式”。但是在现实中，一般的攻击者无法组织各地伙伴协同“作战”，所以会使用“僵尸网络” 来控制 N 多计算机进行攻击。

**啥叫 “僵尸网络” 呢?**

就是数量庞大的僵尸程序 (Bot) 通过一定方式组合，出于恶意目的，采用一对多的方式进行控制的大型网络，也可以说是一种复合性攻击方式。因为僵尸主机的数量很大而且分布广泛，所以危害程度和防御难度都很大。

僵尸网络具备高可控性，控制者可以在发布指令之后，就断开与僵尸网络的连接，而控制指令会自动在僵尸程序间传播执行。

![](http://s1.51cto.com/wyfs02/M00/79/88/wKioL1aUXX_RniAGAAB2yAmU7H8202.jpg)

恶意代码类型

这就像个生态系统一样，对于安全研究人员来说，通过捕获一个节点可以发现此僵尸网络的许多僵尸主机，但很难窥其全貌，而且即便封杀一些僵尸主机，也不会影响整个僵尸网络的生存。

**DDoS 的发展咋样?**

正所谓 “以史为鉴，可以知兴替”。既然大概了解 DDoS 是啥了，咱们就说说它的历史发展吧。

最早的时候，黑客们都是大都是为了炫耀个人技能，所以攻击目标选择都很随意，娱乐性比较强。后来，有一些宗教组织和商业组织发现了这个攻击的效果，就以勒索、报复等方式为目的，对特定目标进行攻击，并开发一些相应的工具，保证攻击成本降低。当国家级政治势力意识到这个价值的时候，DDoS 就开始被武器化了，很容易就被用于精确目标的网络战争中。

根据绿盟科技最新的 DDoS 态势分析，从全球的流量分布来看，中国和美国是 DDoS 受灾的重灾区。

**再谈谈 DDoS 的攻击方式**

分布式拒绝服务攻击的精髓是：利用分布式的客户端，向目标发起大量看上去合法的请求，消耗或者占用大量资源，从而达到拒绝服务的目的。

DDoS 攻击

其主要攻击方法有 4 种：

**1、 攻击带宽**

跟帝都的交通堵塞情况一样，大家都该清楚，当网络数据包的数量达到或者超过上限的时候，会出现网络拥堵、响应缓慢的情况。DDoS 就是利用这个原理，发送大量网络数据包，占满被攻击目标的全部带宽，从而造成正常请求失效，达到拒绝服务的目的。

攻击者可以使用 ICMP 洪水攻击 (即发送大量 ICMP 相关报文)、或者 UDP 洪水攻击 (即发送用户数据报协议的大包或小包)，使用伪造源 IP 地址方式进行隐匿，并对网络造成拥堵和服务器响应速度变慢等影响。

但是这种直接方式通常依靠受控主机本身的网络性能，所以效果不是很好，还容易被查到攻击源头。于是反射攻击就出现，攻击者使用特殊的数据包，也就是 IP 地址指向作为反射器的服务器，源 IP 地址被伪造成攻击目标的 IP，反射器接收到数据包的时候就被骗了，会将响应数据发送给被攻击目标，然后就会耗尽目标网络的带宽资源。

**2、 攻击系统**

创建 TCP 连接需要客户端与服务器进行三次交互，也就是常说的 “三次握手”。这个信息通常被保存在连接表结构中，但是表的大小有限，所以当超过了存储量，服务器就无法创建新的 TCP 连接了。

攻击者就是利用这一点，用受控主机建立大量恶意的 TCP 连接，占满被攻击目标的连接表，使其无法接受新的 TCP 连接请求。如果攻击者发送了大量的 TCP SYN 报文，使服务器在短时间内产生大量的半开连接，连接表也会被很快占满，导致无法建立新的 TCP 连接，这个方式是 SYN 洪水攻击，很多攻击者都比较常用。

![](http://s2.51cto.com/wyfs02/M02/79/89/wKiom1aUXVCiZINqAAAweWBXLwA388.jpg)

DDoS 攻击系统

**3、 攻击应用**

由于 DNS 和 Web 服务的广泛性和重要性，这两种服务就成为了消耗应用资源的分布式拒绝服务攻击的主要目标。

比如向 DNS 服务器发送大量查询请求，从而达到拒绝服务的效果，如果每一个 DNS 解析请求所查询的域名都是不同的，那么就有效避开服务器缓存的解析记录，达到更好的资源消耗效果。当 DNS 服务的可用性受到威胁，互联网上大量的设备都会受到影响而无法正常使用。

近些年，Web 技术发展非常迅速，如果攻击者利用大量的受控主机不断地向 Web 服务器恶意发送大量 HTTP 请求，要求 Web 服务器处理，就会完全占用服务器资源，让正常用户的 Web 访问请求得不到处理，导致拒绝服务。一旦 Web 服务受到这种攻击，就会对其承载的业务造成致命的影响。

**4、 混合攻击**

在实际的生活中，乖哦概念记者并不关心自己使用的哪种攻击方法管用，只要能够达到目的，一般就会发动其所有的攻击手段，尽其所能的展开攻势。对于被攻击目标来说，需要面对不同的协议、不同资源的分布式拒绝服务攻击，分析、响应和处理的成本就会大大增加。

随着僵尸网络向着小型化的趋势发展，为降低攻击成本，有效隐藏攻击源，躲避安全设备，同时保证攻击效果，针对应用层的小流量慢速攻击已经逐步发展壮大起来。因此，从另一个角度来说，DDoS 攻击方面目前主要是两个方面：UDP 及反射式大流量高速攻击、和多协议小流量及慢速攻击。

HACKER

**也说说 DDoS 的攻击工具**

国人比较讲究：工欲善其事必先利其器。随着开源的 DDoS 工具扑面而来，网络攻击变得越来越容易，威胁也越来越严重。 工具有很多，简单介绍几款知名的，让大家有个简单了解。

**LOIC**

![](http://s5.51cto.com/wyfs02/M01/79/89/wKiom1aUXVDifHD5AAA8E9mMFb8348.jpg)

LOIC

LOIC 低轨道离子炮，是一个最受欢迎的 DOS 攻击的淹没式工具，会产生大量的流量，可以在多种平台运行，包括 Linux、Windows、Mac OS、Android 等等。早在 2010 年，黑客组织对反对维基解密的公司和机构的攻击活动中，该工具就被下载了 3 万次以上。

LOIC 界面友好，易于使用，初学者也可以很快上手。但是由于该工具需要使用真实 IP 地址，现在 Anonymous 已经停用了。

**HULK (HTTP Unbearable Load King)**

![](http://s4.51cto.com/wyfs02/M02/79/88/wKioL1aUXX_ROhbpAAArhnbuL58941.jpg)

HULK

HULK 是另一个 DOS 攻击工具，这个工具使用 UserAgent 的伪造，来避免攻击检测，可以通过启动 500 线程对目标发起高频率 HTTP GET FLOOD 请求，牛逼的是每一次请求都是独立的，可以绕过服务端的缓存措施，让所有请求得到处理。HULK 是用 Python 语言编写，对获得源码进行更改也非常方便。

**R.U.D.Y.**

R-U-Dead-Yet 是一款采用慢速 HTTP POST 请求方式进行 DOS 攻击的工具，它提供了一个交互式控制台菜单，检测给定的 URL，并允许用户选择哪些表格和字段应用于 POST-based DOS 攻击，操作非常简单。

R.U.D.Y.

而且它也使用的是 Python 语言编写，可移植性非常好。R.U.D.Y. 能够对所有类型的 Web 服务端软件造成影响，因此攻击的威胁非常大。

这些工具在保持攻击力的同时还再加强易用性，而免费和开源降低了使用的门槛，相信随着攻防对抗的升级，工具会越来越智能化。

**最后唠唠 DDoS 的防御**

我的导师教过我：DDoS 攻击只是手段，最终目的是永远的利益。而未来网络战争将会出现更加广泛的攻击、更加频繁的攻击和更加精准的攻击，面对这些来临的时候，我们应该如何应对?

DDoS 的防御

**设置高性能设备**

要保证网络设备不能成为瓶颈，因此选择路由器、交换机、硬件防火墙等设备的时候要尽量选用知名度高、口碑好的产品。再就是假如和网络提供商有特殊关系或协议的话就更好了，当大量攻击发生的时候请他们在网络接点处做一下流量限制来对抗某些种类的 DDoS 攻击是非常有效的。

**带宽得保证**

网络带宽直接决定了能抗受攻击的能力，假若仅仅有 10M 带宽的话，无论采取什么措施都很难对抗现在的 SYN Flood 攻击。所以，最好选择 100M 的共享带宽，当然是挂在 1000M 的主干上了。

**不要忘记升级**

在有网络带宽保证的前提下，请尽量提升硬件配置，要有效对抗每秒 10 万个 SYN 攻击包。而且最好可以进行优化资源使用，提高 web server 的负载能力。

**异常流量的清洗**

通过 DDoS 硬件防火墙对异常流量的清洗过滤，通过数据包的规则过滤、数据流指纹检测过滤、及数据包内容定制过滤等顶尖技术能准确判断外来访问流量是否正常，进一步将异常流量禁止过滤。

**考虑把网站做成静态页面**

把网站尽可能做成静态页面，不仅能大大提高抗攻击能力，而且还给黑客入侵带来不少麻烦，最好在需要调用数据库的脚本中，拒绝使用代理的访问，经验表明，使用代理访问你网站的 80% 属于恶意行为。

**分布式集群防御**

这是目前网络安全界防御大规模 DDoS 攻击的最有效办法。分布式集群防御的特点是在每个节点服务器配置多个 IP 地址，并且每个节点能承受不低于 10G 的 DDoS 攻击，如一个节点受攻击无法提供服务，系统将会根据优先级设置自动切换另一个节点，并将攻击者的数据包全部返回发送点，使攻击源成为瘫痪状态，从更为深度的安全防护角度去影响企业的安全执行决策。

就 DDoS 防御方面来说，目前主要是两个方面，大流量攻击可以交给运营商及云端清洗，小流量攻击可以在企业本地进行设备防护，这个分界点根据行业及业务特性的不同会有所差异，大概的量级应该在百兆 BPS 左右。相关的缓解与治理，有兴趣的童鞋可以看看鲍旭华的《破坏之王》，会有不小的启示。

![](http://s1.51cto.com/wyfs02/M02/79/89/wKiom1aUXVDh-r4SAABH2n9IKsg298.jpg)

DDoS 的防御

其实，对抗 DDoS 攻击是一个涉及多个层面的问题，在有的环节，有效性和收益率并不对等。所以需要各方面合作，用户也可以多多听听专家的意见，针对攻击事先做好应对的应急方案。有句话说：“god helps those who help themselves.” 意思是，上帝只帮助那些自助的人，因此面对 DDoS 的攻击，大家需要具备安全意识，完善自身的安全防护体系才是正解。

**写在最后的话**

随着全球互联网业务和云计算的发展热潮，可以预见到，针对云数据中心的 DDoS 攻击频率还会大幅度增长，攻击手段也会更加复杂。安全工作是一个长期持续性而非阶段性的工作，所以需要时刻保持一种警觉，而且网络安全不仅仅是某个企业的责任，更是全社会的共同责任，需要大家共同努力。

本文从概念、发展、攻击方式、攻击工具以及防御等各方面全面阐述了 DDoS，是小编在学习过程中的心得和浅析。开始学习以来，一直有个问题萦绕不去：为什么自从 DDoS 出现以后，虽然攻击形式简单，却屡禁不止?? 希望可以和大家共同探讨。