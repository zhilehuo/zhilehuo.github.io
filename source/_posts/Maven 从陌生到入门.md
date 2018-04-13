---
title: Maven 从陌生到入门
date: 2017-04-01 15:21:45
tags:
- Maven
categories:
- Tech
---

## 基本介绍
**官网：** [http://maven.apache.org/][1]

**主要功能：**

- 工程管理（工程创建、编译、测试、发布），其中工程依赖管理功能非常强大，也是我们要使用它的主要目的

**版本说明：** 

- 3.3.+： JDK1.7及以上
- 3.2.+： JDK1.6及以上

### POM (Project Object Model)
Maven的运行是基于一个XML格式的POM文件的，下面是一个简单的POM文件的例子：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zhilehuo</groupId>
    <artifactId>zlhcommon</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.4</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <distributionManagement>
        <repository>
            <id>user-deploy</id>
            <name>Releases</name>
            <url>http://bjoffice1/nexus/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>user-deploy</id>
            <name>Snapshot</name>
            <url>http://bjoffice1/nexus/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
    
</project>
```

### 中央库（Central Repository）
每个Maven项目都可以将自己发布到中央库，发布后其它人就可以在自己的项目中依赖你发布的项目。另外可以自己建私服，将项目发布至自己的私服上。私服是合法的<i class="icon-smile"></i>，后边会介绍如何设置仓库地址\[[点我直接跳过去](#setrepo)\]。

### 项目坐标（Coordinates）
- *groupId*: 组织或工程的标识，全局唯一
- *artifactId*: 项目名字
- *version*: 版本
- *packaging*: 打包格式，比如jar/war/...
- *classifier*: 类型？翻译不准确，比如一个项目打出来多个jar包，分别适配jdk1.5和jdk1.7，就可以定义两个classifier分别叫jdk5和jdk7（比如`net.sf.json`就是这么干的），还有常见的项目发布时，同时发布了source和doc包，也是通过不同的classifier来实现。

### 项目依赖（Depencencies）
Maven一个非常重要的功能就是定义项目之间的依赖关系，这给我们带来的好处就是，当我们引用一个库时，只需要在POM中配置要引用这个库即可，而不需要再去手动下载这个库依赖的其它库。
定义项目依赖时，需要用到如下参数：

- *groupId,artifactId,version:* 必填的3项，定位一个依赖
- *classifier,type:* 不一定每个项目都需要，type相当于前面的packaging
- *scope:* 定义依赖的有效范围，比如javax-servlet-api只在编译时使用，部署时不需要，那么scope就要设置为compile，以免打war包时将这个包也打进去

### <span id="deployproj">发布项目（Deploy）
我们自己的项目可以发布到Maven库中（中央库，或是私服），发布时有两种版本类型：`SNAPSHOT`版本和`Release`版本，`SNAPSHOT`就是开发中的版本，可能会被频繁更新，`Release`则是正式发布的版本，且相同版本不允许覆盖。
定义项目发布时，需要在POM中做如下配置：
```xml
<distributionManagement>
    <repository>
        <id>user-deploy</id>
        <name>Releases</name>
        <url>http://bjoffice1/nexus/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>user-deploy</id>
        <name>Snapshot</name>
        <url>http://bjoffice1/nexus/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
解释几个字段：

- *`id`:* 引用的用户认证ID，这是在settings.xml文件中配置的，下边详细说明。
- *`name`:* 名字而已，没啥用
- *`url`:* 要提交的仓库地址

发布时，如果当前版本号带有`-SNAPSHOT`后缀，则自动发布至snapshot仓库，否则就发布至release仓库。
再来看下settings.xml中的认证配置：
```xml
<servers>
  <server>
    <id>user-deploy</id>
    <username>deploy</username>
    <password>123456</password>
  </server>
</servers>
```
无需太多解释，`id`就是前边`pom.xml`引用的字段。

### Maven的配置文件
Maven常用的配置文件有两个：`setting.xml`和`pom.xml`

- `setting.xml`有一个全局的配置，和一个可选的用户自己的配置。全局的setting一般在Maven的安装目录中，比如在我的Mac中，他存在于`/Users/nianxingyan/work/develop/tools/apache-maven-3.2.5/conf/settings.xml`。该配置文件中要是关于maven的一些全局设定，比如仓库（Repository）的位置，项目部署时使用的认证信息。
- `pom.xml`一般位于每个工程的根目录下，都是关于项目本身的配置，比如坐标、依赖关系等。

#### <span id="setrepo">设置仓库位置</span>
在`setting.xml`文件中，涉及到两个设置，一个是`mirror`，用来配置一个仓库镜像地址，一个是`profile`，用来配置`Release`和`SNAPSHOT`版本使用的镜像。
先来看mirror的配置：
```xml
<mirrors>
  <mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <name>Zhilehuo Inc. Maven Central</name>
    <url>http://bjoffice1/nexus/repository/maven-public/</url>
  </mirror>
</mirrors>
```
我来试图解释一下几个字段：

- *`id`:* 镜像的标识
- *`mirrorOf`:* 可以写个特定的名字，用来后面被profile引用，也可以是`*`，意思就是全部
- *`name`:* 没啥用，说明而已
- *`url`:* 仓库的地址

再来看看prifile的配置：
```xml
<profiles>
  <profile>
      <id>development</id>
      <repositories>
          <repository>
              <id>central</id>
              <url>http://central</url>
              <releases><enabled>true</enabled></releases>
              <snapshots>
                  <enabled>true</enabled>
                  <updatePolicy>always</updatePolicy>
              </snapshots>
          </repository>
      </repositories>
      <pluginRepositories>
          <pluginRepository>
              <id>central</id>
              <url>http://central</url>
              <releases><enabled>true</enabled></releases>
              <snapshots><enabled>true</enabled></snapshots>
          </pluginRepository>
      </pluginRepositories>
  </profile>
</profiles>
```
挑重点解释一下：

- *`repositories`和`pluginRepositories`:* 分别代表`Release`和`SNAPSHOT`版本的仓库配置
- *`url`:* 这里可以写一个实际的仓库地址，也可以引用上边mirror中配置的地址。`http://central`就是引用一个叫`central`的镜像，这就是上边的`mirrorOf`字段，所以这里其实就是命中了`*`这个镜像。
- *`release.enable`和`snapshots.enable`:* 配置这个仓库中哪些类型的版本是有效的。
- *`snapshots.updatePolicy`:* `SNAPSHOT`版本的更新频率，`always`就是每次编译都检查新版本，此外还有`daily`等。`SNAPSHOT`的默认频率好像是`daily`。`Release`默认的更新频率应该是`never`，就是一旦依赖关系下载完成后，只要本地有缓存，就不会去仓库中检查更新。

此外，prifile还要配置一个activeProfile:
```xml
<activeProfiles>
  <activeProfile>development</activeProfile>
</activeProfiles>
```
这里的`development`就是刚才`profile`中的`id`字段。

-------------------------------
## 常用maven命令
maven的可执行程序叫`mvn`，命令格式为 `mvn [options] [<goal(s)>] [<phase(s)>]`。
> 我用得也不多，就简单说一下我理解到的吧，如果不对请告诉我：
> `goal`里边包含`phase`，可以理解为`phase`是`goal`的子任务。有些任务执行时会将`goal`和`phase`直接写成`goal:phase`的形式，比如`mvn jetty:run`。下边写到的几个命令应该是`goal`级别的。

`clean`

用来清理项目。mvn编译时生成的中间文件会放在`target`目录，这个命令其实就是将`target`目录清空。

`compile`

编译项目，这中间就会进行依赖关系的检查和下载。

`package`

打包项目，这个步骤包含了`compile`，并且会将当前项目打包成一个`jar`（也有可能是其它格式），放在`target`目录中。

`deploy`

发布项目，根据项目版本号来决定发布至`Release`或是`SNAPSHOT`仓库。

--------------------------------
## Nexus介绍
Nexus是一个开源的maven仓库服务，我们可以用它来搭建我们自己的“私服”。我已经在192.168.10.8上将Nexus环境搭建好了，一会可以试一下。
> Nexus中可以创建多个仓库，仓库在创建时有3种类型：
> **proxy**
> 这是一个代理，一般情况下，我们会创建一个仓库来代理Maven的中央仓库，这样我们在公司内部就可以通过Nexus来访问中央仓库了，带来的好处就是，别人在中央仓库下载过的包，下次再下载时就不用再去远程下了，节省了带宽和时间。
>
> **hosted**
> 本地仓库，相当于是一个完事的仓库，我们自己发布的项目，如果不想发布到Maven中央库，主可以发布到我们自己的`hosted`仓库中。
>
> **group**
> 这是一个仓库集合，当公司内部拥有不只一个仓库时（同时拥有`proxy`和`hosted`仓库的情况还是很常见的），为了不让客户端将所有仓库都设置一遍，就可以创建一个`group`类型的仓库，将其它的仓库都集成进去，这样客户端只需要设置这个`group`仓库就可以了。

Nexus安装后，默认创建了4个仓库：

- *maven-central*: Maven中央库的代理(`proxy`)
- *maven-release*: 用来存放私有`Release`版本的仓库(`hosted`)
- *maven-snapshot*: 用来存储私有`SNAPSHOT`版本的仓库(`hosted`)
- *maven-public*: 上边3个仓库的集合(`group`)

> **注意：** `maven-release`和`maven-snapshot`是为了管理方便，人为分出来的两个库，其实`Release`和`SNAPSHOT`版本的库是可以同时发布到一个仓库中的。

在我们的Nexus库中，我还创建了一个`3rd-party-repo`仓库，用来存放那些公共的、没有被发布至中央库的第3方库（实际上是存在这种库的，虽然目前这个库里还是空的）。同时将`3rd-party-repo`也加到了`maven-public`集合中。

### 将第三方库手动发布至Nexus
上边说到，我们自己创建了一个叫`3rd-party-repo`的仓库，用来存放第3方库，下面就来说一下具体如何操作。
**Step 1**
找到要发布的第3方库（应该就是一个Jar包），然后为它创建一个POM文件。文件名随意，因为一会发布时会指定POM文件。另外，其实maven也支持通过命令行直接发布一个Jar包，但那样的话定义依赖关系就是很麻烦，所以在没有依赖的时候，可以考虑不创建POM文件。这里以POM文件示例。
**Step 2**
编辑POM文件，需要定义的内容主要包括`坐标定义`和`依赖关系`。比如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.3rd-party</groupId>
    <artifactId>3rd-party-lib</artifactId>
    <version>1.5.3</version>

    <dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>4.1.3</version>
        </dependency>
    </dependencies>
</project>
```
**Step 3**
执行命令：
`mvn deploy:deploy-file -Dfile=[jarfile] -DpomFile=[pomfile] -Durl=http://bjoffice1/nexus/repository/3rd-party-repo/ -DrepositoryId=user-deploy`
上面参数中，`url`就是要发布到的仓库地址，`repositoryId`就是之前说到的用户认证。
> **能不能更简单一点？**
> 上面这种方式写在命令行上的参数还是有点多，所以为了简化，我试过在POM文件中像正常项目一样配置一个`distributionManagement`来定义`url`和`repositoryId`，但是失败了，似乎在部署时maven并没有去关心我配的`distributionManagement`这部分，依然还是去读了命令行里的参数。

上面提到的也可以不使用POM文件进行发布，即通过命令行指定项目坐标，命令是这样的：
`mvn deploy:deploy-file -Dfile=[jarfile] -DgroupId=com.3rd-party -DartifactId=3rd-party-lib -Dversion=1.5.3 -DrepositoryId=user-deploy -Durl=http://bjoffice1/nexus/repository/3rd-party-repo/`

### Nexus管理后台

- 登录地址：[http://bjoffice1/nexus/][2]
- 用户名：admin
- 密码：zlh123456

> - `bjoffice1` 这是一个配置在公司内网的域名，将被解释至`192.168.10.8`，Nexus就装在这台公司内网的机器上。
> - 因为Nexus被安装在了公司内网，所以南京的同事是访问不了的，如果以后南京的同事有访问需求，我们再想办法解决。

### Nexus仓库地址
| 仓库类型             | 地址                                               |
| :------------------- | :------------------------------------------------- |
| 依赖下载地址         | http://bjoffice1/nexus/repository/maven-public/    |
| 内部Release版本发布  | http://bjoffice1/nexus/repository/maven-release/   |
| 内部SNAPSHOT版本发布 | http://bjoffice1/nexus/repository/maven-snapshots/ |
| 第3方库发布          | http://bjoffice1/nexus/repository/3rd-party-repo/  |

---------------------------------
## 实际项目处理
下面我根据工作中实际遇到的一些项目类型，来讲一下如何将让现有项目支持Maven。
我先来列举一下我们工作中遇到的项目类型：`独立项目`（比如ISAS）、`公共库`（比如isasclient）、`Web项目`（比如peanut_baby）、`可独立运行的Jar包`（比如serveralarm.jar）和`Android项目`。

**注意**：公司内部的项目在定义坐标时，`groupId`统一为`com.zhilehuo`，`artifactId`全部由`小写字母`和`横线`组成，比如`isas`、`sum-dem`。

### 独立项目
在Idea中，在工程上添加Framework，选中maven，IDE会自动在工程根目录创建pom.xml，并更新目录结构：
u![cb38a785ed9a4900b4ed26102660e054-1490759582075.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/cb38a785ed9a4900b4ed26102660e054-1490759582075.png) 
![6c850ff713324d9d86ad1e4144a4bb2d-1490759618007.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/6c850ff713324d9d86ad1e4144a4bb2d-1490759618007.png) 

IDEA会主动修改源码目录结构：

- 原来的工程默认源码目录是`src`，默认测试代码目录是`test`
- 新的工程默认源码目录是`src/main/java`，测试代码目录是`src/test/java`；同时，新的工程中还多出来了一种资源目录，我现在不太知道这个目录是干嘛的，猜测是不是打包的时候，资源目录里的东西会被打进去？

之前我们的工程一般会配置一个第3方库的引用目录`lib`，改为Maven后，这个目录不再需要，可以在工程设置中删除相关配置，同时删除这个目录，以免混淆视听。
> 虽然开发阶段不再需要lib目录，但是项目部署时是需要将依赖库上传至线上服务器的，所以在部署阶段，可以将项目依赖的库拷贝出来，这可以通过一个Maven插件来自动实现。
> ```xml
> <build>
> <plugins>
>   <plugin>
>     <groupId>org.apache.maven.plugins</groupId>
>       <artifactId>maven-dependency-plugin</artifactId>
>       <executions>
>         <execution>
>           <id>copy-dependencies</id>
>           <phase>package</phase>
>           <goals>
>             <goal>copy-dependencies</goal>
>           </goals>
>           <configuration>
>             <outputDirectory>${build.directory}/lib/</outputDirectory>
>             <overWriteReleases>false</overWriteReleases>
>             <overWriteSnapshots>false</overWriteSnapshots>
>             <overWriteIfNewer>true</overWriteIfNewer>
>           </configuration>
>         </execution>
>       </executions>
>     </plugin>
>   </plugins>
> </build>
> ```

现在工程已经配置好了，编译、运行、调试和以前没有差别。

#### 导出Jar包
打包时稍有差别，之前是通过在工程配置中增加Artifact配置来实现导出jar包的，现在可以换个方式了（虽然之前的方式也能用，但是既然用了Maven，就可以抛弃之前的方式了）。
在Idea右侧栏上找到**Maven**工具，点开，在`LifeCycle`下会发现里边列出了一系列的指令，其中包括`package`：
![6c3642fbcfb94b4397bf8092f79da280-1490760761843.png](//onpyrjcca.bkt.clouddn.com//file/2017/4/6c3642fbcfb94b4397bf8092f79da280-1490760761843.png) 

双击`package`，就相当于执行了`mvn package`命令，这时Maven就会编译项目，并在target目录中生成Jar包。这个Jar包的名字是由项目坐标决定的。

### 公共库
公共库项目与独立运行的项目唯一的不同点就是需要发布。另外，公共库编译时尽量使用jdk1.6，因为依赖的工程可能是jdk1.6的。
关于项目发布的配置可以参考[项目发布](#deployproj)
> 如何使用SNAPSHOT版本？
> :    发布时，如果版本号带有后缀`-SNAPSHOT`，则发布至shapshot库。Snapshot一般是开发中的版本，可能会被频繁更新，比如被一个工程依赖时，为了修改后在那个工程中进行测试，就可以发布一个shapshot版本，然后在那个工程中依赖配置为使用shapshot：
> ```xml
> <dependency>
> <groupId>com.zhilehuo</groupId>
> <artifactId>zlhredis</artifactId>
> <version>1.0-SNAPSHOT</version>
> </dependency>
> ```

### Web项目
在独立运行项目的基础上，需要添加一个jetty插件。不过，jetty插件在运行时不认识Maven的依赖关系，所以需要将依赖的库拷贝至WEB-INF/lib目录下。最终配置是这样的：
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.eclipse.jetty</groupId>
      <artifactId>jetty-maven-plugin</artifactId>
      <version>9.2.21.v20170120</version>
      <configuration>
        <httpConnector><port>8090</port></httpConnector>
        <webAppSourceDirectory>${basedir}/WebContent/</webAppSourceDirectory>
        <webApp>
          <contextPath>/peanut_baby</contextPath>
        </webApp>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <executions>
        <execution>
          <id>copy-dependencies</id>
          <phase>compile</phase>
          <goals>
            <goal>copy-dependencies</goal>
          </goals>
          <configuration>
            <outputDirectory>${basedir}/WebContent/WEB-INF/lib/</outputDirectory>
            <overWriteReleases>false</overWriteReleases>
            <overWriteSnapshots>false</overWriteSnapshots>
            <overWriteIfNewer>true</overWriteIfNewer>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```
还要增加一个运行配置：之前启动Jetty时，配置的是Jetty运行模式，现在要修改为使用Maven来启动Jetty，如下图：
![50fef16f5ce74e4b9dfef91abe9e620f-1490761779740.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/50fef16f5ce74e4b9dfef91abe9e620f-1490761779740.png) 

之后在运行时，选择`jetty:run`，再启动运行或是调试就可以了：
![1e0fa1f3e8114c1ba73d4d6b91dd597e-1490761823137.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/1e0fa1f3e8114c1ba73d4d6b91dd597e-1490761823137.png) 

> **注意**：使用Maven后，依赖库的管理就都要在Maven的管理下，也就是说，如果要增加一个引用库时，不能直接拷贝至lib目录，而是要在Maven的依赖配置中进行配置，直接拷贝是不好使的。

> **补充说明：**
> 使用Maven管理Web项目时，为了使用Servlet和JSP，通常要在依赖中添加`servlet-api`和`jsp-api`，而在项目发布时，我们又不需要将这两个包拷贝至目标目录，所以，在引入这两个包时可以加上Scope为privoded

### 可独立运行的Jar包
和独立运行工程的区别是：独立运行工程在运行时一般还需要其它库的支持，自己无法运行；而可独立运行的Jar包，不需要其它库支持，自己即可执行，比如 serveralarm.jar。要达到这种效果，就要求在打包时，将所有依赖库一并打出到Jar包中，并修改MANIFAST文件，设置运行时的入口的类。
要完成这个任务，还要用到一个Maven插件：`maven-assembly-plugin`
先看一个配置：
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <executions>
        <execution>
          <phase>package</phase>
          <goals><goal>single</goal></goals>
        </execution>
      </executions>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.message.Send</mainClass>
          </manifest>
        </archive>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <finalName>${artifactId}-${version}-executable</finalName>
        <appendAssemblyId>false</appendAssemblyId>
        <attach>false</attach>
      </configuration>
    </plugin>
  </plugins>
</build>
```
解释一下上边的配置（只解释我可能知道的）：

- `mainClass`：设置运行时的入口类
- `descriptorRef`：定义了打包的方式，除了`jar-with-dependencies`这个好像还有`source`,`bin`啥的，总之这个就是把依赖的Jar包进行合并。
- `finalName`,`apendAssemblyId`,`attach`：有点复杂，下边单独解释。

关于`finalName`,`apendAssemblyId`,`attach`这三个标签，我来尝试解释一下吧。

首先，如果不加这3个标签，则`package`时会打出来两个包，一个是不带要依赖关系的`serveralarm-1.0.jar`，和一个带依赖关系的`serveralarm-1.0-jar-with-dependencies.jar`，而且在`deploy`时，这两个包都将上传至仓库中。如果不是因为第2个包的名字比较奇怪，那这个结果是比较理想的。

但是这个名字我不能忍……所以就发现了`finalName`这个标签，它可以给生成的包改名，于是我加了这个标签，可是结果的包名是这样的：`serveralarm-1.0-executable-jar-with-dependencies.jar`
竟然还是带着后缀，于是下一步，将`appendAssemblyId`配置为`false`，再`package`，OK，这次生成了两个包的名字是完美的，分别是`serveralarm-1.0.jar`和`serveralarm-1.0-executable.jar`。

可是，当我`deploy`的时候，却发现仓库中只有一个版本的jar包，就是`serveralarm-1.2.1.jar`，并没有`serveralarm-1.2.1-executable.jar`，而且更奇怪的是，仓库中的`serveralarm-1.2.1.jar`，并不是我刚打出来的那个，而是`serveralarm-1.2.1-executable.jar`（通过文件大小看出来的，也可以下载下来看）！

下面开始排查问题，我发现了在打包过程中有一个警告：
```accesslog
[WARNING] Configuration options: 'appendAssemblyId' is set to false, and 'classifier' is missing.
Instead of attaching the assembly file: /Users/nianxingyan/work/develop/svn/server/ServerNotification/SendSDK/target/serveralarm-1.2.1-executable.jar, it will become the file for main project artifact.
NOTE: If multiple descriptors or descriptor-formats are provided for this project, the value of this file will be non-deterministic!
[WARNING] Replacing pre-existing project main-artifact file: /Users/nianxingyan/work/develop/svn/server/ServerNotification/SendSDK/target/serveralarm-1.2.1.jar
with assembly file: /Users/nianxingyan/work/develop/svn/server/ServerNotification/SendSDK/target/serveralarm-1.2.1-executable.jar
```
人家说得很明白，用`serveralarm-1.2.1-executable.jar`替换了`serveralarm-1.2.1.jar`！为什么这样子……

百度了一顿饭的时间，说说我的理解：
`appendAssemblyId`为`true`时，相当于创建了一个名为`jar-with-dependencies`的`classifier`，而我们将其设置为`false`后，就没有这个`classifier`了，虽然我们用`finalName`修改了文件名，但Maven仍然认为这个包的`classifier`没有被设置，所以新生成的两个包的`classifier`都是空，但是`jar-with-dependences`是后生成的，所以在逻辑上覆盖了之前生成的包，这样虽然我们能看到两个文件，但是Maven只认识后生成的那个包。

如何解决呢？我试了很多办法，没有成功，最后只能设置了`attach`参数为`false`，意思就是这个包和项目没啥关系，所以项目发布的时候也不会理它。反正这个可执行的Jar包放到依赖库里也没啥用，所以就这样吧~

### Android项目（Gradle）
使用Gradle构建的Android项目，只需要修改一个仓库位置即可：
```gradle
buildscript {
  repositories {
    //mavenCentral()
    maven{ url 'http://bjoffice1/nexus/repository/maven-public/'}
  }
}
```

`dependencies`如果要加我们自己的包的话，也是一样，格式是`<groupid>:<artifactid>:<version>`，比如：
```gradle
buildscript {
  dependencies {
    classpath 'com.zhilehuo:zlhcommon:1.2.+'
  }
}
```

----------------------
## 现有项目SVN的处理
最后说一个SVN的处理。因为目录结构发生了改变，所以SVN处理起来要小心一点。处理流程建议是这样的：

1. 先`update`工程，保证本地没有修改
2. 在Idea中添加Maven Framework
3. 将Idea自动转移至新目录的源码（比如`src/main/java`）整体移动回原来的位置（比如`src`），包括测试代码
4. 在SVN中将新的目录结构进行`add`操作，原来一些没有用的目录（比如`lib`）执行`remove`
5. 在SVN中将原来的代码目录整体`move`到新的目录中
6. 提交SVN之前，记得将`target`目录设置为`ignore`

以上步骤能保证转移过去的代码还能看到历史记录。如果不用`move`指令，而是在SVN中将转移过去的代码进行`add`操作的话，历史记录就全部丢失了。

---------------------
[1]: http://maven.apache.org/ "Maven官网"
[2]: http://bjoffice1/nexus/ "Nexus管理地址"