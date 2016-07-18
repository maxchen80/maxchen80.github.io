---
layout: post
title: 
---

作为git代码托管平台，[github](https://github.com/)的大名，作为程序员即使没有用过，至少也是听过的。而[coding](https://coding.net)，是一个国产的代码托管+项目和质量管理的云平台，在代码托管层面上，能提供大部分github功能的。
项目和质量管理，因为大中型公司信息安全的考量，肯定不会使用coding这样的云平台。但是作为程序员的个人学习，还是会使用到版本控制和pages等功能。目前已经有了全世界活跃的github社群，为啥还需要国产的coding呢？差异在一下几点：

* 你是否有需要免费的私有仓库/项目这样的需求？
* 你如果有pages需求，是否需要被百度蜘蛛索引？

上面任何一个问题如果答案是肯定的，那么就可以考虑使用coding了。

####构建私有仓库
访问[coding](https://coding.net)注册账户，成功后创建一个私有项目project01：
![Alt text](/assets/images/2016-03-20-coding-pages-01.jpg)

在本地创建文件夹project01，并新建一个`新建文本文档.txt`，然后讲这个目录初始化为git项目，并推送到远程仓库
```
git init
git remote add origin https://git.coding.net/{user_name}/{project_name}.git
git add .
git commit -m "commit"
git push -u origin master
```
user_name和project_name请暗宅实际情况替换；关于git命令跟详细介绍，[git - 简易指南](http://www.bootcss.com/p/git-guide/)这篇文章入门非常不错。

我们每次提交到这个创接口的远程私有仓库，肯定是需要账密来验证身份的，为了避免每次都繁琐的输入账密，我们可以使用证书：
```
$ ssh-keygen -t rsa -C "your@mail.com" -f ~/.ssh/csser-coding
```
-t rsa指定秘钥类型，-C设置注释名字，-f指定秘钥文件存储文件名，默认则为id_rsa。
一路回车确认即可，打开公钥文件.pub，复制类容新增至project01项目的SSH公钥里。

至此我们的一个私有免费仓库就已经建好了，关于这个仓库的限制，目前coding是限制最多10个项目成员可访问和最多1GB空间，对于个人而言完全足够了。


####使用coding pages
coding pages和github pages一样，提供了一个免费托管静态网页的空间，和一个免费的二级域名；也可以绑定自己的域名。主要用作用户主页和项目主页，两种主页主要的区别体现在url上，用户主页的url是{user_name}.coding.me，项目主页的url是{user_name}.coding.me/{project_name}。

首先为上面我们创建私有项目project01，申请开通Coding Pages服务，部署分支使用默认的coding-pages。
![Alt text](/assets/images/2016-03-20-coding-pages-02.jpg)

创建coding-pages分支，新建一个index.html，并推送至coding远程仓库。
```
git checkout --orphan coding-pages
git rm -rf .
git add .
git commit -m "commit"
git push -u origin coding-pages
```

访问http://{user_name}.coding.me/{project_name}，即可看见刚刚提交的内容：
![Alt text](/assets/images/2016-03-20-coding-pages-03.jpg)

用户主页的创建更简单，与项目主页的主要区别如下：
* 项目必须和{user_name}一致
* 项目专门用作Pages服务，所有简单起见可直接使用master作为部署分支

提交一个index.html到远程仓库，访问http://{user_name}.coding.me，可以看到如下的页面：
![Alt text](/assets/images/2016-03-20-coding-pages-04.jpg)










