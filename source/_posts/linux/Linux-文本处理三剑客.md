---
title: Linux 文本处理三剑客
date: 2019-09-23 21:14:39
tags: [Linux, grep, awk, sed]
categories: Linux
---

grep、awk、sed 是 linux 中进行文本处理的三大利器，称为 linux 三剑客。grep 适合单纯的文本查找和匹配，awk 适合用于格式化文本，sed 适合编辑匹配的文本。

<!--more-->

## grep
grep 是 linux 的文本搜索工具，根据用户指定的 “模式”（可以是正则表达式）对目标文件逐步进行匹配检查，并打印匹配到的行。

### grep 格式与参数
grep [options] pattern [filepath]

-c ：统计匹配到的行数； 
-E ：使用扩展的正则表达式
-f ：<规则文件> 查找符合指定规则文件的内容，格式为每行一个规则样式；
-i ：忽略大小写；
-n ：输出行号；
-o ：只显示被匹配到的字符串；
-v ：反向匹配，也就是输出不匹配的内容，相当于 [^]； 

若希望输出匹配行的前后行 -A -B -C 参数： 
- grep 字符串 filepath -A 1 ： 输出除匹配的该行外，还显示其后面一行(After 1)
- grep 字符串 filepath -B 1 ： 输出除匹配的该行外，还显示其前面一行(Before 1)
- grep 字符串 filepath -C 1 ： 输出除匹配的该行外，还显示前一行和后一行

### grep 实例
1.统计某个文本中 abc 出现的次数
```sh
grep -o 'abc' filename | wc -l
# 不能用 grep -c 这只显示出符合要求的行数，一行多个的话无法判断
```

## awk
awk 是一种编程语言，用于 linux 处理数据和生成报告，更适合文本格式化，对文本进行较复杂的格式处理。 awk 将文件逐行输入，以空格为默认分隔符将每行切片，然后可以对切开的每部分进行分析和处理。

### awk 格式与参数
`awk [options] 'pattern{action}' filepath` 
pattern 表示 awk 在数据中查找的内容，action 是在找到匹配内容时执行的命令

-F fs ：指定分隔符 fs，fs 可以是字符串或正则表达式，如果有多个分隔符： `awk -F '[:,]' filepath`
-f scriptfile ：从脚本文件中读取 awk 命令
-v var=value ：赋值一个用户定义变量，将外部变量传递给 awk

### awk 变量
#### awk 内建变量
- $0 整行, $1-$n 第n个字段，awk 逐行处理文本，处理的最小单位是字段，每个字段的命名方式为：$n，n 为字段号，从 1 开始，$0 表示一整行。 直接 print 或 print $0 就显示整行。
- NF ：浏览记录的域的个数（字段数）； $NF 是最后一列的内容，$(NF-1) 是倒数第二列
- FS ：输入字段分隔符，默认为空白字符
- OFS ：输出字段分隔符，默认为空白字符
- RS ：输入记录分隔符，指定输入时的换行符，原换行符扔有效，比如 RS=':'，则遇到冒号就换行输出
- ORS ：输出记录分隔符，输出时用指定符号代替换行符
- NR ：已经读出的记录数，即行号，从 1 开始，有多个文件的话值也是不断累加的，用于最后可以输出总共的记录数。
- FNR ：各文件分别计数, 行号，后跟一个文件和 NR 一样，跟多个文件，第二个文件行号从 1 开始
- FILENAME ：当前文件名
- ARGC ：命令行参数的个数
- ARGV ：数组，保存的是命令行所给定的各参数，查看参数

使用时要加 -v

**实例**
1.查看最近 5 条登录用户和 ip 地址，第一列是用户名，第三列是 IP 
```sh
$ last -n 5 | awk '{print $1"\t"$3}'
```

2.过滤文本，查看 /etc/passwd 文件
```sh
$ awk -F ":" '{print NR, $1, $5, $6}' OFS="\t" /etc/passwd # OFS 指定输出格式
```

