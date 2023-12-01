> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/shuaiqing/p/7742382.html)

实现 linux 定时任务有：cron、anacron、at, 使用最多的是 cron 任务

### 名词解释

　　cron-- 服务名；crond--linux 下用来周期性的执行某种任务或等待处理某些事件的一个守护进程，与 windows 下的计划任务类似；crontab-- 是定制好的计划任务表

### 软件包安装

　　要使用 cron 服务，先要安装 vixie-cron 软件包和 crontabs 软件包，两个软件包作用如下：

　　vixie-cron 软件包是 cron 的主程序。crontabs 软件包是用来安装、卸装、或列举用来驱动 cron 守护进程的表格的程序。

 查看是否安装了 cron 软件包: rpm -qa|grep vixie-cron

　　查看是否安装了 crontabs 软件包: rpm -qa|grep crontabs

　　如果没有安装，则执行如下命令安装软件包 (软件包必须存在)  
　　rpm -ivh vixie-cron-4.1-54.FC5*  
　　rpm -ivh crontabs*

　　如果本地没有安装包，在能够连网的情况下可以在线安装

　　yum install vixie-cron  
　　yum install crontabs

### **查看 crond 服务是否运行**

　　pgrep crond 或 /sbin/service crond status 或 ps -elf|grep crond|grep -v "grep"

### **crond 服务操作命令**

　　/sbin/service crond start // 启动服务    
　　/sbin/service crond stop // 关闭服务    
　　/sbin/service crond restart // 重启服务    
　　/sbin/service crond reload // 重新载入配置

### **配置定时任务**

　　cron 有两个配置文件，一个是一个全局配置文件（/etc/crontab），是针对系统任务的；一组是 crontab 命令生成的配置文件（/var/spool/cron 下的文件），是针对某个用户的. 定时任务配置到任意一个中都可以。

　　查看全局配置文件配置情况: cat /etc/crontab

　　---------------------------------------------  
　　SHELL=/bin/bash  
　　PATH=/sbin:/bin:/usr/sbin:/usr/bin  
　　MAILTO=root  
　　HOME=/  
　　# run-parts  
　　01 * * * * root run-parts /etc/cron.hourly  
　　02 4 * * * root run-parts /etc/cron.daily  
　　22 4 * * 0 root run-parts /etc/cron.weekly  
　　42 4 1 * * root run-parts /etc/cron.monthly  
　　----------------------------------------------

　　查看用户下的定时任务: crontab -l 或 cat /var/spool/cron / 用户名

### **crontab 任务配置基本格式**

　　![](https://images2017.cnblogs.com/blog/987385/201710/987385-20171027113420273-2097862222.png)

　　*   *　 *　 *　 *　　command  
　　分钟 (0-59)　小时 (0-23)　日期 (1-31)　月份 (1-12)　星期 (0-6,0 代表星期天)　 命令  
　　第 1 列表示分钟 1～59 每分钟用 * 或者 */1 表示  
　　第 2 列表示小时 1～23（0 表示 0 点）  
　　第 3 列表示日期 1～31  
　　第 4 列表示月份 1～12  
　　第 5 列标识号星期 0～6（0 表示星期天）  
　　第 6 列要运行的命令

　　在以上任何值中，星号（*）可以用来代表所有有效的值。譬如，月份值中的星号意味着在满足其它制约条件后每月都执行该命令。  
　　整数间的短线（-）指定一个整数范围。譬如，1-4 意味着整数 1、2、3、4。  
　　用逗号（,）隔开的一系列值指定一个列表。譬如，3, 4, 6, 8 标明这四个指定的整数。  
　　正斜线（/）可以用来指定间隔频率。在范围后加上 /<integer> 意味着在范围内可以跳过 integer。譬如，0-59/2 可以用来在分钟字段定义每两分钟。间隔频率值还可以和星号一起使用。例如，*/3 的值可以用在月份字段中表示每三个月运行一次任务。  
　　开头为井号（#）的行是注释，不会被处理

### 使用实例

 　　实例 1：每 1 分钟执行一次 command

　　命令：* * * * * command

　　实例 2：每小时的第 3 和第 15 分钟执行

　　命令：3,15 * * * * command

　　实例 3：在上午 8 点到 11 点的第 3 和第 15 分钟执行

　　命令：3,15 8-11 * * * command

　　实例 4：每隔两天的上午 8 点到 11 点的第 3 和第 15 分钟执行

　　命令：3,15 8-11 */2 * * command

　　实例 5：每个星期一的上午 8 点到 11 点的第 3 和第 15 分钟执行

　　命令：3,15 8-11 * * 1 command

　　实例 6：每晚的 21:30 重启 smb 

　　命令：30 21 * * * /etc/init.d/smb restart

　　实例 7：每月 1、10、22 日的 4 : 45 重启 smb 

　　命令：45 4 1,10,22 * * /etc/init.d/smb restart

　　实例 8：每周六、周日的 1 : 10 重启 smb

　　命令：10 1 * * 6,0 /etc/init.d/smb restart

　　实例 9：每天 18 : 00 至 23 : 00 之间每隔 30 分钟重启 smb 

　　命令：0,30 18-23 * * * /etc/init.d/smb restart

　　实例 10：每星期六的晚上 11 : 00 pm 重启 smb 

　　命令：0 23 * * 6 /etc/init.d/smb restart

　　实例 11：每一小时重启 smb 

　　命令：* */1 * * * /etc/init.d/smb restart

　　实例 12：晚上 11 点到早上 7 点之间，每隔一小时重启 smb 

　　命令：* 23-7/1 * * * /etc/init.d/smb restart

　　实例 13：每月的 4 号与每周一到周三的 11 点重启 smb 

　　命令：0 11 4 * mon-wed /etc/init.d/smb restart

　　实例 14：一月一号的 4 点重启 smb 

　　命令：0 4 1 jan * /etc/init.d/smb restart

　　实例 15：每小时执行 / etc/cron.hourly 目录内的脚本

　　命令：_01   *   *   *   *     root run-parts /etc/cron.hourly_

　　说明：

　　run-parts 这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是目录名了