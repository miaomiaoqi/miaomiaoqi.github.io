---
layout: post
title: Mac 下使用 Jekyll 和 github 搭建个人博客
categories: [Others]
description: Mac 下使用 Jekyll 和 github 搭建个人博客
keywords: github, blog
---

* content
{:toc}


## Github Page

github page是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点，更便利地是你直接从你的GitHub存储库托管。只需编辑和推送你的blog，并且你的更改是实时的

## 安装 Ruby

mac自带, 没有则使用brew安装

```bash
brew install ruby
```

查看版本号

```bash
ruby -v
```

## 安装gem

没有gem的参考以下网站:

[https://rubygems.org/pages/download](https://link.jianshu.com/?t=https%3A%2F%2Frubygems.org%2Fpages%2Fdownload)

如果安装好了gem, 建议更换为国内的源

```bash
# 查看源列表
gem sources -l
# 将源移除
gem sources --remove https://rubygems.org/
# 添加国内源
gem sources --add https://gems.ruby-china.org/
# 缓存
gem sources -u
```

输入gem –version查看版本号。对比下官网的版本。可以使用以下命令更新

```bash
sudo gem install --system
```

## 安装jekyll

```bash
sudo gem install jekyll
```

## 安装博客

首先需要安装bundler

```bash
sudo gem install bundler
```

否则会报错:

```bash
Dependency Error: Yikes! It looks like you don't have bundler or one of its dependencies installed
```

我还装了以下这些

```bash
sudo gem install jekyll-paginate
sudo gem install jekyll-gist
```

创建博客,如果没有找到jekyll命令，请查看环境变量是否配置正确, 可以用 `gem env` 命令查看 gem 包的安装目录, 然后手动配置环境变量

```bash
sudo jekyll new blog
```

安装过程会显示一堆安装的内容，最后一行:

```bash
New jekyll site installed in /Users/miaoqi/Documents/git/miaomiaoqi.github.io
```

## 本地启动博客

进入到安装目录

```bash
cd miaomiaoqi.github.io
sudo jekyll serve
```

输出:

```bash
AdmindeiMac:chaiszblog admin$ sudo jekyll serve
Configuration file: /Users/miaoqi/Documents/git/miaomiaoqi.github.io/_config.yml
            Source: /Users/miaoqi/Documents/git/miaomiaoqi.github.io
       Destination: /Users/miaoqi/Documents/git/miaomiaoqi.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.411 seconds.
 Auto-regeneration: enabled for '/Users/miaoqi/Documents/git/miaomiaoqi.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

将 http://127.0.0.1:4000 复制到浏览器打开，就可以看见了。

## 部署到 github

我的用户名为 miaomiaoqi，要按照 username.github.io 创建一个仓库所以，我建立了一个miaomiaoqi.github.io的仓库
得到地址

```http
https://github.com/miaomiaoqi/miaomiaoqi.github.io.git
```

进入到本地, 将本地的内容和github上的仓库关联, 将本地的内容推送到远端

这个时候在浏览器上输入 https://www.miaomiaoqi.github.io, 就可以看到博客了, 至此会生成一个简易的博客

## 绑定域名

在云服务上申请一个自己的域名, 做好域名解析, **这里自己找教程**

在仓库根目录添加 CNAME 文件,并将你的域名写入

将CNAME推送到远端仓库。待域名解析完成，就可以了



## 使用他人的模板

以上经过一系列繁琐的步骤我们终于搭建好了自己的博客

这里说一下比较简单的办法, 网上有许多他人写好的模板, 比自己手动生成的 jekyll 要好看许多, 我们可以下载他人的模板, 将里边的个人信息都修改成自己的, 再推送到远端即可, 如果本地不需要查看效果的话, 那么 ruby, gem... 等都不需要安装

我自己使用的是 https://github.com/mzlogin/mzlogin.github.io 这个作者的模板

博客的本质是 Jekyll, Jekyll 可以将纯文本转换为静态博客网站, 详细内容参考 [Jekyll 官网](http://jekyllcn.com/), 有兴趣的朋友可以自己编写自己的模板