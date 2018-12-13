---
title: Hexo GitHub 搭建我的博客
date: 2018-11-20 23:08:15
tags: 
  - 博客搭建 
  - GitHub Pages
typora-root-url: Hexo-GitHub-搭建我的博客
---

# 前言

​	工作两年了，没有系统的学习过Java知识体系，现在对未来一年做了一个规划，搭建此博客，记录后续学习笔记，也算是为了锻炼自己的文字表达能力。

​	我曾经也用过OneNote、CSDN、博客园、写过博客，甚至自己打算写一个博客系统。当然以上都是半途而废...，我不想永远怎么颓废下去。我要开始执行我的未来规划。再半途而废，活该一辈子当码农

​	写这个开篇第一篇，就当练手了。还不太会用markdownb编辑器

​	废话不多话，为何我要搭建GitHub Pages博客？？？

# Github Pages

[Github Pages](https://pages.github.com/) 是面向用户、组织和项目开放的公共静态页面搭建托管服 务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默 认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持 自动利用 Jekyll 生成站点，也同样支持纯 HTML 文档，将你的 Jekyll 站 点托管在 Github Pages 上是一个不错的选择。

# 开始搭建Github Pages

## 1、注册属于你自己的Github账号

​	首先进入[Github](https://github.com/)站点,然后进行注册(此处不做详细说明可自行阅读[github教程：[1\]注册github](http://jingyan.baidu.com/article/455a9950abe0ada167277864.html))

注册完毕后你就拥有了自己的代码仓库了.

## 2、创建仓库

在Github首页右上角头像左侧加号点选择 New repositor(新存储库)或[点击这里](https://github.com/new)进行创建一个仓库.

![1542803416460](/1542803416460.png)

![1542804875679](/1542804875679.png)

## 3、开启Github Pages

进入设置
![1542805084668](/1542805084668.png)

找到这一块

![1542805154912](1542805154912.png)

当你的仓库名为：用户名.github.io 之后默认开启Github Pages

现在随便选择一个主题,选择上图的 Choose a theme 之后会跳转到下面这个页面

![1846640047](1846640047.png)

设置完毕后你就可以通过 username.github.io(username为你的用户名访问你的博客了)

## 

接下来就需要搭建Hexo了

# Hexo

## 准备工作

要使用Hexo,需要安装Nodejs以及Git

## 安装Node.js

[下载Node.js](https://nodejs.org/en/download/)
参考:[安装Node.js](http://www.runoob.com/nodejs/nodejs-install-setup.html)

## 安装Git

[下载Git](https://git-scm.com/download/)

一路点击Next就行了.

## 安装Hexo

在你需要安装Hexo的目录下(新建一个文件夹)右键选择 Git Bash

```
npm install hexo-cli -g   
hexo init #初始化网站   
npm install   
hexo g #生成或 hexo generate   
hexo s #启动本地服务器 或者 hexo server,这一步之后就可以通过http://localhost:4000  查看了
```

*详细命令请参考Hexo文档*

这里介绍一下怎么创建一篇文章

```
hexo new "文章名" #新建文章
hexo new page "页面名" #新建页面   
```

常用简写

```
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

新建一篇文章后就可以[预览](http://localhost:4000/)了,在hexo new之后执行一次生成hexo g再执行hexo s启动本地服务器,如果之前还在hexo s 按Ctrl + C 结束.

## 添加主题

我这里使用的主题是[next](http://theme-next.iissnan.com/)，为什么使用next，我参考了[这个博文](https://blog.csdn.net/u011475210/article/details/79023429)

写的很好

### 安装主题(next主题):

```
hexo clean
git clone https://github.com/litten/hexo-theme-next.git themes/next   
```

### 启动主题

找到目录下的_config.yml 文件,打开找到 theme：属性并设置为next

### 更新主题

```
cd themes/next
git pull
hexo g
hexo s
```

此时刷新<http://localhost:4000/>页面就能看到新的主题了.

## 使用Hexo deploy部署到github

还是编辑根目录下_config.yml文件

```
deploy:
    type: git
    repo: git@github.com:username/username.github.io.git  #这里的网址填你自己的
    branch: master   
```

保存后需要提前安装一个扩展：

```
npm install hexo-deployer-git --save   
```

接下来就是将Hexo部署到我们的Github仓库上

# 部署到Github

## 1.检查SSH keys的设置

以下命令均是在Git bash里输入

```
cd ~/.ssh
```

输入上面命令后，如果出现：

```
#ssh: No such file or directory 
```

假如你你以前有了，会如下：

```
ls
#此时会显示一些文件
mkdir key_backup
cp id_rsa* key_backup
rm id_rsa*  
#以上三步为备份和移除原来的SSH key设置
ssh-keygen -t rsa -C "邮件地址@youremail.com" #生成新的key文件,邮箱地址填你的Github地址
#Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>
#接下来会让你输入密码（这里我建议自己的笔记本电脑就不要设置密码了，直接回车就行）
```

之后就可以看到成功的界面。

## 2.添加SSH Key到Github

进入[github首页](https://github.com/)

![1542807916514](/1542807916514.png)

添加SSH Key。

![1542810809808](/1542810809808.png)

找到 系统当前用户目录下(开启查看隐藏文件) C:\Users\用户名\ .ssh id_rsa.pub文件以文本方式打开。打开之后全部复制到key中

![1542810923218](/1542810923218.png)

到了这就可以测试一下是否成功了:

```
ssh -T git@github.com
#之后会要你输入yes/no,输入yes就好了。
```

设置你的账号信息:

```
git config --global user.name "你的名字"     #真实名字不是github用户名
git config --global user.email "邮箱@邮箱.com"    #github邮箱
```

## 3.部署到github

```
hexo d
```

这时再刷新 username.github.io 就可以看到你的博客了。

到了这你以为就结束了吗？没有，还有坑没有给你们填好。

# 最后的补充

1. 电脑重装了系统/多台电脑写博客？那就赶紧戳这里[使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762)
2. 不知道如何编写Markdown语法？[Markdown——入门指南](http://www.jianshu.com/p/1e402922ee32/)
3. 想要给网站添加图片？请把图片放入根目录 *source\* 下建立一个文件夹，当你执行hexo g的时候此文件夹自动生成到public中。

# 

**hexo的next主题个性化教程:打造炫酷网站**

https://www.jianshu.com/p/f054333ac9e6

https://www.cnblogs.com/php-linux/p/8416122.html

关于Hexo6.0搭建个人博客(coding+百度-收录篇)https://www.imooc.com/article/31084?block_id=tuijian_wz