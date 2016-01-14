---
layout: post
title: "Bash中的条件表达式"
description: 
headline: 
modified: 2013-01-22
category: development
tags: [linux]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
--- 

每种语言都会有条件表达式，在Linux下的Bash中也不例外。

在Bash中，基本可以分为两类，一类是使用test命令，另一种是使用"[]"表达式。这两种方式的功能是等价的。

在Bash中，要注意，不要把写其他语言的习惯带到这儿来，因为Bash是通过解释来直接执行命令的，不需要编译过程，所以为了规避一些可能出现的问题，在"[]"中间，每个元素都必须用空格隔开。比如：

<pre class="prettyprint linenums:1">
[ -e /home/tom ]
[ $num -eq 2 ]
[ "$string" = "You are a genius!" ]    // 注意，等号两边须发有空格
[ "$p" != "sss" ] || [ "$p" != "ttt" ]    // 同样地，"!="两边也必须有空格
</pre>
<br />
如果写成这样：

<pre class="prettyprint linenums:1">
[ -e /home/tom]
[$num -eq 2]
[ "$string"="You are a genius!"]  // [ ]运算符将"$string"="You are a genius!"视为一个整体了
["$p"!="sss"]||["$p"!="ttt"]
</pre>
<br />
是无法正确执行的。

下面看几个运行符。

<!-- more -->

### 文件比较运算符

[ -e filename ]  如果filename存在,则为真     
        
<pre class="prettyprint linenums:1">
[ -e /var/log/syslog ]
    or
test -e /var/log/syslog
</pre>
<br />
[ -d filename ]  如果filename为目录,则为真   

<pre class="prettyprint linenums:1">       
[ -d /tmp/mydir ]
    or
test -d /tmp/mydir
</pre>
<br />
[ -f filename ]   如果filename为常规文件,则为真    

<pre class="prettyprint linenums:1">
[ -f /usr/bin/httpd ]
    or
test -f /usr/bin/httpd
</pre>
<br />
[ -L filename ]  如果filename为符号链接,则为真   

<pre class="prettyprint linenums:1"> 
[ -L /usr/bin/apache2 ]
    or
test -L /usr/bin/apache2
</pre>
<br />
[ -r filename ]   如果filename可读,则为真        

<pre class="prettyprint linenums:1">     
[ -r /var/log/syslog ]
    or
test -r /var/log/syslog
</pre>
<br />
[ -w filename ]  如果filename可写,则为真       

<pre class="prettyprint linenums:1">      
[ -w /var/haha.txt ]
    or
test -w /var/haha.txt
</pre>
<br />
[ -x filename ]  如果filename可执行,则为真     

<pre class="prettyprint linenums:1">     
[ -x /etc/init.d/mysqld ]
    or
test -x /etc/init.d/mysqld
</pre>
<br />
[ filename1 -nt filename2 ] 如果filename1比filename2新,则为真  

<pre class="prettyprint linenums:1">
[ /tmp/install/etc/services -nt /etc/services ]   // newer than
    or
test /tmp/install/etc/services -nt /etc/services
</pre>
<br />
[ filename1 -ot filename2 ] 如果filename1比filename2旧,则为真  

<pre class="prettyprint linenums:1">
[ /home/tom/test -ot /home/tom/test1 ]    // older than
    or
test /home/tom/test -ot /home/tom/test1
</pre>
<br />
[ filename1 -ef filename2 ] 判断filename1与filename2是否为同一文件，可用在hard link的判定上。主要意义在于判定两个文件是否均指向同一个inode。

<pre class="prettyprint linenums:1">
[ /usr/bin/apache2 -ef /etc/bin/apache2 ]    // equal file
    or
test /usr/bin/apache2 -ef /etc/bin/apache2
</pre>
<br />
###字符串比较运算符

(请注意引号的使用，这是防止空格扰乱代码的好方法)

[ -z string ] 如果string长度为零,则为真  

<pre class="prettyprint linenums:1">
[ -z "$myvar" ]
    or
test -z "$myvar"
</pre>
<br />
[ -n string ] 如果string长度非零,则为真  

<pre class="prettyprint linenums:1">
[ -n "$myvar" ]  // -n也可以省略
    or
test -n "$myvar"
</pre>
<br />
[ string1  = string2 ] 如果string1与string2相同,则为真  

<pre class="prettyprint linenums:1">
[ "$myvar" = "one two three" ]    // "==" 也是可以的，在bash中，进行判断时，"=" 和 "==" 的功能是相同的， 这一点跟编程语言中不太相同
                                  // 但为了区别开来，还是建议用 "=="
    or
test "$myvar" = "one two three"
</pre>
<br />
[ string1 != string2 ] 如果string1与string2不同,则为真  

<pre class="prettyprint linenums:1">
[ "$myvar" != "one two three" ]
    or
test "$myvar" != "one two three"
</pre>
<br />
### 算术比较运算符  

[ num1 -eq num2 ]  等于

<pre class="prettyprint linenums:1">
[ $mynum -eq 3 ]    // equal
    or
test $mynum -eq 3
</pre>
<br />
[ num1 -ne num2 ]  不等于

<pre class="prettyprint linenums:1">
[ $mynum -ne 3 ]    // not equal
    or
test $mynum -ne 3
</pre>
<br />
[ num1 -lt num2 ]  小于

<pre class="prettyprint linenums:1">
[ $mynum -lt 3 ]    // less than
    or
test $mynum -lt 3
</pre>
<br />
[ num1 -le num2 ]  小于或等于

<pre class="prettyprint linenums:1">
[ $mynum -le 3 ]    // less or equal
    or
test mynum -le 3
</pre>
<br />
[ num1 -gt num2 ]  大于

<pre class="prettyprint linenums:1">
[ $mynum -gt 3 ]    // greater than
    or
test $mynum -gt 3
</pre>
<br />
[ num1 -ge num2 ]  大于或等于

<pre class="prettyprint linenums:1">
[ $mynum -ge 3 ]    // greater or equal
    or
test $mynum -ge 3
</pre>
<br />

### 多重条件判定

-a 两个条件同时成立，则为真

<pre class="prettyprint linenums:1">
[ -r /home/tom/test -a -x /home/tom/test ]    // 即test文件同时具有rx权限时，才返回真
    or
test -r /home/tom/test -a -x /home/tom/test 
</pre>
<br />
-o 任意一个条件成立，则为真

<pre class="prettyprint linenums:1">
[ -r /home/tom/test -o -x /home/tom/test ]    // 即test文件具有r或x权限时，都返回真
    or
test -r /home/tom/test -o -x /home/tom/test 
</pre>
<br />
！ 条件不成立，则为真

<pre class="prettyprint linenums:1">
[ ! -x /home/tom/test ]    // 即test文件同时不具有x权限时，才返回真
    or
test ! -x /home/tom/test 
</pre>
<br />