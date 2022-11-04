---
title: Node C++ addons初探
date: 2022-11-04 13:00:23
tags: [node, C++]
---

发现了个好玩的东西，来学一手（实际上就是跟文档随便过一下再拿中文随便记录一下）：https://nodejs.org/dist/latest-v18.x/docs/api/addons.html

> *Addons* are dynamically-linked shared objects written in C++. The `require()` function can load addons as ordinary Node.js modules. Addons provide an interface between JavaScript and C/C++ libraries.

共享模块，一眼看上去好像wasm的说 |･ω･｀)

使用C++编写源码，编译成`addon.node`这样的二进制文件供node调用。

似乎要先面熟一些东西：

- V8：node用来执行js的C++库，萌新阶段就听说的牛逼玩意儿QwQ

- libuv：用来实现node的event loop, worker, asynchronous的C++库
  - > Addon authors should avoid blocking the event loop with I/O or other time-intensive tasks by offloading work via libuv to non-blocking system operations, worker threads, or a custom use of libuv threads.

- Internal Node.js libraries：node自身提供一些addon可以使用的C++ API，最重要比如`node::ObjectWrap`

- 只有 `libuv, OpenSSL, V8, and zlib symbols` 被node重新暴露出来，其他似乎要手动链接qwq

## 编写Hello World

先跟官网来粘一份代码：

```cpp
// hello.cc
#include <node.h>

namespace demo
{

  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void Method(const FunctionCallbackInfo<Value> &args)
  {
    Isolate *isolate = args.GetIsolate();
    args.GetReturnValue().Set(String::NewFromUtf8(
                                  isolate, "world")
                                  .ToLocalChecked());
  }

  void Initialize(Local<Object> exports)
  {
    NODE_SET_METHOD(exports, "hello", Method);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize)

} // namespace demo
```

所有node addon必须像这样暴露一个初始化函数：

```cpp
void Initialize(Local<Object> exports);
NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize) // 后面没分号
```

The `module_name` must match the filename of the final binary (excluding the `.node` suffix).

上面例子中，the initialization function is `Initialize` and the addon module name is `addon`.（先比着抄就完事了）

## 构建

这里使用node-gyp来编译写好的C++成一个二进制文件`addon.node`

node-gyp作为npm内置的一部分，被设计为正常情况下只有在npm install装一些node addons的时候才可以用内置的node-gyp来针对平台编译addon，自己用的话需要手动安装node-gyp，参考[github仓库](https://github.com/nodejs/node-gyp#installation)。

小插曲：不知道为啥遇到一堆网络问题，npm, yarn, tencent都试了一遍，还是cnpm最顶（顺带安利一个叫nrm的工具，顺带半个晚上进去了QAQ）

在项目根目录新建一个叫 `binding.gyp` 的配置文件，供node-gyp来编译addon

binding.gyp：

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": ["hello.cc"] // 这里注意cc文件的路径
    }
  ]
}
```

然后使用 `node-gyp configure` 会发现新增了一个 `build`文件夹，这边mac平台里面有个Makefile，然后再 `node-gyp build` 会发现新增了 `build/Release/addon.node`，就编译完了

## 在node中使用addon

直接在node中使用 `require`即可引入刚刚编译的模块（像调用普通模块一样调用二进制文件，怎么这么像wasm..）

hello.js：

```js
const addon = require('./build/Release/addon');

console.log(addon.hello());
// Prints: 'world'
```

编译的二进制文件路径可能因为编译方式不同而不同，可以使用[node-bindings](https://github.com/TooTallNate/node-bindings)来确定这个路径。

```Bash
yarn add bindings
import bindings from 'bindings';

const addon = bindings('addon.node');

console.log(addon.hello());
```

## 传递参数

还是官网的例子：

addon.cc：

```cpp
#include <node.h>

namespace demo
{
  using v8::Exception;
  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Number;
  using v8::Object;
  using v8::String;
  using v8::Value;

  // This is the implementation of the "add" method
  // Input arguments are passed using the
  // const FunctionCallbackInfo<Value>& args struct
  void Add(const FunctionCallbackInfo<Value> &args)
  {
    Isolate *isolate = args.GetIsolate();

    // Check the number of arguments passed.
    if (args.Length() < 2)
    {
      // Throw an Error that is passed back to JavaScript
      isolate->ThrowException(Exception::TypeError(
          String::NewFromUtf8(isolate,
                              "Wrong number of arguments")
              .ToLocalChecked()));
      return;
    }

    // Check the argument types
    if (!args[0]->IsNumber() || !args[1]->IsNumber())
    {
      isolate->ThrowException(Exception::TypeError(
          String::NewFromUtf8(isolate,
                              "Wrong arguments")
              .ToLocalChecked()));
      return;
    }

    // Perform the operation
    double value =
        args[0].As<Number>()->Value() + args[1].As<Number>()->Value();
    Local<Number> num = Number::New(isolate, value);

    // Set the return value (using the passed in
    // FunctionCallbackInfo<Value>&)
    args.GetReturnValue().Set(num);
  }

  void Init(Local<Object> exports)
  {
    NODE_SET_METHOD(exports, "add", Add);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

} // namespace demo
```

## 总结

这个看上去像wasm的东西大概优点就跟wasm差不多叭，能在node里跑C++的话对性能提升不用说，还有些js实现不了的奇奇怪怪的操作（比如获取某个变量在内存中的表示形式qwq）

顺带隔壁deno什么时候出个Rust addons呀
