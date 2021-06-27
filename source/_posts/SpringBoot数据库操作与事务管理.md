---
title: "[SpringBoot笔记]数据库操作与事务管理"
date: 2021-06-27 23:20:28
tags: [java, SpringBoot, web后端]
---

## 数据库操作

Spring-Data-Jpa：Spring对Hibernate的整合（一脸懵（反正很牛逼就是了

实例（mysql）:

### RESTful API设计

~~原实例过于hentai所以改了一些内容qwq~~

只能用一堆智乃了..不过到底有多少只智乃..（心爱叔狂喜<img src="https://z3.ax1x.com/2021/06/27/RYCBmn.jpg" width="50px">

| 请求类型 | 请求路径  | 功能           |
| -------- | --------- | -------------- |
| GET      | /chino    | 获取智乃列表   |
| POST     | /chino    | 创建新的智乃   |
| GET      | /chino/id | 通过id查询智乃 |
| PUT      | /chino/id | 通过id更新智乃 |
| DELETE   | /chino/id | 通过id删除智乃 |

### 配置

pom.xml添加依赖项: 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

application.yml

```yaml
spring:
  profiles:
    active: dev #dev里边就写了个localhost8080
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/chino
    username: root
    password: chino1204
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
```

配置完毕，创建数据库chino，启动成功。

### 新建表

新建一个java类，命名为chino，随便写点属性

```java
package com.example.demo1;

import javax.persistence.Entity;
import javax.persistence.criteria.CriteriaBuilder;

@Entity //表示这个类对应数据库里的一个表
public class chino {
    private Integer id;
    private String husband;
    private Integer height;
}
```

跑一下，进数据库瞧瞧自动创建了一张表Chino。

手动插一条数据，再跑一遍程序，进数据库瞧瞧刚刚的数据没了QAQ

Application.yml: 

```yaml
jpa:
    hibernate:
      ddl-auto: create
      create 换成 update
```

| 属性        | 操作                       |
| ----------- | -------------------------- |
| create      | 覆盖创建                   |
| create-drop | 程序运行结束后会自动删除表 |
| update      | 更新（不会清除数据）       |
| none        | 啥都不做                   |
| validate    | 验证表结构，与类不同则报错 |

### Controller

新增接口ChinoRepository.java:

```java
package com.example.demo1;

import org.springframework.data.jpa.repository.JpaRepository;

public interface ChinoRepository extends JpaRepository<Chino, Integer> {
}
```

ChinoController.java: 

```java
package com.example.demo1;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class ChinoController {

    @Autowired
    ChinoRepository chinoRepository;

    /**
     * 获取智乃列表
     * @return
     */
    @GetMapping(value = "/chino")
    public List<Chino> chinoList() {
        return chinoRepository.findAll();
    }

    /**
     * 添加一只新的智乃
     * @param height
     * @param husband
     * @return
     */
    @PostMapping(value="/chino")
    public Chino addChino(@RequestParam() Integer height,
                          @RequestParam(required = false, defaultValue = "Cocoa") String husband) {
        Chino chino = new Chino();
        chino.setHeight(height);
        chino.setHusband(husband);

        return chinoRepository.save(chino);
    }

    /**
     * 根据id获取单只智乃信息
     * @param id
     * @return
     */
    @GetMapping(value = "/chino/{id}")
    public Chino getChinoById(@PathVariable("id") Integer id) {
        return chinoRepository.findById(id).orElse(null); //为null就用orElse里的值（虽然还是写的null..hhh..
    }

    /**
     * 根据id修改智乃信息
     * @param id
     * @param height
     * @param husband
     * @return
     */
    @PostMapping(value = "/chino/{id}")
    public Chino updateChino(@PathVariable("id") Integer id,
                            @RequestParam() Integer height,
                            @RequestParam(required = false) String husband) {
        Chino chino = chinoRepository.findById(id).orElse(null);;
        chino.setHeight(height);
        if (husband != null)
            chino.setHusband(husband);
        return chinoRepository.save(chino);
    }

    /**
     * 根据id删除智乃(>_<)
     * @param id
     */
    @DeleteMapping(value = "/chino/{id}")
    public void deleteChino(@PathVariable(value = "id") Integer id) {
        chinoRepository.deleteById(id);
    }
}
```

扩展-通过height查询智乃：

接口中加入方法定义 ChinoRepository.java:

```java
public interface ChinoRepository extends JpaRepository<Chino, Integer> {
    //通过height查询，这里方法名必须是这样的
    public List<Chino> findByHeight(Integer height);
}
```

Controller中添加: 

```java
/**
 * 根据height获取智乃信息
 * @param height
 * @return
 */
@GetMapping(value = "/chino/height/{height}")
public List<Chino> getChinoByHeight(@PathVariable("height") Integer height) {
    return chinoRepository.findByHeight(height);
}
```

## 事务管理

~~没学过数据库在这里瞎搞无所畏惧~~

举例，做一个电商平台，购买时要同时扣掉用户的钱和商家的库存，希望两者同时扣掉。其中一者扣除失败的话，希望另一者也不要被扣除。

实例：同时插入两个智乃。<img src="https://z3.ax1x.com/2021/06/27/RYCBmn.jpg" width="50px">

新建ChinoService类 ChinoService.java:

```java
package com.example.demo1;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.stereotype.Service;@Servicepublic class ChinoService {    @Autowired    private ChinoRepository chinoRepository;    public void insertTwoChino() {        Chino chino1 = new Chino(), chino2 = new Chino();        chino1.setHeight(144);        chino1.setHusband("Cocoa");        chinoRepository.save(chino1);        chino2.setHeight(145);        chino2.setHusband("Hoto Cocoa");        chinoRepository.save(chino2);    }}
```

ChinoController类中添加内容：

```java
@AutowiredChinoService chinoService;@PostMapping(value="/chino/two")public void addTwoChino() {    chinoService.insertTwoChino();}
```

运行，POST一下"localhost:8080/chino/two"成功插入。

故意使其中一条（第二条）插入失败，修改数据库中husband字段长度限制为9。

然后insertTwoChino方法前加注解：

```java
@Transactionalpublic void insertTwoChino() {...}
```

测试没有插入任何一条数据（顺带一提id还是会自增的（并且增加了2））。

