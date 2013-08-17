typecho sqlite转换为mysql
##########################################################################################################################################
:date: 2010-03-10 00:54:58
:category: 系统运维
:slug: typecho_sqlite_to_mysql

一、
Firefox的附加组件SQLite Manager，把除了typecho\_options的表导出

    typecho\_comments.sql typecho\_relationships.sql
    typecho\_contents.sql typecho\_users.sql
    typecho\_metas.sql

二、
然后重新安装typecho源码，安装后用phpmyadmin清空各个表的内容，最后用phpmyadmin导入先前备份的五个sql文件即可
