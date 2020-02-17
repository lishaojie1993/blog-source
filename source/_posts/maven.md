---
title: Maven相关
date: 2018-05-12 23:22:27
categories: Java
tags:
  - version control
  - maven
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## Maven简介

Maven是基于项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。

Maven是跨平台的项目管理工具。主要服务于基于Java平台的**项目构建**，**依赖管理**和项目信息管理。

## 下载安装

官方网站：http://maven.apache.org  

要求 jdk 为1.6及以上版本。

解压缩，解压目录最好不要有中文。

配置环境变量MAVEN_HOME和path。

验证是否安装成功，打开cmd窗口，输入mvn –v。

<!-- more -->

## 配置Maven

**%MAVEN_HOME%/conf/settings.xml** 是maven全局的配置文件。

该配置文件中配置了本地仓库的路径，默认就是：~/.m2/repository。

用户配置：复制maven的全局配置文件settings.xml到~/.m2目录下，即创建用户配置文件。

注意：用户级别的仓库在全局配置中一旦设置，全局配置将不再生效，转用用户所设置的仓库。

## 常用命令

### Mvn compile

执行 mvn compile命令，完成编译操作。

执行完毕后，会生成target目录，该目录中存放了编译后的字节码文件。

### Mvn clean

执行 mvn clean命令。

执行完毕后，会将target目录删除。

### Mvn test

执行 mvn test命令，完成单元测试操作。

执行完毕后，会在target目录中生成三个文件夹：surefire、surefire-reports（测试报告）、test-classes（测试的字节码文件）。

### Mvn package

执行 mvn package命令，完成打包操作。

执行完毕后，会在target目录中生成一个文件，该文件可能是jar、war。

### Mvn install

执行 mvn install命令，完成将打好的jar包安装到本地仓库的操作。

执行完毕后，会在本地仓库中出现安装后的jar包，方便其他工程引用。

## 组合命令

### mvn clean compile

cmd 中录入 mvn clean compile命令。

组合指令，先执行clean，再执行compile，通常应用于上线前执行，清除测试类。

### mvn clean test

cmd 中录入 mvn clean test命令。

组合指令，先执行clean，再执行test，通常应用于测试环节。

### mvn clean package

cmd 中录入 mvn clean package命令。

组合指令，先执行clean，再执行package，将项目打包，通常应用于发布前。

执行过程：

- 清理————清空环境
- 编译————编译源码
- 测试————测试源码
- 打包————将编译的非测试类打包

### mvn clean install

cmd 中录入 mvn clean install 查看仓库，当前项目被发布到仓库中。

组合指令，先执行clean，再执行install，将项目打包，通常应用于发布前。

执行过程：

- 清理————清空环境
- 编译————编译源码
- 测试————测试源码
- 打包————将编译的非测试类打包
- 部署————将打好的包发布到资源仓库中