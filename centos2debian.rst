远程将CentOS改装成Debian
##########################################################################################################################################
:date: 2010-04-16 17:20:09
:category: 系统运维
:slug: centos2debian

 

=环境需求=

 

因一些特殊情况，需要远程将CentOS改装成Debian

当前OS:CentOS 5.4 64bit (本文内容稍作修改也适用于其它发行版)

目标OS:Debian 5.0 (lenny) amd64

文中我是以一块新硬盘举例的，实际可以不用加硬盘，只要有2G左右的空闲分区即可。

 

=debootstrap安装基本系统=

创建目标目录

    sudo mkdir /mnt/target

    sudo fdisk /dev/sdb # 给目标磁盘分区

    sudo mkfs.ext3 /dev/sdb1 #格式化目标磁盘

    sudo mount /dev/sdb1 /mnt/target #挂载目标分区

 

安装debootstrap工具:

    wget
    http://ftp.de.debian.org/debian/pool/main/d/debootstrap/debootstrap\_1.0.10lenny1.tar.gz

    #如果下载链接过期，请去debootstrap主页找新版本:http://packages.debian.org/source/lenny/debootstrap

    tar xvf debootstrap\*.tar.gz -C /tmp/

    cd /tmp/debootstrap

    make install

 

开始安装基本系统:

    sudo ./debootstrap --arch amd64 lenny /mnt/target
    http://mirrors.163.com/debian/
    #从163的源安装一个amd64架构(即64位)的Debian
    5，根据网络情况，要等一段时间

    sudo chroot /mnt/target /bin/bash #Chroot到目标系统

 

修改root密码:

    passwd root

创建 /etc/fstab #根据你的实际情况来修改: 

         # file system   mount point     type    options                
    dump    pass

         /dev/hda1       /               ext3    defaults              
     0       0

         /dev/hda5       none            swap    sw                    
     0       0

         proc            /proc           proc    defaults              
     0       0

         sys             /sys            sysfs   defaults              
     0       0

挂载所有分区: 

         mount -a

         ls /proc # 检查信息是否正确

         mkswap /dev/hda5 #将/dev/hda5格式化成swap

配置键盘:

         dpkg-reconfigure console-setup

配置网络:

         editor /etc/network/interfaces

配置主机名:

         echo debian > /etc/hostname

添加一个普通用户: 

         adduser foo

         echo 'foo ALL=(ALL) ALL' >> /etc/sudoers

         chmod 0440 /etc/sudoers

    passwd foo #修改foo用户的密码

设置/etc/apt/sources.list(Apt软件源),/etc/hosts,/etc/resolv.conf(DNS服务器地址),/etc/network/interfaces(IP配置)

    echo "127.0.0.1 localhost debian" > /etc/hosts

 

安装amd64内核与grub引导器、openssh-server

         apt-get install linux-image-amd64 grub openssh-server

         mkdir -p /boot/grub

         cp /usr/lib/grub/i386-pc/\* /boot/grub

         editor /boot/grub/menu.lst 

         exit # exit the chroot(), that is

将grub引导记录安装到目标磁盘

         sudo grub-install --no-floppy --root-directory=/mnt/target
    /dev/sdb

如不成功，可以进入到grub shell里安装。

    grub

    root (hd1,0)

    setup (hd1)

 

修改CentOS当前grub引导优先级:

    editor /boot/grub/menu.lst #添加Debian并设置为最高

OK!可以重启了。重启之前一定要检查好目标磁盘的/etc/fstab和/boot/grub/menu.lst,以及网络配置等，以免重启后连接不上。

如有疑问请在我博客上找我: http://Linuxzh.org

 
