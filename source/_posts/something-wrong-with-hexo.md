---
title: 折腾博客遇到的蛋疼问题
date: 2016-07-28 11:57:32
categories:
- 技术笔记
tags:
- Hexo
- 博客
---

这几天一直都在研究Hexo建站，目前总算折腾的差不多了，中间也是遇到不少的坑，汇总一下，留个纪念！比如这里的内容就是一段测试内容！
<!--more-->

### 发布

本来很简单的东西，配置啥的也没啥可说的，可是就是死活不行

![error info](/res/img/20160728120237.jpg)

后来在SF上看到了一篇提问，使用git bash执行命令就OK了，自己测试了一下，果然OK

### 中文路径

既然title可以使用中文，发布的时候路径也会包含中文，这就导致某些特定环境中文路径失败。

我使用的是Linux下 `Nginx + Tomcat` 的服务器环境，现别问我为什么这么麻烦，我自己知道直接最简单的服务器就够用。

因为我这个环境之前也是测试很多东西，现在搭建成这样不想（其实是懒）去更改它而已

tomcat 的conf里有个 `server.xml` 文件，找到下面配置，添加 `URIEncoding="utf-8"` 属性即可支持中文路径，原因是因为请求默认的编码问题

```
<Connector port="8888" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```

修改以后

```
<Connector port="8888" protocol="HTTP/1.1" URIEncoding="utf-8" connectionTimeout="20000" redirectPort="8443" />
```

重启服务OK了！

### 奇怪

我本地使用的是window，测试没问题，但是相同的配置放到Linux下就还是不能访问，页面提示404

后来查资料貌似是我的Linux没有中文语言包，导致中文无法识别，没办法，安装吧

输入下面命令查看系统的语言

``` bash
echo $LANG
```

显示 `en_US.UTF-8` 是英文语言，换成中文的，先查看是否已经存在语言包

``` bash
locale
```

如果不存在 `zh_CN.UTF-8` 则说明系统没有中文语言包，通过下面命令下载安装

``` bash
yum groupinstall chinese-support
```

安装成功后，通过下面命令修改配置切换默认语言

``` bash
Vi /etc/sysconfig/i18n
```

修改 `LANG="zh_CN.UTF-8"` 保存，重启，输入date命令测试下，显示中文日期，OK。

### 但是

折腾完了，本以为这样就OK了，结果还是不行，通过 `ls` 命令查看中文文件或者文件夹shell上面显示是 `??` ，而且我把路径里的中文换成 `??` 居然可以访问。

原来不是配置问题，是上传的时候中文被转码了，分析以后发现是FTP上传工具的问题，参考 [百度经验](http://jingyan.baidu.com/article/fdbd427702fb8db89f3f4876.html)这篇文章设置了下我用的FTP软件，重新上传，OK了！

至此折腾告一段落，不经历永远不知道这些坑，任何一个坑都能把你坑死。。。

还有个小问题就是我发现Hexo官网的主题连接的网站有的站点访问的路径并不是title下的文件夹路径。我们默认都是文件名跟路径一致，等下有空研究一下是不是可以自己修改文件名