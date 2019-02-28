---
title: Linux常用命令
date: 2019-02-28 14:53:27
tags: Linux
---

以下是阅读鸟哥的 Linux 私房菜第四版做的笔记。

#### 一、Linux 入门

##### 1. man 命令

虽然绝大多数数的命令带有--help 选项，但是想查看某命令的详细信息还是 man 比较详细。

```bash
man man #查看man命令的manual
man 1 man #查看id为1的man命令
man -f man #查看命令名中包含man字样的所有命令
man -k man #查看命令简介中包含man字样的所有命令
```

##### 2.info 命令

虽然 man 命令很强大，但是一大段的文字在 bash 中展现出来难免难以查阅。info 命令将 man 的内容拆成多个 node 可以让使用者像操作 html 一样来查看命令的使用详情。

```bash
info info #查看info命令的.info文件
```

进入 info 的使用界面，为了方便操作，info 的使用界面带有很多快捷键。

```bash
n #Next 跳到下一个Node
p #prev 跳到上一个Node
] #下一节点，与n不同的是，n会略过子节点直接跳到下一个统计节点，]不会
[ #上一节点，效果和]差不多
u #Up 上一节点
b 或 Home #Begining 会到节点顶部
e 或 End #End 回到节点底部
空格 或 PgDn #向下移动一个屏幕高度，相当翻页
回车 会 Del 或PgUp #向上移动一个屏幕高度
Tab #跳到下一个最近的链接
m #链接跳转，需要input 链接的名字
? #查看更多的操作快捷键
```

#### 二、文件权限

##### 1.改变文件属性和权限

```bash
chgrp #change group
chown #change owner
chmod # change mode
```

###### 1.改变文件所属组

Linux 的所有组的信息保留在/etc/group 文件内。该文件内的内容格式为：

```bash
#组名:密码:组id:组内所有成员
group_name:passwd:GID:user_list
```

为了安全，密码实际显示的是"X"，真正的密码保存在了/etc/shadow 内(该文件只有 root 才能打开)。

修改文件所属组方法如下

```bash
#用法
chgrp [options] group file ...
```

###### 2.改变文件所属用户

linux 的所有用户信息保留在/etc/passwd 文件内。改文件内的内容格式为：

```bash
#用户名:密码:用户id:用户的主要组id:可选字段，通常为了存放信息:home目录所在地址:shell
account:password:UID:GID:GECOS:directory:shell
```

同样，这里的密码显示的是'x'.

修改文件所属用户的方法如下

```bash
#用法
chown [options] user[:group] file ...
#修改index.html所属用户为allen
chown allen index.html
#修改index.html所属组为root
chown :root index.html
#递归修改index.html所属组为root所属用户为allen
chown -R allen:root index.html
```

###### 3.改变文件权限

修改文件的权限有两种方法，数字和符号

1. 数字

r:4 w:2 x:1,根据需要，将权限对应的数字相加即可得到一个数字。

```bash
#用法
chmod [options] mode file ...
#将index.html 的权限改为rwxr-xr--
chmod 754 index.html
```

2. 符号

a:所有对象 u:所属用户 g:所属组 o:其他人，根据需要，将对应的权限赋值给对应的对象就好

```bash
#用法
chmod [options] mode file ...
#将index.html 的权限改为rwxrwxrwx
chmod a=rwx index.html
```

如果想去掉或添加某个对象的某个权限，可以用'-'和'+'

```bash
#index.html的权限为rwxrwxrwx,现在想去掉其他用户的可执行权限
chmod o-x index.html
#index.html的权限为rwxrwxrw-,现在想添加其他用户的可执行权限
chmod o+x index.html
```

3. 隐藏权限

在老的 Ext 文件系统上，可以添加与查看隐藏权限,在 xfs 文件系统(CentOS 7 默认)内仅支持部分参数

```bash
chattr #添加隐藏权限
isattr #查看隐藏权限
```

4. 默认权限 umask

用户创建文件和文件是都会带有默认的权限。

```bash
umask #查看默认权限 4位数
umask -S #查看默认权限 标识符的形式
```

