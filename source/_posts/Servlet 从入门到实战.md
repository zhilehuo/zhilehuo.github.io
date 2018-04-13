---
title: Servlet 从入门到实战
date: 2017-04-28 17:12:11
tags:
- Java
categories:
- Tech
---

Javaweb的后端研发需要学习的是tomcat+servlet+jsp+mysql 这些技术，其中的核心技术就是servlet。本篇详细介绍servlet。

# Servlet 简介

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录。

简而言之呀，servlet就是将从web界面或者app界面这些前端界面上获取的含有参数的请求request进行解析处理，用响应response返回这些界面需要的参数。

# Servlet 环境配置

servlet 作为一个服务器端运行的后台服务程序，如果想要本地运行需要配置一系列的环境，简介如下：

## JDK（Java Development Kit）
JDK是一个软件开发工具包，包含了java的运行环境，java工具和java基础的类库，有一点点java基础的同学应该知道这是什么的。Java Servlet当然依赖Java环境。

JDK完美配置教程链接：

windows：[http://jingyan.baidu.com/article/6dad5075d1dc40a123e36ea3.html](http://jingyan.baidu.com/article/6dad5075d1dc40a123e36ea3.html)

mac：[http://jingyan.baidu.com/article/1612d500afc297e20f1eee7f.html](http://jingyan.baidu.com/article/1612d500afc297e20f1eee7f.html)

这里说明下：跳转链接不是随便找的！我是过来人，基本都是我自己搜索过并且自己亲自实践安装配置成功了，哪些链接写得好才放在这里，大家放心跳转跟着配置即可。

配置成功后终端运行：java -version 进行测试是否安装成功。

　　![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420193839102-297833836.png)

## web服务器Tomcat
Tomcat是一个支持Servlet的web服务器，如果想在本地运行Servlet的话当然需要本地配置服务器。而Tomcat可以作为测试 Servlet 的独立服务器。

Tomcat完美安装配置教程链接：

windows：[http://blog.csdn.net/q_l_s/article/details/51736613](http://blog.csdn.net/q_l_s/article/details/51736613)

Mac：[http://blog.csdn.net/huyisu/article/details/38372663](http://blog.csdn.net/huyisu/article/details/38372663)

配置成功后测试是否安装成功：在上述教程链接中按照方法打开终端解压\bin目录下，输入startup.sh 在浏览器中输入：[http://localhost:8080/](http://localhost:8080/) 进行测试。

　　![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420193846165-561910361.png)

## Java集成开发环境IntelliJ IDEA
Java当然需要个编写代码的环境。一般我们使用的工具叫IDE（Integrated Development Environment 集成开发环境）

Java的业界用的较多的两款开发软件是Eclipse 和 IntelliJ IDEA。这里为什么要推荐IDEA呢？

原因有两点：第一就我个人经验而言，发现Eclipse在学校学习用的较多，而来公司的第一天就是自己配IntelliJ IDEA，大型工程都是用IDEA的环境，比Eclipse更强大。第二就是我发现IDEA比Eclipse好用太多，所以建议大家要是开始学Servlet的话使用IDEA，为自己当前学习和对以后的工作或者大项目都有好处。

IDEA完美安装配置教程链接：

windows系统：[http://jingyan.baidu.com/article/fdbd4277d47cfbb89e3f48f3.html](http://jingyan.baidu.com/article/fdbd4277d47cfbb89e3f48f3.html)

mac系统：[http://jingyan.baidu.com/article/5552ef47e85780518ffbc991.html](http://jingyan.baidu.com/article/5552ef47e85780518ffbc991.html)

(初次使用IDEA可能会不习惯，大家自己上网搜索如何改键成自己熟悉的编码形式)

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420193855134-1193258648.png)

## MYSQL数据库
作为后端，当然需要建立数据库存储数据，在MYSQL和SQL SEVER之间，建议大家使用MYSQL这种应用范围更为广泛的数据库（大型工程软件大部分都是用mysql），所以会SQL SEVER的同学也不要嫌麻烦，二者语法差别不大，安装配置一下MYSQL。

MYQSL完美安装配置教程链接：

windows系统：[http://jingyan.baidu.com/article/6181c3e06d6804152ef15318.html](http://jingyan.baidu.com/article/6181c3e06d6804152ef15318.html)

mac系统：[http://www.cnblogs.com/macro-cheng/archive/2011/10/25/mysql-001.html](http://www.cnblogs.com/macro-cheng/archive/2011/10/25/mysql-001.html)

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420202051509-583115557.png)

MYSQL安装完大家最好自己建库试一试。MYSQL有自带的管理工具workbench，但是推荐一款mac比较好用的mysql管理工具，Sequel Pro，只有mac版，安装比较简单，在这里不再赘述。
**至此，学习Servlet的准备工作已经完成，下面介绍Servlet基本概念以及如何建立和使用。**

# Servlet 生命周期

**一个Servlet的完整的生命周期（从创建到毁灭）包括：init()方法，service()方法，doGet()方法，doPost()方法，destroy()方法**

**init()方法** 用于 Servlet 在服务器第一次启动时被加载时，init() 方法里可简单地创建或加载一些数据，一般用的不是很多。

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420194837056-236730677.png)

**destroy() 方法** 在 Servlet 生命周期结束时被调用。用于让 Servlet 关闭数据库连接、停止后台线程等，执行类似的清理活动，一般用的也不是很多。

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420194844852-2063113438.png)

**service()方法** 是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

但service()方法由**容器**调用，所以我们不需对service()方法进行操作。

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420194914852-793480029.png)

现在来到重点标红的servlet里的doGet()和doPost()方法

从浏览器到 Web 服务器，最终到后台程序。浏览器使用两种方法可将这些信息传递到 Web 服务器，分别为 GET 方法和 POST 方法。

## **GET 方法**

页面请求发送已编码的用户信息。页面和已编码的信息中间用 ? 字符分隔表示，

如：[www.baidu.com](www.baidu.com)?key1=value1&key2=value2

## doGet()方法

doGet()方法可以处理一个 GET请求---URL 的正常请求（或者来自于一个未指定 METHOD 的 HTML 表单）。

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420195841165-1148752842.png)

## **POST 方法**

向后台程序传递信息的比较可靠的方法。POST 方法打包信息的方式与 GET 方法基本相同，但是 POST 方法不是把信息作为 URL 中 ? 字符后的文本字符串进行发送，而是把这些信息作为一个单独的消息。消息以标准输出的形式传到后台程序，您可以解析和使用这些标准输出。Servlet 使用 doPost() 方法处理这种类型的请求。

## doPost()方法

doPost() 方法处理一个POST 请求---一个特别指定了 METHOD 为 POST 的 HTML 表单。

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420200358931-1498468013.png)

我们开发servlet主要是写servlet中的doGet()、doPost()方法，来处理前端请求并返回前端所需要的数据。

下面通过实例讲述Servlet()

# Servlet实例【图文】

本篇通过图文实例给大家详细讲述如何建立一个Servlet，配置好运行环境并成功连接到MYSQL的数据库，进行数据的查询展示。

**项目创建：IDEA -> Create New Project**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420202949149-741910561.png)

**选择Project SDK（自己装的JDK版本，系统没提示的话自己选择JDK目录）,下个界面自己选择工程存储目录和工程名，我起名为DemoServlet**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420202936556-462426628.png)

