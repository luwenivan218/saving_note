> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7328744761954828328)

### 1、权限管理概述

Linux 通过 rwx3 种权限控制系统与保护系统, 组成 9 位权限.

Linux 权限体系中还有 3 位特殊权限, 组合起来就是 12 位权限体系.

Linux 这简单的 rwx 控制整个 Linux 系统的安全, 权限与用户共同组成 Linux 系统的安全防护体系.

### 2、linux 权限计算

<table><thead><tr><th>权限</th><th></th></tr></thead><tbody><tr><td>r</td><td>read 是否可读</td></tr><tr><td>w</td><td>write 是够可写</td></tr><tr><td>x</td><td>execute 是否可以执行（一般是命令脚本）</td></tr></tbody></table>

Linux 下面任何一个文件 / 目录与用户的关系有 3 种关系

<table><thead><tr><th>文件 / 目录与用户的关系</th><th>含义</th></tr></thead><tbody><tr><td>所有者</td><td>这个文件或目录属于某个用户 (所有者)。</td></tr><tr><td>用户组（家庭）</td><td>你自己这个文件或目录属于某个用户组 (家庭)。家人。</td></tr><tr><td>其他人（陌生人）</td><td>这个文件或目录不属于某个用户 也不属于这个用户组。隔壁老王。</td></tr></tbody></table>

人们为了更加方便的使用权限，于是给每个权限字母设置了一个对应的数字，通过数字表示对应的权限。

<table><thead><tr><th>文件 / 目录与用户的关系</th><th>含义</th><th></th></tr></thead><tbody><tr><td>r</td><td>是否可读</td><td>4</td></tr><tr><td>W</td><td>是够可写</td><td>2</td></tr><tr><td>X</td><td>是否可执行</td><td>1</td></tr><tr><td>-</td><td>没有权限</td><td>0</td></tr></tbody></table>

<table><thead><tr><th>7 全部权限</th><th>6 不可以执行</th><th>5 不可以写</th><th>4 只能读</th><th></th></tr></thead></table>

`chmod`命令：change mode 使用数字或字母形式修改权限 `chown`命令：change owner 修改文件所有者，用户组。`-R`循环赋予权限给文件夹下面（慎用）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fffda156fae949f5a1d8e6c9a10f59cc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=853&h=248&s=22902&e=png&b=a9e9f0)

```
#all可以省略
chmod +x 1.txt
chmod a+x 1.txt


```

### 3、linux 权限测试

> 对于文件而言

对于文件来说只有 w 权限不够，需要有 r 权限配合

如果文件只有 w 只能通过追加方式写入，如果 vi/vim 写入会清空文件内容只留最新的 (:wg!)。

x 权限也需要有 r 配合

> 对于目录而言

目录的 r 权限查看目录下内容, 如果只有 r 目录下文件的属性信息无法查看提示 "?", 目录的 r 权限需要 x 权限配合.

对于目录 x 权限表示是否能够进入目录权限，是否能够查看与修改目录下文件的 (属性信息) 权限。

目录的 w 权限表示在目录下面创建，删除，重命名文件，只有 w 还不够，需要? 配合

对于目录 x 权限表示是否能够进入目录权限，是否能够查看与修改目录下文件的 (属性信息) 权限。

实际应用建议: 如果要对某个目录拥有 “写 " 权限，则授予目录 rwx 即可。

> 小结

<table><thead><tr><th>权限</th><th>文件</th><th>目录</th></tr></thead><tbody><tr><td>r</td><td>是否可以读取文件内容 (r)</td><td>是否可以查看目录内容，需要 x 权限配合 (rx)</td></tr><tr><td>w</td><td>是否可以修改文件内容，一般还需要 r 权限配合.(rw)</td><td>是否可以在目录中创建，删除，重命名文件权限，需要 rx 权限配合 (rwx)</td></tr><tr><td>x</td><td>是否可以执行文件，(命令，脚本)，一般还需要 r 权限配合（rx，rwx）</td><td>是否可以进入目录，是否可以访问目录下文件属性 (rx，rwx)</td></tr></tbody></table>

### 4、删除文件需要什么权限

```
mkdir shishu
chown shishuwu:shishuwu shishu#这里不加-R
touch shishu/root{01..10}.txt


```

问：shishuwu 能否删除 shishu 下面属于 root 的文件? 解释：删除文件看文件所在目录权限 mode-dir 目录 755, 所有者是 shishuwu 可以删除

```
[root@nanjing shishuwu]# ll
-rwxrwxrwx 1 root root    0 1月  28 20:39 shishuwu.txt
[root@nanjing shishuwu]# ls -dl /shishuwu/
drwxr-xr-x 3 root root 4096 1月  28 20:39 /shishuwu/
[shishuwu@nanjing ~]$ \rm /shishuwu/shishuwu.txt 
rm: cannot remove ‘/shishuwu/shishuwu.txt’: Permission denied
#因目录为没有w权限 所以无法删除.


```

> 小结：
> 
> 要删除一个文件，不仅需要对该文件所在的目录有读写权限，还需要对文件本身有删除权限。如果文件属于其他用户，即使目录对您有读写权限，您也可能无法删除该文件，除非您是超级用户或文件的所有者。

### 5、Permission denied 故障排查

分析权限拒绝的流程

