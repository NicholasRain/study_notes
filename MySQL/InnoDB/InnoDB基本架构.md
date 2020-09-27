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

