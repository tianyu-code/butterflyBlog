---
title: hexo+Github搭建个人博客
date: '2020-03-26 09:47:28'
updated: '2020-03-26 09:47:29'
tags:
  - 博客
categories:
  - 博客
keywords: Hexo, Github, 搭建个人博客
cover: /images/封面图/bilibili.jpg
top_img: /images/封面图/bilibili.jpg
description: 本文介绍从零开始使用hexo和github搭建个人博客
sticky: 0
---

**声明：本文转载于如下博客，对其过程遇到的问题添加解决方法**


> <https://ryanluoxu.github.io/2017/11/24/用-Hexo-和-GitHub-Pages-搭建博客/>

+ GitHub Pages
+ Hexo 博客框架
+ 部署
+ Next主题

# 一、GitHub Page

Github Pages 其实本身就是 Github 提供的博客服务。 我们在 Github 中创建一个特定格式的 Repository，Github Pages 就会将里面的信息生成一个网页，展示出来
<!-- more -->
**操作如下**

1. 注册 Github 账号，然后在 Github 中创建一个以 .github.io 结尾的 Repository。
   1. Repository name: ryanluoxu.github.io
   2. 勾选 Initialize this repository with a README
   3. Create repository
2. 简单地编辑一下 README.md 这个文档。 比如添加：I am trying to create my own blog.. 保存(Commit changes)。
3. 打开网页：ryanluoxu.github.io 这里就可以看到 README.md 里的内容了。

如果没有太多的要求，其实直接用 README.md 来写博客也是不错的。

这个生成好的 Repository 就是用来存放博客内容的地方，也只有这个仓库里的内容，才会被 ryanluoxu.github.io 这个网页显示出来。

# 二、Hexo

Hexo 是一个博客框架。它把本地文件里的信息生成一个网页。如果不需要放在网上给别人看，就没 Github Pages 什么事了。

使用 Hexo 之前，需要先安装 Node.js 和 Git。
**操作如下：**
1. 安装 Node.js
   + 前往 https://nodejs.org/en/
   + 下载安装
   + 打开cmd，输入`node -v`
   + 输出版本信息即代表安装成功
   
2. 安装Git
   + 前往 https://git-scm.com/
   + 下载安装对应版本
   + 打开cmd，输入`git --version`
   + 输出版本信息即代表安装成功
3. <font color=red>由于国内镜像源速度慢，安装cnpm</font>
   + 使用阿里定制的 cnpm 命令行工具代替默认的 npm，输入下面代码进行安装：`$ npm install -g cnpm --registry=https://registry.npm.taobao.org`
   + 检测cnpm版本，如果安装成功可以看到cnpm的基本信息：`cnpm -v`
   + 以后安装插件只需要使用`cnpm intall`即可

4. 创建本地博客
   + 在D盘下创建文件夹 blog
   + 鼠标右键 blog，选择 Git Bash Here。 如果没有安装 Git，就不会有这个选项
   + Git Bash 打开之后，所在的位置就是 blog 这个文件夹的位置。（/d/blog）
   + 输入 `hexo init` 将 blog 文件夹初始化成一个博客文件夹。
   + 输入 npm install 安装依赖包
   + 输入 hexo g 生成（generate）网页。 由于我们还没创建任何博客，生成的网页会展示 Hexo 里面自带了一个 Hello World 的博客
   + 输入 hexo s 将生成的网页放在了本地服务器（server）
   + 浏览器里输入 http://localhost:4000/ 。 就可以看到刚才的成果了。
   + 回到 Git Bash，按 Ctrl+C 结束。
   
5. 发布一篇博客
   + 继续在 Git Bash 里，所在路径还是 /d/blog。输入`hexo new "My First Post"`
   + 在 D:\blog\source_posts 路径下，会有一个 My-First-Post.md 的文件。 编辑这个文件，然后保存
   + 回到 Git Bash，输入`hexo g` 
   + 输入`hexo s` 
   + 前往 http://localhost:4000/ 查看成果



# 三、将本地 Hexo 博客部署在 Github 上

前面两个部分，我们已经有了本地博客，和一个能托管这些资料的线上仓库。只要把本地博客部署（deploy）在我们的 Github 对应的 Repository 就可以了。

**操作如下：**
1. 获取 Github 对应的 Repository 的链接。
> 为了能够直接使用<用户名>.github.io打开博客，在github上应该建立一个名为<用户名>.github.io的代码库
   + 登陆 Github，进入到 ryanluoxu.github.io
   + 点击 Clone or download
   + 复制 URL 待用
   > 此处补充说明，部署时不使用http连接，使用ssh连接，为了之后deploy不用频繁输入密码，还需要创建ssh-key，方法如下，参考博客https://www.jianshu.com/p/7d57ce4147d3
   + 客户端生成ssh key
   `ssh-keygen -t rsa -C "youremail@example.com"`
   youremail@example.com改为自己的邮箱即可，途中会让你输入密码啥的，不需要管，一路回车即可，会生成你的ssh key。（如果重新生成的话会覆盖之前的ssh key。）
   然后再终端下执行命令:`ssh -v git@github.com`
   最后两句会出现:
   ```
   No more authentication methods to try.
   Permission denied (publickey).
   ```
   在终端再执行以下命令`ssh-agent -s`
   接着在执行`ssh-add ~/.ssh/id_rsa`
   > 若执行ssh-add ~/.ssh/id_rsa时出现这个错误:Could not open a connection to your authentication agent，则先执行如下命令即可`ssh-agent bash`
   + 配置服务端
   打开你刚刚生成的id_rsa.pub，将里面的内容复制，进入你的github账号，在settings下，SSH and GPG keys下new SSH key，然后将id_rsa.pub里的内容复制到Key中，完成后Add SSH Key
   + 验证Key
   `ssh -T git@github.com `
   提示：Hi xxx! You've successfully authenticated, but GitHub does not provide shell access. 问题就解决啦
2. 修改博客的配置文件
打开配置文件 /d/blog/_config.yml （使用 bash 里的 vi 或者 notepad++）,找到 #Deployment，填入以下内容：
```
deploy:  
  type: git  
  repository: 上面复制的URL
  branch: master
```
3. 部署
   + 回到 Git Bash，输入`npm install hexo-deployer-git --save`安装hexo的git插件
   + 输入`hexo d`，输出`INFO Deploy done: git`即为部署成功
4. 查看结果
前往<用户名>.github.io即可


# 四、使用Next主题

[更多Hexo主题](https://hexo.io/themes/)
这里以 Next 为例。 大概思路就是把整个主题的文件克隆到我们的主题文件夹中。在配置文件中注明使用该主题。

**操作如下:**
1. 在git bash输入`git clone https://github.com/iissnan/hexo-theme-next themes/next`下载
2. 修改配置文件
   + 打开 D:\blog_config.yml
   + 将`theme:`下的`lanscape`修改为`next`







      

   




   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
