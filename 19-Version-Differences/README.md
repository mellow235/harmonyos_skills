# ArkTS 版本差异对照

> 🔄 ArkTS 各版本特性演进与迁移指南

## 📖 目录

1. [版本演进总览](#版本演进总览)
2. [API 9 / ArkTS 3.0](#api-9--arkts-30)
3. [API 10 / ArkTS 4.0](#api-10--arkts-40)
4. [API 11 / ArkTS 5.0](#api-11--arkts-50)
5. [API 12 / ArkTS 6.0](#api-12--arkts-60)
6. [版本间的关键语法差异](#版本间的关键语法差异)
7. [.ets 文件兼容性模式](#ets-文件兼容性模式)
8. [迁移检查清单](#迁移检查清单)

---

## 版本演进总览

```
HarmonyOS 3.x          HarmonyOS 4.x          HarmonyOS NEXT
├── API 9               ├── API 10              ├── API 11           ├── API 12
├── ArkTS 3.0           ├── ArkTS 4.0           ├── ArkTS 5.0        ├── ArkTS 6.0
│                        │                       │                    │
├── TS 完全兼容          ├── 强制静态检查          ├── @Sendable类       ├── @Sendable函数
├── .ets = TS语法        ├── compatibleSdkVersion ├── TS 4.9.5         ├── LongTask
├── 无ArkTS严格模式      ├── 标准模式(>=10)       ├── 共享容器增强       ├── Sendable LruCache
│   (兼容模式)           │   编译失败             ├── AsyncLock         ├── V1→V2迁移
│                        │   warning模式(<10)     │   ConditionVariable  │   状态管理重构
│                        │   编译成功              │                    ├── 更多系统Sendable
│                        │                        │                    │   对象接入
```

### 各版本定位

| 版本 | API Level | 定位 | 关键变化 |
|------|-----------|------|----------|
| ArkTS 3.0 | API 9 | 兼容过渡期 | TS语法完全兼容，无严格ArkTS检查 |
| ArkTS 4.0 | API 10 | 语法规范化 | 强制静态类型检查，禁止`any`/`unknown` |
| ArkTS 5.0 | API 11 | 并发能力增强 | @Sendable类、共享容器、AsyncLock |
| ArkTS 6.0 | API 12 | 生产级成熟 | @Sendable函数、LongTask、状态管理V2 |

---

## API 9 / ArkTS 3.0

### 语法特征

```typescript
// API 9 中以下代码均合法（等同于标准TS）：
let value: any = 'hello';  // ✅ any 允许
value = 123;

let obj: Record<string, number> = {};
obj.newProp = 42;  // ✅ 运行时添加属性允许

for (let key in obj) {  // ✅ for...in 允许
  console.info(key);
}

const { name, age } = user;  // ✅ 解构赋值允许

var oldVar = 'old';  // ✅ var 允许（但不推荐）
```

### 识别标志

```json5
// build-profile.json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compatibleSdkVersion": 9,  // API 9
        "runtimeOS": "HarmonyOS"
      }
    ]
  }
}
```

### 注意事项

- `.ets` 文件完全遵循标准 TypeScript 规范
- **无 ArkTS 编译时语法检查**
- 可以无痛使用任何 TS 特性
- 编译产物仍为方舟字节码，但运行时需处理动态类型

---

## API 10 / ArkTS 4.0

### 关键变化：引入强制静态检查

从 API 10 开始，ArkTS 明确定义了**自己的语法规则**，与标准 TS 有了明确区分。

```typescript
// API 10 中以下代码编译失败：
let value: any = 'hello';  // ❌ any 禁止
value = 123;

let obj: Record<string, number> = {};
(p as any).z = 10;  // ❌ 运行时变更对象布局禁止

for (let key in obj) {  // ❌ for...in 禁止
  console.info(key);
}

const { name, age } = user;  // ❌ 解构赋值禁止

var oldVar = 'old';  // ❌ var 禁止
```

### 新增约束速查

| 特性 | API 9 | API 10+ | API 10 替代方案 |
|------|-------|---------|----------------|
| `any`/`unknown` | ✅ | ❌ | 使用具体类型或 `Object` |
| `var` | ✅ | ❌ | `let`/`const` |
| `for...in` | ✅ | ❌ | `for...of` 或 `Object.keys()` |
| 解构赋值 | ✅ | ❌ | 直接访问属性 |
| `#` 私有字段 | ✅ | ❌ | `private` 关键字 |
| `Symbol()` | ✅ | ❌ | 字符串常量 |
| `structural typing` | ✅ | ❌ | 继承或接口实现 |
| `as const` | ✅ | ❌ | 使用具体类型字面量 |
| `globalThis` | ✅ | ❌ | 避免全局对象 |

### .ets 文件编译模式

```typescript
// compatibleSdkVersion >= 10 → "标准模式"
// → 所有违反 ArkTS 语法的代码编译不通过

// compatibleSdkVersion < 10 → "兼容模式"
// → 违反 ArkTS 语法的代码以 warning 形式提示
// → 仍可编译成功，但需完全适配后才能升级
```

---

## API 11 / ArkTS 5.0

### 关键变化：并发能力增强

#### @Sendable 装饰器（类）

```typescript
// API 11 新增
@Sendable
class SharedData {
  count: number = 0;
  name: string = '';

  increment(): void {
    this.count++;
  }
}

// 可在 TaskPool/Worker 间引用传递
@Concurrent
function processData(data: SharedData): void {
  data.increment();  // 直接修改共享对象
}
```

#### 共享容器

```typescript
import { collections } from '@kit.ArkTS';

// API 11 新增
const sharedArray = collections.Array.create<number>(10, 0);
const sharedMap = new collections.Map<string, number>();
const sharedSet = new collections.Set<string>();
```

#### 异步锁

```typescript
import { ArkTSUtils } from '@kit.ArkTS';

const lock = new ArkTSUtils.locks.AsyncLock();

await lock.lockAsync(() => {
  // 临界区代码，线程安全
  sharedArray[0]++;
});
```

### 新增特性清单

| 特性 | 说明 |
|------|------|
| `@Sendable` 类 | 并发实例间引用传递 |
| `collections.Array/Map/Set` | 共享容器集 |
| `ArkTSUtils.locks.AsyncLock` | 异步锁 |
| `ArkTSUtils.locks.ConditionVariable` | 条件变量 |
| TS 版本升级 | 4.9.5，target es2017 |
| 强制严格模式 | `use strict` 强制开启 |
| 禁止 `eval()` | 动态执行代码 |
| 禁止 `with()` | 作用域污染 |
| 禁止循环依赖 | 模块加载失败 |

---

## API 12 / ArkTS 6.0

### 关键变化：生产级成熟

#### @Sendable 装饰器（函数）

```typescript
// API 12 新增
@Sendable
function topLevelHandler(): void {
  console.info('Sendable function');
}

@Sendable
class MyClass {
  callback: () => void = topLevelHandler;  // 可赋值给 Sendable 类型
}
```

#### LongTask

```typescript
import { taskpool } from '@kit.ArkTS';

@Concurrent
function longRunningTask(): void {
  // 可执行超过 3 分钟的任务
}

const task = new taskpool.Task(longRunningTask);
const longTask = new taskpool.LongTask(task);  // API 12 新增
await taskpool.execute(longTask);
```

#### Sendable 系统对象接入

```typescript
// API 12 新增 Sendable 封装
import { SendablePreferences } from '@kit.ArkData';
import { SendableColorSpaceManager } from '@kit.GraphicKit';
import { SendableImage } from '@kit.ImageKit';
import { SendableResourceManager } from '@kit.LocalizationKit';
import { SendableContextManager } from '@kit.AbilityKit';
```

#### 状态管理 V2（V1→V2 迁移）

```typescript
// V1 状态管理（API 9-11）
@Component
struct OldStyle {
  @State count: number = 0;
  @Prop title: string = '';
  @Link value: number = 0;
  build() { /* ... */ }
}

// V2 状态管理（API 12+）
@ComponentV2
struct NewStyle {
  @Local count: number = 0;
  @Param title: string = '';
  @Param @Once value: number = 0;
  build() { /* ... */ }
}
```

### API 12 完整新增特性

| 类别 | 特性 | 说明 |
|------|------|------|
| 并发 | `@Sendable` 函数 | 函数级 Sendable |
| 并发 | `LongTask` | 超 3 分钟任务 |
| 并发 | `SendableLruCache` | 共享 LRU 缓存 |
| 系统 | SendablePreferences | 共享用户首选项 |
| 系统 | SendableImage | 共享图片处理 |
| 系统 | SendableContext | 共享上下文管理 |
| UI | `@ComponentV2` | 新版组件装饰器 |
| UI | `@Local` | 替代 @State |
| UI | `@Param` / `@Once` | 替代 @Prop/@Link |

---

## 版本间的关键语法差异

### 1. 变量声明

| 写法 | API 9 | API 10+ |
|------|-------|---------|
| `let x: any = 1` | ✅ | ❌ |
| `let x: number = 1` | ✅ | ✅ |
| `let x = 1` (类型推断) | ✅ | ✅ |
| `var x = 1` | ✅ | ❌ |

### 2. 对象操作

| 写法 | API 9 | API 10+ |
|------|-------|---------|
| `obj.newProp = val` | ✅ | ❌ |
| `delete obj.prop` | ✅ | ❌ |
| `(obj as any).prop` | ✅ | ❌ |
| `obj.prop = val` (已知类型) | ✅ | ✅ |

### 3. 函数

| 写法 | API 9 | API 10+ |
|------|-------|---------|
| 函数重载 | ✅ | ❌ |
| 函数内声明函数 | ✅ | ❌ |
| `function` 作为对象 | ✅ | ❌ |
| 生成器函数 `function*` | ✅ | ❌ |

### 4. 并发

| 能力 | API 9-10 | API 11 | API 12 |
|------|----------|--------|--------|
| TaskPool | ✅ | ✅ | ✅ |
| Worker | ✅ | ✅ | ✅ |
| @Sendable 类 | ❌ | ✅ | ✅ |
| @Sendable 函数 | ❌ | ❌ | ✅ |
| 共享容器 | ❌ | ✅ | ✅ |
| AsyncLock | ❌ | ✅ | ✅ |
| LongTask | ❌ | ❌ | ✅ |
| Sendable 系统对象 | ❌ | ❌ | ✅ |

### 5. 类

| 写法 | API 9 | API 10+ |
|------|-------|---------|
| `constructor` 中声明字段 | ✅ | ❌ |
| `#private` | ✅ | ❌ |
| `private` 关键字 | ✅ | ✅ |
| 类表达式 | ✅ | ❌ |
| `implements` 子句 | ✅ | ❌ |

### 6. 类型系统

| 写法 | API 9 | API 10+ |
|------|-------|---------|
| `any` / `unknown` | ✅ | ❌ |
| `index signature` | ✅ | ❌ |
| `intersection type` (`&`) | ✅ | ❌ |
| `this` type | ✅ | ❌ |
| `conditional type` | ✅ | ❌ |
| `mapped type` | ✅ | ❌ |

### 7. 模块

| 写法 | API 9 | API 10+ |
|------|-------|---------|
| `require()` | ✅ | ❌ |
| `export = ...` | ✅ | ❌ |
| `ambient module` | ✅ | ❌ |
| `import = require()` | ✅ | ❌ |

---

## .ets 文件兼容性模式

### 模式决策逻辑

```typescript
// compatibleSdkVersion 决定了 ArkTS 语法检查模式

if (compatibleSdkVersion < 10) {
  // 兼容模式 (Compatibility Mode)
  // - 违反 ArkTS 语法的代码产生 warning
  // - 编译可以成功
  // - 具体策略：
  //   - warning: 提示但不阻断编译
  //   - 建议：尽快适配，因为升级后会编译失败
} else {
  // 标准模式 (Standard Mode)  
  // - 违反 ArkTS 语法的代码产生 error
  // - 编译一定失败
  // - 必须修正所有问题才能通过编译
}
```

### 工程配置示例

```json5
// API 10+ 标准模式
{
  "app": {
    "products": [
      {
        "compatibleSdkVersion": 12,  // ≥10 启用标准模式
        "runtimeOS": "HarmonyOS"
      }
    ]
  }
}
```

---

## 迁移检查清单

### API 9 → API 10 迁移

- [ ] 替换所有 `any`/`unknown` 为具体类型
- [ ] 替换 `var` 为 `let`/`const`
- [ ] 移除 `for...in` 循环，改用 `for...of` 或 `Object.keys()`
- [ ] 移除解构赋值（`const {a} = obj`）
- [ ] 替换 `#private` 为 `private` 关键字
- [ ] 移除 `Symbol()` 调用
- [ ] 移除函数重载
- [ ] 修复对象运行时布局变更
- [ ] 移除 `delete` 运算符使用
- [ ] 确保类属性显式初始化
- [ ] 设置 `compatibleSdkVersion >= 10`

### API 10 → API 11 迁移

- [ ] 识别需要跨线程共享的类，添加 `@Sendable`
- [ ] 大数据量跨线程传递改用 `collections.Array/Map/Set`
- [ ] 多线程共享数据加 `AsyncLock` 保护
- [ ] 检查并移除循环依赖
- [ ] 检查并移除 `eval()`、`with()` 使用
- [ ] TS 语法适配 4.9.5 标准

### API 11 → API 12 迁移

- [ ] 识别需要跨线程共享的函数，添加 `@Sendable`
- [ ] 超过 3 分钟的任务改用 `LongTask`
- [ ] 状态管理从 V1 迁移到 V2（可选但有性能提升）
- [ ] `@State` → `@Local`
- [ ] `@Prop` → `@Param`
- [ ] `@Link` → `@Param @Once`
- [ ] 使用 `SendablePreferences` 替代传统 `preferences`
- [ ] 图片处理使用 `SendableImage` 提升跨线程性能

---

## 📚 延伸阅读

- [ArkTS 语法适配背景](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-migration-background)
- [从 TypeScript 到 ArkTS 的适配规则](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide)
- [ArkTS 高性能编程实践](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-high-performance-programming)
- [ArkTS 学习路线](https://developer.huawei.com/consumer/cn/arkts/)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 全版本 (API 9-12)