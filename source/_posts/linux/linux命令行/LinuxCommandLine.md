---
title: Linux 命令行
tags:
     - linux
     - linux 命令行
     - linux shell
---

### 启动，停止

* 启动：
    * Ctrl + Alt + T
    * 鼠标右键-->在命令行中打开
* 停止：
    * exit

<!-- more -->

### 文件系统跳转
* pwd  Print name of current working directory! 打印当前工作目录名称
* cd   Change Directory 更改当前工作目录路径
    * cd 无任何参数     更改当前工作目录到你的home目录
    * cd -            更改当前工作目录到先前工作目录
    * cd ～username    更改工作目录到指定用户主目录
    
### 探究操作系统
* ls   List current directory content 列出当前目录内容
    * ls -a/-all            列出所有文件，包括所有以.开头默认被隐藏的文件
    * ls -d/-directory      列出指定目录中的全部内容而不是当前目录
    * ls -F/-classify       此命令会为每个条目后面加上一个标识符，比如文件夹‘/’
    * ls -h/-human-readable 此命令以便于人眼阅读的模式展示文件信息，比如文件大小为KB或者，MB，而不是bytes
    * ls -l                 以长格式现实文件信息，一行一个文件
        * 示例：-rw-r--r--  1 zhangsan zhangsan 0  4月1921:24 xx.ts
        * -rw-r--r-- 文件访问权限，开头的“-”说明是一个普通文件,“d”表明是一个目录。其后三个字符是文件所有者的访问权限,再其后的三个字符是文件所属组中成员的访问权限,最后三个字符是其他所有人的访问权限
        * 1          文件硬连接的数目
        * zhangsan   文件所属用户名称
        * zhangsan   文件所属用户组名称
        * 0          以字节数展示的文件大小
        * 4月1921:24  最后一次文件修改时间
        * xx.ts      文件名称
    * ls -r/-reverse        以相反的顺序展示文件，默认是按照字母表升序展示
    * ls -S                 输出文件按照文件大小排序
    * ls -t                 输出文件按照修改时间排序
* file  确定文件类型
    * file picture.jpg --> picture.jpg: JPEG image data, JFIF standard 1.01
* less 浏览文件
    * less somefile.txt  浏览指定文件
        * page up     向上翻滚一页
        * page down   向下翻滚一页
        * up arrow    向上翻滚一行
        * down arrow  向下翻滚一行
        * G           跳转到最后一行
        * 1g/1G/g     跳转到第一行
        * /charaters  向前查找指定字符串
        * n           向前查找字符串，这个字符串是之前查找的
        * h           显示帮助屏幕
        * q           退出程序
* more
    * less is more(色即是空)
    * less 相对与 more功能更加强大，more只支持分页浏览
* 一些常用目录
    * /         根目录，万物起源
    * /bin      包含系统启动和运行所必须的二进制程序
    * /boot     linux 内核
    * /dev      一个包含设备文件的目录，linux中一切皆文件。
    * /etc      一些系统层面的配置文件
        * /etc/crontab      定义自动运行的任务
        * /etc/passwd       包含账户列表
        * /etc/hosts        主机配置信息
        * /etc/profile      系统环境变量
        * /etc/nginx/sites-available/default     nginx配置文件位置，如果你用apt-get安装nginx的话
    * /home     通常，linux会在home目录下面为每一个用户创建一个目录，每个普通用户只能访问和操作自己用户目录下面的文件
    * /lib      系统运行所需要的核心类库
    * /media    挂载设备所在文件夹（一切皆文件）
    * /root     管理员用户根目录
    * /sbin     包含系统二进制文件，通常为超级用户保留
    * /tmp      用于存储系统程序运行时的临时文件，某些系统配置会在系统启动时清空这个目录
    * /usr      包含普通用户所需要的程序和文件
    * /usr/bin  普通用户安装的文件
    * /usr/lib  普通用户共享的类库
    * /usr/local  通常用于安装从程序源码编译安装的软件
    * /var/log   系统日志所在文件

### 操作文件和目录
* cp        copy file and directory             复制文件和目录
* mv        move/rename files and directory     移动/重命名文件和目录
* mkdir     credit directories                  创建目录
* rm        remove file and directory           移除文件和目录
* ln        create hard and symbolic links      创建硬连接和符号链接

* 通配符
在介绍上面这些命令之前，我们首先一个使命令行无比强大的shell特性。因为shell命令需要频繁的使用文件名，所以shell提供了一组特殊字符用来快速指定某一组文件名，这些特殊字符就叫做通配符。
    * 常见通配符
        * \*                 匹配任意多的字符（包括零个或者一个）
        * ?                 匹配任意一个字符（不包括零个）
        * [characters]      匹配任意一个属于字符集中的字符
        * [!characters]     匹配任意一个不再指定字符集中的字符
        * [[:class:]]       匹配任意一个属于指定字符类中的字符
            * [:alnum:]         匹配任意一个字母或者数字
            * [:alpha:]         匹配任意一个字母
            * [:digit:]         匹配任意一个数字
            * [:lower:]         匹配任意一个小写字母
            * [:upper:]         匹配任意一个大写字母
    * 常见通配符示例
        * \*                 全部文件
        * g*                所有以g开头的文件
        * b*.txt            以b开头中间有零个或者多个任意字符，并以.txt结尾的文件
        * Data???           以Data开头，其后紧接着3个字符的文件
        * [abc]*            文件名以abc开头的文件
        * BACKUP.[0-9][0-9][0-9]        以BACKUP.开头并紧接着三个数字的文件
        * [[:upper:]]*      以大写字母开头的文件
        * *[[:lower:]123]   文件名以小写字母结尾或者是以1,2,3结尾
    
