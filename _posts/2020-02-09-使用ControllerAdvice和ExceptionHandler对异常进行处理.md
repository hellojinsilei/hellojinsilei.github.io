---
layout:     post                    # 使用的布局（不需要改）
title:      使用ControllerAdvice和ExceptionHandler对异常进行处理            # 标题 
subtitle:   使用ControllerAdvice和ExceptionHandler对异常进行处理            #副标题
date:       2020-01-21              # 时间
author:     ListJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
    - 异常
---

项目中使用了ControllerAdvice和ExceptionHandler对异常进行处理，这里简单记录下整个流程
#### CustomizeExceptionHandler.java

```java
package com.listj.community.community.advice;

import com.alibaba.fastjson.JSON;
import com.listj.community.community.dto.ResultDTO;
import com.listj.community.community.exception.CustomizeErrorCode;
import com.listj.community.community.exception.CustomizeException;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@ControllerAdvice
public class CustomizeExceptionHandler {
    @ExceptionHandler(Exception.class)
    ModelAndView handle(Throwable e, Model model, HttpServletRequest request, HttpServletResponse response) {
        String contentType=request.getContentType();
        if("application/json".equals(contentType)){
            ResultDTO resultDTO;
            if(e instanceof CustomizeException){
                resultDTO = ResultDTO.errorOf((CustomizeException)e);
            }else{
                resultDTO = ResultDTO.errorOf(CustomizeErrorCode.SYS_ERROR);
            }
            try {
                response.setContentType("application/json");
                response.setStatus(200);
                response.setCharacterEncoding("utf-8");
                PrintWriter writer = response.getWriter();
                writer.write(JSON.toJSONString(resultDTO));
                writer.close();
            } catch (IOException ioe) {
            }
            return null;
        }else{
            if(e instanceof CustomizeException){
                model.addAttribute("message",e.getMessage());
            }else{
                model.addAttribute("message",CustomizeErrorCode.SYS_ERROR.getMessage());
            }
            return new ModelAndView("error");
        }

    }
}

```
这段代码对异常进行了拦截，并跳转到error.html页面
#### CustomizeException.java

```java
package com.listj.community.community.exception;

public class CustomizeException extends RuntimeException {
    private String message;
    private Integer code;

    public CustomizeException(ICustomizeErrorCode errorCode) {
        this.code=errorCode.getCode();
        this.message=errorCode.getMessage();
    }

    @Override
    public String getMessage() {
        return message;
    }

    public Integer getCode() {
        return code;
    }
}

```
CustomizeException继承了RuntimeException的类并定义了message和code两个变量，用于记录错误信息和错误代码。
#### ICustomizeErrorCode.java

```java
package com.listj.community.community.exception;

public interface ICustomizeErrorCode {
    String getMessage();
    Integer getCode();
}

```
接口定义两个方法，用于获取错误信息和错误代码
#### CustomizeErrorCode.java

```java
package com.listj.community.community.exception;

public enum CustomizeErrorCode implements ICustomizeErrorCode {
    QUESTION_NOT_FOUND(2001,"问题不存在，换一个试试"),
    TARGET_PARAM_NOT_FOUND(2002,"未选中任何问题或评论进行回复"),
    NO_LOGIN(2003,"未登录，不能进行评论,请先登录"),
    SYS_ERROR(2004,"服务器异常！"),
    TYPE_PARAM_WRONG(2005,"评论类型错误或不存在！"),
    COMMENT_NOT_FOUND(2006,"评论不存在，换一个试试"),
    INVALID_INPUT(2007,"操作错误")
    ;
    private Integer code;
    private String message;


    CustomizeErrorCode(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }

    @Override
    public Integer getCode() {
        return code;
    }
}

```
CustomizeErrorCode是ICustomizeErrorCode接口的实现类，实现了`getMessage()`和`getCode()`这两个方法，并定义了异常的一些信息。
#### 具体使用

```java
if(comment.getParentId()==null||comment.getParentId()==0){
            throw new CustomizeException(CustomizeErrorCode.QUESTION_NOT_FOUND);
        }
```