**了解了文件目录，我们继续往下配置，菜单栏 ->run -> Edit Configurations 下进行配置**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420204540884-207284157.png)

**进去后点击左上角+号（注意不要选择default），添加 tomcat server->local ，配置 修改name，我这里改为了tomcat；然后Deployment里可以加个war包，点击OK**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420204718618-573066939.png)

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420204926977-69832025.png)

**配置完成以后，选择菜单栏->file->project structure ，来添加一些我们需要的包**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170420211403743-1449418131.png)

**选择Modules-> Dependencies 点击下面+号 添加 如图所示的几个包，第一个（含jsp-api和servlet-api）可以在 tomcat的解压文件夹的\lib里导入，第二个大家可以去下载下，是连接mysql的包，附上下载地址：**

[http://download.csdn.net/detail/oyuntaolianwu/5822697](http://download.csdn.net/detail/oyuntaolianwu/5822697)

注意：一定要选Export，不然可能会报错连接不上数据库

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421164732165-295383668.png)

**配置完成以后，新建我们的Servlet，选中Servlet文件夹-> 右键-> new ->Servlet，我的命名为：Servlet 和文件夹名保持一致。**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421164931274-987329511.png)

**IDEA里自带简单的数据库管理工具，在视图的最右边可以找到，Database，这里我们选择添加 Data Sourse -> MySQL**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421165036415-1969349783.png)

