# Linux使用笔记
## 一、Linux文件与目录管理

- **绝对路径：**
  路径的写法，由根目录 / 写起，例如： /usr/share/doc 这个目录。

- **相对路径：**
  路径的写法，不是由 / 写起，例如由 /usr/share/doc 要到 /usr/share/man 底下时，可以写成： cd ../man 这就是相对路径的写法啦！

  注：`/` 表示根目录，`..`表示 当前目录

- **ls**: 列出目录
- **cd**：切换目录
- **pwd**：显示目前的目录
- **mkdir**：创建一个新的目录
- **rmdir**：删除一个空的目录
- **cp**: 复制文件或目录
- **rm**: 移除文件或目录
- **mv**: 移动文件与目录，或修改文件与目录的名称

> **你可以使用 man [命令] 来查看各个命令的使用文档，如 ：man cp。**

## 二、Linux 文件内容查看

Linux系统中使用以下命令来查看文件的内容：

- **cat**  由第一行开始显示文件内容
- **tac**  从最后一行开始显示，可以看出 **tac 是 cat 的倒着写**！
- **nl**   显示的时候，顺道输出行号！
- **more** 一页一页的显示文件内容
- **less** 与 **more** 类似，但是比 more 更好的是，他可以往前翻页！
- **head** 只看头几行
- **tail** 只看尾巴几行

### more

一页一页翻动

```
[root@www ~]# more /etc/man_db.config 
#
# Generated automatically from man.conf.in by the
# configure script.
#
# man.conf from man-1.6d
....(中间省略)....
--More--(28%)  <== 重点在这一行喔！你的光标也会在这里等待你的命令
```

在 more 这个程序的运行过程中，你有几个按键可以按的：

- 空白键 (space)：代表向下翻一页；
- Enter         ：代表向下翻『一行』；
- /字串         ：代表在这个显示的内容当中，向下搜寻『字串』这个关键字；
- :f            ：立刻显示出档名以及目前显示的行数；
- q             ：代表立刻离开 more ，不再显示该文件内容。
- b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。

## 三、Linux系统用户账号的管理

指定和修改用户口令的Shell命令是`passwd`。超级用户可以为自己和其他用户指定口令，普通用户只能用它修改自己的口令。命令的格式为：

```
passwd 选项 用户名
```

可使用的选项：

- -l 锁定口令，即禁用账号。
- -u 口令解锁。
- -d 使账号无口令。
- -f 强迫用户下次登录时修改口令。

如果默认用户名，则修改当前用户的口令。

## 四、Linux 磁盘管理

Linux磁盘管理好坏直接关系到整个系统的性能问题。

Linux磁盘管理常用三个命令为df、du和fdisk。

- df：列出文件系统的整体磁盘使用量
- du：检查磁盘空间使用量
- fdisk：用于磁盘分区

## 五、Linux vi/vim

**vi/vim 的使用**

基本上 vi/vim 共分为三种模式，分别是**命令模式（Command mode）**，**输入模式（Insert mode）**和**底线命令模式（Last line mode）**。 这三种模式的作用分别是：

### 命令模式：

用户刚刚启动 vi/vim，便进入了命令模式。

此状态下敲击键盘动作会被Vim识别为命令，而非输入字符。比如我们此时按下i，并不会输入一个字符，i被当作了一个命令。

以下是常用的几个命令：

- **i** 切换到输入模式，以输入字符。
- **x** 删除当前光标所在处的字符。
- **:** 切换到底线命令模式，以在最底一行输入命令。

若想要编辑文本：启动Vim，进入了命令模式，按下i，切换到输入模式。

命令模式只有一些最基本的命令，因此仍要依靠底线命令模式输入更多命令。

### 输入模式

在命令模式下按下i就进入了输入模式。

在输入模式中，可以使用以下按键：

