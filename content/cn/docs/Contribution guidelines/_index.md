---
title: "贡献"
linkTitle: "贡献"
weight: 2
---

## 概述
我们希望 rk-docs 可以帮助用户清晰快速地了解 RK 启动器以及其他附属产品。

如果您想添加/修改文档中的内容，请您先在创建一个 issue，描述您的提案，讨论此次的修改。

在沟通交流的过程中，请授予 rk-docs 贡献者您的尊重，并且认真对待我们的[行为准则](/cn/docs/contribution-guidelines/code-of-conduct/)。

## 设置
> 目前我们没有使用类似 netlify 的服务来自动化部署此文档，所以，在做修改的时候，请遵从传统的 Git 流程。

### 1.Fork 项目
[fork](https://github.com/rookie-ninja/rk-docs/fork)

### 2.Clone 到本地
```
git clone --recurse-submodules --depth 1 https://github.com/rookie-ninja/rk-docs.git
```

### 3.安装依赖
rk-docs 使用了 [hugo](https://gohugo.io/) 来生成文档页面，并且使用了 [docsy](https://github.com/google/docsy) 模版。

因此，如果想要本地调试，我们需要安装如下依赖：

#### 安装 hugo
官方文档：[https://gohugo.io/getting-started/installing/](https://gohugo.io/getting-started/installing/)

#### 安装 Node.js
> docsy 模版使用了 Node.js，所以如果想要在本地运行 rk-docs，我们需要在本地也安装 Node.js

官方文档：[https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/)

#### 安装依赖
```shell script
sudo npm install -D --save autoprefixer
sudo npm install -D --save postcss-cli
```

### 4.运行
```shell script
$ hugo serve
```

### 5.访问
[http://localhost:1313](http://localhost:1313)

## 修改代码
创建一个新的 branch（请尽量根据功能名字来命名 branch）:

```
git checkout master
git fetch upstream
git rebase upstream/master
git checkout -b cool_new_feature
```

您可以随时通过 ($ hugo serve) 命令来验证修改。

## 提交代码
```
git push origin cool_new_feature
```

此时，您正在等待我们审核您的更改。我们**尝试**回应在几个工作日内提出问题和拉取请求，我们可能会建议一些改进或替代方案。一旦您的更改获得批准，其中项目维护者将合并它们。

如果您符合以下条件，我们更有可能批准您的更改：

- 添加测试代码。
- 写一个[好的提交信息](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)

## 资源引用
- [Docsy 用户指南](https://www.docsy.dev/docs/): 所有关于 Docsy 的信息，包括它如何管理导航、外观以及多语言支持。
- [Hugo 文档](https://gohugo.io/documentation/): Hugo 的综合参考。


