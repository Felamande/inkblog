title: 博客的建站过程记录
date: 2016-06-09 21:09:34 +0800
update: 2016-06-09 22:50:17 +0800
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
我的博客是用```Go```和```ink```构建的，话说Go本身就可以当服务端用，为什么还要Nginx呢？
其实是这样的
之前我给老爸写的一个[大乐透和值筛选](https://lotdb.kirigiri.me)的web
应用放在服务器上，现在要撘博客，两个程序不能耦合进一个程序里，分开又不能同时占用80端口
于是就愉快地决定用Nginx来当代理。

可是我的域名的子域名总是解析不了，我开始以为DNSPOD子域名解析是残废的，于是我又愉快决定根据不同的URL路径
来决定代理到哪个upstream。可是到这一步就不愉快了
#### 首先我把```kirigiri.me/lotdb/*```代理到我的大乐透筛选服务端
```javascript
location  /lotdb {
    proxy_pass http://127.0.0.1:$lotdb_port/
}
location ^~ /lotdb/ {
    proxy_pass http://127.0.0.1:$lotdb_port/
}
```
第一个规则防止nginx把```/lotdb```302跳转到```/lotdb/```
第二个规则把类似请求```http://kirigiri.me/lotdb/other/path```
代理到```http://127.0.0.1:$lotdb_port/other/path```

#### 然后就是把```kirigiri.me/blog/*```代理到博客程序了
