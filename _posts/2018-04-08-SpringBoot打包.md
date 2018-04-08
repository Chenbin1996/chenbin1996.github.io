---
layout:     post
title:      SpringBoot打包
subtitle:   打包的一个技巧，避免打包时走错路
date:       2018-04-08
author:     如漩涡
header-img: img/avatar.png
catalog: true
tags:
    - SpringBoot
---

> 更多文章查看本人CSDN博客 [《如漩涡的博客》](https://blog.csdn.net/m0_37701381)

### pom.xml文件引入插件

在build下添加插件：

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <layout>MODULE</layout>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>repackage</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```
如若想要把Spring Boot中的.properties文件、XML文件以及ftl(html)页面一起打包的话，需要加上以下代码：

```xml
<resources>
   <resource>
      <directory>src/main/java</directory>
      <includes>
         <include>**/*.properties</include>
         <include>**/*.xml</include>
         <include>**/*.ftl</include>
      </includes>
      <filtering>false</filtering>
   </resource>
   <resource>
      <directory>src/main/resources</directory>
      <includes>
         <include>**/*.properties</include>
         <include>**/*.xml</include>
         <include>**/*.ftl</include>
      </includes>
      <filtering>false</filtering>
   </resource>
</resources>
```

因为打包工具默认是把除了class文件打包以外，其余的都是会过滤掉的，本人就踩过这个坑，打包了好几次，ftl文件就是找不到，后来加了这段话就好了，完整代码：
```xml
<build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
              <configuration>
                  <layout>MODULE</layout>
              </configuration>
              <executions>
                  <execution>
                      <goals>
                          <goal>repackage</goal>
                      </goals>
                  </execution>
              </executions>
          </plugin>
      </plugins>
<resources>
   <resource>
      <directory>src/main/java</directory>
      <includes>
         <include>**/*.properties</include>
         <include>**/*.xml</include>
         <include>**/*.ftl</include>
      </includes>
      <filtering>false</filtering>
   </resource>
   <resource>
      <directory>src/main/resources</directory>
      <includes>
         <include>**/*.properties</include>
         <include>**/*.xml</include>
         <include>**/*.ftl</include>
      </includes>
      <filtering>false</filtering>
   </resource>
</resources>
  </build>
```

### 使用

而后在IDEA右侧导航栏中找到Maven导航

![](https://img-blog.csdn.net/20180305162406936?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbTBfMzc3MDEzODE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如图，点开Maven导航栏之后Lifecycle，按照顺序运行clean—compile—package，先清理了编译的文件，再重新编译，最后打包，之后在项目target文件夹下就可以找到生成好的jar包

说明一点，使用package打包只会在targer目录下，使用install打包的话，会在target下找到，也会在C盘用户
目录Maven本地仓库地址找到，install适用分布式模块时开发调试

不喜欢Spring Boot打包时候还启动Spring测试的，可以在pom.xml文件的<properties>标签中加入一句话：

```xml
<skipTests>true</skipTests>
```
这样就把打包时候的测试略掉了