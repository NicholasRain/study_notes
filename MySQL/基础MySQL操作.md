# MySQL基本操作

使用 mysql -h localhost -P 3306 -u root -ppassword mysql 进入mysql的mysql数据库

Ctrl  + D 退出mysql，相当于exit

撤销命令使用Ctrl + C 或者 \c

；和 \g 对应水平输出  \G 对应垂直输出

反标记字符\`\`用来对数据库有特殊字符使用。例如：\`use  my.contacts `。

通过show  variables like ‘datadir’ ； 得知数据目录

## 创建表

```mysql
CREATE TABLE IF NOT EXISTS `offices` (
  `officeCode` varchar(10) NOT NULL,
  `city` varchar(50) NOT NULL,
  `phone` varchar(50) NOT NULL,
  `addressLine1` varchar(50) NOT NULL,
  `addressLine2` varchar(50) DEFAULT NULL,
  `state` varchar(50) DEFAULT NULL,
  `country` varchar(50) NOT NULL,
  `postalCode` varchar(15) NOT NULL,
  `territory` varchar(10) NOT NULL,
  PRIMARY KEY (`officeCode`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

使用DESC tablename； 或者 show create table tablename \G；来显示表的详细信息。

使用InnoDB会在数据文件中创建.idb文件

克隆表

create table new_table like old_table;

但不会克隆数据。 

## 插入数据

```mysql
insert [ignore] into tablename (name,age)
values
('nicholas',21),
('mark',22),
('lalala',21);
```

ingore 用于如果该行已经存在，则新数据被忽略，如果有ignore子句正常执行，没有则会生成错误信息。

replace同insert，如果没有重复行就插入，有就先删除原来有的行，插入新的行。

on duplicate key update: https://blog.csdn.net/zyb2017/article/details/78449910

## 更新数据

```mysql
update tablename set name='nicholasrain',age=18 where name='nicholas' and age=21;
```

如果没有where子句则会修改整个表。建议在事务中进行update操作，利于回滚。

## 删除数据

```mysql
delete from tablename where name='nicholasrain' and age=18;
```

如果没有where则删除所有行，建议在事务中进行update操作，利于回滚。

**删除整表且保留表结构**

使用truncate table tablename;

该操作是DDL操作，一旦执行不能恢复。

**source  XXX.sql 可以执行sql文件**

