---
layout: post
title: "docker 学习笔记"
date: 2018-04-11 23:50:15 +0800 
categories: 我的笔记
tag: Node
---
* content
{:toc}

`Node`docker 学习笔记

<!-- more -->

# centos docker安装步骤

* 查看系统版本 
*   uname -a  cat /etc/redhat-release
* 安装EPEL 因为系统自带的repo中不带docker需要安装epel
*   rpm -Uvh http://ftp.riken.jp/Linux/fedora/epel/6Server/x86_64/epel-release-6-8.noarch.rpm
* 安装Docker 
*   yum install -y docker-io
* 开机自启动与启动Docker
*   service docker start
*   chkconfig docker on
*   chkconfig docker --list



