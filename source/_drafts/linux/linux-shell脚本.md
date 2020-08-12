# Shell

## 认识Shell

### 什么是shell

- Shell是命令解释器
- Shell有很多
  - cat /etc/shells
- CentOS 7 默认使用的Shell是bash
  
### Linux的启动过程

- BIOS -> MBR -> BootLoader(grub) -> kernel -> systemd -> 系统初始化 -> shell

### 怎么编写一个shell脚本

- UNIX的哲学： 一条命令只做一件事
- #! /bin/bash 申明解释器
- chmod u+rx file

### shell脚本的执行方式

- 产生子进程然后执行 对当前环境不会影响
  - bash ./filename.sh
  - ./filename.sh
- 当前进程执行 会影响当前环境
  - source ./filename.sh
  - . filename.sh

### 内建命令和外部命令的区别

- 内建命令不需要创建子进程
- 内建命令对当前 Shell 生效

## 管道与重定向
  
- 管道与管道符
- 子进程与子shell
- 重定向符号
  - `>`    `>>`    `2>`    `&>`

## 变量

- 变量的定义
- 变量的赋值
  - `a=1`;
  - `let a=1+2`;
  - `l=ls`;
  - `l=$(ls -al /etc)`;
  - `s="hello word"`
- 变量的引用
  - echo $var
  - echo ${var}
- 变量的作用范围
  - export a 这样子进程就可以获取父进程的值。
- 系统环境变量
  - 环境变量：每个Shell打开都能获得到的变量
    - set 和 env 命令
    - 预定义变量
      - `$?`判断上一条命令是否执行成功 `$$`显示进程的PID `$0`当前进程的名称
    - $PATH 搜索路径
    - $PS1 当前提示终端
  - 位置变量
    - $1 $2 ... $9 ${10} $n
    - ${2-defult} if $2 not None $2 else defult;
- 环境变量配置文件
  - 配置文件 通用配置所有用户
    - /etc/profile
    - /etc/profile.d
    - /etc/bashrc
  - 用户特有配置 家目录
    - ~/.bash_profile
    - ~/.bashrc

## 数组

```shell
$ array=(one two three)
$ echo ${array[@]}
one two three
$ echo ${array[2]}
three
$ echo ${#array[@]}
3
$ array[3]=four
$ echo ${array[*]}  
one two three four
```

## 算数运算符

- 使用expr进行运算
  - expr 4+5 expr只支持整数
  - a=\`expr\` 4+5
- 数字常量的使用方法
  - let “变量名=变量值” 很少使用
  - 变量值使用0开头为八进制
  - 变量值使用0x开头为十六进制
- 双圆括号是let命令的简化
  - (( a=10 ))
  - (( a++ ))
  - echo $((10+20))

## 特殊符号大全

[字符大全](https://www.cnblogs.com/balaamwe/archive/2012/03/15/2397998.html)

### 引号

- ‘完全引用
- “不完全引用
- `执行命令

### 括号

- () (()) $() 圆括号
  - 单独使用圆括号会产生一个子shell ( xyz=123 )
  - 数组初始化 IPS=( ip1 ip2 ip3 )
- [] [[]] 方括号
  - 单独使用方括号是测试(test)或数组元素功能
  - 两个方括号表示测试表达式
- <> 尖括号 重定向符号
- {} 花括号
  - 输出范围 echo {0..9}
  - 文件复制 cp /etc/passwd{,.bak}

### 运算和逻辑符号

### 转义符号

- 转义某字符
  - \n普通字符转义后用不同的功能
  - \\`特殊字符转义之后，当作普通字符来使用
  
### 其他符号

- #注释符
- ;命令分隔符
  - case语句的分隔符要转义;;
- :空指令
- . 和source命令相同
- ~家目录
- ,分隔目录
- *通配符
- ?条件测试或通配符
- $取值符号
- |管道符
- &后台运行
- _空格

## if-then-else 语句

- 基本用法
  - if [ 测试条件成立 ] 或 命令返回值是否为0
  - then 执行相应命令
  - elif
  - then
  - else 测试条件不成立，执行相应命令
  - fi 结束

## case使用

```shell
case "$1" in
      "start|START")
      echo $0 start....
      ;;

      "stop")
      echo $0 stop.....
      ;;

      "restart|reload")
      echo $0 restart.....
      ;;

      *)
      echo "Usage: $0 {start|stop|restart|reload}"
      ;;
esac
```

## 循环

- for循环的语法
  - for 参数 in 列表 或者 for ((i = 1; i < 10; i++))
  - do 执行的命令
  - done 封闭一个循环
- 使用反引号$()方式执行命令，命令的结果当作列表进行处理
  
```shell
for filename in `ls *.mp3`
do
  mv $filename $(basename $filename .mp3).mp4
done
```

- while循环
  - while test测试是否成立
  - do
  - done
  
```shell
a=1
while [ $a -lt 10 ]
do
  ((a++))
  echo $a
done
```

```shell
a=1
until [ $a -gt 10 ]
do
  ((a++))
  echo $a
done
```

### 命令行参数

- 命令行参数可以使用 $1 $2 ... ${10} ... $n 进行读取
- $0 代表脚本名称
- $* 和 $@ 代表所有的位置参数
- $# 代表位置参数的数量

```shell
for pos in $*
do
  if [ "$pos" = "help" ]; then
    echo $pos $pos
  fi
done
```

```shell
# shift使参数左移
while [ $# -ge 1]
do
  echo $#
  echo "do something"
  if [ "$1" = "help" ]; then
    echo $1 $1
  fi
  shift
done
```

## 自定义函数

- 自定义函数

```shell
function fname() {
  命令
}
```

- 函数的执行
  - fname
- 函数作用范围的变量
  - local 变量名
- 函数的参数
  - $1 $2 $3 ... $4;

### 系统脚本

每个进程都在/proc/下有一个和进程号同名的文件
- 系统自建了函数库，可以在脚本中引用
  - /etc/init.d/functions
- 自建函数库
  - 使用 source 函数脚本文件 “导入” 函数

- ulimit -a查看使用限制
- fork炸弹 .(){.|.&}; .

## 捕获信号

- 捕获信号脚本的编写
  - kill 默认会发送15号信号给应用程序
  - ctrl + c 发送2号信号给应用程序
  - 9号信号不可阻塞

```shell
trap "echo sig 15" 15
echo $$
```

## 计划任务

### 一次性计划任务 

at 在指定时间运行

### 周期性计划任务

- cron
  - 配置方式
    - crontab -e
  - 查看现有的计划任务
    - crontab -l
  - 配置格式
    - 分钟 小时 日期 月份 星期 执行的命令
    - 注意命令的路径问题
  
```shell
0 0 * * 1-7 /usr/bin/date >> /tmp/date.txt
```

### 计划任务加锁 flock

- 如果计算机不能按照预期时间运行
  - anacrontab 延时计划任务
  - flock 锁文件