3.统计 ip.txt 文件中出现次数最多的 ip， 文本第二个字段是 ip
```sh
$ cat ip.txt | awk 'NR!=1 {if($2) print $2}' | sort | uniq -c | sort -nr | head -n 1
# uniq 命令用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用, -c 在每列旁边显示该行重复出现的次数。
# sort -n 是按照数值排序，否则会按字典序； sort -r 是倒序，由大到小排序
# NR!=1 去掉第一行，也就是 title；if($2) 保证非空 
```

4.统计文件中每个单词出现的次数，一行有多个单词，每个单词按照空格分割
```sh
$ awk -v RS=' ' '{if($0) print}'  test1 | grep -v "^$" | sort | uniq -c
# RS=' ' 按照空格切分，并遇到空格就换行；grep -v "^$" 去掉空白行，这个空白行是由于空格分割导致的
```
如果每个单词占一行的话：
`cat word.txt | sort | uniq -c`

#### awk 自定义变量
使用 `-v var=value` 结构可以自定义变量，可以在 print 前后定义变量
```sh
$ awk -v name="myname" '{print name, $NF}' test.txt
$ awk '{name="myname"; print name, $NF}' test.txt # 直接在编程体中定义
```

### awk pattern 匹配
1. 未指定，则匹配每一行
2. /regular expression/ ：仅处理能够模式匹配到的行，正则表达式需要用 `/ /` 括起来
3. relational expression：关系表达式，结果为“真”才会被处理，即非 0 非空

4. line ranges：行范围
   startline(起始行), endline(结束行)：/pat1/,/pat2/  不支持直接给出数字，可以有多段，中间可以有间隔

5. BEGIN/END 模式
` awk 'BEGIN{BEGIN 操作} {文件行处理块} END{END 操作}' filepath `

- BEGIN 模块是在文件输入前执行的，不输入任何文件数据也会执行该模块，常用于设置修改内置变量如 OFS,RS 等，为用户自定义的变量赋初始值或者打印标题信息等。操作语句以 ";" 或分行隔开。BEGIN 可缺省。  
- END 模块是处理完文件后的操作

**实例**
1.正则匹配
~ 表示模式开始，/pattern/ 中间是模式，用 !~ // 表示模式取反
```sh
$ awk '/80/{print $4}' netstat.txt # 匹配字段中有 80 的
$ awk '$4 ~ /^c/{print $4}' netstat.txt # 第 4 列是以字母 c 开头的
$ awk '$4 !~ /^c/{print $4}' netstat.txt # 第 4 列不是以字母 c 开头的
$ awk '$6 ~ /FIN|WAIT/ || NR==1 {print $4,$6}' netstat.txt # 第 6 列包含 FIN 或 WAIT 的，并显示 titie，对第一行不做处理
```

2.使用关系表达式
`awk '$0{print}' file` 去掉空行 

3.统计每个用户的进程占了多少内存（计算 RSS 列）
```sh
$ ps aux | awk 'NR!=1{a[$1]+=$6;} END {for (i in a) print i "," a[i]"KB";}'
```

### awk 高阶用法
#### 条件语句 if-else
if(condition){statement;...} else{statement}
if(condition1){statement1} else if(condition2){statement2} else{statement3}

1.使用 awk 拆分文件，可以使用重定向，如按照第 6 列拆分文件 NR!=1 表示不处理表头，文件名即为第 6 列的名字
```sh
$ awk '{if(NR!=1 && $6) print > $6}' netstat.txt
```

2.复杂一点，可以使用 if-else if 语句，将符合某个条件的行划分到一个文本中
```sh
$ awk 'NR!=1 && $6 {if($6 ~ /FIN|WAIT/) print > 1.txt; 
      else if($6 ~ /LISTEN/) print > 2.txt; 
      else print > 3.txt}' netstat.txt
```

#### 循环语句
##### while 循环
语法 ：while(condition){statement;...}

