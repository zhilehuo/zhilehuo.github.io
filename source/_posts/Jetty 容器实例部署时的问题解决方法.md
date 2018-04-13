---
title: jetty 容器实例部署时的问题解决方法
date: 2017-05-03 14:47:15
tags:
- Jetty
categories:
- Tech
---

# jetty应用部署时的注意点

本文的目的不是为了手把手教你部署一个jetty容器实例，而是为了让你在碰到部署问题是来查阅，节省网上搜索的时间，所以如果你发现并解决了本文未涵盖的部署问题，欢迎编辑此文。

1. 在start.d/http.ini修改容器监听的**端口号**：
   	
   jetty.port=8046

2. 向jetty实例中**添加**`jsp`、`jsp-impl`、`jstl`、`plus`四个**模块**，命令：

    java -jar ../../jetty-distribution-9.2.21.v20170120/start.jar --add-to-start=jsp,jstl,jsp-impl,plus

3. 配置**`dataSource`**，不用`META-INF/context.xml`了，换成`WEB-INF/jetty-env.xml`，配置内容如下：

   <?xml version="1.0" encoding="UTF-8"?>
   	<!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">

   	<Configure id="wac" class="org.eclipse.jetty.webapp.WebAppContext">
   	  <New id="jdbc/mysql.demo" class="org.eclipse.jetty.plus.jndi.Resource">
   	     <Arg>jdbc/mysql.demo</Arg>
   	     <Arg>
   	     <New class="org.apache.commons.dbcp.BasicDataSource">
   	     <Set name="driverClassName">com.mysql.jdbc.Driver</Set>
   	     <Set name="url">jdbc:mysql://rds7niju37niju3.mysql.rds.aliyuncs.com:3306/clinic?characterEncoding=UTF-8&amp;autoReconnect=true&amp;zeroDateTimeBehavior=round</Set>
   	     <Set name="username">clinic</Set>
   	     <Set name="password">Clinic423</Set>
   	     <Set name="maxActive">20</Set>
   	     <Set name="maxIdle">3</Set>
   	     <Set name="maxWait">10</Set>
   	     <Set name="removeAbandoned">true</Set>
   	     <Set name="removeAbandonedTimeout">5</Set>
   	     <Set name="logAbandoned">false</Set>
   	     <Set name="validationQuery">select 1</Set>
   	     <Set name="testOnBorrow">false</Set>
   	     <Set name="testOnReturn">false</Set>
   	     <Set name="testWhileIdle">true</Set>
   	     <Set name="timeBetweenEvictionRunsMillis">60000</Set>
   	     <Set name="numTestsPerEvictionRun">100</Set>
   		 <Set name="connectionInitSqls">
   		  <New class="java.util.ArrayList">
   			<Call name="add">
   			  <Arg>set names utf8mb4</Arg>
   			</Call>
   		   </New>
   		  </Set>
   	     </New>
   	     </Arg>
   	  </New>
   	</Configure>

4. 修改Spring配置文件中的**`jndi`名称**，去掉`java:comp/env/`,否则jetty无法找到该资源。

   <!-- connection pool-->
    	<jee:jndi-lookup id="dataSource" jndi-name="jdbc/mysql.demo"></jee:jndi-lookup>

5. 如果`dataSource`配置的是**`dbcp`**连接池，需要在应用的`WEB-INF/lib`目录中放入`commons-dbcp.jar`和`commons-pool.jar`两个实现jar包。

6. 在`start.ini`中配置**`jvm`**选项

   --exec
   	-Xms200m
   	-Xmx200m
   	-XX:MaxPermSize=256m
   	-verbose:gc
   	-Xloggc:/home/jetty/instance/njmatrix/logs/jvmgc.log
   	-XX:+PrintGCDateStamps
   	-XX:+PrintGCTimeStamps
   	-XX:+PrintGCDetails
   	-XX:+PrintTenuringDistribution
   	-XX:+PrintCommandLineFlags
   	-XX:+PrintReferenceGC
   	-XX:+PrintAdaptiveSizePolicy

7. 设置jetty运行使用的**`JAVA_HOME`**,在`ijetty.sh`文件中设置，如：

   javabin=/etc/alternatives/java_sdk_1.8.0/bin/java
   	

8. 应用内**获取配置文件完整路径**的方式，需要改为：

   String logfile = getInitParameter("log4j-init-file");
   	String logfilepath = getServletContext().getRealPath("/"+logfile);

   旧的读取方式是：
   	
   	String prefix = getServletContext().getRealPath("/");
   	String logfilepath = prefix + getInitParameter("log4j-init-file");

   这将导致`prefix`结尾少个"/"

9. **静态文件编码设置**

  在web.xml中加入配置：

  	<locale-encoding-mapping-list>
  		<locale-encoding-mapping>
  			<locale>zh</locale>
  			<encoding>UTF-8</encoding>
  		</locale-encoding-mapping>
  	</locale-encoding-mapping-list>

10. `web.xml`中**删除无用的Servlet配置**

  jetty中，如果`web.xml`定义的`Servlet`类不存在，应用启动时会报错，注意将没用的配置删除。

#### 结束

以上是一部分在部署jetty容器时碰到的问题，如果部署、启动时有其它问题，可以查看`logs`目录下的日志，找到运行错误并修复。