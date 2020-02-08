---
layout:     post                    # 使用的布局（不需要改）
title:      SpringBoot+Mybatis实现用户增删改查               # 标题 
subtitle:   SpringBoot+Mybatis实现用户增删改查 #副标题
date:       2020-01-19              # 时间
author:     ListJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
    - spring
    - mybatis
---

## 用Springboot简单实现一个对用户增删改查的页面

#### 项目环境

 - IDEA 2019.2
 - SpringBoot 2.2.0
 - Mybatis 3.4.0
 - JDK 1.8
 
 #### 过程及源码
 - 创建SpringBoot项目
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200110085821314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTk1MzQ5,size_16,color_FFFFFF,t_70)
 - 对参数进行![在这里插入图片描述](https://img-blog.csdnimg.cn/20200110085931150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTk1MzQ5,size_16,color_FFFFFF,t_70)
 - 点击next，会进入创建依赖包的步骤，这里选择Spring Web。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200110090108823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTk1MzQ5,size_16,color_FFFFFF,t_70)

 - 创建完成后需要完善目录结构如图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200110090419735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTk1MzQ5,size_16,color_FFFFFF,t_70)
 - 在pom文件中添加相关依赖jar包

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jin</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.1.1</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.9</version>
        </dependency>


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
    </dependencies>

    <build>


    </build>

</project>

```

 - 在properties文件中写入配置信息
 

```java
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ssm?rewriteBatchedStatements=true&serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=password




spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
mybatis.type-aliases-package=com.jin.demo
```

 - 接下来是各个模块代码，首先创建Users实体类
 

```java
package com.jin.demo;

public class Users {
    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    private Integer id;
    private String name;
    private Integer age;
}

```

 - UsersController
 

```java
package com.jin.controller;


import com.jin.demo.Users;
import com.jin.service.UsersService;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
@RequestMapping("/users")
public class UsersController {

    @Autowired
    private UsersService usersService;

    /**
     * 页面跳转
     */
    @RequestMapping("/{page}")
    public String showPage(@PathVariable String page){
        return page;
    }


    /**
     * 添加用户
     */
    @RequestMapping("/addUser")
    public String addUser(Users users){
        this.usersService.addUser(users);
        return "ok";
    }

    /**
     * 查询全部用户
     */
    @RequestMapping("/findUserAll")
    public String FindUserAll(Model model){
        List<Users> list=this.usersService.findUsersAll();
        model.addAttribute("list",list);
        return "showUsers";
    }

    /**
     * 根据用户id查询用户
     */
    @RequestMapping("/findUserById")
    public String findUserById(Integer id,Model model){
        Users user=this.usersService.findUserById(id);
        model.addAttribute("user",user);
        return "updateUser";
    }
    /**
     * 更新用户
     */
    @RequestMapping("/editUser")
    public String editUser(Users users){
        this.usersService.updateUser(users);
        return "ok";
    }
    /**
     *删除用户
     */
    @RequestMapping("/deleteUsers")
    public String deleteUsers(Integer id){
        this.usersService.deleteUser(id);
        return "ok";
    }


}

```

 - UserService接口
 

```java
package com.jin.service;

import com.jin.demo.Users;

import java.util.List;

public interface UsersService {
    void addUser(Users users);
    List<Users> findUsersAll();
    Users findUserById(Integer id);
    void updateUser(Users users);
    void deleteUser(Integer id);
}

```

 - UserServie接口实现类
 

```java
package com.jin.service.impl;

import com.jin.demo.Users;
import com.jin.mapper.UsersMapper;
import com.jin.service.UsersService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional
public class UsersServiceImpl implements UsersService {

    @Autowired
    private UsersMapper usersMapper;

    @Override
    public void addUser(Users users) {

        this.usersMapper.insertUser(users);
    }

    @Override
    public List<Users> findUsersAll() {
        return this.usersMapper.selectUsersAll();
    }

    @Override
    public Users findUserById(Integer id) {
        return this.usersMapper.selectUsersById(id);
    }

    @Override
    public void updateUser(Users users) {
        this.usersMapper.updateUser(users);
    }

    @Override
    public void deleteUser(Integer id) {
        this.usersMapper.deleteUser(id);
    }
}

```

 - UserMapper映射文件
 

```java
package com.jin.mapper;

import com.jin.demo.Users;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Service;

import java.util.List;


@Mapper
@Service
public interface UsersMapper {
     void insertUser(Users users);
     List<Users>  selectUsersAll();
     Users selectUsersById(Integer id);

     void updateUser(Users users);
     void deleteUser(Integer id);
}

```

 - 在UsersMapper.xml文件中对应写入sql语句
 

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jin.mapper.UsersMapper">
    <insert id="insertUser" parameterType="users">
        insert into users(name,age) values(#{name},#{age})
    </insert>


    <select id="selectUsersAll" resultType="users">
        select id,name,age from users
    </select>
    <select id="selectUsersById" resultType="users">
        select id,name,age from users where id = #{value}
    </select>
    <update id="updateUser" parameterType="users">
        update users set name=#{name},age =#{age} where id=#{id}
    </update>

    <delete id="deleteUser">
        delete from users where id=#{id}
    </delete>
</mapper>

```

 - 在templates文件夹中创建对应的html文件
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200110091442125.png)
 - 打开浏览器，页面显示如下
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200110091621214.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTk1MzQ5,size_16,color_FFFFFF,t_70)

