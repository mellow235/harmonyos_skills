# 跨语言交互 (Node-API)

> 🔗 ArkTS 与 C/C++ Native 代码的互操作

## 📖 目录

1. [Node-API 概述](#node-api-概述)
2. [环境搭建](#环境搭建)
3. [基础数据类型映射](#基础数据类型映射)
4. [创建 Native 模块](#创建-native-模块)
5. [ArkTS 调用 Native](#arkts-调用-native)
6. [Native Sendable 对象](#native-sendable-对象)
7. [性能优化](#性能优化)
8. [常见问题](#常见问题)

---

## Node-API 概述

Node-API 是 ArkTS 与 C/C++ 进行跨语言交互的标准接口，基于 Node.js N-API 扩展实现。

### 适用场景

| 场景 | 说明 |
|------|------|
| 性能敏感计算 | 图像处理、音视频编解码、加密算法 |
| 复用现有库 | 调用 C/C++ 开源库（OpenCV、FFmpeg 等） |
| 硬件访问 | 直接操作设备硬件接口 |
| 游戏开发 | 游戏引擎集成、物理模拟 |

### 架构图

```
ArkTS 层
    │
    ▼
Node-API 接口层
    │
    ▼
Native 层 (C/C++)
    │
    ▼
系统底层 / 硬件
```

---

## 环境搭建

### 1. 创建 Native 工程

在 DevEco Studio 中：
1. 新建项目时勾选 **"Include Native Code"**
2. 或现有项目右键 → **New → Native C++ Module**

### 2. 工程结构

```
entry/
├── src/
│   ├── main/
│   │   ├── ets/              # ArkTS 代码
│   │   │   └── pages/
│   │   └── cpp/              # C++ 代码
│   │       ├── types/        # 类型声明
│   │       │   └── libentry/
│   │       │       ├── index.d.ts
│   │       │       └── oh-package.json5
│   │       ├── CMakeLists.txt
│   │       └── napi_init.cpp # Native 入口
│   └── ohosTest/
└── oh-package.json5
```

### 3. CMake 配置

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.4.1)
project(MyNativeModule)

set(NATIVERENDER_ROOT_PATH ${CMAKE_CURRENT_SOURCE_DIR})

include_directories(${NATIVERENDER_ROOT_PATH}
                    ${NATIVERENDER_ROOT_PATH}/include)

add_library(entry SHARED napi_init.cpp)

target_link_libraries(entry PUBLIC libace_napi.z.so)
```

---

## 基础数据类型映射

| ArkTS 类型 | Native 类型 | 说明 |
|-----------|------------|------|
| `boolean` | `napi_boolean` | 布尔值 |
| `number` | `napi_number` | 数值 |
| `string` | `napi_string` | 字符串 |
| `object` | `napi_object` | 对象 |
| `Array` | `napi_array` | 数组 |
| `ArrayBuffer` | `napi_arraybuffer` | 二进制数据 |
| `undefined` | `napi_undefined` | 未定义 |
| `null` | `napi_null` | 空值 |

---

## 创建 Native 模块

### 1. 注册 Native 方法

```cpp
// napi_init.cpp
#include "napi/native_api.h"
#include "hilog/log.h"

#define LOG_TAG "NativeModule"
#define LOGI(...) OH_LOG_INFO(LOG_APP, LOG_TAG, __VA_ARGS__)

// Native 方法：两数相加
static napi_value Add(napi_env env, napi_callback_info info) {
    size_t argc = 2;
    napi_value args[2] = {nullptr};
    napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);

    double value1, value2;
    napi_get_value_double(env, args[0], &value1);
    napi_get_value_double(env, args[1], &value2);

    napi_value sum;
    napi_create_double(env, value1 + value2, &sum);

    return sum;
}

// Native 方法：字符串处理
static napi_value ProcessString(napi_env env, napi_callback_info info) {
    size_t argc = 1;
    napi_value args[1] = {nullptr};
    napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);

    char buffer[256];
    size_t length;
    napi_get_value_string_utf8(env, args[0], buffer, sizeof(buffer), &length);

    // 处理字符串（转大写）
    for (size_t i = 0; i < length; i++) {
        if (buffer[i] >= 'a' && buffer[i] <= 'z') {
            buffer[i] -= 32;
        }
    }

    napi_value result;
    napi_create_string_utf8(env, buffer, length, &result);
    return result;
}

// 模块注册
static napi_value Init(napi_env env, napi_value exports) {
    napi_property_descriptor desc[] = {
        { "add", nullptr, Add, nullptr, nullptr, nullptr, napi_default, nullptr },
        { "processString", nullptr, ProcessString, nullptr, nullptr, nullptr, napi_default, nullptr }
    };
    napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc);
    return exports;
}

EXTERN_C_START
static napi_value InitModule(napi_env env, napi_value exports) {
    return Init(env, exports);
}
EXTERN_C_END

static napi_module demoModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = InitModule,
    .nm_modname = "entry",
    .nm_priv = nullptr,
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void RegisterEntryModule() {
    napi_module_register(&demoModule);
}
```

### 2. 类型声明文件

```typescript
// cpp/types/libentry/index.d.ts
export const add: (a: number, b: number) => number;
export const processString: (str: string) => string;
```

### 3. 模块配置

```json5
// cpp/types/libentry/oh-package.json5
{
  "name": "libentry.so",
  "types": "./index.d.ts",
  "version": "1.0.0",
  "description": "Native API"
}
```

---

## ArkTS 调用 Native

### 1. 导入 Native 模块

```typescript
// pages/Index.ets
import { add, processString } from 'libentry.so';

@Entry
@Component
struct NativeDemo {
  @State result: string = '';

  build() {
    Column() {
      Button('Native Add')
        .onClick(() => {
          const sum = add(10, 20);
          this.result = `10 + 20 = ${sum}`;
        })

      Button('Process String')
        .onClick(() => {
          const result = processString('hello');
          this.result = result;  // "HELLO"
        })

      Text(this.result)
        .fontSize(20)
    }
  }
}
```

### 2. 处理 ArrayBuffer

```cpp
// Native 图像处理示例
static napi_value ProcessImage(napi_env env, napi_callback_info info) {
    size_t argc = 2;
    napi_value args[2] = {nullptr};
    napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);

    // 获取 ArrayBuffer
    napi_value arrayBuffer = args[0];
    uint8_t* data;
    size_t length;
    napi_get_arraybuffer_info(env, arrayBuffer, (void**)&data, &length);

    // 获取宽度、高度参数
    int32_t width, height;
    napi_get_value_int32(env, args[1], &width);
    // ... 获取 height

    // 图像处理（灰度化）
    for (size_t i = 0; i < length; i += 4) {
        uint8_t gray = (data[i] * 299 + data[i + 1] * 587 + data[i + 2] * 114) / 1000;
        data[i] = gray;      // R
        data[i + 1] = gray;  // G
        data[i + 2] = gray;  // B
        // data[i + 3] = alpha（不变）
    }

    return arrayBuffer;
}
```

```typescript
// ArkTS 调用
import { processImage } from 'libentry.so';

async function convertToGray(imageBuffer: ArrayBuffer, width: number): Promise<ArrayBuffer> {
    return processImage(imageBuffer, width);
}
```

---

## Native Sendable 对象

### 创建 Native Sendable 类

```cpp
#include "napi/native_api.h"
#include "napi_sendable.h"

class NativeData : public NapiSendable {
public:
    int32_t value;
    std::string name;

    NativeData(int32_t v, const std::string& n) : value(v), name(n) {}

    // 序列化
    void Serialize(napi_env env, napi_value obj) override {
        napi_value valueProp;
        napi_create_int32(env, value, &valueProp);
        napi_set_named_property(env, obj, "value", valueProp);

        napi_value nameProp;
        napi_create_string_utf8(env, name.c_str(), name.length(), &nameProp);
        napi_set_named_property(env, obj, "name", nameProp);
    }

    // 反序列化
    void Deserialize(napi_env env, napi_value obj) override {
        napi_value valueProp;
        napi_get_named_property(env, obj, "value", &valueProp);
        napi_get_value_int32(env, valueProp, &value);

        napi_value nameProp;
        napi_get_named_property(env, obj, "name", &nameProp);
        char buffer[256];
        size_t length;
        napi_get_value_string_utf8(env, nameProp, buffer, sizeof(buffer), &length);
        name = std::string(buffer, length);
    }
};

// 创建 Sendable 对象
static napi_value CreateSendable(napi_env env, napi_callback_info info) {
    auto data = std::make_shared<NativeData>(42, "NativeData");
    napi_value result;
    napi_create_sendable_object(env, data.get(), &result);
    return result;
}
```

```typescript
// ArkTS 使用 Native Sendable
import { createSendable } from 'libentry.so';

@Sendable
class SharedData {
  nativeRef: object;  // Native Sendable 对象

  constructor() {
    this.nativeRef = createSendable();
  }
}

// 可在多线程间传递
@Concurrent
function processNativeData(data: SharedData): void {
  console.info(`Native value: ${data.nativeRef.value}`);
}
```

---

## 性能优化

### 1. 减少跨语言调用次数

```cpp
// ❌ 频繁调用
static napi_value GetPixel(napi_env env, napi_callback_info info) {
    // 每次获取一个像素，效率低
}

// ✅ 批量处理
static napi_value ProcessPixels(napi_env env, napi_callback_info info) {
    // 一次处理整个数组
}
```

### 2. 使用 TypedArray

```cpp
// 直接操作 TypedArray，避免拷贝
static napi_value FastProcess(napi_env env, napi_callback_info info) {
    napi_value typedArray;
    // ... 获取参数

    void* data;
    size_t length;
    napi_typedarray_type type;
    napi_value arrayBuffer;
    size_t byteOffset;

    napi_get_typedarray_info(env, typedArray, &type, &length, &data, &arrayBuffer, &byteOffset);

    // 直接修改内存
    float* floatData = static_cast<float*>(data);
    for (size_t i = 0; i < length; i++) {
        floatData[i] *= 2.0f;
    }

    return typedArray;
}
```

### 3. 异步 Native 操作

```cpp
#include <thread>
#include <future>

// 异步执行耗时操作
static napi_value AsyncProcess(napi_env env, napi_callback_info info) {
    napi_value promise;
    napi_deferred deferred;
    napi_create_promise(env, &deferred, &promise);

    // 在后台线程执行
    std::thread([env, deferred]() {
        // 耗时操作...
        
        napi_value result;
        napi_create_int32(env, 42, &result);
        napi_resolve_deferred(env, deferred, result);
    }).detach();

    return promise;
}
```

---

## 常见问题

### Q1: Native 模块编译失败？

**A:** 
1. 检查 CMakeLists.txt 配置
2. 确认 NDK 版本兼容
3. 检查 C++ 代码语法

### Q2: ArkTS 导入 Native 模块报错？

**A:**
1. 确认 `oh-package.json5` 中已声明依赖
2. 检查类型声明文件路径
3. 重新编译项目

### Q3: Native 代码崩溃？

**A:**
1. 检查参数类型匹配
2. 确认数组越界访问
3. 使用 `hilog` 添加日志定位

### Q4: 如何调试 Native 代码？

**A:**
1. 使用 DevEco Studio 的 C++ 调试器
2. 添加 `hilog` 日志输出
3. 使用 `lldb` 命令行调试

---

## 📚 延伸阅读

- [Node-API 简介](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/napi-introduction)
- [Node-API 数据类型](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/napi-data-types-interfaces)
- [Native Sendable 对象](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/napi-define-sendable-object)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
