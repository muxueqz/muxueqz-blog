mysql运维常用命令
##########################################################################################################################################
:date: 2010-12-28 08:11:51
:category: 系统运维
:slug: mysql_sa_commands

使用兼容mysql323的方式备份 

::

 mysqldump ,--compatible=mysql323 
 Mysql表引擎更改为InnoDB 
 alter table $TABLENAME type=InnoDB 
 链接数据库 
 mysql -h localhost -u username -p password; 
 备份整个库 
 mysqldump -u $USER -p $DBNAME> $FILENAME 
 备份单个表或几个表 
 mysqldump --opt --add-drop-table -u $USER -p $DBNAME $TABLE1 $TABLE2> $FILENAME 
 恢复数据库 
 shell> mysqladmin -h $HOST -u $USER -p create $DBNAME 
 shell> mysql -h $HOST -u $USER -p $DBNAME <$FILENAME 
 显示所有数据库 
 show databases;
 选定数据库 
 use database\_name ; 
 查看所有表 
 show tables; 
 查看所有表状态(引擎等) 
 show tables status; 
 查看表中内容
 select \* from table\_name; 
 查看用户信息 
 SELECT host, user FROM mysql.user where user='用户名'; 
 查看所有用户 
 select \* from mysql.user 
 查看用户权限 
 show grants for '$USER'@'localhost'; 
 添加用户(无再授权的权限) 
 GRANT ALL PRIVILEGES ON $DBNAME.\* TO $USER@$IP IDENTIFIED BY '$PASSWORD'; 
 添加用户(有再授权的权限) 
 （如 GRANT ALL PRIVILEGES ON \*.\* TO user@IP IDENTIFIED BY 'password' WITH GRANT OPTION; ） 
 GRANT ALL PRIVILEGES ON $DBNAME.\* TO $USER@$IP IDENTIFIED BY '$PASSWORD' WITH GRANT OPTION; 
 （如 GRANT ALL PRIVILEGES ON \*.\* TO user@IP IDENTIFIED BY 'password' WITH GRANT OPTION; ） 
 SQL常用语句：
 查询 
 SELECT column1,columns2,...FROM table\_name WHERE conditions 
 插入 
 INSERT INTO table\_name (column1,column2,...) VALUES ( value1,value2, ...) 
 更新 
 UPDATE table\_name SET column1='xxx' WHERE conditoins 
 删除 
 DELETE FROM table\_name WHERE conditions 
 添加主键 
 create unique index groupId on TaskGroup(groupId) ; 
 替换，当表中有唯一主键的时候replace=delect+insert 
 replace into Goods(id,type,name,price,description,discarded,deal,sell,maxoverlapm,level,usetype,bindtype,usetime,cdTime,locationtype,url,impacttype,putact,parabola,hittype,stricken,confine,movetype,validpoints,spreadpoints,sellPrice) value('110040008','11004','黄金眼镜',520,'采用黄金制作而成的眼镜，只有贵族的人才能拥有',1,1,1,1,0,'1',0,0,0,4,'res/item/11/004/110040008.swf',0,0,0,0,0,0,0,0,0,130);

 设置变量，不需重启即可生效配置。
 set 可以使用几种语法形式来设置或检索全局或会话变量。下面的例子使用了sort\_buffer\_sizeas作为示例变量名。
 要想设置一个GLOBAL变量的值，使用下面的语法：
 mysql> SET GLOBAL sort\_buffer\_size=value; 
 mysql> SET @@global.sort\_buffer\_size=value; 
 要想设置一个SESSION变量的值，使用下面的语法：
 mysql> SET SESSION sort\_buffer\_size=value; 
 mysql> SET @@session.sort\_buffer\_size=value; 
 mysql> SET sort\_buffer\_size=value; 
 LOCAL是SESSION的同义词。

如果设置变量时不指定GLOBAL、SESSION或者LOCAL，默认使用SESSION。参见13.5.3节，“SET语法”。
 要想检索一个GLOBAL变量的值，使用下面的语法：

::

 mysql> SELECT @@global.sort\_buffer\_size; 
 mysql> SHOW GLOBAL VARIABLES LIKE 'sort\_buffer\_size'; 
 要想检索一个SESSION变量的值，使用下面的语法：
 mysql> SELECT @@sort\_buffer\_size; 
 mysql> SELECT @@session.sort\_buffer\_size; 
 mysql> SHOW SESSION VARIABLES LIKE 'sort\_buffer\_size'; 
 这里，LOCAL也是SESSION的同义词。
 当你用SELECT @@var\_name搜索一个变量时(也就是说，不指定global.、session.或者local.)，MySQL返回SESSION值（如果存在），否则返回GLOBAL值。
 对于SHOW VARIABLES，如果不指定GLOBAL、SESSION或者LOCAL，MySQL返回SESSION值。

当设置GLOBAL变量需要GLOBAL关键字但检索时不需要它们的原因是防止将来出现问题。如果我们移除一个与某个GLOBAL变量具有相同名字的SESSION变量，具有SUPER权限的客户可能会意外地更改GLOBAL变量而不是它自己的连接的SESSION变量。如果我们添加一个与某个GLOBAL变量具有相同名字的SESSION变量，想更改GLOBAL变量的客户可能会发现只有自己的SESSION变量被更改了。

 mysql详细内容可查询mysql5.1参考手册 
 网址：[[http://dev.mysql.com/doc/refman/5.1/zh/index.html]]
