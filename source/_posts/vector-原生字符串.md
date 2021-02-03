---
title: '[C++笔记] vector,stack,原生字符串'
date: 2021-02-03 21:30:17
tags: [C++, C++笔记]
---

## vector类

让咱先来秀(づ￣ 3￣)づ

> 循秩访问，插入删除遍历查询替换，加倍扩容，分摊复杂度，置乱，无序查找，有序二分查找，去重，排序blabla...~~摸了摸了~~

### 声明

理解为可变大小的数组，自动调整大小，用 `vector <TypeName> Name;` 声明。

解决初始数据规模未知 ~~又懒得手写动态数组~~ 的问题， **常与\<algorithm\>库混合使用**，~~所以懒啊~~。

### 常用函数

常用函数与array类似：[[C++笔记] string类与array类](https://eruihniyhbkbnf.github.io/blog/2021/01/27/string与array类/)

实例：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
void print(vector <int>& v)
{
    for (auto i : v)
        cout << i << ' ';
    cout << endl;
}
int main()
{
    vector <int> v{2, 3, 1, 4}; // C++17已引入对模板类型自动推导，C++17可省略<int>
    cout << v.size() << ' ' << v.empty() << endl; // 4 0
    v.pop_back();
    print(v); // 2 3 1
    sort(v.begin(), v.end());
    print(v); // 1 2 3
    sort(v.rbegin(), v.rend());
    print(v); // 3 2 1
    v.push_back(4);
    print(v); // 3 2 1 4
    reverse(v.begin() + 1, v.end() - 1);
    print(v); // 3 1 2 4 
    vector <int> v2(v.begin(), v.end() - 1);
    print(v2); //3 1 2
    v2.insert(v2.begin() + 1, 4);
    print(v2); // 3 4 1 2 
    vector <int> v3(5, 3);
    v3[0] = 2;
    print(v3); // 2 3 3 3 3
    v2.swap(v3);
    print(v2); // 2 3 3 3 3
    print(v3); // 3 4 1 2
    return 0;
}
```

## C++11原生字符串

消除转义符，方便正则之类的书写，，~~所以学正则的时候再说，现在摸了~~

使用方法： `R"delimter(raw_characters)delimter"`  //delimter两侧相同即可，可不加。

*其实就是括号里的内容*，括号里的内容会被**原封不动地**照搬。 

<img src="https://s3.ax1x.com/2021/02/03/yQxDeI.png" width="500px">

据说还支持Unicode什么的，现在用不到先摸了(´▽｀)ノ♪

## C++14字符串字面量

已对 `""s` 进行重载表示字符串字面量，区别于 `const char*`。例如auto的推断。

## stack类

写好的栈，用法同vector，下列区别：

| 函数     | 功能         |
| -------- | ------------ |
| s.top()  | 返回栈顶元素 |
| s.push() | 压入栈顶     |
| s.pop()  | 弹出栈顶     |