一般用户的默认 umask 是 0002，要如何解读这 4 位数呢。第一位是特殊权限的位置，后三位就是我们平时理解的 user group others。但是有所不同的是显示的数字是表示要排除的特权。

最后一位是 2，那表是要排除掉可写权限。于是 0002 对于新创建的文件，它的默认权限是-rw-rw-r;对于新创建的目录，它的默认权限是-rwxrwxr-x。

注意这里的文件和目录的默认权限有所不同，文件本身就默认没有可执行权限的，而目录默认是有可执行权限。因为对大部分文件来说，可执行权限没有意义，但对于目录来说就很有必要。

root 用户的默认 umask 是 0022，这是为了安全考虑，这样 root 创建的文件或目录，在默认情况下，其同组成员是无法修改的。

5. 特殊权限

SUID SGID TBIT 是三个特殊权限。

有时候会看到 rwsr-xr-x 的权限，这里的 s 就是 SUID。该权限的作用就是其他用户在执行的时候可以获得该文件所属用户的权限(注意仅在文件执行期间)。，SUID 仅可用在 binary program 上， 不能够用在 shell script 上面！以下是设置方法。

```bash
#将权限为 rwxr-xr-x的test修改为rwsr-xr-x
chmod 4755 test
```

有时候会看到 rwx--s--x 的权限，这里的 s 就是 SUID。其作用类似 SUID,就是在文件的执行期间其他用户可以获得该程序群组的支持。

SUID 同时也可以用在目录上，其作用就是使用者进入该目录时，它的有效群组会变为该目录的群组。同时，使用者在该目录下创建的文件群组和该目录的群组相同。

以下是设置方法。

```bash
#将权限为 rwxr-xr-x的test修改为rwxr-sr-x
chmod 2755 test
```

查看/tmp 的权限是，会看到 drwxrwxrwt,这个 t，这里的 t 就是 TBIT。它可以让使用者在它下面创建的文件仅有使用者自己和 root 能删除。

以下是设置方法。

```bash
#将权限为 rwxr-xr-x的test修改为rwxr-xr-t
chmod 1755 test
```

其实权限设置是有 4 位的，第一位表示的是特殊权限。SUID:4,SGID:2,TBIT:1。把他们相应的数字相加就能组合使用。注意如果出现大写的 S 或 T，那说明这是无效的特殊权限，因为其所在的位置没有设置可执行权限。

#### 三、文件与目录管理

##### 1.路径与文件、目录管理

###### 1.路径

```bash
cd #变换目录
pwd #显示当前的路径
```

特殊的目录一览

```bash
. #代表此层目录
.. #代表上一层目录
- #代表前一个工作目录
~ #代表“目前使用者身份”所在的主文件夹
~account #代表 account 这个使用者的主文件夹（account是个帐号名称）
```

###### 2.文件、目录管理

```bash
mkdir #创建文件夹
rmdir #移除文件夹
touch #创建文件，也可以用来刷新文件的mtime ctime atime
ls #查看目录下的文件
rm #删除文件
mv #移动文件 可以替代rename的功能
cp #拷贝
file #查看文件类型 如sticky directory
```

##### 2.文件内容查看

查看文件内容可以用各种编辑器例如 nano、vim 等。在 X window 下还可以用 gedit 编辑器。

如果仅仅只是查看文件，用编辑器就有点小题大做。linux 有直接查看文件的命令

```bash
cat #查看文件
tac #从文件尾部开始查看文件，与cat相反
nl #带行号的文件查看
more #带翻页的查看
less #也是带翻页的查看，比more强大，man page默认的就是less
head #查看文件头部
tail #查看文件尾部，通常加个-f选项可以watch文件的变化，可用来查看logger文件
od #查看二进制文件
```

##### 3.指令文件名的搜索

```bash
which #查看指令的可执行文件所在目录，它是按PATH的所规范的路径寻找
type #which查不到的，可能是bash自带的指令，此时就可以用type来查了
```

##### 4.文件的查找

