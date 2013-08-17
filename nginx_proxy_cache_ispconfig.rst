ispconfig使用nginx反向代理+缓存，性能提高200倍!
##########################################################################################################################################
:date: 2011-01-08 13:53:05
:category: 系统运维
:slug: nginx_proxy_cache_ispconfig

目标 

 - 缓存动态脚本生成的html 
 - 缓存静态文件(ico\|css\|js\|gif\|jpe?g\|png\|txt)，让nginx直接从缓存中提供静态文件，不用再透过apache2来提供(众所周知，Apache2的静态文件性能远不如nginx)
 - 对首页($uri = /)，控制合适的过期时间，既要考虑性能，也要考虑用户要即时访问到最新信息。

环境 

 - OS(操作系统):Debian GNU/Linux Lenny 5.0 
 - 前端Web Server(反向代理):Nginx 0.7.67 
 - 后端Web Server(处理动态页面):Apache2 
 - 涉及网站类型:主要是PHP，Typecho，Wordpress 

参照:[[http://www.linuxzh.org/Linux/apache2nginx.html\|ispconfig使用nginx
反向代理]]

过程记录 
 关闭Apache2的gzip压缩，交给nginx去压缩。 
 echo 'SetEnv no-gzip' >> /etc/apache2/apache2.conf 
 nginx的配置文件添加如下内容: 

:: 

   http { 
   ... 
   #添加一个名为STATIC的cache空间 
   proxy\_cache\_path /var/tmp/nginx-cache/STATIC levels=1:2 keys\_zone=STATIC:1000m 
   inactive=24h max\_size=1g; 
   } 
   server { 
   listen 8.8.8.8:80; #你自己的IP 
   server\_name \_; #泛域名支持 
   gzip on; 
   gzip\_static on; 
   gzip\_proxied any; 
   gzip\_disable "MSIE [1-5]."; 
   gzip\_comp\_level 9; 
   gzip\_min\_length 1000; 
   gzip\_buffers 4 8k; 
   gzip\_types text/plain application/x-javascript text/css text/html
   application/xml text/javascript; 
   location / { 
   proxy\_pass http://127.0.0.1:80 ; #后端的Apache2 
   proxy\_redirect off; 
   proxy\_set\_header Host $host; 
   proxy\_set\_header X-Real-IP $remote\_addr; 
   proxy\_set\_header X-Forwarded-For $proxy\_add\_x\_forwarded\_for; 
   proxy\_pass\_header Set-Cookie;
   #对用户传输Set-Cookie的http头，不然无法支持一些包含cookie的应用，比如我的typecho
   proxy\_hide\_header X-Powered-By; #隐藏不必要的头，减少传输数据 
   proxy\_hide\_header X-Mod-Pagespeed; #隐藏不必要的头，减少传输数据 
   proxy\_cache STATIC; #使用先前定义的cache空间 
   proxy\_cache\_valid 200 404 304 1m; 
   proxy\_cache\_use\_stale error timeout invalid\_header updating 
   http\_500 http\_502 http\_503 http\_504; 
   proxy\_cache\_key "$host$uri$is\_args$args$http\_cookie";
   #这是关键，以免用户的cookie混用 
   location ~\* .(ico\|css\|js\|gif\|jpe?g\|png)$ {
   #针对静态文件单独处理 
   proxy\_pass http://127.0.0.1:80 ; #后端的Apache2 
   proxy\_set\_header Host $host; 
   expires max; 
   break; 
   proxy\_cache STATIC; 
   proxy\_cache\_valid 200 404 304 1m; 
   proxy\_cache\_use\_stale error timeout invalid\_header updating 
   http\_500 http\_502 http\_503 http\_504; 
   proxy\_cache\_key $host$uri$is\_args$args;
   #可以看出，与上面的相比，这里没有cookie，最大化利用静态文件的缓存。 
   } 
   } 
   }

压力测试 

 优化之前: 

之所以用并发100,请求100,是因为1000的时候已经挂掉了(测试环境的硬件性能有限)……

