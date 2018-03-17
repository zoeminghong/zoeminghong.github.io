---
title: 教你如何建高逼格个人网站
date: 2016-04-20
tags: code
---
> 从大学开始我就希望能有一个自己的个人网站，觉得那样真的很酷，就自学了HTML和Java编程，从此踏上了码农搬砖的不归路。。。。现如今，建一个网站的成本真的是太低了，特别是有了GitHub以后，不懂代码的孩子都可以自己建站了。好，废话就放到最后说。

<!-- more -->
### 一、Git安装配置
1、安装Git软件

2、在Github上注册一个帐号

  地址：[Github](www.github.com)
  
3、在桌面打开Git，进行设置

```shell
git config --global user.name "username"//github的帐号名
git config --global user.email "username@163.com"//github的邮箱地址
```
4、生成ssh密钥

```shell
ssh-keygen -t rsa -C "username@163.com"//github邮箱地址
```
接着会提醒你输入名字和密码，可以为空
**会在C盘的该电脑用户下面生成一个.ssh文件，其中的id_rsa和id_rsa.pub，id_rsa要好好保存，id_rsa.pub用来在github网站做配置用**

5、配置github

![github设置](http://img.blog.csdn.net/20160420182107897)

使用Add SSH key，tittle可以随意输，将id_rsa.pub中的密钥保存到这里
设置完成后，可以在本地输入

```shell
ssh -T git@github.com
```
其会将github中的公钥与本地的私钥进行匹配，成功则会返回成功信息

![成功返回](http://img.blog.csdn.net/20160420182427932)

### 二、安装Ruby
1、在安装Ruby时一定要勾选Add RubyExcutables to your Path，否则自己要配置环境变量
2、查看是否安装成功

```shell
ruby -v
```
### 三、安装devkit
### 四、将Ruby与devkit关联起来
1、在devkit安装目录下

```shell
ruby dk.rb init
```
![inti返回](http://img.blog.csdn.net/20160420182629700)

会生成一个config.yml
成功的情况下在该文件下面会有一行是关于ruby的安装路径的数据
如果失败也没事，只要通过手动输入就行了

![修改](http://img.blog.csdn.net/20160420182713024)

2、在devkit安装目录下

```shell
ruby dk.rb install
```
### 五、安装配置octopress
1、克隆octopress，切换到自己要安装的文件下

```shell
git clone git://github.com/imathis/octopress.git octopress
```
2、在octopress的文件根目录下

```shell
gen sources -a http://gems.ruby-china.org/ 
```
一个国内的软件源
移除自带的软件源，因为在国内会被墙

```shell
gem sources -r http://rubygems.org
```
查看软件源

```shell
gem sources -l
```
3、修改octopress文件下面的Gemfile文件中的source地址，也改为**http://gems.ruby-china.org/**
如果上面的命令存在执行错误，可以使用windows自带的CMD命令行去执行

4、在octopress下

```shell
gem install bundle
```
过程有点长，会有successful
接着执行

```shell
bundle install
```
过程有点长
最后

```shell
rake install
```
会生成source和public文件，source是源代码的文件，而public是生成的文件

5、编译octopress

在octopress文件下

```shell
rake generator
```
该指令会编译修改的内容，生成好的文件会在public文件下

6、运行

在octopress文件下

```shell
rake preview
```
会使用4000端口，开一个服务
使用localhost:4000访问
### 六、部署到Github网站上去
1、在Github中创建一个username.github.io的仓库，username为用户自己的Github帐号名

2、在octopress文件下

```shell
rake setup_github_pages
```
会提示你输入github中的git地址

![github地址](http://img.blog.csdn.net/20160420183249933)

会创建_deploy文件，并且跟Github绑定好了
如果上面的指令不起作用，可以使用下面的方式   

```shell
rake setup_github_pages[github中username.github.io仓库的ssh地址]
```
3、使用rake generator

4、使用rake deploy，将本地编译好的文件上传到Github中
### 七、将source目录代码上传到Github分支

```shell
git add .
git commit -m 'your message'//注释
git push origin source
```


#### ** 相关软件下载地址：[软件](http://pan.baidu.com/s/1geYwwb5) **
#### ** 学习网站：**[jekyll](http://jekyll.bootcss.com/docs/installation/)


 - 以上です(Ending)
 - ありがどう(Thank You)
