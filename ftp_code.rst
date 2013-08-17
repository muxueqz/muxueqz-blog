关于FTP编码
##########################################################################################################################################
:date: 2010-03-16 04:33:07
:category: 系统运维
:slug: ftp_code

Linux上的FTP Server中文编码问题历来有些……(即Windows
explorer资源管理器访问时中文乱码)

解决的办法有很多，最好的当然原生gb2312编码，次之可以用支持utf8编码的FTP客户端，如FileZilla，又次之可以用pure-ftpd
--enable-rfc2640来支持转换编码，这里有个更中用的方法……相比于pure-ftpd转换编码，更建议使用基于fuse的convmvfs转换FileSystem的编码，我测试过，pure-ftpd转换编码后的传输性能大大折扣，10MB/s转换之后只有3MB/s，而convmvfs转换后仍能达到7MB/s以上，缺点是CPU占用较高，偶尔用用还是不错的。

convmvfs项目：\ `http://fuse-convmvfs.sourceforge.net/ `_

转换示例：

    convmvfs /data/FTP/ -o
    srcdir=/data/share/,icharset=utf8,ocharset=gbk,allow\_other

srcdir=源目录

/data/FTP=目标目录

icharset=源编码

ocharset=目标编码

allow\_other=允许其它用户

.. _`http://fuse-convmvfs.sourceforge.net/ `: http://fuse-convmvfs.sourceforge.net/
