---
layout:     post
title:      SpringBoot上传文件出错
subtitle:   遇到一个关于Tomcat文件夹的问题
date:       2018-04-25
author:     如漩涡
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - SpringBoot
---

## 现象

Spring Boot项目，今天做了一个与前端对接富文本的上传图片到服务器，返回一段URL给前端，一直运行着，前端一直请求接口一直上传图片做测试的时候，后台报了一个错误

`Could not parse multipart servlet request; nested exception is java.io.IOException: The temporary upload location [/tmp/tomcat.**.8081/work/Tomcat/localhost/ROOT] is not valid。`

> 当时就把我整蒙了，什么玩意儿，还没遇到过这种问题

**后来查阅了一下资料发现**

1. Spring Boot的应用服务在启动的时候，会生成在操作系统的/tmp目录下生成一个Tomcat.*的文件目录，用于"java.io.tmpdir"文件流操作

`TomcatEmbeddedServletContainerFactory`


2. 程序对文件的操作时：会生成临时文件，暂存在临时文件中；
长时间不操作，导致/tmp下面的tomcat临时文件目录被删除，且删除的文件不可恢复，上传文件时获取不到文件目录，报错

## 解决方案

1. 重启服务，临时方案：会重新生成tomcat目录，但是生产环境不建议如此操作；

2. 改变临时文件的存储路径，如下

```java
@Configuration
public class MultipartConfig {

    /**
     * 文件上传临时路径
     */
    @Bean
    MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        String location = System.getProperty("user.dir") + "/data/tmp";
        File tmpFile = new File(location);
        if (!tmpFile.exists()) {
            tmpFile.mkdirs();
        }
        factory.setLocation(location);
        return factory.createMultipartConfig();
    }
}
```
