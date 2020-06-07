---
layout:     post
title:      【Spring Boot（三）】
subtitle:   Spring Boot + Mybatis
date:       2020-06-07 19.32
author:     Raserting_L
header-img: img/artical/smooth_sharpen.png
catalog: true
tags:
    - Spring Boot




---



**写在前面：**

​	**这是我第一天接触Spring Boot并且完成了第一个简单的Spring Boot服务程序搭建，再根据搜集到的各种资料摸索完成了Spring Boot + Mysql 的程序小实验，Spring Boot + Mybatis的程序小实验，Spring Boot + Redis的程序小实验之后写的，算是记录自己学习过程，如有错误望见谅。对了，一共分了四篇小文章来记录噢。**



# 一、 总体概述

![jiegou4](F:\RasertingL.github.io\img\imgs_in_articals\jiegou4.PNG)



上面是最终的项目目录结构，这是我第一次使用Redis。。。所以希望没有错误。

对了，要进行下面的实验，首先一定要先装好Redis┗|｀O′|┛ 嗷~~，在Windows下的话，记得取github下载，官网只提供Linux的。

# 二、依赖引入

下面是pom.xml文件的需要添加的内容

```xml
        <!--引入redis的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```





# 三、配置信息

还是老规矩，对application,properties进行配置信息的编写

```properties
# redis的配置文件
##  redis config
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
spring.redis.database=0
# 最大空闲连接数
spring.redis.jedis.pool.max-active=8
# 最小空闲连接数
spring.redis.jedis.pool.max-idle=8
# 等待可用连接的最大时间，负数为不限制
spring.redis.jedis.pool.max-wait=-1
# 最大活跃连接数，负数为不限制
spring.redis.jedis.pool.min-idle=1
# 数据库连接超时时间，2.0 中该参数的类型为Duration，这里在配置的时候需要指明单位  1.x可以将此参数配置10000 单位是ms
# 连接池配置，2.0中直接使用jedis或者lettuce配置连接池
spring.redis.timeout=60s

```

其中各个信息的意义如注释所示，另外由于我的Redis没有设置密码，所以密码这一栏是空着的。



# 四、代码编写

其实这个倒是只需要加一个Controller类就好了。

我先在Redis客户端里加了一个键值对： ay -   "an"

所以我的小实验做的就是给Redis加入一个键值对，和根据键茶值的操作

UserController.java文件实现如下:

```java
package com.example.redisdemo.usercontroller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.Mapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@EnableAutoConfiguration
@RequestMapping("/")
public class UserController {
    @Autowired
    private StringRedisTemplate stringredistemplate ;

    @RequestMapping("/addandget")
    public String addandget(){
        stringredistemplate.opsForValue().set("aaa", "1111");

        String str = stringredistemplate.opsForValue().get("ay") ;
        System.out.println("We get the Val:" + str);

        return "success and We get the Val:"+str ;
    }

}

```

可以看到，主要使用的是StringRedisTemplate这个类的方法来进行操作。在服务程序启动之后，出入URL最后返回结果与预期相同。