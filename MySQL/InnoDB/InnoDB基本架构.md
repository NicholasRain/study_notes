# InnoDB 存储引擎

InnoDB存储引擎是事务安全的存储引擎，该引擎是第一个完整支持ACID事务的MySQL存储引擎（BDB是第一个支持事务的引擎）。

ACID四大特性：

- Atomicity（原子性）：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
- Consistency（一致性）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设约束、触发器、级联回滚等。
- Isolation（隔离性）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- Durability（持久性）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## InnoDB存储引擎体系架构

![image-20200926173435999](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200926173438.png)

后台线程主要负责刷新内存池中的数据，保证缓冲池的内存缓存时最新的。同时将刷新发数据存入磁盘中。同时保证数据库发生异常时能正常恢复

## 后台线程

InnoDB是多线程的存储引擎，有过个线程在后台工作

1.  Master Thread

   非常核心的一个后台线程，负责缓冲池里的数据**异步**刷新回磁盘，保证数据的一致性。包括脏页刷新、合并插入缓冲、UNDO页的回收

2.  IO Thread

   Innodb大量使用AIO来进行回调处理。主要有read、write、insert buffer、log IO Thread。Windows下通过innodb_read_io_threads 和 innodb_write_io_threads参数进行设置。

   ![image-20200926175303493](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200926175536.png)

    可以用`SHOW ENGINE INNODB STATUS\G;` 来观察InnoDB的IO Thread：

   ![image-20200926175649816](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200926175654.png)
   
3.   Purge Thread

    回收事务已经提交不需要的undo页。从InnoDB 1.2版本开始，InnoDB支持多个Purge Thread，这样做的目的是为了进一步加快undo页的回收。同时由于Purge Thread需要 离散地读取undo页，这样也能更进一步利用磁盘的随机读取性能。如用户可以设置4个Purge Thread：

![image-20200927175339348](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200927175343.png)

4.  Page Clearner Thread

   将脏页的刷新操作放到单独的线程进行操作，提高存储引擎的性能。

## 内存

1. 缓冲池

   InnoDB存储引擎是基于磁盘存储的，并将记录按**页**的方式管理。因此CPU和磁盘之间的速度鸿沟需要缓冲池来提高数据库的性能。缓冲池实际是内存的一块区域。对于读操作，采用CPU\==》内存==》磁盘的读取方式。写操作时，先修改缓存中的页，然后以一定的速度刷新回磁盘，并不是每次修改就直接写回磁盘，通过checkpoint机制。

   ![image-20200927181239656](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200927181241.png)

   使用set global innodb_buffer_pool_size = 2097152; 进行修改。

   调优参数：

   val = Innodb_buffer_pool_pages_data / Innodb_buffer_pool_pages_total * 100%
   val > 95% 则考虑增大 innodb_buffer_pool_size， 建议使用物理内存的75%
   val < 95% 则考虑减小 innodb_buffer_pool_size， 建议设置为：Innodb_buffer_pool_pages_data\*Innodb_page_size\*1.05/(1024\*1024*1024)      参考文章：https://www.cnblogs.com/wanbin/p/9530833.html                

   缓存的数据页类型:

   + 索引页
   + 数据页
   + undo页
   + 插入缓冲（insert buffer）
   + 自适应哈希索引 （adaptive hash index）
   + Innodb存储的锁信息（lock info）
   + 数据字典信息（data dictionary）

   ![image-20200928164156951](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200928164158.png)

​                                                 上图：InnoDB内存数据对象

​			通过增加缓冲池实例来减少数据库内部资源的竞争，使用命令

```mysql
show variables like 'innodb_buffer_pool_instances'\G;
```

​			显示缓冲池实例信息。使用show engine innodb status\G；显示详细信息。

​			在配置文件中修改缓冲池实例数量。可以通过information_schema架构下的 

表INNODB_BUFFER_POOL_STATS来观察缓冲的状态。

```mysql
SELECT POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES  FROM INNODB_BUFFER_POOL_STATS\G;
```

2. LRU List、Free List和Flush List

   一般采用LRU（最近最少使用）算法来管理缓冲池中的页，频繁使用的放在前端，使用少的放在后端，当新的页缓冲池放不下时将释放LRU尾端的页。

   ![image-20200928185132031](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20200928185132031.png)

   使用优化的LRU算法。默认是距离尾端37%（约3/8）位置处，叫midpoint，midpoint前的叫new列表，之后的叫old列表。