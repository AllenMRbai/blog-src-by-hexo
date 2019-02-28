---
title: Linux中的正则
date: 2019-02-28 15:32:10
tags: Linux
---

linux 中的正则有特殊符号，其他的内容和 javascript 中都相同。注意语系会对正则产生影响，一般使用与 POSIX 相容的语系。
特殊符号 | 意义
---|---
[:alnum:] | 代表英文大小写字符及数字，亦即 0-9, A-Z, a-z
[:alpha:] | 代表任何英文大小写字符，亦即 A-Z, a-z
[:blank:] | 代表空白键与 [Tab] 按键两者
[:cntrl:] | 代表键盘上面的控制按键，亦即包括 CR, LF, Tab, Del.. 等等
[:digit:] | 代表数字而已，亦即 0-9
[:graph:] | 除了空白字符 （空白键与 [Tab] 按键） 外的其他所有按键
[:lower:] | 代表小写字符，亦即 a-z
[:print:] | 代表任何可以被打印出来的字符
[:punct:] | 代表标点符号 （punctuation symbol），亦即：" ' ? ! ; : # \$...
[:upper:] | 代表大写字符，亦即 A-Z
[:space:] | 任何会产生空白的字符，包括空白键, [Tab], CR 等等
[:xdigit:] | 代表 16 进位的数字类型，因此包括： 0-9, A-F, a-f 的数字与字符

### 带正则功能的命令

并不是所有命令都能使用正则，只有支持正则的命令和程序才可以使用。

##### 1.grep 的一些进阶选项

- 用法

```bash
grep [-A] [-B] [--color=auto] '搜寻字串' filename
#选项与参数：
#-A ：后面可加数字，为 after 的意思，除了列出该行外，后续的 n 行也列出来；
#-B ：后面可加数字，为 befer 的意思，除了列出该行外，前面的 n 行也列出来；
#--color=auto 可将正确的那个撷取数据列出颜色

#用 dmesg 列出核心讯息，再以 grep 找出内含 qxl 那行(鸟哥用的是QXL虚拟显卡，大家可能会搜不到)
dmesg | grep 'qxl'
#承上题，要将捉到的关键字显色，且加上行号来表示
dmsg | grep -n --color=auto 'qxl'
#承上题，在关键字所在行的前两行与后三行也一起捉出来显示
dmesg | grep -n -A3 -B2 --color=auto 'qxl'
```

- 例子

鸟哥提供了一个练习用的文件：[regular_express.txt](http://linux.vbird.org/linux_basic/0330regularex/regular_express.txt)

内容如下

```txt
"Open Source" is a good mechanism to develop programs.
apple is my favorite food.
Football game is not use feet only.
this dress doesn't fit me.
However, this dress is about $ 3183 dollars.^M
GNU is free air not free beer.^M
Her hair is very beauty.^M
I can't finish the test.^M
Oh! The soup taste good.^M
motorcycle is cheap than car.
This window is clear.
the symbol '*' is represented as start.
Oh! My god!
The gd software is a library for drafting programs.^M
You are the best is mean you are the no. 1.
The world &lt;Happy&gt; is the same with "glad".
I like dog.
google is the best tools for search keyword.
goooooogle yes!
go! go! Let's go.
# I am VBird
```

```bash
#查找‘the’并带上行号
grep -n 'the' regular_express.txt
#查找没有‘the’的行，并带上行号，这里的v相当于相反
grep -vn 'the' regular_express.txt
#忽略大小写
grep -in 'the' regular_express.txt
#搜索含tast或test的行
grep -n 't[ae]st' regular_express.txt
#搜索带有‘oo’并且前一个字符不为小写字母
grep -n '[^[:lower:]]oo' regular_express.txt
#搜索啥都没有的一行
grep -n '^$' regular_express.txt
#搜索非空行和开头不为‘#’的行
grep -v '^$' /etc/rsyslog.conf &#124; grep -v '^#'
```

##### 2.sed 工具

sed 是一个管线命令，可以将数据进行取代、删除、新增、攫取。

- 用法

```bash
sed [-nefr] [动作]
#选项与参数：
#-n ：使用安静（silent）模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到屏幕上。但如果加上 -n 参数后，则只有经过 sed 特殊处理的那一行（或者动作）才会被列出来。
#-e ：直接在命令行界面上进行 sed 的动作编辑；
#-f ：直接将 sed 的动作写在一个文件内， -f filename 则可以执行 filename 内的 sed 动作；
#-r ：sed 的动作支持的是延伸型正则表达式的语法。（默认是基础正则表达式语法）
#-i ：直接修改读取的文件内容，而不是由屏幕输出。

#动作说明： [n1[,n2]]function
#n1, n2 ：不见得会存在，一般代表“选择进行动作的行数”，举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则“ 10,20[动作行为] ”

#function 有下面这些咚咚：
#a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现（目前的下一行）～
#c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
#d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
#i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现（目前的上一行）；
#p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
#s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正则表达式！例如 1,20s/old/new/g 就是啦！
```

- 例

```bash
#删除2-5行的内容
nl /etc/passwd | sed '2,5d'
#在第二行后添加‘drink tea?’
nl /etc/passwd &#124; sed '2a drink tea'
#将2-5行的内容取代为‘No 2-5 number’
nl /etc/passwd | sed '2,5c No 2-5 number'
#仅列出 /etc/passwd 文件内的第 5-7 行
nl /etc/passwd | sed -n '5,7p'
```

sed 不仅可以进行行的处理，还可以进行部分数据的搜索并取代的功能。

```bash
#使用方法
sed 's/要被取代的字串/新的字串/g'
#
/sbin/ifconfig eth0 | grep 'inet ' | sed 's/^.*inet //g'
```