- **字符按键以及Shift组合**，输入字符
- **ENTER**，回车键，换行
- **BACK SPACE**，退格键，删除光标前一个字符
- **DEL**，删除键，删除光标后一个字符
- **方向键**，在文本中移动光标
- **HOME**/**END**，移动光标到行首/行尾
- **Page Up**/**Page Down**，上/下翻页
- **Insert**，切换光标为输入/替换模式，光标将变成竖线/下划线
- **ESC**，退出输入模式，切换到命令模式

### 底线命令模式

在命令模式下按下:（英文冒号）就进入了底线命令模式。

底线命令模式可以输入单个或多个字符的命令，可用的命令非常多。

在底线命令模式中，基本的命令有（已经省略了冒号）：

- q 退出程序
- w 保存文件

按ESC键可随时退出底线命令模式。

简单的说，我们可以将这三个模式想成底下的图标来表示：

![](E:\MyBlog\docs\MyNote\工作模式.png)

## 六、yum常用命令

- 1.列出所有可更新的软件清单命令：yum check-update
- 2.更新所有软件命令：yum update
- 3.仅安装指定的软件命令：yum install <package_name>
- 4.仅更新指定的软件命令：yum update <package_name>
- 5.列出所有可安裝的软件清单命令：yum list
- 6.删除软件包命令：yum remove <package_name>
- 7.查找软件包 命令：yum search <keyword>
- 8.清除缓存命令:
  - yum clean packages: 清除缓存目录下的软件包
  - yum clean headers: 清除缓存目录下的 headers
  - yum clean oldheaders: 清除缓存目录下旧的 headers
  - yum clean, yum clean all (= yum clean packages; yum clean oldheaders) :清除缓存目录下的软件包及旧的headers



------

# Shell 教程

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。



## Shell 脚本

Shell 脚本（shell script），是一种为 shell 编写的脚本程序。

业界所说的 shell 通常都是指 shell 脚本，但读者朋友要知道，shell 和 shell script 是两个不同的概念。

由于习惯的原因，简洁起见，本文出现的 "shell编程" 都是指 shell 脚本编程，不是指开发 shell 自身。



## Shell 环境

Shell 编程跟 JavaScript、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。

Linux 的 Shell 种类众多，常见的有：

- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）
- ……

本教程关注的是 Bash，也就是 Bourne Again Shell，由于易用和免费，Bash 在日常工作中被广泛使用。同时，Bash 也是大多数Linux 系统默认的 Shell。



## 第一个shell脚本

打开文本编辑器(可以使用 vi/vim 命令来创建文件)，新建一个文件 test.sh，扩展名为 sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用 php 写 shell 脚本，扩展名就用 php 好了。

```shell
#!/bin/bash
echo "Hello World !"
```

**#! 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell。**

### 运行 Shell 脚本有两种方法：

**1、作为可执行程序**

将上面的代码保存为 test.sh，并 cd 到相应目录：

```shell
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

注意，一定要写成 **./test.sh**，而不是 **test.sh**，运行其它二进制的程序也一样，直接写 test.sh，linux 系统会去 PATH 里寻找有没有叫 test.sh 的，而只有 /bin, /sbin, /usr/bin，/usr/sbin 等在 PATH 里，你的当前目录通常不在 PATH 里，所以写成 test.sh 是会找不到命令的，要用 ./test.sh 告诉系统说，就在当前目录找。

**2、作为解释器参数**

这种运行方式是，直接运行解释器，其参数就是 shell 脚本的文件名，如：

```
/bin/sh test.sh
/bin/php test.php
```

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

# Shell 数组

数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小（与 PHP 类似）。

与大部分编程语言类似，数组元素的下标由0开始。

Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式如下：

```
array_name=(value1 ... valuen)
```



### 显示命令执行结果

```
echo `date`
```

**注意：** 这里使用的是反引号 **`**, 而不是单引号 **'**。

结果将显示当前日期

```
Thu Jul 24 10:08:46 CST 2014
```



# Shell 输入/输出重定向

大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。

重定向命令列表如下：

