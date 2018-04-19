---
title: Jetty UrlRewrite 使用
date: 2017-07-12 18:02:48
tags:
- Jetty
categories:
- 构建/部署
---

#### [Jetty下载][1]
#### [Jetty文档][2]

## UrlRewrite

> 1. 更加简洁明快，例如`http://www.zhilehuo.com/video.jsp?id=1`的网址可以使用rewrite写成`http://www.zhilehuo.com/video/1.html`
> 2. 提高安全性，有效避免一些参数名，ID等直接暴露在用户面前，同时不出    现`.jsp`、`.aspx`等字眼，能够隐藏网站开发语言
3. 有利于搜索引擎更好的抓取网站内容，搜索引擎更喜欢静态页面形式的网页，对静态页面的评分相对高于动态页面，使用UrlRewrite重写URl后更容易被搜索引擎收录


## 目录结构
| Location                   | Description     |
| -------------------------- | --------------- |
| license-eplv10-aslv20.html | Jetty的许可文件 |
|README.txt|有用的开始信息
|VERSION.txt|版本信息
|bin/|存放在Unix系统下运行的shell脚本
|demo-base/|一个可运行包含示例web应用的Jetty服务器基目录
|etc/|Jetty的配置文件
|lib/|Jetty运行所必须的jar文件
|logs/|日志
|modules/|各个模块
|notice.html|许可信息等
|resources/|包含新增到classpath配置文件夹，如log4j.properties
|start.ini|存放启动信息
|start.jar|运行Jetty的jar
|webapps/|一个用来存放运行在默认配置下的Jetty Web应用目录

OK，看完下载解压后的目录结构，进行Rewrite之前肯定先要跑一个项目看看效果。
## 运行Jetty
执行以下代码，默认在8080端口开启服务

    cd $JETTY_HOME
    java -jar start.jar

执行成功会出现以下信息

    2017-07-12 15:01:02.906:INFO::main: Logging initialized @417ms
    2017-07-12 15:01:02.956:WARN:oejs.HomeBaseWarning:main: This instance of Jetty is not running from a separate {jetty.base} directory, this is not recommended.  See documentation at http://www.eclipse.org/jetty/documentation/current/startup.html
    2017-07-12 15:01:03.126:INFO:oejs.Server:main: jetty-9.2.22.v20170606
    2017-07-12 15:01:03.144:INFO:oejdp.ScanningAppProvider:main: Deployment monitor [file:/Users/user/Desktop/jetty/webapps/] at interval 1
    2017-07-12 15:01:04.156:INFO:oejsh.ContextHandler:main: Started o.e.j.w.WebAppContext@3c756e4d{/test,file:/Users/user/Desktop/jetty/webapps/test/,AVAILABLE}{/test}
    2017-07-12 15:01:04.177:INFO:oejs.ServerConnector:main: Started ServerConnector@52a8ebc{HTTP/1.1}{0.0.0.0:8080}
    2017-07-12 15:01:04.178:INFO:oejs.Server:main: Started @1689ms

这个时候可以通过浏览器访问`http://localhost:8080` 但是这时候你会惊喜的发现你看到了一个伟大的404错误，为啥呢？因为你的`$JETTY_HOME/webapps` 目录下没有部署任何web应用。但是刚刚目录结构有提到一个`demo-base`是一个包含示例web应用的服务器基目录，那么，我们运行这个demo看看：

    cd $JETTY_HOME/demo-base/
    java -jar $JETTY_HOME/start.jar

这次运行成功之后将会看到以下信息：
这次运行成功之后将会看到以下信息：

    2017-07-12 15:11:40.083:INFO::main: Logging initialized @394ms
    2017-07-12 15:11:40.346:WARN::main: demo test-realm is deployed. DO NOT USE IN PRODUCTION!
    2017-07-12 15:11:40.352:INFO:oejs.Server:main: jetty-9.2.22.v20170606
    2017-07-12 15:11:40.372:INFO:oejdp.ScanningAppProvider:main: Deployment monitor [file:/Users/user/Desktop/jetty/demo-base/webapps/] at interval 1
    2017-07-12 15:11:40.612:WARN::main: test-jaas webapp is deployed. DO NOT USE IN PRODUCTION!
    2017-07-12 15:11:41.039:INFO:oejsh.ContextHandler:main: Started o.e.j.w.WebAppContext@343f4d3d{/test-jaas,file:/private/var/folders/7j/v6j553qx4n9d21lzcjk3w_yw0000gp/T/jetty-0.0.0.0-8080-test-jaas.war-_test-jaas-any-6566505087846391340.dir/webapp/,AVAILABLE}{/test-jaas.war}
    ......
    ......
    2017-07-12 15:11:42.152:INFO:oejs.Server:main: Started @2463ms

