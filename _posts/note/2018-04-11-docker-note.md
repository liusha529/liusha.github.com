---
layout: post
title: "docker 学习笔记"
date: 2018-04-11 18:25:15 +0800 
categories: 我的笔记
tag: Node
---
* content
{:toc}

`Node`docker 学习笔记

<!-- more -->

# docker 学习笔记
    
### 查看版本
docker version

### 查找镜像
docker search tutorial 

### 下载镜像
docker pull learn/tutorial (centos) 

### 容器中安装程序
docker run centos yun install ping

### 保存容器的修改
* 查看安装完的容器ID docker ps -l
* 运行 docker commit 查看该命令的参数列表
* 需要指定要提交保存的容器ID
* docker commit 333 centos/ping
 
###  测试ping
docker run centos/ping ping www.baidu.com

### 查看正在运行的容器
* 查看运行中的容器 docker ps 
* 查看详细的内容 docker inspect 容器id

### 发布自己的镜像
* docker images 命令可以列表所有安装过的镜像
* docker push 命令可以将一个镜像发布到官方网站
* 你只能将镜像发布到自己的空间下面。这个模拟器登录的是learn帐号

### 删除镜像
* 停止所有container docker stop $(docker ps -a -q)
* 删除所有container docker rm $(docker ps -a -q)
* 查看当前images 
* 通过images id 删除指定镜像 docker rmi <image id>
* 删除全部images docker rmi $(docker images -q)


### 创建镜像
1. 先下载一个容器 命令： docker pull training/sinatra
2. 然后用容器启动这个镜像 命令： docker run -t -i training/sinatra /bin/bash
3. 接下来就是给使用中的容器，添加自己需要的工具等，来组装自己的运行环境
4. 将上一步组装好的环境copy一份镜像 命令： docker commit -m “Added json gem” -a “KateSmith” \ 0b2616b0e5a8 ouruser/sinatra:v2 
 


