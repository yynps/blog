---
title: 教你优雅地搭建hexo博客
date: 2018-07-05 17:09:03
updated: 2018-07-05 17:09:03
comments: true
tags: 
    - hexo
    - github pages
    - travis ci
categories:
    - 环境搭建
---

使用 `github pages + hexo + travis ci` 来优雅的搭建博客。

<!-- more -->

本文主要是解决hexo在不同主机上写博客较繁琐的问题，比如在家搭好了hexo环境，想在公司电脑写写文章，那就又得重新搭建环境，git+npm+hexo，安装也是繁琐。能不能做到只关注写文章本身呢？当然是可以的，可以简单到你只需要在github上新增一篇markdown文章，稍等一会，你的博客网站也同步更新了。当然如果只是单机写文章的话，就没必要这么麻烦啦，这也算是一个hexo博客搭建的进阶教程吧

以前折腾过 word press，zblog 等来搭建博客，但总感觉这些博客系统较重，需要去弄个PHP+MySQL环境（一般都需要收费，免费的一般都不好用），现在使用 `github pages + hexo + travis ci` 就能拥有一个快速、简洁且高效的博客啦，并且 all free!!!

以下步骤是在已经存在 npm、git 等环境下进行的，如未安装环境请先搜索安装

## Github Pages

