# MySQL

## 一、MySQL体系结构

### 1.1 定义数据库和实例

- 数据库

  ```
  物理操作系统文件或其他形式的文件的集合。在MySQL中，数据库文件可以使frm、myd、myi、ibd结尾的文件。当使用NDB引擎时，数据库的文件可能不是操作系统上的文件，而是存放于内存之中的文件，但定义任然不变。
  ```

- 数据库实例

  ```
  有数据库后台进程/线程和一个共享内存区组成。共享内存可以被运行的后台进程/线程所共享。需要牢记的是，数据库实例才是真正用来操作数据库文件的。
  ```

​		MySQL被设计为一个单进程多线程架构的数据库，这点与SQL Server比较类似，但与Oracle多进程的架构有所不同（Oracle的Windows版本也是单进程多线程的架构）。这就是说，MySQL数据库实例在系统上的表现就是一个进程。

​		当启动实例时，MySQL数据库会去读取数据库配置文件，根据配置文件的参数来启动数据库实例。这与Oracle的参数文件（spfile）相似，不同的是，如果没有参数文件，启动时会提示找不到该参数文件。而在MySQL中，可以没有配置文件，在这种情况下，MySQL会按照编译时的默认参数设置启动实例。用以下命令可以查看MySQL启动时，它会在哪里查看配置文件：

```sh
[root@frost2 ~]# mysql --help | grep my.cnf
order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
```

可以看到MySQL是按照如下顺序读取配置文件的。

```mermaid
graph LR
/etc/my.cnf --> /etc/mysql/my.cnf
/etc/mysql/my.cnf --> /usr/local/mysql/etc/my.cnf
/usr/local/mysql/etc/my.cnf --> a["~/.my.cnf"]
```

**如果几个配置文件中都有同一个参数，MySQL会以读取到的最后一个配置文件中的参数为准。**

### 1.2 MySQL体系结构

MySQL数据库体系结构图如下：

![](./img/MySQL Server.png)

从图1-1可以看出，MySQL由以下几个部分组成：

- 连接池组件
- 管理服务和工具组件
- SQL接口组件
- 查询分析器组件
- 优化器组件
- 缓冲（cache）组件
- 插件式存储引擎
- 物理文件

MySQL数据库区别于其他数据库的最重要的一个特点就是其**插件式的表存储引擎**。MySQL插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者可以按照自己的一员来进行开发。

**需要特别注意的是，存储引擎是基于表的，而不是数据库。**

## 二、Innodb存储引擎

​		Innodb是事务安全的MySQL存储引擎，设计上采用了类似于Oracle数据库的架构。通常来说，Innodb存储引擎是OLTP应用中核心表的首选存储引擎。其特点是行锁设计、支持MVCC、支持事务、支持外键、提供一致性非锁定读，同事被设计用来最有效地利用以及使用内存和CPU。

### 2.1 Innodb体系架构

![](/blog/mysql/img/innodb.png)

#### 2.1.1 后台线程

#### 2.1.2 内存

##### 1. 缓冲池

​		Innodb存储引擎是基于磁盘存储的，并将其中的记录按照页的方式进行管理。由于CPU与磁盘之前的速度鸿沟，基于磁盘的数据库系统通常采用缓冲池技术来提高数据库的整体性能。

​		缓冲池简单来说就是一块内存区域。在数据库中进行读取页的操作，首先将从磁盘读取的页放到缓冲池中，这个过程称为将页“FIX“到缓冲池。下一次读取相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中被命中。否则，读取磁盘上的页。

​		对于页的修改，首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘。数据库通过一种Checkpoint机制将页刷新到磁盘，而不是每次修改页都刷新到磁盘。

​		综上所述，缓冲池的大小直接影响这数据库的整体性能。对于Innodb存储引擎而言，其缓冲池的配置通过参数`innodb_buffer_pool_size`来设置。如下所示数据库缓冲池大小为128M。

```mysql
mysql> show variables like 'innodb_buffer_pool_size';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)
```

​		具体来看，缓冲池中的缓存的数据页类型有：索引页、数据页、undo页、插入缓冲（insert buffer）、自适应哈希索引、Innodb存储的锁信息（lock info）、数据字典信息等。不能简单的认为只是缓存索引页和数据页，他们只是占缓冲池很大的一部分而已。

![](.\img\Innodb001.png)

​		从Innodb 1.0X版本开始，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同的缓冲池实例中。这样做的好处是减少数据库内部的资源竞争，增加数据库的并发处理能力。可以通过参数`innodb_buffer_pool_instances`来进行配置，该值默认为1。

​		我们可以通过`show engine Innodb status`查看缓冲池信息。或者通过查询数据库`information_schema`下的表`innodb_buffer_pool_stats`来查看缓冲池信息，一条数据就代表有一个缓冲池实例。

