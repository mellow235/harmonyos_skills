# 质量检查清单

> ✅ Agent 代码交付前的自检标准

## 📖 目录

1. [ArkTS 语法合规](#arkts-语法合规)
2. [导入与依赖](#导入与依赖)
3. [组件规范](#组件规范)
4. [状态管理规范](#状态管理规范)
5. [工程配置](#工程配置)
6. [性能规范](#性能规范)
7. [安全检查](#安全检查)
8. [交付清单](#交付清单)

---

## ArkTS 语法合规

> **检查时机**：每写完一个 `.ets` 文件后立即执行。

| # | 检查项 | 检查方法 | ❌ 错误示例 | ✅ 正确示例 |
|---|--------|---------|------------|-----------|
| 1 | 无 `any`/`unknown` | 搜索 `: any` `: unknown` | `let x: any = 1` | `let x: number = 1` |
| 2 | 无 `var` | 搜索 `var ` | `var x = 1` | `let x = 1` |
| 3 | 无 `for...in` | 搜索 `for (... in` | `for (let k in obj)` | `Object.keys(obj)` |
| 4 | 无解构赋值 | 搜索 `const {` `const [` | `const {a} = obj` | `const a = obj.a` |
| 5 | 无 `Symbol()` | 搜索 `Symbol(` | `Symbol('key')` | `'key'` |
| 6 | 无 `#` 私有字段 | 搜索 `#` | `#privateField` | `private fieldName` |
| 7 | 无函数重载 | 搜索多个同名 function 签名 | `function fn(x: string)` + `function fn(x: number)` | `function fn(x: string \| number)` |
| 8 | 无 `require()` | 搜索 `require(` | `const lib = require('')` | `import { lib } from ''` |
| 9 | 无 `delete` | 搜索 `delete ` | `delete obj.prop` | `obj.prop = undefined` |
| 10 | 无 `eval()` / `with()` | 搜索 `eval(` `with (` | `eval(code)` | 用逻辑替代 |
| 11 | 函数参数有类型注解 | 检查函数签名 | `function fn(x) {}` | `function fn(x: number): void {}` |
| 12 | 无 `as const` | 搜索 `as const` | `obj as const` | 使用字面量类型 |
| 13 | 无 `globalThis` | 搜索 `globalThis` | `globalThis.xxx` | 避免全局对象 |

---

## 导入与依赖

| # | 检查项 | 正确写法 |
|---|--------|---------|
| 1 | 日志用 `@kit.PerformanceAnalysisKit` | `import { hilog } from '@kit.PerformanceAnalysisKit'` |
| 2 | 路由/窗口用 `@kit.ArkUI` | `import { router, window } from '@kit.ArkUI'` |
| 3 | 网络用 `@kit.NetworkKit` | `import { http } from '@kit.NetworkKit'` |
| 4 | 数据存储用 `@kit.ArkData` | `import { preferences, relationalStore } from '@kit.ArkData'` |
| 5 | 应用框架用 `@kit.AbilityKit` | `import { UIAbility, Want } from '@kit.AbilityKit'` |
| 6 | 并发用 `@kit.ArkTS` | `import { taskpool, worker } from '@kit.ArkTS'` |
| 7 | 通知用 `@kit.NotificationKit` | `import { notificationManager } from '@kit.NotificationKit'` |
| 8 | 剪贴板用 `@kit.BasicServicesKit` | `import { pasteboard } from '@kit.BasicServicesKit'` |
| 9 | 禁止 `@ohos.*` 旧路径 | ❌ `@ohos.hilog` → ✅ `@kit.PerformanceAnalysisKit` |

---

## 组件规范

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | UI 组件用 `struct` 定义 | 不用 `class` |
| 2 | 页面组件有 `@Entry` | 每个页面入口必须有 |
| 3 | 自定义组件有 `@Component` | 每个复用组件必须有 |
| 4 | 组件有 `build()` 方法 | 必须有返回值 |
| 5 | `@Builder` 方法不访问 `this`（或使用 `@BuilderParam`） | 按值传递 |
| 6 | 页面已注册到 `main_pages.json` | 每个 `@Entry` 页面必须注册 |
| 7 | `ForEach` 第3参数（keyGenerator）存在 | `(item: T) => item.id` |
| 8 | 条件渲染用 `if/else` | 不用三元表达式做组件级渲染 |
| 9 | 列表大数据用 `LazyForEach` | 替代 `ForEach` (超过1000条) |

---

## 状态管理规范

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | `@State` 仅用于组件内部状态 | 不可跨组件传递 |
| 2 | `@Prop` 不能直接修改 | 单向传递，子组件只读 |
| 3 | `@Link` 需要父组件传 `$` 前缀 | `$this.count` |
| 4 | `@Provide/@Consume` key 一致 | 提供方和消费方 key 字符串必须匹配 |
| 5 | `@Watch` 回调方法存在 | 回调方法名不能拼写错误 |
| 6 | `@StorageLink` key 已初始化 | 确保 `AppStorage` 中有初始值 |
| 7 | 避免在 `build()` 中修改状态 | 会导致无限循环重渲染 |
| 8 | `@Prop` 不传函数引用 | `@Prop onClick: () => void` 应避免，改用事件回调 |

---

## 工程配置

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | `compatibleSdkVersion >= 10` | 标准模式下 ArkTS 语法错误会导致编译失败 |
| 2 | `runtimeOS: "HarmonyOS"` | build-profile.json5 中必须有 |
| 3 | `mainElement` 指向正确的 Ability | module.json5 中 `"mainElement": "EntryAbility"` |
| 4 | `pages` 引用 `main_pages` | `"pages": "$profile:main_pages"` |
| 5 | 资源文件引用正确 | `$r('app.media.icon')` / `$r('app.string.app_name')` |
| 6 | 权限在 module.json5 中声明 | `requestPermissions` 数组 |
| 7 | 多模块依赖在 oh-package.json5 | dependencies 字段 |

---

## 性能规范

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | 优先 `const` | 可被编译器常量折叠优化 |
| 2 | 大量数值数据用 `TypedArray` | `Float32Array` / `Int32Array` |
| 3 | 稀疏键值对用 `Map` | 非 `Array` 留空位 |
| 4 | 提取循环不变量 | `const len = arr.length` 放在 for 外 |
| 5 | 查找用 `Set.has()` / `Map.has()` | O(1) 替代 O(n) `Array.includes()` |
| 6 | 数组字面量初始化 | `[1,2]` 而非 `new Array(); push()` |
| 7 | 避免在 `build()` 中做计算 | 放到 `aboutToAppear()` 或 computed getter |
| 8 | 列表缓动用 `LazyForEach` | 长列表性能关键 |

---

## 安全检查

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | 网络请求用 HTTPS | 生产环境不传输明文 |
| 2 | 敏感数据加密存储 | 使用 `securityLevel: S3+` |
| 3 | 日志不含敏感信息 | 密码/token 不出现在日志 |
| 4 | 用户输入验证 | 防注入、防溢出 |
| 5 | 最小权限原则 | 只声明实际使用的权限 |
| 6 | 混淆开启（发布版） | `obfuscation.enable: true` |

---

## 交付清单

### 新建项目时

- [ ] 项目目录结构完整（见 SKILL.md 项目结构模板）
- [ ] `build-profile.json5`（项目级）配置正确
- [ ] `oh-package.json5`（项目级）
- [ ] `AppScope/app.json5`
- [ ] `entry/build-profile.json5`（模块级）
- [ ] `entry/oh-package.json5`
- [ ] `entry/src/main/module.json5`
- [ ] `entry/src/main/resources/base/profile/main_pages.json`
- [ ] `entry/src/main/resources/base/element/string.json`
- [ ] `entry/src/main/ets/entryability/EntryAbility.ets`
- [ ] 至少一个页面（`pages/Index.ets`）
- [ ] 所有 `.ets` 文件通过 ArkTS 语法合规检查

### 新增页面时

- [ ] 页面文件放在 `pages/` 目录
- [ ] `@Entry` + `@Component` 装饰器
- [ ] 注册到 `main_pages.json`
- [ ] 通过语法合规检查

### 新增组件时

- [ ] 组件放在 `components/` 目录
- [ ] `@Component` + `export struct`
- [ ] `@Prop` 有默认值
- [ ] 通过语法合规检查

### 新增功能模块时

- [ ] Model 定义在 `models/`
- [ ] Service 在 `services/`（如有数据层）
- [ ] ViewModel 在 `viewmodels/`（如有状态管理）
- [ ] Page 在 `pages/`
- [ ] 页面注册到 `main_pages.json`
- [ ] 权限声明（如需要网络/存储）

---

> 📝 **最后更新**: 2026-05-10
> 📌 **适用版本**: HarmonyOS NEXT (API 12)