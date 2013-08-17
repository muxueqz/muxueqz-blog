xrdp
##########################################################################################################################################
:date: 2010-06-25 07:44:49
:category: 系统运维
:slug: xrdp

xrdp：开源远程桌面协议（RDP）服务器

项目主页:\ `http://xrdp.sourceforge.net/`_

 

以下是项目介绍： 基于
rdesktop，xrdp使用远程桌面协议提出一个图形用户界面给用户。

该项目目标是提供一个全功能的Linux终端服务器(RDP)，客户能够从接受连接rdesktop和Windows中的终端服务器/远程桌面。
不同于Windows
NT/2000/2003/2008中的终端服务器，xrdp不会显示Windows桌面，但会提供一个X窗口桌面给用户。
Xrdp使用Xvnc或X11rdp管理X会话。

要了解有关xrdp如何工作，请点击 `设计`_\ 和 `这里`_\ 。

如有问题或意见，请邮件 xrdp-devel@lists.sourceforge.net

该项目是下发布的GNU公共许可证（GPL）。

在Debian中安装很方便 :sudo apt-get install xdrp
然后就可以用rdesktop或Windows 远程协助来连接了，有图有真相：

`|image2|`_

.. _`http://xrdp.sourceforge.net/`: http://xrdp.sourceforge.net/
.. _设计: http://xrdp.sourceforge.net/documents/xrdpdesign/index.html
.. _这里: http://xrdp.sourceforge.net/documents/asession/index.html
.. _|image1|: http://www.yupoo.com/photos/muxueqz/75703065/

.. _|xrdp|: http://pic.yupoo.com/muxueqz/16814987d69f/small.jpg
.. _|image2|: http://pic.yupoo.com/muxueqz/16814987d69f/small.jpg
