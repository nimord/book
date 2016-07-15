[TOC]
### Chapter2 命令之乐
#### cat
`cat file1 file2 file3`：将 file1 file2 file3 内容拼接
`echo 'text through stdin' | cat - `：`-`被作为stdin的文件名
`cat -s file`：压缩相邻的空白行
`cat -T file`：显示制表符
`cat -n file`：输出行号

#### script & scriptreplay
`script -t 2>timing.log -a output.session` 开始录制
`scriptreplay timing.log output.session`

#### xargs
xargs命令把从stdin中的数据流重新格式化，再将其作为参数提供给其他命令。
- 将多行输入转换成单行输出
只需要将换行符移除，再用" "（空格）进行代替，就可以实现多行输入的转换。'\n'被解释成一个换行符，换行符其实就是多行文本之间的定界符。利用xargs，我们可以用空格替换掉换行符，这样就能够将多行文本转换成单行文本：
```
$ cat example.txt # 样例文件
1 2 3 4 5 6
7 8 9 10
11 12
$ cat example.txt | xargs
1 2 3 4 5 6 7 8 9 10 11 12
```
- 将单行输入转换成多行输出
指定每行最大的参数数量n，我们可以将任何来自stdin的文本划分成多行，每行n个参数。每一个参数都是由" "（空格）隔开的字符串。空格是默认的定界符。下面的方法可以将单行划分成多行：
```
$ cat example.txt | xargs -n 3
1 2 3
4 5 6
7 8 9
10 11 12
```
xargs有一个选项`-I`，`-I {}`指定了替换字符串。对于每一个命令参数，字符串{}都会被从stdin读取到的参数替换掉。
##### 结合find使用xargs
xargs和find两者结合使用可以让任务变得更轻松。不过人们通常却是以一种错误的组合方式使用它们。例如：
`$ find . -type f -name "*.txt"  -print | xargs rm -f`
这样做有时可能达不到预期的效果。我们没法预测分隔find命令输出结果的定界符究竟是什么（'\n'或者' '）。很多文件名中都可能会包含空格符（' '），因此xargs很可能会误认为它们是定界符（例如，hell text.txt会被xargs误解为hell和text.txt）。
只要我们把find的输出作为xargs的输入，就必须将-print0与find结合使用，以字符null（'\0'）来分隔输出。
用find匹配并列出所有的 .txt文件，然后用xargs将这些文件删除：
`$ find . -type f -name "*.txt" -print0 | xargs -0 rm -f`
这样就可以删除所有的.txt文件。`xargs -0`将`\0`作为输入定界符。

#### tr
`tr`可以对来自标准输入的内容进行字符替换、字符删除以及重复字符压缩。它可以将一组字符变成另一组字符，因而通常也被称为转换（translate）命令。
`tr [options] set1 set2`
将来自stdin的输入字符从set1映射到set2，然后将输出写入stdout（标准输出）。set1和set2是字符类或字符集。如果两个字符集的长度不相等，那么set2会不断重复其最后一个字符，直到长度与set1相同。如果set2的长度大于set1，那么在set2中超出set1长度的那部分字符则全部被忽略。
- 用tr删除字符
tr有一个选项-d，可以通过指定需要被删除的字符集合，将出现在stdin中的特定字符清除掉：
```
$ echo "Hello 123 world 456" | tr -d '0-9'
Hello world
# 将stdin中的数字删除并打印出来
```
- 字符集补集
我们可以利用选项`-c`来使用`set1`的补集。下面的命令中，`set2`是可选的：
`tr -c [set1] [set2]`
`set1`的补集意味着这个集合中包含`set1`中没有的所有字符。
```
$ echo hello 1 char 2 next 4 | tr -d -c '0-9 \n'
 1  2  4
```
- 用tr压缩字符
`tr`的`-s`选项可以压缩输入中重复的字符，方法如下：
```
$ echo "GNU is       not     UNIX. Recursive   right ?" | tr -s ' '
GNU is not UNIX. Recursive right ?
# tr -s '[set]'
```
- 字符类
alnum：字母和数字。
alpha：字母。
cntrl：控制（非打印）字符。
digit：数字。
graph：图形字符。
lower：小写字母。
print：可打印字符。
punct：标点符号。
space：空白字符。
upper：大写字母。
xdigit：十六进制字符。
`tr [:class:] [:class:]`
例如：
`tr '[:lower:]' '[:upper:]'`

