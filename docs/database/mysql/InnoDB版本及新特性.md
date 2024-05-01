# Innodb版本及新特性

## 一、概述

什么是InnoDB？请看官网：[Introduction to InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-introduction.html)

不晓得从什么时候开始InnoDB的版本号与MySQL的版本号保持一致了。通过变量`innodb_version`可以查看。

- mysql5.7

```
mysql> SHOW VARIABLES LIKE "innodb_version";
+----------------+--------+
| Variable_name  | Value  |
+----------------+--------+
| innodb_version | 5.7.37 |
+----------------+--------+
1 row in set (0.01 sec)
```

- mysql8.0

```
mysql> SHOW VARIABLES LIKE "innodb_version";
+----------------+--------+
| Variable_name  | Value  |
+----------------+--------+
| innodb_version | 8.0.32 |
+----------------+--------+
1 row in set (0.05 sec)
```

版本新增特性如下：

- [5.7版本新增特性说明](https://dev.mysql.com/doc/refman/5.7/en/mysql-nutshell.html)
- [8.0版本新增特性说明](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)

## 二、体系架构

### 1.InnoDB架构图

- 5.7版本架构图

![img](.\img\Innodb_structure57.png)

- 8.0版本架构图

![img](.\img\Innodb_structure8.png)

### 2.Innodb8.0版本新增功能

#### 2.1 Doublewrite Buffer File

MySQL 8.0.20之前，双写缓冲区存储区域位于`InnoDB`系统表空间中。从MySQL 8.0.20开始，双写缓冲区存储区域位于双写文件中。将存储区域移出系统表空间可减少写入延迟、提高吞吐量，并为双写缓冲区页面的放置提供灵活性。为双写缓冲区配置提供以下变量：

- [`innodb_doublewrite_dir`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite_dir)

定义双写缓冲区文件目录。

- [`innodb_doublewrite_files`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite_files)

定义双写文件的数量。

- [`innodb_doublewrite_pages`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite_pages)

定义批量写入的每个线程的最大双写页数。

- [`innodb_doublewrite_batch_size`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite_batch_size)

定义批量写入的双写页数。

#### 2.2 其他的以后遇到再说