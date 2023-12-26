> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316532841923837952)

数据库管理员的日常任务之一是保持 MySQL 服务器正常运行。他们还必须确保数据库保持健康。

然而，即使采取了所有预防措施，也会出现可能导致数据库损坏的情况。

MySQL 数据库损坏的原因有多种，例如服务器故障、硬件或软件问题、病毒攻击等。

当数据库损坏时，DBA 面临的最大挑战是避免停机，因为这可能会导致生产力和业务损失。

因此，尽快修复损坏的数据库以最大限度地减少停机时间并防止业务损失非常重要。

在本指南中，我们将介绍 MySQL 数据库损坏的原因，并提及一些快速修复数据库的有效解决方案。

我们还分享了一些防止 MySQL 数据库损坏的最佳实践。

**MySQL 数据库损坏的原因**
==================

MySQL 数据库可能因以下因素而损坏：

*   **MySQL 服务器故障**

如果 MySQL 服务器由于突然断电或任何其他原因而发生故障，则可能会扰乱数据完整性，从而导致损坏。

*   **硬件故障**

硬件故障或问题也可能导致数据库损坏。

*   **MySQL 应用程序在写入过程中被终止**

如果 MySQL（单个多线程程序）由于磁盘故障或任何应用程序级问题而在写入操作过程中突然终止或终止，则可能会导致数据库损坏。

*   **不兼容的数据类型 / 事务**

如果您错误地使用外部实用程序来修改表，数据库可能会损坏。例如，不正确的事务 / 数据类型。

*   **MySQL 存储引擎配置不正确**

MySQL 存储引擎 (MyISAM/InnoDB) 中的错误代码或配置可能会导致数据库损坏。

*   **有缺陷的应用程序 / 更新**

MySQL 数据库的损坏也可能由于应用程序错误或操作系统更新而发生。

*   **磁盘空间不足**

当磁盘空间不足时，MySQL Server 可能无法执行读写操作，从而导致数据库损坏。

*   **人为错误**

人为错误（例如运行不正确的 SQL 语句和删除关键文件）可能会损坏数据库。

**如何修复损坏的 MySQL 数据库？**
======================

请按照以下故障排除方法修复和恢复损坏的 MySQL 数据库。

**方法 1 - 从备份恢复 MySQL 数据库**
--------------------------

如果数据库损坏，最简单的方法是从上次已知的备份中恢复数据库副本。

如果您已使用 mysqldump 实用程序创建了逻辑备份，请按照以下步骤进行恢复数据库：

首先，使用以下命令创建一个空数据库：

```
mysql > create db_name


```

然后，使用以下命令恢复数据库：

```
mysql -u root -p db_name < dump.sql


```

此处， u = 用户名

p = 密码

db_name = 数据库名称

dump.sql = 转储文件的路径

现在，使用以下命令检查数据库中恢复的对象：

```
mysql> use db_name; mysql > show tables;


```

**方法 2 - 使用 Mysqlcheck 命令检查和修复 MySQL 数据库**
------------------------------------------

如果您尚未创建任何备份或备份已过时，请使用 mysqlcheck 命令检查并修复中的表 MySQL 数据库。

此命令使用 SQL 语句列表来检查和修复数据库表。步骤如下：

在托管 MySQL 服务器的系统上打开命令行终端。

首先，检查 MySQL 数据库。要检查特定数据库，请运行以下命令：

```
mysqlcheck database_name


```

如果要检查数据库中的特定表，请运行以下命令：

```
mysqlcheck database_name table_name


```

如果数据库中的特定表已损坏，请使用以下命令修复该表：

```
mysqlcheck --r database_name table_name


```

要修复所有数据库，请使用以下命令：

```
mysqlcheck --repair --all-databases


```

**方法 3 - 修复 MySQL 数据库**
-----------------------

MySQL 存储引擎管理数据在数据库中的存储、管理和操作方式。MySQL 支持两种类型的存储引擎——InnoDB 和 MyISAM。

