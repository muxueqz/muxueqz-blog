OpenVZ桥接设置(翻译)
##########################################################################################################################################
:date: 2009-08-02 15:02:56
:category: 系统运维
:tags: openvz, 虚拟化, 虚拟机
:slug: openvz-bridge

本文介绍OpenVZ如何使用桥接，这可以用在OpenVZ与Virtualbox、VMware等并存的环境。
 关于OpenVZ的介绍：\ `http://zh.wikipedia.org/wiki/OpenVZ`_
 名词解释：
 HN：HardWare
 Node，硬件节点，在OpenVZ中指运行OpenVZ的主系统，即相当于VMware的Host System。
 使用空闲IP在同一网段
 创建一个备用接口和IP

::

     [HN] ifconfig eth0:1 \*.\*.\*.\*
     [HN] vzctl set 101 --nameserver \*.\*.\*.\*

先决条件
本文经过RHEL5的OpenVZ硬件节点和Fedora
Core5的模板测试，其他系统和模板可能需要一些配置修改。(我用的是Debian，已经测试本文可用)

本文假定存在'brctl','ip','ifconfig'工具，您可能需要安装缺失软件，如'bridge-utils','iproute','net-tools'或其他名字的包。
 本文假定您已经有安装OpenVZ的系统和已创建的模板。如果不是这样，请先创建。
 注：请不要使用重复IP。
 一个OpenVZ硬件节点惟一一个以太网接口。
 (假设为eth0)
 硬件节点配置
 创建一个桥

::

 [HN]# brctl addbr br0
 清除eth0接口的IP配置。
 [HN]# ifconfig eth0 0
 将eth0接口添加到桥中。
 [HN]# brctl addif br0 eth0
 指定桥接口的IP
 (应该与eth0先前的IP一样。)
 [HN]# ifconfig br0 10.0.0.2/24
 恢复默认路由
 [HN]# ip route add default via 10.0.0.1 dev br0

警告：如果您正在配置远程节点，请必须编写脚本执行上述命令，并在后台运行它和重定向输出到某个记录文件。否则您可能会与这个节点失去连接。
 脚本范例：

::

 [HN]# cat /tmp/br\_add
 #!/bin/bash
 brctl addbr br0
 ifconfig eth0 0
 brctl addif br0 eth0
 ifconfig br0 10.0.0.2/24
 ip route add default via 10.0.0.1 dev br0
 [HN]# /tmp/br\_add >/dev/null 2>&1 &
 VPS/VE（OpenVZ GuestOS）配置
 启动一个VPS
 [HN]# vzctl start 101
 添加一个eth0接口到101
 VPS中，同时会在OpenVZ的硬件节点中添加一个veth的接口。
 [HN]# vzctl set 101 --netif\_add eth0 --save
 设置eth0的IP与子网掩码。
 [HN]# vzctl exec 101 ifconfig eth0 85.86.87.195/26
 将veth接口添加到桥中。
 [HN]# brctl addif br0 veth101.0

注：有可能会延时约15秒（默认为2.6.18内核）(我运道差，每次都得几分钟，这也是为什么后来扔掉veth的原因。)，而桥运行STP（生成树）检测循环和过渡veth接口到转发状态。
 设置VPS的缺省路由为eth0

