---
layout: post
title: "Linux 打印文本部分行内容（前几行，指定行，中间几行，跨行，奇偶行，后几行，最后一行，匹配行）"
date: "2020-07-10 01:00"
category: Linux
tags: Linux sed grep awk tail head
author: jiangliheng
---
* content
{:toc}




# 背景

打印对账文件最后一行汇总信息，通过钉钉定时发送到运维群。顺便总结下 Linux 打印文本部分行内容的各种方法。

# 测试文本

```bash
# 生成测试文本内容
$ seq -f "%02g daodaotest" 1 10 > test.txt

# 查看测试文本内容，并显示行号
$ cat -n test.txt
     1	01 daodaotest
     2	02 daodaotest
     3	03 daodaotest
     4	04 daodaotest
     5	05 daodaotest
     6	06 daodaotest
     7	07 daodaotest
     8	08 daodaotest
     9	09 daodaotest
    10	10 daodaotest

$ awk '{print NR" "$0}' test.txt
1 01 daodaotest
2 02 daodaotest
3 03 daodaotest
4 04 daodaotest
5 05 daodaotest
6 06 daodaotest
7 07 daodaotest
8 08 daodaotest
9 09 daodaotest
10 10 daodaotest
```

# 打印前 N 行内容

```bash
# head 打印前 5 行内容
$ head -5 test.txt
$ head -n 5 test.txt

# sed 打印前 5 行内容
$ sed -n '1,5p' test.txt

# awk 打印前 5 行内容
$ awk 'NR<6' test.txt
```

# 打印指定行内容

```bash
# sed 打印第 5 行内容
$ sed -n '5p' test.txt

# awk 打印第 5 行内容
$ awk 'NR==5' test.txt

# tail 配合 head，打印指定行内容
$ tail -n +5 test.txt | head -1
```

# 打印指定范围行内容

```bash
# sed 打印 5~10 行内容
$ sed -n '5,10p' test.txt

# awk 打印 5~10 行内容
$ awk 'NR>4 && NR<11' test.txt

# tail 配合 head，打印 5~10 行内容
$ tail -n +5 test.txt | head -6
```

# 打印跨行内容

```bash
# sed 打印第 3 行 和 5~7 行内容
$ sed -n '3p;5,7p' test.txt

# awk 打印第 3 行 和 5~7 行内容
$  awk 'NR==3 || (NR>4 && NR<8)' test.txt
```

# 打印奇偶行内容

```bash
# 打印奇数行内容
## NR 表示行号
$ awk 'NR%2!=0' test.txt
$ awk 'NR%2' test.txt

## i 为变量，未定义变量初始值为 0，对于字符运算，未定义变量初值为空字符串
## 读取第 1 行记录，进行模式匹配：i=!0（!表示取反）。! 右边是个布尔值，0 为假，非 0 为真，!0 就是真，因此 i=1，条件为真打印第一条记录。
## 读取第 2 行记录，进行模式匹配：i=!1（因为上次 i 的值由 0 变成了 1），条件为假不打印。
## 读取第 3 行记录，因为上次条件为假，i 恢复初值为 0，继续打印。以此类推...
## 上述运算并没有真正的判断记录，而是布尔值真假判断。
$ awk 'i=!i' test.txt

## m~np：m 表示起始行；~2 表示：步长
$ sed -n '1~2p' test.txt

## 先打印第 1 行，执行 n 命令读取当前行的下一行，放到模式空间，后面再没有打印模式空间行操作，所以只保存不打印，同等方式继续打印第 3 行。
$ sed -n '1,$p;n' test.txt
$ sed -n 'p;n' test.txt

# 打印偶数行内容
$ awk 'NR%2==0' test.txt
$ awk '!(NR%2)' test.txt
$ awk '!(i=!i)' test.txt
$ sed -n 'n;p' test.txt
$ sed -n '1~1p' test.txt
$ sed -n '1,$n;p' test.txt
```

# 打印最后 N 行内容

```bash
# tail 打印后 5 行内容
$ tail -5 test.txt
$ tail -n 5 test.txt
```

# 打印最后一行内容

```bash
# tail 打印最后一行内容
$ tail -n 1 test.txt

# sed 打印最后一行内容
$ sed -n '$p' test.txt

# awk 打印最后一行内容
$ awk 'END {print}' test.txt
```

# 打印匹配行内容

```bash
# 打印以 "1" 开头的行内容
$ sed -n '/^1/p' test.txt
$ grep "^1" test.txt

# 打印不以 "1" 开头的行内容
$ sed -n '/1/!p' test.txt
$ grep -v "^1" test.txt

# 从匹配 "03" 行到第 5 行内容
$ sed -n '/03/,5p' test.txt

# 打印匹配 "03" 行 到匹配 "05" 行内容
$ sed -n '/03/,/05/p' test.txt
```

> 微信公众号：daodaotest
