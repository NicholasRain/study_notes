# linux的用户管理

## 用户账号

登录  ---》 查看/etc/passwd下是否有账号 ----》去/etc/shadow下核对密码

### /etc/passwd的文件结构

![image-20200923225554541](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20200923225554541.png)

用   ：号隔开

+ 账号名称
+ 密码
+ UID  root的UID是0，1~999 系统账号，1000~60000可登录账号
+ GID 与/etc/group有关
+ 用户信息说明栏
+ 家目录  默认为/home/yuorIDname
+ shell

### /etc/shadow的文件结构

![image-20200923230452286](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20200923230452286.png)

用  :  号隔开

+ 账号名称
+ 密码 加上！或*使密码失效  
+ 最近修改密码的日期 从1970年1月1日做为第一日累加
+ 密码不可被修改的天数 为0则随时可以修改
+ 密码需要重新修改的天数 99999为273年
+ 密码需要修改前的警告天数 
+ 密码过期后的宽限时间
+ 账号失效日期
+ 保留字段

密码的固定格式：

```shell
$id$salt$encrypted
```

id表示加密算法，1代表`MD5`，5代表`SHA-256`，6代表`SHA-512` salt表示密码学中的Salt,系统随机生成 encrypted表示密码的hash

https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/  SHA-256已经不再安全

忘记密码操作：

+ 普通用户：联系系统管理员
+ root 用户 1、进入单人维护模式，系统主动给予root的bash权限，再以passwd修改密码

2、以Live CD挂载根目录去修改/etc/shadow，清空root密码

查看加密机制：

```shell
authconfig --test | grep hashing
```

## 用户组

### /etc/group的文件结构

![image-20200924001412537](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20200924001412537.png)

用 ： 号隔开

+ 组名 与GID对应
+ 用户组密码
+ GID
+ 此用户组自持的账号名称

新版Linux中初始用户不会加入第四个字段

groups命令查看所有支持的用户组，第一个显示的为有效用户组

![image-20200924002407129](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20200924002407129.png)