在debian上测试最新进入主线内核的容器虚拟化-lxc
##########################################################################################################################################
:date: 2010-11-18 12:25:06
:category: 系统运维
:tags: lxc, debian, 虚拟化, 虚拟机
:slug: debian_lxc

**介绍** 
Linux conatiners (LXC) 是在Linux平台上基于容器的虚拟化技术的未来标准，它和传统的解决方案如Linux-VServer和OpenVZ有所区别。最初的LXC技术是由IBM研发的，目前已经进入Linux内核主线，这意味着LXC技术将是目前最有竞争力的轻量级虚拟容器技术，相比较传统的VServer和OpenVZ轻量级虚拟技术（两者都需要对标准内核进行补丁），发展潜力更大。

详细内容请看[[http://huatai.me/?p=573\|阿泰的lxc介绍]] 
deb http://mirrors.sohu.com/debian/ lenny main non-free contrib
以下是openvz guest to lxc 的转换脚本:

::

  #!/bin/bash 
  
  # 
  # lxc: linux Container library 
  
  # Authors: 
  # zhangmingyuan240@gmail.com 
  
  # This library is free software; you can redistribute it and/or 
  # modify it under the terms of the GNU Lesser General Public 
  # License as published by the Free Software Foundation; either 
  # version 2.1 of the License, or (at your option) any later version.
 
  
  # This library is distributed in the hope that it will be useful, 
  # but WITHOUT ANY WARRANTY; without even the implied warranty of 
  # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU 
  # Lesser General Public License for more details. 
  
  # You should have received a copy of the GNU Lesser General Public 
  # License along with this library; if not, write to the Free Software
 
  # Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307 USA 
  
  configure\_debian() 
  { 
  rootfs=$1 
  hostname=$2 
  
  #mv to lxc 
  path=\`echo $path\|sed 's#/$##g'\` 
  mv $path $path-rootfs 
  mkdir -p $path 
  mv $path-rootfs $path/rootfs 
  # configure the inittab 
  cat < $rootfs/etc/inittab 
  id:3:initdefault: 
  si::sysinit:/etc/init.d/rcS 
  l0:0:wait:/etc/init.d/rc 0 
  l1:1:wait:/etc/init.d/rc 1 
  l2:2:wait:/etc/init.d/rc 2 
  l3:3:wait:/etc/init.d/rc 3 
  l4:4:wait:/etc/init.d/rc 4 
  l5:5:wait:/etc/init.d/rc 5 
  l6:6:wait:/etc/init.d/rc 6 
  # Normally not reached, but fallthrough in case of emergency. 
  z6:6:respawn:/sbin/sulogin 
  1:2345:respawn:/sbin/getty 38400 console 
  c1:12345:respawn:/sbin/getty 38400 tty1 linux 
  c2:12345:respawn:/sbin/getty 38400 tty2 linux 
  c3:12345:respawn:/sbin/getty 38400 tty3 linux 
  c4:12345:respawn:/sbin/getty 38400 tty4 linux 
  EOF 
  
  # disable selinux in debian 
  mkdir -p $rootfs/selinux 
  echo 0 > $rootfs/selinux/enforce 
  
  # configure the network using the dhcp 
  cat < $rootfs/etc/network/interfaces 
  auto lo 
  iface lo inet loopback 
  EOF 
  
  # set the hostname 
  cat < $rootfs/etc/hostname 
  $hostname 
  EOF 
  
  # remove pointless services in a container 
  chroot $rootfs /usr/sbin/update-rc.d -f umountfs remove 
  chroot $rootfs /usr/sbin/update-rc.d -f hwclock.sh remove 
  chroot $rootfs /usr/sbin/update-rc.d -f hwclockfirst.sh remove 
  
  echo "root:root" \| chroot $rootfs chpasswd 
  echo "Root password is 'root', please change !" 
  
  return 0 
  } 
  
  copy\_configuration() 
  { 
  path=$1 
  rootfs=$2 
  name=$3 
  
  cat <> $path/config 
  lxc.tty = 4 
  lxc.pts = 1024 
  lxc.rootfs = $rootfs 
  lxc.cgroup.devices.deny = a 
  # /dev/null and zero 
  lxc.cgroup.devices.allow = c 1:3 rwm 
  lxc.cgroup.devices.allow = c 1:5 rwm 
  # consoles 
  lxc.cgroup.devices.allow = c 5:1 rwm 
  lxc.cgroup.devices.allow = c 5:0 rwm 
  lxc.cgroup.devices.allow = c 4:0 rwm 
  lxc.cgroup.devices.allow = c 4:1 rwm 
  # /dev/{,u}random 
  lxc.cgroup.devices.allow = c 1:9 rwm 
  lxc.cgroup.devices.allow = c 1:8 rwm 
  lxc.cgroup.devices.allow = c 136:\* rwm 
  lxc.cgroup.devices.allow = c 5:2 rwm 
  # rtc 
  lxc.cgroup.devices.allow = c 254:0 rwm 
  
  # mounts point 
  lxc.mount.entry=proc $rootfs/proc proc nodev,noexec,nosuid 0 0 
  lxc.mount.entry=devpts $rootfs/dev/pts devpts defaults 0 0 
  lxc.mount.entry=sysfs $rootfs/sys sysfs defaults 0 0 
  EOF 
  
  if [ $? -ne 0 ]; then 
  echo "Failed to add configuration" 
  return 1 
  fi 
  
  return 0 
  } 
  
  usage() 
  { 
  cat < $1 -h\|--help -p\|--path= --clean 
  EOF 
  return 0 
  } 
  
  options=$(getopt -o hp:n:c -l help,path:,name:,clean -- "$@") 
  if [ $? -ne 0 ]; then 
  usage $(basename $0) 
  exit 1 
  fi 
  eval set -- "$options" 
  
  while true 
  do 
  case "$1" in 
  -h\|--help) usage $0 && exit 0;; 
  -p\|--path) path=$2; shift 2;; 
  -n\|--name) name=$2; shift 2;; 
  -c\|--clean) clean=$2; shift 2;; 
  --) shift 1; break ;; 
  \*) break ;; 
  esac 
  done 
  
  if [ ! -z "$clean" -a -z "$path" ]; then 
  clean \|\| exit 1 
  exit 0 
  fi 
  
  if [ -z "$path" ]; then 
  echo "'path' parameter is required" 
  exit 1 
  fi 
  
  if [ "$(id -u)" != "0" ]; then 
  echo "This script should be run as 'root'" 
  exit 1 
  fi 
  
  rootfs=$path/rootfs 
  
  configure\_debian $rootfs $name 
  if [ $? -ne 0 ]; then 
  echo "failed to configure debian for a container" 
  exit 1 
  fi 
  
  copy\_configuration $path $rootfs 
  if [ $? -ne 0 ]; then 
  echo "failed write configuration file" 
  exit 1 
  fi 
  
  if [ ! -z $clean ]; then 
  clean \|\| exit 1 
  exit 0 
  fi 