| 命令            | 说明                                               |
| :-------------- | :------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |



## /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：

```
$ command > /dev/null
```

/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

如果希望屏蔽 stdout 和 stderr，可以这样写：

```
$ command > /dev/null 2>&1
```

> **注意：**0 是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。



## Linux 常用命令全拼

- pwd: print work directory 打印当前目录 显示出当前工作目录的绝对路径

- ps: process status(进程状态，类似于windows的任务管理器)

  常用参数：－auxf

  ps -auxf 显示进程状态

- df: disk free 其功能是显示磁盘可用空间数目信息及空间结点信息。换句话说，就是报告在任何安装的设备或目录中，还剩多少自由的空间。

- du: Disk usage

- rpm：即RedHat Package Management，是RedHat的发明之一

- rmdir：Remove Directory（删除目录）

- rm：Remove（删除目录或文件）

- cat: concatenate 连锁

- cat file1file2>>file3 把文件1和文件2的内容联合起来放到file3中

- insmod: install module,载入模块

- ln -s : link -soft 创建一个软链接，相当于创建一个快捷方式

- mkdir：Make Directory(创建目录)

- touch: touch

- man: Manual

- su：Swith user(切换用户)

- cd：Change directory

- ls：List files

- ps：Process Status

- mkdir：Make directory

- rmdir：Remove directory

- mkfs: Make file system

- fsck：File system check

- uname: Unix name

- lsmod: List modules

- mv: Move file

- rm: Remove file

- cp: Copy file

- ln: Link files

- fg: Foreground

- bg: Background

- chown: Change owner

- chgrp: Change group

- chmod: Change mode

- umount: Unmount

- dd: 本来应根据其功能描述"Convert an copy"命名为"cc"，但"cc"已经被用以代表"CComplier"，所以命名为"dd"

- tar：Tape archive （磁带档案）

- ldd：List dynamic dependencies

- insmod：Install module

- rmmod：Remove module

- lsmod：List module

- 文件结尾的"rc"（如.bashrc、.xinitrc等）：Resource configuration

- Knnxxx /Snnxxx（位于rcx.d目录下）：K（Kill）；S(Service)；nn（执行顺序号）；xxx（服务标识）

- .a（扩展名a）：Archive，static library

- .so（扩展名so）：Shared object，dynamically linked library

- .o（扩展名o）：Object file，complied result of C/C++ source file

- RPM：Red hat package manager

- dpkg：Debian package manager

- apt：Advanced package tool（Debian或基于Debian的发行版中提供）

### 其他 Linux 命令缩写

