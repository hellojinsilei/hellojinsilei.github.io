---
layout:     post                    # 使用的布局（不需要改）
title:      添加Interceptors拦截器后不能加载静态资源的问题  随手记            # 标题 
subtitle:   添加Interceptors拦截器后不能加载静态资源的问题              #副标题
date:       2020-01-19              # 时间
author:     ListJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
    - springboot
    - 异常
    - 随手记
---

先上代码：

```java
package com.listj.community.community.interceptor;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private SessionInterceptor sessionInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(sessionInterceptor).addPathPatterns("/**");
    }
}

```
项目中我添加了Interceptors拦截器，后运行程序后项目无法加载静态资源
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200114150006590.png)
## 解决办法
将@EnableWebMvc注解注释掉，静态资源又能加载成功了

## 问题分析
之后我查阅资料，并且看了源码之后简单分析了下：

> @EnableWebMvc此注解加上后在程序运行后会自动拦截静态资源，如果需要加载静态资源，我们需要重写一个addResourceHandlers方法

```java
@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {

    }
```
因为我们没有重写方法，因此无法加载静态资源。
而在我们注释完@EnableWebMvc注解后，因为我们实现的是WebMvcConfigurer接口，我查看源码后，接口中有addResourceHandlers方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200114150640416.png)
我在WebMvcAutoConfigurationAdapter实现类中，查看该方法的实现，以下为源码。

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }
            }
        }
```
该方法会帮我们自动加载resource目录下的静态资源。
