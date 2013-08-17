pyifstat
##########################################################################################################################################
:date: 2011-03-03 02:46:54
:category: Python
:tags: python,ifstat
:slug: pyifstat

python写的ifstat，用于查看当前带宽(流量)占用情况 
 借用ifstat的介绍： 
 ifstat is a tool to report network interfaces bandwidth just like
 vmstat/iostat do for other system counters. It can monitor local
 interfaces by polling the kernel counters

::

 #!/usr/bin/env python
 import time
 import sys
 if len(sys.argv) > 1:
 INTERFACE = sys.argv[1]
 else:
 INTERFACE = 'eth0'
 STATS = []
 print 'Interface:',INTERFACE
 def rx():
 ifstat = open('/proc/net/dev').readlines()
 for interface in ifstat:
 if INTERFACE in interface:
 stat = float(interface.split()[1])
 STATS[0:] = [stat]
 def tx():
 ifstat = open('/proc/net/dev').readlines()
 for interface in ifstat:
 if INTERFACE in interface:
 stat = float(interface.split()[9])
 STATS[1:] = [stat]
 print 'In Out'
 rx()
 tx()
 while True:
 time.sleep(1)
 rxstat\_o = list(STATS)
 rx()
 tx()
 RX = float(STATS[0])
 RX\_O = rxstat\_o[0]
 TX = float(STATS[1])
 TX\_O = rxstat\_o[1]
 RX\_RATE = round((RX - RX\_O)/1024/1024,3)
 TX\_RATE = round((TX - TX\_O)/1024/1024,3)
 print RX\_RATE ,'MB ',TX\_RATE ,'MB'

文件在 http://openwrt-desktop.googlecode.com/files/pyifstat.py