#### sort
1. 对一组文件（例如file1.txt和file2.txt）进行排序
`$ sort file1.txt file2.txt > sorted.txt`
或者
`$ sort file1.txt file2.txt -o sorted.txt`
2. 按照数字顺序进行排序
`$ sort -n file.txt`
3. 按照逆序进行排序
`$ sort -r file.txt`
4. 按照月份进行排序（依照一月，二月，三月……）
`$ sort -M months.txt`
5. 合并两个已排序过的文件
`$ sort -m sorted1 sorted2`
6. 找出已排序文件中不重复的行
`$ sort file1.txt file2.txt | uniq`
7. 检查文件是否已经排序过
```
#!/bin/bash
#功能描述：排序
sort -C filename ;
if [ $? -eq 0 ]; then
   echo Sorted;
else
   echo Unsorted;
fi
```

#### mktemp
`mktemp`命令的用法非常简单。它生成一个临时文件并返回其文件名（如果创建的是目录，则返回目录名）
1. 创建临时文件：
```
$ filename=`mktemp`
$ echo $filename
/tmp/tmp.8xvhkjF5fH
```
上面的代码创建了一个临时文件，并打印出存储在`$filename`中的文件名。

2. 创建临时目录：
```
$ dirname=`mktemp -d`
$ echo $dirname
tmp.NI8xzW7VRX
```
上面的代码创建了一个临时目录，并打印出存储在`$dirname`中的目录名。
如果仅仅是想生成文件名，又不希望创建实际的文件或目录，方法如下：
```
$ tmpfile=`mktemp -u`
$ echo $tmpfile
/tmp/tmp.RsGmilRpcT
```
文件名被存储在`$tmpfile`中，但并没有创建对应的文件。
根据模板创建临时文件名：
```
$mktemp test.XXX
test.2tc
```
如果提供了定制模板，`X`会被随机的字符（字母或数字）替换。注意，`mktemp`正常工作的前提是保证模板中只少要有3个`X`。

#### split & csplit
在某些情况下，必须把文件分割成多个更小的片段。
假设有一个叫做data.file的测试文件，其大小为100KB。你可以将该文件分割成多个大小为10KB的文件，方法如下：
```
$ split -b 10k data.file
$ ls
data.file  xaa  xab  xac  xad  xae  xaf  xag  xah  xai  xaj
```
上面的命令将data.file分割成多个文件，每一个文件大小为10KB。这些文件以xab、xac、xad的形式命名。这表明它们都有一个字母后缀。如果想以数字为后缀，可以另外使用`-d`参数。此外，使用`-a length`可以指定后缀长度：
```
$ split -b 10k data.file -d -a 4
```
除了k（KB）后缀，我们还可以使用M（MB）、G（GB）、c（byte）、w（word）等后缀。
```
$ ls
data.file  x0009  x0019  x0029  x0039  x0049  x0059  x0069  x0079
```

