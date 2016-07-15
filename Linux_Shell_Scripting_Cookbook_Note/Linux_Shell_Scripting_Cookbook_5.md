[TOC]
### Chapter5 实战
#### Web 页面下载
##### wget
wget是一个用于文件下载的命令行工具，选项繁多且用法灵活。
用wget可以下载网页或远程文件：
```
$ wget URL1 URL2 ...
```
通常下载的文件名和URL中的文件名保持一致，下载日志或者进度被写入stdout中。可以通过`-O`指定输出文件名，通过`-o`指定一个日志文件。
对于不稳定的互联网连接，下载有可能被迫中断，我们可以使用`-t`选项指定重试的连接次数：`wget -t 5 URL`，或者要求wget不停地进行重试：`wget -t 0 URL`
下载限速：`wget --limit-rate 20k URL`
最大下载配额：`wget -Q 100m URL`
断点续传：`wget -c URL`
复制网站：`wget --mirror --convert-links exampledomain.com` 或者`wget -r -N -l -k DEPTH URL`(todo)
需要认证的页面：`wget --user username --password pass URL`

##### lynx
用lynx命令可以获取纯文本形式的网页
```
lynx URL -dump > webpage_as_text.txt
```
##### curl
todo

#### 归档、压缩和备份
#####  tar
所有Unix类操作系统中都默认包含tar命令，它语法简单，文件格式具备可移植性。tar支持的参数包括：A、c、d、r、u、x、f和v。其中的参数可以根据情况使用。
* 用tar对文件进行归档
```
$ tar -cf output.tar [SOURCES]
```
`-c`代表“创建文件”， `-f`代表“指定文件名”

* 使用选项`-t`列出归档文件中包含的文件
```
$ tar -tf archive.tar
```

* 若希望在归档或获取归档列表时获知更多的信息，可以使用`-v`或`-vv`参数。

* 若希望向归档文件中添加一个文件
```
$ tar -rvf original.tar new_file
```

* 从归档文件中删除文件
```
$ tar -f archive.tar --delete file1 file2 ...
```

* 归档时排除部分文件
```
$ tar -cf archive.tar *  --exclude "*.txt" 
# 排除 txt 文件
```
也可以将排除的文件列表放入文件中
```
$ cat list
file1
file2
$ tar -cf archive.tar * -X list
#这样就无需对file1 file2进行归档
```

* 从归档文件中提取文件或文件夹
```
$ tar -xf archive.tar
```
`-x`表示提取（extract）, 可以通过`-C`来指定需要将文件提取到哪个目录。我们可以通过将文件名指定为命令行的参数来提取指定的文件：`tar -xvf file.tar file1 file4`仅提取 file1 和 file4

* 拼接两个归档文件
```
tar -Af file1.tar file2.tar
```

##### cpio
cpio 通过stdin 获取输入文件名，并将归档文件写入stdout。
```
$ touch file1 file2 file3  # 创建文件

$ echo file1 file2 file3 | cpio -ov > archive.cpio # 归档

$ cpio -it < archive.cpio # 列出cpio归档文件中的内容

$ cpio -id < archive.cpio # 从cpio归档文件中提取文件
```

##### rsync
Rsync的命令格式可以为以下六种：
```
　　rsync [OPTION]... SRC DEST 
　　rsync [OPTION]... SRC [USER@]HOST:DEST 
　　rsync [OPTION]... [USER@]HOST:SRC DEST 
　　rsync [OPTION]... [USER@]HOST::SRC DEST 
　　rsync [OPTION]... SRC [USER@]HOST::DEST 
　　rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST] 
```
　　对应于以上六种命令格式，rsync有六种不同的工作模式： 
　　1)拷贝本地文件。当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。如：rsync -a /data /backup 
　　2)使用一个远程shell程序(如rsh、ssh)来实现将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号":"分隔符时启动该模式。如：rsync -avz *.c foo:src 
　　3)使用一个远程shell程序(如rsh、ssh)来实现将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号":"分隔符时启动该模式。如：rsync -avz foo:src/bar /data 
　　4)从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。如：rsync -av root@172.16.78.192::www /databack 
　　5)从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。如：rsync -av /databack root@172.16.78.192::www 
　　6)列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。如：rsync -v rsync://172.16.78.192/www 

rsync参数的具体解释如下： 
-v, --verbose 详细模式输出 
-q, --quiet 精简输出模式 
-c, --checksum 打开校验开关，强制对文件传输进行校验 
-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD 
-r, --recursive 对子目录以递归模式处理 
-R, --relative 使用相对路径信息 
-b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。 
--backup-dir 将备份文件(如~filename)存放在在目录下。 
-suffix=SUFFIX 定义备份文件前缀 
-u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件) 
-l, --links 保留软链结 
-L, --copy-links 想对待常规文件一样处理软链结 
--copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结 
--safe-links 忽略指向SRC路径目录树以外的链结 
-H, --hard-links 保留硬链结 
-p, --perms 保持文件权限 
-o, --owner 保持文件属主信息 
-g, --group 保持文件属组信息 
-D, --devices 保持设备文件信息 
-t, --times 保持文件时间信息 
-S, --sparse 对稀疏文件进行特殊处理以节省DST的空间 
-n, --dry-run现实哪些文件将被传输 
-W, --whole-file 拷贝文件，不进行增量检测 
-x, --one-file-system 不要跨越文件系统边界 
-B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节 
-e, --rsh=COMMAND 指定使用rsh、ssh方式进行数据同步 
--rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息 
-C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件 
--existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件 
--delete 删除那些DST中SRC没有的文件 
--delete-excluded 同样删除接收端那些被该选项指定排除的文件 
--delete-after 传输结束以后再删除 
--ignore-errors 及时出现IO错误也进行删除 
--max-delete=NUM 最多删除NUM个文件 
--partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输 
--force 强制删除目录，即使不为空 
--numeric-ids 不将数字的用户和组ID匹配为用户名和组名 
--timeout=TIME IP超时时间，单位为秒 
-I, --ignore-times 不跳过那些有同样的时间和长度的文件 
--size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间 
--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0 
-T --temp-dir=DIR 在DIR中创建临时文件 
--compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份 
-P 等同于 --partial 
--progress 显示备份过程 
-z, --compress 对备份的文件在传输时进行压缩处理 
--exclude=PATTERN 指定排除不需要传输的文件模式 
--include=PATTERN 指定不排除而需要传输的文件模式 
--exclude-from=FILE 排除FILE中指定模式的文件 
--include-from=FILE 不排除FILE指定模式匹配的文件 
--version 打印版本信息 
--address 绑定到特定的地址 
--config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件 
--port=PORT 指定其他的rsync服务端口 
--blocking-io 对远程shell使用阻塞IO 
-stats 给出某些文件的传输状态 
--progress 在传输时现实传输过程 
--log-format=formAT 指定日志文件格式 
--password-file=FILE 从FILE中得到密码 
--bwlimit=KBPS 限制I/O带宽，KBytes per second 
-h, --help 显示帮助信息