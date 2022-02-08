---
title: 项目开发记录——reacst仿写低配掘金客户端
date: 2022-02-08 17:45:27
tags: [前端, react]
---

# 开始做一个React项目叭——历程记录

有几次vue项目经历，尝试写个react小项目（~~比着跟官方教程写的tictactoe抄抄就行了~~），在此做点记录。

来自[字学镜像计划](https://bytedancecampus1.feishu.cn/docs/doccn4ArhE7N3J3V2OszhGlZQkd#)，最终代码存在github：[ERUIHNIYHBKBNF/react-juejin](https://github.com/ERUIHNIYHBKBNF/react-juejin)

## 初始化

按官方文档来，创建一个新的单页应用。

```bash
npx create-react-app react-juejin
cd react-juejin
npm start
```

然后一堆奇奇怪怪看不懂的文件全删掉（逃），最后留一点变成这样就好了叭：

![](https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022020702.webp)

## 添加路由

**看文档之前一定先看好版本QAQ**

看上去还不错的一篇文章：[React Router v6 使用指南](https://zhuanlan.zhihu.com/p/191419879)

抄一下官方的demo：[Basic Example](https://reactrouter.com/docs/en/v6/examples/basic)

看起来只有 文章列表 和 文章 两个页面，不是很麻烦的样子qwq

```bash
npm install -s react-router-dom@6
```

然后用Router就相当于一个组件一样方便使用，具体去翻文档就好了唔。

index.js：

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import { BrowserRouter } from 'react-router-dom';

ReactDOM.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>,
  document.getElementById('root')
);

```

App.js：

```jsx
import { Routes, Route } from 'react-router-dom';
import ArticleList from './articleList';
import Article from './article';
export default function App() {
  return (
    <Routes>
      <Route path="/" element={<ArticleList />} />
      <Route path="/article" element={<Article />} />
    </Routes>
  );
}
```

然后写写articleList和article两个组件的初始化：

```jsx
import React from "react";
export default class ArticleList extends React.Component {
  render() {
    return (
      <div>ArticleList</div>
    );
  }
}
```

可以正常访问，然后开始撸页面就好了唔。

顺带现在长这个样子：<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022020703.webp" width="200px">

## 开始撸页面

### 使用css

[阮一峰-CSS Modules 用法教程](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)

> CSS的规则都是全局的，任何一个组件的样式规则，都对整个页面有效。
>
> 产生局部作用域的唯一方法，就是使用一个独一无二的`class`的名字，不会与其他选择器重名。这就是 CSS Modules 的做法。

算了算了，反正也不急着弄明白啥意思（~~赶紧水完项目要紧~~）简单来说，局部样式和全局样式分别这样用：

* 局部
  * 命名：`style.module.css`
  * 引入：`import style from 'style.module.css'`
  * 使用： `<div className={style[className]}>`
* 全局
  * 命名：`style.css`
  * 引入：`import 'style.css'`
  * 使用：`<div className='className'>`

顺带用一下scss，至少能嵌套一下就很方便qwq

```bash
npm install -d node-sass
```

于是大部分就这样子写了：

```jsx
import React from "react";
import style from "../style.module.scss";
export default class Bottom extends React.Component {
  render() {
    return (
      <div className={ style['bottom'] }>
        <ul>
            <li>最新</li>
            <li>热门</li>
            <li>历史</li>
        </ul>
      </div>
    );
  }
}
```

### 基础页面绘制

写过一堆css之后变成了这个样子：<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022020801.webp" width="200px">

<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022020802.webp" width="1000px">

## 文章列表相关动态操作

### 切换tabs

这里所有tab的状态统一存到了公共的父组件 `/articleList/index.js` 里面，仍然以最底部的导航栏为例，父组件的state里存两个东西：

`bottomTabs: ['热门', '最新', '历史'], activeBottomTab: 0,`

分别表示都有哪些tabs和当前点击的tab是哪个，作为props传给底部导航栏组件处理即可，顺带父组件的changeTab事件也一并作为props传过去便于切换。

index.js:

```jsx
<Bottom
  tabs={ this.state.bottomTabs }
  activeTab={ this.state.activeBottomTab }
  changeTab={ this.changeBottomTab }
/>
```

bottom.js:

```jsx
import React from "react";
import style from "../style.module.scss";
export default class Bottom extends React.Component {
  render() {
    return (
      <div className={style['bottom']}>
        <ul>
          {this.props.tabs.map((item, index) => (
            <li
              onClick={ () => this.props.changeTab(index) }
              // 为选中的组件动态绑定蓝色特效
              className={ this.props.activeTab === index ? style['active-tab'] : '' }
              key={ index }
            >
              {item}
            </li>
          ))}
        </ul>
      </div>
    );
  }
}
```

### 动态渲染文章预览

其实跟tabs差不多，仍然是父组件 `/articleList/index.js` 去存储文章列表，把文章列表传给了 `body.js` 也就是页面主体部分，再由其分别传给每个小卡片。

*感觉这样安排也许会有些不合理的地方，不过主要考虑是，接口获取文章列表也需要当前各种tabs的选中情况，而这些状态都存在了 `index.js` 里，所以为了对接口方便只好这样安排了qwq*

### 对接fake-api动态获取数据

因为是假的接口不需要网络请求，写起来就非常简单了，直接参考 `fake-api/index.js` 里的注释就好：

/articleList/index.js：

```jsx
//...
import { getCategories, getArticles } from "../fake-api";
//...
export default class ArticleList extends React.Component {
  constructor(props) { /*......*/ }
  componentDidMount() {
    this.fetchCategories();
    this.fetchArticles();
  }
  async fetchCategories() { /*......*/ }
  async fetchArticles() { /*......*/ }
  //...
}
```

这样看上去就比较像样子了。

<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022020803.webp" width="600px">

关于无限下拉列表，跟上面提到的一样，这里父子组件关系安排有点乱，不如记录文章评论那里的无限下拉列表比较方便qwq

## 文章内容相关动态操作

### 路由跳转

一种方式：

```jsx
// routes
import { Routes, Route } from 'react-router-dom';
<Routes>
  <Route path="/" element={<ArticleList />} />
  <Route path="/article/:id" element={<Article />} />
</Routes>
// link
import { Link } from "react-router-dom";
<Link
  style={{ textDecoration:'none', color: '#000' }}
  to={ '/article/' + item.article_id }
>
</Link>
// 获取参数
import { useParams } from "react-router-dom";
const { id } = useParams();
```

### 组件功能划分

文件结构长这样：<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022020804.webp" width="250px">

其中 `index.js` 负责从路由获取文章id，分别传给文章主体部分 `body.js` 和评论部分 `comments.js` ，相关内容由这两个组件分别获取。

文章接口可以直接获取html代码，~~无视xss~~使用 `<div dangerouslySetInnerHTML={{__html: this.state.article }}/>` 直接嵌入即可。

### 评论区无限下拉实现

实现思路主要是这些：

* 设定好一个 `加载中` 元素始终位于最底部。
* 使用 `IntersectionObserver` 监听这个 `加载中` 元素与视窗的重合部分。
* 到达一定重合面积（也就是当用户能看到"加载中"这三个字）时，获取新的数据，`offset` 就是本地已获取的数据列表长度。
* 接口提供总数据量 `total`，当本地已获取数据列表长度与之相等时，代表所有数据已经加载完毕。
* 之后清除 `observer` 并将 `加载中` 替换为 `没有更多了`。

~~也许整个组件的代码都可以丢这里占篇幅QwQ~~

comments.js：

```jsx
import React from "react";
import style from "../style.module.scss";
import { getCommentsByArticleId } from "../../fake-api";
export default class Comments extends React.Component {
  constructor(props) {
    super(props);
    this.loading = React.createRef();
    this.state = {
      comments: [],
      observer: null,
      total: 0,
    }
  }
  componentDidMount() {
    // 创建observer，重合10%时开始获取新的数据。
    const observer = new IntersectionObserver(
      () => {
        this.fetchComments();
      },
      {
        threshold: 0.1,
      }
    );
    observer.observe(this.loading.current);
    this.fetchComments();
  }
  // 获取新的数据
  async fetchComments() {
    const response = await getCommentsByArticleId(this.props.id, this.state.comments.length);
    let comments = this.state.comments;
    comments = comments.concat(response.data.comments);
    this.setState({
      comments,
      total: response.total,
    });
  }
  renderCommentCard(comment) {
    /* 渲染一个评论卡片 */
  }
  render() {
    return (
      <div className={style['comments']}>
        <ul>
          { this.state.comments.map(this.renderCommentCard) }
        </ul>
          {/*这个span就是实现无限下拉的关键元素*/}
          <span ref={this.loading} className={ style['loading'] }>
            { this.state.total === this.state.comments.length ? '没有更多了嘤~' : '加载中...' }
          </span>
      </div>
    );
  }
}
```

## 历史记录

简单粗暴，在用户点击一篇文章时直接存localStorage即可：

*这里除了暴力没有想到什么比较好的去重方法，所以~~直接懒得写~~没有写去重qwq*

```jsx
addToHistory(articleInfo) {
  const history = localStorage.getItem('historyArticles');
  if (history) {
    const historyArticles = JSON.parse(history);
    historyArticles.unshift(articleInfo);
    localStorage.setItem('historyArticles', JSON.stringify(historyArticles));
  } else {
    localStorage.setItem('historyArticles', JSON.stringify([articleInfo]));
  }
}
```

在获取文章时，先检查当前选中的tab是否为 `历史` ，如果是，则文章列表直接在localStorage中获取：

```jsx
async fetchArticles() {
  if (this.state.activeBottomTab === 2) {
    this.fetchHistoryArticles();
  } else {
	/*......*/
  }
}
fetchHistoryArticles() {
  const history = localStorage.getItem('historyArticles');
  // 是否有历史记录
  if (history) {
    const historyArticles = JSON.parse(history);
    this.setState({
      articleList: historyArticles,
      totalArticles: historyArticles.length,
    });
  } else {
    this.setState({
      articleList: [],
      totalArticles: 0,
    });
  }
}
```

## 总结

唔姆唔姆，不知道总结什么，但总该有个结尾qwq

普普通通的一个项目叭，算是第一次体验了一下使用react进行开发~~（tictactoe不算）~~，用起来还挺舒服的。

