---
title: nginx开启https
date: 2016-07-28 14:59:51
categories:
- 技术笔记
tags:
- Nginx
- https
- 服务器
---
服务器环境使用的是nginx，其实用其他服务器也都是一样的，配置可以直接百度或者google，也可以直接到服务器官网直接查文档。本文主要讲如何申请免费证书，内容门槛低，老司机可以绕路了。
<!--more-->
### 获取证书

本文主要参考下面两篇文章
* [分享一个免费SSL证书申请网站，给网站开启https协议](http://zhangge.net/4890.html)
* [Linux+Nginx/Apache/Tomcat新增SSL证书，开启https访问教程](http://zhangge.net/4861.html)

参考上面的教程，这里我也用的是沃通

1 注册账号 [https://login.wosign.com/reg.html](https://login.wosign.com/reg.html)
2 申请免费证书 [https://buy.wosign.com/free/FreeSSL.html](https://buy.wosign.com/free/FreeSSL.html)

这个申请的操作也很简单，按照网站的提示一步一步操作下来就好了

申请通过以后，到我的订单里找到一条刚刚申请的数据，后面有个取走证书的连接，点击下载证书

注：申请过程中有个步骤是设置文件密码的操作，其实就是设置个压缩包密码，这个随便写，不过为了以后能方便使用，最好也记住一下

到此我们的免费证书就拿到手了！

### Nginx配置Https

一、先通过命令查看下Nginx是否安装了SSL模块

``` bash
nginx -V
```

这里注意是大写的V，界面显示有 `--with-http_ssl_module` 说明已经安装了，如果没有先安装该模块，这里不作演示了

然后把申请的证书解压出来，有两个文件 `XXXX.key` 和 `XXXX.crt` ，把两个文件上传到服务器上，推荐放在nginx下的ssl目录

二、修改Nginx配置

``` nginx
server {
    listen 80;
    #新增监听443端口，并指定443为ssl：
    listen 443 ssl;
    server_name yourdomain.com;
    #新增ssl配置---开始：
    ssl_certificate /usr/local/nginx/ssl/yourdomain_bundle.crt; #证书公钥文件路径
    ssl_certificate_key /usr/local/nginx/ssl/yourdomain.key;   #证书私钥文件路径
    ssl_session_timeout 5m;  #5分钟session会话保持
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!ADH:!EXPORT56:RC4+RSA:+MEDIUM;
    #新增ssl配置---结束：
    location / {
        #其他规则保持不变
    }
}
```

测试下配置

``` bash
nginx -t
```

如下显示则为正确无误：

```
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

确认无误之后，执行如下命令重载nginx，让配置生效：

``` bash
nginx -s reload
```

现在访问http或者https都能正常访问你的网站了。

如果你不想使用http而只用https可以按照下面配置

``` nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /path/for/yourdomain.com;
    location / {
        rewrite (.*) https://yourdomain.com$1 permanent;
    }
}

server {
    listen 443 ssl;
    server_name yourdomain.com;
    #新增ssl配置---开始：
    ssl_certificate /usr/local/nginx/ssl/yourdomain.com.crt; #证书公钥文件路径
    ssl_certificate_key /usr/local/nginx/ssl/yourdomain.com.key;   #证书私钥文件路径
    ssl_session_timeout 5m;  #5分钟session会话保持
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!ADH:!EXPORT56:RC4+RSA:+MEDIUM;
    #新增ssl配置---结束：
    location / {
        #其他规则保持不变
    }
}
```

如果想一部分连接使用https一部分连接使用http，请参考开头引用的那篇文章！