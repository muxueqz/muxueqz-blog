Debian启动SNMPv3+监控宝
##########################################################################################################################################
:date: 2010-04-16 17:20:00
:category: 系统运维
:slug: Debian监控宝

=安装snmpd=

sudo apt-get install snmpd

=配置snmpd=

停止snmpd服务: /etc/init.d/snmpd stop

创建只读的snmpv3用户: net-snmp-config --create-snmpv3-user -ro -A
$PASSWORD -X privpassword $USERNAME
#把USERNAME换成你要创建的用户名，$PASSWORD换成你要使用的密码。

修改非127.0.0.1访问

用VIM或是其它编辑器打开 /etc/default/snmpd文件，修改

SNMPDOPTS='-lSD -lF /dev/null -u snmp -I -smux -p /var/run/snmp.pid
127.0.0.1'

删除 127.0.0.1，之后-->

SNMPDOPTS='-lSD -lF /dev/null -u snmp -I -smux -p /var/run/snmp.pid'

OK，如果开了iptables，请：

iptables -A INPUT -i eth0 -p udp -s 125.76.229.215 --dport 161 -j ACCEPT

iptables -A INPUT -i eth0 -p udp -s 60.195.249.83 --dport 161 -j ACCEPT

有关在监控宝中添加服务器监控，请到：

http://blog.jiankongbao.com/?p=160 《在Linux服务器上开启安全的SNMP代理》

 
