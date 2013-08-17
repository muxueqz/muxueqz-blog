记RH无能之SCSI驱动
##########################################################################################################################################
:date: 2010-01-08 02:15:54
:category: 系统运维
:slug: rh

昨天装了一台CentOS 4.7，用KVM /dev/sda
这样安装的，安装后无法启动，找不到硬盘。(Debian正常，并在Debian下获得硬盘模块名)

按以往的经验，更新initrd就可以，所以我:

::

    mkinitrd -f /boot/initrd-\`uname -r\`.img \`uname -r\`

 一般这样就OK了，但这次的aic7xxx更麻烦……
 最后下面这个搞定了：
  
:: 

     mkinitrd -f /boot/initrd-\`uname -r\`.img \`uname -r\` --with aic7xxx 

  

猜想：RH系这种机制的内核引导会比Debian快些，initrd的大小也比Debian小很多，但可惜RH系的init机制太慢，反而不如Debian启动快。
