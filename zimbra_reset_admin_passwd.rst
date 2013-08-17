zimbra小技巧之重设管理员密码
##########################################################################################################################################
:date: 2011-04-06 06:13:58
:category: 系统运维
:tags: zimbra, 邮件服务器, server
:slug: zimbra_reset_admin_passwd

su - zimbra
 zmprov sp $admin@email\_address $new\_password
 
 当然，此方法也可以重设普通用户的密码
