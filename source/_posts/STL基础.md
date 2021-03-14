---
title: '[C++笔记] STL基础'
date: 2021-03-12 10:25:16
tags: [C++, C++笔记]
---

早就想补这部分东西了(｀･ω･´)ﾉ  ~~所以拖到了现在 也就是正式学习的时候QAQ~~

## STL组成

* Container 容器
  * 用于保存一组数据，数据个体称元素。
  * 分类
    * 顺序容器：线性数据结构，多个元素的有序集合，有头有尾有前有后： `vector, list(链表), deque`
    * 关联容器：可快速对应元素的非线性数据结构，可存储键值对： `set, multiset, map, multimap`
    * 容器适配器：顺序容器的受限版本： `stack, queue, priority_queue`
* Iterator 迭代器
  * 用于遍历容器中的元素，例如数组指针。
  * 实现：将\*, ->, ++等指针相关操作重载的类模板。
  * 每个容器都有专有的迭代器。
* Algorithm 算法
  * 独立于容器。
* Function Objects 函数对象
* Memory Allocation 空间分配器

## STL容器

也许有用的表格：

<img src="https://s3.ax1x.com/2021/02/10/y005PH.png" width="750px">

<img src="https://s3.ax1x.com/2021/02/10/y00hIe.png" width="750px">

<img src="https://s3.ax1x.com/2021/02/10/y00faD.png" width="750px">

## STL迭代器

* 用于访问和处理一级容器（包含顺序容器与关联容器）中的元素。
* 一级中的某些函数也与迭代器有关：`begin(), end() ...`

实例：

```cpp
int a[]{0, 1, 2, 3, 4, 5};
vector<int> v(a + 1, a + 4 + 1); // 神奇的初始化方式[begin, end)
vector<int>::iterator p1;
for (p1 = v.begin(); p1 != v.end(); p1++)
    cout << *p1 << ' '; // 1 2 3 4
cout << endl;
p1 -= 2;
cout << *p1 << endl; // 3
cout << p1[-2] << endl; // 1
p1[0] = 233;
cout << v[2] << endl; // 233
```

### 迭代器类型

分成五类：

* 读写：
  * Input iterators 输入，可读取
  * Output iterators 输出，可修改
* 变化方式：
  * Forward iterators 前向，单步向前移动
  * Bidirectional  iterators 双向，单步前后移动
  * Random access  iterators 随机，任意跳转

也许有用的表格：

<img src="https://s3.ax1x.com/2021/02/10/y06yxH.png" width="750px">

## ex

#### 顺序容器

双向迭代器： `list`

随机迭代器：`vector, deque`

也许有用的表格：

<img src="https://s3.ax1x.com/2021/03/11/6NM44x.png" width="500px">

### 关联容器

双向迭代器： `set, multiset, map, multimap`

* 使用key快速存取元素~~（但是总是被人说常数爆炸）~~
* 元素按规则排序，默认使用 `<` 排序。

也许有用的表格（和崔毅东老师帅气的额头）：

<img src="https://s3.ax1x.com/2021/03/12/6NbzkV.png" width="750px">

set:

```cpp
set <int, greater<int> > s{1, 2, 3, 4}; //greater反向
set<int>::iterator p;
for (p = s.begin(); p != s.end(); p++)
    cout << *p << ' '; //4 3 2 1
s.insert(5);
p = s.find(5);
if (p == s.end())
    cout << "Not Found\n";
else
    cout << "Found\n"; // <=
s.erase(4);
p = s.find(4);
if (p == s.end())
    cout << "Not Found\n"; // <=
else
    cout << "Found\n";
for (p = s.begin(); p != s.end(); p++)
    cout << *p << ' '; //5 3 2 1
```

map:

```cpp
map<string, int> m;
m.insert(map<string, int>::value_type("qwq", 1));
m["owo"] = 2;
cout << m["qwq"] <<' '<< m["owo"] << endl; //1 2
map<string, int>::iterator p;
for (p = m.begin(); p != m.end(); p++)
    cout << p -> first << ':' << p -> second << endl; //owo:2 qwq:1
m.erase("qwq");
p = m.find("qwq");
if (p == m.end())
    cout << "QAQ" << endl; // <=
else
    cout << "OwO" << endl;
```

 在Menci看不见的地方膜Menci： https://oi.men.ci/stl-in-oi/