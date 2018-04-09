---
title: Hexo + Github 搭建博客
date: 2018-04-09 15:14:39
tags:
---
[Hexo](https://hexo.io/) 是一款基于 Node.js 的静态博客框架，支持 Github Flavored Markdown 语法，可一键部署到 Github Pages。



## 配置环境

安装 Hexo 前，需要检查是否已经安装下列应用；

* Node.js
* Git

如已安装上述必备程序，通过 `sudo npm install -g hexo-cli` 安装Hexo。



## 建站流程

1. 在 Github 中创建仓库 `username.github.io`
2. 创建本地目录 `username.github.io` 并在目录中执行 `hexo init`
4. 修改目录中的 `_config.yml` ，详见 [Hexo 配置](https://hexo.io/zh-cn/docs/configuration.html) ，如下为部署相关配置：

```安装 hexo-deployer-git，npm install hexo-deployer-git --save
deploy:
  type: git
  repo: git@github.com:User/username.github.io.git
  branch: master
```

5. 安装 hexo-deployer-git，`npm install hexo-deployer-git --save`
6. 初始化本地 git 仓库，`git init`
7. 本地仓库关联远程仓库，`git remote add origin git@github.com:User/username.github.io.git`
8. 本地创建并切换到 hexo 分支 `git checkout -b hexo` 
9. 提交本地 hexo 分支，推送并关联远程 hexo 分支


```shell
$ git add .
$ git commit -m'init hexo branch'
$ git push -u origin hexo
```

10. 将 hexo 分支设置为默认分支（使 clone 操作获取 hexo 分支内容）
11. 生成静态页面并部署至 Github，`hexo g -d`



## 写作

`hexo new [layout] <title>` 命令用于创建新文章，layout 默认为 post，可以通过修改 -config.yml 指定默认布局。

使用 layout 参数创建文章时，Hexo 会尝试在 scaffold 目录中寻找对应的模板，根据模板创建文件。

默认布局中 draft 不会显示在页面中，可通过命令 `hexo publish [layout] <title>` 发布文章。

文章上方以 `—` 分隔的区域用于指定标题等变量，如：

```markdown
title: Hello World
date: 2018/4/9 13:46:25
categories:
- tech
- java
tags:
- spring

---
```

常用参数有 title、date、updated、tags、categories、comments 等。

Hexo 中分类具有顺序性和层次性，不支持一篇文章有多个同级分类，多个类别时会成为子分类而不是并列分类。



## 使用主题

首先在 Hexo 根目录下安装主题

```shell
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

修改 hexo 根目录配置 `_config.yml`

```yaml
theme: yilia
```

调整主题配置，修改 `theme/yilia/_config.yml` ，如

```yaml
# Header

menu:
  主页: /
  随笔: /tags/随笔/

# SubNav
subnav:
  github: "#"
  weibo: "#"
  rss: "#"
  zhihu: "#"
  #qq: "#"
  #weixin: "#"
  #jianshu: "#"
  #douban: "#"

rss: /atom.xml
```


