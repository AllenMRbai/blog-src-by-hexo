---
title: 日志查询常用命令
date: 2025-06-10 21:14:27
tags:
---

### cat 命令查看日志

查看整个日志文件

```shell
# 显示日志里匹配字串那行以及前后10行
cat filename | grep -C 10 '关键字'
# 显示匹配字串及前10行
cat filename | grep -B 10 '关键字'
# 显示匹配字串及后10行
cat filename | grep -A 10 '关键字'
# -n 可以显示行号
cat -n filename
```

### tail 命令查看日志

查看日志尾部的日志

```shell
# 实时刷新最新日志
tail -f xxx.log
# 实时刷新最新的10行日志
tail -10f xxx.log
# 查找最新的10行中与关键字匹配的行
tail -10f xxx.log | grep [关键字]
# 查找最新的10行中时间范围在2021-02-04 11:40-2021-02-04 11:49范围中的行
tail -10f xxx.log | grep '2021-02-04 11:4[0-9]'
# 查看最新的10行中与关键字匹配的行加上匹配行后的5行
tail -10f xxx.log | grep -A 5 [关键字]
# 查询日志尾部最后10行的日志
tail  -n  10   test.log   
# 查询10行之后的所有日志
tail  -n +10   test.log   
# 循环实时查看最后1000行记录 ****************** 最常用的
tail -fn 10 test.log | grep [关键字]
```

### vi 命令查看日志

```
1、进入vim编辑模式:vim filename

2、输入“/关键字”,按enter键查找

3、查找下一个,按“n”即可，查找上一个,按“shift+n”即可

退出:按ESC键后,接着再输入:号时,vi会在屏幕的最下方等待我们输入命令

wq! 保存退出;

q! 不保存退出;
```

```
/关键字   注:正向查找,按n键把光标移动到下一个符合条件的地方
?关键字   注:反向查找,按shift+n 键,把光标移动到下一个符合条件的
```

### less 命令查看日志

```text 
less log.log

shift + G 命令到文件尾部  然后输入 ？加上你要搜索的关键字例如 ？1213

按 n 向上查找关键字

shift+n  反向查找关键字
```

```text
less与more类似，使用less可以随意浏览文件，而more仅能向前移动，不能向后移动，而且 less 在查看之前不会加载整个文件。
less log2013.log 查看文件
ps -ef | less   ps查看进程信息并通过less分页显示
history | less   查看命令历史使用记录并通过less分页显示
less log2013.log log2014.log   浏览多个文件
```