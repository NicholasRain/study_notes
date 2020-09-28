# systemd系统服务

> service :  常驻内存中的进程并且可以提供一些系统和网络功能
>
> daemon ：守护进程，完成service的程序

daemon程序的命名总时以d结尾，比如sshd，atd。

CentOS7.X以后使用systemd代替init启动脚本

使用systemd的优点：

+ 并行处理所有服务，加速开机
+ 一经要求就响应on-demand启动方式
+ 服务依赖性自我检查
+ 依daemon功能分类 service、socket、target、path、snapshot、timer
+ 将多个daemon集合成一个群组
+ 向下兼容就有的init版本

配置文件目录：

+ /usr/lib/systemd/system/: 每个服务最主要的脚本
+ /run/systemd/system/：系统执行过程中产生的服务脚本，优先级比/usr/lib/systemd/system/高
+ /etc/systemd/system：管理员建立的服务脚本文件执行优先级比/run/systemd/system/高

## 通过systemctl管理服务

启动服务的两个阶段：

1、开机时是否自动启动

2、现在是否启用

```shell
systemctl [command] [unit]
command:
start          #立即启动后面接的unit
stop           #立即停止后面接的unit
restart        #立即重启后面接的unit
reload         #不关闭unit的情况下重新加载配置文件
enable         #下次开启自启
disable        #下次开机不自启
status         #查看unit的状态
is-active      #目前有没有正在运行
is-enable      #开机有没有默认使用这个unit
mask           #注销服务
unmask         #取消注销
```

若直接用kill关闭服务，则systemctl将无法再监控该服务

## 使用systemctl查看所有的服务

```shell
systemctl [command] [--type=unit] --all

command:
	list-units    :依据unit显示当前已经启动的服务，加上all显示所有
	list-unit-files  :依据/usr/lib/systemd/system/内的文件
--type: service、socket、target、path、snapshot、timer这些
```

![image-20200927135748112](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200927140254.png)

![image-20200927140105745](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200927140248.png)

## 图形界面和命令行界面的切换

```shell
systemctl get-default
systemctl set-default   multi-user.target
systemctl isolate multi-user.target    #命令行
systemctl isolate graphical.target     #图形
```

```
systemctl poweroff 
systemctl reboot
systemctl suspend      #挂起模式
systemctl hibernate    #休眠模式
systemctl rescue       #进入强制恢复模式
systemctl emergency    #进入紧急恢复模式
```

## 通过system分析各服务之间的依赖

```shell
systemctl list-dependencies [unit] [--reverse]
--reverse: 反向跟踪谁会使用这个unit
```

![image-20200927142820883](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200927142823.png)

使用cat /etc/services | grep 查看服务和端口号

## 网络服务

查看启用端口  netstat -tlunp

关闭网络服务

![image-20200928110034553](https://cdn.jsdelivr.net/gh/NicholasRain/pictures@master/20200928110036.png)