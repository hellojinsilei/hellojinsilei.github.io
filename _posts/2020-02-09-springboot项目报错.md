---
layout:     post                    # 使用的布局（不需要改）
title:      springboot项目部署到服务器上后，无报错却不显示内容的问题解决  随手记            # 标题 
subtitle:   springboot项目部署到服务器上后，无报错却不显示内容的问题解决  随手记            #副标题
date:       2020-01-22              # 时间
author:     ListJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - springboot
    - 异常
    - java
    - 随手记
---

最近使用springboot完成了一个项目，之后部署到个人服务器上，显示运行成功，但是页面却无内容展示，后经过检查和查阅资料发现原因是数据库的问题：
windows和linux下的Mysql对大小写敏感是有区别的：

 - windows下：不区分大小写（不敏感）
 -  linux下：区分大小写（敏感）
所以只需要把Mysql对大小写设置成不敏感就可以了。
首先先查询下是不是设置了对大小写敏感，可以在Mysql中输入：

```java
mysql>show variables like '%lower_case_table_names%'
```
**如果查询结果为0**说明对大小写是敏感的，接下来`编辑查找到的my.cnf文件`

```java
find / -name my.cnf
```
在`my.cnf`文件中的`[mysqld]`下加入一行：

```java
lower_case_table_names=1
```
然后重启Mysql服务：

```java
service mysql restart
```
就成功了！！！

