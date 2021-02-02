---
title: '[C++笔记] string类与array类'
date: 2021-01-27 11:26:12
tags: [C++, C++笔记, 模板]
---

## string类

### 创建string对象

```cpp
string str1;

string str2{"qwq"};

string str3 = "qwq";

char str4[] = {'q', 'w', 'q', '\0'};
string str5{str3};
```

### 常用函数

很多内置函数接收的两个参数：`index` 起始位置， `base` 往后base个字符。 范围 $[index, index + base)$

* `s.append((times), string/char, (index), (base)) ` 追加字符串

* `s.asign((times), string/char, (index), (base))` 重新赋值

  ```cpp
  string s1{ "Welcome" };
  s1.append( " to C++" ); // Welcome to C++
  
  string s2{ "Welcome" };
  s2.append( " to C and C++", 3, 2 ); // Welcome C
  
  string s3{ "Welcome" };
  s3.append( " to C and C++", 5); // Welcome to C
  
  string s4{ "Welcome" }; 
  s4.append( 4, 'G' ); // WelcomeGGGG
  // 此处'G'仅能是字符而不是字符串
  ```

* `s.at(index)` 返回index位置字符。

* `s.substr(index, (base))` 获取对应区间子串，不指定n则为到末尾。

* `s.find(string/char, (index))` 从index开始查找(不指定则为开头)，返回第一次出现的位置。

* `s.clear()` 清空字符串。

* `s.erase(index, base)` 删除对应区间字符。

* `s.empty()` 判断是否为空。

  ```cpp
  string s1{ "Welcome" };
  cout << s1.at(3) << endl; // c
  cout << s1.erase(2, 3) << endl; // Weme
  cout << s1.substr(1, 2) << endl; //em
  cout << s1.find('e') << endl; // 1
  s1.clear();
  cout << s1.empty() << endl; // 1
  ```

* `s.compare(string)` 比较字符串（对应ascii码作差 s1 - s2)

  ```cpp
  string s1{ "Welcome" };
  string s2{ "Welcomg" };
  cout << s1.compare(s2) << endl; // -2
  cout << s2.compare(s1) << endl; // 2
  cout << s1.compare("Welcome") << endl; // 0
  ```

* `s.insert(index, string)` 插入。

* `s.replace(index, base, string)` 替换，*注意 \0 占一个位置*。

  ```cpp
  string s1("Welcome to C++");
  s1.insert(11, "Java and "); // Welcome to Java and C++
  
  string s2{ "AA" };
  s2.insert(1, 4, 'B'); // ABBBBA
  
  string s3{ "Welcome to Java" };
  s3.replace(11, 4, "C++"); // Welcome to C++ 
  ```

* `stoi(string)` 转换为整数

### 运算符

| Operator             | Description                            |
| -------------------- | -------------------------------------- |
| [ ]                  | 下标访问                               |
| =                    | 内容复制                               |
| +                    | 连接，返回新串 *（这是好文明）*        |
| +=                   | 末尾追加                               |
| <<                   | 字符串插入一个流（目前好像只有cout）   |
| >>                   | 从一个流提取字符串，分界为空格或结束符 |
| ==, !=, <, <=, >, >= | 字符串比较                             |

## array类 (C++11)

### C风格数组的缺陷

* 可能退化为指针
* 无法确定大小
* 无法直接赋值
* 无法自动推断

### C++风格的数组

模板类，可容纳任何类型数据。

```cpp
#include <array>
std::array <Type, Size> ArrayName; //可进行列表初始化
```

C++17新特性：对类模板的参数进行推导（类型，大小）

### array成员函数

常用：

* `a.at(index)` 访问元素。 已重载 `[]` 运算符。
* `a.front()` `a.back()` 首元素，尾元素。
* `a.size()` `a.empty()` 大小，判空。
* `a.fill(val)` 用val填满数组。
* `a.swap(b)` 互换 a, b 数组。
* `a.begin()` `a.end()` 首尾指针。

实例：

```cpp
#include <iostream>
#include <array>
#include <algorithm>
using namespace std;
void print(array <int, 3>& arr) //引用避免拷贝，效率高一seisei
{
    for (auto x : arr)
        cout << x;
    cout << endl;
}
int main()
{
    array <int, 3> a{2, 3, 3};
    print(a); // 233
//赋值
    a = {4, 6, 6};
    print(a); // 466
//交换数组        
    array <int, 3> b{3, 2, 1};
    a.swap(b);
    print(a); // 321
    print(b); // 466
//大小
    cout << a.size() << ' ' << a.max_size() << endl; // 3 3
//排序
    sort(a.begin(), a.end());
    print(a); // 123
    sort(a.rbegin(), a.rend());
    print(a); // 321
//其它函数测试
    a.fill(0);
    print(a); // 000
    cout << a.empty() << endl; // 0
//二维
    array <array<int, 3>, 4> c;
    return 0;
}
```

比较全~~但用处不大~~的表格：

| 元素访问                                                     |                                      |
| ------------------------------------------------------------ | ------------------------------------ |
| [at](https://zh.cppreference.com/w/cpp/container/array/at)   | 访问指定的元素，同时进行越界检查     |
| [operator](https://zh.cppreference.com/w/cpp/container/array/operator_at)[] | 访问指定的元素 , 无越界检查          |
| [front](https://zh.cppreference.com/w/cpp/container/array/front) | 访问第一个元素                       |
| [back](https://zh.cppreference.com/w/cpp/container/array/back) | 访问最后一个元素                     |
| [data](https://zh.cppreference.com/w/cpp/container/array/data) | 返回指向内存中数组第一个元素的指针   |
| **容量**                                                     |                                      |
| [empty](https://zh.cppreference.com/w/cpp/container/array/empty) | 检查容器是否为空                     |
| [size](https://zh.cppreference.com/w/cpp/container/array/size) | 返回容纳的元素数                     |
| [max_size](https://zh.cppreference.com/w/cpp/container/array/max_size) | 返回可容纳的最大元素数               |
| **操作**                                                     |                                      |
| [fill](https://zh.cppreference.com/w/cpp/container/array/fill) | 以指定值填充容器                     |
| [swap](https://zh.cppreference.com/w/cpp/container/array/swap) | 交换内容                             |
| **迭代器**                                                   |                                      |
| [beginc](https://zh.cppreference.com/w/cpp/container/array/begin)    [begin](https://zh.cppreference.com/w/cpp/container/array/begin) | 返回指向容器第一个元素的迭代器       |
| [endc](https://zh.cppreference.com/w/cpp/container/array/end)    [end](https://zh.cppreference.com/w/cpp/container/array/end) | 返回指向容器尾端的迭代器             |
| [rbeginc](https://zh.cppreference.com/w/cpp/container/array/rbegin)    [rbegin](https://zh.cppreference.com/w/cpp/container/array/rbegin) | 返回指向容器最后元素的**逆向迭代器** |
| [rendc ](https://zh.cppreference.com/w/cpp/container/array/rend)   [rend](https://zh.cppreference.com/w/cpp/container/array/rend) | 返回指向前端的逆向迭代器             |

`rbegin()` 和 `rend()` 用来比如从大到小排序什么的老好用了qwq

