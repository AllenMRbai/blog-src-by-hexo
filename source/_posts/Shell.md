---
title: Shell
date: 2019-02-28 15:10:45
tags: Linux
---

### 什么是 shell

为了避免非专业人士直接操作系统导致系统受破坏，因此需要有个 shell 来操作。每个命令都是一个应用，shell 是一个能调用各个应用接口的程序。

##### shell 的命令

可以用 type 命令来查看那些命令是 shell 内置

```bash
type cd #cd是shell内置的
```

如果命令过长，可以用‘\’将回车转义(escape)，这样就可以在下一行继续输入命令。

有时候需要删除和编辑写好的命令,可以通过下面的组合键操作。

| 组合键 | 功能                   |
| ------ | ---------------------- |
| ^u     | 删除光标处到头部的内容 |
| ^k     | 删除光标处到尾部的内容 |
| ^a     | 到命令头部             |
| ^e     | 到命令尾部             |

### shell 的变量功能

获得变量

```bash
#shell中获取变量需要在变量前加$
#打印HOME变量
echo $HOME
```

设置变量

```bash
#用法
variable=value
#设置变量myname的值为Allen
myname=Allen
```

删除变量

```bash
unset variable
```

##### 变量的设置规则

- 变量名由字母、数字、下划线组成，并且不能由数字开头
- 变量赋值等号左右两侧不能由空格，如“name = allenbai”或“name=allen bai”
- 如果变量的值内要包含空格需要用‘"’或‘'’来包裹,或‘\’转义。如“name="hello world"”,“name=hello\ world”。注意如果使用的是单引号，那么单引号的内容就会变成一般的字符，里面定义的变量就会失去意义。如“name='${name} is you'”,这里的“${name}”会变成字符，而不是变量。
- 如果在赋值的过程中引用变量需要“"$变量名"”或“${变量名}”。如“name="$name"love”或“name=${name}love”
- 如果需要其他指令提供的信息，可以用反单引号“\`指令\`”或“$(指令)”。如“name=$(uname -r)”
- “export variable”可以将变量导到环境变量中。可以用“env”查看所有环境变量
- 系统的变量默认全大写，自己的变量可以用小写，以便区分。这点要不要遵守看个人偏好。
- “unset varaible”可以取消变量

##### 范例

```bash
#进入到目前的核心目录模块
cd /lib/modules/`uname -r`/kernal
cd /lib/modules/$(uname -r)/kernal
#列出crontab相关文件的权限
ls -ld `locate crontab`
ls -ld $(locate crontab)
```

##### 环境变量

查看当前环境变量

```bash
#查看当前环境变量
env
```

export 不加参数也可以查看当前环境的变量

```bash
export
```

如果需要查看包括自定义变量和环境变量在内的所有变量可以用 set

```bash
#不带任何参数即可
#貌似我安装的CentOS 7内无法显示变量
set
```

一些需要了解的系统环境变量

- HOME 代表使用者的主文件夹
- SHELL 当前环境用的是哪个 shell
- HISTSIZE 能被系统记录的历史指令的数量
- MAIL 当前用户的邮件信箱文件
- PATH 可执行文件搜索的路径
- LANG 语系数据
- RANDOM 随机数变量。值介于 0~32767 之间。如果要取得 0~9 之间可以这样：

```bash
declare -i number=$RANDOM*10/32768;echo $number
```

bash 不只有环境变量，还有一些 bash 操作接口有关的变量，以及自定义的变量。用 set 可以观察所有变量。

除了环境变量外，其他 bash 内的变量

- PS1 提示字符的设置
- \$ shell 的 PID
- ? 关于上个指令的回传值，0 表示成功，如果上个指令执行报错，会回传错误代码
- OSTYPE,HOSTTYPE,MACHTYPE 主机硬件

环境变量与自订变量的差别是该变量是否会被子程序继续引用。

##### 语系变量

```bash
locale
```

##### 变量的有效范围

鸟哥书里提到的环境变量和自订变量也可以称为全局变量(global variable)和局部变量(local variable)。

##### read,array,declare

read 可以读取来自键盘输入的变量

```bash
#用法
read [-pt] variable
#-p :后面可以接提示字符
#-t :后面可以接等待的“秒数”

