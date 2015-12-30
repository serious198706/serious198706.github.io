---
layout: post
title: "CentOS源码编译安装gcc-4.8.0"
description: 
headline: 
modified: 2013-04-03
category: linux development
tags: [linux, c/c++]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
---


今天突然想尝试一下最新的gcc编译器，体验一下C++11的新功能。

我使用的是CentOS，目前的gcc版本是4.4.6

<pre class="prettyprint linenumbers:1">
[root@localhost ~]# gcc --version
gcc (GCC) 4.4.6 20120305 (Red Hat 4.4.6-4)
Copyright (C) 2010 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.</pre><br />

不支持C++11。<br />
<br />
尽管我知道安装GCC的过程是多么的坎坷，但为了折腾。。折腾吧。<br />
<br />
有的人要问了，你傻啊，为什么不用RPM包安装呢，为什么不用YUM包无脑安装呢。我得说，CentOS的RPM包只编译到4.4.6，YUM的也是。<br />
所以想用4.8.0，只能用源码编译安装。<br />
<br />
废话少说，开始吧。

### 一、下载gcc及依赖包。
<br />
到<a href="http://gcc.gnu.org/" target="_blank">GCC的官网</a>，找到下载链接，我倾向于选择<a href="ftp://gd.tuwien.ac.at/gnu/gcc/" target="_blank">澳大利亚镜像</a>。<br />
目录如下：

<img alt="" src="http://images.cy198706.com/Programming/QQ20130403101003.png" style="text-align: center;" />

进入**infrastructure**目录，下载下面几个依赖包（就足够了）：

<img alt="" src="http://images.cy198706.com/Programming/QQ20130403101101.png" style="text-align: center;" />
<span style="color:#b22222;">注意：关于依赖包，我建议不要去各个依赖包的官网去下载&ldquo;最新版本&rdquo;，因为编译gcc-4.8.0所需要的最完美的配置都在**infrastructure**中，使用这些依赖包不会出任何问题。</span><br />
<br />
进入**releases**目录，下载gcc-4.8.0。<br />
<br />
下载完成后，找个位置，开始解压缩吧。

<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# tar -jxvf gmp-4.3.2.tar.bz2 &amp;&amp; \
tar -jxvf mpfr-2.4.2.tar.bz2 &amp;&amp; \
tar -zxvf mpc-0.8.1.tar.gz &amp;&amp; \
tar -zxvf ppl-0.11.tar.gz &amp;&amp; \
tar -zxvf cloog-0.18.0.tar.gz &amp;&amp; \
tar -jxvf gcc-4.8.0.tar.bz2
</pre>
<br />

### 二、安装依赖包。
<br />
安装有顺序。依次是

 - gmp-4.3.2
 - mpfr-2.4.2
 - mpc-0.8.1
 - ppl-0.11
 - cloog-0.18.0
 - gcc-4.8.0



#### <span style="color:#0000cd;">1. 安装gmp-4.3.2</span>

在安装gmp时有个建议。gmp，The GNU MP Bignum Library，是一个开源的数学运算库，它可以用于任意精度的数学运算，包括有符号整数、有理数和浮点数。它本身并没有精度限制，只取决于机器的硬件情况。在*uix安装完成后，应该都会自带该库，可以在/usr/local/lib/下找到类似limgmp.so.3的文件。<br />

如果使用原版的gmp库安装gcc的话，很可能会出现编译gcc时找不到libgmp库的情况。解决这种问题的办法是安装在一个新的地方，然后在配置gcc时将gmp库指向这个新的目录。<br />

<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# cd gmp-4.3.2
[root@localhost gmp-4.3.2]# ./configure --prefix=/usr
...
[root@localhost gmp-4.3.2]# make &amp;&amp; make install
</pre>
<br />
如果没什么问题的话，应该安装很顺利。<br />

#### <span style="color:#0000cd;">2. 安装mpfr-2.4.2</span>

<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# cd mpfr-2.4.2
[root@localhost mpfr-2.4.2]# ./configure --prefix=/usr --with-gmp=/usr
...
[root@localhost mpfr-2.4.2]# make &amp;&amp; make install</pre>
<br />
从mpfr的安装开始，就必须要指定gmp的路径了。<br />
mpfr也是一个数学运算库，提供的是高精度的数学运算支持，并支持四舍五入。<br />

#### <span style="color:#0000cd;">3. 安装mpc-0.8.1</span>

<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# cd mpc-0.8.1
[root@localhost mpc-0.8.1]# ./configure --prefix=/usr --with-gmp=/usr
...
[root@localhost mpc-0.8.1]# make &amp;&amp; make install
</pre>
<br />
mpc，multiprecision，也是一个数学运算库，以提供高速度的高精度运算支持作为主要目标。<br />

#### <span style="color:#0000cd;">4. 安装ppl-0.11</span>

<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# cd ppl-0.11
[root@localhost ppl-0.11]# ./configure --prefix=/usr --with-gmp=/usr
...
[root@localhost ppl-0.11]# make &amp;&amp; make install
</pre>
<br />
ppl，The Parma Polyhedra Library，依旧是提供数学运算，不过它的主要目标是在分析和验证复杂系统时提供数值抽象化。<br />
这个编译过程有点长，10分钟左右（看你电脑配置，我有可能是因为使用虚拟机的原因）。<br />

#### <span style="color:#0000cd;">5. 安装cloog-0.18.0</span>

<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# cd cloog-0.18.0
[root@localhost cloog-0.18.0]# ./configure --prefix=/usr --with-gmp=/usr
...
[root@localhost cloog-0.18.0]# make &amp;&amp; make install
</pre>
<br />
cloog是更复杂的数学运算库。具体是干嘛的，实在是看不懂了。。有兴趣的自行Google一下吧。。<br />
<br />
<span style="color:#b22222;">注意：上面的5个依赖包完成后，要记得使用**ldconfig**命令，将所有的库链接更新一遍，防止编译gcc时找不到对应的库，尤其是libgmp。<br />
在使用完成后，验证一下是否已经配置好库链接：</span>

