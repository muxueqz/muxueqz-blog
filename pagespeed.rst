在ubuntu/debian上安装google pagespeed
##########################################################################################################################################
:date: 2010-12-19 02:05:37
:category: 系统运维
:slug: pagespeed

Google推出mod-pagespeed免费模块用于优化Apache
HTTP服务器框架下的网站速度。该模块从多个方面对服务器运行速度进行优化，例如可以对图片进行再压缩，通过CMS(内容管理系统)改变网页构造但不改变CMS本身等。在此次开源之前，Google内部也一直使用该模块。

作用如下：

 - 不需要对网站 CMS 系统进行改变即可应用。
 - 加速模块可以自行对网络传输的 html 字节优化及对图象 、css 进行压缩优化传输
 - 智能缓存是一大亮点，它可以自动智能缓存，加速下载


目前这套优化模块已经应用GoDaddy服务器上，而且反响良好。根据此前的一些实践来看， 通过mod\_pagespeed可以对 Web 性能的多个方面，包括缓存、客户端与服务器之间的连接、载荷大小等进行优化，最大可将页面加载时间缩短 50% 。

**For 32 bit**

::

 wget [[https://dl-ssl.google.com/dl/linux/direct/mod-pagespeed-beta\_current\_i386.deb]]
 dpkg -i mod-pagespeed-beta\_current\_i386.deb 
 apt-get -f install 
 /etc/init.d/apache2 restart 

**For 64 bit**

::

 wget [[https://dl-ssl.google.com/dl/linux/direct/mod-pagespeed-beta\_current\_amd64.deb]]
 dpkg -i mod-pagespeed-beta\_current\_amd64.deb 
 apt-get -f install 
 /etc/init.d/apache2 restart 