#请用户输入用户名
read -p '请在30秒内输入用户名' -t 30 user_name
```

declare/typeset 设置变量类型

```bash
#用法
declare [-aixr] variable
#-a :将变量声明为数组array
#-i :将变量声明为整数integer
#-x :将变量声明为环境变量
#-r :将变量声明为readonly类型，改变量不可更改内容，也不可以unset

#以下两种变量声明结果不同
sum=100+300+50;echo $sum #结果是 100+300+50
declare -i sum=100+300+50;echo $sum #结果是 450

#将-改成+ 可以进行取消的操作
declare +x sum
```

定义数组

```bash
person[1]='allen'
person[2]='jack'
person[3]='jane'
echo "${person[1]},${person[2]},${person[3]}"
```

##### 变量的删除、取代和替换

从前面开始删除变量中的某些内容

```bash
#用法 变量名+‘#’+匹配规则(pattern)
#这里的‘#’表示从最前面开始向右删，且删最短的那个
${variable#pattern}
#如果要删除最长的要写‘##’,有点像正则表达式内的贪婪匹配
${variable##pattern}
```

```bash
echo $PATH
#/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/allen/.local/bin:/home/allen/bin
#删除/usr/local/bin:这个串
${PATH#/*local/bin:}
#删除前面所有目录，保留最后一个。注意这里是‘##’,如果是‘#’的话，它只删掉第一个
${PATH##/*:}
```

从后面开始删除变量中的某些内容

```bash
#从后面开始向左删，且删最短的那个。用法与‘#’类似
${variable%pattern}
#同样的，如果要删尽可能多的内容，是‘%%’
${variable%%pattern}
```

替换内容

```bash
#用法 pattern是指匹配规则 new是新内容
${variable/pattern/new}
#全部替换
${variable//pattern/new}
```

```bash
#将匹配到的第一个sbin替换为大写的SBIN
${PATH/sbin/SBIN}
#如果要全部替换，第一个 ‘/’要写成‘//’
${PATH//sbin/SBIN}
```

##### 变量的测试与内容替换

‘-’

```bash
# 如果old_var未定义 则new_var的值为content
new_var=${old_var-content}
```

有点像 js 里的管道符号

```javascript
var new_var = old_var || content;
```

```bash
# 如果old_var未定义或为空值 则new_var的值为content
#注意这里多了个‘:’
new_var=${old_var:-content}
```

‘+’

```bash
# 如果old_var有定义(包括空值) 则new_var的值为content
new_var=${old_var+content}
```

有点像 js 里的‘&&’

```javascript
var new_var = old_var && content;
```

```bash
# 如果old_var有定义且值不为空 则new_var的值为content
#注意这里多了个‘:’
new_var=${old_var:+content}
```

‘=’

```bash
# 这个效果和‘-’一样，但多了一个效果，就是content赋值给new_var的同时也会赋值给old_var
new_var=${old_var=content}
```

```bash
# 这个效果和‘:-’一样，但多了一个效果，就是content赋值给new_var的同时也会赋值给old_var
#注意这里多了个‘:’
new_var=${old_var:=content}
```

‘?’

```bash
# 如果old_var未定义 就会告知控制台warn的内容
new_var=${old_var?warn}
```

```bash
# 如果old_var未定义或空值 就会告知控制台warn的内容
#注意这里多了个‘:’
new_var=${old_var:?warn}
```

其实不难发现‘-’,‘=’,‘?’都是假设 old_var 未定义才生效的，如果需要 old_var 为空值也生效加个‘:’就好。

而‘+’是假设 old_var 已定义才生效(无论有没有值)，如果需要 old_var 为空值不生效加个‘:’就好。

### 命令别名与历史命名

##### 命令别名

```bash
# 查看所有命令别名
alias
# 生成命令别名
alias comand_name='comand'
#例子 命令别名是lm,指令是‘ls -al | more’
alias lm='ls -al | more'
# 删除命令别名
unalias comand_name
#例子 删除命令别名 lm
unalias lm
```

### bash 的环境配置文件

##### 重要的环境配置文件

| 配置文件                                       | 功能                                                                                                  |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| /etc/issue                                     | bash 的开机画面                                                                                       |
| /etc/motd                                      | bash 登录后的提示信息                                                                                 |
| /etc/locale.conf                               | 决定默认使用语系，由/etc/profile.d/lang.sh 调用进来                                                   |
| /etc/profile                                   | 登录 bash 时读取的文件，是系统整体的设置                                                              |
| /etc/profile.d/\*.sh                           | /etc/profile 运行时回去执行/etc/profile.d 下所有.sh 结尾的文件                                        |
| /usr/share/bash-completion/completions/\*      | Tab 的补全设置，由/etc/profile.d/bash_completion.sh 调用进来                                          |
| ~/.bash_profile 或 ~/.bash_login 或 ~/.profile | 属于使用者个人的设置。其实只会读取其中的一个。最先读取~/.bash_profile，如果没有再读取下一个，以此类推 |
| ~/.bashrc                                      | 获取 no-login shell 时会读取该文件。还有~/.bash_profile 会读该文件                                    |
| ~/.bash_history                                | 历史命令记录的地方                                                                                    |
| ~/.bash_logout                                 | 登出的时候执行的文件                                                                                  |

##### 读入环境配置文件的指令

```bash
#用法
source 配置文件文件名
#例子 当修改了~/.bash_profile时需要登出再登录后才生效，为了让设置生效可以用source指令
source ~/.bash_profile
#soucrce指令也可以写成‘.’
. ~/.bash_profile
```

### 万用字符和特殊符号

##### 万用字符

万用字符有点像正则匹配有点类似但有差别

| 符号 | 意义                                              |
| ---- | ------------------------------------------------- |
| \*   | 代表 0 到无穷个任意字符，相当于正则的.\*          |
| ?    | 随机的一个字符                                    |
| []   | 表示括号内的任意一个字符                          |
| [-]  | 字符编码范围，例如[0-9]表示 0 到 9 之间的所有字符 |
| [^]  | 反向选择，例如[^abc]表示非 a,b,c 的字符           |

##### 特殊符号

| 符号     | 意义                                               |
| -------- | -------------------------------------------------- |
| \#       | 注释符号                                           |
| \\       | 转义                                               |
| \|       | 管线(pipe),分隔两个管线命令的界定                  |
| ;        | 连续指令下达分隔符号                               |
| ~        | 使用者的主文件夹                                   |
| \$       | 取用变量前置字符：亦即是变量之前需要加的变量取代值 |
| &        | 工作控制 （job control）：将指令变成背景下工作     |
| !        | 逻辑运算符,not 的意思                              |
| /        | 目录符号：路径分隔的符号                           |
| \>和\>\> | 数据流重导向：输出导向，分别是“取代”与“累加”       |
| \<和\<\< | 数据流重导向：输入导向                             |
| ' '      | 单引号，不具有变量置换的功能 （\$ 变为纯文本）     |
| " "      | 具有变量置换的功能！ （\$ 可保留相关功能）         |
| \`       | 两个“\`”中间为可以先执行的指令，亦可使用 \$（ ）   |
| （）     | 在中间为子 shell 的起始与结束                      |
| { }      | 在中间为命令区块的组合！                           |

##### 数据流重导向

数据流重导向就是将某个指令执行后应该要出现在屏幕上的数据， 给他传输到
其他的地方，例如文件或者是设备。

| 符号         | 意义                                   |
| ------------ | -------------------------------------- |
| \<           | 标准输入，将某个文件代替键盘输入       |
| \<\<         | 标准输入，以某个符号作为文件的结束标记 |
| \>或 1\>     | 标准输出,覆盖原内容                    |
| \>\>或 1\>\> | 标准输出，添加的原内容尾部             |
| 2\>          | 标准错误输出,覆盖原内容                |
| 2\>\>        | 标准错误输出，添加的原内容的尾部       |

以下是例子

```bash
#例 将ll列出的内容保存到~/rootfile内
ll / > ~/rootfile

#例 创建catfile文件并将~/.bashrc的内容给它
cat > catfile < ~/.bashrc

#原来cat不仅仅只有读文件的功能，不加参数还会读取键盘录入
#例 用cat命令新建catfile，并以键盘的录入内容作为该文件的内容。想要结束输入按^d。
cat > catfile

#例 用cat命令新建catfile，并以‘eof’做为结束标记。
cat > catfile << 'eof' #当你输入eof回车时，就和按^d的效果是一样的

#例 使用find指令时，遇到没有读取权限的文件会报错，通过2\>将错误导到/dev/null
find /home -name .bashrc 2> /dev/null #/dev/null相当于垃圾桶黑洞
#例 将正确和错误的数据都写入同一个文件
find /home -name .bashrc > list 2> &1
#或者这么写
find /home -name .bashrc &> list
```

##### ;和&&和||

‘;’这个很好理解，相当于每个语句的结束符。有了它，就可以同时写个条指令了。

```bash
#例 关机前执行两次sync同步写入磁盘
sync;sync;shutdown -h now
```

&&前后连接两个指令，前一个指令报错就不执行后一个指令；||前后连接两个指令，前一个不报错就不执行后一个指令。

```bash
#非root用户执行find /root，$?返回错误代码，所这里的ls不执行
find /root && ls
#这里的ls会执行
find /root || ls
```

```bash
#我们要在tmp下面创建/abc/hehe，但我们不知道abc是否存在。于是我们可以用下面的方法。
ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/hehe
```

###### 解析：

情况一，tmp 下没 abc 文件夹。

由于没有 abc 文件夹，那第一个指令回传值（\$?）不为 0，也就是说第一个指令报错了，于是会执行第二指令新建 abc 目录。第二个指令执行成功，于是会执行第三个指令创建 hehe 文件。

情况二，tmp 下有 abc 文件夹。
第一个指令成功，所以第二个指令不执行。注意了，这行语句并没有就此结束。第一个回传值会继续传递，由于第一个回传值是 0，所以第三个指令会执行。

结论：无论有没有 abc 目录，我们都能创建/tmp/abc/hehe 文件

##### 管线符号(pipe)

‘|’就是管线符号了。如果数据流重导向是将指令的结果导到设备和文件或设备和文件的数据导到指令的话；那么 pipe 就是将前一个命令的数据输出做为后一个命令的输入。
使用管线符号需要注意两点：

1. 管线符号后面的那个命令必须是管线命令。能接受前一个指令的数据成为 standard input 才是管线命令。
2. 管线命令仅会处理 standard output，对于 standard error output 会被忽略。

##### 管线符号的组合运用

- cut

cut 可以将数据按我们的要求切出来，可以想象成 split。该命令是按行来操作的。

```bash
#用法一；按分隔字符将一行数据切成多段，并攫取其中的几个片段
cut -d '分隔字符' -f 分段区间
#例 将$PATH按‘:’分隔，并取第一段和第4段
echo ${PATH} | cut -d ':' -f 1,4
#例 将$PATH按‘:’分隔，并取第4段到最后一段
echo ${PATH} | cut -d ':' -f 4-
#例 用last查看登录者信息，只显示登陆者名称
last | cut -d ' ' -f 1
```

```bash
#用法二；
cut -c 字符区间
#例 将export的信息前面 declare -x的字去掉
export | cut -c 12-

```

- grep

查询每一行，把含有我们想要的文字的句子列出来。grep 能配合正则使用。

```bash
grep [-acinv] [--color=auto] '搜寻字符' filename
#选项与参数：
#-a ：将 binary 文件以 text 文件的方式搜寻数据
#-c ：计算找到 '搜寻字串' 的次数
#-i ：忽略大小写的不同，所以大小写视为相同
#-n ：顺便输出行号
#-v ：反向选择，亦即显示出没有 '搜寻字串' 内容的那一行！
#--color=auto ：可以将找到的关键字部分加上颜色的显示喔！

#例 将last中出现root的那行取出
last | grep 'root'
#例 取出 /etc/man_db.conf 内含 MANPATH 的那几行
grep --color=auto 'MANPATH' /etc/man_db.conf
```

- sort

sort 很明显是用来排序的。排序的字符和语系有关，因此， 如果您需要排
序时，建议使用 LANG=C 来让语系统一，数据排序比较好一些。

```bash
#用法
sort [-fbMnrtuk] [file or stdin]
#选项与参数：
#-f ：忽略大小写的差异，例如 A 与 a 视为编码相同；
#-b ：忽略最前面的空白字符部分；
#-M ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
#-n ：使用“纯数字”进行排序（默认是以文字体态来排序的）；
#-r ：反向排序；
#-u ：就是 uniq ，相同的数据中，仅出现一行代表；
#-t ：分隔符号，默认是用 [tab] 键来分隔；
#-k ：以那个区间 （field） 来进行排序的意思

#例 将/etc/passwd下的账号进行排序
cat /etc/passwd | sort

#例 /etc/passwd内容是以‘:’分隔的，我们想用第三栏来排序
cat /etc/passwd | sort -t ':' -k 3

#取出last中的所有账号，并排序
last | cut -d ' ' -f1 | sort
```

- uniq

将排序完的数据去重。（只有连续的重复内容才会被去除，所以使用前需要排序）

```bash
#用法
uniq [-ic]
#选项与参数：
#-i ：忽略大小写字符的不同；
#-c ：进行计数

#例 将last取出的账号排序并除重
last | cut -d ' ' -f1 | sort | uniq
#例 如果要在上个例子的基础上加上计数
last | cut -d ' ' -f1 | sort | uniq -c
```

- wc

查看文件多少行，多少个字符

```bash
#用法
wc [-lwm]
#选项与参数：
#-l ：仅列出行；
#-w ：仅列出多少字（英文单字）；
#-m ：多少字符；

#查看/etc/man_db.conf里面有多少岁字、行、字符
cat /etc/man_db.conf | wc
```

- tee

我们可以使用数据流重导向‘>’将 stdout 输入到文件。如果我们在输入到文件的同时，还要继续使用这个 stdout 的话，我们可使用 tee（双向重导向）。

```bash
#用法
tee [-a] file
#选项与参数：
#-a ：以累加 （append） 的方式，将数据加入 file 当中！

#例 将last输出一份到last.list
last | tee last.list | cut -d " " -f1
```

- unix2dos dos2unix

unix2dos 是将文本的 unix 换行符号换成 dos 的。

- tr

删除或替换文本

```bash
#用法
tr [-ds] SET1 ...
#选项与参数：
#-d ：删除讯息当中的 SET1 这个字串；
#-s ：取代掉重复的字符！

#例 将last输出的信息中所有的小写变成大写
last | tr [a-z] [A-Z]
#例 将/etc/passwd输出的小写中的冒号‘:’删除
cat /etc/passwd | tr -d ':'
#例 将/etc/passd转存成dos断行到/root/passwd中，再将^M符号删除
cp /etc/passwd ~/passwd && unix2dos ~/passwd
#例 上面那个方法可以用tr的方法来代替。^M可以用\r来代替
cat ~/passwd | tr -d '\r' > ~/passwd.linux
```

- col

将 tab 转成对等的空白键

```bash
#用法
col [-xb]

#cat -A可以看到tab的符号为^I
cat -A /etc/man_db.conf
#将/etc/man_db.conf的tab转为空白键后再cat -A查看
cat /etc/man_db.conf | col -x | cat -A | more
```

- join

将含有相同数据的行拼接在一起。使用时最好排序下。

```bash
#用法
join [-ti12]
#选项与参数：
#-t ：join 默认以空白字符分隔数据，并且比对“第一个字段”的数据，
#如果两个文件相同，则将两笔数据联成一行，且第一个字段放在第一个！
#-i ：忽略大小写的差异；
#-1 ：这个是数字的 1 ，代表“第一个文件要用那个字段来分析”的意思；
#-2 ：代表“第二个文件要用那个字段来分析”的意思。

#例 用root 的身份，将 /etc/passwd 与 /etc/shadow 相关数据整合成一栏
join -t ':' /etc/passwd /etc/shadow | head -n 3
#例 ：我们知道 /etc/passwd 第四个字段是 GID ，那个 GID 记录在/etc/group 当中的第三个字段，请问如何将两个文件整合？
join -t ':' -1 4 /etc/passwd -2 3 /etc/group | head -n 3
```

- paste

paste 比 join 简单粗暴，它直接将两个文件的行相连，并用 tab 隔开

```bash
#用法
paste [-d] file1 file2
#选项与参数：
#-d ：后面可以接分隔字符。默认是以 [tab] 来分隔的！
#- ：如果 file 部分写成 - ，表示来自 standard input 的数据的意思。
```

- expand

将 tab 转成空白键

```bash
#用法
expand [-t] file
#选项与参数：
#-t ：后面可以接数字。一般来说，一个 tab 按键可以用 8 个空白键取代。
#我们也可以自行定义一个 [tab] 按键代表多少个字符呢！#
```

- split

他可以帮你将一个大文件，依据文件大小或行数来分区，就可以将大文件分区成为小文件了！

```bash
#用法
split [-bl] file PREFIX
#选项与参数：
#-b ：后面可接欲分区成的文件大小，可加单位，例如 b, k, m 等；
#-l ：以行数来进行分区。
#PREFIX ：代表前置字符的意思，可作为分区文件的前导文字。

#例 我的 /etc/services 有六百多K，若想要分成 300K 一个文件时？
cd /tmp; split -b 300k /etc/services services
#例 将上面分割出来的三个文件合成一个文件，名为servicesback
cat services* >> servicesback

#例 使用ls -al / 输出的信息中，没十行记录成一个文件
ls -al | split -l 10 - lsroot
```

- xargs

xargs 可以读入 stdin 的数据，并且以空白字符或断行字符作为分辨，将 stdin 的数据分隔成为 arguments

```bash
xargs [-0epn] command
#选项与参数：
#-0 ：如果输入的 stdin 含有特殊字符，例如 `, \, 空白键等等字符时，这个 -0 参数可以将他还原成一般字符。这个参数可以用于特殊状态喔！
#-e ：这个是 EOF （end of file） 的意思。后面可以接一个字串，当 xargs分析到这个字串时，就会停止继续工作！
#-p ：在执行每个指令的 argument 时，都会询问使用者的意思；
#-n ：后面接次数，每次 command 指令执行时，要使用几个参数的意思。当 xargs后面没有接任何的指令时，默认是以 echo 来进行输出喔！

#例 取出/etc/passwd中的前三个用户，并依次对每个用户执行id命令
cut -d ':' -f 1 /etc/passwd | head -n 3 | xargs -n 1 id
```

- \-

在管线命令的使用中‘-’是非常重要的，它指代前一个命令的标准输出

```bash
mkdir /tmp/homeback
tar -cvf - /home | tar -xvf - -C /tmp/homeback
```
