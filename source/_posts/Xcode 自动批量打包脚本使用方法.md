---
title: Xcode 自动批量打包脚本使用方法
date: 2017-04-14 16:53:02
tags:
- Xcode
- iOS
categories:
- Tech
---

#### 所需工具
* xlcodetool.sh
> SVN地址：`svn://svn.zaijiawan.com/mengwenfeng/CommonLib/PackageTool`

#### 1.Xcode中配置

* 检查`target`的名字（名字中间不要出现空格）
  ![c7bee8c083854df7a491f4d90023d455-201704143.09.34.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/c7bee8c083854df7a491f4d90023d455-201704143.09.34.png) 


* 将`scheme`里的名字需要和`target`的名字对应
  ![62968b7b442d4048a41c3e1ac957ee2f-201704143.22.27.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/62968b7b442d4048a41c3e1ac957ee2f-201704143.22.27.png) 


* 修改target对应的info.plist名字(如 `FM收音机tian` 对应的是 `FM收音机.plist`)，需要修改三个地方，名称保持一致

        > `Build Settings`
      ![76ab7de08e5a40b18fee7cc01803ce9f-201704143.25.00.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/76ab7de08e5a40b18fee7cc01803ce9f-201704143.25.00.png) 


        > `Xcode目录`
  ![8c336d66f965409783541f6e6cd59439-201704143.26.07.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/8c336d66f965409783541f6e6cd59439-201704143.26.07.png) 


        > `文件目录`
  ![e2753db7710c48e69b6874e89c58fa7c-201704143.27.11.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/e2753db7710c48e69b6874e89c58fa7c-201704143.27.11.png) 


* 将`info.plist`的位置和工程文件`***.xcxcodeproj`在同一目录
  ![aca23d27ba6a4f7e89f633340df0b42d-201704143.28.35.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/aca23d27ba6a4f7e89f633340df0b42d-201704143.28.35.png) 


* `target`对应的证书需要手动选择，不能使用`Automatically manage signing`
  ![15b7a201aae9463ea9e2ff2680c033b0-201704143.29.37.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/15b7a201aae9463ea9e2ff2680c033b0-201704143.29.37.png) 

* 检查`Build Setting`中的证书和开发者是否选择正确
  ![371fbce1d9c7448596ec649520b0881a-201704143.33.01.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/371fbce1d9c7448596ec649520b0881a-201704143.33.01.png) 



#### 2.脚本使用

* 打开终端，将`xlcodetool.sh`拖入，再将需要打包的工程拖入，如`CustomAudio.xcodeproj`,然后键入回车

    > 注意：没有使用`cocoapods`的拖入`***.xcodeproj`,使用`cocoapods`的拖入`***.xcworkspace`
    > ![b5af421f396d4790a86d82506a6176fa-201704143.36.24.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/b5af421f396d4790a86d82506a6176fa-201704143.36.24.png) 


* 输入想打包的版本号，如需多版本打包则在版本号中间加入空格，完成后键入回车
  ![26f3c0c84c8244978137203f1414077d-201704143.38.49.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/26f3c0c84c8244978137203f1414077d-201704143.38.49.png) 


* 选择打包方式，1是选择一个`target`,2是将工程内所有`target`全部打包
  ![180651e3c95344ecb18397131d331581-201704143.41.08.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/180651e3c95344ecb18397131d331581-201704143.41.08.png) 


* 如果选择`2`则开始自动打包，选择`1`，将会出现工程内所有`target`的列表，输入想打包的`target`名(只支持单个)
  ![55c1030d8180415c8f20567783e59b63-201704143.42.44.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/55c1030d8180415c8f20567783e59b63-201704143.42.44.png) 


#### 3.完成打包

* `archive`文件将会存放在`/build/archives`中
  ![00c02bc54f4647bb99a72475ac1b1bf6-201704143.52.15.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/00c02bc54f4647bb99a72475ac1b1bf6-201704143.52.15.png) 


* `ipa`文件将会存放在`/build/ipa`中
  ![15e67253ab1c46d09fbf3008647a78dc-201704143.54.09.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/15e67253ab1c46d09fbf3008647a78dc-201704143.54.09.png) 

* 如想打出`ipa`包的名字，在`scheme`中修改
  ![17c1bd9af8de40eab8a6999dd3dcbc9a-201704143.57.44.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/17c1bd9af8de40eab8a6999dd3dcbc9a-201704143.57.44.png) 