您可以根据您的数据库存储引擎，按照以下步骤修复 MySQL 数据库：

### **A - 如果您使用 InnoDB 存储引擎**

步骤 1 - 重启 Mysql 服务

*   在运行 对话框中，输入 services。MSC。
*   在服务 窗口中，找到并右键单击 MySQL 服务 .
*   点击重新启动服务。

步骤 2 - 使用 innodb_force_recovery 启动 Mysql 服务器

您需要从配置文件中启用'innodb_force_recovery'选项。步骤如下：

*   找到配置文件 (my.conf)。my.cnf 文件的位置因操作系统而异。在 Windows 中，配置文件位于 “/etc” 目录中。默认路径是 / etc/mysql/my.cnf 参见
*   找到 my.conf 文件后，搜索 [mysqld] 部分，然后插入以下语句。

```
[mysqld] 
Innodb_force_recovery=1 
service mysql restart


```

*   innodb_force_recovery 的默认值为 0。但是，您可能需要将其值修改为 “1”；启动 InnoDB 数据库引擎并转储表。

注意：请确保在继续操作之前创建数据库备份。转储 innodb_force_recovery 值为 4 或更高的表可能会导致数据丢失。

*   如果您能够访问损坏的表，请使用 mysqldump 命令转储表数据，如下所示：

```
mysqldump -u user -p database_name table_name > single_dbtable_dump.sql


```

*   要将所有数据库导出到 dump.sql 文件，请使用以下命令：

```
mysqldump --all-databases --add-drop-database --add-drop-table > dump.sql


```

*   接下来，重新启动 MySQL 服务器，然后使用 DROP DATABASE 命令删除数据库。
*   如果删除数据库失败，请使用以下命令手动删除数据库：

```
cd /var/lib/mysql 
rm -rf db_name


```

*   删除数据库后，通过在 [mysqld] 中注释以下行来禁用 InnoDB 恢复模式：

```
#innodb_force_recovery=...


```

*   将所有更改应用到 my.cnf 文件后。cnf 文件，保存它，然后重新启动 MySQL 服务器。

### **B - 如果您的数据库源引擎是 Myisam**

您可以运行 myisamchk 命令来修复 MyISAM 表。

以下是执行此操作的步骤：

1.  首先，停止 MySQL 服务器。
    
2.  使用以下命令检查损坏的 MyISAM 表：
    
    ```
    myisamchk TABLE
    
    
    ```
    
3.  如果检测到表损坏，您可以通过执行以下命令来恢复表：
    
    ```
    myisamchk –recover TABLE
    
    
    ```
    
4.  接下来，启动 MySQL 服务器。
    

**防止 MySQL 数据库损坏的技巧**
=====================

您可以按照给定的提示来防止 MySQL 数据库损坏：

*   测试 MySQL 内核有助于识别任何可能导致损坏的软件问题。因此，请定期使用 MySQL 命令测试 MySQL 内核。
*   根据您的应用程序需求，使用一致的备份方法定期备份 MySQL 数据库。
*   断电可能会中断 MySQL 应用程序的运行。正在进行的操作并导致数据库损坏。因此，请确保您有不间断电源 / UPS，以避免突然断电。
*   在您的计算机上保留更新版本的防病毒软件以防止病毒 / 恶意软件攻击。

**结论**
======

上述讨论的故障排除方法可以帮助您修复和恢复损坏的 MySQL 数据库。然而，手动方法可能需要时间并导致数据丢失。如果您不能冒丢失数据的风险，那么您可以选择 Stellar Repair for MySQL。它可以帮助您快速修复损坏的 MySQL 数据库并完全完整地恢复其所有对象。该工具还可以帮助您一次性修复多个 MySQL 数据库。

作者：Priyanka Chauhan

更多技术干货请关注公号【**云原生数据库**】

**squids.cn**，云数据库 RDS，迁移工具 DBMotion，云备份 DBTwin 等数据库生态工具。

**irds.cn**，多数据库管理平台（私有云）。

​