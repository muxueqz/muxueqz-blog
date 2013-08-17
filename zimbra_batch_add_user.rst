zimbra小技巧之批量添加用户
##########################################################################################################################################
:date: 2011-04-06 10:03:40
:category: 系统运维
:tags: zimbra, 邮件服务器, server
:slug: zimbra_batch_add_user

::

 NAMELIST=`cat userlist.txt`
 DOMAIN=email.com
 PASSWD=111111
 ZMPROV=/opt/zimbra/bin/zmprov
 for NAME in $NAMELIST
 do
 $ZMPROV ca $NAME@$DOMAIN $PASSWD
 done

userlist.txt格式如下：

::

 name1
 name2

