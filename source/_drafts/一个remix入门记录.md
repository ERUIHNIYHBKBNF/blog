---
title: 一个remix入门记录
date: 2022-02-21 14:40:37
tags: [前端, 全栈, react]
---

> [Remix - Build Better Websites](https://remix.run/)

~~入门记录（x），翻译文档（√）~~

## 什么是Remix

Remix是一个可以让你专注于用户界面并且重新学习 Web 基础知识来为用户提供快速流畅的用户体验的全栈 Web 框架。

它是由 React Router 的团队研发的一个基于 React 的框架，相对于其他的框架，这个框架的一个特点就是专注于 “嵌套路由” 这个概念，允许组件直接与其他页面相连接，简化代码的编写。

前置技能： React, Typescript.

## Quickstart体验

环境需求：

* Node.js >= 17
* npm >= 7
* 一个代码编辑器qwq

### 创建启动项目

```sh
npx create-remix@latest
# IMPORTANT: Choose "Remix App Server" when prompted 选择"Remix App Server"
cd [whatever you named the project]
npm run dev
```

文件结构是这样的：<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022022101.webp" width="200px">

```sh
npm run postinstall
npm run dev
```

### 添加路由

接下来添加一个 `/post` 的路由，文件在 `app/routes/index.tsx` 

```tsx
import { Link } from "remix";
// 随便找个地方写进去
<Link to="/posts">Posts</Link>
```

然后新建 `app/routes/posts/index.tsx`，随便写点什么。

```tsx
export default function Posts() {
  return (
    <div>
      <h1>Posts</h1>
    </div>
  );
}
```

### 加载数据

> In Remix your frontend component is also its own API route and it already knows how to talk to itself on the server from the browser. That is, you don't have to fetch it.

修改posts组件，这样相当于获取数据：

```tsx
import { Link, useLoaderData } from "remix";

export type Post = {
  slug: string;
  title: string;
};

export const loader: () => Promise<Post[]> = async () => {
  const posts: Post[] = [
    {
      slug: "my-first-post",
      title: "My First Post"
    },
    {
      slug: "90s-mixtape",
      title: "A Mixtape I Made Just For You"
    }
  ];
  return posts;
};

export default function Posts() {
  const posts = useLoaderData<Post[]>();
  console.log(posts);
  return (
    <div>
      <h1>Posts</h1>
      <ul>
        {posts.map(post => (
          <li key={post.slug}>
            <Link to={post.slug}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

再重构一下：

新建 `app/post.ts`:

```tsx
export type Post = {
  slug: string;
  title: string;
};

export function getPosts() {
  const posts: Post[] = [
    {
      slug: "my-first-post",
      title: "My First Post"
    },
    {
      slug: "90s-mixtape",
      title: "A Mixtape I Made Just For You"
    }
  ];
  return posts;
}
```

`app/routes/posts/index.tsx`:

```tsx
import { Link, useLoaderData } from "remix";

import { getPosts } from "~/post";
import type { Post } from "~/post";

export const loader = async () => {
  return getPosts();
};

// ...
```

