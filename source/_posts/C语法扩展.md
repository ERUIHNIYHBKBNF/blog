---
title: '[C++笔记] C语法扩展'
date: 2021-01-21 16:32:15
tags: 
---

## 引用 Reference

> 引用就是另一个变量的别名，可以理解为给变量取一个外号。

性质：

* 对引用的操作会改变原变量。
* 声明时必须初始化，非常量引用必须为左值。
* 一旦进行初始化，引用的名字不能再传给其它变量。（外号只对应一个人）

实例-引用传参：

```cpp
#include <iostream>
using namespace std;
void fun(int &x) //此处 x 与 a 的地址相同
{
    x += 2;
}
int main()
{
    int a = 3;
    fun(a);
    /*
    fun(a + 1)会报错，非常量引用必须为左值
    */
    cout << a << endl; //输出为 5 
    return 0;
}
```

 实例-初始化之后不能传给其它变量：

```cpp
#include <iostream>
using namespace std;
int main()
{
    int a = 2, b = 3;
    int& r = a;
    r = b;
    cout << r << ' ' << a << endl; //输出 3 3
    return 0;
}
```

## 空指针与动态内存分配

空指针：

* 为解决0带来的二义性问题，C++11中引入保留字nullptr作为空指针。

动态内存分配：

*  通过**new**运算符申请动态内存
   * new 类型(初始值) 
   * new 类型[数量]

*  通过**delete**运算符释放内存
   * delete 指针

实例：

```cpp
#include <iostream>
using namespace std;
int main()
{
    int* p = nullptr;
    int* q{nullptr};
    p = new int(233);
    q = new int[4];
    cout << *p << endl; // 233
    *(q + 1) = *p;
    cout << q[1] << endl; // 233
    delete p;
    return 0;
}
```

## 列表初始化

> C++逐渐python化（抱头蹲防

C++11标准之前的初始化方法：

```cpp
int x = 0;
int y(2);
int arr[] = {2, 3, 3};
char s[] = "Hello";
```

C++11新特性：

```cpp
// 直接列表初始化
int x{}; // 0
int y{1}; // 1
int arr1[]{2, 3, 3}; // 2, 3, 3
char s1[4]{'q', 'w', 'q'}; // "qwq\0" 多余的初始化为0，对比s3
char s2[]{"qwq"}; // "qwq\0"
// 拷贝列表初始化
int x = {}; //同上，加上等号，一般情况下等价使用
char s3[]  {'a', 'b'}; // "ab"
char s4[] = "ab"; // "ab\0"
```

> 尽量使用列表初始化，除非你有个很好的不用它的理由。

列表初始化不允许丢失数据精度的隐式类型转换。

eg:  `int x{1.1}` 编译器报错。

## 类型转换

* 隐式类型转换
* C风格的强制类型转换: (Type)Value
* C++风格的强制类型转换: **static_cast\<Type\>(Value)**

实例：

```cpp
cout << static_cast<double>(1) / 2 << endl; // 0.5
cout << static_cast<double>(1 / 2) << endl; // 0
```

RTFM : [Runoob_C++ 强制转换运算符](https://www.runoob.com/cplusplus/cpp-casting-operators.html) ~~懒得看了，以后有机会回来补，溜了，，~~

## 自动类型推导:auto与decltype

### auto关键字

> C++03及之前的标准种，auto放在变量声明之前，声明变量的存储策略。但是这个关键字常省略不写。

> C++11中，auto关键字放在变量之前，作用是在声明变量的时候 根据变量初始值的类型 自动为此变量选择匹配的类型。

使用限制：

* auto 变量必须在**定义时初始化**，这类似于const关键字

  ```cpp
  auto a = 10; // 正确 
  auto b; // 无法推断，报错
  b = 10;
  ```

* 定义**在一个auto序列的变量**必须始终推导成**同一类型**

  ```cpp
  auto a = 2, b{3};
  auto c = 4, d{'c'};
  ```

* **去除引用与const**，添加'&'可保留原类型

  ```cpp
  int a{10};
  int& b = a;
  auto c = b;
  c = 5;
  cout << a << ' ' << b << ' ' << c << endl; // 10, 10, 5
  ```

  ```cpp
  int a{10};
  int& b = a;
  auto& c = b;
  c = 5;
  cout << a << ' ' << b << ' ' << c << endl; // 5, 5, 5
  ```

  ```cpp
  const int a{10};
  auto& c = a;
  c = 5; // 报错
  ```

* 初始化为**数组推断为指针**，添加'&'可保留数组

  ```cpp
  int a[]{2, 3, 3};
  auto b = a;
  auto& c = a;
  cout << typeid(b).name() << ' ' << typeid(c).name() << endl; //Pi, A3_i
  ```

  此处注意：

  ```cpp
  int a[]{2, 3, 3};
  auto& c = a; // 虽然c类型是数组，但其仍然是引用a
  c[2] = 1;
  cout << c[2] << ' ' << a[2] << endl; // 1, 1
  ```

* C++14中，auto可以作为函数的返回值类型和参数类型

**A**lmost **A**lawys **A**uto: 尽可能使用auto

一些例子：

```cpp
auto x = 42;
auto x = int{42};
auto x = 42.f;
auto x = 42ul;
auto x = "42"s; //字符串, C++14
auto f(double) -> int; // 声明函数
auto f(double) {...};
```

数组的初始化在运行期间完成，而auto是在编译期间完成，eg:

```cpp
auto x[] = {1, 2, 3}; // 编译器报错
```

### decltype关键字

发音：*declare_type*

> decltype是在编译期推导一个表达式的类型，它只做静态分析，因此它不会导致已知类型表达式执行。

decltype 主要用于泛型编程（模板），~~所以现在先跑路了~~

下篇：[[C++笔记] C语法扩展II](https://eruihniyhbkbnf.github.io/blog/2021/01/21/C%E8%AF%AD%E6%B3%95%E6%89%A9%E5%B1%95II/)

