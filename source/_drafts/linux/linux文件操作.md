# 文件操作

## 正则表达式

1. 文件的查找命令 find
   - find path -name 通配符方式
   - find path -regex regex
   - find path -type f 只匹配普通文件
2. 文本内容的过滤（查找）grep
   - grep regex target
[阮一峰的这篇文章解释了通配符的问题](http://www.ruanyifeng.com/blog/2018/09/bash-wildcards.html)

## 行编辑器介绍

### sed 的基本用法演示

- sed 一般用于对文本内容做替换
- sed 的模式空间 基本工作方式
  1. 将文件以行为单位读取到内存（模式空间）
  2. 使用 sed 的每个脚本对该行进行操作
  3. 处理完成后术输出该行

```shell
# sed 替换内容 其中的分隔符是可以更改对
sed 's/a/bb/' file 等价与 sed 's!a!bb!' file

# sed 多次替换
sed -e 's/a/bb/' -e 's/bb/aa/' filea fileb
等价与
sed 's/a/bb/;s/bb/aa/' filea fileb

# sed 替换更改原文件
sed -i 's/a/bb/;s/bb/aa/' file

# sed 重定向
sed '/s/a/bb;s/bb/aa/' filea >> fileb

# sed 扩展元字符  使用分组演示
sed -r 's/(aa)|(bb)/!/' file

# sed 分组的回调
echo axyzb >> file
sed -r 's/(a.*b)/\1:\1/' file
> axyzb:axyzb
```

### sed 的替换命令加强版

#### 全局替换

- s/old/new/g
  - g 为全局替换， 用于替换所有出现的次数
  - / 如果和正则匹配的内容冲突可以使用其他符号，如：
    - s@old@new@g
  
#### 标志位

- s/old/new/标志位
  - 数字，第几次出现的才进行替换
  - g，每次出现都进行替换
  - p 打印模式空间的内容
    - sed -n‘scirpt’ filename 阻止默认输出
  - w file 将模式空间的内容写入到文件

#### 寻址

默认对每行进行操作，增加寻址后对匹配的行进行操作

- /正则表达式/s/old/new/g
- 行号s/old/new/g
  - 行号可以是具体的行，也可以是最后一行$符号
- 可以使用两个寻址符号，也可以混合使用行号和正则地址  

#### 分组

- 寻址可以匹配多条命令
- /regex/{ s/old/new/ ; s/old/new/ }

#### sed 脚本文件

- 可以将选项保存为文件，使用-f加载脚本文件
- sed -f sedscript filename

#### sed 的其他命令

- 删除命令
  - [ 寻址 ]d
- 追加、插入、更改
  - 追加命令 a 在匹配的下一行追加
  - 插入命令 i 在匹配的上一行插入
  - 更改命令 c
- 打印
  - 打印命令 p
- 下一行
  - 打印行号命令 =
  - 下一行命令 n
- 读文件和写文件
  - 读文件命令 r
  - 写文件命令 w
- 退出命令
  - 退出命令 q

#### sed 的多行模式

- 多行模式处理命令N、P、D
  - N 将下一行加入到模式空间
  - P 打印模式空间中的第一个字符到第一个换行符
  - D 删除模式空间中的第一个字符到第一个换行符

#### sed 的保持空间

- 保持空间也是多行的一种操作方式
- 将内容暂存在保持空间，便于做多行处理
- 保持空间命令
  - h 和 H 将模式空间内容存放到保持空间
  - g 和 G 将保持空间内容取出到模式空间
  - x 交换模式空间和保持空间内容

### AWK 的基本用法演示

- AWk 更像是脚本语言
- AWK 一般用于对文本内容进行统计、按需要的格式输出
  - cut 命令 cut -d
- AWK 脚本的控制流程
  - 输入数据前 BEGIN{}
  - 主输入循环 {}
  - 所有文件读取完成例程 END{}

#### 系统变量

- FS 和 OFS 字段分隔符，OFS 表示输出的字符分隔符
- RS 记录分隔符
- NR 和 FNR 行数
- NF 字段数量，最后一个字段内容可以用 $NF 取出

```shell
head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print $1}'
head -5 /etc/passwd | awk 'BEGIN{FS=":";OFS="-"}{print $1,$2}'
head -5 /etc/passwd | awk 'BEGIN{RS=":"}{print $0}'
head -5 /etc/passwd | awk '{print NR,$0}'
head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print NF }'
```

awk -f xxx.awk file

