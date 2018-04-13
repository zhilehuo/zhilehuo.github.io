---
title: Git 操作指南
date: 2017-12-12 13:41:56
tags:
- Git
categories:
- Tech
---
## 安装git

### Linux

	yum install git 

### Mac OS

[下载地址](https://git-scm.com/downloads)

## 让仓库忽略一些文件

编辑 .gitignore 文件

	target/
	!.mvn/wrapper/maven-wrapper.jar
	
	### STS ###
	.apt_generated
	.classpath
	.factorypath
	.project
	.settings
	.springBeans
	
	### IntelliJ IDEA ###
	.idea
	*.iws
	*.iml
	*.ipr
	
	### NetBeans ###
	nbproject/private/
	build/
	nbbuild/
	dist/
	nbdist/
	.nb-gradle/% 

## 在git服务器增加用户

如果没有公钥，生成一个:
	
	ssh-keygen -t rsa -P ''

将ssh公钥内容发给管理员。

	cat ~/.ssh/id_rsa.pub

## 查看自己可访问的项目列表

	ssh git@git.zaijiawan.com info

执行上面的命令，你会看到：
	
	hello xiaokaiqun, this is git@server6 running gitolite3 v3.6.7-18-g3e0c51e on git 1.7.1
	
	 R W	gitolite-admin
	 R W	hospital-crm
	 R W	testing

其中`gitolite-admin`、`hospital-crm`、`testing`为我能访问的三个项目。访问权限是可读、可写。

## 从服务端clone一个项目到本地

	git clone git@git.zaijiawan.com:hospital-crm

## 在本地配置远程库地址

	git remote add server git@git.zaijiawan.com:hospital-crm

查看远程库信息：
	
	git remote -v

执行后你会看到：

	server	git@git.zaijiawan.com:hospital-crm (fetch)
	server	git@git.zaijiawan.com:hospital-crm (push)

## 把本地仓库的内容推送到远程仓库

	git push server master //server 是远程库名称，master是本地分支名

## 推送时出现冲突

说明有人修改过这个分支，请先将最新的内容拉取下来，并在本地解决冲突。

	git pull
	如果失败，先关联本地与远程分支 git branch --set-upstream dev origin/<branch>

解决冲突后，再次push即可

## 在服务器上新建一个工程

修改并提交gitolite-admin仓库里的conf/gitolite.conf配置文件即可。前提是你需要成为gitolite的管理员。

## 使用客户端

我用了总工说的：GitKraken
![gitkraken.png](http://onpyrjcca.bkt.clouddn.com/3d7a13efeda04631b433b47069cc5537.png)

## 一些建议

1. 按功能点提交，git log很清晰
2. 功能用"feature xxx"格式，修改bug用 "bug-fix xxx"格式 