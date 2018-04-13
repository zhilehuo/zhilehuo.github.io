---
title: Jetty 概览
date: 2017-04-01 15:55:40
tags:
- Jetty
categories:
- Tech
---

> 不严肃地说，这并不算是一篇文章，只能算是一篇笔记，时间有限，等有时间我再来好好整理一下吧。

官方文档感觉很不错：

[http://www.eclipse.org/jetty/documentation/9.2.21.v20170120/index.html](http://www.eclipse.org/jetty/documentation/9.2.21.v20170120/index.html)

版本记录：

[http://www.eclipse.org/jetty/documentation/current/what-jetty-version.html](http://www.eclipse.org/jetty/documentation/current/what-jetty-version.html)

| Version | Year      | Home            | JVM  | Protocols                                                    | Servlet | JSP  | Status     |
| :------ | :-------- | :-------------- | :--- | :----------------------------------------------------------- | :------ | :--- | :--------- |
| 9.4     | 2016      | Eclipse         | 1.8  | HTTP/1.1 (RFC 7230), HTTP/2 (RFC 7540), WebSocket (RFC 6455, JSR 356), FastCGI | 3.1     | 2.3  | Stable     |
| 9.3     | 2015      | Eclipse         | 1.8  | HTTP/1.1 (RFC 7230), HTTP/2 (RFC 7540), WebSocket (RFC 6455, JSR 356), FastCGI | 3.1     | 2.3  | Stable     |
| 9.2     | 2014      | Eclipse         | 1.7  | HTTP/1.1 RFC2616, javax.websocket, SPDY v3                   | 3.1     | 2.3  | Stable     |
| 8       | 2009-2014 | clipse/Codehaus | 1.6  | HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3                | 3.0     | 2.2  | Deprecated |

我们因为没有全面升级至java1.8，所以目前使用9.2版本。

目前最新版本：

[http://www.eclipse.org/jetty/download.html](http://www.eclipse.org/jetty/download.html)

| Release          |                                                              |                                                              |                                                              |                                                              |                   |
| :--------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------- |
| 9.4.1.v20170120  | [.zip](http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.4.1.v20170120/jetty-distribution-9.4.1.v20170120.zip) | [.tgz](http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.4.1.v20170120/jetty-distribution-9.4.1.v20170120.tar.gz) | [apidocs](http://download.eclipse.org/jetty/9.4.1.v20170120/apidocs) | [source](https://github.com/eclipse/jetty.project/tree/jetty-9.4.1.v20170120) | Latest (JDK 8+)   |
| 9.3.16.v20170120 | [.zip](http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.3.16.v20170120/jetty-distribution-9.3.16.v20170120.zip) | [.tgz](http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.3.16.v20170120/jetty-distribution-9.3.16.v20170120.tar.gz) | [apidocs](http://download.eclipse.org/jetty/9.3.16.v20170120/apidocs) | [source](https://github.com/eclipse/jetty.project/tree/jetty-9.3.16.v20170120) | Latest (JDK 8+)   |
| 9.2.21.v20170120 | [.zip](http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.2.21.v20170120/jetty-distribution-9.2.21.v20170120.zip) | [.tgz](http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.2.21.v20170120/jetty-distribution-9.2.21.v20170120.tar.gz) | [apidocs](http://download.eclipse.org/jetty/9.2.21.v20170120/apidocs) | [xref](http://download.eclipse.org/jetty/9.2.21.v20170120/xref) | Release (Java 7+) |

启动：

进到jetty目录里，执行：java -jar start.jar

其中start.jar就在解压后的jetty目录中

如上命令将启动当前目录下的jetty服务

jetty启动时，基于两个变量：

**jetty.home**

The property that defines the location of the jetty distribution, its libs, default modules and default XML files (typically start.jar, lib, etc)

**jetty.base**

The property that defines the location of a specific instance of a jetty server, its configuration, logs and web applications (typically start.ini, start.d, logs and webapps)

意思是，jetty.home可以整个系统中共享一份，jetty.base要每个实例有单独的配置

创建一个新的jetty.base：

	mkdir newbase
	java -jar $jetty.home/start.jar —add-to-startd=http,http2,deploy

使用

	java -jar $jetty.home/start.jar —help

可以查看帮助

修改端口：

	jara -jar $jetty.home/start.jar jetty.port=8081

或修改配置文件：start.d/http.ini

最后，我写了一个管理Jetty实例的工作`jettys`，SVN地址在`/server/server_script/jettys`，具体使用请参考里边的`README.md`