Pages服务，即一个静态空间，比如国内的[Coding Pages](https://coding.net/pages)，国外的[Github Pages](https://pages.github.com)
  
* [Coding Pages](https://coding.net/pages)现在与腾讯云合作，服务器在香港，延迟比较低，支持自定义域名与自动SSL，但最近(2019年6月)老抽风，部署的pages时不时页面打不开
  
* [Github Pages](https://pages.github.com)为github出品，github你懂的，世界上第一基友交流中心，配套的生态比较多，且在2018年5月1日之后，GitHub Pages也支持自定义域名与自动SSL了，不用自己去弄证书，延迟可能高点，但重在较稳定，由于我们要使用到travis ci，这里选用Github Pages

GitHub Pages有两种基本类型：[项目页面站点，用户和组织页面站点](https://help.github.com/cn/articles/user-organization-and-project-pages)

* 项目页面站点用于项目使用，不限制仓库名，**只能从 master，gh-pages，位于 master 分支中名为 docs 的文件夹发布**

* 用户和组织页面站点不依赖于特定项目，仓库名称必须为 *<你的github用户名>.github.io*，**只能从master分支发布**

使用不同的分支发布，配置方法有些不相同，见[配置 GitHub 页面的发布源](https://help.github.com/cn/articles/configuring-a-publishing-source-for-github-pages)

这里选用**blog仓库的gh-pages分支作为pages仓库，blog的master分支作为blog的源码分支**

## Hexo

静态博客工具有[hexo](https://hexo.io)，[jekyll](https://jekyllrb.com)，[hugo](https://gohugo.io)等，各自有各自的优点，这里选用 `hexo+NexT` 主题

### 安装[hexo-cli](https://hexo.io/zh-cn/docs/)

```bash
npm install hexo-cli -g
```

### 初始化博客

```bash
hexo init blog
cd blog
npm install
```

### 将博客使用git管理

```bash
git init
git add --all
git commit -m 'init blog'
```

### 安装主题[NexT](https://github.com/theme-next/hexo-theme-next)

由于后边我们需要修改主题配置或者自定义修改主题，我们最好将主题也加入版本控制，这样过一段时间后我们也可以知道之前修改过些什么，以后主题有更新我们可以比较方便的将更新合入，由于主题本身也是一个仓库，我们需要在我们的仓库中将另一个仓库添加进来，这里需要使用 [git submodule](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)，步骤如下

1. 在github.com将主题 [NexT](https://github.com/theme-next/hexo-theme-next) fork一份到我们自己的github仓库。因为原始主题的仓库我们一般是没有权限提交代码的，后面我们需要将主题在本地修改的代码提交到远程仓库，如果原主题有修改，我们可以将原主题的代码合并到我们的仓库，可参考[github上fork原项目，如何将本地仓库代码更新到最新版本？](https://www.cnblogs.com/eyunhua/p/8463200.html)

2. 添加submodule

   ```bash
   git submodule add https://github.com/<你的github用户名>/hexo-theme-next themes/next
   git commit -m 'add themes/next'
   ```

   *注意此处添加的主题的git路径是我们fork后自己仓库的路径*

3. 还有很多其他主题可自行选择安装，安装完此主题后可将默认主题 `themes/landscape` 删除

   ```bash
   git rm -r themes/landscape
   git commit -m 'delete themes/landscape'
   ```

### 配置hexo与主题

* 配置主题
  
  找到 `themes/next/_config.yml`，主题方面可根据自己喜好配置，可参考[我的修改](https://github.com/yynps/hexo-theme-next/commit/5371266ed7bf1394f301caf74e977e148c2c96c7)
  
  这里推荐两个生成favicon网站
  
  比较全面的favicon生成：[https://realfavicongenerator.net](https://realfavicongenerator.net)
  
  可由字母或者emoji生成favicon：[https://favicon.io](https://favicon.io)

* 安装插件
  
  `hexo-deployer-git` 用来部署文章到pages
  
  ```bash
  npm install hexo-deployer-git --save
  ```
  
  `hexo-generator-searchdb` 用来搜索文章
  
  ```bash
  npm install hexo-generator-searchdb --save
  ```
  
  其他插件可自行添加

* 自定义域名

  如果需要自定义域名，应该在 `source` 目录添加一个CNAME文件，内容为自定义的域名，防止deploy时自定义域名被覆盖，导致自定义域名失效

* 配置hexo
  
  找到根目录的 `_config.yml`，将Site与URL按自己的需要修改，部分其他修改如下
  
  ```yaml
  ...
  language: zh-CN # next低版本的中文语言是 zh-Hans
  
  timezone: Asia/Shanghai
  ...
  theme: next
  ...
  deploy:
    type: git
    repo:
      github: https://github.com/<你的github用户名>/blog,gh-pages
  ```

配置完成后，执行如下命令，然后在浏览器访问 [http://localhost:4000](http://localhost:4000) 即可预览

```bash
hexo server
```

配置并且调整完后，提交修改，步骤如下：

这里由于使用了submodule，首先提交submodule的修改

```bash
cd themes/next
git add -A
git commit -m '配置主题'
git push origin master
```

然后提交hexo的修改

```bash
cd ../..
git add -A
git commit -m '配置hexo'
```

在本地提交后，我们将需要将博客源码放到Github方便后续随时随地写博客，也方便使用travis ci，步骤如下

1. 在Github上新建一个存源码的仓库比如blog

2. 推送到远程分支

   ```bash
   git remote add origin https://github.com/<你的github用户名>/blog
   git push -u origin master
   ```

## 配置Travis CI

持续集成工具有[travis ci](https://travis-ci.org)，[circleci](https://circleci.com)等，这里选用 travis ci，具体怎么使用travis ci不多说，可参考[使用 Travis CI 部署你的 Hexo 博客](https://zhuanlan.zhihu.com/p/37014376)，原理呢就是当你的仓库有修改时比如推送，travis ci可以执行一些任务，这里重点在 `.travis.yml` 文件，在我们Github的blog仓库新建一个 `.travis.yml` 文件，内容如下

```yaml
language: node_js
node_js: --lts
cache: npm
before_script:
  - export TZ="Asia/Shanghai"
  - git config user.name "Yuan"
  - git config user.email "yuan@hicoder.cn"
  - sed -i "s~https://github.com/<你的github用户名>/blog~https://${GITHUB_ACCESS_TOKEN}@github.com/<你的github用户名>/blog~" _config.yml
script:
  - hexo g
  - hexo d
```

这里需要设置下时区，不然默认时区是UTC时间，时间会少8小时，`${GITHUB_ACCESS_TOKEN}` 为在travis ci中配置项，配置完这个，我们提交到代码到仓库就不需要 Github的密码，所以不要将token泄露了，至此配置完成

## 不同设备操作方法

* 方法一：直接使用Github页面新增文件并提交

* 方法二：使用Git客户端，由于使用了submodule，需要额外进行下操作，可用如下两种方式clone仓库

  ```bash
  git clone https://github.com/<你的github用户名>/blog
  git submodule init
  git submodule update
  ```

  或者

  ```bash
  git clone https://github.com/<你的github用户名>/blog --recursive
  ```

将代码提交后，只需稍等片刻，博客就会自动更新啦

## 其他

这里还有一种部署博客的方式 [netlify](https://www.netlify.com/) 没去研究，大概就是将博客源码存在Github，每次向Github推送代码时，netlify会将代码拉过去执行hexo的命令，然后直接将博客生成在netlify提供的pages服务上（netlify大概就像相当于travis+pages），有兴趣的可参考[Hexo+Netlify快速搭建个人博客](https://segmentfault.com/a/1190000015756932)搭建一下

## 参考链接

[Travis CI 自动部署hexo到GitHub/Coding](https://hadronw.com/2018/05-27/travis-autodeploy-github-with-coding/)

[在 hexo 中使用 git submodules 管理主题](https://juejin.im/post/5c2e22fcf265da615d72c596)

[使用 Travis CI 部署你的 Hexo 博客](https://zhuanlan.zhihu.com/p/37014376)

[github上fork原项目，如何将本地仓库代码更新到最新版本？](https://www.cnblogs.com/eyunhua/p/8463200.html)

[Hexo+Netlify快速搭建个人博客](https://segmentfault.com/a/1190000015756932)
