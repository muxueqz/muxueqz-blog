ZTE AC 580 for Linux 1.0
##########################################################################################################################################
:date: 2009-09-04 06:28:49
:category: 系统运维
:slug: zte-3g-new

呵呵，幸好有空。

注:此脚本灵感与办法来自于\ `http://forum.ubuntu.org.cn/viewtopic.php?f=116&t=202430&start=0`_\ ，感谢作者！
 驱动模块制作方法：

原来使用的usbserial是针对低速的设备设计的，使用的缓冲区很小，并没考虑现在这种高速上网设备，因此造成网速有限制。可以通过修改usbserial的源代码，使其具有大小可变的缓冲区（参考www.evdoforums.com/thread4381.html），其步骤如下：

- 下载安装当前内核源码。在\ `http://www.kernel.org/pub/linux/kernel/v2.6`_ 中搜索你当前使用的内核版本，然后请猛击下载
- copy当前内核配置文件至内核源代码目录下，

     cp /boot/config-$(uname -r) /usr/src/linux-2.6.28/.config

  
- cd到内核源码目录，为编译模块创建配置文件。

     cd /usr/src/linux-source-2.6.24/ && make menuconfig

- 创建一个usbserial.c的补丁文件/root/usbserial.c.patch，内容如下：
  
::

     --- linuxold/drivers/usb/serial/usb-serial.c 2006-12-31
    17:40:28.000000000 -0600
     +++ linux/drivers/usb/serial/usb-serial.c 2009-05-02
    23:55:08.000000000 -0600
     @@ -58,4 +58,5 @@
     \*/
     +static ushort maxRSize, maxWSize, maxISize;
     static int debug;
     static struct usb\_serial \*serial\_table[SERIAL\_TTY\_MINORS]; /\*
    initially all NULL \*/
     @@ -817,4 +818,6 @@
     }
     buffer\_size = le16\_to\_cpu(endpoint->wMaxPacketSize);
     + if (buffer\_size < maxRSize)
     + buffer\_size = maxRSize;
     port->bulk\_in\_size = buffer\_size;
     port->bulk\_in\_endpointAddress = endpoint->bEndpointAddress;
     @@ -841,4 +844,6 @@
     }
     buffer\_size = le16\_to\_cpu(endpoint->wMaxPacketSize);
     + if (buffer\_size < maxWSize)
     + buffer\_size = maxWSize;
     port->bulk\_out\_size = buffer\_size;
     port->bulk\_out\_endpointAddress = endpoint->bEndpointAddress;
     @@ -866,4 +871,6 @@
     }
     buffer\_size = le16\_to\_cpu(endpoint->wMaxPacketSize);
     + if (buffer\_size < maxISize)
     + buffer\_size = maxISize;
     port->interrupt\_in\_endpointAddress = endpoint->bEndpointAddress;
     port->interrupt\_in\_buffer = kmalloc (buffer\_size, GFP\_KERNEL);
     @@ -1191,2 +1198,8 @@
     module\_param(debug, bool, S\_IRUGO \| S\_IWUSR);
     MODULE\_PARM\_DESC(debug, "Debug enabled or not");
     +module\_param(maxRSize, ushort, 0);
     +MODULE\_PARM\_DESC(maxRSize, "User specified USB input buffer
    size");
     +module\_param(maxWSize, ushort, 0);
     +MODULE\_PARM\_DESC(maxWSize, "User specified USB output buffer
    size");
     +module\_param(maxISize, ushort, 0);
     +MODULE\_PARM\_DESC(maxISize, "User specified USB interrupt buffer
    size");

- 对内核源码中的usbserial.c应用补丁文件

     cd /usr/src/linux-2.6.28 && patch -Np0 -i /root/usbserial.c.patch


如果未能运行成功，也可以手工对usbserial.c进行修改，只要把以上标有“＋”的行加入usbserial.c中相应位置即可。

- 编译修改后的模块（这里实际上编译了所有的USB串口模块，但至少比编译整个内核快得多）

     make -C /lib/modules/$(uname -r)/build M=/usr/src/linux-2.6.28/drivers/usb/serial

  
- 备份当前使用的usbserial.ko，然后将上步生成的usbserial.ko copy到/lib/modules/kernel/drivers/usb/serial/

 安装及使用方法：
 下载地址：\ `http://210.56.192.43/xyj/Downloads/zte-1.0.tar.bz2`_

注：这只是拨号脚本，不包含驱动。驱动由于内核版本的不同，需要根据内核来制作。稍候会详细说明。
 下载之后解压到/tmp/

     tar xvfj zte-1.0.tar.bz2 -C /tmp/

 进入解压后的目录

     cd /tmp/zte/

 安装

      ./zte-install.sh

 好了!开始拨号!

     zte-3g.sh

.. _`http://forum.ubuntu.org.cn/viewtopic.php?f=116&t=202430&start=0`: http://forum.ubuntu.org.cn/viewtopic.php?f=116&t=202430&start=0
.. _`http://www.kernel.org/pub/linux/kernel/v2.6`: http://www.kernel.org/pub/linux/kernel/v2.6
.. _`http://210.56.192.43/xyj/Downloads/zte-1.0.tar.bz2`: http://210.56.192.43/xyj/Downloads/zte-1.0.tar.bz2
