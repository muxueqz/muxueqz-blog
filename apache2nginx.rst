ispconfig使用nginx 反向代理
##########################################################################################################################################
:date: 2010-06-27 05:00:09
:category: 系统运维
:slug: apache2nginx

OS:Debian Lenny 5

修改Apache端口：

vim /etc/apache2/ports.conf 将其中的

Listen 80

 修改为

Listen 127.0.0.1:80

 其它不变

Nginx的vhost配置文件可以大致如下：

    server {
    listen   8.8.8.8:80; #把8.8.8.8改成你的IP即可
    server\_name  \_;  #泛域名支持
    expires 1d;
    gzip             on;
    # gzip\_static on;
    gzip\_proxied        any;
    gzip\_disable        "MSIE [1-5].";
    gzip\_comp\_level     9;
    gzip\_min\_length  1000; 
    gzip\_buffers     4 8k; 
    gzip\_types       text/plain application/x-javascript text/css
    text/html application/xml text/javascript; 
    location / {
    proxy\_pass  http://127.0.0.1:80 ; 
    proxy\_redirect     off;
    proxy\_set\_header   Host             $host;
    proxy\_set\_header   X-Real-IP        $remote\_addr;
    }
    }


