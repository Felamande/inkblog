title: 博客的建站过程记录
date: 2016-06-09 21:09:34 +0800
update: 2016-07-26 22:59:24 +0800
author: me
draft: false
tags: 
    - 博客

---



## Namesilo
也不知道是发了什么疯，突然就想着要建一个个人博客了。以前总觉得买域名
太贵，而且个人博客也不是刚需，但昨天不声不响就剁手花了100多大洋,买了```kirigiri.me```
这个域名。然后就折腾起了我搬瓦工的服务器。

## DNS
卖完域名当然就是选DNS服务商了，什么NS记录用来指定域名解析服务器、
A记录用来对应ipv4地址与域名、AAAA记录用来对应ipv6地址与域名、
CNAME用来指定别名,这些我才不知道呢。

开始是选的DNSPOD来解析域名，但是添加子域名怎么也解析不了，经过chrome吧吧友指点后，
放弃了神坑DNSPOD换了CloudXns，不知道怎么的反正就是能够解析了，结果后来才发现
其实是学校DNS的锅。

## Nginx
我的博客是用```Go```和```ink```构建的。Go本身就可以当服务端用，为什么还要Nginx呢？
其实是这样的, 
之前我给老爸写的一个[大乐透和值筛选](https://lotdb.kirigiri.me)的web
应用放在服务器上，现在要撘博客，两个程序不能同时占用80端口，要用Nginx来当代理，监听80端口，再根据域名把流量代理到实际的程序端口上。

### 博客服务端
[ink](https://github.com/InkProject/ink)是一个开源的静态博客生成器，默认主题看上去比较简洁清新。
因为仍然在开发中，所以在线编辑器的功能还没开发，我本来想自己写一个在线编辑器的，后来发现工作量太大，
自己又不熟悉前端，就默默放弃了，等作者来填坑。
nginx.conf这样写：
```javascript
server {
		listen 443 ssl;
		server_name kirigiri.me;
		location / {
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header Host $host;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_pass_header Server;
			add_header X-Proxy-Server nginx;
			proxy_pass http://127.0.0.1:1313/;
		}
	}
```
ink服务端监听本地1313端口。
然后就是公共的静态文件
```javascript
server {
    listen 443 ssl;
    server_name static.kirigiri.me;
    location / {
        autoindex off;
        root /var/www/public/static;
        
    }
}
```

### 其他服务端
我还在服务器上搭了其他的服务：[https://lotdb.kirigiri.me](大乐透筛选)、[https://git.kirigiri.me]一个自助的git服务端。

#### ```lotdb.kirigiri.me```
我老爸迷上彩票很久了，每次总是抱怨说用手算和值、猜数字很难，于是我就花了大概一个星期业余时间写了一小Web应用，
后端是Go和sqlite，就是简单的读取数据库，前端用Vuejs和VueStrap，界面还马马虎虎，开源在[https://github.com/Felamande/lotdb](github)上，
与其说是开源，不如说是存储在github上吧。
nginx.conf里配置和```kirigiri.me```大同小异。

#### ```git.kirigiri.me```
用的是开源的git服务端[https://github.com/gogits/gogs](Gogs),
不过我那256M内存的小服务器好像承受不起，web端三天两头抛500错误，大概是git操作比较费内存。

#### Shadowsocks服务端
对了，其实这才是我买这个服务器最开始的目的，那就是across the wall。shadowsocks服务端我也用的是Go语言版本的。
因为我比较熟悉Go，要添加功能的话，源码修改起来比较方便，内存占用也比较省。

### Let's Encrypt
现在都2016年了，搞什么网站都要赶潮流，HTTPS就是现代web世界不可逆转的潮流。
大部分的证书都是商业公司颁发，要钱的。不过目前也出现了像Let's Encrypt这样的组织，免费签发
可信任的证书，推动互联网全面走向安全加密的时代。
Let's Encrypt证书的签发和配置非常简单。
首先从从他们的github仓库克隆下来[https://github.com/certbot/certbot](全自动脚本),然后运行：
```./letsencrypt-auto certonly -d kirigiri.me -d lotdb.kirigiri.me -d static.kirigiri.me -d git.kirigiri.me```
letsencrypt不支持通配符，防止正式滥用。在运行自动脚本之前先要关闭nginx，
因为脚本需要通过nginx来验证这个域名确实是你的。然后证书以及配置会存在```/etc/letsencrypt```目录下。
然后配置nginx：
```javascript
ssl_certificate /etc/letsencrypt/live/kirigiri.me/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/kirigiri.me/privkey.pem;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
ssl_dhparam /etc/nginx/dhparams.pem;
```
配置证书位置，协议版本，以及加密方式。那一长串```ssl_ciphers```就是各种加密方式的优先度，
最优先的当然是最安全的，感叹号阻止掉不安全加密方式。ssl_dhparam用来定义一个非默认的，更强的
Diffiel-Hellman输入参数，主要用来保证前向保密。
这个文件使用openssl产生的，```openssl dhparam -out dhparam.pem 4096```.
然后别忘了在server字段里写上```listen 443 ssl;```

最后还有两点，就是http到https的跳转以及HSTS。
因为没有监听80端口，不提供HTTP的访问，所有的http协议的请求都无法访问网站，
这就造成用户体验上的缺失，nginx.conf这样写：
```javascript
server {
    listen 80;
    server_name _;
    rewrite ^ https://$host$request_uri? permanent;
}
```
把所有80端口http请求重定向到https.

HSTS是用来强制客户端使用HTTPS协议的，在每个域名的server字段里加上：
```add_header Strict-Transport-Security "max-age=31536000";```

至此HTTPS的配置就完成了.如果你想测试自己Web服务器的安全性到底如何，可以去[https://ssllab.com](ssllab网站)上测试，
测试完之后会给你测试报告和一些提升安全性的配置建议。

## 后记
本来配置完了除了写博客也就没事事情了，但是后来又看到了一些关于HTTP/2的东西.
作为一个时刻追求最新技术的人，怎么能不开HTTP/2呢。后来发现软件仓库里的nginx版本
不支持HTTP/2，无奈就自己编译了一个最新稳定版的nginx，开启了HTTP/2.