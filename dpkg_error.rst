一次处理dpkg错误的记录
##########################################################################################################################################
:date: 2011-03-18 06:17:06
:category: 系统运维
:slug: dpkg_error

起因：突然断电，丢失数据 

::

 软件包“reportbug”的文件名列表文件最后结尾的换行符 
 不能使用dpkg，解决办法：
 rm /var/lib/dpkg/info/reportbug\* 
 sudo aptitude reinstall reportbug 
 即可 
 在用apt-get的时候还有很多类似于 
 dpkg：警告：无法找到软件包 libgc1c2 的文件名列表文件，现假定该软件包目前没有任何文件被安装在系统里。
 这样的错误，全部reinstall 
 aptitude reinstall libncurses5 2>&1 \|grep 现假定\|cut -d" " -f2\|xargs aptitude reinstall 
 startx失败，桌面进不去，将所有x相关的包都重装 
 dpkg -l \|cut -d" " -f3\|grep xserver\|egrep -v "nxserver\|xserver-xorg-video-radeonhd\|xserver-xorg-video-v4l"\|xargs aptitude reinstall -y 
 最后发现是udev的原因，aptitude reinstall udev重启，即可。
