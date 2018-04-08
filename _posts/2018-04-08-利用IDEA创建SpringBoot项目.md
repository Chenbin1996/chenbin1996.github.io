---
layout:     post
title:      利用IDEA创建SpringBoot项目
subtitle:   创建SpringBoot项目
date:       2018-04-08
author:     如漩涡
header-img: img/post-bg-miui-ux.jpg
catalog: true
tags:
    - SpringBoot
---

> 更多文章查看本人CSDN博客 [《如漩涡的博客》](https://blog.csdn.net/m0_37701381)

### 前言
IDEA是一个很好用的工具，可以创建很多类型的项目，Maven，Spring Boot等，还支持多种文件类型，前端的后台的脚本的等等

用IDEA创建Spring Boot很简单，只需要 New--New Project 选择Spring Initializr

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-04-08-3.png)

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-04-08-4.png)

接着就看你需要什么东西，比如我要一个带有Mybatis，MySql的项目，那就在这里勾上

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-04-08-5.png)

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-04-08-6.png)

之后点Next，一个全新的Spring Boot项目就创建完成了，后续需要什么依赖的话，在pom.xml中新增依赖地址就可以了
