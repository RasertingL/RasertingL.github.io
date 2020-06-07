---
layout:     post
title:      【Spring Boot】(二)
subtitle:   Spring Boot + Mysql 使用
date:       2020-6-7 19.03
author:     Raserting_L
header-img: img/artical/artical2.PNG
catalog: true
tags:
    - JAVA
    — Spring Boot
---

**写在前面：**

​	**这是我第一天接触Spring Boot并且完成了第一个简单的Spring Boot服务程序搭建，再根据搜集到的各种资料摸索完成了Spring Boot + Mysql 的程序小实验，Spring Boot + Mybatis的程序小实验，Spring Boot + Redis的程序小实验之后写的，算是记录自己学习过程，如有错误望见谅。对了，一共分了四篇小文章来记录噢。**



# 一、 总体概述

首先可以了解一下最终的目录结构

![jiegou2'](/imgs_in_articals/jiegou2'.PNG)

首先要保证电脑里以及安装好了MySQL数据库，然后才能进行下面的步骤。

我们有三个重要步骤，首先是再pom.xml文件中添加MySQL的依赖，然后在application.properties中添加MySQL的配置信息，最后就是进行代码编写测试。

# 二、依赖配置

需要添加的依赖如下所示：

```xml
 <!--添加Mysql连接依赖包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--引入jdbc支持-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```

需要添加对Mysql的连接依赖包，然后引入地jdbc的支持

然后等待Maven将其引入就好。

# 三、 MySQL配置信息

然后打开application.properties文件，输入以下信息：

```properties
# Mysql的一些配置信息
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=true  #这个test为数据库名称
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#貌似上面这几行是最重要的，尤其是第一行，test数据库后面的?后面的东西，好像是设置时区对于Mysql8一定要有
```

**这里有个坑，网上大部分的连接都使用的是5.xx的MySQL，但是现在最新的MySQL已经是8.xx版本的了，所以需要在url那里添加一大段设置时区的语句，如上方第一行有效配置语句所示。**

# 四、代码编写测试

我在test包中进行了代码测试，我的jdbcdemoApplicationtests.java如下所示：

```java
package com.example.jdbcdemo;

import  java.util.List ;
import  java.util.Map ;


import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

//下面表示用来做单元测试的类
@RunWith(SpringRunner.class)
@SpringBootTest
class JdbcdemoApplicationTests {

    @Autowired
    private JdbcTemplate jdbcTemplate ;


    @Test
    void contextLoads() {
    }

    @Test
    public void testmysql(){
        List<Map<String, Object>> result = jdbcTemplate.queryForList("select  * from test1") ;
        System.out.println("Query result is " + result.size());
        for(Map<String,Object> out : result){
            System.out.println(out);
        }
        System.out.println("success!");
    }

    @Test
    public void testmysql_forupdate(){
        jdbcTemplate.execute("update test1 set name='666' where id='2'");
        testmysql();


    }

}

```

可以看到我们主要使用的是一个叫做JdbcTemplate的类创建的对象进行操作的。

在数据库中先建立了一张具有两个属性的表，属性名分别为 id 和name。对于查出来的每一条结果使用Map<String , Object>来接收。所有的结果使用List来接受。

在这里我做了两个测试上面是查询然后输出，后者是进行更新操作的实验。结果都和预期一样。