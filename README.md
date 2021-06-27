# Headline

> An awesome project. Hello lilpum~ 666 牛逼77

```java
package com.wxchina.fp.approval.controller;

import com.wxchina.fp.approval.entity.MyResponseEntity;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

/**
 * @program: fp-docker部署版本
 * @description:
 * @author: lilpum
 * @create: 2021-06-16 18:10
 **/
@ControllerAdvice
public class ExceptionConfigController {
    // 专门用来捕获和处理Controller层的空指针异常
    @ExceptionHandler(NullPointerException.class)
    public ResponseEntity<MyResponseEntity> nullPointerExceptionHandler(NullPointerException e) {
        return new ResponseEntity<>(new MyResponseEntity(HttpStatus.BAD_REQUEST.value(), "空指针异常!" + e.getCause()), HttpStatus.BAD_REQUEST);
    }

    // 专门用来捕获和处理Controller层的运行时异常
    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<MyResponseEntity> runtimeExceptionHandler(RuntimeException e) {
        return new ResponseEntity<>(new MyResponseEntity(HttpStatus.BAD_REQUEST.value(), "运行时异常!" + e.getCause()), HttpStatus.BAD_REQUEST);
    }

    // 专门用来捕获和处理Controller层的异常
    @ExceptionHandler(Exception.class)
    public ResponseEntity<MyResponseEntity> exceptionHandler(Exception e) {
        return new ResponseEntity<>(new MyResponseEntity(HttpStatus.BAD_REQUEST.value(), "出现异常!" + e.getCause()), HttpStatus.BAD_REQUEST);
    }
}

```