缕清用户与文件 / 目录权限关系，你要知晓你对于这个文件或目录拥有什么权限?

分析缺少了什么权限导致的问题? 根据操作分析是与文件的权限有关，还是目录的权限有关?

得出结论，缺少了文件 xxxx 权限，目录的 xxxx 权限导致的故障.

<table><thead><tr><th>日常操作</th><th>需要的权限</th></tr></thead><tbody><tr><td>查看文件的内容</td><td>文件要有 r 权限</td></tr><tr><td>编辑或修改文件内容</td><td>文件要有 rw 权限</td></tr><tr><td>执行脚本 / 命令</td><td>文件需要有 rx 权限</td></tr><tr><td>查看目录内容</td><td>目录要有 rx 权限</td></tr><tr><td>创建文件，删除文件</td><td>文件所在目录要有 rwx 权限</td></tr><tr><td>重命名</td><td>文件所在目录要有 rwx 权限</td></tr></tbody></table>

### 6、系统默认权限 (了解)

Linux 系统通过 umask 命令控制文件和目录的默认权限.

如何控制的？

> 普通文件最大权限（缺少执行权限）: 666
> 
> 普通目录最大权限: 777
> 
> 减去 umask 的值, 文件 umask 如果某一位是奇数, 需要减去 umask 后这一位上 + 1

```
[root@nanjing ]# umask
0022


```

得出 root 用户创建文件和目的权限

文件：644；目录：755

```
[root@nanjing test]# touch moren.txt
[root@nanjing test]# mkdir moren
[root@nanjing test]# ll
总用量 4
drwxr-xr-x 2 root root 4096 1月  28 23:17 moren
-rw-r--r-- 1 root root    0 1月  28 23:17 moren.txt


```

### 7、Linux 权限控制与系统安全 (了解)

通过权限控制让系统安全:

*   搭建网站来说，服务器权限设置

*   最小化原则: 既要保证网站可以正常访问，也要保证网站安全

推荐的网站的权限配置为

*   文件 644 root.root

*   目录 755 root.root

网站在运行的时候需要用户: 这个用户不推荐是 root，推荐自己 / 自动创建虚拟用户 www/nginx.

#### 1）单台机器

网站运行的时候是 www 用户, 网站程序代码 / app/code/www 目录, 为例如何设置权限?

```
#01./app/code/www目录
文件和目录所有者 root root(查看权限)
文件和目录权限   644 755
​
#02. /app/code/www/upload 上传目录
#如果不修改,则用户无法上传文件到upload目录下面
文件和目录所有者 www www 
文件和目录权限644 755
​
#03:控制用户上传指定类型的文件
sh、py等执行、破坏、垃圾文件
#04.只能上传，不能执行


```

#### 2）集群

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e8b406c452b4f95be363cf90cd897a2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2030&h=1174&s=385782&e=png&b=b3ebf1)

### 8、3 个特殊权限

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/801aab93d0e14bbfa8c268487d63e2d5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1824&h=272&s=194334&e=png&b=000000)

<table><thead><tr><th>特殊权限</th><th>判断</th><th>含义</th><th>例子</th></tr></thead><tbody><tr><td>set uid=suid</td><td>命令 u 的位置上有个 s 或 S，对应权限 4</td><td>运行这个命令的时候相当于是这个命令的所有者的权限</td><td>passwd</td></tr><tr><td>sticky</td><td>目录的 o 的位置上有个 t，对应的权限数字 1</td><td>对于包含 sticky 权限的目录，每个用户都可以在目录下面创建内容，但是每个用户只能管理自己的文件.</td><td>/tmp/</td></tr><tr><td>set gid =sgid</td><td>命令的 g 的位置上有个 s 或 S，对应的权限数字 2</td><td>运行这个命令的时候相当于是这个命令的用户组的权限</td><td>常用暂无</td></tr></tbody></table>

12 位权限对应 4 位数字，第一位无特殊权限默认是 0

```
[root@nanjing ~]# stat /tmp
  文件："/tmp"
  大小：4096            块：8          IO 块：4096   目录
设备：fd01h/64769d      Inode：8193        硬链接：9
权限：(1777/drwxrwxrwt)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2024-01-28 10:06:38.701953788 +0800
最近更改：2024-01-28 23:15:40.652951330 +0800
最近改动：2024-01-28 23:15:40.652951330 +0800
创建时间：-
[root@nanjing ~]# stat /root/test/moren
  文件："/root/test/moren"
  大小：4096            块：8          IO 块：4096   目录
设备：fd01h/64769d      Inode：1180625     硬链接：2
权限：(0755/drwxr-xr-x)  Uid：( 1000/shishuwu)   Gid：( 1000/shishuwu)
最近访问：2024-01-28 23:17:17.373062218 +0800
最近更改：2024-01-28 23:17:17.373062218 +0800
最近改动：2024-01-28 23:31:05.345002814 +0800
创建时间：-


```

### 9、Linux 特殊属性

目的: 预防重要文件或命令被修改.

`lsattr` 查看这种特殊属性

`chatrr` 修改这种特殊属性

*   `a`属性 append 只能追加
*   `i`属性 immutable 不朽的, 无法被毁灭的.

```
chattr +a oldboy.txt #删除权限-a 
chattr +i oldboy.txt #删除权限-i


```