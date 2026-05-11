# 项目工程化

> 🏗️ HarmonyOS 项目从零搭建、构建与部署完整指南

## 📖 目录

1. [项目创建](#项目创建)
2. [工程配置文件详解](#工程配置文件详解)
3. [ArkTS 编译与约束](#arkts-编译与约束)
4. [多模块架构](#多模块架构)
5. [代码混淆](#代码混淆)
6. [构建与打包](#构建与打包)
7. [签名与发布](#签名与发布)
8. [调试与性能分析](#调试与性能分析)

---

## 项目创建

### DevEco Studio 方式

1. 打开 DevEco Studio 5.0+
2. `File → New → Create Project`
3. 选择模板：`Empty Ability`
4. 配置参数：
   - Project name: example `MyHarmonyApp`
   - Bundle name: `com.example.myapp`
   - Save location: `<workspace>/MyHarmonyApp`
   - **Compatible SDK**: `5.0.0(12)` (HarmonyOS NEXT)
   - Module name: `entry`

### Agent 手动创建方式

当 Agent 需要创建项目文件时，按以下顺序生成文件：

```
步骤1: 创建目录结构（使用项目结构模板）
步骤2: 写入 build-profile.json5（项目级）
步骤3: 写入 oh-package.json5（项目级）
步骤4: 写入 hvigorfile.ts（项目级）
步骤5: 写入 AppScope/app.json5
步骤6: 写入 entry/build-profile.json5（模块级）
步骤7: 写入 entry/oh-package.json5（模块级）
步骤8: 写入 entry/hvigorfile.ts（模块级）
步骤9: 写入 entry/src/main/module.json5
步骤10: 写入 entry/src/main/resources/base/profile/main_pages.json
步骤11: 写入 entry/src/main/resources/base/element/string.json
步骤12: 写入 entry/src/main/ets/entryability/EntryAbility.ets
步骤13: 写入 entry/src/main/ets/pages/Index.ets（首页）
```

---

## 工程配置文件详解

### 项目级 build-profile.json5

```json5
{
  "app": {
    "signingConfigs": [
      // 调试签名（自动生成）
      // 发布签名需在 DevEco Studio 中配置
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compatibleSdkVersion": 12,    // 目标 API Level
        "targetSdkVersion": 12,
        "runtimeOS": "HarmonyOS",
        "buildOption": {
          "strictMode": {
            "useDefaultStrictMode": true  // API 10+ 强制 ArkTS 严格检查
          },
          "arkOptions": {
            "obfuscation": {
              "ruleOptions": {
                "enable": false,           // 发布时设为 true
                "files": ["./obfuscation-rules.txt"]
              }
            }
          }
        }
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [
        {
          "name": "default",
          "applyToProducts": ["default"]
        }
      ]
    }
    // 多模块时添加更多 module 条目
  ]
}
```

### 模块级 entry/build-profile.json5

```json5
{
  "apiType": "stageMode",
  "buildOption": {
    "sourceOption": {
      "workers": [
        "./src/main/ets/workers/MyWorker.ets"  // 注册 Worker 文件
      ]
    }
  },
  "buildOptionSet": [
    {
      "name": "release",
      "arkOptions": {
        "obfuscation": {
          "ruleOptions": {
            "enable": true,
            "files": ["./obfuscation-rules.txt"]
          }
        }
      }
    }
  ],
  "targets": [
    {
      "name": "default"
    },
    {
      "name": "ohosTest"
    }
  ]
}
```

### module.json5 配置

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": [
      "phone",
      "tablet",
      "2in1"
    ],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "icon": "$media:layered_image",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:startIcon",
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ],
    "requestPermissions": [
      // 在此声明需要的权限
      // { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

### hvigorfile.ts（项目级）

```typescript
import { appTasks } from '@ohos/hvigor-ohos-plugin';

export default {
  system: appTasks,
  plugins: []
}
```

### hvigorfile.ts（模块级）

```typescript
import { hapTasks } from '@ohos/hvigor-ohos-plugin';

export default {
  system: hapTasks,
  plugins: []
}
```

### local.properties

```properties
# 自动生成，指向 SDK 路径
nodejs.dir=C:/Users/<user>/AppData/Local/Huawei/Sdk/hmscore/3.1.0/toolchains/node
hwsdk.dir=C:/Users/<user>/AppData/Local/Huawei/Sdk
```

---

## ArkTS 编译与约束

### 编译流程

```
.ets 源文件
    ↓
arkts-loader (TS → ArkTS 语法检查)
    ↓ 严格模式: compatibleSdkVersion ≥10 → error on violation
    ↓ 兼容模式: compatibleSdkVersion <10 → warning only
    ↓
方舟编译器 (Ark Compiler)
    ↓
abc 字节码 (Ark Bytecode)
    ↓
HAP 包 (HarmonyOS Ability Package)
```

### 关键检查点

| 检查 | 触发条件 | 结果 |
|------|---------|------|
| 使用 `any`/`unknown` | API 10+ 标准模式 | ❌ error |
| `var` 声明 | API 10+ 标准模式 | ❌ error |
| `for...in` | API 10+ 标准模式 | ❌ error |
| 解构赋值 | API 10+ 标准模式 | ❌ error |
| 函数重载 | API 10+ 标准模式 | ❌ error |
| `require()` | API 10+ 标准模式 | ❌ error |
| `eval()` / `with()` | API 11+ 标准模式 | ❌ error |
| 循环依赖 | 任何模式 | ❌ error |

---

## 多模块架构

### HAP + HSP 模式

```
project/
├── entry/               # HAP - 主入口模块
├── feature_login/       # HAP - 登录功能模块
├── feature_home/        # HAP - 首页功能模块
├── library_common/      # HSP - 公共库（动态共享包）
└── library_network/     # HSP - 网络库（动态共享包）
```

### HSP 模块配置示例

```json5
// library_common/build-profile.json5
{
  "apiType": "stageMode",
  "buildOption": {
    "arkOptions": {
      "runtimeOnly": {
        "packIntoApp": true     // 打包到 App 中分发
        // "sources": {           // 或部署到企业服务器
        //   "deliveryWithInstall": false
        // }
      }
    }
  }
}
```

### 模块间依赖 (oh-package.json5)

```json5
// entry/oh-package.json5
{
  "dependencies": {
    "@ohos/library_common": "file:../library_common",
    "@ohos/library_network": "file:../library_network"
  }
}
```

---

## 代码混淆

### obfuscation-rules.txt

```text
# 以下是混淆保留规则示例

# 保留 EntryAbility 不被混淆
-keep-file-name
entryability/EntryAbility

# 保留所有接口实现
-keep-property-name
globalThis

# 保留特定模块不混淆
-enable-property-obfuscation
xxx

# 保留文件名（重要：防止 Worker 找不到文件）
-enable-filename-obfuscation
true

# 保留属性名（重要：防止 JSON 序列化失败）
-enable-property-obfuscation
false
```

---

## 构建与打包

### 构建命令

```bash
# Debug 构建
hvigorw assembleHap --mode module -p product=default -p buildMode=debug

# Release 构建（含混淆）
hvigorw assembleHap --mode module -p product=default -p buildMode=release

# 全量构建
hvigorw --mode project -p product=default assembleApp
```

### 产物位置

```
entry/build/default/outputs/default/
├── entry-default-unsigned.hap   # 未签名 HAP
├── entry-default-signed.hap     # 已签名 HAP（调试签名）
└── entry-default-unsigned.app   # 未签名 APP 包
```

### HAP vs APP 包

| 包类型 | 内容 | 用途 |
|--------|------|------|
| `.hap` | 单个模块 | 开发调试、模块分发 |
| `.app` | 所有模块打包 | 应用商店上架 |

---

## 签名与发布

### 签名配置层级

```
调试签名: DevEco Studio 自动生成（开发用）
    ↓
发布签名: 华为 AppGallery Connect 申请（上架用）
    ├── .p12 密钥库文件
    ├── .cer 证书文件
    └── .p7b  Profile 文件
```

### 发布前检查清单

- [ ] 更新 `versionCode`（整数，递增）
- [ ] 更新 `versionName`（语义版本，如 `2.0.0`）
- [ ] 启用代码混淆（`obfuscation.enable = true`）
- [ ] 关闭调试日志
- [ ] 检查权限只声明实际使用的
- [ ] 移除未使用的资源和依赖
- [ ] 测试所有 deviceTypes 上的表现
- [ ] 在真机上完整回归测试

---

## 调试与性能分析

### HiLog 日志

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit';

const DOMAIN: number = 0x0000;
const TAG: string = 'MyTag';

hilog.debug(DOMAIN, TAG, 'Debug: %{public}d', 42);
hilog.info(DOMAIN, TAG, 'Info: %{public}s', 'hello');
hilog.warn(DOMAIN, TAG, 'Warning');
hilog.error(DOMAIN, TAG, 'Error: %{public}s', errMsg);
hilog.fatal(DOMAIN, TAG, 'Fatal error');
```

### 性能调试

```bash
# SmartPerf 性能分析工具
hdc shell aa start -b com.example.myapp -a EntryAbility
hdc file recv /data/app/el2/100/base/com.example.myapp/haps/entry/cache/profiler_data.json .
```

### 常见编译错误速查

| 错误信息 | 原因 | 修复 |
|---------|------|------|
| `Cannot find name 'any'` | 使用了 `any` 类型 | 改为具体类型 |
| `'var' is not allowed` | 使用了 `var` | 改为 `let`/`const` |
| `for...in is not supported` | 使用了 `for...in` | 改为 `Object.keys()` + for |
| `Destructuring is not supported` | 使用了解构 | 改为直接属性访问 |
| `Function overload is not supported` | 定义了函数重载 | 合并为单一签名 |
| `Cannot find module '@ohos.xxx'` | 使用了旧 API 路径 | 改为 `@kit.XxxKit` |
| `Circular dependency detected` | 模块间循环引用 | 提取公共接口打破循环 |
| `eval() is not allowed in strict mode` | 使用了 `eval()` | 用逻辑替代 |
| `Constructor field declaration` | constructor 中声明字段 | 类体顶层声明 |

---

## 📚 延伸阅读

- [DevEco Studio 使用指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-overview)
- [Hvigor 构建系统](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hvigor-overview)
- [多HAP构建](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/multi-hap-build)
- [应用/服务签名](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/signing-overview)
- [ArkGuard 代码混淆](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/source-obfuscation)

---

> 📝 **最后更新**: 2026-05-10
> 📌 **适用版本**: HarmonyOS NEXT (API 12)