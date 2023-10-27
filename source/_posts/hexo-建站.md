---
title: hexo 建站总结
date: 2023-10-22 17:59:59
categories: 
    - 博客建站
---
# Hexo 建站过程

## Prerequisite
1. 一个 Github 账号，[申请教程](https://blog.csdn.net/yaorongke/article/details/119086305)

2. 在电脑上安装 git，[git安装教程]在(https://blog.csdn.net/yaorongke/article/details/119085413)

3. 在电脑上安装 NodeJS，[NodeJS安装教程](https://blog.csdn.net/yaorongke/article/details/119084295)

## Hexo 安装和本地测试

1. 安装 hexo
``` bash
npm install -g hexo-cli
```

2. 查看 hexo 版本
```bash
hexo -v
```

3. 创建 blog
```
hexo init {YOUR_BLOG_NAME}
cd {YOUR_BLOG_NAME}
npm install
```

4. blog的本地启动
```bash
hexo g -d # generate and deploy
hexo s # server
```

5. 浏览器访问 `http://localhost:4000` 即可打开新建的博客，页面的默认风格如下
![hexo默认页面](hexo_default.png)

Hexo 的默认主题可能比较丑陋，不过hexo本身提供了很多不同的主题，比较常用（互联网上资源较多的）有 [NeXT](https://theme-next.js.org/) 和 [Fluid](https://hexo.fluid-dev.com/)
   
## Github pages部署

# Hexo 官方 Hello world
{% note info %}
这个官方的 Hello world 可以看做是一个操作手册
{% endnote %}

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)