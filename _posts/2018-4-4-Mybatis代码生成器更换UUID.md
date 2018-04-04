---
layout:     post
title:      Mybatis代码生成器更换UUID
subtitle:   代码生成器
date:       2018-04-04
author:     如漩涡
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - mybatis
---

> 更多文章查看本人CSDN博客 [《如漩涡的博客》](https://blog.csdn.net/m0_37701381)

<p>Mybatis代码生成器是默认int类型ID有SQL回显的以及update时候的自增，而表是UUID的话没法自增也没法回显，不知道mybatis代码生成器的查阅文章，Mybatis代码生成器（纯Java），需要在MybatisConfig这个类里面，更改一下配置</p>

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-4-4-3.png)

<p>改成之后，再生成代码的类main方法中，int类型改成String类型</p>

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-4-4-4.png)

<p>要是表是Int类型的就不用改了</p>