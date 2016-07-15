[TOC]
### 前言
《Linux Shell 脚本攻略》这本书我已经看过一遍，属于比较基础的非常适合初学者学习使用。由于目前自己也写不出什么高深的东西，就拿这本书中的内容练练手。另外，这次笔记我打算使用*Markdown*语法来写，权当自己熟悉一下*Markdown*语言了。废话不多说了～V～ Let's do it！

>1. 系统环境：Linux（Neokylin Linux Desktop 6.0）
>2. 编辑器：Haroopad
>3. 文件管理：github

### Chapter1 小试牛刀
shell 环境使得用户能够与操作系统的核心功能进行交互，我们常说的脚本更多涉及的便是这种环境，由于目前常用的桌面操作系统使用的Bash（Bourne Again Shell），所以后续的例子皆是针对于此环境而言。
#### 终端&Bash
终端是交互式工具，用户可以通过他与shell环境进行交互，对于Linux桌面操作系统命令是通过shell终端进入并执行的，打开终端后一般会出现一个提示符。其形式通常如下：
>`[user@localhost ~]$ `：普通用户
>`[root@localhost ~]#`：管理员用户root

shell脚本通常是一个以shebang（#！）起始的文件`#!/bin/sh`，但有时候文件中省去该行也可以正常执行，前提是该文件必须有可执行权限。

在Bash中每个命令或者命令序列是通过使用分号或则换行符来分隔的。比如
`$ cmd1 ； cmd2`
等同于
`$cmd1`
`$cmd2`
注释部分以**\#**为起始，一直延续到行尾。

echo是用于终端打印的基本命令。在默认情况下echo在++*每次调用后会添加一个换行符*++。
echo命令在不使用引号，使用单引号及使用双引号的副作用：
- 使用不带引号的echo时，没法在所要显示的文本中使用分号(；)，因为分号在Bash中被作为命令定界符
- 变量替换在单引号中会失效
- 使用双引号时如遇特殊字符需要转义

在默认情况下echo会将一个换行符追加到输出文本的尾部，可以使用`-n`参数来忽略结尾的换行符。
另外也可以使用echo在终端中生成彩色输出，如`echo -e "\e[42;31m The text has green backaground and red foreground\e[0m "` **\e[*m**为设置颜色的格式。
```
          0     to restore default color
          1     for brighter colors
          4     for underlined text
          5     for flashing text
         30     for black foreground
         31     for red foreground
         32     for green foreground
         33     for yellow (or brown) foreground
         34     for blue foreground
         35     for purple foreground
         36     for cyan foreground
         37     for white (or gray) foreground
         40     for black background
         41     for red background
         42     for green background
         43     for yellow (or brown) background
         44     for blue background
         45     for purple background
         46     for cyan background
         47     for white (or gray) background
```

#### 变量&环境变量
变量可以通过以下方式进行赋值
`var=value`
注意，var = value不同于var=value。前者相等操作，后者是赋值。在变量前加入`$`前缀就可以打印出变量的内容。
常用的环境变量： PATH HOME PWD USER UID SHELL等
**补充：**
1.获取字符串长度 `length=${#var}`, length 为字符串包含的字符数；
2.识别当前所使用的shell `echo $SHELL` 或者 `echo $0`
3.root用户的UID为0
4.Bash的提示字符串一般定义为PS1， 如执行`echo $PS1` 后显示 `[\u@\h \W]\$`,其中`\u`可以扩展为用户名，`\h`可扩展为主机名，`\W`可扩展为当前工作目录
5.${parameter:+expression},如果parameter有值且不为空则使用expression的值。
#### 算术操作
一般使用let、(())和 [ ], 高级一些的则使用expr及bc。
```
#!/bin/sh
no1=3
no2=4
let var1=no1+no2
echo $var1
var2=$[ no1 + no2 ]
echo $var2
var3=$((no1 + no2))
echo $var3
var4=$(expr $no1 + $no2)
echo $var4
```
bc的使用
`echo "4*0.8" | bc`
`echo "$no *0.5" | bc`
`echo "scale=3;3/8" | bc` 设置小数精度为3
`echo "ibase=10;obase=2;$no" | bc` ibase为输入进制数，obase为输出进制数
`echo "sqrt($no)" | bc` 计算平方根

#### 文字描述符&重定向
>- 0------ stdin
>- 1------ stdout
>- 2------ stderr

创建一个文件描述符并读取文件
```
$ echo this is a test line > input.txt
$ exec 3<input.txt
$ cat<&3
this is a test line

$ exec 4>output.txt
$ exec newline>&4
$ cat output.txt
newline
```
#### 数组
`array_var=(1 2 3 4 5 6)` 这些值会存储在以0为索引起点的连续位置上，注意要以“ ”进行分割不要使用“，”。
定义关联数组
`declare -A array`
$ array=([index1]=val1 [index2]=val2)
如果想列出数组的索引`echo ${!array[*]}` 或者 `echo ${!array[@]}`

#### 别名
alias new_command='command sequence'

#### 获取终端信息
tput & stty
```
tput cols  #行数
tput lines #列数
tput longname #打印当前终端名
tput cup 100 100 #将光标移到（100， 100）
tputsetb n #n取值0-7 设置终端背景
tputsetf n #n取值0-7 设置终端前景
tput bold #设置文本样式为粗体
tput smul #设置下划线的起
tput rmul #设置下划线的止
tput ed #删除从当前光标位置到行尾的所有内容

stty -echo #禁止将输出发送到终端
stty echo #输出发送到终端
```
例子：
```
#！/bin/sh
# file: sleep.sh
echo -n Count:
tput sc #存储光标位置

count=0;
while true；
do
	if [ $count -lt 40 ]；
    then
    	let count++；
        sleep 1;
        tput rc #恢复光标位置
        tput ed #清除光标位至行尾内容
        echo -n $Count;
    else
    	exit 0
    fi
done
```
#### 调试脚本
> set -x：在执行时显示参数和命令
> set +x：禁止调试
> set -v：当前命令读取时显示输入
> set +v：禁止打印输入

#### 函数&参数
function fname()
{
	statements;
}
或
fname()
{
	statements;
}
调用时执行 fname arg1 arg2
> $1 参数1
> $2 参数2
> $n 参数n
> $@ 扩展为“$1” "$2" "$3"
> $* 扩展为“$1 $2 $3”
> $? 命令返回值
> $# 参数个数

使用（）可以生成一个子shell，不会影响主shell中的环境。

#### 循环比较
略








