---
title: Docker实践
date: 2017-09-16 21:16:00
tags: Docker,容器技术
---
## 前言
本文并非一篇Docker的入门文章, 而是面向对Docker有一定了解的开发人员, 文章不会介绍如何安装Docker, 以及什么是Docker镜像,什么是Docker容器这些概念,如果需要了解请前往[Docker官方文档](https://docs.docker.com/ "Docker官方文档")。这里我们只探讨一下Docker在开发中的实践，或者说我们怎么用Docker来帮助我们在开发中受益。

## 背景
随着NodeJS的发布，前端便像嗑了药似的疯狂发展，各种概念，框架，插件层出不穷，导致前端小伙伴压力很大（鄙人后端，前端不大专业，如有不当请包涵），别的不说，就一个前端开发环境的搭建可能就要花费很长时间，一个npm install执行半天，中途还有可能因为本机环境的原因报各种摸不着头脑的错，而且node_modules文件夹异常庞大，win下删都删不掉，经常心生骂娘冲动。因此本文就实践一下如何用Docker来帮我们解决前端开发中遇到的这些问题，最后也会简单分析下后端的Docker应用。

<!-- more -->

## 项目
话不多说，我们开始，直接上项目，码代码....  
假如我们有如下项目 dockerdemo：  
![目录结构](https://static.oschina.net/uploads/img/201610/12232138_JONp.jpg "项目目录结构")
![package.json](https://static.oschina.net/uploads/img/201610/12232247_7pcC.jpg "package.json文件")  

很明显这个项目依赖了很多插件（项目比较早，用的grunt，用gulp，webpack的小伙伴勿喷），如果是您的第一个项目的话你大概需要做以下步骤去搭建整个开发环境。
* 安装node，需要跟团队负责人确定版本信息
* 安装grunt或者gulp
* 安装bower
* 安装express
* 安装sass或less，貌似要依赖ruby
* 克隆源码
* 执行npm install
* 执行bower install
* ......  

其间还要经历各种等待，甚至出现不明不白的错误。下面我们就一步步用Docker搭建我们的前端开发环境  

### Dockerfile
分析一下不难发现，我们需要的是一个安装了node，npm的环境，如果我们能有一个预装好的系统那就最好不过了，幸运的是Docker恰好能满足我们的这个需求，去[DockerHub](https://hub.docker.com/ "Docker Hub") 搜一下就会发现官方提供了node的镜像，而且各个版本一应俱全，这样事情就好办了，开始写Dockerfile构建我们的镜像。没错我们的镜像就是基于官方node镜像。假如你本机的项目目录位于/home/xiaoqiang/projects/dockerdemo，我们在项目目录下新建一个Dockerfile文件，并添加如下代码：  

```
FROM node:4.4.7

RUN mkdir -p /home/project/dockerdemo

ADD . /home/project/dockerdemo  

WORKDIR /home/project/dockerdemo

VOLUME /home/project/dockerdemo

EXPOSE 9000 35729

RUN npm install -g grunt && npm install -g bower

CMD ["./grunt_release.sh"]
```
grunt_release.sh如下
```
#!/bin/sh

npm install
bower install --allow-root
grunt build
```
分析下上面文件的内容：  
1. 第一行，表示我们的镜像给予官方node4.4.7版本镜像  
2. 第二行，RUN指令表示在镜像构建时执行后面的命令，这里我们在镜像中创建项目目录
3. 第三行，ADD指令表示将宿主机中的文件添加到镜像中去，这里我们将本机的项目目录中的源码内容一并添加到镜像中去
4. 第四行，WORKDIR指令表示将工作目录切换到制定目录下，相当于cd，这里就是我们的项目目录
5. 第五行，VOLUME指令表示为镜像添加一个卷，而卷具有如下特点：  
    1. 卷可以在多个容器中共享数据
    2. 对于卷的修改会立即响应到宿主机或者容器中
    3. 对于卷的修改不会被提交到镜像中去
    4. ...  
这里我们指定项目目录作为卷，后面我们在运行容器的时候会指定一个本机的文件夹作为容器中卷在宿主机上存储数据的地方。
6. 第六行，EXPOSE指令表示所运行的容器会公开指定端口，同样我们在运行容器的时候会指定一个本机的端口映射到容器公开的端口上。
7. 第七行，运行安装grunt跟bower的命令，这样当我们在启动容器的时候，就可以直接使用他们了。
8. 第八行，CMD指令是指定在容器启动后默认执行的命令，该命令可以被覆盖。

好了，代码分析完了，总之大体思路就是我们把本机项目目录映射到容器中去，并且在容器中公开可以访问网页的端口，并在容器启动是执行相应命令，如安装插件，启编译，启动server等。

## 构建
Dockerfile写完了下面就是基于该文件构建镜像了，代码很简单：  
build_docker.sh
```
#!/bin/sh
sudo docker build -t dockerdemo .
```
执行该命令等待完成后，我们就可以基于该镜像运行我们的容器了，或者说我们的环境基本搭建完成了。

## 容器
运行容器的命令也很简单，代码如下：
run_docker.sh
```
sudo docker run -it --rm --name dockerdemo -v /home/xiaoqiang/projects/dockerdemo:/home/project/dockerdemo -p 9000:9000 -p 35729:35729 dockerdemo
```
1. --name指定容器的名字
2. --rm指定在容器退出后自动删除该容器
3. -v表示用本机的指定文件夹映射到容器中的卷，这里是为了保证我们的代码可以即时相应到容器中去。
4. -p表示用本机指定端口映射到容器中的端口

这样你就可以享受编码的愉快过程了，而不用头疼环境的搭建了。

## 总结
你可能觉得还得写这么多命令多么麻烦，如果这么想说明你还是没有理解其中的道理。  
1. 这些命令只需要编写一次，而不用每个进入该项目的开发人员都去编写，通常由项目搭建者写好，其他人只需要执行构建(build_docker.sh)跟运行(run_docker.sh)脚本就可以了，其他的都不需要关心。
2. 整个环境的安装中除非有网络断开的情况，否则不会报错，因为镜像是官方的，不可能有问题，你一定会一次安装成功。
3. 我们可以把容器部署到测试环境中，生产环境中，这样就可以保证我们这些环境中的运行环境是一致的，不会再说”我这里是好的呀，你哪里怎么不行呢？一定是你的问题“，换句话说，我们打包的是运行环境，这个是一致的。

## 最后
Docker技术还比较年轻，对于它的理解目前是仁者见仁，智者见智，很难说最佳实践是什么，这一套只是我的一些思考跟应用，可能有很多不合适不合理不正确的地方，还望指点，如果感觉有用还望不吝点赞~~~
