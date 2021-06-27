---
title: "[SpringBoot笔记]项目创建与Controller使用"
date: 2021-06-27 23:18:15
tags: [java, SpringBoot, web后端]
---

## 项目结构与配置

目录：

<img src="https://z3.ax1x.com/2021/06/08/2BLrrT.png" width="300px">

配置文件application.yml:

```yaml
server:
  port: 8080
  context-path: /
```

## Controller的使用

### @RestController和@RequestMapping注解

~~@Controller需要配合@ResponseBody使用~~，Spring4之后新增注解@RestController相当于前边两个合起来。

| 注解            | 功能              |
| --------------- | ----------------- |
| @RestController | 处理Http请求      |
| @RequestMapping | 配置url映射       |
| @GetMapping     | 同上，方法指定GET |
| POST..blabla..  | 同上blabla...     |

实例：

HelloChinoController.java

```java
package com.example.demo1;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
//可以直接import org.springframework.web.bind.annotation.*;

//@RestController = @Controller + @ResponseBody
@RestController
//RequestMapping配置整个类的url
@RequestMapping("/hello")
public class HelloChinoController {
    /*
    单独配置每个方法的url，{}可包含多个
    不写method则支持全部方法（然而并不规范的样子qwq
    支持多个url，这里通过"/hello/chino"或"/hello/chinochann"访问
    */
    @RequestMapping(value = {"/chino", "/chinochann"}, method = RequestMethod.GET)
    public String hello() {
        return "Hello Chino!";
    }
}
```

### 请求参数的获取

| 注解          | 功能           |
| ------------- | -------------- |
| @PathVariable | 从url获取参数  |
| @RequestParam | 从body获取参数 |

从url获取参数：

```java
@RequestMapping(value = "/{id}chino/{id2}", method = RequestMethod.GET)
public String hello(@PathVariable("id") Integer id, @PathVariable("id2") Integer id2) {
    return "Hello Chino! " + id + " " + id2;
}
```

通过 http://localhost:8080/hello/233chino/466 访问，返回“Hello Chino! 233 466”

GET参数获取：

```java
@GetMapping("/chino")
public String hello(@RequestParam(required = false, defaultValue = "1") Integer id) {
    //id只能叫id
    return "Hello Chino! " + id;
}
//或者这样之类的，，反正到时候查手册啦，，
@GetMapping("/chino")
public String hello(@RequestParam(value = "id", required = false, defaultValue = "1") Integer myId) {
    return "Hello Chino! " + myId;
}
```

通过 http://localhost:8080/hello/chino 访问，返回 “Hello Chino! 1”，通过 http://localhost:8080/hello/chino/?id=233 访问，返回 “Hello Chino! 233”

随便粘个POST的实例：

```java
@PostMapping("/pay")
public AjaxResult<PayResult> pay(@RequestBody PayParam param, HttpServletRequest request) {
    Long userId = UserUtils.getUserId();
    String ip = IPUtils.getRequestClientRealIP(request);
    PayResult payResult = orderService.pay(param, ip, userId);
    return AjaxResult.SUCCESS(payResult);
}
```

