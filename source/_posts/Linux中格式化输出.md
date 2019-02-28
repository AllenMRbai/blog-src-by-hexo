---
title: Linux中格式化输出
date: 2019-02-28 15:32:55
tags: Linux
---

### 目录

- 格式化打印：printf
- 数据处理工具：awk
- 文件对比工具：diff cmp patch
- 文件打印：pr

#### 格式化打印：printf

我们在 bash 中想输出一个表格，但由于每个字段的长度不同，会导致输出的样子像下面这样，很乱，不方便看。

```bash
# printf.txt
Name Chinese English Math Average
DmTsai 80 60 92 77.33
VBird 75 55 80 70.00
Ken 60 90 70 73.33
```

我们想让表格格式化成下面那样，我们就可以用 printf 命令了。

```bash
Name   Chinese  English  Math  Average
DmTsai 80       60       92    77.33
VBird  75       55       80    70.00
Ken    60       90       70    73.33
```

用法

```bash
printf '打印格式' 实际内容
#选项与参数：
#关于格式方面的几个特殊样式：
#   \a 警告声音输出
#   \b 倒退键（backspace）
#   \f 清除屏幕 （form feed）
#   \n 输出新的一行
#   \r 亦即 Enter 按键
#   \t 水平的 [tab] 按键
#   \v 垂直的 [tab] 按键
#   \xNN NN 为两位数的数字，可以转换数字成为字符。
#关于 C 程序语言内，常见的变量格式
#   %ns 那个 n 是数字， s 代表 string ，亦即多少个字符；
#   %ni 那个 n 是数字， i 代表 integer ，亦即多少整数码数；
#   %N.nf 那个 n 与 N 都是数字， f 代表 floating （浮点），如果有小数码数，假设我共要十个位数，但小数点有两位，即为 %10.2f 啰！
```

```bash
#例 将刚才的print.txt格式换成好看的表格
printf '%s\t %s\t %s\t %s\t %s\t \n' $(cat printf.txt)

#例 将刚才的print.txt的标题去掉，并将每列固定长度
printf '%10s %5i %5i %5i %8.2f \n' $(cat printf.txt | grep -v Name)
```

printf 另一个用处是将 16 进制显示为 ASCII

```bash
#例
printf '\x45\n'
```

#### 数据处理工具：awk

```bash
awk '条件类型1{动作1} 条件类型2{动作2} ...' filename
```

内置变量

| 变量名 | 意义                            |
| ------ | ------------------------------- |
| NF     | 每一行 （\$0） 拥有的字段总数   |
| NR     | 目前 awk 所处理的是“第几行”数据 |
| FS     | 目前的分隔字符，默认是空白键    |
| \$N    | 第几个字段                      |

```bash
#例
last -n | awk '{print $1 "\t lines:" NR "\t columns:" NF}'
```

BEGIN 和 END

```bash
#例 awk默认分隔符是空格，但现在我们想改成‘:’。我们可以设置FS为‘:’。但下面这样写只有第二行开始生效。
cat /etc/passwd | awk '{FS=":"} $3 < 10 {print $1 "\t " $3}
#例 为了让它从第一行就开始生效。可以用BEGIN关键字
cat /etc/passwd | awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t " $3}'
```

awk 还可以对表内的数据做处理。

```bash
# pay.txt
Name 1st 2nd 3th
VBird 23000 24000 25000
DMTsai 21000 20000 23000
Bird2 43000 42000 41000
```

假设我们有上面那个表(pay.txt)，想添加一列 Total 做求和。我们可以这样写

```bash
# 下面的‘>’表示换行。在一个动作内写多个命令可以用回车或‘;’隔开。而且这里定义的total变量可以直接使用，不需要要‘$’
cat pay.txt | \
>awk 'NR==1{printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total" }
> NR>=2{total = $2 + $3 + $4
> printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'
```

awk 动作内{}是支持 if 的，上面的命令也可以写成下面那样

```bash
cat pay.txt &#124; \
> awk '{if（NR==1） printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total"}
> NR&gt;=2{total = $2 + $3 + $4
> printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'
```

#### 文件对比工具：diff cmp patch

##### diff

diff 是比较两个文件间的差异，并以行为单位进行比较。

```bash
# 用法
diff [-bBi] from-file to-file
#选项与参数：
#from-file ：一个文件名，作为原始比对文件的文件名；
#to-file ：一个文件名，作为目的比对文件的文件名；
#注意，from-file 或 to-file 可以 - 取代，那个 - 代表“Standard input”之意。
#-b ：忽略一行当中，仅有多个空白的差异（例如 "about me" 与 "about me" 视为相同
#-B ：忽略空白行的差异。
#-i ：忽略大小写的不同。
```

现在有 passwd.old passwd.new 文件。passwd.old 拷贝自/etc/passwd;passwd.new 是在/etc/passwd 的基础上做了修。删除了第四行，并将第六行替换为‘no six line’。

```bash
mkdir -p /tmp/testpw
cd /tmp/testpw
cp /etc/passwd passwd.old
cat /etc/passwd | sed -e '4d' -e '6c no six line' > passwd.new
```

现在我们可以查看新旧的 passwd 的差别了

```bash
diff passwd.old passwd.new
```

diff 还以可以比较两目录下的文件名

```bash
diff /etc/rc0.d/ /etc/rc5.d/
```

##### cmp

对比两文件，以字节的地位对比，默认仅会输出第一个发现的不同点。

```bash
# 用法
cmp [-l] file1 file2
#选项与参数：
#-l ：将所有的不同点的字节处都列出来。因为 cmp 默认仅会输出第一个发现的不同点。
```

##### patch

该命令可以用来做打补丁的用途，也可以将文件回退到补丁前的状态。
centOS7 默认没有安装 patch 文件，我们需要自己安装。

```bash
su - #切换到root账户
mount /dev/sr0 /mnt #将安装光盘挂载到/mnt
rpm -ivh /mnt/Packages/patch-2.*
umount /mnt
exit
```

用法

```bash
#更新
patch -pN < patch_file
#还原
patch -R -pN < patch_file
```

制作补丁文件。以上面的 passwd.old 和 passwd.new 为例。

```bash
#制作补丁文件
diff -Naur passwd.old passwd.new > passwd.patch
#将刚刚制作出来的 patch file 用来更新旧版数据
patch -p0 < passwd.patch
#恢复旧文件的内容
patch -R -p0 < passwd.patch
```

#### 文件打印：pr

```bash
#打印/etc/man_db.conf
pr /etc/man_db.conf
```
