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

![jiegou3](F:\RasertingL.github.io\img\imgs_in_articals\jiegou3.PNG)

上面的是最终的一个项目目录结构



MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

另外，这次实验我是用的是注解的方式进行配置。

首先还是需要对pom.xml文件进行依赖引用，然后对application.properties文件进行一个配置信息的编写。主要是Mysql和Mybatis的配置信息。之后我们开始代码编写，为了简洁性，我没有做太多的分层处理了，我这里就分为：实体类、Mapper接口、service类，Controller类，使用Controller调用Service类中的函数方法来调用Mapper接口进行数据持久化处理。



# 二、依赖引入

下面是pom.xml文件的需要添加的内容

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

        <!--引入对Mybatis的集成-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
```

主要是引入对 Mysql 的依赖包，对jdbc的支持以及对Mybatis的集成。



# 三、 配置信息

下面是application.properties的内容

```properties
# Mysql的一些配置信息
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=true  #这个test为数据库名称
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#貌似上面这几行是最重要的，尤其是第一行，test数据库后面的?后面的东西，好像是设置时区对于Mysql8一定要有


#Mybatis的配置信息
mybatis.type-aliases-package=com.example.mybastisdemo.mapper

```

可以看到，就是两个方面的配置信息。对于Mysql 还是版本问题，之前已经提过。



# 四、 分层代码编写

 ### 1. 启动文件

首先，直接在启动文件SpringbootApplication.java的类上配置@MapperScan，这样就可以省去，单独给每个Mapper上标识@Mapper的麻烦。

如下：

```java
package com.example.mybatisdemo;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.example.mybatisdemo.mapper")
public class MybatisdemoApplication {

    public static void main(String[] args) {

        SpringApplication.run(MybatisdemoApplication.class, args);
    }

}

```

### 二. 各层代码

首先是entity层的User.java文件，该文件里定义的是一个类，该类的属性和数据库中要查询的表属性一一对应。代码如下：

```java
package com.example.mybastisdemo.entity;

public class User {

    private String id ;
    private String name ;

    public User(){}

    public User(String id, String name){
        this.id = id ;
        this.name = name ;
    }

    public void setId(String id){
        this.id = id ;
    }


    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public String getId() {
        return id;
    }
}

```

然后是mapper层，该层主要实现的是一个接口，而所有的SQL语句都是卸载该接口里，每一条语句对应的一个接口的方法。

**Mapper里的注解说明**

- @Select 查询注解
- @Result 结果集标识，用来对应数据库列名的，如果实体类属性和数据库属性名保持一致，可以忽略此参数
- @Insert 插入注解
- @Update 修改注解
- @Delete 删除注解

代码如下：

```java
package com.example.mybastisdemo.mapper;

import com.example.mybastisdemo.entity.User;
import org.apache.ibatis.annotations.* ;
import java.util.List ;

public interface Usermapper {
    @Select("select * from test1")
/*    @Results({
            @Result(property = "name", column = "name")
    })*/
    List<User> getAll();

    @Select("select * from test1 where id=#{id}")
    User getById(String id);

    @Insert({"insert into test1(id,name) values(#{id},#{name})"})
    void install(User user);

    @Update({"update test1 set name=#{name} where id=#{id}"})
    void Update(User user);

    @Delete("delete from test1 where id=#{id}")
    void delete(String id);
}

```

然后是实现Userservice类，其作用是编写不同的方法函数，而每种函数的实现是通过调用Usermapper中的接口函数来实现的。具体代码如下

```java
package com.example.mybastisdemo.service;

import com.example.mybastisdemo.entity.User;
import com.example.mybastisdemo.mapper.Usermapper ;
import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.stereotype.Service ;

import java.util.List ;

@Service
public class Userservice {
    @Autowired(required = false)  //玄学，，，如果没有这个required，那下面的usermapper会报错
    private Usermapper usermapper;

    /**
     * 获取所有用户信息
     */
    public List<User> getAll() {
        return usermapper.getAll();
    }

    /**
     * 添加用户
     */
    public void insert(User User){
        usermapper.install(User);
    }

	//根据id来查用户
    public User getById(String id){
        return usermapper.getById(id) ;
    }
    
	//更新用户信息
    public void update(User user){
        usermapper.Update(user);
    }
	
	//删除用户
    public void delete(String id){
        usermapper.delete(id) ;
    }



}

```



最后实现Controller控制器，这里直接调用Userservice中的函数作为服务。并且使用不同的@RequestMapping("")来配置URL映射来进行不同功能是实验实现。

最终代码如下:

```java
package com.example.mybastisdemo.web;

import com.example.mybastisdemo.entity.User ;
import com.example.mybastisdemo.mapper.Usermapper ;
import com.example.mybastisdemo.service.Userservice ;

import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping ;
import org.springframework.web.bind.annotation.RestController ;
import org.springframework.web.servlet.ModelAndView ;

import java.util.List ;

@RestController
public class UserController {
    @Autowired(required = false)
    private Usermapper usermapper ;


    @RequestMapping("/getUsers")
    public List<User> getUsers() {
        List<User> users=usermapper.getAll();
        return users;
    }

    @RequestMapping("/getUser")
    public User getUser() {
        String id = "1" ;
        User user=usermapper.getById(id);
        return user;
    }

    @RequestMapping("/add")
    public void save() {
        User user = new User() ;
        user.setId("4");
        user.setName("demo4");

        usermapper.install(user);
    }

    @RequestMapping("/update")
    public void update() {
        User user = new User() ;
        user.setId("2");
        user.setName("demo2");
        usermapper.Update(user);
    }

    @RequestMapping(value="/delete/{id}")
    public void delete(@PathVariable("id") String id) {
        usermapper.delete(id);
    }


}

```



最后启动服务，在浏览器中输入localhost:8080/需要的功能    可以查看到输出效果

最终结果和预期符合。