---
title: 关于跨域——浏览器使用less不生效的问题解决
date: 2022-01-16 13:40:10
tags: [前端, css]
---

## 问题描述

想用原生三件套写点东西来着，css嫌麻烦就引了一下less，按官方给的 `<script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.11.1/less.min.js" ></script>` 并没有生效。

一段示例代码：

1.html：

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet/less" type="text/css" href="1.less" />
    <script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.11.1/less.min.js" ></script>
  </head>
  <body>
    <!--奈何本人没文化，divspan走天下-->
    <div class="container">
      <div></div>
    </div>
  </body>
</html>
```

1.less:

```less
.container {
  height: 200px;
  width: 200px;
  background: aqua;
  display: flex;
  align-items: center;
  justify-content: center;
  div {
    height: 100px;
    width: 100px;
    background: bisque;
  }
}
```

## 解决方案

于是搜到了这个东西：

由于浏览器端使用Less时，是使用 ajax 来拉取 .less 文件，如果直接在本机文件系统打开（即地址是file://开头）或者是有跨域的情况下，会拉取不到 .less 文件，导致样式无法生效。因此，必须在http(s)协议下使用，即必须在服务器环境下使用。

所以也许要本地开一个静态文件服务器emm？

尝试用[anywhere](https://github.com/JacksonTian/anywhere)：

> AnyWhere是一款随启随用的静态文件服务器，可以随时随地将你的当前目录变成一个静态文件服务器的根目录。

```bash
npm install -g anywhere
anywhere

Running at http://192.168.10.9:8000/
Also running at https://192.168.10.9:8001/
```

就可以用了的样子qwq