使用场景 ：用于对一行内多个字段进行逐一处理，或对数组中的各元素逐一处理

1.以 : 分割，显示每一行长度大于 3 的单词及其长度
```sh
$ awk -F : '{i=1; while(i<NF){if(length($i)>=3){print $i, length($i)}; i++}}' test
```

2.计算 1+2+3+...+100
```sh
$ awk 'BEGIN{i=1; sum=0; while(i<=100){sum+=i; i++}; print sum}'
```

##### for 循环
语法 ：for(expr1; expr2; expr3){statement;...}
遍历数组 ：for(var in array){statement;...} 

显示每行每个单词的长度，按照空格分隔
```sh
$ awk '{for(i=1;i<=NF;i++){print $i, length($i)}}' test
```

## sed
sed 是一种流编辑器（stream editor），一次处理一行内容，是一个文本编辑工具，实现数据的替换，删除，增加，选取等。

原理：
1. sed 将当前处理的行存储在临时缓冲区中，即模式空间(patternspace)
2. 使用 sed 命令处理缓冲区中的内容
3. 处理完成后，将缓冲区内容输出到屏幕，然后读下一行进行处理。

### sed 格式和参数
`sed [options] '[地址定界]动作' filepath`

**options：**
- `-e <script>` ：直接在命令列模式上进行 sed 的动作编辑，可以用多个 -e 'actions' -e 'actions'
- `-f <script文件>` ：将 sed 动作写在文件内，以指定的 script 来处理输入的文本
- -i ：直接修改读取的文件内容，不输出到终端，也可选择将修改重定向到一个新的文件
- -n ：使用安静模式，一般 sed 会将所有的输入都列出在终端，使用 -n 就只有被匹配处理过的行会显示出来
- -r ：使用扩展的正则表达式

**地址定界：**
- 无地址 ：全文处理
- 单地址 ： 
    - `#` ：指定的行
    - `/pattern/` ：被此处模式所能够匹配到的每一行
- 地址范围：`#,#`   `/pat1/,/pat2/`     `#,/pat1/`
- 步长 ~：
    - `sed -n '1~2p'` ：只打印奇数行 （1~2 从第 1 行，一次加 2 行）
    - `sed -n '2~2p'` ：只打印偶数行

**动作说明：**
- a ：新增，a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
- i ：插入，在当前行之前插入文本。多行时除最后一行外，每行末尾需用"\"续行
- d ：删除行
- p ：打印，将某个选择的数据打印，通常和 sed -n 搭配使用
- s ：替换，搭配正则表达式，如 's/old/new/g'

### sed 实例
1.在每行最前面或最后面添加 #
```sh
$ sed 's/^/#/g' filepath
$ sed 's/$/#/g' filepath
```

2.指定需要替换的内容，允许多个匹配，用分号隔开
```sh
$ sed '3s/old/new/g' filepath # 替换第 3 行的
$ sed '3,6s/old/new/g' filepath # 替换第 3-6 行的
$ sed 's/old/new/1' filepath # 替换每行的第一个
$ sed 's/old/new/3g; xxx' filepath # 替换每行的第三个以后的
$ sed '1~2s/old/new/g' filepath # 替换奇数行的 old
```

3.使用 & 表示被匹配的变量，在周围添加内容
```sh
$ sed 's/my/[&]/g' filepath # 将所有的 my 变为 [my]
```

4.删除
```sh
$ sed '2d' filepath # 删除第 2 行
```

5.打印
```sh
$ sed -n '2!p' filepath # 打印除了第 2 行的内容
$ sed -n '1,3p' filepath # 打印第 1-3 行
$ sed -n '/aaa/p' filepath # 打印包含 aaa 的行
```


## 参考
[Linux文本三剑客超详细教程---grep、sed、awk](https://www.cnblogs.com/along21/p/10366886.html#auto_id_15)
[AWK 简明教程](https://coolshell.cn/articles/9070.html)
[SED 简明教程](https://coolshell.cn/articles/9104.html)