这次有点过分，输出好多，为了避免凑字数嫌疑，我加上了省略号
再次通过浏览器访问`http://localhost:8080` 这个时候就可以看到Jetty的欢迎页

![Jetty欢迎页][3]

好吧，用jetty开启一个服务，也看到了欢迎页了，到此结束，直接看配置Rewrite吧。

## Rewrite使用
 1. 首先直接在Jetty中的webapps中部署项目，在这里面就跟正常的使用tomcat发布web应用一样，使用Intellij IDEA的可以在` Project Structure > Artifacts` 中添加一个` :war exploded` 后，指定输出目录后，便可以得到一个直接部署的war包，放在webapps中就可以了。当然还有个粗鲁的方法就是直接找到你的项目位置，直接复制需要部署的目录文件夹，例如使用Intellij IDEA，就是项目的web文件夹复制过去就行。
 2. 在Jetty的根目录中新建一个文件夹(test_etc)存放Rewrite的配置xml文件。
 3. 开始根据项目进行编写Rewrite配置(官网和`demo-base`中的`etc/demo-rewrite-rules.xml`均可以进行参考)，后面我将会用实际代码举例。
 4. 将此xml命名为rewrite.xml放入刚刚新建的文件夹中。
 5. 为服务器添加重写模块

    ```
    cd $JETTY_HOME
    java -jar start.jar --add-to-start=rewrite
    ```
    然后你可以在Jetty根目录的`start.ini`中看到新增了一句 `--module=rewrite` ,然后在这句后面添上刚刚配置的rewrite.xml文件路径(此处例子全部是配置在Jetty的根目录的，可以直接写成`test_etc/rewrite.xml`)

### rewrite的匹配规则xml文件示例：

```
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_3.dtd">

<Configure id="Server" class="org.eclipse.jetty.server.Server">
  <Ref refid="Rewrite">
      <!-- Add rule to protect against IE ssl bug -->
      <Call name="addRule">
        <Arg>
          <New class="org.eclipse.jetty.rewrite.handler.MsieSslRule"/>
        </Arg>
      </Call>

      <!-- redirect from the welcome page to a specific page -->
      <Call name="addRule">
        <Arg>
          <New class="org.eclipse.jetty.rewrite.handler.RewritePatternRule">
            <Set name="pattern">/test.html</Set>
            <Set name="replacement">/test/world.html</Set>
          </New>
        </Arg>
      </Call> 

      <!-- reverse the order of the path sections -->
      <Call name="addRule">
        <Arg>
          <New class="org.eclipse.jetty.rewrite.handler.RewriteRegexRule">
            <Set name="regex">^/world/(\d+).html$</Set>
            <Set name="replacement">/test/world.jsp?tid=$1</Set>
          </New>
        </Arg>
      </Call>
      
      </Ref>
</Configure>
```

> xml中< set>标签中配置有`pattern` 为重写后的显示在地址栏的URl,replacement即为实际的地址
> ` regex` 便可以优雅的使用正则进行URLRewrite。
> 本例子中展示了下这两种常用的重写方式，均可正常使用。

到此，便可以看到URL实现了重写，当然如果你链接跳转访问被重写的页面，你的URL也要写成重写后的样式，不然地址栏还是显示重写前的地址，当然你直接在地址栏输入重写后的地址也是可以访问的。

> 注意：匹配重写的xml文件在`start.ini`文件中路径一定要设置正确，不然会报错not found

当然生产环境下，有可能并不会在Jetty根目录下的webapps中部署项目，这个时候就需要出现创建新的Jetty基目录。
[创建Jetty基目录][4],此处官网有演示，这时进行Rewrite的话也就是在新建的基目录文件夹中增加` start.ini`，并且也是必须在`start.ini`文件中添加rewrite模块以及配置重写规则xml文件的路径。


如下截图设置
![1.png](http://onpyrjcca.bkt.clouddn.com/69d2386a2b4d4aa9b3a830887137c12e.png)
![2.png](http://onpyrjcca.bkt.clouddn.com/40112ab2a75e404cbf64aa248016b917.png)



[1]: https://www.eclipse.org/jetty/download.html
[2]:http://www.eclipse.org/jetty/documentation/current/index.html
[3]:http://images2015.cnblogs.com/blog/936870/201609/936870-20160902101000277-826824358.png
[4]: http://www.eclipse.org/jetty/documentation/current/quickstart-running-jetty.html