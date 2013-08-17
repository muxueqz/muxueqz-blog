ArchLinux休眠快速启动
##########################################################################################################################################
:date: 2011-05-08 05:26:28
:category: 系统运维
:slug: arch_uswsusp

休眠的特点：
 - 状态保存到硬盘
 - 启动快
 - 可以立刻恢复到工作状态
睡眠的特点：
 - 状态保存到内存
 - 对内存供电
 - 省电

AUR安装uswsusp

::

 wget http://aur.archlinux.org/packages/uswsusp-git/uswsusp-git.tar.gz
 tar xvf uswsusp-git.tar.gz
 cd uswsusp-git
 sudo makepkg --asroot
 pacman -U uswsusp-git-\*-i686.pkg.tar\*
 #这时就可以s2ram来睡眠(保存在内存中，只对内存供电)了
 #修改/etc/mkinitcpio.conf
 #在HOOKS中添加uresume，如下：
 HOOKS="base udev autodetect pata scsi sata filesystems uresume" }}}
 #更新initcpio
 sudo mkinitcpio -p kernel26
 #这时就可以s2disk来休眠(保存在swap中，不供电)了
