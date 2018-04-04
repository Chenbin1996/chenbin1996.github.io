---
layout:     post
title:      关于List中addAll()方法的错误纠正
subtitle:   使用不当引发的错误
date:       2018-04-04
author:     如漩涡
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - Java
---

> 更多文章查看本人CSDN博客 [《如漩涡的博客》](https://blog.csdn.net/m0_37701381)

##起因
<p>  在项目中构造树形关系的时候，一级下面挂两个二级，比如说浙江省下面挂着归属浙江省的市，但还要挂一个浙江省所在的人员，人员和市就同样是二级关系，项目里有一个TreeNode类，专门用来构建树的返回结果，TreeNode有个setChild用于加入下一级的List</p>

##思路
<p>  先查出所有一级信息，根据这个一级的size进行循环，在这个循环内做查询，查和一级有关的二级信息，做两次，一次查人员，一次查市区，用List的addAll方法，将两个合并，在放入setChild中</p>

##错误
<p>  本来应该是两个二级中的其中某一个addAll()，例如人员.addAll(市区)，或者市区.addAll(人员)，然后浙江省.get(i).setChild(人员或者市区)，但是我午睡起来后有点懵逼。。写成了浙江省.addAll(人员)，然后做了setChild的操作，导致了第一级的size扩充变大了，一直在那里跑循环，把我整懵了，这才一级二级，有这么多数据吗，后来才发现是自己addAll()合并错了，在这里记录下来，避免有人发生一样的错误</p>

![](https://raw.githubusercontent.com/Chenbin1996/chenbin1996.github.io/master/img/2018-4-4-2.png)