Cisco模拟器for Linux 1.1
##########################################################################################################################################
:date: 2009-11-19 14:26:15
:category: 系统运维
:slug: ciscotest

呵呵，其实这是我以前学习Cisco时的小作品了，现在发布，愿能方便大家。

基于Dynamips及Dynagen，pemu制作，包含Cisco Router
3660，3620+交换模块，PIX。除非是Cisco内部模拟器，我想不会有比这个更好的了。

下载地址：`http://download.linuxzh.org/xyj/Downloads/CiscoTestForLinux-1.1.tar.bz2`

以下是使用说明：

将CiscoTestForUnix-Center.tar.bz2解压。
注意：！进入解压目录方可运行，拓扑图在top目录中
先运
行./nowindowVS.sh，然后再另开一个终端运行./General3660.sh，输入list查看拓扑中的设备和对应的端口，接着输入
start R1，这样就启动了第一台路由器，然后在其它终端中用telnet 连接
127.0.0.1 的3001端口（注：list可以查看到R1对应的端口是3001），如telnet
127.0.0.1 3001就可以连上R1了。
名称规范：R1 ，R即Router，路由器，R1即第一台路由器，依次类推，R2、R3等等。
                   
sw1,sw即Switch，交换机，sw1即第一台交换机，依次类推，sw2、sw3等等（注：无法直接模拟交换机，这里的交换机只是加了交换模块的路由器而已）

如需桥接网卡，则修改/Cisco/net/6-3660forLinux.net,将NIO\_linux\_eth:之后修改为自己
要桥接的网卡，然后运行./控制台桥接网卡版.sh，获取网卡参数使用/Ciscotest/bin/dynamips
-e命令
获取idlepc值的命令 /Ciscotest/bin/dynamips -P 3600 -t 3620
/root/dynamips/xxx.bin
PIX模拟器支持，运行方法：
先运行./PIXVS.sh
然后运行./nowindowVS.sh
最后运行./PIXControl.sh

进入模拟器控制台后
list 查看设备

start 设备名，如start R1，即开启R1

拓扑图：

 

.. figure:: http://docs.google.com/File?id=dzb2h88_123ch24c4dz_b
   :align: center
   :alt: 
