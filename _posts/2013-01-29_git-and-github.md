---
layout: post
title: "Git & GitHub 合作使用"
description: 
headline: 
modified: 2013-01-29
category: development
tags: [git]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
---

### Git与GitHub

前两日，GitHub火了。

原因是这不是要临近春节了么，大批大批的归乡客开始在网上订火车票。随着技术的日新月异，和123X6网站的漏洞百出，一些强大的程序猿攻城狮开始利用这些漏洞，开发基于浏览器的抢票插件。有位仁兄把自己的研究成果上传到了世界上最大的程序源代码组织基地 -- GitHub上，结果被国内上亿人访问，几乎要把GitHub拖垮。GitHub的管理员开始以为是DDos攻击，后来才发现，原来这些访问几乎全部来自天朝。

GitHub的运维攻城狮不得不Ban掉了这个Repo，并呼吁123X6的攻城狮去掉这个Repo，否则GitHub真的可能会完蛋。

这个工程师显然忽略了两点：
1. 春运是什么。
2. 123X6是什么。

后来，123X6的人发现，大家用的抢票插件都是来自GitHub，于是伙同GFW，将GitHub墙掉了。

天朝程序员们一夜之间发现自己的代码死活都不能访问了。GitHub上拥有数量非常之多的天朝程序猿，此处已经不再是传统意义上的源代码库，在提供技术支持的同时，也为来自世界各地的程序猿们提供一个交流的平台。

所幸，这两日GitHub又恢复到了正常的状态，还得多谢李开复大哥的倾请呼吁。

Git与GitHub是两回事。Git是一个分布式文件管理工具，与SVN类似，但要比SVN性感的多。很多Linux的版本控制及代码存放，基本都是在Git上的。

原理就不多说了，用过版本控制软件的，都不会陌生。

### 使用Git管理GitHub上的项目

Git显然是可以与GitHub绑定，并使用Git来管理GitHub上的项目的。Git的基础教程也不缀述了，都是很简单的命令，Git的强大与方便，需要程序猿们在使用过程中一点点体会，并不是可以用语言来形容的。

### 注册GitHub与建立Repository

访问https://github.com，看到如下页面，在右边建立一个新的账户。
<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130128160511.jpg" />

建立完成后，就会跳到欢迎页面。页面上方有一个四格图，指导用户如何开始使用GitHub。
<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130128160555.jpg" />

我们点击右下角的New Repository，进入建立Repository页面。
<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130128160626.jpg" />

输入Repository的名称，描述，选择Public（因为Public不要钱。。），如果你即刻就想同步该Repository的话，就选中“Initialize this repository with a README”。

下方的gitignore表示在管理项目时，我们想忽略哪些文件。比如Wordpress，无论怎么修改，有些文件很可能是万年不变的，就可以在下拉菜单里找到Wordpress。那么在该Repository建立后，就会有一个.gitignore文件，内容大致如下：

<pre class="prettyprint linenums:1">
.htaccess
wp-config.php
wp-content/uploads/
wp-content/blogs.dir/
wp-content/upgrade/
wp-content/backup-db/
wp-content/advanced-cache.php
wp-content/wp-cache-config.php
sitemap.xml
*.log
wp-content/cache/
wp-content/backups/
sitemap.xml
sitemap.xml.gz
</pre>
<br />

建立完Repository，就可以开始操作代码神马的了。

还有一种方式，可以在本地初始化Repository，然后上传到GitHub。当然，前提是必须要在网站上已经建立好了对应的Repository。

Git使用ssh tunnel来提交源码，这个加密通道可以避免源码的封包被拦截而截取。因此要先产生并上传ssh key到GitHub，方便之后与服务器之间的通讯。
Linux && MAC OS下使用如下命令来生成公钥与私钥：

<pre class="prettyprint linenums:1">
# ssh-keygen -t rsa "xxx@xxx.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa.
Your public key has been saved in id_rsa.pub.
The key fingerprint is:
fb:bb:4a:0b:b4:71:d8:c9:65:d6:88:a2:fa:1e:4b:c0 xxx@xxx.com
The key's randomart image is:
// 省略
</pre>
<br />
Windows下打开git-bash，也能使用如下命令：

<pre class="prettyprint linenums:1">
ssh-keygen -t rsa -C "xxx@xxx.com"
</pre>
<br />
为了省略此后输入私钥的步骤，可以使用下面两条命令：

<pre class="prettyprint linenums:1">
# eval 'ssh-agent'
# ssh-add /home/serious/id_rsa   // private key 所在的位置
</pre>
<br />
cd到一个合适的目录，比如/home/serious，执行如下命令：

