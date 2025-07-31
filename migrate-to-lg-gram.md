Title: 迁移到新笔记本——LG gram
Date: 2019-08-09 22:20
Tags: DevOps, 系统运维
Slug: migrate-to-lg-gram
Author: muxueqz

# 前言
* 更换了一台新笔记本，995g，续航22小时
* 在Windows上硬盘安装Debian
  * 试过[unetbootin](http://unetbootin.sourceforge.net)
  * [YUMI - Multiboot USB Creator](https://www.pendrivelinux.com/yumi-multiboot-usb-creator/)
最终，通过:
* grub4win
* 打包Linux: tar zcvf /media/muxueqz/usb_data/x360-sys-bak.tar.gz !(dev|proc|data|sys|lost+found|media|mnt|tmp)
* 完成迁移

# 参考
[联想笔记本windows10使用uefi启动引导ntfs硬盘上的iso光盘安装ubuntu双系统](https://gmd20.github.io/blog/%E8%81%94%E6%83%B3%E7%AC%94%E8%AE%B0%E6%9C%ACwindows10%E4%BD%BF%E7%94%A8UEFI%E5%90%AF%E5%8A%A8%E5%BC%95%E5%AF%BCNTFS%E7%A1%AC%E7%9B%98%E4%B8%8A%E7%9A%84ISO%E5%85%89%E7%9B%98%E5%AE%89%E8%A3%85Ubuntu%E5%8F%8C%E7%B3%BB%E7%BB%9F/)
