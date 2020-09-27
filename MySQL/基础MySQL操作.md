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

使用DESC tablename； 或者 show create table tablename 、G；来显示表的详细信息。

使用InnoDB会在数据文件中创建.idb文件

克隆表

create table new_table like old_table;

但不会克隆数据。 