<pre class="prettyprint linenums:1">
# mkdir wordpress
# cd wordpress
# git init
// git自动生成一些文件，放在wordpress目录下
# touch README
// GitHub在显示项目时，会自动读取README文件中的内容并显示，所以我们先建立一个README文件，内容可有可无
# git add README
# git commit -m 'first commit'
# git remote add origin git@github.com:xxx/wordpress.git
# git push -u origin master
</pre>
<br />
这儿需要注意一下，官方的文档是不靠谱的，它在添加origin条目时有类似下面的命令：

<pre class="prettyprint linenums:1">
git remote add origin https://github.com/xxx/test.git
</pre>
<br />
但是会出现如下错误：

<pre class="prettyprint linenums:1">
error: The requested URL returned error: 401 while accessing https://github.com/xxx/test.git/info/refs
fatal: HTTP request failed
</pre>
<br />
git push这条命令里，-u参数加与不加都可以，如果加了，那么git会记录当前的提交路径，下次默认可以使用git push来代替git push origin master。

### 代码、文件同步

#### fork到本地：
使用Git Clone命令将刚才建立的库fork到本地。cd到一个合适的目录，比如/home/serious/GitLib，执行如下命令：

<pre class="prettyprint linenums:1">
# git clone https://github.com/xxx/test.git
Initialized empty Git repository in /home/serious/GitLib/test/.git/
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 7 (delta 1), reused 0 (delta 0)
Unpacking objects: 100% (7/7), done.
</pre>
<br />
进入test，可以看到如下目录：

<pre class="prettyprint linenums:1">
# ll -a
total 20
drwxr-xr-x.  3 root    root    4096 Jan 28 17:02 .
drwx------. 41 serious serious 4096 Jan 28 17:02 ..
drwxr-xr-x.  8 root    root    4096 Jan 28 17:02 .git
-rw-r--r--.  1 root    root       0 Jan 28 17:02 README.md
</pre>
<br />
cd到.git目录中，再次查看目录下的内容：

<pre class="prettyprint linenums:1">
# cd .git
# ll
total 52
drwxr-xr-x.  2 root root 4096 Jan 28 17:02 branches
-rw-r--r--.  1 root root  271 Jan 28 17:02 config
-rw-r--r--.  1 root root   73 Jan 28 17:02 description
-rw-r--r--.  1 root root   23 Jan 28 17:02 HEAD
drwxr-xr-x.  2 root root 4096 Jan 28 17:02 hooks
-rw-r--r--.  1 root root  184 Jan 28 17:02 index
drwxr-xr-x.  2 root root 4096 Jan 28 17:02 info
drwxr-xr-x.  3 root root 4096 Jan 28 17:02 logs
drwxr-xr-x. 11 root root 4096 Jan 28 17:02 objects
-rw-r--r--.  1 root root   94 Jan 28 17:02 packed-refs
drwxr-xr-x.  5 root root 4096 Jan 28 17:02 refs
</pre>
<br />
我们打开 config 文件，可以看到当前库的所有设置信息：

<pre class="prettyprint linenums:1">
# cat config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "origin"]
    fetch = +refs/heads/*:refs/remotes/origin/*
    url = https://github.com/xxx/test.git
[branch "master"]
    remote = origin
    merge = refs/heads/master
</pre>
<br />
如果你以后想对这个库进行同步、管理操作的话，那就需要修改一下 url 部分了：

<pre class="prettyprint linenums:1">
url = git@github.com:xxx/test.git
</pre>
<br />
#### 修改后提交：
比如我们来修改一下README.md，为其添加一行"This is created by my virtual machine!"，然后提交。

<pre class="prettyprint linenums:1">
# sed -i '$a This is created by my virtual machine!' README.md
# cat README.md
This is created by my virtual machine!
# git commit -a
</pre>
<br />
然后会转入修改COMMIT_EDITMSG文件的vi进程中：

<pre class="prettyprint linenums:1">
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   README.md
#
</pre>
<br />
我们把"modified:    README.md"前面的#去掉，保存并退出。

<pre class="prettyprint linenums:1">
# git commit -a   // 接上面
[master 717dbd1] 	modified:   README.md
 1 files changed, 1 insertions(+), 0 deletions(-)
# git push origin master
Counting objects: 5, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 303 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:xxx/test.git
    506f3b0..717dbd1  master -> master
</pre>
<br />
打开GitHub的Repository页面，发现README.md的内容已经被修改过了。 