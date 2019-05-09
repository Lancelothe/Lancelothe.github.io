---
title: 利用Hexo + GitHub搭建免费的个人博客
date: 2017-07-03 16:50:15
categories:
  - Hexo
tags:
  - Hexo
  - 博客
---

<p>今天给大家带来一份用Hexo + Github 搭建属于自己的个人博客的教程。其实现在这种博客的搭建方式有很多种。最早我见的是Jekyll、接下来这个Hexo，最近在研究Python发现用Flask和Django都可以搭建的，只是路数不同罢了。

我选择了这个Hexo，目前这个用的人也比较多，应该说是简单易搭建吧，可扩展也可自己美化。话不多说，下面就开始。</p>


----------
* [1.环境配置](#1)
* [2.Hexo安装部署](#2)
* [3.错误总结](#3)


<h2 id="1">1.环境配置</h2>
首先要明确有几个环境是必须有的。没有的话请先安装。

- Git（官网下载较慢，我建议用这个网址[gitb,org](http://gitb.org/)）
　作用：用来提交Hexo的内容。
- nodejs（我建议用v4.\*版本较好，官网有6.\*和8.\*的我装上后有点问题）
　作用：用来生成静态页面的。
- Github的账号（自己申请，并创建一个名为`yourname.github.io`的仓库）

自行安装完环境后，将自己 SSH key导入到申请的GitHub中。建议导入，不然后面每次提交代码都要手动输入GitHub的账号密码，相当麻烦。

<h2 id="2">2.Hexo安装部署</h2>
Hexo的安装也相当简单，以Windows为例：
<pre><code class="bash">$ cd d:/hexo
$ npm install hexo-cli -g   #  -g代表全局安装
$ hexo init blog            #  初始化blog目录
$ cd blog
$ npm install               #  安装依赖的组件
$ hexo g # 或者hexo generate
$ hexo s # 或者hexo server，可以在http://localhost:4000/ 查看</code></pre>
<b>说几个比较常用的命令:</b>

1. hexo generate (hexo g) 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
2. hexo server (hexo s) 启动本地web服务，用于博客的本地预览
3. hexo deploy (hexo d) 发布博客到远端
4. hexo clena 清除hexo g生成的本地缓存
5. hexo new "postName" #新建文章
6. hexo new page "pageName" #新建页面

<b>还有一些常用的组合</b>
hexo d -g #生成后+远程部署
hexo s -g #生成后+本地预览

现在我们来看一下本地预览的效果吧
`hexo g`
`hexo s`
　
其实这里有个问题，默认的4000端口我在Windows上一直打不开，我没有在配置文件中重新设置，所以我每次启动的时候都会用这个命令指定端口`hexo s -p 40000` 我直接加了个0，用了40000端口。
出现下面这个就说明你可以开始用网页在本地预览了。
<pre>INFO  Start processing
INFO  Hexo is running at http://localhost:40000/. Press Ctrl+C to stop.
</pre>
预览图如下：
![预览效果](/images/20170703161639.png)

<p>然后你就可以发布到你的Github上了</p>
用命令`hexo d` 即可。但是我们发现需要输入Github的账号密码，所以这里还需要配置你的Github的地址，
编辑本地的`_config.yml` 文件，这是关于Hexo的配置文件。找到如下配置项：
<pre>deploy:
  type: git
  repository: git@github.com:Lancelothe/Lancelothe.github.io.git
  branch: master
</pre>

按照我的格式填写即可，我的Github名字是Lancelothe，所以你需要将这个改为你的名字。关于type和repository的格式似乎在Hexo3.0版本以上就是需要这样写，以前好像type可以写成github.
如何查看你的Hexo版本呢？
　　<br/>用`hexo -v`命令
<pre>hexo: 3.3.7
hexo-cli: 1.0.3
os: Windows_NT 6.1.7601 win32 x64
http_parser: 2.5.2
node: 4.4.2
v8: 4.5.103.35
uv: 1.8.0
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 56.1
modules: 46
openssl: 1.0.2g
</pre>

<p>到此为止，你的Hexo个人博客就已经搭建完毕了。
下面来看看搭建过程中哪些可能遇到的问题</p>

<h2 id="3">3.错误总结</h2>
#### 问题1：npm install 报 command not found
原因：PATH配置不对。
解决：选择『计算机』-『属性』-『高级系统设置』-『环境变量』，先查看了『系统变量』部分，发现安装后确实在系统变量的Path后追加了安装路径，即：C:\Program Files\nodejs；然后，打开『用户环境变量』部分查看了下Path的值，发现在最后系统自动加入了C:\Users\s94983\AppData\Roaming\npm，在『用户环境变量』部分的Path下再追加C:\Program Files\nodejs，然后关闭掉git base，（有可能需要重启电脑才可生效）问题解决！

#### 问题2：hexo deploy时重复输入用户名密码的问题
原因：repository配置时没有采用git@github.com的形式，而是采用了老的https://github.com
解决：采用git@github.com形式即可。

#### 问题3：ERROR Deployer not found: git 或者 ERROR Deployer not found: github
原因：未安装依赖包
解决：使用 `npm install hexo-deployer-git --save` 安装依赖

#### 问题4：npm安装时下载速度慢
解决： 可以换成淘宝的镜像源。
镜像举例：
1.临时使用
`npm --registry https://registry.npm.taobao.org install express`

2.持久使用
```
npm config set registry https://registry.npm.taobao.org`
//配置后可通过下面方式来验证是否成功
npm config get registry
或
npm info express
```

3.通过cnpm
  使用
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
使用cnpm install expresstall express
```
后面还会为大家带来Hexo的高级用法的，谢谢支持。
