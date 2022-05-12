---
title: "文件路径问题 typeorm 'No metadata for User was found' 解决"
date: 2022-05-12 14:47:03
tags: [typeorm, node, 前端]
---

## 问题描述

问题来自一个typeorm的dataSource配置：

```typescript
const dataSource = new DataSource({
  type: "mysql",
  host: MYSQL_HOST,
  port: Number(MYSQL_PROT),
  username: MYSQL_USER,
  password: MYSQL_PWD,
  database: MYSQL_DB,
  synchronize: process.env.NODE_ENV === "development",
  entities: ["src/entity/*.ts"],
});
```

开发环境正常，生产环境报错：

```bash
******\src\entity\user.ts:1
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, DeleteDateColumn } from 'typeorm';
^^^^^^

SyntaxError: Cannot use import statement outside a module
```

蜜汁调用了ts文件。

## 解决

entities路径指向了未被编译的ts文件，顺带如果路径写错的话就会出现 "No metadata for User was found" 的报错。

`__dirname` 指向当前模块的路径。

改一下就好了：

```typescript
const dataSource = new DataSource({
  type: "mysql",
  host: MYSQL_HOST,
  port: Number(MYSQL_PROT),
  username: MYSQL_USER,
  password: MYSQL_PWD,
  database: MYSQL_DB,
  synchronize: process.env.NODE_ENV === "development",
  entities: [__dirname + "/../entity/**/*.js", __dirname + "/../entity/**/*.ts"],
});
```

才发现自己原来根本不理解这个路径的意思qwq
