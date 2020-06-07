---
layout:     post
title:      【Spring Boot（一）】
subtitle:   Spring Boot搭建第一个Web小程序
date:       2020-06-07 18.02
author:     Raserting_L
header-img: img/artical/smooth_sharpen.png
catalog: true
tags:
    - Spring Boot
    - Java





---





**写在前面：**

​	**这是我第一天接触Spring Boot并且完成了第一个简单的Spring Boot服务程序搭建，再根据搜集到的各种资料摸索完成了Spring Boot + Mysql 的程序小实验，Spring Boot + Mybatis的程序小实验，Spring Boot + Redis的程序小实验之后写的，算是记录自己学习过程，如有错误望见谅。对了，一共分了四篇小文章来记录噢。**

# 一、关于Spring Boot

​	我们指导Sprnig Boot是和Spring息息相关的，Spring Boot是Spring发展到一定程度的产物，但是并不是Spring的替代品，Spring Boot是为了让程序员更好地使用Spring。

​	Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化Spring应用初始化搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使得开发人员不再需要定义样板化的配置。Spring Boot其实就是一个整合了很多可插拔的组件（框架），内嵌了使用工具（比如内嵌了Tomcat，Jetty等），为了方便开发人员快速搭建和开发的一个框架。



# 二、本文开发环境

IntelliJ IDEA 2019.3

JDK 11

Spring Boot 2.3.0



# 三、开始搭建

### 1、 新建项目

再IDEA中新建项目，在左侧选择Spring Initializr  ，可以直接用默认的初始化服务URL。

![springbegin](/imgs_in_articals/springbegin.PNG)

之后可以给项目取个名字再next

![springbegin](/imgs_in_articals/springbegin.PNG)

下面这一步很重要，我们需要做的是将Web->Spring Web处勾选

![web](/imgs_in_articals/web.PNG)

最后给项目选个位置，就可以了

![location](/imgs_in_articals/location.PNG)



### 2、 进行配置

首先看看最终的目录结构：

![jiegou1](/imgs_in_articals/jiegou1.PNG)

对于pom.xml配置文件的一点解释，如下：

```xml
 <dependencies>
        <!--下面引入了开发web项目相关依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--springboot单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!--添加Mysqly依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

    </dependencies>

    <!--maven构建-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```



com.example.springbootdeom目录下的SpringbootdemoApplication.java文件是是本来就存在的，而里面的@SpringBootApplication语句，可以看作是程序入口。

我们需要做的就是创建一个SpringBootdemoController.java作为“控制器”来完成这个服务小程序。

下面就是SpringBootdemoController.java文件。

```java
package com.example.springbootdemo;

import org.springframework.web.bind.annotation.RequestMapping ;
import org.springframework.web.bind.annotation.RestController ;

@RestController
public class SpringbootdemoController {
    @RequestMapping("/Hello")
    public String hello(){
        return "Hello, this is a springboot demo!" ;
    }
}


```

其中的 @@RequestMapping("/Hello")  是配置URL映射的，这样的话，待会程序运行之后，要进行测试，打开浏览器输入的是：localhost:8080/Hello

就会发现浏览器上面显示的是返回的Hello, this is a springboot demo!



至此，一个最简单的Spring Boot 程序就跑起来了。