---
title: 如何在默认安装openjdk的Linux系统中安装独立JDK
date: 2017-07-10 12:58:01
categories:
  - Java
tags:
  - Jdk
---


今天总结一下在Linux系统中卸载默认的openjdk改为我们自己独立安装的oracle的Jdk版本。就像我们在Windows上安装的环境一样。

----------
* [1.卸载&下载](#1)
* [2.解压安装](#2)
* [3.修改配置](#3)
* [4.选择使用JAVA的版本](#4)
* [5.测试](#5)


<h2 id="1"> 一、卸载&下载   </h2>
#### 1、卸载自带的opendjk

    rpm –qa|grep jdk
    rpm –e –nodeps  jdk***	//将过滤出的结果进行卸载
#### 2、下载一个Sun的JDK（现在应该叫oracle的JDK）
下一个自解压的tar包。
这里我下载的是jdk-7u79-linux-x64.tar.gz

下载地址 : [点击这里](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

<h2 id="2"> 二、解压安装 </h2>
1.解压jdk-7u79-linux-x64.tar.gz<br>

2.将解压后jdk-7u79-linux-x64.tar.gz复制到/usr/java下

<h2 id="3"> 三、修改配置 </h2>
#### 方法1：修改/etc/profile 文件
所有用户的 shell都有权使用这些环境变量
**<1> 在 shell终端执行命令：vi /etc/profile**
**<2> 在 profile文件末尾加入：**

    JAVA_HOME=/usr/jdk1.7.0_79
  	JRE_HOME=/usr/local/java/jdk1.7.0_79/jre
  	PATH=$JAVA_HOME/bin:$PATH
  	CLASSPATH=.:$JAVA_HOME/lib/dt.jar: $JAVA_HOME/lib/tools.jar
  	export JAVA_HOME,JRE_HOME,PATH,CLASSPATH

`PS：上面我们说的修改配置系统环境变量是在Unix操作系统下面的。`<p>
下面来说说在DOS系统下面如何更改配置系统环境变量。
#### DOS:

1. 在系统变量里新建JAVA_HOME变量，变量值为：`D:\others\JAVA\JDK`（根据自己的安装路径填写）

2. 新建Classpath变量，在Classpath变量（已存在不用新建）添加变量值，变量值为：
`.;%JAVA\_HOME%\lib;%JAVA_HOME%\lib\tools.jar;`

3. 在path变量（已存在不用新建）添加变量值：
`%JAVA\_HOME%\bin;%JAVA_HOME%\jre\bin（注意变量值之间用“;”隔开）`<br>
`eg:;%JAVA\_HOME%\lib;%JAVA\_HOME%\lib\tools.jar;%JAVA\_HOME%\bin;%JAVA\_HOME%\jre\bin;`

4. 测试是否配置成功
在dos中，输入命令java，回车后应该会出现java的各种命令；
javac  出现相关编译的命令；
java -version 出现jdk版本号
补充环境变量的解析:
JAVA_HOME:jdk的安装路径
Classpath:java加载类路径，只有类在classpath中java命令才能识别，在路径前加了个"."表示当前路径。
path：系统在任何路径下都可以识别java,javac命令。

**<3>重启系统**
ps: 如果你不想重新系统，可以用命令`source /etc/profile`使配置文件立即生效。否则只能重启系统才能使配置参数生效。然后我们可以通过

    echo $JAVA_HOME
    echo $PATH
    echo $CLASSPATH
查看配置的信息。
但是我最后还是重启了才真正生效。
#### 方法2：修改.bashrc文件
如果你需要给某个用户权限使用这些环境变量，你只需要修改其个人用户主目录下的.bashrc就可以了,而不像第一种方法给所有用户权限。
**<1>在 shell终端执行命令：vi ~/.bashrc**
**<2>在.bashrc文件末尾加入：**

    set JAVA_HOME=/usr/jdk1.7.0_79
    export JAVA_HOME
    set PATH=$JAVA_HOME/bin:$PATH
    export PATH
    set CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export CLASSPATH
**<3>重新登录**
`注意：Linux使用:(冒号)而不是;(分号)来分隔路径`

<h2 id="4"> 四、选择使用JAVA的版本 </h2>
1. 更新参数使配置生效<br>
	`update-alternatives --install /usr/bin/java java /usr/jdk1.7.0\_79/bin/java 300`<br>
	`update-alternatives --install /usr/bin/javac javac /usr/jdk1.7.0\_79/bin/javac 300`

2. 选择需要使用的版本
在终端输入命令：`update-alternatives –config java`

<h2 id="5"> 五、测试 </h2>
进行完如上配置后，就可以进行测试了<br>
在DOS或终端下输入 `java -version`，然后输出显示，显示出来的是当前系统JRE的最高版本
我这里显示的是：<p>

**ps：环境变量配置文件**
在Ubuntu中有如下几个文件可以设置环境变量<p>

1. /etc/profile:在登录时,操作系统定制用户环境时使用的第一个文件,此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。

2. /etc/environment:在登录时操作系统使用的第二个文件,系统在读取你自己的profile前,设置环境文件的环境变量。
3. ~/.bash_profile:在登录时用到的第三个文件是.profile文件,每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.

4. ~/.bashrc:该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。
几个环境变量的优先级
**1>2>3**