::

 [HN]# vzctl exec 101 ip route add default via 85.86.87.193 dev eth0
 (Optional) Add CT↔HN routes
 The above configuration provides the following connections:
 \* CT X ↔ CT Y (where CT X and CT Y can locate on any OVZ HN)
 \* CT ↔ Internet
 Note that
 \* The accessability of the CT from the HN depends on the local gateway providing NAT (probably - yes)
 \* The accessability of the HN from the CT depends on the ISP gateway being aware of the local network (probably not)
 So to provide CT ↔ HN accessibility despite the gateways' configuration you can add the following routes:
 [HN]# ip route add 85.86.87.195 dev br0
 [HN]# vzctl exec 101 ip route add 10.0.0.2 dev eth0
 Resulting OpenVZ Node configuration
 Resulting OpenVZ Node configuration
 Making the configuration persistent
 Set up a bridge on a HN
 This can be done by configuring the ifcfg-\* files located in /etc/sysconfig/network-scripts/.
 Assuming you had a configuration file (e.g. ifcfg-eth0) like:
 DEVICE=eth0
 ONBOOT=yes
 IPADDR=10.0.0.2
 NETMASK=255.255.255.0
 GATEWAY=10.0.0.1
 To automatically create bridge br0 you can create ifcfg-br0:
 DEVICE=br0
 TYPE=Bridge
 ONBOOT=yes
 IPADDR=10.0.0.2
 NETMASK=255.255.255.0
 GATEWAY=10.0.0.1
 and edit ifcfg-eth0 to add the eth0 interface into the bridge br0:
 DEVICE=eth0
 ONBOOT=yes
 BRIDGE=br0
 Edit the container's configuration
 Add these parameters to the /etc/vz/conf/$CTID.conf file which will be used during the network configuration:
 \* Add/change CONFIG\_CUSTOMIZED="yes" (indicates that a custom
 script should be run on a container start)
 \* Add VETH\_IP\_ADDRESS="IP/MASK" (a container can have multiple
 IPs separated by spaces)
 \* Add VE\_DEFAULT\_GATEWAY="CT DEFAULT GATEWAY"
 \* Add BRIDGEDEV="BRIDGE NAME" (a bridge name to which the
 container veth interface should be added)
 An example:
 # Network customization section
 CONFIG\_CUSTOMIZED="yes"
 VETH\_IP\_ADDRESS="85.86.87.195/26"
 VE\_DEFAULT\_GATEWAY="85.86.87.193"
 BRIDGEDEV="br0"
 Create a custom network configuration script
 which should be called each time a container is started (e.g.  /usr/sbin/vznetcfg.custom):
 #!/bin/bash
 # /usr/sbin/vznetcfg.custom
 # a script to bring up bridged network interfaces (veth's) in a container
 GLOBALCONFIGFILE=/etc/vz/vz.conf
 CTCONFIGFILE=/etc/vz/conf/$VEID.conf
 vzctl=/usr/sbin/vzctl
 brctl=/usr/sbin/brctl
 ip=/sbin/ip
 ifconfig=/sbin/ifconfig
 . $GLOBALCONFIGFILE
 . $CTCONFIGFILE
 NETIF\_OPTIONS=\`echo $NETIF \| sed 's/,/
 /g'\`
 for str in $NETIF\_OPTIONS; do
 # getting 'ifname' parameter value
 if echo "$str" \| grep -o "^ifname=" ; then
 # remove the parameter name from the string (along with '=')
 CTIFNAME=${str#\*=};
 fi
 # getting 'host\_ifname' parameter value
 if echo "$str" \| grep -o "^host\_ifname=" ; then
 # remove the parameter name from the string (along with '=')
 VZHOSTIF=${str#\*=};
 fi
 done
 if [ ! -n "$VETH\_IP\_ADDRESS" ]; then
 echo "According to $CONFIGFILE CT$VEID has no veth IPs configured."
 exit 1
 fi
 if [ ! -n "$VZHOSTIF" ]; then
 echo "According to $CONFIGFILE CT$VEID has no veth interface configured."
 exit 1
 fi
 if [ ! -n "$CTIFNAME" ]; then
 echo "Corrupted $CONFIGFILE: no 'ifname' defined for host\_ifname $VZHOSTIF."
 exit 1
 fi
 echo "Initializing interface $VZHOSTIF for CT$VEID."
 $ifconfig $VZHOSTIF 0
 CTROUTEDEV=$VZHOSTIF
 if [ -n "$BRIDGEDEV" ]; then
 echo "Adding interface $VZHOSTIF to the bridge $BRIDGEDEV."
 CTROUTEDEV=$BRIDGEDEV
 $brctl addif $BRIDGEDEV $VZHOSTIF
 fi
 # Up the interface $CTIFNAME link in CT$VEID
 $vzctl exec $VEID $ip link set $CTIFNAME up
 for IP in $VETH\_IP\_ADDRESS; do
 echo "Adding an IP $IP to the $CTIFNAME for CT$VEID."
 $vzctl exec $VEID $ip address add $IP dev $CTIFNAME
 # removing the netmask
 IP\_STRIP=${IP%%/\*};
 echo "Adding a route from CT0 to CT$VEID using $IP\_STRIP."
 $ip route add $IP\_STRIP dev $CTROUTEDEV
 done
 if [ -n "$CT0\_IP" ]; then
 echo "Adding a route from CT$VEID to CT0."
 $vzctl exec $VEID $ip route add $CT0\_IP dev $CTIFNAME
 fi
 if [ -n "$VE\_DEFAULT\_GATEWAY" ]; then
 echo "Setting $VE\_DEFAULT\_GATEWAY as a default gateway for CT$VEID."
 $vzctl exec $VEID
 $ip route add default via $VE\_DEFAULT\_GATEWAY dev $CTIFNAME
 fi
 exit 0
 Note: this script can be easily extended to work for multiple triples , see http://vireso.blogspot.com/2008/02/2-veth-with-2-brindges-on-openvz-at.html
 Make the script to be run on a container start
 In order to run above script on a container start create the file /etc/vz/vznet.conf with the following contents:
 EXTERNAL\_SCRIPT="/usr/sbin/vznetcfg.custom"
 Note: /usr/sbin/vznetcfg.custom should be executable (chmod +x /usr/sbin/vznetcfg.custom)
 Note: When CT is stoped there are HW → CT route(s) still present in route table. We can use On-umount script for solve this.
 Create On-umount script for remove HW → CT route(s)
 which should be called each time a container with VEID (/etc/vz/conf/$VEID.umount), or any container (/etc/vz/conf/vps.umount) is stop.
 #!/bin/bash
 # /etc/vz/conf/$VEID.umount or /etc/vz/conf/vps.umount
 # a script to remove routes to container with veth-bridge from bridge
 CTCONFIGFILE=/etc/vz/conf/$VEID.conf
 ip=/sbin/ip
 . $CTCONFIGFILE
 if [ ! -n "$VETH\_IP\_ADDRESS" ]; then
 exit 0
 fi
 if [ ! -n "$BRIDGEDEV" ]; then
 exit 0
 fi
 for IP in $VETH\_IP\_ADDRESS; do
 # removing the netmask
 IP\_STRIP=${IP%%/\*};
 echo "Remove a route from CT0 to CT$VEID using $IP\_STRIP."
 $ip route del $IP\_STRIP dev $BRIDGEDEV
 done
 exit 0
 Note: The script should be executable (chmod +x /etc/vz/conf/vps.umount)
 Setting the route CT → HN
 To set up a route from the CT to the HN, the custom script has to get a HN IP (the $CT0\_IP variable in the script). There are several ways to specify it:
 1. Add an entry CT0\_IP="CT0 IP" to the $VEID.conf
 2. Add an entry CT0\_IP="CT0 IP" to the /etc/vz/vz.conf (the global configuration config file)
 3. Implement some smart algorithm to determine the CT0 IP right in the custom network configuration script
 Each variant has its pros and cons, nevertheless for HN static IP configuration variant 2 seems to be acceptable (and the most simple).
 An OpenVZ Hardware Node has two Ethernet interfaces
 Assuming you have 2 interfaces eth0 and eth1 and want to separate local traffic (10.0.0.0/24) from external traffic. Let's assign eth0 for the external traffic and eth1 for the local one.
 If there is no need to make the container accessible from the HN and vice versa, it's enough to replace 'br0' with 'eth1' in the following steps of the above configuration:
 \* Hardware Node configuration → Assign the IP to the bridge
 \* Hardware Node configuration → Resurrect the default routing
 It is nesessary to set a local IP for 'br0' to ensure CT ↔ HN connection availability.

.. _`http://zh.wikipedia.org/wiki/OpenVZ`: http://zh.wikipedia.org/wiki/OpenVZ