```bash
find #强大的查找功能，但比较消耗性能和时间
whereis #由于只搜索某几个目录，所以比find速度快
locate #通过数据库查询，所以比较快。但是有时候，有些文件还没更新到数据库内，会导致查不到。
updatedb #手动更新数据库 方便locate查找
```

#### 四、磁盘与文件系统管理

##### 1.查看文件系统的 superblock

ext 文件系统

```bash
dumpe2fs [options] 设备文件名
```

xfs 文件系统

```bash
xfs_info 挂载点 | 设备文件名
```

##### 2.查看磁盘或目录的容量

```bash
df #列出文件系统的整体磁盘使用量
du #评估文件系统的使用量
```

```bash
#用法
df [options] [目录或文件名]
#将系统内所有的filesystem列出来
df -h
#查看/etc目录的可用的磁盘容量列出
df -h /etc
#将目前各个partition当中可用的inode数量列出
df -ih
```

```bash
#用法
du [options] [目录或文件名]
#列出目前目录下的所有目录的大小
du
#列出目前目录下的所有文件和目录的大小
du -a
```

##### 3.创建连接

```bash
#用法
ln [-sf] 来源文件 目标文件
-s #不带该参数，默认是创建实体连接(hard link)加上是创建符号连接(symbol link)
```

##### 4.查看磁盘分区情况

```bash
#查看磁盘分区情况
lsblk [options] [device]
```

```bash
#查看分区uuid
blkid
```

```bash
#列出磁盘的分区表类型与分区信息
parted device_name print
```

##### 5.磁盘分区：gdisk/fdisk

注意 MBR 分区表用 fdisk;GPT 分区表用 gdisk。

```bash
gdisk 设备名
```

分区完毕并写入后，分区表并没有更新。可以通过重启来更新，或者 partprobe 更新 linux 核心的分区表信息。

```bash
partprobe [-s]
```

我们可以用 lsblk 来查看分区表的姿态，也可以：

```bash
cat /proc/partitions #核心的分区纪录
```

##### 5.磁盘格式化

make filesysytem 创建文件系统即是格式化。

这部分理解比较吃力，请查阅鸟哥的 linux 382 页

格式化为 xfs 文件系统

```bash
mkfs.xfs  [-b bsize] [-d parms] [-i parms] [-l parms] [-L label] [-f] [-r parms] 设备名
```

格式化为 ext4 文件系统

```bash
mkfs.ext4 [-b size] [-L label] 设备名称
```

格式化为其他文件系统

```bash
mkfs
#查看支持的文件系统
mkfs[tab][tab]
```

##### 6.文件系统挽救

每个文件系统都有其对应的指令

xfs

```bash
xfs_repair [-fnd] 设备名称
```

ext4。fsck.ext4 这是个综合指令

```bash
fsck.ext4 [-pf] [-b superblock] 设备名称
```

##### 7.文件系统挂载与卸载

进行挂载前要确定好几件事：

- 单一文件系统不应该被重复挂载在不同的挂载点（目录）中；
- 单一目录不应该重复挂载多个文件系统；
- 要作为挂载点的目录，理论上应该都是空目录才是。

```bash
mount #比较复杂 请用man来查看
```

```bash
#鸟哥现在比较推荐UUID的方式来挂载，应为UUID是唯一标识符，不会发生重复
#挂载前用blkid查看要挂载的分区的UUID
blkid
#然后通过UUID进行挂载到对应的目录，注意该目录必须要存在
mount UUID='分区的UUID' 目录
```

有挂载就有卸载

```bash
umount [-fn] 设备文件名或挂载点
```

##### 7.磁盘/文件系统参数修订

```bash
mknod
xfs_admin #修改 XFS 文件系统的 UUID 与 Label name
tune2fs #修改 ext4 的 label name 与 UUID

```

一个有趣的命令 uuidgen

```bash
uuidgen #生成一个新的uuid
```

##### 8.开机挂载 /etc/fstab 及 /etc/mtab

修改/etc/fstab 即可开机自动挂载

#### 五、文件与文件系统的压缩,打包与备份

##### 1.文件压缩

压缩指令 gzip,bzip2,xz

###### 1.gzip/zcat/zmore/zless/zgrep

