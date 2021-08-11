---
title: "sass一个入门笔记"
date: 2021-08-11 20:46:19
tags: [前端, sass]
---

> 写css懒癌发作学一手sass，果然懒才是第一生产力QwQ

## sass与scss与less

​	sass与scss可以看作同一种东西，，sass需要遵守严格的语法标住，scss包含sass并且语法格式与css非常相似（所以一般写`lang="scss"`而不是`lang="sass"`吗qwq），并且好像统称sass的样子。

​	 与less，都是css的预编译处理语言，不同的是less是基于js在客户端进行处理，sass是基于ruby在服务器处理。

## Vue项目引入sass

因为版本问题各种报错QAQ

最终是这么搞的：

```text
cnpm install sass-loader@9.0.0 -d
cnpm install node-sass@4.14.1 -d
```

然后vue里面这么写：

```vue
<style lang="scss" scoped>

</style>
```

里面按sass语法去写就可以了的样子qwq

一个测试是可行的：

```scss
<style lang="scss" scoped>
.hello {
  $height: 200px;
  height: $height;
  width: 100px;
  background-color: aqua;
}
</style>
```

## 基本语法

### 变量使用

* 用 `$` 表示变量
* 可以包含很多东西
* 不区分中划线和下划线
* 仅原生css支持的注释会被编译

```scss
$color-red: red;
$border-style: 5px solid $color-red;
$color-red: blue; //变量重新声明
.hello {
  $height: 200px;
  height: $height;
  width: 100px;
  background-color: $color_red;
  border: $border_style; /*注释*/
}
//编译后
.hello {
  height: 200px;
  width: 100px;
  background-color: blue;
  border: 5px solid red;
  /*注释*/
}
```

* 使用 `!default` 防止重新声明

```scss
$color-red: red;
$border-style: 5px solid $color-red;
$color-red: blue !default;
.hello {
  background-color: $color_red;
  border: $border_style;
}
//编译后
.hello {
  background-color: red;
  border: 5px solid red;
}
```

* 大段样式的复用使用混合器

### 嵌套规则

#### 嵌套选择器

* 默认每套一层都是一层子元素

```scss
.hello {
  height: 200px;
  width: 100px;
  background-color: aqua;
  border: 5px solid red;
  .test {
    background-color: blue;
  }
}
//编译后
.hello {
  height: 200px;
  width: 100px;
  background-color: aqua;
  border: 5px solid red;
}
.hello .test {
  background-color: blue;
}
```

* `&`父选择器
* 群组选择器也没什么意外

```scss
a, p {
  color: red;
  &:hover {
    color: blue;
  }
}
//编译后
a, p {
  color: red;
}
a:hover, p:hover {
  color: blue;
}
```

#### 嵌套属性

也许比缩写更清晰一些。

```scss
.hello {
  border: {
    style: solid;
    width: 5px;
    color: red;
  }
}
//编译后
.hello {
  border-style: solid;
  border-width: 5px;
  border-color: red;
}
```

## 导入sass文件

* 不同于css执行到`@import`才会去下载资源，sass在生成css时就会整合成一个文件。
* 约定以下划线开头的.scss文件仅用于import，并不会编译生成单独的css文件。

假设有文件\_hello.scss

引入该文件（比如在相同文件夹下）：

```scss
@import "./hello"; //这里路径书写还挺智能的
```

* 在某处导入文件相当于原封不动搬过来

## 混合器

### 基础用法

* 使用 `@mixin` 包含一大段样式。

```scss
@mixin round-border {
  border: {
    style: solid;
    width: 5px;
    color: red;
    radius: 10px;
  }
}
.hello {
  height: 100px;
  width: 100px;
  background-color: aqua;
  @include round-border;
}
//编译后
.hello {
  height: 100px;
  width: 100px;
  background-color: aqua;
  border-style: solid;
  border-width: 5px;
  border-color: red;
  border-radius: 10px;
}
```

### 带参数的混合器

* 像函数一样使用参数，包括默认值（可以引用其它参数）等
* 可以通过`$name: value`传参，不必按顺序，别漏参数

```scss
@mixin round-border($border-color: red) {
  border: {
    style: solid;
    width: 5px;
    color: $border-color;
    radius: 10px;
  }
}
.hello {
  height: 100px;
  width: 100px;
  background-color: aqua;
  @include round-border(blue);
  .test {
    @include round-border();
  }
}
//编译后
.hello {
  height: 100px;
  width: 100px;
  background-color: aqua;
  border-style: solid;
  border-width: 5px;
  border-color: blue;
  border-radius: 10px;
}
.hello .test {
  border-style: solid;
  border-width: 5px;
  border-color: red;
  border-radius: 10px;
}
```

## 选择器继承

### 基础用法

* 使用`@extend`继承选择器，只会继承选择器所命中元素的样式
  * 例如 `@extend qwq.owo`并不会分别继承 `.qwq` 和 `.owo` 下的样式

```scss
.father {
  border: {
    style: solid;
    width: 5px;
    color: red;
    radius: 10px;
  }
}
.hello {
  height: 100px;
  width: 100px;
  background-color: aqua;
  //看下面编译规则，下边这两行换一下也不会怎么样emm
  @extend .father;
  border-color: blue;
}
//编译后（竟然是组合选择器qwq）
.father, .hello {
  border-style: solid;
  border-width: 5px;
  border-color: red;
  border-radius: 10px;
}

.hello {
  height: 100px;
  width: 100px;
  background-color: aqua;
  border-color: blue;
}
```

### 神奇用法

* 任何css规则都可以被继承

```scss
//禁用链接样式
.disabled {
  color: gray;
  @extend a;
}
```

* 一般`@extend` 比 `@mixin`生成的css更小，因为只涉及选择器的重复。

咕~
