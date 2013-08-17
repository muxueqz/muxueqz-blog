lftp详解
##########################################################################################################################################
:date: 2009-09-14 13:03:18
:category: 系统运维
:tags: ftp
:slug: lftp

 

lftp是我认为最好的FTP客户端，ftp,http,sftp,fish,file,BitTorrent(bt,想不到吧，我也是刚看到的)等协议,命令行也很易用高效，支持多编码多队列多线程，可以用“TAB”键来补全命令和文件名哦。

首先，所有的lftp终端内的命令，都可以用

 

    help    (显示所有可以使用的命令)

 

    help 命令名

 

的方式来查看帮助信息。

 

1、登录ftp

 

    lftp 用户名:密码@ftp地址:传送端口（默认21）

 

也可以先不带用户名登录，默认是anonymous匿名用户，然后在命令行下用login或user命令来用指定账号登录，密码不显示。

 

2、查看文件与改变目录

 

    ls

 

    cd 对应ftp目录

 

嘿嘿，很简单吧？其实，在lftp终端中，前面带一个l的命令例如lcd，指的是local，就是在本机的操作，而对应的没有这个l的，都是对ftp
site的操作。还有就是要执行本地终端命令，也可以用前面带一个!的方式。这样，配合起来，终端，本地的操作都很方便。

例如，查看ftp上所有名字中包含mp3的文件：

 

    find . \|grep -i mp3

 

3、下载

 

    get当然是可以的，还可以

 

    mget -c \*.pdf

 

    把所有的pdf文件以允许断点续传的方式下载。m代表multi

 

    mirror aaa/

 

    将aaa目录整个的下载下来，子目录也会自动复制

 

    pget -c -n 10 file.dat

 

    以最多10个线程以允许断点续传的方式下载file.dat

    可以通过设置pget:default-n的值而使用默认值。

 

4、上传

 

    同样的put，mput，都是对文件的操作，和下载类似。

 

    mirror -R 本地目录名

 

    将本地目录以迭代（包括子目录）的方式反向上传到ftp site。

 

5、模式设置。

 

    set ftp:charset gbk

 

    远程ftp site用gbk编码，对应的要设置为utf8,只要替换gbk为utf8即可。

 

    set file:charset utf8

 

    本地的charset设定为utf8,如果你是gbk，相应改掉。

 

    set ftp:passive-mode 1

 

  
 使用被动模式登录，有些site要求必须用被动模式或者主动模式才可以登录，这个开关就是设置这个的。0代表不用被动模式。

 

6、书签

 

    其实命令行也可以有书签，在lftp终端提示符下：

 

    bookmark add ustc

 

    就可以把当前正在浏览的ftp
site用ustc作为标签储存起来。以后在shell终端下，直接

 

    lftp ustc

 

    就可以自动填好用户名，密码，进入对应的目录了。

 

    bookmark edit

 

  
 会调用编辑器手动修改书签。当然，也可以看到，这个书签其实就是个简单的文本文件。密码，用户名都可以看到。

 

7、配置文件

 

/etc/lftp.conf

 

一般，我会添加这几行：

 

    set ftp:charset gbk

    set file:charset utf8

    set pget:default-n 5

 

这样，就不用每次进入都要打命令了。其他的set 可以自己tab然后help 来看。

 

8. lftp 的下载上传的速度限制

 

於 lftp.conf 中加入

 

    set net:limit-rate 10000,10000 （单位是字节,前者是下载，后者是上传）

 

9. lftp 缺省不会显示 ftp
服务器的欢迎信息和错误信息，这在很多时候不方便，因为你有可能想知道这个服务器到底是因为没开机连不上，还是连接数已

 

满。如果是这样，你可以在 ~/.lftprc 里写入一行

 

    debug 3

 

下载/上传/多线程 文件夹

 

    lftp> mirror dir

 

    lftp> mirror -R dir

 

    lftp> mirror –parallel=3 dir

 

纠正乱码显示中文及显示登录消息设置，修改/etc/lftp.conf，加入一段

 

    #myset begin

 

    set ftp:charset “gbk”

 

    set file:charset “UTF-8″

 

    #end

 

    debug 3

 