```shell
bin = Binaries (二进制文件)

/dev = Devices (设备)

/etc = Etcetera (等等)

/lib = LIBrary

/proc = Processes

/sbin = Superuser Binaries (超级用户的二进制文件)

/tmp = Temporary (临时)

/usr = Unix Shared Resources

/var = Variable (变量)

FIFO = First In, First Out

GRUB = GRand Unified Bootloader

IFS= Internal Field Seperators

LILO = LInux LOader

MySQL = My 是最初作者女儿的名字，

SQL = Structured QueryLanguage

PHP = Personal Home Page Tools = PHP HypertextPreprocessor

PS = Prompt String

Perl = “Pratical Extraction and Report Language”(实际的抽取和报告语言) =”Pathologically Eclectic Rubbish Lister”

Python 得名于电视剧Monty Python’s Flying Circus

Tcl = Tool Command Language

Tk = ToolKit

VT = Video Terminal

YaST = Yet Another Setup Tool

apache = “a patchy” server

apt = Advanced Packaging Tool

ar = archiver

as = assembler

awk = “Aho Weiberger and Kernighan”三个作者的姓的第一个字母

bash = Bourne Again SHell

bc = Basic (Better) Calculator

bg = BackGround

biff = 作者HeidiStettner在U.C.Berkely养的一条狗,喜欢对邮递员汪汪叫。

cal = Calendar (日历)

cat = Catenate (链接)

cd = Change Directory

chgrp = Change Group

chmod = Change Mode

chown = Change Owner

chsh = Change Shell

cmp = compare

cobra = Common Object Request BrokerArchitecture

comm = common

cp = Copy

cpio = CoPy In and Out

cpp = C Pre Processor

cron = Chronos 希腊文时间

cups = Common Unix Printing System

cvs = Current Version System

daemon = Disk And Execution MONitor

dc = Desk Calculator

dd = Disk Dump (磁盘转储)

df = Disk Free

diff = Difference

dmesg = diagnostic message

du = Disk Usage

ed = editor

egrep = Extended GREP

elf = Extensible Linking Format

elm = ELectronic Mail

emacs = Editor MACroS

eval = EVALuate

ex = EXtended

exec = EXECute (执行)

fd = file descriptors

fg = ForeGround

fgrep = Fixed GREP

fmt = format

fsck = File System ChecK

fstab = FileSystem TABle

fvwm = F*** Virtual Window Manager

gawk = GNU AWK

gpg = GNU Privacy Guard

groff = GNU troff

hal = Hardware Abstraction Layer

joe = Joe’s Own Editor

ksh = Korn SHell

lame = Lame Ain’t an MP3 Encoder

lex = LEXical analyser

lisp = LISt Processing = Lots of IrritatingSuperfluous Parentheses

ln = Link

lpr = Line PRint

ls = list

lsof = LiSt Open Files

m4 = Macro processor Version 4

man = MANual pages

mawk = Mike Brennan’s AWK

mc = Midnight Commander

mkfs = MaKe FileSystem

mknod = Make Node

motd = Message of The Day

mozilla = MOsaic GodZILLa

mtab = Mount TABle

mv = Move

nano = Nano’s ANOther editor

nawk = New AWK

nl = Number of Lines

nm = names

nohup = No HangUP

nroff = New ROFF

od = Octal Dump

passwd = Passwd

pg = pager

pico = PIne’s message COmposition editor

pine = “Program for Internet News &Email” = “Pine is not Elm”

ping = 拟声 又 = Packet Internet Grouper

pirntcap = PRINTer CAPability

popd = POP Directory

pr = pre

printf = Print Formatted

ps = Processes Status

pty = pseudo tty

pushd = PUSH Directory

pwd = Print Working Directory

rc = runcom = run command, rc还是plan9的shell

rev = REVerse

rm = ReMove

rn = Read News

roff = RunOFF

rpm = RPM Package Manager = RedHat PackageManager

rsh, rlogin, rvim中的

r = Remote

rxvt = ouR XVT

seamoneky = 我

sed = Stream Editor

seq = SEQuence

shar = Shell ARchive

slrn = S-Lang rn

ssh = Secure Shell

ssl = Secure Sockets Layer

stty = Set TTY

su = Substitute User

svn = SubVersion

tar = Tape ARchive

tcsh = TENEX C shell

tee = T (T形水管接口)

telnet = TEminaL over Network

termcap = terminal capability

terminfo = terminal information

tex = τέχνη的缩写，希腊文art

tr = traslate

troff = Typesetter new ROFF

tsort = Topological SORT

tty = TeleTypewriter

twm = Tom’s Window Manager

tz = TimeZone

udev = Userspace DEV

ulimit = User’s LIMIT

umask = User’s MASK

uniq = UNIQue

i = VIsual = Very Inconvenient

vim = Vi IMproved

wall = write all

wc = Word Count

wine = WINE Is Not an Emulator

xargs = eXtended ARGuments

xdm = X Display Manager

xlfd = X Logical Font Description

xmms = X Multimedia System

xrdb = X Resources DataBase

xwd = X Window Dump

yacc = yet another compiler compiler

Fish = the Friendly Interactive SHell

su = Switch User

MIME = Multipurpose Internet Mail Extensions

ECMA = European Computer ManufacturersAssociation
```