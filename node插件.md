# 编写模板

```c++
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
  args.GetReturnValue().Set(String::NewFromUtf8(
      isolate, "world").ToLocalChecked());
}

void Initialize(Local<Object> exports) {
  NODE_SET_METHOD(exports, "hello", Method);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize)

}  // namespace demo
```

- Isolate：V8 工具类，代表一个 V8 引擎实例。
- Local：V8 存储对象的结构，代表了内存分配释放管理的对象引用。



向外导出方法，必须要使用**NODE_SET_METHOD**，导出一个方法就写一个，如果需要导出多个的话，那么你就写多个

`Local < Number > num = Number: :New(isolate, value);`通过这个方法可以把c++的数据类型转为js的数据类型，返回出去。



# 上下文感知插件

## 应用环境

可能要在多个上下文中加载node插件，例如单个进程运行多个node实例，每个实例都有自己的require缓存。那么就意味着插件必须支持多个初始化

## 构建方法

方法1：使用`NODE_MODULE_INITIALIZER` 构建上下文感知插件

```c++
using namespace v8;

extern "C" NODE_MODULE_EXPORT void
NODE_MODULE_INITIALIZER(Local<Object> exports,
                        Local<Value> module,
                        Local<Context> context) {
  /* 在此处执行插件初始化步骤。 */
}
```



方法2：使用`NODE_MODULE_INIT()`

在调用 `NODE_MODULE_INIT()` 之后，可以在函数体内使用以下三个变量：

- `Local<Object> exports`，
- `Local<Value> module`，和
- `Local<Context> context`

## 区别

`NODE_MODULE_INITIALIZER`围绕初始化构造插件，而 `NODE_MODULE_INIT()` 用作此类初始化器的声明

# Node.js原生抽象

在上面的例子中我们都是直接使用v8的api，但是对于v8的api可能会经常发生变化，所以我们可以使用node.js原生抽象nan，他在v8这一层进行了一个抽象，用来帮助开发者进行兼容，过去的版本和未来的版本，我们开发中尽量使用他

# Node-api

他独立于底层JavaScript运行时，他旨在将插件与底层js引擎隔离开来，**并允许为一个版本编译的模块无需重新编译即可在更高版本的 Node.js 上运行**，我个人理解就是说，即使底层的js发生了变化也不会影响到你插件需要重写

它的区别和node原生抽象和原生api集的区别就是，你使用的式node-api的可用函数，而不是v8和nan

**可以兼容不同的node版本，使得即使你的node升级了，但是你的c++插件不需要重新编译，这就是它的优点**

```c++
// 使用 Node-API 的 hello.cc
#include <node_api.h>

namespace demo {

    napi_value Method(napi_env env, napi_callback_info args) {
        napi_value greeting;
        napi_status status;

        status = napi_create_string_utf8(env, "world", NAPI_AUTO_LENGTH, &greeting);
        if (status != napi_ok) return nullptr;
        return greeting;
    }

    napi_value init(napi_env env, napi_value exports) {
        napi_status status;
        napi_value fn;

        status = napi_create_function(env, nullptr, 0, Method, nullptr, &fn);
        if (status != napi_ok) return nullptr;

        status = napi_set_named_property(env, exports, "hello", fn);
        if (status != napi_ok) return nullptr;
        return exports;
    }

    NAPI_MODULE(NODE_GYP_MODULE_NAME, init)

}  // namespace demo
```

# nan和node-api区别

https://www.css3.io/nan-vs-n-api.html

node-api是nan基础上的更进一步的发展，它的作用都是希望能够让你的代码能够兼容不同的node版本，但是他们的实现存在一定的区别。

nan是使用宏来编写的，**通过定义一系列的宏**，让你的代码在不同的node版本下调用不同的方法，但是他有一个缺点就是，因为用宏，当你的node版本升级过后你还是需要重新编译一次，但是你不需要改你的代码而已。

node-api，希望连重新编译这一步都不需要，他不采用宏的形式，而是使用运行时代码来区分api的调用，那么这样就不需要重新编译一次那么麻烦

# 插件各种基础写法

## 加入回调函数

```c++
void RunCallback(const FunctionCallbackInfo<Value>& args) {
        Isolate* isolate = args.GetIsolate();
        Local<Context> context = isolate->GetCurrentContext();
        Local<Function> cb = Local<Function>::Cast(args[0]);
        const unsigned argc = 2;
  			
        Local<Value> argv[argc] = {
                String::NewFromUtf8(isolate,"hello world").ToLocalChecked(),
                String::NewFromUtf8(isolate,"caonima").ToLocalChecked(),
        };
        cb->Call(context, Null(isolate), argc, argv).ToLocalChecked();
    }
```

`Local<Value> argv[argc] = {}`通过这个可以定义你的回调函数的参数

## 返回一个对象

```c++
void CreateObject(const FunctionCallbackInfo<Value>& args) {
        Isolate* isolate = args.GetIsolate();
        Local<Context> context = isolate->GetCurrentContext();

        Local<Object> obj = Object::New(isolate);
        obj->Set(context,
                 String::NewFromUtf8(isolate,"msg").ToLocalChecked(),
                  args[0]->ToString(context).ToLocalChecked()).FromJust();

        obj->Set(context,
                 String::NewFromUtf8(isolate,"haa").ToLocalChecked(),
                 args[1]->ToString(context).ToLocalChecked()).FromJust();

        args.GetReturnValue().Set(obj);
}
```

`obj->Set`这句话就是可以用来定义对象的键值对

## 返回闭包函数

## 封装对象

```cpp
#include <node.h>

namespace demo {

using v8::Context;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  Local<Object> obj = Object::New(isolate);
  obj->Set(context,
           String::NewFromUtf8(isolate,
                               "msg").ToLocalChecked(),
                               args[0]->ToString(context).ToLocalChecked())
           .FromJust();

  args.GetReturnValue().Set(obj);
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", CreateObject);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

# 插件的应用场景

我们知道nodejs不擅长做一些cpu密集型的工作，因为node是基于单线程的，会产生一个阻塞问题。

那么对于一些cpu密集型的工作我们就可以通过node插件机制实现，因为c++跑得不叫快，同时底层时基于libuv事件循环，有线程池，帮助我们完成这些任务。

例如一些：密码解析，视频解析，图像处理等等这些东西都是一些cpu密集型工作，我们就可以通过node插件完成