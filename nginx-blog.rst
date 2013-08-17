博客迁移到nginx的过程
##########################################################################################################################################
:date: 2009-08-02 01:50:37
:category: 系统运维
:slug: nginx-blog

我这个博客用的是Debian的VPS，先前的Web环境用的是XAMPP，即Web
Server是Apache，数据库是Sqlite，这两个都是XAMPP中包含的，而博客源码是typecho。

 前些天抽空换成了nginx、fastcgi和Debian中的Sqlite，以下是迁移过程。
 参考了http://blog.chinaitlab.com/html/30/104830-166235.html，特此声明。
 一、备份typecho
 我的typecho安装在/opt/lampp/typecho，备份这个目录即可。
 二、删除xampp
 呵呵，话说XAMPP真方便，解压就能用，我放在了/opt/lampp中，所以直接删除即可。
 注：事先备份！
 三、安装nginx
 执行以下命令：

::

    sudo apt-get update
    sudo apt-get install nginx

 配置文件默认安装位置：

::

    conf: /etc/nginx/nginx.conf
     bin：/usr/sbin/nginx
     vhost: /etc/nginx/sites-enable/default
     cgi-params: /etc/nginx/fastcgi-params

 例：建立一个虚拟主机，写在/etc/nginx/sites-enable/中的一个文件即可

::

    server {
     listen 80;
     server\_name www.23day.com;
     access\_log /var/log/nginx/home.ucenter.access.log;
     location / {
     root /var/www/23day.com;
     index index.php;
     location ~ .php$ {
     fastcgi\_pass 127.0.0.1:9000;
     fastcgi\_index index.php;
     fastcgi\_param SCRIPT\_FILENAME
    /var/www/23day.com$fastcgi\_script\_name;
     include /etc/nginx/fastcgi\_params;
     }
     }

 四、安装php-cgi模块

::

    执行sudo apt-get install php5-cgi

 配置文件默认安装位置：

::

    php-cgi: /usr/bin/php-cgi
     php5-cgi: /usr/bin/php5-cgi
     cgi config: /etc/php5/cgi/php.ini
     修改php.ini文件的cgi.fix\_pathinfo数据为1，默认为0
     cgi.fix\_pathinfo=1;
    这样php-cgi方能正常使用SCRIPT\_FILENAME这个变量

 五、安装spawn-fcgi

spawn-fcgi是lighttpd的一个用来控制php-cgi的工具，现在已从lighttpd中独立出来成为一个开源项目。
 安装有两种办法：

    1、Apt，现在spawn-fcgi已经进入testing源，等进入stable应该也快了。如果用的是testing源，现在就可以sudo
    apt-get install spawn-fcgi

    2、编译安装，去官网下载源码编译。可以直接用我已经编译好的文件，下载链接在http://210.56.192.43/xyj/Downloads/spawn-fcgi

 以下是官网链接
 `http://redmine.lighttpd.net/projects/spawn-fcgi/news`_

如果系统没有安装GCC编译环境，刚需要在安装lighttpd之前要安装build-essential工具包，执行以下命令

    sudo apt-get install build-essential

 然后编译即可。
 这样cgi控制器就安装完成.
 四、重新配置权限
 将typecho目录cp到/var/www/nginx-default，然后执行：

    chown -R www-data.www-data /var/www/nginx-default

 4.启动测试系统.启动fast\_cgi:

    spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u www-data -g www-data -f
    /usr/bin/php-cgi

 参数含义如下-f <fcgiapp> 指定调用FastCGI的进程的执行程序位置，根据系统上所装的PHP的情况具体设置
 -a <addr> 绑定到地址addr
 -p <port> 绑定到端口port
 -s <path> 绑定到unix socket的路径path
 -C <childs> 指定产生的FastCGI的进程数，默认为5（仅用于PHP）
 -P <path> 指定产生的进程的PID文件路径
 -u和-g FastCGI使用什么身份（-u 用户 -g 用户组）运行，Ubuntu下可以使用www-data，其他的根据情况配置，如nobody、apache等
 注意:ip,端口与nginx服务器中的cgi-pass要对应. -C表示打开几个cgi进程
 启动nginx
 sudo /etc/init.d/nginx start
 好了，如果没有出错信息，则说明配置成功了，现在写个phpinfo测试下吧！

最后，附上我的/etc/nginx/sites-enable/default的配置文件，此配置文件启用了rewrite功能

::

    server {
     listen 80;
     server\_name localhost;
     access\_log /var/log/nginx/localhost.access.log;
     location / {
     root /var/www/nginx-default;
     index index.php;
     if (-f $request\_filename/index.html){
     rewrite (.\*) $1/index.html break;
     }
     if (-f $request\_filename/index.php){
     rewrite (.\*) $1/index.php;
     }
     if (!-f $request\_filename){
     rewrite (.\*) /index.php;
     }
     }
     #error\_page 404 /404.html;
     # redirect server error pages to the static page /50x.html
     #
     error\_page 500 502 503 504 /50x.html;
     location = /50x.html {
     root /var/www/nginx-default;
     }
     # proxy the PHP scripts to Apache listening on 127.0.0.1:80
     #
     #location ~ .php$ {
     #proxy\_pass http://127.0.0.1;
     #}
     # pass the PHP scripts to FastCGI server listening on
    127.0.0.1:9000
     #
     location ~ .php$ {
     fastcgi\_pass 127.0.0.1:9000;
     fastcgi\_index index.php;
     fastcgi\_param SCRIPT\_FILENAME
    /var/www/nginx-default$fastcgi\_script\_name;
     include /etc/nginx/fastcgi\_params;
     }
     # deny access to .htaccess files, if Apache’s document root
     # concurs with nginx’s one
     #
     #location ~ /.ht {
     #deny all;
     #}
     }
     # another virtual host using mix of IP-, name-, and port-based
    configuration
     #
     #server {
     #listen 8000;
     #listen somename:8080;
     #server\_name somename alias another.alias;
     #location / {
     #root html;
     #index index.html index.htm;
     #}
     #}
     # HTTPS server
     #
     #server {
     #listen 443;
     #server\_name localhost;
     #ssl on;
     #ssl\_certificate cert.pem;
     #ssl\_certificate\_key cert.key;
     #ssl\_session\_timeout 5m;
     #ssl\_protocols SSLv2 SSLv3 TLSv1;
     #ssl\_ciphers
    ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
     #ssl\_prefer\_server\_ciphers on;
     #location / {
     #root html;
     #index index.html index.htm;
     #}
     #}

 末、收尾
 创建一个脚本，让spawn-fcgi开机自动运行。
  
::

     echo "spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u www-data -g www-data -f /usr/bin/php-cgi" > /usr/sbin/fast-cgi.sh
     chmod a+x /usr/sbin/fast-cgi.sh
     echo "/usr/sbin/fast-cgi.sh" >> /etc/rc.local

 这样就可以了，注：Debian中的rc.local默认会有exit 0 ，一定要把脚本加在exit 0之前执行。

.. _`http://redmine.lighttpd.net/projects/spawn-fcgi/news`: http://redmine.lighttpd.net/projects/spawn-fcgi/news