::

 ispconfig:/tmp# ab -c 100 -n 100 [[http://www.linuxzh.org/jobs.html]]

 This is ApacheBench, Version 2.3 <$Revision: 655654 $> 
 Copyright 1996 Adam Twiss, Zeus Technology Ltd,
 http://www.zeustech.net/ 
 Licensed to The Apache Software Foundation, http://www.apache.org/ 
 Benchmarking 127.0.0.1 (be patient).....done 
 Server Software: Apache 
 Server Hostname: www.linuxzh.org 
 Server Port: 80 
 Document Path: /jobs.html 
 Document Length: 31483 bytes 
 Concurrency Level: 100 
 Time taken for tests: 18.370 seconds 
 Complete requests: 100 
 Failed requests: 0 
 Write errors: 0 
 Total transferred: 3178200 bytes 
 HTML transferred: 3148300 bytes 
 Requests per second: 5.44 [#/sec] (mean) 
 Time per request: 18369.758 [ms] (mean) 
 Time per request: 183.698 [ms] (mean, across all concurrent requests)

 Transfer rate: 168.96 [Kbytes/sec] received 
 Connection Times (ms) 
 min mean[+/-sd] median max 
 Connect: 1 2 0.5 2 3 
 Processing: 467 9837 4912.1 9825 18365 
 Waiting: 455 9820 4912.1 9804 18350 
 Total: 470 9839 4911.6 9827 18367 
 Percentage of the requests served within a certain time (ms) 
 50% 9827 
 66% 12902 
 75% 14182 
 80% 15097 
 90% 16446 
 95% 17511 
 98% 18322 
 99% 18367 
 100% 18367 (longest request) 
 优化之后: 
 ispconfig:/tmp# ab -c 1000 -n 1000 [[http://www.linuxzh.org/jobs.html]]

 This is ApacheBench, Version 2.3 <$Revision: 655654 $> 
 Copyright 1996 Adam Twiss, Zeus Technology Ltd,
 http://www.zeustech.net/ 
 Licensed to The Apache Software Foundation, http://www.apache.org/ 
 Benchmarking 210.56.192.69 (be patient) 
 Completed 100 requests 
 Completed 200 requests 
 Completed 300 requests 
 Completed 400 requests 
 Completed 500 requests 
 Completed 600 requests 
 Completed 700 requests 
 Completed 800 requests 
 Completed 900 requests 
 Completed 1000 requests 
 Finished 1000 requests 
 Server Software: nginx 
 Server Hostname: www.linuxzh.org 
 Server Port: 80 
 Document Path: / 
 Document Length: 17117 bytes 
 Concurrency Level: 1000 
 Time taken for tests: 0.160 seconds 
 Complete requests: 1000 
 Failed requests: 0 
 Write errors: 0 
 Total transferred: 17451000 bytes 
 HTML transferred: 17117000 bytes 
 Requests per second: 6236.55 [#/sec] (mean) 
 Time per request: 160.345 [ms] (mean) 
 Time per request: 0.160 [ms] (mean, across all concurrent requests)

 Transfer rate: 106283.28 [Kbytes/sec] received 
 Connection Times (ms) 
 min mean[+/-sd] median max 
 Connect: 35 57 7.7 58 68 
 Processing: 55 62 4.9 62 71 
 Waiting: 12 40 15.7 35 69 
 Total: 104 119 5.7 119 130 
 Percentage of the requests served within a certain time (ms) 
 50% 119 
 66% 121 
 75% 123 
 80% 124 
 90% 126 
 95% 128 
 98% 129 
 99% 130 
 100% 130 (longest request)

注意 

 1、proxy\_key 中只有用户发出的Cookie才会单独生成一个缓存，也就是说，不会每个未登陆的用户都生成一个缓存文件。

 另外，针对静态文件，不记录cookie。

2、记得关闭后端Server的gzip压缩，nginx会直接缓存gzip的内容，不支持gzip的浏览器可能会出现乱码等问题。
