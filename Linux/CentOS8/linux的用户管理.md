# linux的用户管理

## 用户账号

登录  ---》 查看/etc/passwd下是否有账号 ----》去/etc/shadow下核对密码

### /etc/passwd的文件结构

![image-20200923225554541](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200923225554541.png)

用   ：号隔开

+ 账号名称
+ 密码
+ UID  root的UID是0，1~999 系统账号，1000~60000可登录账号
+ GID 与/etc/group有关
+ 用户信息说明栏
+ 家目录  默认为/home/yuorIDname
+ shell

### /etc/shadow的文件结构

![image-20200923230452286](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200923230452286.png)

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

![image-20200924001412537](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200924001412537.png)

用 ： 号隔开

+ 组名 与GID对应
+ 用户组密码
+ GID
+ 此用户组自持的账号名称

新版Linux中初始用户不会加入第四个字段

groups命令查看所有支持的用户组，第一个显示的为有效用户组

![image-20200924204327131](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200924204711.png)

newgrp wheel  用来切换有效用户组，记得用exit退出

![image-20200924163018593](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200924204716.jpg)

### /etc/gshadow的文件结构

![image-20200924164426835](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200924164426835.png)

用 ：号隔开

+ 组名
+ 密码栏  ！开头表示无合法的密码
+ 用户组管理员的账号
+ 有加入该用户组支持的所属账号

# 账号管理

## 新增用户

adduser

![image-20200924165425829](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200924165425829.png)

useradd

![image-20200924171355123](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200924171355123.png)

useradd的默认值可以用useradd -D 查看

/etc/skel是用户家目录参考基准目录

UID/GID的密码参数参考：/etc/login.defs

## 用户删除和密码设置

账户刚刚创建后默认是锁定的，不能登录，需要设置密码

passwd  [user]

![image-20200924192613665](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/image-20200924192613665.png)

若直接输入passwd会修改当前用户的密码

现在的Linux会使用PAM模块检查密码，在/etc/pam.d/passwd中

历史命令在/root/.bash_history里

使用管道输入修改密码

```shell
echo "abc543cc" | passwd --stdin user
```

不用重复输入，创建大量用户时使用

passwd -S 查看密码参数

锁定用户

```shell
passwd -l user
```

解锁用户

```shell
passwd -u user
```

用chage详细显示密码参数

![image-20200925093355637](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200925093357.png)

如果想让用户登陆后强制更改密码

```shell
chage -d 0 user
```

## 修改用户

usermod  [-cdegGlsuLU]  username

![image-20200925095505281](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200925095506.png)

删除用户

userdel  -r user

-r：连同家目录一起删除

## ACL（access control list）主机详细权限规划

查看是否启动了ACL

```shell
dmesg | grep -i acl
```

getfacl 和setfacl

为用户nicholas 添加文件file1的rx权限

```shell
setfacl -m u:nicholas:rx file1
```

查看file1的acl参数

```shell
getfacl file1
```

d:u:username:rwx  子文件默认继承

g:groupname:rwx  设置用户组

全部取消ACL 使用 -b  ； 单一的取消使用 -x

```shell
setfavl -x u:nicholas /file1
```

 acl设置时权限项不能留空白，用  -   代替没有权限

# 用户切换

## su

su  [-lm] [-c命令] [username]

+ \-  表示使用login-shell的变量文件的读取方式登录
+ -l   login-shell方式，后面需要加使用者账号
+ -m 表示使用当前的配置文件
+ -c 只使用一次命令，后面加一条命令

切换用户时最好用 su **-**

 使用root身份执行一条命令

```shell
su - -c "head -n 3 /etc/shadow"
```

## sudo

只有在/etc/sudoers里的才能使用sudo命令

sudo [-b] [-u a new username]

+ -b  将后续的命令放到后台让系统执行，不与当前的shell产生影响
+ -m 要切换的用户

使用visudo编辑/etc/sudoers

%groupname  表示用户组

![image-20200925131934262](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200925132257.jpg)

The first `ALL` means that `root` is able to use `sudo` from any Terminal. The second `ALL` means that `root` can use `sudo` to impersonate any other user. The third `ALL` means that `root` can impersonate any other group. Finally, the last `ALL` refers to what commands this user is able to do; in this case, any command he or she wishes.