**这里大家要去自己的MYSQL里建立Database和User，工具为MYSQL Workbench 或者Sequel Pro，建立了后可在IDEA里点Test Connection测试是否能够成功连接。 IDEA里可以连接了数据库以后实现建表和添加数据，点开IDEA里的控制界面执行建表以及数据插入语句：**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421165132759-218599979.png)

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421165905321-1310255962.png)

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421165915118-596434136.png)

    CREATE TABLE `websites` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `name` char(20) NOT NULL DEFAULT '' COMMENT '站点名称',
     `url` varchar(255) NOT NULL DEFAULT '',
     `alexa` int(11) NOT NULL DEFAULT '0' COMMENT 'Alexa 排名',
     `country` char(10) NOT NULL DEFAULT '' COMMENT '国家',
     PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;
    
    INSERT INTO `websites` VALUES ('1', 'Google', 'https://www.google.cm/', '1', 'USA'),
       ('2', '淘宝', 'https://www.taobao.com/', '13', 'CN'),
       ('4', '微博', 'http://weibo.com/', '20', 'CN'),
       ('5', 'Facebook', 'https://www.facebook.com/', '3', 'USA');

**数据库的表和数据建好以后，我们编辑Servlet，写一个简单连接自己建立的mysql库里的一个表的例子**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421170116524-488896084.png)

    package Servlet;
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.sql.*;
    import java.io.*;
    
    /**
     * Created by weber on 2017/4/20.
     */
    @WebServlet("/Servlet")
    public class Servlet extends HttpServlet {
     private static final long serialVersionUID = 1L;
     // JDBC 驱动名及数据库 URL
     static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
     static final String DB_URL = "jdbc:mysql://localhost:3306/test?characterEncoding=utf8&useSSL=false";
    
     // 数据库的用户名与密码，需要根据自己的设置
     static final String USER = "weber";
     static final String PASS = "123456";
     protected void doPost(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
     doGet(request, response);
     }
     protected void doGet(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
    
     Connection conn = null;
     Statement stmt = null;
     // 设置响应内容类型
     response.setContentType("text/html;charset=UTF-8");
     PrintWriter out = response.getWriter();
     String title = "Servlet Mysql 测试";
     String docType = "\n";
     out.println(docType +
    	 "\n" +
    	 "" + title + "\n" +
    	 "\n" +
    	 "" + title + "\n");
    
     try{
     // 注册 JDBC 驱动器
     Class.forName(JDBC_DRIVER);
    
     // 打开一个连接
     conn = DriverManager.getConnection(DB_URL,USER,PASS);
    
     // 执行 SQL 查询
     stmt = conn.createStatement();
     String sql;
     sql = "SELECT id, name, url FROM websites " ;
     ResultSet rs = stmt.executeQuery(sql);
    
     // 展开结果集数据库
     while(rs.next()){
     // 通过字段检索
     int id  = rs.getInt("id");
     String name = rs.getString("name");
     String url = rs.getString("url");
    
     // 输出数据
     out.println("ID: " + id);
     out.println(", 站点名称: " + name);
     out.println(", 站点 URL: " + url);
     out.println("");
     }
     out.println("");
    
     // 完成后关闭
     rs.close();
     stmt.close();
     conn.close();
     } catch(SQLException se) {
     // 处理 JDBC 错误
     se.printStackTrace();
     } catch(Exception e) {
     // 处理 Class.forName 错误
     e.printStackTrace();
     }finally{
     // 最后是用于关闭资源的块
     try{
     if(stmt!=null)
     stmt.close();
     }catch(SQLException se2){
     }
     try{
     if(conn!=null)
     conn.close();
     }catch(SQLException se){
     se.printStackTrace();
     }
     }
    
     }
    }

代码附上

**这里我们在web.xml里配置Servlet的mapping,就是配置Servlet的前端路径，这样我们可以在浏览器里输入 localhost:8080/test来访问查看**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421170237837-477410673.png)

**一个Servlet基本写完了，这里我们运行project，打开浏览器输入localhost:8080/test来查看是否Servlet建立成功，并且成功连接到数据库，输出结果如下。**

![](http://images2015.cnblogs.com/blog/1129717/201704/1129717-20170421170420852-1252368822.png)

**至此，一个能成功连接到数据库的Servlet的Demo已经基本完成，下篇介绍如何将Servlet搭建到服务器上，实现和app/web的具体交互。**