##### 2. LRU List

​		在Innodb存储引擎中，缓冲池中页的大小为16K，使用LRU算法对缓冲池中进行管理。不同的是Innodb存储引擎对传统的LRU算法做了一些优化。在Innodb存储引擎中，LRU列表中增加了midpoint位置。新读取到的页，并不是放到LRU列表首部，而是放到LRU列表midpoint位置。这个算法在Innodb中称为**`midpoint insertion sttategy`**。

在Innodb存储引擎中，可以通过参数`innodb_old_blocks_pct`参数来控制，如下：

```mysql
mysql> show variables like 'innodb_old_blocks_pct';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_old_blocks_pct | 37    |
+-----------------------+-------+
1 row in set (0.01 sec)
```

​		参数默认值为37，表示新读取的页插入到LRU列表尾端37%的位置。在Innodb存储引擎中，把midpoint之后的列表称为old列表，之前的列表称为new列表。可以简单的理解为new列表中的页都是最活跃的热点数据。

​		之所以增加midpoint位置，是因为如果新读取的页放在LRU列表首部，那么某些SQL擦欧洲哦可能会使缓冲池中的页被刷新出去，从而影响缓冲池的效率。常见的这类操作为索引或数据的扫描操作。这类操作需要扫描访问表中的很多页，甚至是全部的页，如果这些页只是这次查询才需要，并不是活跃的热点数据。那么将这些页放都到LRU列表的首部，真正的热点数据就会从LRU列表中被移除。

​		为了解决这个问题，Innodb存储引擎还引入另一个参数`innodb_old_blocks_time`来进一步管理LRU列表，这个参数表示页读取到midpoint位置后需要等待多久才会被加入到LRU列表的热端。

##### 3. Free List

​		LRU列表用来管理已经读取的页，数据库刚启动时，LRU列表是空的。这时页都存放在Free列表中。当页从LRU列表的old部分加入到new部分时，此时发生的操作称为`page made young`，而因为`innodb_old_blocks_time`的设置而导致没有移动到new部分的操作称为`page not made young`。

通过命令**`show engine innodb status`**来观察LRU列表和Free列表的情况。

```properties
**注意**
show engine innodb status命令显示的不是当前的状态，而是过去某个时间范围内Innodb存储引擎的状态。
可以看到这句信息，Per second averages calculated from the last 23 seconds,这表示此次执行显示的是过去23秒内的Innodb存储引擎的状态。
```

```mysql
mysql> show engine innodb status\G;
*************************** 1. row ***************************
  Type: InnoDB
  Name: 
Status: 
=====================================
2021-11-19 11:06:39 0x7f3a502b8700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 23 seconds
......
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992
Dictionary memory allocated 518774
Buffer pool size   8191
Free buffers       7479
Database pages     710
Old database pages 242
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 247, created 463, written 1138
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 710, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
......
```

​		可以看到：当前Buffer pool size共有8191个页，即缓冲池大小为8191*16K。Free buffer表示当前Free列表中页的数量，Database pages表示LRU列表中的页的数量。可以看到Free buffer与Database pages之和不等于Buffer pool size。因为缓冲池中的页还可能会被分配非自适应哈希索引、Lock信息、Insert Buffer等页，而这部分不需要LRU算法进行维护，因此不存在与LRU列表中。

​		Pages made young显示了LRU列表中页移动到前端的次数。还有一个重要的观察变量——**`Buffer pool hit rate`**，表示缓冲池的命中率，通常该值不应该小于95%，否则需要观察是否由于全表扫描导致LRU列表被污染。

​		从Innodb 1.2版本开始，可以通过表`INNODB_BUFFER_POOL_STATS`来观察缓冲池的运行状态，通过表`INNODB_BUFFER_PAGE_LRU`来观察每个LRU列表中每个页的具体信息。

##### 4. Flush List

​		在LRU列表中的页被修改后，称该页为脏页（dirty page）,即缓冲池中的页和磁盘中的不一致。此时数据库就会通过checkpoint机制将脏页刷新到磁盘，而Flush列表中的页即为脏页列表。**需要注意的是，脏页既存在于LRU列表中，又存在于Flush列表中**。LRU列表用来管理缓冲池中页的可用性，Flush列表用来管理奖页刷新到磁盘，二者花不影响。

通过`show engine Innodb status`命令可以查看脏页，即前面例子中的`Modified db pages`。

##### 5. 重做日志缓冲