* mkdir -- 创建目录
    * mkdir directory           创建文件directory
    * mkdir dir1 dir2 dir3      创建文件夹dir1,dir2,dir3
* cp -- 复制文件和目录
    * cp fromFileOrDir toFileOrDir    复制文件或文件夹fromFileOrDir到文件或文件夹toFileOrDir
    * cp item1 item2 ... toDirectory  复制多个文件到文件夹toDirectory
    * 有用的选项
        * -a        -archive            复制文件和目录，以及他们的属性，包括所有权和权限
        * -i        -interactive        在重写已存在文件之前提示用户确认，如果不指定，cp命令会默认重写文件
        * -r        -recursive          递归的复制目录以及目录中的内容，当复制目录的时候需要指定这个选项
        * -u        -update             当复制文件时，只有在目标文件夹中不存在这个文件或者当前文件新于目标文件夹中的文件时才复制文件
        * -v        -verbose            现实详细的命令操作信息
    * 有用的示例
        * cp file1 file2                复制文件file1到file2,如果file2不存在那么创建，如果存在，那么覆盖
        * cp -i file2 file2             复制文件file1到file2,如果file2不存在那么创建，如果存在，那么提醒用户确认
        * cp file1 file2 dir            复制文件file1,file2到文件夹dir
        * cp dir1/* dir2                复制文件夹1中的文件到dir2，dir2必须存在
        * cp -r dir1 dir2               递归复制文件夹dir1中的内容到dir2
* mv 移动和重命名文件
    * mv fromFileOrDir toFileOrDir      移动（重命名）文件或文件夹fromFileOrDir到toFileOrDir
    * mv item1 item2 ... toDirectory    将一个或多个条目移动到另一个目录中
    * 有用的选项
        * -i        -interactive        在重写一个已经存在的文件之前，提示用户确认，如果不指定，mv命令会默认重写文件内容
        * -u        -update             当把一个文件从一个目录移动到另一个目录时，只移动不存在的文件，或者较新的文件
        * -v        -verbose            显示详细的信息
    * 有用的示例
        * mv file1 file2                移动文件file1到file2,如果file2不存在直接复制，否则覆盖文件
        * mv -i file1 file2             移动文件file1到file2,如果file2不存在直接复制，否则提用户确认
        * mv file1 file2 dir            移动file1,file2到文件夹dir中，dir必须已经存在
        * mv dir1 dir2                  如果dir2不存在，那么新建dir2,并且移动dir1中的内容到dir2,然后删除dir1。如果dir2存在，移动dir1中的内容到dir2.
* rm -- 删除文件和目录
    * rm item1 item2 。。。              删除一个或多个文件或目录
    * 选项
        * -i        -interactive        在删除已经存在的文件前，提示用户确认，不然，此命令会默默删除文件
        * -r        -recursive          递归的删除文件,用于删除目录
        * -f        -force              不显示任何提示信息，同-interactive命令相反
        * -v        -verbose            现实详细的操作信息
    * 命令
        * rm file1                      默默地删除文件file1
        * rm -i file1                   删除文件之前，提醒用户确认
        * rm -r file1,dir1              删除文件file1,递归的删除文件夹dir1中的全部内容（文件以及dir）
        * rm -rf file1,dir1             同上，如果file1或者dir1不存在，此命令仍然会执行
* ln -- 创建链接
    * ln file link                      创建一个硬连接
    * ln -s file link                   创建一个软链接
    * 硬链接
        * 硬连接和符号链接比起来，硬连接是最初Unix创建链接的方式，而符号链接更加现代，在默认情况下，每个文件有一个硬链接，这个硬链接给文件起名字。当我们创建了一个硬链接之后，就为文件创建了一个额外的目录条目。
        * 一个硬链接不能关联它所在文件系统之外的其他文件，这是说一个链接不能关联与链接本身不在同一个磁盘分区上的文件。
        * 一个硬连接不能关联一个目录
        * 一个硬链接和文件本身没有什么区别。不像符号链接,当你列出一个包含硬链接的目录内容时,你会看到没有特殊的链接指示说明。当一个硬链接被删除时,这个链接被删除,但是文件本身的内容仍然存在(这是说,它所占的磁盘空间不会被重新分配),直到所有关联这个文件的链接都删除掉。
          
    *符号链接
        * 一个符号链接指向一个文件,而且这个符号链接本身与其它的符号链接几乎没有区别。例如,如果你往一个符号链接里面写入东西,那么相关联的文件也被写入。然而,当你删除一个
          符号链接时,只有这个链接被删除,而不是文件自身。如果先于符号链接删除文件,这个链接仍然存在,但是不指向任何东西。在这种情况下,这个链接被称为坏链接。

### 使用命令

* 什么是命令
    * 命令可以是一个可执行程序，例如任何在位于/usr/bin中的文件,java,python等
    * 内建与shell自身的命令，或者说shell内部命令
    * shell脚本，其通常包含一组想关联的shell命令，共同完成某项功能
    * 一个命令别名（网络上有一些很有意思的整蛊的文章，比如将ls命令作为rm -rf命令的别名，这样就不是列出一个文件夹的内容了，而是默默删除整个文件夹）
* 识别命令--type
    * type command
    * command 是需要识别的命令名称
    
            icekredit@IceKredit ~ $ type type
            type is a shell builtin
            icekredit@IceKredit ~ $ type ls
            ls is aliased to `ls --color=auto'
            icekredit@IceKredit ~ $ type cp
            cp is /bin/cp
            icekredit@IceKredit ~ $ type java
            java is /usr/local/java/jdk1.8.0_73/bin/java
    * 显示一个可执行程序的位置
    
            icekredit@IceKredit ~ $ which ls
            /bin/ls
            icekredit@IceKredit ~ $ which cp
            /bin/cp
            icekredit@IceKredit ~ $ which java
            /usr/local/java/jdk1.8.0_73/bin/java
     此命令只对可执行程序有效
    * 得到shell内部命令的帮助文档
        
        icekredit@IceKredit ~ $ help cd
        cd: cd [-L|[-P [-e]] [-@]] [dir]
            Change the shell working directory.
            
            Change the current directory to DIR.  The default DIR is the value of the
            HOME shell variable.
            
            The variable CDPATH defines the search path for the directory containing
            DIR.  Alternative directory names in CDPATH are separated by a colon (:).
            A null directory name is the same as the current directory.  If DIR begins
            with a slash (/), then CDPATH is not used.
            
            If the directory is not found, and the shell option `cdable_vars' is set,
            the word is assumed to be  a variable name.  If that variable has a value,
            its value is used for DIR.
            
            Options:
                -L	force symbolic links to be followed: resolve symbolic links in
            	DIR after processing instances of `..'
                -P	use the physical directory structure without following symbolic
            	links: resolve symbolic links in DIR before processing instances
            	of `..'
                -e	if the -P option is supplied, and the current working directory
            	cannot be determined successfully, exit with a non-zero status
                -@  on systems that support it, present a file with extended attributes
                    as a directory containing the file attributes
            
            The default is to follow symbolic links, as if `-L' were specified.
            `..' is processed by removing the immediately previous pathname component
            back to a slash or the beginning of DIR.
            
            Exit Status:
            Returns 0 if the directory is changed, and if $PWD is set successfully when
            -P is used; non-zero otherwise.
        *出现在命令语法说明中的方括号表示可选的项目，一个竖杠字符表示互斥选项

    * 显示程序手册 -- --help
    
        icekredit@IceKredit ~ $ mkdir --help
        Usage: mkdir [OPTION]... DIRECTORY...
        Create the DIRECTORY(ies), if they do not already exist.
        
        Mandatory arguments to long options are mandatory for short options too.
          -m, --mode=MODE   set file mode (as in chmod), not a=rwx - umask
          -p, --parents     no error if existing, make parent directories as needed
          -v, --verbose     print a message for each created directory
          -Z, --context=CTX  set the SELinux security context of each created
                              directory to CTX
              --help     display this help and exit
              --version  output version information and exit
    * 显示程序手册 -- man
        * 大多数系统中，man使用less程序显示参考手册
    * 显示适当的命令 -- apropos
        * 基于关键字进行匹配
    
        icekredit@IceKredit ~ $ apropos disk
        arm_sync_file_range (2) - sync a file segment with disk
        baobab (1)           - A graphical tool to analyze disk usage
        cfdisk (8)           - display or manipulate disk partition table
        cgdisk (8)           - Curses-based GUID partition table (GPT) manipulator
        cryptdisks_start (8) - wrapper around cryptsetup that parses /etc/crypttab.
        cryptdisks_stop (8)  - wrapper around cryptsetup that parses /etc/crypttab.
        df (1)               - report file system disk space usage
        。。。
        
    * 显示非常简洁的命令说明 -- whatis
    
        icekredit@IceKredit ~ $ whatis ls
        ls (1)               - list directory contents

    * 显示程序info条目 -- info
    * 可以将多个命令放在一行，用多个分号分隔
    * 为一个或多个命令创建别名
        
        icekredit@IceKredit ~ $ alias foo='cd /usr; ls; cd -'
        icekredit@IceKredit ~ $ type foo
        foo is aliased to `cd /usr; ls; cd -'

    * 删除别名
    
        icekredit@IceKredit ~ $ type foo
        foo is aliased to `cd /usr; ls; cd -'
        icekredit@IceKredit ~ $ unalias foo
        icekredit@IceKredit ~ $ type foo
        bash: type: foo: not found
    

    
        
       