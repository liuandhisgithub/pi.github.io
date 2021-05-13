---
layout: post
title:  搭建博客时遇到的问题
tags: Other
categories: learning
---
> 博客的搭建我是根据网上的一些教程做的，这些教程都是大同小异，随便一搜一大堆，我这里就稍微说一下流程，这篇博文的重点是根据那些流程中那些地方阻碍到我了。
>
> 1.创建一个```github response```，这个项目的名称必须是```xxx.github.io```的格式
>
> 2.在项目的setting中可以选择一个官方的主题，选择后就可以通过“```githubname.github.io/xxx.github.io```”来访问网站了，我的网站是：https://liuandhisgithub.github.io/pi.github.io 
>
> 3.安装ruby环境，安装后通过```gem install jekyll bundler``` 安装```jekyll```，安装成功后就可以直接生成博客模板了
>
> 4.在clone下来的博客文件夹(```gh分支```)中使用```jekyll new . --force```命令来生成模板，生成完成后可以需要发布的markdown文件就直接放到_posts文件下，文件名遵守```yyyy-mm-dd-name.md```的格式，这样在生成网站的时候会解析前面的时间，设置博客的生成时间，name就是文章标题，也可以在markdown文件中配置，markdown文件中的配置优先级更高

### 问题一：安装jekyll失败

​    在github上已经创建好response了，但是博客不可能就一篇博文吧，emmm，我就是个萌新，看看其他用户是用jekyll来自动生成博客的模板的，我也就打算这样做。我的电脑是windows系统，之前也没安装ruby，为了可以运行jekyll，我也自己去ruby官网上安装了一个，然后用```gem install jekyll bundler```来安装jekyll，但是失败了，错误信息如下：

```
ERROR： Error installing jekyll:
	   ERROR: Failed to build gem native extension.
```

​    注意这个错误，我去百度了一下，说是我ruby安装的环境没有devkit，对的，我安装ruby的时候想着我也不用ruby做开发，就安装了withoutDEVKIT的版本，估计是这个问题，安装之后就解决了

---

---

### 注意：在那个分支上发布博客

​	现在我已经安装好了jekyll，并且把博客启动了，但是页面就很一般。之后可以通过jekyll的模板来替换一下。我也在gh分支上发布了一些文章，这一篇稍后也会提交上去，但是呢，项目默认是有main分支的，我在想在main分支提文章会不会更好一点？不过我测试了，不可以，只有在gh分支上更新的md文件才会被算作博客，显示在首页上。这里还要注意一下，使用了jekyll模板后博文的命名最好用***yyyy-mm-dd-name.markdown***这样的格式来命名。

---

---

### 问题二：点击相应博文跳转404页面

​    使用jekyll搭建博客之后我也正常发送了几篇文章，在gh分支上提交之后，在博客首页都可以看到生成了相应博客的链接，如下图：

​    但是，点击之后会显示404，我百度了一下，是配置文件(_config.yml)里面的baseUrl没有配置，将其配置成GitHub的项目名就可以了。

​	具体就是编辑项目下的_config.yml文件，修改27行的baseurl:"xxxx.github.io",然后就可以正常查看博客了。也可以修改其他的配置信息，给博客设置标题，联系方式字段。

​	而且每一篇的名称可以在文章开头设置title，这样首页显示的标题就是设置的字段。

```markdown
---
layout: post
title:  "搭建博客时遇到的问题"
---
```

---

---

​	今天遇到的问题就这些，稍后我还会看一下怎么再创建一个图床，这样我博客里的图片也可以正常显示，然后再换一个好看的主题。

​	```这次搭环境最大的困难就是github总是上不去，我的天，卡的我头疼```