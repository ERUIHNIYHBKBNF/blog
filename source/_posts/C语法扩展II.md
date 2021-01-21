---
title: '[C++笔记]杂七杂八的C语法扩展II'
date: 2021-01-21 23:27:30
tags:
---

接上篇[[C++笔记]杂七杂八的C语法扩展]()， 内容太多就分成两篇了。（~~顺带可以多水一篇文章~~）

## 内存四区

* 栈Stack：编译器自动分配释放
* 堆Heap：程序员手动分配释放，结束时OS回收未释放的部分
* 全局区 / 静态区 Global / Static：全局/静态变量，程序结束时释放
* 常量区Const：不可修改 

## 常量与指针

### 常量

声明： **const Type const_name = value** 其中 const 与 Type 可互换位置。

ex: `const char* STR = "Hello";` 不加const会报错*(试出来是warning?)*，因为"Hello"存放在常量区，指针要指向常量区。（可以看下面qwq）

### 指针

开始念咒....常量指针，指针常量，常量指针常量...阿巴阿巴(º﹃º )

其实就是**const离谁近修饰谁**：

```cpp
// 指针
int x = 5;
int *p = &x;

// 常量指针(常指针)
const int x = 5;
const int* p = &x; //const修饰int*

// 指针常量
int x = 5;
int* const p = &x; //const修饰p

// 常量指针常量
const int x = 5; //似乎不写const也可
const int* const p = &x; //不能通过*p修改x的值，且不能修改p的值

int x = 2;
auto p1 = &x; // p1类型int*
auto p2 = "qwq"; // p2类型const char*
auto const p3 = "owo"; // p3类型const char* const
```

## 各种宏定义

*  **\#define**： 仅用于替换 `#define macro_name sth.`

* **typedef**： 创建代替类型名的别名`Typedef Type NewTypeName`

* **using**替代typedef： `using identifier = type-id`

  只能用于类型，`using in = std::cin`报错，因为cin是类。

  实例：

  ```cpp
  using ull = unsigned long long;
  ull x = 233u;
  using Fptr = void(*)(int, int); //函数指针
  void example(int, int){...};
  Fptr fp = example;
  ```

  **定义模板的别名，只能使用using**

## 变量作用域

### 局部作用域

文件作用域、函数作用域以及函数内部的块作用域。

就近原则：作用域不同的同名变量，优先使用作用域小的。

实例：

```cpp
#include <iostream>
using namespace std;
int a = 3;
int main()
{
    int a = 2;
    cout << a << endl; // 2
    return 0;
}
```

### 一元作用域解析运算符

局部变量与全局变量同名时，使用`::`访问全局变量

实例：

```cpp
#include <iostream>
using namespace std;
int a = 3;
int main()
{
    int a = 2;
    cout << a << ' ' << ::a << endl; // 2, 3
    return 0;
}
```

## 函数相关

### 重载函数Overloading Funcitons

假设已定义函数 `int mx(int, int)` 再定义 `double mx(double, double)`（一个同名不同参的函数），调用函数 `mx(v1, v2)`时：

编译器**仅通过参数**来匹配重载函数的调用，优先级：

1. 参数个数
2. 参数类型
3. 不同类型参数顺序

实例：

```cpp
#include <iostream>
using namespace std;
int mx(int num1, int num2)
{
    return num1 > num2 ? num1 : num2;
}
double mx(double num1, double num2)
{
    return num1 < num2 ? num1 : num2;
}
int main()
{
    cout << mx(2, 3) << endl; //3
    cout << mx(2.0, 3.0) << endl; //2
    cout << mx(2.0, 3) << endl; //报错
    return 0;
}
```

**二义调用**会导致编译错误，例如 `int fun(int, double)` 与 `double fun(double, int)`，调用 `fun(1, 2)` 时会报错。又例如同时声明 `int fun(int, int)` 与 `void fun(int, int)` 编译器报错。

### 带有默认参数值的函数

`Type Fun(v1, v2, v3 = , v4 = )`之类的写法。例如左边，调用时可传递2 ~ 4 个参数，未接收实参的形参有默认值。

需要注意的是**带有默认值的参数在右边**，**传递实参时依次传给左边**。

实例：

```cpp
#include <iostream>
using namespace std;
int mx(int a, int b = 3)
{
    return a > b ? a : b;
}
int main()
{
    cout << mx(2) << endl; // 3
    cout << mx(2, 4) << endl; // 4
    return 0;
}
```

### 默认参数与重载

**函数重载时，不允许重定义默认参数**

以下实例均为错误：

```cpp
double area(double r = 1){...}
double area(int r = 2){...}
----
int add(int x, int y = 3)
{
	return x + y;
}
int add(int x)
{
    return x + 3;
}
add(x);
```

~~反正一般编译器搞不定的这种问题都会报错qwq~~

### 内联函数inline

> 一句话：用空间换取时间。

声明方法： `inline Fun(...){...}`， 内联函数定义与声明一般不分开。

仅适用于频繁调用的短函数，是**对编译器的建议**，并不一定都会进行内联编译(循环，递归，静态变量等)。

## 基于范围的for循环

> ~~python党狂喜~~ 文明进化的象征！:satisfied::satisfied::satisfied:

语法： `for (元素名变量 : 广义集合) {循环体}`

广义集合例子：

```cpp
auto a1[]{1, 3, 5, 7}; // 不再推荐使用原始数组
std::array <int, 4> a2{2, 4, 6, 8};
std::vector <int> v = {2, 3, 3};
std::vector <std::string> s{"qwq", "owo"};
```

要对广义集合里的元素进行操作，需要使用引用。

实例：打印数组a元素并将其翻倍。

```cpp
int a[]{2, 3, 3}; //写auto会报错qwq
for (auto i : a)
    cout << a << endl;
for (auto& i : a)
    i *= 2;
```

## 带有初始化器的if和switch[C++17]

熟悉的初始化器： `for (initializer; condition ; increment)`

然后 `if` 与 `switch` 也能用：`if (initializer; condition)` `switch(initializer; condition)`

实例：

```cpp
int main()
{
    //比起原先在外面定义x，改变了作用域
    if (auto x =  fun(233); x > 123)
    {
        //do something with x
    }
    else
    {
        //do something with x
    }
    auto x = 233; //此处变量名x可重用
}
```

