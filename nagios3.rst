Debian 安装 nagios3
##########################################################################################################################################
:date: 2009-06-15 12:01:30
:category: 系统运维
:slug: nagios3

nagios是一种开源服务器监控软件，能在某个服务当掉时发邮件或短信给你哦！

::

 sudo apt-get update #更新软件列表
 sudo apt-get install nagios3 #安装nagios3
 这时如果登陆http://127.0.0.1/nagios3 会提示输入帐号密码
 回到Debian，创建nagios登陆用户
 sudo htpasswd -c /etc/nagios3/htpasswd.users username

#username为你的用户名，然后会让你设置密码
 可以用这个用户登陆http://127.0.0.1/nagios3 了，但无法显示服务器运行信息，需要添加cgi权限

::

 #先备份cgi.cfg文件
 cp /etc/nagios3/cgi.cfg /etc/nagios3/cgi.cfg-bak
 #编辑/etc/nagios3/cgi.cfg
 sed ’s/=nagiosadmin/=username/’ /etc/nagios3/cgi.cfg-bak > /etc/nagios3/cgi.cfg
 #username为你在hwpass时创建的用户名，这句命令的意思是允许你的用户使用cgi权限
 最后，重启nagios3和apache服务

::

 /etc/init.d/nagios3 restart
 /etc/init.d/apache2 restart

好了，现在试试吧！