gzip 同时可以解压 compress zip gzip 文件。

```bash
#用法
gizp [options] 压缩文件
#压缩文件，并查看压缩比
gizp -v services
#保留原文件的文件名，并且创建压缩文件！
gizp -c services > services.gz
#解压
gizp -d services.gz
```

zcat/zmore/zless/zgrep 分别对应 cat/more/less/grep。用来直接查看压缩文件内容。

```bash
#查看services.gz的内容
zcat services.gz
#查看压缩文件内带'http'字样的文本
zgrep -n 'http' services.gz
```

压缩的时候配合上 time 命令，还可以了解压缩用的时间。

```bash
time gzip services
```

###### 2.bzip2/bzcat/bzmore/bzless/bzgrep

用法和 gizp 相同，更多信息可以 man 它。它的优点是压缩率比 gzip 更高，同时花的时间也会多一些。

###### 3.xz/xzcat/xzmore/xzless/xzgrep

用法和 gizp/bzip2 相同，它的压缩比例比 bzip2 还要高，因此比较花费时间。

它还提供了一些更方便的 option

```bash
#列出压缩文件详细息信息
xz -l services.xz
#保留原文件的文件名，并且创建压缩文件！
xz -k services
```

##### 2.打包指令

打包命令有点复杂，以下列出了常用的简单命令。高级用法请自行 man 它。

```bash
#压缩 将z改成j可以用bzip2压缩，改成J可以用xz压缩
tar -zcv -f filename.tar.gz 要被压缩的文件或目录名称（可以多个,用空格隔开）
#查看压缩包
tar -ztv -f filename.tar.gz
#解压
tar -zxv -f filename.tar.gz [要解开的某个文件] -C 欲解压缩到的目录
#参数说明
-z 通过gzip的支持进行压缩/解压
-j 通过bzip2的支持进行压缩/解压
-J 通过xz的支持进行压缩/解压
-c 创建打包文件
-t 查看打包文件
-x 解开压缩包
-v 列出压缩包内的文件
-f 后面要立刻接要被处理的文件名！建议 -f 单独写一个选项啰！（比较不会忘记）
-C 这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。

-p 保留备份数据的原本权限与属性，常用于备份（-c）重要的配置文件
-P 保留绝对路径，亦即允许备份数据中含有根目录存在之意；
```

tar 可以用来备份文件

```bash
#备份/etc目录,这里的time命令只是为了查看时间开销
#p是为了保留原本的权限与属性。
time tar -jcpv -f /root/etc.bar.bz2 /etc
```

tar 可以取出压缩包内的某个文件

```bash
#以上面备份好的文件为例
#我们先查看压缩包，找到要取出的文件
tar -jt /root/etc.bar.bz2 | grep 'shadow'
#发现要找的文件目录为 etc/shadow
#然后用下面的方法取出
tar -jxv /root/etc.bar.bz2 etc/shadow
```

tar 打包的时候可以将不需要的文件排除在外

```bash
tar -jcv -f /root/system.tar.bz2 --exclude=/root/etc* --exclude=/root/sysytem.tar.bz2 /etc /root
```

tar 可以仅备份比某个时刻要新的文件

```bash
#查找比/etc/passwd的变更日期还新的文件
find /etc --newer /etc/passwd
#如果我们只备份比/etc/passwd新的内容，需要先知道/etc/passwd更变日期
ll /etc/passwd
#/etc/passwd变更日期为2018/5/5
bar -jcv -f /root/etc.newer.than.passwd.tar.bz2 --newer-mtime="2018/05/05" /etc/*
```

##### 3.XFS 文件系统的备份与还原

使用 tar 通常是针对目录系统来进行备份的工作，如果想针对整个 xfs 文件系统进行备份需要 xfsdump。

还原的话用 xfsrestore。

该部分内容比较麻烦，请看鸟哥第四版 8.5 部分。

拷贝还可以用 dd。dd 该方法是一个扇区一个扇区拷的。

```bash
dd -if input_file -of output_file -bs 每个block的大小(默认一个扇区的大小) count 多少个block
```