更改本地下载目录

 

    ftp> lcd ldir

 

默认为本地当前目录

 

比如改成lcd /home/user/download

 

队列命令（不必等到下载完毕再输入命令）

 

    > queue

 

查看后台队列

 

    > jobs

 

加入队列

 

    >queue get file

 

    >jobs

 

    > queue start

 

    > jobs

 

ctrl+z 后台运行

 

退出

 

    ctrl+d

 

    $lftp

 

    >help lftp

 

    可以查看lftp的更多命令，其中尤其是put, mput, 和get，mget完全对应。

 

例如

 

上传单个html文件到服务器一级目录下

 

    >put localdir/index.html

 

上传多个html文件到服务器一级目录下

 

    >mput localdir/\*.html

 

10．

 

远程文件目录操作

 

    cat[-b]:滚屏显示文件的内容。

 

    more:分屏显示文件的内容。

 

    zcat:滚屏显示.gz文件的内容。

 

    zmore:分屏显示.gz文件的内容。

 

    mv:文件改名。

 

    rm[-r][-f]:删除文件。

 

    mrm:删除文件（可用通配符）。

 

    du[opts]:显示整个目录的容量。

 

    find[directory]:递归显示指定目录的所有文件（用于ls–R失效时）。

 

11．上传和下载

 

    get[opts][-o]:下载文件，可以改名后存储在本地。

 

    mget[opts]:下载多个文件。

 

    pget[opts][-o]:多线程下载。

 

    regetrfile[-olfile]:下载续传。

 

    put[opts][-o]:上传文件，可以改名后存储在远程。

 

    mput[opts]:上传多个文件。

 

    reputlfile[-orfile]:上传续传。

 

12．连接会话和队列管理

 

    scache[]:显示所有连接会话或切换至指定的连接会话。

 

    queue[opts][]:将命令置于队列等待执行。

 

    jobs[-v]:显示后台执行的作业。

 

    wait[]\|all:将后台进程换到前台执行（fg是wait的别名）。

 

    killall\|:删除后台作业。

 

    exitbg:退出lftp后，所有任务移至后台继续执行。

 

13．Cache管理

 

    cache[SUBCMD]:管理lftp的cache。

 

    [re]ls[args]:读取cache显示远程文件列表，rels不读取cache。

 

    [re]cls[opts][path/][pattern]:cls提供了比ls更丰富的列表功能。

 

    [re]nlist[]:nlist只显示文件名（没有颜色区分）。

 

14．站点镜像

 

    mirror[opts][remote[local]]:用于实现站点镜像。

 

15．书签管理

 

    bookmark[SUBCMD]:用于管理书签。

 

16．环境参数设置

 

    set[opts][[]]:用户设置lftp的环境参数。

17.    shell功能

    shell:shell 是 ! 的一个内建别名

用法: !<shell-command>

启动 shell 或 shell 命令

当然，也可以直接输入shell或!，这样就可以进入当前shell中，便于执行命令。

 

18.BitTorrent(bt)下载

用法：   torrent torrent-file [-O directory]

       Start BitTorrent process for the given torrent-file,  which  can
 be  a local  file  or URL. Existing files are first validated. Missing
pieces are downloaded. Files are stored  in  specified  directory  or
 current working directory by default. Seeding continues until ratio
reachs  torrent:stop-on-ratio setting or time of torrent:seed-max-time
outs.

 

可以用set torrent 来设置BT的上传和下载速率等。

 

提示: lftp的功能

 

支持ftp、ftps、http、https、sftp,file(可以当作文件管理器哦),fish,BitTorrent等传输协议；

 

支持FXP（在两个FTP服务器之间传输文件）；

 

支持代理；

 

支持多线程传输；

 

支持传输队列（queue）；

 

支持书签；

 

类似bash，它提供后台命令、nohop模式、命令历史、命令别名、命令补齐

和作业控制支持。

 

支持镜像（mirror）；

 

lftp还有许许多多的功能，可以通过man lftp来了解。

 
