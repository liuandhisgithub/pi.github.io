---
title:在docker上安装elasticSearch
---

> ​    本菜鸟电脑是Windows的系统，我看网上很多的docker使用教程都是在Linux系统上使用的，但是无奈笔者的电脑实在没有多余的内存也没有多余的时间安装一个虚拟机，就在Windows系统上慢慢摸索着安装docker和elasticSearch，顺便把踩得坑记录一下

## 安装docker

​	已经2021年了，docker已经支持在Windows系统上安装了，但是应该还是先安装一个小型的Linux系统然后运行在Linux虚拟机上。直接就在官网上面下载安装包安装。安装完成之后当时是启动不起来的，但是根据有一个提示，在Microsoft store里面下载了一个Ubuntu的命令行工具，我不太清楚这个是一个Linux虚拟机还是就是一个Linux风格的命令行，我个人感觉像是只是一个命令行，虚拟机应该是在docker安装包里面内置的。

​    安装好docker之后可以在我上述的命令行或者Windows的命令行里使用```docker -v```查看docker的版本。docker就可以正常的使用了。

---

---

## docker入门

​    我是第一次接触docker，其实是没有什么东西好写的，我自己都没有学会，也就在这里记录一下我目前用到的一些docker命令，之后可能会补充也可能再写一篇文章单独写docker的使用。

### 1.docker run

```
docker run -d --name name -p hostPost:containerPost imageID
```

​    这个可以运行已经拉取下来的镜像，如果本地还没有镜像文件，imageID就可以替换为需要的镜像地址和版本。

​	```-d``` 是指明在后台运行镜像程序，也可以使用-it以交互页面运行镜像。```-it```是两个独立的参数，具体什么意思我不记得了，可以用```--help```查看，我现在就知道想交互式的进入镜像就使用```-it```。也可以使用```-dit```，这样也还是后台运行，但是可以通过```docker attach name```进入容器，如果只是```-d```能不能进入我现在没有试验，先留个坑，想起来了再填。然后就是```--name```，可以给容器设定名称，也可以不设，应该会自动生成一个名称。```-p```是用来指定容器的端口和外部主机端口之间的映射。每一个容器都可以看成一个小的Linux虚拟机，麻雀虽小五脏俱全，想要访问容器内的端口，就只能通过映射将容器内端口映射到外部，这样就可以通过外部端口访问容器了，不然容器内部运行的服务外部和其他容器是无法使用的。前一个端口号是主机的端口号，后一个端口号是容器的端口号。一个容器可以使用多个```-p```参数指定端口映射

​    这个应该是我用的比较多的命令了，我也不是要学习docker，而是要学习elasticSearch，所以就疯狂的启动我下载的es镜像，不过能启动起来的次数很少。

---

### 2 docker pull

```
docker pull ubuntu
```

​    可以从docker仓库拉取镜像文件，docker仓库的性质和gitHub有点相似，上面这个命令就直接拉取ubuntu的最新版本，可以通过ubuntu:15.10指定拉取的版本，ubuntu的位置可以最好填写具体的仓库地址信息。

---

### 3 docker images

```
docker images
```

​    查看本地的docker镜像列表，我一般运行使用的imageID就是在这个命令下显示的ImageID

---

### 4 docker ps

```
docker ps
```

​	查看正在运行的docker镜像

### 5 docker start 和docker stop

```
docker start <CONTAINER ID | CONTAINER NAME> 开启docker容器
docker stop <CONTAINER ID | CONTAINER NAME> 关闭docker容器
```

​    这里的开启的容器和run的时候的参数是一致的，run选择-d，start时也会-d开启，端口映射也一致

---

---

## 运行elasticSearch

​    先使用```docker pull elasticsearch:7.7.0```下载es的镜像文件。

​    下载之后理论上使用```docker run -d --name es -p 9200:9200 -p 9300:9300 imageID```命令就可以启动了。但是我这里启动出了一些问题，有两个错误:

 ```
 ERROR: [2] bootstrap checks failed
 [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
 [2]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
 ```

​    两个错误我们一个一个解决哈，第一个是说虚拟内存太少，Linux默认给用户分配的虚拟内存只有65530，es想启动起来最少需要262144才可以，我们只用修改一下配置文件，把这个值修改一下就可以了。修改方法有两个，笔者也说过，我的电脑是Windows系统，网上给出的解决方法大多都是基于Linux系统的，但是我们之前不是安装了一个Ubuntu风格的命令行嘛，可以在那个命令行里面进行修改。

### 处理max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]错误

```
方法1：
    执行命令
    sudo sysctl -w vm.max_map_count=262144
    可以通过 sysctl -a|grep vm.max_map_count 来查看是否修改成功
方法2：
	在/etc/sysctl.conf文件最后添加一行
	vm.max_map_count=262144
注意：方法2笔者并没有成功，修改之后查看还是65530，重启命令行和重启电脑都没用，所以我最后选择第一种方法
```

### 处理the default discovery settings are unsuitable for production use错误

​    百度了一波，发现一些人选择使用修改配置文件，问题是我不知道配置文件在哪，怎么修改，遂放弃。又百度一波，发现要加```-e "discovery.type=single-node"``` ，就和下面的方法一样了，有点小无语，先这样处理吧，反正我本机测试肯定也是单节点，部署生产环境嘛，估计今年之内不用我部署哈。

​    也可以不用处理上面两个问题，```docker run -d -–name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" imageId```使用这个命令可以直接启动，不会报错，可能是因为单节点所需的内存会少一点吧。

---

---

