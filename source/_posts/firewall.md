---
title: CentOS 7防火墙设置-iptables
date: 2020-02-17 15:29:48
categories: System
tags: 
  - CentOS
  - FireWall
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash3mdauaj31960u04qp.jpg
---

### 关闭默认防火墙

CentOS 7默认使用的防火墙是firewall，需要将其关闭

- systemctl stop firewalld.service #停止firewall
- systemctl disable firewalld.service #禁止firewall开机启动

### 安装 iptables service

yum -y install iptables-services

### 编辑配置文件

vi /etc/sysconfig/iptables <!--more-->

### 在配置文件中增加规则

-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

### 保存退出，重启防火墙

- systemctl restart iptables.service #重启防火墙使配置生效
- systemctl enable iptables.service #设置防火墙开机启动

### iptables防火墙常用命令

- systemctl start iptables.service #打开防火墙
- systemctl stop iptables.service #关闭防火墙
- systemctl restart iptables.service #重启防火墙