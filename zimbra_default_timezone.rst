zimbra小技巧之修改默认时区
##########################################################################################################################################
:date: 2011-04-06 09:54:30
:category: 系统运维
:tags: zimbra, 邮件服务器, server
:slug: zimbra_default_timezone

获取某用户的时区

::

 prompt> zmprov ga foo@company.com \| grep -i timezone

更改为亚洲/北京/上海/香港的时区

::

 prompt> zmprov mc default zimbraPrefTimeZoneId 'Asia/Hong\_Kong'

