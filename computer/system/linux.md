

# 手册说明

* 本手册基于`CentOS 64位`操作系统的使用.



# 安装使用

## 安装教程



### 0. 参考资料

* 视频教程: 
  * b站: [尚硅谷大数据之Linux教程](https://www.bilibili.com/video/av31976945/?p=6)
  * 谷粒学院: [尚硅谷大数据之Linux教程](http://www.gulixueyuan.com/course/272)

* 安装教程: https://blog.csdn.net/wu_zeqin/article/details/79833046



### 1. 工具下载

网盘位置: `/study/resouce/linux/`
*  VMware_10.0安装包
* CentOS_6.8_64位安装包





### 2. VMware安装

安装过程都是下一步, 下一步即可... 没有需要注意的地方.



### 3. CentOS安装



### 4. VMtools安装

VMTools可以让我们

* 在`主机`和`虚拟机`之间进行文件的复制粘贴操作.
* 为`主机`和`虚拟机`建立共享文件夹.

安装步骤如下:

* VMware选项卡[虚拟机] => 安装VMwareTools
  * 此时会将虚拟机上插入一个**光盘**, 光盘中有VMtools的安装文件.
* 将光盘中的`vmwaretools...tar.gz`拷贝到任意一个文件夹.
* 解压上述文件, 然后运行其中的`vmware-install.pl`文件, 一路回车即可.



### 共享文件夹配置

![1556701842687](assets/1556701842687.png)

我们只需要在`虚拟机设置` => `选项` => `共享文件夹`中配置路径即可.

* `c:\vms\share` 就会映射到Linux系统当中的 `/mnt/hgfs/share` 目录了.



## 虚拟机网络

### 网络连接方式

虚拟机的网络连接拥有三种模式, 我们在VMware当中虚拟机设置当中就会看到如下三种模式:

* `桥接模式`: 主机和虚拟机使用相同的网段, 例如*192.168.124.0.*
  * 好处: 主机也好, 虚拟机也好, 大家都在同一个网段, 可以互相通信
  * 坏处: 需要占用一个ip地址, 耗费资源
* `NAT模式`: 在主机上新虚拟出一个网络, 虚拟机使用这个网络.
  * 好处: 不占用ip地址, 不会出现冲突
  * 坏处: 其他主机无法访问到虚拟机
* `仅主机模式`: 独立的ip, 一般不会用



### 新装的虚拟机联网

![1556700367694](assets/1556700367694.png)

点击图片中的位置即可.



## Linux目录结构

* `/dev` 内置设备管理, CPU/硬盘等硬件都被映射成了一个文件, 方便我们管理.
* `/media` 外置设备管理, 光盘/U盘等设备插入后, 就会挂载到这个目录.
* `/etc` 配置文件, 系统的相关配置文件都在这个目录当中.
* `/mnt` 提供这个目录是为了让用户临时挂载其他的文件系统.
  * `/mnt/hgfs` 共享文件夹所在的目录.
* `/bin` 系统命令, 常见的`cd`, `ls`等命令都在这个文件夹当中.
* `/sbin` 超级用户才可以使用的系统命令.
* `/opt` 应用程序的安装包存放的目录, 默认为空.
* `/usr` 应用程序的安装目录, 类似于Windows当中的`Program Files`文件夹.
  * `/usr/bin` 系统级的应用一般都会在这里.
  * `/usr/local/bin`: 用户级的应用一般应该放在这里.
* `/root` root用户的家目录.
* `/home` 普通用户的家目录.
  * `/home/firene` firene用户的家目录.
* `/lib` 系统开机所需要使用的动态连接共享库, 类似于Windows当中的`.dll`文件.
* `/lost+found` 一般是空的, 非法关机的话会产生一些文件.
* `/tmp` 存放一些临时文件.
* `/var` 存放经常变化的东西, 例如各种日志文件.
* `/boot` & `/proc` & `/sys`: Linux高手去玩的地方.
* `/selinux`类似于360安全管家的目录, 可以管理系统...

* `/srv` 该目录存放一些服务启动之后需要提取的数据.



### 分类总结

* `/boot` `sys` `/srv` `/lib` `/proc` `/selinux` `/lost+found`
* `/dev` `/media` `/mnt`
* `/bin` `/sbin` `/opt` `/usr`
* `/root` `/home`
* `/tmp` `/var`



## 远程登录Linux

### XShell

### Xftp

没啥子好说的...



## 开机/重启/登录/注销

* `shutdown -h now` 立即关机
* `shutdown -h 1` "hello, 1分钟关机了."
* `shutdown -r now` 立即重启
* `halt` 立即关机
* `reboot` 立即重启
* `sync` 把内存中的数据同步到磁盘上

> 不管是关机还是重启, 都应该提前执行以下`sync`命令, 避免数据丢失.

* `logout` 注销用户(图形界面的终端这个命令是无效的, 远程登录是可以的)

* `su - root`: 切换到root用户

* `exit`: 回退到原来的用户
* `whoami`: 我是谁?



## 运行级别

Linux系统有7个运行级别:

* `[0]关机`
* `[1]单用户` 不需要密码即可登录
* `[2]多用户无网络服务`
* `[3]多用户有网络服务` 常用
* `[4]保留级别`
* `[5]图形界面` 常用
* `[6]重启`

系统的运行级别的配置文件是`/etc/inittab`:

```powershell
[root@centos6 ~]# cat /etc/inittab
......
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
id:5:initdefault:
```

可以看到, 我们装的图形化界面的系统, 默认的运行级别是5, 如果我们改为3的话, 就可以小黑屏启动了~ 当然, 我们可以通过如下的命令来即时修改系统的运行级别:

```sehll
init 3
```

上面的命令可以将系统的运行级别切换为3.



### 如何找回丢失的root密码?

如果我们不小心忘记了root密码.... 我们可以这样:

1. 进入到单用户模式
2. 修改root密码即可

上面只是我们的逻辑思路, 如果我们使用ssh登录会发现 , `init 1`命令进入到了不可描述的地方, 并不能真的进入到单用户状态, 所以说我们需要按照如下的步骤进行:

1. 开机启动的时候, 疯狂按`<Enter>`键.

![1556775923549](assets/1556775923549.png)

2. 按下`e`键.

![1556775784749](assets/1556775784749.png)

3. 选中`kernel`, 按下`e`键

![1556775789771](assets/1556775789771.png)

4. 输入`_1`(下划线代表空格), 按下`<Enter>`键.

![1556775795300](assets/1556775795300.png)

5. 按下`b`键.

![1556775801098](assets/1556775801098.png)

6. 已经进入到`单用户模式`, 可以使用`passwd`命令修改密码了.

![1556775857907](assets/1556775857907.png)



> **既然可以直接修改root用户的密码? 安全性何在?**
>
> 要知道, 在公司工作的时候, 多数人都是通过ssh客户端连接到linux服务器的, 真正能进入到机房接触到服务器本机的人很少的, 既然都能接触到本机了, 密码就没什么意义了, `init 1`的模式远程登录是无法进入的, 这也就保证了linux系统的安全性了.







# 基本命令

## 帮助信息

我们在使用Linux的基本命令的时候, 难免会遇到忘记语法等等的情况, 这个时候我们可能回去查询文档, 但是不如直接通过学习几个指令来帮助我们获取到命令的相关信息.



### man

```shell
man [命令或者配置文件]
```

例如: `man ls`



### help

```shell
help [命令或者配置文件]
```

例如: `help cd`





## 不起眼的命令

* `cal` 打印当前日期的日历

  * ```shell
    [root@centos6 ~]# cal
          五月 2019     
    日 一 二 三 四 五 六
              1  2  3  4
     5  6  7  8  9 10 11
    12 13 14 15 16 17 18
    19 20 21 22 23 24 25
    26 27 28 29 30 31
    ```

  * `cal ` 显示当前月份的信息

  * `cal 2019` 显示2019年所有月份的信息

  * `cal 4 2019` 显示2019年4月份的信息

  * `cal -m` 一二三四五六日, 这种格式显示

* `history` 查看命令的记录

  * `history` 查看使用过的所有记录
  * `history 10` 查看最近使用过的10条记录
  * `!299` 执行id为299的历史指令

* `date` 查看日期/时间

  * `date` 显示当前日期: ==2019年 05月 02日 星期四 16:29:22 CST==
  * `date +%Y` 显示当前 年份
    * `+%Y` `+%m` `+%d` `+%H` `+%M` `+%S` 年月日时分秒
  * `date "+today is %d %m"` 格式化输出, 注意前面的`+`, 必须写上, 但是结果没有`+`
  * `date -s "2019-01-24 12:24:24"` 设置系统时间



## 查找相关

* `find` 查找文件
  * 语法: `find [查找范围] [选项]`
  * `find /home -name aaa` 去/home目录中找名字为aaa的文件
    * `-name *.txt` 支持通配符哦
  * `find / -user firene` 查找用户为firene的文件
  * `find / -size +20M` 查找大于20M的文件
    * `+n` 大于多少 `-n` 小于多少 `n` 等于多少 `k` `M` `G`
* `locate` 利用建立好的文件数据库进行查找, 有"延迟", 但是速度快
  * 由于locate指令基于数据库, 我们第一次运行之前, 应该使用`updatedb`命令更新一下数据库.
  * `locate profile` 查找文件名称为profile的文件的所有所在位置
* `grep` 查找文件内容  中我们想要的`行`
  * 语法: `grep [选项] [查找内容] [源文件列表]`
  * `grep firene /etc/passwd`: 在/etc/passwd文件当中查找包含firene字样的行
    * `grep firene /etc/passwd /etc/group`: 可以同时查找多个文件哦~
    * `grep firene *.txt`: 通配符选中多个文件当然也可以啦~
  * `grep -r firene /etc`: 递归查找/etc目录中的所有文件, 找到其中包含firene字样的行
  * `-n` 显示行号 `-i` 不区分大小写 `-v` 反向检索(选择不符合条件的)
  * 正则表达式是支持的, 不过要带上引号哦~ : `grep 'a\w\+' /etc/passwd`
    * `^` `$` `.` `\?` `*` `\+` `[]` `\(\)` `\<\>`  `\{\}` `\w` `\b` `\|`



## 归档/压缩/解压

* `tar` 归档(打包), 拆包
  * 语法: `tar [选项] [归档文件名称] 要归档的文件/目录列表`
  * `-c` 代表要归档(create) `-v` 打印详细信息 `-f -name` 指定压缩后的文件名
  * `-t` 代表要查看 `-x` 代表要解压 `-z` 代表要调用gzip进行压缩 `-C path`指定解压路径
  * 参数分析: 肯定要带着`v`, 肯定要带着`-f xxx`, 所以如下几种: `-cvf` `-tvf` `-xvf` `-zcvf` `-zxvf`
  * `tar -cvf my.tar aa a` 把aa目录和a文件归档成为my.tar
  * `tar -zcvf my.tar.gz aa a` 把aa目录和a文件归档成为my.tar然后压缩成my.tar.gz
    * 相当于`tar -cvf my.tar aa a` + `gzip my.tar`
  * `tar -tvf my.tar` 查看my.tar当中的内容, ==也可以查看`.gz`的压缩包内容哦~ 很suang==
  * `tar -xvf my.tar` 拆包my.tar当中的内容, 也可以解压+拆包`.gz`结尾的压缩包....
  * `tar -zxvf my.tar.gz` 先解压, 再拆包
  * `tar -zxvf my.tar.gz -C /tmp` 解压到tmp目录去...==(-C一定要写在后面, 没有原因)==
  * 参数再分析: `-xvf name` 是 `-x -v -f name`的简写, `-f name`是一组, 所以`-vxf`也是对的, 不过`-fvx`就是错的... 因为`-x name`不是一组....
  * 最终记忆版本:
    * `-cvf` 我要归档 `-zcvf` 我要归档+压缩 `-tvf` 我要看内容 `-xvf` 我要拆包 `-zxvf` 我要解压+拆包

* `gzip` & `gunzip` 压缩和解压缩文件

  * 特点一: 这两个命令==只能对文件进行操作==, 不能选中一个目录!
  * 特点二: `a`经过压缩之后得到`a.gz`, 并且==原来的文件不存在==了!
  * `gzip my.tar` 压缩my.tar这个文件, 得到my.tar.gz
  * `gunzip my.tar.gz` 将压缩文件解压缩得到源文件my.tar

* `zip` & `unzip` 压缩和解压缩文件或者目录

  * 特点一: 这两个命令可以==对文件或者文件夹操作==, 其实就是操作一个节点树.

  * 特点二: 就是往常的情况, 原来的文件可不会消失...

  * `zip aaaa a.txt` 将a.txt文件打包成aaaa.zip

  * `zip -r bbbb aa` 将aa目录中的内容递归地打包成bbbb.zip

    * `zip bbbb aa` 这样的话, 只会将aa这个目录放进压缩包而已....

  * `zip -r cccc /home/*.txt` 将/home下的所有文本文件打包一下...

    ---

  * `unzip aaaa.zip` 将aaaa.zip解压到当前的工作路径

  * `unzip -d /tmp bbbb.zip` 将bbbb.zip解压到/tmp目录下...

    







## 文件管理

* `pwd` 显示当前的工作路径
* `ls` 列出指定目录下的文件列表
  * `ls` 列出当前工作路径下的全部文件
  * `ls /etc` 列出etc目录下的全部文件
  * `ls -a` 列出当前工作路径下的全部文件, 包含隐藏文件
  * `ls -l` 列出当前工作路径下的全部文件, 用列表的形式展示
* `cd` 切换工作路径
  * `cd` | `cd ~` 切换到家目录
  * `cd ..` 切换到上层目录
* `mkdir` 创建目录
  * `mkdir aa` 创建一个目录aa
  * `mkdir -p aaa/aa` 创建多级目录
* `rmdir` 删除一个空目录
  * `rmdir aa` 移除空的目录aa
* `rm` 删除目录或者文件
  * `rm -r aa` 递归删除目录aa的所有内容
  * `rm -f a` 删除a文件, 不要提示是否删除的信息
  * `rm -rf /`  把自己的系统搞垮
* `touch` 创建空文件
  * `touch a` 创建一个空文件a
* `cp` 拷贝
  * `cp a /tmp` 将当前目录下的a文件拷贝到/tmp文件夹下
  * `cp -r aa /tmp` 将当前目录下的aa目录中的所有内容递归地拷贝到/tmp文件夹下
  * `\cp a /tmp` 同样还是复制, 但是如果有覆盖的情况, 不要提示我, ==强制覆盖==!
* `mv` 移动&重命名
  * `mv aa /tmp` 移动==文件或者目录==到指定位置
  * `mv aa aa.txt` 将文件aa重新命名为aa.txt
* `cat` 查看文件内容
  * `cat /etc/passwd` 查看passwd文件中的内容
  * `cat -n /etc/profile` 查看文件, 并且显示行号
* `more` 分页显示文件内容
  * `<Space>`|`<C-f>` 下一页
  * `<C-b>` 上一页
  * `<Enter>` 向下一行
  * `q` 退出
  * `=` 显示当前行号
  * `:f` 输出文件名和行号
* `less` 和more命令一样, 不过只加载文件的指定部分, 效率高.
* `head` 查看文件的前几行
  * `head a.txt` 默认查看前10行
  * `head -n 5 a.txt` 查看前5行
* `tail` 查看文件的后几行
  * 默认查看10行, 可以使用`-n 5`来指定行数
  * `tail -f /etc/profile` 动态地监控指定文件, 有新的变化马上可以看到.



### 软/硬链接

* `ln -s /bin/cat cgdir` 创建一个软链接, 指向/bin/cd命令
* `ln -s /usr/local/bin ulb` 创建一个软链接, 指向一个目录, 这两个目录中的内容一致的.
* 





## 拓展部分

### 重定向: `>>` & `>`

对于一些"SELECT"类型的命令, 他们都是有输出结果的, 不出所料, 这些结果都显示在了屏幕上, 换句话说, 这些结果==打印到了标准输出==, 那么能不能让结果打印到其他地方呢?

* `cat /etc/profile > a.txt`: 将cat命令的执行结果**写入**到a.txt当中(会覆盖原有内容)
* `cat a.txt >> b.txt` 将a.txt当中的内容, **追加**到b.txt当中



### echo命令

```shell
echo "hello, world"
echo $PATH
```

这个命令, 可以向标准输出中打印一个字符串, 当然了, 我们也可以重定向到文件...

```shell
echo "you can't roll like this like." > a.txt
```

这样也是可以的...



### 管道: `|`

管道的作用就是==将前一个程序的输出作为后一个程序的输入==, 如果你熟悉==函数调用==, 那么你可以理解这就是==命令调用==而已, 没什么大惊小怪的, `fun1(fun2())` & `cmd1 | cmd2` 都是一样的...

```shell
cat /etc/profile | more
```

将cat命令的结果, 交给了more命令, 仅此.



### 多命令依次执行: `;`

有的时候, 我们需要一行同时执行多条命令, 例如想要"进入根目录并且查看根目录的内容":

```shell
cd / ; ls
```

这个时候就可以利用`;`来组合两条命令, 这两条命令会依次执行.



### 后台运行: `&`

如果我们想要后台运行一个程序, 就可以在命令之后添加一个`&`:

```shell
./redis-server &
```



### 多行命令: `\`

如果我们一个命令很长的话.. 可以使用`\`来书写多行命令, 例如:

```shell
bin/startup.sh & tail -f \
$CATALINA_HOME/logs/catalina.out
```





# 用户 & 权限

## 用户管理

### useradd

```shell
useradd firene
```

当我们执行上面的命令的时候, 发生了下面的三个事情:

* 新建立了一个组`firene`
* 新建立了一个用户`firene`, 并且用户`firene`在组`firene`当中.
* `/home/`下产生了一个新的目录`firene/`

也就是说, 当我们创建一个用户不指定组的时候, ==默认会创建一个与用户名相同的组==, 然后将该用户放入到此组当中.

##### 指定家目录

```shell
useradd -d /tmp/firene firene
```

这个命令就可以指定用户的家目录了...

##### 指定主组

```shell
useradd -g admin firene
```

上面的命令在创建`firene`用户的时候, 还指定了其所在的主组: `admin`

##### 指定副组

```shell
useradd -G g1,g2,g3 firene
```

指定firene所属的副组

##### 指定注释

```shell
useradd -c "firenefallen" firene
```

##### 指定shell

```shell
useradd -s /bin/sh firene
```



### passwd

```shell
passwd firene
```

上面这条命令可以为用户设置密码, 我们就可以登录了.



```shell
passwd
```

这样可以修改*root*用户的密码.



### userdel

```shell
userdel firene
```

上面这条命令会移除firene用户, 但是不会删除firene用户的家目录.



```shell
userdel -r firene
```

这样就可以同时移除用户的家目录了.



### usermod

```sehll
usermod -g general firene
```

将`firene`用户的组修改为`general`.

* `-g GROUP` 修改主组
* `-G GROUPS` 修改副组
* `-c "COMMENT"` 修改注释信息
* `-s SHELL_PATH` 修改shell
* `-d PATH` 修改家目录



### id

```shell
[root@centos6 home]# id firene
uid=500(firene) gid=500(firene) 组=500(firene)
```

上面这条命令可以看到用户firene的相关信息.



## 用户组管理



### groupadd

```shell
groupadd admin
```



### groupdel

```shell
groupdel admin
```



### groupmod

```shell
groupmod -n devo dev
```



### gpasswd

* `gpasswd -a firene admin`: 将用户firene添加到admin组
* `gpasswd -A user1,user2 admin`: 将用户user1, user2设置为admin组的管理员(仅root)
* `gpasswd -M user1,user2 admin`: 将一批用户添加到admin组(仅root)
* `gpasswd -d firene admin`: 将用户firene从admin组移除
* `gpasswd admin`: 设置admin群的密码



### newgrp

如果一个用户隶属于多个群组, 可以通过这个命令来切换当前所在的群组(并不是修改主组, 只是当前使用的组)

* `newgrp test`: 切换到测试组
* `exit`: 退出测试组



## 相关配置文件

### /etc/passwd

```txt
[root@centos6 ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
......
firene:x:500:500:firene fallen:/home/firene:/bin/bash
```

这个配置文件配置了*用户信息*, 格式如下:

* `用户名:密码:uid:gid:注释信息:家目录:shell`



### /etc/shadow

```txt
[root@centos6 ~]# cat /etc/shadow
root:$6$pgMUiYdKq2...muCR0:18017:0:99999:7:::
......
firene:$6$ULqn8Xwx...X/wJ.:18017:0:99999:7:::
```

*口令(密码)*的配置文件, 格式如下:

* `登录名:口令:最后修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志`



### /etc/group

```txt
[root@centos6 ~]# cat /etc/group
root:x:0:
......
firene:x:500:
admin:x:503:
general:x:504:
```

*用户组*的配置文件, 格式如下:

* `组名称:组口令:gid:组内用户列表`



### /etc/gshadow

```txt
[root@centos6 ~]# cat /etc/gshadow
root:::
firene:!!::
pro:!::
test:!::firene
dev:!:user1:user2
```

用户组**的配置文件, 和上个文件当中的内容区别不大:

* `组名:口令:组管理员列表:组普通成员列表`



## 用户&组总结

### 用户和组的关系

* 一个用户属于多个组, 其中一个组为主组, 其他的组为副组
* 一个组中有若干个用户, 某些是狗理员, 某些是沙雕群员



### 命令总结

* `useradd` `usermod` `userdel`
  * `-c` `-d` `-s` `-g` `-G` 可以指定用户的主组和副组
* `groupadd` `groupmod` `groupdel`
  * `-n`
* `passwd` `gpasswd` `newgrp`
  * `-a` `-d` `-A` `-M` 可以添加/移除用户, 设置管理员
  * `newgrp` 切换所在的组



## 权限

### `ls -l`

```shell
[root@centos6 ~]# mkdir aa
[root@centos6 ~]# touch a
[root@centos6 ~]# ln -s /bin/cat neko
[root@centos6 ~]# ls -l
总用量 4
-rw-r--r--. 1 root root    0 5月   2 20:36 a
drwxr-xr-x. 2 root root 4096 5月   2 20:36 aa
lrwxrwxrwx. 1 root root    8 5月   2 20:40 neko -> /bin/cat
```

我们通过`ls -l /`命令或者`ll`命令可以查看到文件的详细信息, 显示的列格式如下:

- `类型` `权限`
  - `-` 文件 `d` 目录 `l` 连接
  - `r` `w` `x` 拥有相应权限 `-` 没有相应权限
- `文件硬链接数` | `目录中的子节点数`
- `文件拥有者` 默认为该文件的创建者
- `文件所属组` 默认为该文件的创建者**所在的组** (默认在主组, 可以通过newgrp命令切换)
- `文件大小(字节)` 目录的大小都是4096
- `文件创建月份`+ `文件创建日期`+`文件创建时间`
- `文件名` 如果为`.`开头, 代表是一个隐藏文件
- `->` 说明这是一个链接, 指向了相应的节点



### `-rwxr-xr-x`

`-rwxrwxrwx` 我们应该如何理解呢?

* `第一个rwx`: 文件**所有者**对其的权限, 新创建的文件默认权限为`6`, 目录权限为`7`
* `第二个rwx`: 文件**所属组**对其的权限, 新创建的文件默认权限为`4`, 目录权限为`5`
* `第三个rwx`: **其他组**对其的权限 (也就是文件所属组之外的所有用户), 新创建的文件默认权限为`4`, 目录权限为`5`



而对于`文件`或者`目录`来说, `r` `w` `x`三个权限的含义是不同的:

|      |       `r`        |                       `w`                        |     `x`      |
| :--: | :--------------: | :----------------------------------------------: | :----------: |
| 文件 |  读(cat, more)   |                    写(>, vim)                    | 执行(./xxx)  |
| 目录 | 浏览目录内容(ls) | 创建/删除/移动/重命名目录中的文件(touch, rm, mv) | 进入目录(cd) |



我们还可以使用数组来代表一组`rwx`, 使用二进制数即可:

* `rwx` = `111` = `4+2+1` = `7`
* `rw-` = `110` = `4+2+0` = `6`
* `r-x` = `101` = `4+0+1` = `5`
* `r--` = `100` = `4+0+0` = `4`



### 节点的默认拥有者/所属组

新建的的一个文件/目录, 他们都各自拥有默认的*拥有者*, *所属组* :

* 拥有者: 默认的所有者就是该文件的`创建者`
* 所属组: 默认的所属组就是该文件的`创建者`所在的`主组`



### 修改节点的拥有者/所属组

* `chown [选项] [用户名] [文件名]`: 可以修改文件的所有者 (也可以修改所属组)
  * `chown firene NODE`: 修改所有者
  * `chown firene:admin NODE`: 修改所有者和所属组
  * `-R`: 递归修改, 当然包含选中的节点.
* `chgrp [选项] [用户名] [文件名]`: 可以修改文件的所属组
  * `chgrp firene NODE`: 修改所属组
  * `-R`: 递归修改, 当然包含选中的节点.

```shell
chown firene note					# 将文件note的所有者修改为firene
chown -R firene notes				# 将目录notes及其里面的所有节点的所有者修改为firene
chown firene:admin notes			# 将目录notes的所有者修改为firene, 所属组修改为admin
chgrp admin note					# 将文件note的所属组修改为admin
chgrp -R admin notes				# 将目录notes及其里面的所有节点的所属组修改为admin
```



### 节点的默认权限

新创建的一个文件/目录, 他们都各自拥有默认的*权限*:

- 文件: `rw-r--r--` = `644` (默认文件是不可执行的, 即没有`x`权限)
- 目录: `rwxr-xr-x` = `755` (目录当然要有`x`权限, 也就是可以进入的权限)



### 修改节点的权限

* `chmod [选项] [权限表达式] [节点名]`: 修改节点的权限
  * `644`: 数字型的权限表达式, 含义就是`rw-r--r--`
  * `[ugo][+-=][rwx]`
    * `u+rw`: 给所属用户添加`r`和`w`权限
    * `o-x`: 给其他人去除`x`权限
    * `u=rw`: 将*属主*的权限设置为`rw-`
    * `ugo=`: 将*属主*和*属组*和*其他人*的权限设置为`---`
  * `chmod u=rw,g=rw,o=r file`: 这种写法也是支持的哦~
    * 上面这种写法相当于`chmod u=rw` + `chmod g=rw` + `chmod o=r` 三条命令依次执行.
  * `chmod -R 644 notes`: 递归修改不再废话.





# 网络配置

### 遇到的问题

我们现在开机启动我们的Linux的时候, 会发现一个问题, 我们总要去点一下右上角的`小红叉`来让eth0去连接网络... 能不能不用每次都点呢? 当然是可以的, 只需要做一些简单配置:

![1556859466709](assets/1556859466709.png)

这样以后开机就会自动连接网络了, 可是这样子的ip地址是不固定的, 我们可以通过图形界面来设置我们的ip地址, 但是实际的服务器开发的时候是无法使用图形界面的, 那么我们应该如何配置呢?



### 配置固定IP

`/etc/sysconfig/network-scripts/ifcfg-eth0`

```ini
ONBOOT=yes						# 必须配置! 启动配置的意思
BOOTPROTO=static				# 必须配置! none改为static, 静态ip的意思
IPADDR=192.168.124.93			# 必须配置! 设置好固定ip
GATEWAY=192.168.124.2			# 必须配置! 设置好网关
DNS1=192.168.124.2				# 建议配置! 设置好DNS与网关一致即可
DEVICE=eth0
TYPE=Ethernet
UUID=17449808-fbfe-4230-bbea-06248cfc650b
NM_CONTROLLED=yes
HWADDR=00:0C:29:04:55:CD
PREFIX=24
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME="System eth0"
LAST_CONNECT=1556858993
```

我们通过图形界面修改的其实就是这个配置文件而已啦~ 然后我们可以通过下面的命令来刷新网络服务:

```shell
service network restart
```



### 网络监控

* `netstat -anp` 

```shell
[root@centos6 ~]# netstat -anp | more
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      2109/sshd           
tcp        0      0 127.0.0.1:631               0.0.0.0:*                   LISTEN      1941/cupsd          
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      2282/master         
tcp        0     64 192.168.124.93:22           192.168.124.1:3803          ESTABLISHED 3307/sshd           
tcp        0      0 :::22                       :::*                        LISTEN      2109/sshd           
tcp        0      0 ::1:631                     :::*                        LISTEN      1941/cupsd          
tcp        0      0 ::1:25                      :::*                        LISTEN      2282/master         
udp        0      0 0.0.0.0:631                 0.0.0.0:*                               1941/cupsd          
udp        0      0 0.0.0.0:60485               0.0.0.0:*                               2551/local 
```

通过这个命令, 我们就可以看到网络状态了...





# 进程 & 服务

## 进程管理

### 查看进程

* `ps aux` | `ps -ef`
  * 参数说明: `a` 显示所有进程 `u` 以用户友好的格式显示 `x` 显示后台进程
  * 列说明
    * `command` 就是说这个进程是通过什么命令运行起来的
    * `time` 指的是占用cpu的总时间
    * `stat` 指的是当前线程的状态: `s` 终端 `r` 运行 `t` 停止 `z` 僵死 `d` 不可中断

```shell
[root@centos6 ~]# ps aux
用户名|PID|CPU占用率|内存占用率|虚拟内存|物理内存|终端|状态|开始时间|CPU时间|启动命令
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0  19344  1548 ?        Ss   13:18   0:04 /sbin/init
root         2  0.0  0.0      0     0 ?        S    13:18   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    13:18   0:00 [migration/0]
......
------------------------------------------------
[root@centos6 ~]# ps -ef
用户名 | 进程ID | 父进程ID | 优先级 | 开始时间 | 终端 | CPU时间 | 启动命令
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 13:18 ?        00:00:04 /sbin/init
root         2     0  0 13:18 ?        00:00:00 [kthreadd]
root         3     2  0 13:18 ?        00:00:00 [migration/0]
```

---

* `top` 动态监控进程信息, 类似于任务管理器
  * 参数说明: `-d 10` 设置每10s刷新一次 `-c` 显示完整信息 `-n 5` 刷新5次后关闭
  * 操作说明:
    * `<Space>` 立即刷新 `n` 显示的进程数 `k` 杀死指定pid的进程 `q` 退出 `u` 按用户过滤
    * `l` 是否显示top `t` 是否显示Tasks和Cpu `m` 是否显示Mem和Swap
    * `P` 按照CPU排序 `M` 按照内存排序 `N` 按照pid进行排序
    * `x` 高亮排序列 `>` `<` 变更排序列(高亮状态下)

```shell
top - 14:13:49 up 55 min,  3 users,  load average: 0.31, 0.13, 0.13
Tasks: 199 total,   1 running, 198 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.1%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   2046320k total,   594032k used,  1452288k free,    25832k buffers
Swap:  2047996k total,        0k used,  2047996k free,   202728k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND             
 2712 root      20   0  334m  19m  15m S  0.3  1.0   0:09.37 vmtoolsd            
 1552 root      20   0  175m 4392 3596 S  0.2  0.2   0:08.61 vmtoolsd          
 3513 root      20   0 15032 1300  940 R  0.2  0.1   0:00.18 top 
```

---

* `pstree` 自己玩玩吧...



### 终止进程

```shell
kill -9 PID
```

上面的命令就可以*强制终止*一个进程, `-9`的含义就是如此.

```shell
kill -u firene
```

这就是杀死用户firene的所有进程.

```shell
killall gedit
```





## 服务管理

服务是一种特殊的进程, 特点如下:

* 后台运行.
* 同样都会监听一个端口, 等待其他程序的请求.

我们在Linux当中也可以称服务为"守护进程", 其实就是监听器啦~



### 开启/关闭服务

* `service [服务名] [start|stop|restart|reload|status]`

常见的服务有, mysqld, 防火墙, sshd(就是通过22端口远程登录那个服务啦), 下面以关闭防火墙为例:

```shell
[root@centos6 ~]# service iptables status
表格：filter
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
2    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
4    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
5    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         
1    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         

[root@centos6 ~]# service iptables stop
iptables：将链设置为政策 ACCEPT：filter                    [确定]
iptables：清除防火墙规则：                                 [确定]
iptables：正在卸载模块：                                   [确定]
You have new mail in /var/spool/mail/root
[root@centos6 ~]# service iptables status
iptables：未运行防火墙。
```

可以看到, 默认防火墙只开启了`22`端口, 所以说有些软件无法连接就是因为这个了.



> **如何测试虚拟机的某个端口是否开启?**
>
> 可以在Window上安装**telnet**组件, 然后使用`telnet 192.168.124.93 22`命令来测试虚拟机端口是否开启.



> **Centos7不支持`service`命令了, 采用了`systemctl`来处理服务了.**



### 查看服务

如果想查看Linux系统当中到底有哪些服务, 可以通过以下两种方式:

* `setup` => `系统服务` (被标记`*`号的就是已经启动的, 按`<Space>`就可以切换)
* `ls /etc/init.d` 这个目录中的每一个文件都代表了一个服务

> ```shell
> [root@centos6 etc]# ll init.d
> lrwxrwxrwx. 1 root root 11 5月   2 21:15 init.d -> rc.d/init.d
> ```
>
> **`/etc/init.d`是`/etc/rc.d/init.d`的软链接.**



### 添加/删除服务

你也注意到了, 其实所有的服务都保存在了`/etc/init.d`当中, 我们想要添加服务肯定要将自己的程序放入到这个文件夹, 所以说分如下的两步:

* [1] 将自己的服务脚本放入到`/etc/init.d`当中 (保证状态为可执行)
* [2] `checkconfig --add 服务脚本` 添加系统服务
* [2.1] `checkconfig --del 服务脚本` 移除系统服务



### 服务的运行级别

Linux开机的流程是: `开机` => `BIOS` => `/boot` => `init进程1` => `运行级别` => `运行级别对应的服务`

服务的运行级别就是Linux系统的7个运行级别, 不再赘述, 为什么要说运行级别呢?

==一个服务在不同的7个运行级别都可以分别设置是否自启动==



### 服务自启动配置

`service xxx start/stop` 只能暂时地关闭或者开启服务, 但是重启之后是无法生效的, 我们需要通过下面的命令来处理:

* `chkconfig --list` | `chkconfig`

  * ```shell
    [root@centos6 ~]# chkconfig --list
    NetworkManager 	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
    abrt-ccpp      	0:关闭	1:关闭	2:关闭	3:启用	4:关闭	5:启用	6:关闭
    abrtd          	0:关闭	1:关闭	2:关闭	3:启用	4:关闭	5:启用	6:关闭
    acpid          	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
    atd            	0:关闭	1:关闭	2:关闭	3:启用	4:启用	5:启用	6:关闭
    ......
    ```

  * 可以看出, 这个命令就是查看每个服务在不同的系统运行级别下是否自启动.

* `chkconfig --list [服务名]` 查看具体某个服务的是否自启动状态.

* `checkconfig --level [级别列表] [服务名] [on/off]`

  * ```shell
    [root@centos6 ~]# chkconfig --level 35 iptables off
    [root@centos6 ~]# chkconfig --list | grep iptables
    iptables       	0:关闭	1:关闭	2:启用	3:关闭	4:启用	5:关闭	6:关闭
    ```

  * 上面的这条命令就可以关闭防火墙啦~ 再也不用担心防火墙搞事惹.

  * `checkconfig iptables off` 如果不指定`--level`就是指定所有级别.





# 软件安装



## RPM

### 什么是RPM

`RPM = RedHat Package Manager` 也就是红帽软件包管理工具, 由于设计得比较好, 已经成为了一种标准, 我们可以下载`.rpm`包, 这有点像Windows系统中的`setup.exe`, 就可以安装软件了.



### RPM包查看

* `rpm -qa` 查看本机中所有的rpm包

```shell
[root@centos6 ~]# rpm -qa | grep 'firefox'
firefox-45.0.1-1.el6.centos.x86_64
```

* `firefox-45.0.1-1.el6.centos.x86_64`
  * `firefox` 软件包名称
  * `45.0.1-1` 版本信息
  * `el6.centos.x86_64` 适用的系统版本
    * `el6` 6版本, 指centos6
    * `centos` 系统是centos
    * `x86_64` 64位的系统 `i686` `i386` 32位的系统 `noarch` 通用

一些其他的包名称:

```txt
gdm-user-switch-applet   -   2.30.4-64       .   el6.x86_64
upstart				     -   0.6.5-16        .   el6.x86_64
festvox-slt-arctic-hts   -   0.20061229-18   .   el6.noarch
libpng                   -   1.2.49-2        .   el6_7.x86_64
```

* `rpm -q firefox` 查询是否安装了指定的包

  * ```shell
    [root@centos6 ~]# rpm -q firefox
    firefox-45.0.1-1.el6.centos.x86_64
    [root@centos6 ~]# rpm -q aaa
    package aaa is not installed
    ```

* `rpm -qi firefox`  查询软件包的详细信息

  * ```shell
    [root@centos6 ~]# rpm -q
    rpm: no arguments given for query
    [root@centos6 ~]# rpm -qi firefox
    Name        : firefox                      Relocations: (not relocatable)
    Version     : 45.0.1                            Vendor: CentOS
    Release     : 1.el6.centos                  Build Date: 2016年05月12日 星期四 04时49分
    ......
    ```

* `rpm -ql firefox` 查询firefox在哪里创建了文件... 也就是安装到了哪里呢?

  * ```shell
    [root@centos6 ~]# rpm -ql firefox
    /etc/firefox/pref
    /usr/bin/firefox
    /usr/lib64/firefox
    /usr/lib64/firefox/LICENSE
    ......
    ```

* `rpm -qf /etc/passwd` 查询指定的文件属于哪个软件包的

  * ```shell
    [root@centos6 ~]# rpm -qf /etc/passwd
    setup-2.8.14-20.el6_4.1.noarch
    ```



### 卸载RPM包

* `rpm -e 软件包名称` 卸载很简单 (如果其他的软件依赖于要卸载的软件就会报错)
* `rpm -e --nodeps 软件包名称` 强制卸载, 不考虑依赖... 不建议这么做



### 安装RPM包

* `rpm -ivh [RPM包.rpm]` 安装也很简单
  * 选项解释: `i` 安装 `v` 提示 `h` 进度条

我们可以试着尝试安装刚刚卸载掉的火狐浏览器, 至于安装包在哪里, 可以去我们下载的`centos iso 镜像`当中找到, ==当光驱插入到Linux系统中, 会挂载到/media目录.==

```shell
[root@centos6 ~]# cd /media
[root@centos6 media]# ll
总用量 4
dr-xr-xr-x. 7 root root 4096 5月  23 2016 CentOS_6.8_Final
[root@centos6 media]# cd CentOS_6.8_Final/Packages/
[root@centos6 Packages]# ll | grep firefox
-r--r--r--. 2 root root 77493120 5月  12 2016 firefox-45.0.1-1.el6.centos.x86_64.rpm
```

接下来, 我们将文件拷贝到一个空闲目录即可开始rpm的安装了.

```shell
[root@centos6 Packages]# cp firefox-45.0.1-1.el6.centos.x86_64.rpm /opt
[root@centos6 opt]# rpm -ivh firefox-45.0.1-1.el6.centos.x86_64.rpm 
warning: firefox-45.0.1-1.el6.centos.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Preparing...                ########################################### [100%]
   1:firefox                ########################################### [100%]
[root@centos6 opt]# rpm -q firefox
firefox-45.0.1-1.el6.centos.x86_64
```

这样子我们完成了firefox的安装了.



## YUM

`YUM` 是Shell前端软件包管理器, 基于RPM包管理, 能够==从指定的服务器自动下载RPM包并安装, 并能够解决依赖关系==. 说到底, YUM就是为了简化RPM的安装.

### YUM常用命令

* `yum list` 查看可以从服务器下载安装的软件包, CentOS系统就是CentOS提供的yum服务器.
* `yum list installed` 查看已经安装好的RPM包
* `yum search [关键字]` 根据关键字查询RPM包
* `yum install [软件包名称]` 安装软件包(需要联网)
* `yum -y install [软件包名称]` 同样是安装, 这个不需要一直输入`y`|`yes`
* `yum remove [软件包名称]` 卸载软件包, 解决依赖关系
* `yum -y remove [软件包名称]` 同理
* `yum update` 更新安装的所有的软件包和系统内核 (核弹命令, 拒绝使用!)



### YUM配置文件









## Java开发所需软件

### JDK

* [1] 解压安装包

```shell
[root@centos6 opt]# tar -zxvf jdk-7u79-linux-x64.gz
```

* [2] 移动 & 重命名

```
[root@centos6 opt]# mv jdk1.7.0_79/ /usr/local/bin/java
```

* [3] 配置环境变量

```shell
[root@centos6 opt]# vim /etc/profile
Go		# vim移动到最后, 然后下一行追加如下内容
# java
JAVA_HOME=/usr/local/java
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib
export JAVA_HOME PATH CLASSPATH
:wq		# 退出vim
```

* [4] 刷新环境变量

```shell
[root@centos6 opt]# source /etc/profile
```

* [5] 测试



### Tomcat

* [1] 解压安装包

```shell
[root@centos6 opt]# tar -zxvf apache-tomcat-7.0.94.tar.gz 
```

* [2] 移动 & 重命名

```shell
[root@centos6 opt]# mv apache-tomcat-7.0.94 /usr/local/bin/tomcat
```

* [3] 配置环境变量 & 刷新配置文件

```shell
[root@centos6 opt]# vim /etc/profile
# tomcat
export CATALINA_HOME=/usr/local/bin/tomcat
export PATH=$PATH:$CATALINA_HOME/bin
export CLASSPATH=$CLASSPATH:$CATALINA_HOME/lib
[root@centos6 opt]# source /etc/profile
```

* [4] 启动tomcat并且监控日志信息

```shell
[root@centos6 ~]# startup.sh & tail -f $CATALINA_HOME/logs/catalina.out
```



### MySQL

* [0] 卸载已有的mysql, 如果提示依赖错误, 就`--nodeps`强制删除

```shell
[root@centos6 opt]# rpm -qa | grep mysql
mysql-libs-5.1.73-7.el6.x86_64
[root@centos6 opt]# rpm -e mysql-libs
error: Failed dependencies:
	libmysqlclient.so.16()(64bit) is needed by (installed) postfix-2:2.6.6-6.el6_7.1.x86_64
	libmysqlclient.so.16(libmysqlclient_16)(64bit) is needed by (installed) postfix-2:2.6.6-6.el6_7.1.x86_64
	mysql-libs is needed by (installed) postfix-2:2.6.6-6.el6_7.1.x86_64
[root@centos6 opt]# rpm -e --nodeps mysql-libs
```

* [1] 解压压缩包

```shell
[root@centos6 opt]# tar -zxvf mysql-5.6.14.tar.gz
```

* [2] 安装编译所需的包

```shell
[root@centos6 opt]# yum -y install make cmake gcc-c++ bison-devel ncurses-devel
```

* [3] 

```shell
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci
```









# 附加内容

## 环境变量

### Linux环境变量的特点

* 引用环境变量需要使用语法`$XXX`, 而不是`%XXX%`
* 环境变量的多个值之间使用`:`分隔, 而不是`;`



### 查看环境变量

* `echo $LANG` 显示某个环境变量的值
* `env` 显示所有的环境变量
* `export -p` 查看所有的环境变量



### 临时修改环境变量

* `export PATH=$PATH:/usr/local/java/bin` 重启之后失效



### 永久修改环境变量

* `~/bash_profile` 用户级别的配置文件
* `/etc/profile` 系统级别的配置文件

在上面这两个配置文件中, 我们都可以通过`export KEY=VALUE`的语法来永久地配置我们的环境变量:

```shell
PATH=$PATH:$HOME/bin
export PATH
```

或者

```shell
export PATH=$PATH:$HOME/bin
```

这样就可以了.



### 修改之后生效

* `source /etc/profile` 修改之后不会马上生效, 可以执行这条命令
* `logout` 注销也可以



## 防火墙

防火墙在Linux当中就是一个服务: `iptables`, 我们可以使用通用的linux命令来操作它:

* `service iptables status` 查看防火墙状态
* `service iptables stop` 临时关闭防火墙
  * `start` 启用 `restart` 重启
* `chkcofig --level 35 iptables off` 运行级别为3或5的时候, 开机不要自启动防火墙

可是不是所有时候我们都想要彻底地关闭防火墙, 这样会出现一些安全问题, 我们能不能配置防火墙呢?



### 防火墙配置

Linux系统中防火墙的配置文件是`/etc/sysconfig/iptables`, 我们修改其中内容即可:

```shell
[root@centos6 ~]# more /etc/sysconfig/iptables
......
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
......
```

  * 上面这一行的含义就是开放`22`端口, 我们想要开放其他端口, 复制即可

  * ```shell
    # 开放8080端口
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
    # 开放8081-8090端口
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 8081:8090 -j ACCEPT
    ```

* 不要忘记重启防火墙: `service restart iptables`



## 主机名

### 查看主机名

* `hostname` 直接输入这个命令就可以查看当前的主机名了



### 临时修改主机名

* `hostname [主机名]` 这样子就可以临时修改主机名了



### 永久修改主机名

* `vim /etc/sysconfig/network` => `HOSTNAME=[主机名]`
* `vim /etc/hosts` => `替换`
* `reboot` 重启机器才能永久生效, 也可以通过`hostname`命令设置, 避免重启.





## Crontab定时任务

### Crontab配置文件

```shell
[root@centos6 ~]# cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```

`/etc/crontab`就是crond的配置文件, 我们依次说明:

* `SHELL=`: 指定了系统要使用的bash
* `PATH=`: 指定了执行命令的路径
* `MAILTO=`: 指定了任务信息会通过邮件发送给root用户
* `HOME=`: 执行命令或者脚本的时候使用的主目录



### Crontab命令

```shell
crontab -e
```

这条命令会让我们去编辑一张当前用户的"任务表", 这张表中数据的结构类似于这样子:

```shell
0,15,30,45 18-06 * * * /bin/echo `date` > /dev/tty1
0,15,30,45 18-06 * * * /bin/echo `date` > /dev/tty1
0,15,30,45 18-06 * * * /bin/echo `date` > /dev/tty1
#   任务执行时间的表达式  |  要执行的任务(命令)
```

##### 命令的相关参数

* `-u` 指定用户, 如果不指定, 就是当前用户
* `-e` 编辑时刻表
* `-r` 删除目前的时刻表
* `-l` 查看时刻表

##### 表达式写法

* `* * * * *` = `每分钟 每小时 每天 每月 每星期`

  * ```txt
    # Example of job definition:
    # .---------------- minute (0 - 59)
    # |  .------------- hour (0 - 23)
    # |  |  .---------- day of month (1 - 31)
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name command to be executed
    ```

* `24` 代表具体的时刻

* `*` 代表每一个时刻, 每个月, 每个小时, 每个星期几

* `3-6` 代表3,4,5,6这几个时刻都要执行

* `*/10` 代表每隔10个单位就要一次

* `3,6,9` 代表第3个单位, 第6个单位, 第9个单位都要执行以下

##### 示例

* `0 * * * *` 每月, 每天, 每小时 的 每个0分的时候 执行一次
* `*/20 6-12 * 12 *` 每个12月, 每天, 6小时到12小时内的每个小时, 每隔20分钟执行一次
* `0 17 * * 1-5` 每周一到周五, 每个下午五点的0分都执行一次
* `20 0-23/2 * * *` 每个月, 每天, 所有小时选中, 每隔2个小时, 的20分执行一次
* `0 23-7/2,8 * * * ` 每天晚上11点到早上7点的每两个小时每个0分的时候, 以及早上8点0分的时候..



## 克隆虚拟机

在我们以后的学习过程中, 需要多次克隆虚拟机, 那么每个克隆的虚拟机需要变化的部分如下:

* 主机名
* 静态IP地址
* HOSTS配置 (按需要)



### 主机名配置

```shell
vim /etc/sysconfig/network
```



### 静态IP配置

由于我们克隆的虚拟机的网卡的物理地址和原虚拟机的网卡地址是一样的, 这是不可以的, 所以说, 我们首先应该修改网卡的物理地址

vim /etc/udev/rules.d/70-persistent-net.rules

##### 1. 修改网卡物理地址

```shell
vim /etc/udev/rules.d/70-persistent-net.rules
```

![1556945712447](assets/1556945712447.png)

1. 删除eht0网卡
2. 修改eth1为eth0
3. 拷贝物理地址

最终的修改效果如下:

```shell
# This file was automatically generated by the /lib/udev/write_net_rules
# program, run by the persistent-net-generator.rules rules file.
#
# You can modify it, as long as you keep each rule on a single
# line, and change only the value of the NAME= key.

# PCI device 0x8086:0x100f (e1000)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="需要拷贝的部分", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
```



##### 2. 修改静态IP地址

```shell
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

1. 修改网卡物理地址为[上一步拷贝的网卡地址]
2. 修改静态IP即可, 保持原有规则

最终的修改效果如下:

```shell
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.124.61				# 新的IP地址
GATEWAY=192.168.124.2
DNS1=192.168.124.2
HWADDR=00:0c:29:be:db:af			# 刚才拷贝的物理地址
```



### HOST配置(可选)

我们在学习Hadoop的时候就需要配置HOST, 很简单:

```
vim /etc/hosts
```

例如Hadoop的学习, 我们可能会修改成这样子:

```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.124.61 hadoop-1
192.168.124.62 hadoop-2
192.168.124.63 hadoop-3
192.168.124.64 hadoop-4
192.168.124.65 hadoop-5
192.168.124.66 hadoop-6
192.168.124.67 hadoop-7
192.168.124.68 hadoop-8
192.168.124.69 hadoop-9
```



### 修改完之后记得重启!

```
reboot
```



### 熟练之后

```shell
[root@centos6 ~]# vim /etc/sysconfig/network
[root@centos6 ~]# vim /etc/udev/rules.d/70-persistent-net.rules
[root@centos6 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
[root@centos6 ~]# vim /etc/hosts
[root@centos6 ~]# reboot
```