<pre class="prettyprint linenumbers:1">
[root@localhost gcc-4.8.0]# cd host-x86_64-unknown-linux-gnu/gcc/
[root@localhost gcc]# ldd cc1
linux-vdso.so.1 =&gt; &nbsp;(0x00007fff9c934000)
libmpc.so.2 =&gt; /usr/lib/libmpc.so.2 (0x00007fcd3503d000)
libmpfr.so.1 =&gt; /usr/lib/libmpfr.so.1 (0x00007fcd34df1000)
libgmp.so.3 =&gt; /usr/lib/libgmp.so.3 (0x00007fcd34b9c000)
libdl.so.2 =&gt; /lib64/libdl.so.2 (0x0000003ce8e00000)
libm.so.6 =&gt; /lib64/libm.so.6 (0x0000003ce9a00000)
libc.so.6 =&gt; /lib64/libc.so.6 (0x0000003ce9200000)
/lib64/ld-linux-x86-64.so.2 (0x0000003ce8a00000)
</pre>
<br />
<span style="color:#b22222;">如果上面有任意一项是not found，那就表示你的某个库没有安装好，或者没有执行**ldconfig**命令，请重新安装或者执行命令。</span>
<h4>
<span style="color:#0000cd;">6. 安装gcc-4.8.0</span></h4>
好了，终于要到重头戏了。
<pre class="prettyprint linenumbers:1">
[root@localhost gcc_src]# cd gcc-4.8.0
[root@localhost gcc-4.8.0]# ./configure --with-gmp=/usr/ --with-mpfr=/usr/ --with-mpc=/usr/ --with-cloog=/usr/ --enable-languages=c,c++ --enable-threads=posix --enable-__cxa_atexit --with-cpu=generic --disable-multilib --with-ppl=/usr/
...
</pre>
<br />

**configure**参数的选择：<br />

- **--enable-languages=c,c++**：支持的语言，有如下几种选项：
all, ada, c, c++, fortran, go, java, objc, obj-c++。如果不指明的话，除了Ada，Go和Objective-C++外，其他是默认选中的。

- **--enable-threads=posiz**：可以使用线程。有如下几种选项：

  - aix -- 支持AIX线程.

  - dce -- 支持DCE络.

  - lynx -- 支持LynxOS的线程.

  - mipssde -- 支持MIPS SDE线程.

  - no -- 与single相同.

  - posix -- 支持POSIX/Unix98通用线程.

  - rtems -- 支持RTEMS线程.

  - single -- 关闭线程支持，所有平台都通用. &nbsp;（默认）

  - tpf -- 支持TPF线程.

  - vxworks -- 支持VxWorks线程.

  - win32 -- 支持Microsoft Win32 API线程.

- **--enable__cxa_atexit**：可以使用__cxa_atexit函数来代替atexit函数。符合C++析构规则。

- **--with-cpu=generic**：使用普通模式来编译，而不是32位与64位与ARM齐上阵。

- **--disable-multilib**：禁止编译适用于多重目标体系的库。纯32位系统或纯64位系统都是NON-Multilib，但是如果有x64的U，想要既可以运行64bit的程序又可以运行32bit的程序，就得安装Multilib。64位与32位的区别还是有的，在写程序时尽量32位与64位都编译一个版本。

如果在配置过程中出现下面这个错误：<br />

<pre class="prettyprint linenumbers:1">
checking for suffix of object files... configure: error: in ` /home/serious/Desktop/gcc_src/gcc-4.8.0/x86_64-unknown-linux-gnu/libgcc&#39;:
configure: error: cannot compute suffix of object files: cannot compile
See ` config.log ' for more details.
make[2]: *** [configure-stage1-target-libgcc] Error 1
make[2]: Leaving directory `/home/serious/Desktop/gcc_src/gcc-4.8.0&#39;
make[1]: *** [stage1-bubble] Error 2
make[1]: Leaving directory `/home/serious/Desktop/gcc_src/gcc-4.8.0&#39;
make: *** [all] Error 2
</pre>
<br />


建议你进入x86_64_unknown-linux-gnu/libgcc/下，找到config.log文件，查看一下。<br />
我出现这个问题的原因是忘记使用**ldconfig**命令，以至于找不到libgmp.so.2文件。<br />

<pre class="prettyprint linenumbers:1">
[root@localhost gcc-4.8.0]# make &amp;&amp; make install
</pre>
<br />
编译过程非常非常漫长，大约有半小时左右。 一般来说，配置成功了，编译与安装就会很顺利。提心吊胆半小时过去后，一切完成。

<pre class="prettyprint linenumbers:1">
[root@localhost gcc-4.8.0]# gcc --version
gcc (GCC) 4.8.0
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions. &nbsp;There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
</pre>
<br />

### 三、测试
<br />
来玩个Lambda表达式试试看：

<pre class="prettyprint linenumbers:1">
[root@localhost cpp]# vim test.cpp
</pre>
<br />
test.cpp：

<pre class="prettyprint linenumbers:1 lang-cpp">
#include &lt;iostream&gt;
int main(int argc, char** argv)
{
        int i = 100;

        auto func = [=] { printf(&quot;%d\n&quot;, i); };

        func();

        return 0;
}
</pre>
<br />
编译执行：
<pre class="prettyprint linenumbers:1">
[root@localhost cpp]# g++ -std=c++11 test.cpp -o test
[root@localhost cpp]# ./test
100
</pre>
<br />