​		从图2-2可以看到，Innodb存储引擎的内存区域除了有缓冲池外，还有重做日志缓冲（redo log buffer）。Innodb存储引擎首先将日志放到这个缓冲区，然后按照一定的评率将其刷新到重做日志文件（redo log）。重做日志缓冲一般不需要设置得很大，因为一般情况下每秒钟会将重做日志缓冲刷新到重做日志文件，因此**用户只需要保证每秒产生的事务量在这个缓冲大小之内即可。**该值由配置参数innodb_log_buffer_size控制。

```mysql
mysql> show variables like '%innodb_log_buffer_size%';
+------------------------+----------+
| Variable_name          | Value    |
+------------------------+----------+
| innodb_log_buffer_size | 16777216 |
+------------------------+----------+
```

下列三种情况会将重做日志缓冲刷新到重做日志中：

- Master Thread每秒将重做日志缓存刷新到重做日志文件
- 每个事务提交时会将重做日志缓冲刷新到重做日志文件
- 重做日志缓冲剩余空间小于1/2时，会将重做日志缓冲刷新到重做日志文件中

##### 6. 额外的内存池

​		在Innodb存储引擎中，对内存的管理是通过一种称为内存堆（heap）的方式进行的。在对一些数据结构本省的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中申请。例如，分配了缓冲池（innodb_buffer_pool），但是每个缓冲池中的帧缓冲（frame buffer）还有对应的缓冲控制对象（buffer control block）,对这些对象记录了一些诸如LRU、锁、等待等信息，而这个对象的内存需要从额外的内存池中申请。因此，在申请了很大的Innodb存储引擎缓冲池时，也应该考虑相应的增加这个值。

### 2.1.3 CheckPoint 机制





## 附录一：变量

| 变量                         | 说明                                                        |
| ---------------------------- | ----------------------------------------------------------- |
| innodb_buffer_pool_size      | 缓冲池大小                                                  |
| innodb_buffer_pool_instances | 缓冲池实例，默认为1                                         |
| innodb_old_blocks_pct        | midpoint位置：到LRU列表尾端的距离                           |
| innodb_old_blocks_time       | 页读取到midpoint位置后需要等待多久才会被加入到LRU列表的热端 |
| innodb_log_buffer_size       | 重做日志缓冲池大小                                          |
|                              |                                                             |



```mysql
mysql> show engine innodb status\G;
*************************** 1. row ***************************
  Type: InnoDB
  Name: 
Status: 
=====================================
2021-11-19 11:06:39 0x7f3a502b8700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 23 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 113 srv_active, 0 srv_shutdown, 25892446 srv_idle
srv_master_thread log flush and writes: 25892559
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 38
OS WAIT ARRAY INFO: signal count 36
RW-shared spins 0, rounds 60, OS waits 30
RW-excl spins 0, rounds 0, OS waits 0
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 60.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
------------
TRANSACTIONS
------------
Trx id counter 7786
Purge done for trx's n:o < 7663 undo n:o < 0 state: running but idle
History list length 0
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421363692980960, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421363692980048, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
--------
FILE I/O
--------
I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
I/O thread 1 state: waiting for completed aio requests (log thread)
I/O thread 2 state: waiting for completed aio requests (read thread)
I/O thread 3 state: waiting for completed aio requests (read thread)
I/O thread 4 state: waiting for completed aio requests (read thread)
I/O thread 5 state: waiting for completed aio requests (read thread)
I/O thread 6 state: waiting for completed aio requests (write thread)
I/O thread 7 state: waiting for completed aio requests (write thread)
I/O thread 8 state: waiting for completed aio requests (write thread)
I/O thread 9 state: waiting for completed aio requests (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
280 OS file reads, 1944 OS file writes, 836 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 34673, node heap has 0 buffer(s)
Hash table size 34673, node heap has 0 buffer(s)
Hash table size 34673, node heap has 0 buffer(s)
Hash table size 34673, node heap has 0 buffer(s)
Hash table size 34673, node heap has 1 buffer(s)
Hash table size 34673, node heap has 1 buffer(s)
Hash table size 34673, node heap has 0 buffer(s)
Hash table size 34673, node heap has 0 buffer(s)
0.00 hash searches/s, 0.00 non-hash searches/s
---
LOG
---
Log sequence number 3344205
Log flushed up to   3344205
Pages flushed up to 3344205
Last checkpoint at  3344196
0 pending log flushes, 0 pending chkp writes
462 log i/o's done, 0.00 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992
Dictionary memory allocated 518774
Buffer pool size   8191
Free buffers       7479
Database pages     710
Old database pages 242
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 247, created 463, written 1138
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 710, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=9986, Main thread ID=139888343111424, state: sleeping
Number of rows inserted 8781, updated 1, deleted 0, read 13118
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.30 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================

1 row in set (0.00 sec)

ERROR: 
No query specified

```

