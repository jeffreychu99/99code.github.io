---
layout: post
title: C++编写Node.js插件(Addon)
categories: Node.js
description: C++编写Node.js插件(Addon)
keywords: Node.js, Addon, Node.js插件
---

Google V8引擎的性能无用质疑，不过相对C/C++而言，还是有差距的，毕竟JavaScript是脚本语言。对于性能要求苛刻的可以考虑C++编写，本文介绍如何使用C++编写Node.js插件。

# 编写C++代码

```cpp
// hello.cc
#include <node.h>

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void Method(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world"));
}

void init(Local<Object> exports) {
  NODE_SET_METHOD(exports, "hello", Method);
}

NODE_MODULE(addon, init)

}  // namespace demo
```

# 编写构建脚本building.gyp文件

```javascript
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "hello.cc" ]
    }
  ]
}
```

# 编写package.json

```javascript
{
  "name": "node",
  "version": "1.0.0",
  "description": "",
  "main": "test.js",
  "dependencies": {},
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "install": "node-gyp rebuild"
  },
  "author": "",
  "license": "ISC",
  "gypfile": true
}
```

# 安装

npm install

系统需要安装python工具，下载地址：https://www.python.org/downloads/release/python-2712/

![](/images/posts/node-addon/1.png)

# 测试运行

```javascript
// hello.js
const addon = require('./build/Release/addon');

console.log(addon.hello()); // 'world'
```

![](/images/posts/node-addon/2.png)

完整代码参考：https://git.oschina.net/zhujf21st/node-cc.git