vyatta 软路由负载均衡配置
##########################################################################################################################################
:date: 2010-08-19 14:49:51
:category: 系统运维
:slug: vyatta_two

Vyatta是基于Linux的路由器、防火墙、VPN。凭借它开源软件的性能、可靠性和专有联网设备特性，完全可以构筑起企业级网络。

::

 Vyatta能帮你：
 完全胜任大规模BGP环境 
 把小型办公网接入Internet 
 利用状态检测防火墙保护你的网络安全 
 用VPN技术安全地接入到你的办公网 
 使用该软件适应从DSL到10Gbps容量 
 避免专用网络设备更新费用 
 利用Xen、VMWare运行虚拟网络环境 
 在数据中心环境，添加网络和安全性到刀片服务器 
 提供基于网络的，可管理的完全服务 
 增加网络冗余 
 建立你自己的品质一流的办公网解决方案。 

**配置文件说明**

::

 vyatta@vyatta# run show configuration 
 interfaces { #配置接口 
 ethernet eth1 { #配置eth1接口 
 address 1.1.1.2/24 #配置eth1接口的IP 
 hw-id 00:1c:2a:e3:ca:09 #默认即可，是MAC地址 
 } 
 ethernet eth2 { #配置eth2接口 
 duplex auto 
 pppoe 0 { #配置eth2的ADSL 
 default-route auto #使用pppoe为默认网关 
 password \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* #pppoe密码 
 user-id username #pppoe用户 
 } 
 } 
 ethernet eth3 { #配置eth3接口 
 address 192.168.4.1/24 #配置eth3的IP 
 } 
 ethernet eth4 { #配置eth4接口 
 address 192.168.1.1/24 #配置eth4的IP 
 } 
 _loopback lo { 
 } 
 } 
 load-balancing { #负载均衡配置 
 wan { 
 interface-health eth1 { #用来负载的资源1 
 failure-count 1 
 nexthop GW #下一跳 
 } 
 interface-health pppoe0 { #用来负载的资源2 
 failure-count 1 
 nexthop dhcp #pppoe的IP是动态的，所以是dhcp 
 test 1 { #注意，不配置此项的话，pppoe的负载均衡无法使用 
 target 8.8.8.8 #检查用的IP 
 type ping #检查方式为ping 
 } 
 } 
 rule 1 { #被负载的策略 
 destination { 
 address !192.168.0.0/16
 #这段的意思是目标地址不包含192.168.0.0/16这个子网，如果不配置此项，负载均衡后无法联系其它子网。

 } 
 inbound-interface eth4 #被负载的接口 
 interface eth1 { #使用哪个负载接口(即选择出口) 
 weight 1 #权重 
 } 
 interface pppoe0 { 
 weight 1 
 } 
 } 
 rule 2 { 
 destination { 
 address !192.168.0.0/16 
 } 
 inbound-interface eth3 
 interface eth1 { 
 weight 1 
 } 
 interface pppoe0 { 
 weight 1 
 } 
 } 
 } 
 } 
 protocols { #配置协议 
 static { #配置静态路由 
 route 192.168.2.0/24 { #目标为192.168.2.0/24的网络 
 next-hop 192.168.1.8 { #下一跳为192.168.1.8 
 } 
 } 
 route 192.168.3.0/24 { 
 next-hop 192.168.1.8 { 
 } 
 } 
 } 
 } 
 service { #配置服务 
 dhcp-server { #配置DHCP服务器 
 disabled false 
 shared-network-name kooland { #网络名 
 authoritative disable 
 subnet 192.168.1.0/24 { #子网 
 default-router 192.168.1.1 #客户端获得的网关地址 
 dns-server 202.106.0.20 #客户端获得的DNS 
 start 192.168.1.50 { #起始IP 
 stop 192.168.1.250 #结束IP 
 } 
 } 
 } 
 } 
 dns { #配置DNS服务器 
 forwarding { #转发 
 cache-size 150 #缓存大小 
 listen-on eth4 #侦听在哪个接口 
 name-server 8.8.8.8 #转发到哪个DNS 
 name-server 8.8.4.4 
 } 
 } 
 https #https web管理 
 nat { #NAT网络地址转换 
 rule 1 { #策略1 
 outbound-interface eth1 #出口为eth1 
 source { #源 
 address 192.168.0.0/16 #源地址为192.168.0.0/16这个子网 
 } 
 type masquerade #类型为伪装，即pat 
 } 
 rule 2 { 
 outbound-interface pppoe0 
 source { 
 address 192.168.0.0/16 
 } 
 type masquerade 
 } 
 } 
 ssh { #开启ssh服务 
 port 22 
 } 
 } 
 system { 
 host-name vyatta 
 login { 
 user vyatta { 
 authentication { 
 encrypted-password \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
 plaintext-password \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
 } 
 } 
 } 
 name-server 8.8.8.8 
 name-server 8.8.4.4 
 ntp-server 0.vyatta.pool.ntp.org 
 package { 
 auto-sync 1 
 repository community { 
 components main 
 distribution stable 
 url http://packages.vyatta.com/vyatta 
 } 
 repository debian { 
 components "main non-free contrib" 
 distribution stable 
 url http://mirrors.163.com/debian/ 
 } 
 } 
 syslog { 
 global { 
 facility all { 
 level notice 
 } 
 facility protocols { 
 level debug 
 } 
 } 
 } 
 time-zone GMT-8 
 }
