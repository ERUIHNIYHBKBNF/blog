---
title: '[C++学习笔记] C++中的namespace'
date: 2021-01-18 08:51:29
tags:
---

>  内容~~CPOY~~参考自菜鸟教程：[C++ 命名空间](https://www.runoob.com/cplusplus/cpp-namespaces.html)

## namespace std

> C++标准库中的所有标识符被包含于名为std的名字空间中。

### 引用方式

* 直接全局使用

  ```cpp
  #include <iostream>
  using namespace std;
  int main()
  {
      cout << "Hello Chino." << endl;
      return 0;
  }
  ```

  **为了防止出现各种奇奇怪怪的错误，需要尽量避免直接全局使用 namespace std**

  ~~然而打比赛的时候还是直接用了QwQ~~

* 全局使用内部的某个特定标识符

  ```cpp
  #include <iostream>
  using std::cout;
  using std::endl;
  int main()
  {
      cout << "Hello Chino." << endl;
      return 0;
  }
  ```

* 局部使用

  ```cpp
  #include <iostream>
  void f1()
  {
      cout << "qwq" << endl; //报错
  }
  int main()
  {
      using namespace std;
      cout << "Hello Chino." << endl;
      f1();
      return 0;
  }
  ```

* 每次使用标识符时指定所属namespace

  ```cpp
  #include <iostream>
  void f1()
  {
      std::cout << "qwq" << std::endl;
  }
  int main()
  {
      using namespace std;
      cout << "Hello Chino." << endl;
      f1();
      return 0;
  }
  ```

## 自定义namespace

### 定义与引用方式

* 就。。直接定义呗qwq

  ```cpp
  #include <iostream>
  namespace n1
  {
      void fun()
      {
          std::cout << "qwq" << std::endl;
      }
  }
  int main()
  {
      n1::fun();
      return 0;
  }
  ```

* 不允许在主函数内进行定义

  ```cpp
  int main()
  {
      namespace n1 //报错
      {
          void fun()
          {
              std::cout << "qwq" << std::endl;
          }
      }
      n1::fun();
      return 0;
  }
  ```

* 嵌套定义

  ```cpp
  #include <iostream>
  namespace n
  {
      namespace n1
      {
          void fun()
          {
              std::cout << "qwq" << std::endl;
          }
      }
      namespace n2
      {
          void fun()
          {
              std::cout << "owo" << std::endl;
          }
      }
  }
  int main()
  {
      using namespace n::n2;
      fun(); // owo
      n::n1::fun(); // qwq
      return 0;
  }
  ```

### 其它奇奇怪怪的东西

* 类似变量的用法？

  ```cpp
  #include <iostream>
  namespace namespace1
  {
      void fun()
      {
          std::cout << "qwq" << std::endl;
      }
  }
  namespace n = namespace1;
  int main()
  {
      using namespace n;
      fun();
      return 0;
  }
  ```

* 在当前编译单元使用，避免在其他地方撞名

  没搞过大工程咱也不是很懂QAQ, 以后慢慢来叭..

  > 参考自：https://www.cnblogs.com/lialong1st/p/12006169.html

  ```cpp
  #include <iostream>
  namespace
  {
      void fun()
      {
          std::cout << "qwq" << std::endl;
      }
  }
  int main()
  {
      fun();
      system("pause");
      return 0;
  }
  ```


### 能干啥用

​	布叽道QAQ,，用到了回来再写写~

​	防止变量名一直是a, b, c...并且为了避免撞名字，起名废的福音(?)

​	溜了溜了，。，。