之前那些分割后的文件都有一个文件名前缀x。我们也可以通过提供一个前缀名来使用自己的文件名前缀。split命令最后一个参数是PREFIX，其格式如下：
```
$ split [COMMAND_ARGS] PREFIX
```
我们加上文件名前缀再运行先前那个命令来分割文件：
```
$ split -b 10k data.file -d -a 4 split_file
$ ls
data.file       split_file0002  split_file0005  split_file0008  strtok.c
split_file0000  split_file0003  split_file0006  split_file0009
split_file0001  split_file0004  split_file0007
```
如果不想按照数据块大小，而是需要根据行数来分割文件的话，可以使用`-l no_of_lines`：
```
$ split -l 10 data.file
#分割成多个文件，每个文件包含10行
```
另一个有趣的的工具是`csplit`。它能够依据指定的条件和字符串匹配选项对日志文件进行分割。来看看这个工具是如何运作的。
`csplit`是`split`工具的一个变体。`split`只能够根据数据大小或行数分割文件，而`csplit`可以根据文本自身的特点进行分割。是否存在某个单词或文本内容都可作为分割文件的条件。
看一个日志文件示例：
```
$ cat server.log
SERVER-1
[connection] 192.168.0.1 success
[connection] 192.168.0.2 failed
[disconnect] 192.168.0.3 pending
[connection] 192.168.0.4 success
SERVER-2
[connection] 192.168.0.1 failed
[connection] 192.168.0.2 failed
[disconnect] 192.168.0.3 success
[connection] 192.168.0.4 failed
SERVER-3
[connection] 192.168.0.1 pending
[connection] 192.168.0.2 pending
[disconnect] 192.168.0.3 pending
[connection] 192.168.0.4 failed
```
我们需要将这个日志文件分割成server1.log、server2.log和server3.log，这些文件的内容分别取自原文件中不同的SERVER部分。那么，可以使用下面的方法来实现：
```
$ csplit server.log /SERVER/ -n 2 -s {*}  -f server -b "%02d.log"  ; rm server00.log
$ ls
server01.log  server02.log  server03.log  server.log
```
有关这个命令的详细说明如下。
`/SERVER/`用来匹配某一行，分割过程即从此处开始。
`/[REGEX]/`表示文本样式。包括从当前行（第一行）直到（但不包括）包含“SERVER”的匹配行。
`{*}`表示根据匹配重复执行分割，直到文件末尾为止。可以用{整数}的形式来指定分割执行的次数。
`-s`使命令进入静默模式，不打印其他信息。
`-n`指定分割后的文件名后缀的数字个数，例如01、02、03等。
`-f`指定分割后的文件名前缀（在上面的例子中，server就是前缀）。
`-b`指定后缀格式。例如%02d.log，类似于C语言中printf的参数格式。在这里文件名=前缀+后缀=server + %02d.log。
因为分割后的第一个文件没有任何内容（匹配的单词就位于文件的第一行中），所以我们删除了server00.log。
#### # %
借助`%` `#`操作符可以轻松将相应部分从“名称.扩展名”这种格式中提取出来。
`${VAR%.*}`的含义如下所述：
从`$VAR`中删除位于`%`右侧的通配符（在前例中是`.*`）所匹配的字符串。通配符从右向左进行匹配。
给`VAR`赋值，`VAR=sample.jpg`那么通配符从右向左就会匹配到`.jpg`，因此，从`$VAR`中删除匹配结果，就会得到输出`sample`。
`%`属于***非贪婪***（non-greedy）操作。它从右到左找出匹配通配符的最短结果。还有另一个操作符`%%`，这个操作符与`%`相似，但行为模式却是***贪婪*** 的，这意味着它会匹配符合条件的最长的字符串。例如，我们现在有这样一个文件：
```
VAR=hack.fun.book.txt
```
使用`%`操作符：
```
$ echo ${VAR%.*}
```
得到输出：hack.fun.book
操作符`%`使用`.*`从右向左执行非贪婪匹配（.txt）。
使用`%%`操作符：
```
$ echo ${VAR%%.*}
```
得到输出：hack
操作符`%%`则用`.*`从右向左执行贪婪匹配（.fun.book.txt）。
在第二个任务中，我们用`#`操作符从文件名中提取扩展名。这个操作符与`%`类似，不过求值方向是从左向右。
`${VAR#*.}`的含义如下所述：
从`$VAR`中删除位于`#`右侧的通配符（即在前例中使用的`*.`）所匹配的字符串。通配符从左向右进行匹配。
和`%%`类似，`#`也有一个相对应的贪婪操作符`##`。
`##`从左向右进行贪婪匹配，并从指定变量中删除匹配结果。
来看一个例子：
```
VAR=hack.fun.book.txt
```
使用`#`操作符：
`$ echo ${VAR#*.}`
得到输出：fun.book.txt
操作符`#`用`*.`从左向右执行非贪婪匹配（hack）。
使用`##`操作符：
`$ echo ${VAR##*.}`
得到输出：txt
操作符`##`则用`*.`从左向右执行贪婪匹配（txt）。

#### expect
```
#!/bin/bash
#文件名: interactive.sh
read -p "Enter number:" no ;
read -p "Enter name:" name
echo You have entered $no, $name;
```
用expect实现自动化
```
#!/usr/bin/expect
#文件名: automate_expect.sh
spawn ./interactive.sh
expect "Enter number:"
send "1\n"
expect "Enter name:"
send "hello\n"
expect eof
```
在这个脚本中：
- `spawn`参数指定需要自动化哪一个命令；
- `expect`参数提供需要等待的消息；
- `send`是要发送的消息；
- `expect eof`指明命令交互结束

#### wait
并行进程加速命令执行
```
#/bin/bash
#文件名: generate_checksums.sh
PIDARRAY=()
for file in File1.iso File2.iso
do
  md5sum $file &
  PIDARRAY+=("$!")
done
wait ${PIDARRAY[@]}
```
我们利用了Bash的操作符`&`，它使得shell将命令置于后台并继续执行脚本。这意味着一旦循环结束，脚本就会退出，而`md5sum`命令仍在后台运行。为了避免这种情况，我们使用`$!`来获得进程的`PID`，在Bash中，`$!`保存着最近一个后台进程的`PID`。我们将这些PID放入数组，然后使用`wait`命令等待这些进程结束。