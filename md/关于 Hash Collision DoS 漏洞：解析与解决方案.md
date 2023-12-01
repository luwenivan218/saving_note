> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.iteye.com](https://www.iteye.com/news/23939/)

最近一个 Hash Collision DoS（Hash 碰撞的拒绝式服务攻击）漏洞影响颇大，有恶意的人会通过这个安全弱点会让你的服务器运行巨慢无比，本文试图对这一漏洞的原理及可采取措施做一解析，供大家参考。

一言蔽之，

**该安全弱点利用了各语言的 Hash 算法的 “非随机性” 可以制造出 N 多的 value 不一样，但是 key 一样数据，然后让你的 Hash 表成为一张单向链表，而导致你的整个网站或是程序的运行性能以级数下降（可以很轻松地让你的 CPU 升到 100%）。**

目前，这个问题出现于 Java、JRuby、PHP、Python、Rubinius、Ruby 这些语言中，主要有：

*   [Java](http://www.java.com/)，所有版本
*   [JRuby](http://jruby.org/) <= 1.6.5（目前 fix 在 1.6.5.1）
*   [PHP](http://www.php.net/) <= 5.3.8，<= 5.4.0RC3（目前 fix 在 5.3.9、5.4.0RC4）
*   [Python](http://python.org/)，所有版本
*   [Rubinius](http://rubini.us/)，所有版本
*   [Ruby](http://www.ruby-lang.org/) <= 1.8.7-p356（目前 fix 在 1.8.7-p357、1.9.x）
*   [Apache Geronimo](http://geronimo.apache.org/)，所有版本
*   [Apache Tomcat](http://tomcat.apache.org/) <= 5.5.34，<= 6.0.34，<= 7.0.22（目前 fix 在 5.5.35、6.0.35、7.0.23）
*   [Oracle Glassfish](http://glassfish.java.net/) <= 3.1.1（目前 fix 在 mainline）
*   [Jetty](http://www.eclipse.org/jetty/)，所有版本
*   [Plone](http://plone.org/)，所有版本
*   [Rack](http://rack.rubyforge.org/) <= 1.3.5, <= 1.2.4, <= 1.1.2 （目前 fix 在 1.4.0、1.3.6、1.2.5、1.1.3）
*   [V8 JavaScript Engine](http://code.google.com/p/v8/)，所有版本
*   ASP.NET，没有打 MS11-100 补丁之前

注意，Perl 没有这个问题，因为 Perl 在 N 年前就 fix 了这个问题了。关于这个列表的更新，请参看

[oCERT 的 2011-003 报告](http://www.ocert.org/advisories/ocert-2011-003.html)

。这个问题早在 2003 年就在论文《

[通过算法复杂性进行拒绝式服务攻击](http://www.cs.rice.edu/~scrosby/hash/CrosbyWallach_UsenixSec2003.pdf)

》中被报告了，但是好像没有引起注意，尤其是 Java。

**弱点攻击解释**

你可能会觉得这个问题没有什么大不了的，因为黑客看不到 hash 算法，如果你这么认为，那么你就错了，这说明对 Web 编程的了解还不足够底层。

无论你用 JSP、PHP、Python、Ruby 来写后台网页的时候，在处理 HTTP POST 数据的时候，你的后台程序可以很容易地以访问表单字段名来访问表单值，就像下面这段程序一样：

Javascript 代码 

1.  $usrname = $_POST['username'];  
2.  $passwd = $_POST['password'];  

```
$usrname = $_POST['username'];
$passwd = $_POST['password'];

```

这是怎么实现的呢？这后面的东西就是 Hash Map 啊，所以，我可以给你后台提交一个有 10K 字段的表单，这些字段名都被我精心地设计过，他们全是 Hash Collision ，于是你的 Web Server 或语言处理这个表单的时候，就会建造这个 hash map，于是在每插入一个表单字段的时候，都会先遍历一遍你所有已插入的字段，于是你的服务器的 CPU 一下就 100% 了，你会觉得这 10K 没什么，那么我就发很多个的请求，你的服务器一下就不行了。

举个例子，你可能更容易理解：

如果你有 n 个值—— v1, v2, v3, … vn，把他们放到 hash 表中应该是足够散列的，这样性能才高：

引用    0 -> v2  
    1 -> v4  
    2 -> v1  
    …  
    …  
    n -> v(x)

但是，这个攻击可以让我造出 N 个值——  dos1, dos2, …., dosn，他们的 hash key 都是一样的（也就是 Hash Collision），导致你的 hash 表成了下面这个样子：

引用    0 – > dos1 -> dos2 -> dos3 -> …. ->dosn  
    1 -> null  
    2 -> null  
    …  
    …  
    n -> null

于是，单向链接就这样出现了。这样一来，O(1) 的搜索算法复杂度就成了 O(n)，而插入 N 个数据的算法复杂度就成了 O(n^2)，你想想这是什么样的性能。

（关于 Hash 表的实现，如果你忘了，那就把大学时的《数据结构》一书拿出来看看）

**Hash Collision DoS 详解**

StackOverflow.com 是个好网站，合格的程序员都应该知道这个网站。上去一查，就看到了这个贴子 “

[Application vulnerability due to Non Random Hash Functions](http://stackoverflow.com/questions/8669946/application-vulnerability-due-to-non-random-hash-functions)

”。这里把这个贴子里的东西摘一些过来。

首先，这些语言使用的 Hash 算法都是 “非随机的”，如下所示，这个是 Java 和 Oracle 使用的 Hash 函数：

Java 代码 

1.  static int hash(int h)  
2.  {  
3.  h ^= (h>>> 20) ^ (h>>> 12);  
4.  return h ^ (h>>> 7) ^ (h>>> 4);  
5.  }  

```
static int hash(int h)
{
h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);
}

```

所谓 “非随机的” Hash 算法，就可以猜。比如：

1）在 Java 里，Aa 和 BB 这两个字符串的 hash code（或 hash key）是一样的，也就是 Collision 。

2）于是，我们就可以通过这两个种子生成更多的拥有同一个 hash key 的字符串。如：”AaAa”, “AaBB”, “BBAa”, “BBBB”。这是第一次迭代。其实就是一个排列组合，写个程序就搞定了。

3）然后，我们可以用这 4 个长度的字符串，构造 8 个长度的字符串，如下所示：

引用  
"AaAaAaAa", "AaAaBBBB", "AaAaAaBB", "AaAaBBAa",  
"BBBBAaAa", "BBBBBBBB", "BBBBAaBB", "BBBBBBAa",  
"AaBBAaAa", "AaBBBBBB", "AaBBAaBB", "AaBBBBAa",  
"BBAaAaAa", "BBAaBBBB", "BBAaAaBB", "BBAaBBAa",

4）同理，我们就可以生成 16 个长度的，以及 256 个长度的字符串，总之，很容易生成 N 多的这样的值。

在攻击时，我只需要把这些数据做成一个 HTTP POST 表单，然后写一个无限循环的程序，不停地提交这个表单。你用你的浏览器就可以了。当然，如果做得更精妙一点的话，把你的这个表单做成一个跨站脚本，然后找一些网站的跨站漏洞，放上去，于是能过 SNS 的力量就可以找到 N 多个用户来帮你从不同的 IP 来攻击某服务器。

**防御措施**

要防守这样的攻击，可以尝试下面几招：

*   打补丁，把 hash 算法改了。
*   限制 POST 的参数个数，限制 POST 的请求长度。
*   最好还有防火墙检测异常的请求。