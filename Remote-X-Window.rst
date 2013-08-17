远程X-Window
##########################################################################################################################################
:date: 2009-06-15 12:15:44
:category: 系统运维
:slug: Remote-X-Window

X-Window本身支持远程连接，所以简单配置一下便可远程连接上了
 注：X-Window远程连接太慢，如需连接图形界面的Linux，建议使用nxserver!
 必须先开启X，下面是查看X的ID

     ps aux \|grep X

 设置DISPLAY

     export DISPLAY=’:0′

 开启远程服务，+是允许所有连接

     xhost +

 GDM远程
 XDMCP即“X Display Manager Control Protocol”，是一种网络协议。由它来建立图形客户端程序与X Window服务器的连接与通信。
 XDM即“X Display Manager”，由它来启动X Window服务器，并管理图形客户端程序的登录、会话、启动窗口管理器（KDE、Gnome）等。KDE和Gnome也提供了自己的xdm的实现，分别叫kdm和gdm。
 1.gdm具体配置方法

:: 

     [root@mis-redhat ~]# cd /etc/gdm/
     [root@mis-redhat gdm]# ls
     custom.conf locale.alias PostLogin PreSession XKeepsCrashing
     Init modules PostSession securitytokens.conf Xsession
     [root@mis-redhat gdm]#cp -p custom.conf gdm.conf


其中custom.conf为默认配置，可将此文件复制为gdm.conf，又或可直接在此文件中修改配置，均可生效。

::

     [root@mis-redhat gdm]#vim gdm.conf
     [xdmcp]
     Enable=1
     [root@mis-redhat gdm]# gdm-restart

 检查177端口是否有开启

::

     [root@mis-redhat gdm]# netstat -ntpul \| grep 177
     udp 0 0 0.0.0.0:177 0.0.0.0:\* 6056/gdm-binary

 简单的配置后，即可使用xmanager连接X了。。。
 如果有开启防火墙的话，需要开启此端口

     [root@mis-redhat gdm]# iptables -A INPUT -p udp -s 0.0.0.0/0 -d 0.0.0.0/0 –dport 177 -j ACCEPT


gdm默认是不允许root用户登录的，如果需要启用允许root用户登录，则可在gdm.conf中如此修改

::

     [security]
     AllowRemoteRoot=true
     #如需启用gdm的日志功能
     [debug]
     Enable=true

 所有对gdm.conf配置文件修改后都需记得重启gdm

     [root@mis-redhat gdm]# gdm-restart

 如果想关闭gdm,也很简单

     [root@mis-redhat gdm]# gdm-stop

 另外，gdm配置方法除了修改配置文件外，还可用图形界面进行设置

     [root@mis-redhat gdm]# gdmsetup

 2.配置xfs
 xfs的配置文件是/etc/X11/fs/config，内容如下：

::

     #
     # Default font server configuration file for Mandrake Linux workstation
     #
     # allow a max of 10 clients to connect to this font server
     client-limit = 10
     # when a font server reaches its limit, start up a new one
     clone-self = off
     # alternate font servers for clients to use
     #alternate-servers = foo:7101,bar:7102
     # where to look for fonts
     # Some of these are commented out, i.e. the TrueType and Type1
     # directories in /usr/share, because they arent forced to be
     # installed alongside X.
     #
     catalogue = /usr/X11R6/lib/X11/fonts/misc:unscaled,
     /usr/X11R6/lib/X11/fonts/75dpi:unscaled,
     /usr/X11R6/lib/X11/fonts/100dpi:unscaled,
     /usr/X11R6/lib/X11/fonts/misc:unscaled,
     /usr/X11R6/lib/X11/fonts/Type1,
     /usr/X11R6/lib/X11/fonts/Speedo,
     /usr/X11R6/lib/X11/fonts/mdk:unscaled,
     /usr/share/fonts/default/Type1,
     /usr/share/fonts/ttf/big5,
     /usr/share/fonts/ttf/gb2312,
     /usr/share/fonts/ttf/decoratives,
     /usr/share/fonts/ttf/western
     # in 12 points, decipoints
     default-point-size = 120
     # 100 x 100 and 75 x 75
     default-resolutions = 75,75,100,100
     # how to log errors
     use-syslog = on
     # For security, don’t listen to TCP ports by default.
     no-listen = tcp


在配置文件中可以定义最大客户端连接数量，这里缺省是10。配置文件中也指明了字体文件的位置，特别注意包含了中文字体，否则在客户端无法正确显示中文字体。
 其中需将#no-listen = tcp的注释去掉，启用tcp监听，其默认端口为7100
 使用如下命令来重启xfs：

::

     service xfs stop
     service xfs start

 xfs启动成功后，可以使用netstat -ln命令来确认7100端口已绑定：

     tcp 0 0 0.0.0.0:7100 0.0.0.0:\* LISTEN
