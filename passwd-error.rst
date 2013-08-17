passwd :Authentication token lock busy的解决方法
##########################################################################################################################################
:date: 2009-09-10 11:49:37
:category: 系统运维
:slug: passwd-error

呵呵，今天IRC中有个朋友遇到的问题比较有趣，普通用户不能修改密码，错误信息为：
  

    passwd :Authentication token lock busy

 仔细看了一下，是/usr/bin/passwd的权限不对，以下是错误时的权限：
  

    [root@Eos ~]# ls -alh /usr/bin/passwd
     -rwxr-xr-x 1 root root 25K 2009-09-05 16:13 /usr/bin/passwd

 以下是正确的权限：

    [root@Eos ~]# ls -alh /usr/bin/passwd
     -rws--x--x 1 root root 25K 2009-09-05 16:13 /usr/bin/passwd

 修改权限为以下：

    chmod -v 4711 /usr/bin/passwd
