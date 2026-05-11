# ArkTS 语言基础

> 🎯 HarmonyOS 应用开发的核心语言

## 📖 目录

1. [语言概述](#语言概述)
2. [ArkTS 与 TypeScript 的关系](#arkts-与-typescript-的关系)
3. [.ets 文件编译模式](#ets-文件编译模式)
4. [基础语法](#基础语法)
5. [类型系统](#类型系统)
6. [函数](#函数)
7. [类与接口](#类与接口)
8. [装饰器](#装饰器)
9. [ArkTS扩展特性](#arkts扩展特性)
10. [ArkTS与TypeScript差异](#arkts与typescript差异)
11. [高性能编程实践](#高性能编程实践)
12. [常见问题](#常见问题)
13. [最佳实践](#最佳实践)

---

## 语言概述

ArkTS 是 HarmonyOS 应用开发的主力语言。**它不是 TypeScript 的超集，而是基于 TS 语法、对其进行了大量约束的精简子集，并扩展了 ArkUI 声明式 UI 能力。**

### 核心认知

```
TypeScript 语言规范
    │
    ├── ArkTS 移除了大量 TS 特性（严格约束子集）
    │   ├── 禁止 any/unknown、for...in、解构、函数重载...
    │   ├── 强制静态类型 + AOT 编译
    │   └── = 更小、更确定的语法面
    │
    └── ArkTS 扩展了自身独有特性
        ├── 声明式 UI（struct + build()）
        ├── 状态管理装饰器（@State、@Prop...）
        ├── 并发模型（@Sendable、TaskPool）
        └── 平台 API 集成
```

### 核心特性
- **TypeScript 约束子集**：从 API 10 起明确移除了大量 TS 特性以保障 AOT 性能
- **声明式 UI**：基于 ArkUI 框架，`struct` + `build()` 模式
- **强制静态类型**：编译时类型检查，确定性对象布局
- **高性能**：方舟编译器 AOT 编译，无运行时动态类型开销
- **内置并发模型**：TaskPool + Worker + Sendable 对象

---

## ArkTS 与 TypeScript 的关系

### ⚠️ 关键纠正：ArkTS 不是 TS 的超集

```
常见误解 ❌:
  TypeScript ⊂ ArkTS (ArkTS = TS + 扩展)

正确理解 ✅:
  ArkTS ⊂ TypeScript  (ArkTS = TS 严格子集 + 专有扩展)
```

| 维度 | TypeScript | ArkTS (API 10+) |
|------|-----------|----------------|
| `any` / `unknown` | ✅ 核心类型 | ❌ 禁止 |
| `for...in` | ✅ | ❌ 禁止 |
| 解构赋值 | ✅ | ❌ 禁止 |
| 函数重载 | ✅ | ❌ 禁止 |
| `Symbol()` | ✅ | ❌ 禁止 |
| `#` 私有字段 | ✅ | ❌ 禁止 |
| `delete` 运算符 | ✅ | ❌ 禁止 |
| 声明式 UI | ❌ | ✅ `struct` + `build()` |
| 状态管理装饰器 | ❌ | ✅ `@State`、`@Prop`... |
| 并发装饰器 | ❌ | ✅ `@Concurrent`、`@Sendable` |

> **一句话总结**：ArkTS = "精简版 TypeScript + HarmonyOS 专属扩展"。如果你写惯了 TS，进入 ArkTS 时需要**收敛语法习惯**，而非扩展。

---

## .ets 文件编译模式

同一份 `.ets` 文件，在不同 `compatibleSdkVersion` 下编译行为截然不同：

| compatibleSdkVersion | 编译模式 | 行为 |
|---------------------|---------|------|
| `< 10` (如 9) | **兼容模式** | 违反 ArkTS 语法 → ⚠️ warning，编译可成功 |
| `>= 10` (如 10/11/12) | **标准模式** | 违反 ArkTS 语法 → ❌ error，编译一定失败 |

### 工程配置

```json5
// build-profile.json5 - 关键字段
{
  "app": {
    "products": [
      {
        "compatibleSdkVersion": 12,  // ≥10 → 标准模式，严格检查
        "runtimeOS": "HarmonyOS"
      }
    ]
  }
}
```

### 实际影响

```typescript
// 以下代码在 compatibleSdkVersion = 9 时只产生 warning，编译可过
// 在 compatibleSdkVersion = 10+ 时产生 error，编译失败
let value: any = 'hello';  // API 9: warning  |  API 10+: ❌ error
for (let key in obj) {}    // API 9: warning  |  API 10+: ❌ error
```

> **迁移建议**：从 API 9 升级时，先在兼容模式下消除所有 warning，再将 `compatibleSdkVersion` 设为 >= 10。</parameter>

---

## 基础语法

### 变量声明

```typescript
// let - 可变变量
let name: string = 'HarmonyOS';
let version: number = 6.0;
let isActive: boolean = true;

// const - 不可变变量
const PI: number = 3.14159;
const APP_NAME: string = 'MyApp';

// 类型推断
let count = 10;  // 自动推断为 number
let items = ['a', 'b', 'c'];  // 自动推断为 string[]
```

### 数据类型

```typescript
// 基础类型
let str: string = 'hello';
let num: number = 42;
let bool: boolean = true;
let undef: undefined = undefined;
let nul: null = null;

// 数组
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ['Alice', 'Bob'];

// 元组
let tuple: [string, number] = ['age', 25];

// 枚举
enum Color {
  Red = 'RED',
  Green = 'GREEN',
  Blue = 'BLUE'
}

// 联合类型
let id: string | number = '001';
id = 1;

// 字面量类型
type Direction = 'up' | 'down' | 'left' | 'right';
```

### 控制流

```typescript
// if-else
if (score >= 90) {
  grade = 'A';
} else if (score >= 80) {
  grade = 'B';
} else {
  grade = 'C';
}

// 三元运算
let status = isActive ? 'Active' : 'Inactive';

// switch
switch (direction) {
  case 'up':
    y--;
    break;
  case 'down':
    y++;
    break;
  default:
    break;
}

// for 循环
for (let i = 0; i < 10; i++) {
  console.info(i);
}

// for...of (遍历值)
for (const item of items) {
  console.info(item);
}

// while
while (condition) {
  // do something
}
```

---

## 类型系统

### 接口

```typescript
interface User {
  id: number;
  name: string;
  email?: string;  // 可选属性
  readonly createdAt: Date;  // 只读属性
}

interface Admin extends User {
  permissions: string[];
}

// 使用
const user: User = {
  id: 1,
  name: 'Alice',
  createdAt: new Date()
};
```

### 类型别名

```typescript
type Point = {
  x: number;
  y: number;
};

type StringOrNumber = string | number;

type Callback = (data: string) => void;
```

### 泛型

```typescript
// 泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 泛型接口
interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
}

// 泛型类
class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
}

// 泛型约束
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): number {
  return arg.length;
}
```

---

## 函数

### 函数定义

```typescript
// 基础函数
function add(a: number, b: number): number {
  return a + b;
}

// 箭头函数
const multiply = (a: number, b: number): number => a * b;

// 可选参数
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Hello'}, ${name}!`;
}

// 默认参数
function createUser(name: string, role: string = 'user'): User {
  return { name, role };
}

// 剩余参数
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, val) => acc + val, 0);
}
```

### 回调函数

```typescript
type EventHandler = (event: Event) => void;

function onClick(handler: EventHandler): void {
  // 注册点击事件
}

// 使用
onClick((event) => {
  console.info('Clicked:', event);
});
```

---

## 类与接口

### 类定义

```typescript
class Person {
  // 属性声明
  private name: string;
  protected age: number;
  public email: string;

  // 构造函数
  constructor(name: string, age: number, email: string) {
    this.name = name;
    this.age = age;
    this.email = email;
  }

  // 方法
  greet(): string {
    return `Hi, I'm ${this.name}`;
  }

  // getter/setter
  get info(): string {
    return `${this.name}, ${this.age} years old`;
  }
}

// 继承
class Employee extends Person {
  private department: string;

  constructor(name: string, age: number, email: string, department: string) {
    super(name, age, email);
    this.department = department;
  }

  // 方法重写
  greet(): string {
    return `${super.greet()} and I work in ${this.department}`;
  }
}
```

### 抽象类

```typescript
abstract class Shape {
  abstract area(): number;
  abstract perimeter(): number;

  describe(): string {
    return `Area: ${this.area()}, Perimeter: ${this.perimeter()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }

  perimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}
```

### 接口实现

```typescript
interface Printable {
  print(): void;
}

interface Serializable {
  serialize(): string;
}

class Document implements Printable, Serializable {
  constructor(private content: string) {}

  print(): void {
    console.info(this.content);
  }

  serialize(): string {
    return JSON.stringify({ content: this.content });
  }
}
```

---

## 装饰器

### 组件装饰器

```typescript
// @Entry - 入口组件
@Entry
@Component
struct Index {
  build() {
    Column() {
      Text('Hello')
    }
  }
}

// @Component - 组件装饰器
@Component
struct MyComponent {
  build() {
    Row() {
      // ...
    }
  }
}

// @CustomDialog - 自定义弹窗
@CustomDialog
struct MyDialog {
  controller: CustomDialogController;

  build() {
    Column() {
      // ...
    }
  }
}
```

### 状态装饰器

```typescript
@Component
struct Counter {
  @State count: number = 0;  // 组件内状态
  @Prop title: string;  // 父组件单向传递
  @Link value: number;  // 父子双向同步

  build() {
    Column() {
      Text(`${this.title}: ${this.count}`)
      Button('Add')
        .onClick(() => {
          this.count++;
        })
    }
  }
}
```

---

## ArkTS扩展特性

### 声明式 UI

```typescript
// ArkTS 声明式 UI 语法
@Entry
@Component
struct HelloWorld {
  @State message: string = 'Hello HarmonyOS';

  build() {
    // 声明式描述 UI 结构
    Column() {
      Text(this.message)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 20 })

      Button('Click Me')
        .onClick(() => {
          this.message = 'Clicked!';
        })
        .backgroundColor('#007DFF')
        .borderRadius(8)
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### 链式调用

```typescript
// 组件属性通过链式调用设置
Text('Hello')
  .fontSize(16)
  .fontColor('#333')
  .textAlign(TextAlign.Center)
  .margin({ top: 10, bottom: 10 })
  .padding(8)
  .backgroundColor('#F5F5F5')
  .borderRadius(4)
```

### 条件渲染

```typescript
@Component
struct ConditionalRender {
  @State isVisible: boolean = true;

  build() {
    Column() {
      // 三元运算
      if (this.isVisible) {
        Text('Visible')
      } else {
        Text('Hidden')
      }

      // 条件组件
      this.isVisible ? this.buildVisible() : this.buildHidden()
    }
  }

  @Builder
  buildVisible() {
    Text('Visible Content')
  }

  @Builder
  buildHidden() {
    Text('Hidden Content')
  }
}
```

### 列表渲染

```typescript
@Component
struct ListRender {
  @State items: string[] = ['Apple', 'Banana', 'Cherry'];

  build() {
    List() {
      ForEach(this.items, (item: string, index: number) => {
        ListItem() {
          Text(`${index + 1}. ${item}`)
            .fontSize(16)
            .padding(12)
        }
      }, (item: string) => item)  // keyGenerator
    }
  }
}
```

---

## ArkTS与TypeScript差异

### 差异总览（按生效版本）

| 类别 | TS 特性 | API 9 | API 10+ | ArkTS 替代方案 |
|------|--------|-------|---------|---------------|
| **类型** | `any` / `unknown` | ✅ | ❌ | 具体类型或 `Object` |
| **类型** | `index signature` `[key: string]` | ✅ | ❌ | `Record<K, V>` 或 `Map` |
| **类型** | `intersection type` (`&`) | ✅ | ❌ | 接口继承 |
| **类型** | `conditional type` | ✅ | ❌ | 重载函数式声明 |
| **类型** | `mapped type` | ✅ | ❌ | 显式声明每个属性 |
| **变量** | `var` | ✅ | ❌ | `let` / `const` |
| **变量** | 解构赋值 `const {a} = obj` | ✅ | ❌ | 直接属性访问 |
| **变量** | `as const` 常量断言 | ✅ | ❌ | 字面量类型 |
| **对象** | `delete obj.prop` | ✅ | ❌ | 赋值 `undefined` 或重构 |
| **对象** | `(obj as any).newProp = v` | ✅ | ❌ | 显式声明完整类型 |
| **对象** | `structural typing` | ✅ | ❌ | 类继承或接口实现 |
| **函数** | 函数重载 | ✅ | ❌ | 可选参数 + 联合类型 |
| **函数** | 函数内声明函数 | ✅ | ❌ | Lambda 表达式 |
| **函数** | 生成器 `function*` | ✅ | ❌ | 使用数组/迭代器替代 |
| **类** | `#` 私有字段 | ✅ | ❌ | `private` 关键字 |
| **类** | 类表达式 `const C = class {}` | ✅ | ❌ | 显式命名类 |
| **类** | `implements` 子句 | ✅ | ❌ | `extends` 继承 |
| **类** | `constructor` 中声明字段 | ✅ | ❌ | 类体顶层声明字段 |
| **遍历** | `for...in` | ✅ | ❌ | `for...of` 或 `Object.keys()` |
| **符号** | `Symbol()` | ✅ | ❌ | 字符串常量 |
| **模块** | `require()` | ✅ | ❌ | `import` 语句 |
| **模块** | `export = ...` | ✅ | ❌ | `export default` |
| **全局** | `globalThis` | ✅ | ❌ | 避免全局对象 |
| **并发** | `@Sendable` 类 | ❌ | API 11+ ✅ | — |
| **并发** | `@Sendable` 函数 | ❌ | API 12+ ✅ | — |

> **版本说明**：从 **API 10（ArkTS 4.0）** 起，上表中标记 ❌ 的特性全部进入标准模式，编译报错。API 9 中这些特性仍可正常使用。

### 代码对照示例

```typescript
// ── 类型安全 ──
// ❌ TS 合法，ArkTS API 10+ 禁止
let value: any = 'hello';
value = 123;

// ✅ ArkTS 正确写法
let value: string | number = 'hello';
value = 123;

// ── 对象布局 ──
// ❌ TS 合法，ArkTS API 10+ 禁止
class Point {
  x: number = 0;
  y: number = 0;
}
let p = new Point();
(p as Record<string, number>).z = 10;  // 运行时变更布局

// ✅ ArkTS 正确写法
class Point3D {
  x: number = 0;
  y: number = 0;
  z: number = 0;
}
let p = new Point3D();
p.z = 10;

// ── 解构赋值 ──
// ❌ TS 合法，ArkTS API 10+ 禁止
const { name, age } = user;
const [first, second] = array;

// ✅ ArkTS 正确写法
const name = user.name;
const age = user.age;
const first = array[0];
const second = array[1];

// ── for...in ──
// ❌ TS 合法，ArkTS API 10+ 禁止
for (let key in obj) {
  console.info(key, obj[key]);
}

// ✅ ArkTS 正确写法
const keys = Object.keys(obj);
for (let i = 0; i < keys.length; i++) {
  const key = keys[i];
  console.info(key, obj[key]);
}

// ── 函数重载 ──
// ❌ TS 合法，ArkTS API 10+ 禁止
function fn(x: string): string;
function fn(x: number): number;
function fn(x: string | number): string | number {
  return x;
}

// ✅ ArkTS 正确写法：单一签名
function fn(x: string | number): string | number {
  return x;
}
```

---

## 高性能编程实践

> 来源：[ArkTS 高性能编程官方指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-high-performance-programming)

### 1. 优先使用 const

```typescript
// ❌ 低效 - let 需要运行时检查变更
let MAX_SIZE: number = 100;

// ✅ 高效 - const 允许编译器做常量折叠优化
const MAX_SIZE: number = 100;
const PI: number = 3.14159;
const API_BASE: string = 'https://api.example.com';
```

### 2. 避免可选属性——用默认值替代

```typescript
// ❌ 可选属性增加运行时 undefined 检查
interface Config {
  timeout?: number;
  retry?: number;
}

// ✅ 使用必填属性 + 默认值
interface Config {
  timeout: number;
  retry: number;
}
const defaultConfig: Config = { timeout: 3000, retry: 3 };

function processConfig(config?: Config): void {
  const merged: Config = {
    timeout: config?.timeout ?? defaultConfig.timeout,
    retry: config?.retry ?? defaultConfig.retry
  };
}
```

### 3. 数值数组用 TypedArray

```typescript
// ❌ 普通数组 - 每次访问都有装箱/拆箱开销
const temps: number[] = [36.5, 36.7, 36.8];

// ✅ TypedArray - 连续内存布局，缓存友好
const temps: Float32Array = new Float32Array([36.5, 36.7, 36.8]);

// 大规模整数数据
const ids: Int32Array = new Int32Array(100000);
```

### 4. 避免稀疏数组

```typescript
// ❌ 稀疏数组 - 引擎退化为字典模式
const sparse: number[] = [];
sparse[0] = 1;
sparse[99999] = 2;  // 中间全是空位

// ✅ 使用 Map 存储稀疏键值
const sparse: Map<number, number> = new Map();
sparse.set(0, 1);
sparse.set(99999, 2);
```

### 5. 循环优化——提取不变量

```typescript
// ❌ 每次迭代都访问 this.obj.prop
for (let i = 0; i < this.data.length; i++) {
  process(this.data[i], this.config.threshold);
}

// ✅ 提取常量到循环外
const len: number = this.data.length;
const threshold: number = this.config.threshold;
for (let i = 0; i < len; i++) {
  process(this.data[i], threshold);
}
```

### 6. 避免频繁异常

```typescript
// ❌ 用异常处理业务逻辑——极低效
function parseAge(input: string): number {
  try {
    return Number.parseInt(input);
  } catch (e) {
    return -1;
  }
}

// ✅ 用条件判断
function parseAge(input: string): number {
  const result: number = Number.parseInt(input);
  return Number.isNaN(result) ? -1 : result;
}
```

### 7. 查找场景用 Map/Set

```typescript
// ❌ 数组线性查找——O(n)
const allowedIds: string[] = ['a', 'b', 'c'];
function isAllowed(id: string): boolean {
  return allowedIds.includes(id);
}

// ✅ Set O(1) 查找
const allowedIds: Set<string> = new Set(['a', 'b', 'c']);
function isAllowed(id: string): boolean {
  return allowedIds.has(id);
}
```

### 8. 数组字面量初始化

```typescript
// ❌ new Array() + push
const list: number[] = new Array<number>();
list.push(1);
list.push(2);

// ✅ 字面量初始化——引擎优化
const list: number[] = [1, 2];
```

---

## 常见问题

### Q1: ArkTS 和 TypeScript 到底有什么区别？

**A:** ArkTS **不是** TypeScript 的超集。准确关系：

```
ArkTS = (TypeScript 严格约束子集) + (ArkUI 声明式扩展)
```

**关键差异：**
- API 9 阶段 TS 语法基本完全兼容（兼容模式）
- API 10+ **移除了 20+ 项 TS 特性**（标准模式，编译必过检查）
- ArkTS 额外增加了声明式 UI、状态管理装饰器、并发原语

### Q2: 为什么 `.ets` 文件有时能用 TS 语法，有时不能？

**A:** 由 `compatibleSdkVersion` 决定：
- `compatibleSdkVersion < 10`（通常是 9）→ **兼容模式**，TS 语法只报 warning
- `compatibleSdkVersion >= 10` → **标准模式**，违反 ArkTS 规则直接 error

这是从旧项目平滑升级的过渡设计。

### Q3: 为什么需要使用 struct 而不是 class 定义组件？

**A:** ArkTS 中 UI 组件使用 `struct` 定义：
- 更轻量，无 class 继承链开销
- 编译期可完全确定布局，利于 AOT 优化
- 符合声明式 UI 的设计哲学
- `@Component` 装饰器只能修饰 `struct`

### Q4: 从 TypeScript 项目迁移到 ArkTS API 10+ 需要注意什么？

**A:** 按优先级检查：
1. **检查 compatibleSdkVersion**：先设为 9 看 warning 数量
2. 替换所有 `any`/`unknown` 为具体类型
3. 移除 `for...in`、解构赋值
4. 替换 `#private` → `private`，移除 `Symbol()`
5. 移除函数重载、类表达式、`implements`
6. 确认 `compatibleSdkVersion >= 10`，逐一修复 error

### Q5: API 11 和 API 12 相比 API 10 主要增强了什么？

**A:**
- **API 11**：并发能力（@Sendable 类、共享容器、AsyncLock）、移除 `eval()`/`with()`
- **API 12**：@Sendable 函数、LongTask（超3分钟任务）、状态管理 V2（@ComponentV2/@Local/@Param）

---

## 最佳实践

### 1. 类型安全——零 any
```typescript
// ✅ 明确类型
function getUser(id: number): User | null {
  // ...
}

// ❌ any 在 API 10+ 直接编译失败
function getData(id: number): Record<string, Object> {
  // ...
}
```

### 2. const 优先
```typescript
// ✅ 优先 const
const API_BASE: string = 'https://api.example.com';
const MAX_RETRY: number = 3;

// let 仅用于确实需要重新赋值的场景
let retryCount: number = 0;
```

### 3. 常量枚举——避免魔法字符串
```typescript
// ✅ 使用 const enum
const enum Status {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE'
}

// ❌ 字符串散落
if (status === 'ACTIVE') { }
```

### 4. 接口声明对象形状
```typescript
// ✅ 接口——可被 extends 扩展
interface User {
  name: string;
  age: number;
}

// ⚠️ type 别名——简单联合类型可用
type Status = 'active' | 'inactive';
```

### 5. 函数职责单一
```typescript
// ✅ 单一职责
function validateEmail(email: string): boolean { }
function formatUser(user: User): string { }

// ❌ 职责混乱
function processUser(user: User): string {
  // 验证 + 格式化 + 保存
}
```

### 6. 并发场景用 Sendable（API 11+）
```typescript
// ✅ 大数据量跨线程使用 Sendable + 共享容器
@Sendable
class SharedModel {
  data: collections.Array<number> = new collections.Array<number>();
}

// 避免序列化拷贝，使用引用传递
```

### 7. 性能敏感场景：TypedArray 替代 Array\<number\>
```typescript
// ✅ 大规模数值数据用 TypedArray
const vertices: Float32Array = new Float32Array(300000);

// ❌ 普通数组：装箱开销大
const vertices: number[] = new Array<number>(300000);
```

---

## 📚 延伸阅读

- [ArkTS 官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts)
- [TypeScript 官方文档](https://www.typescriptlang.org/)
- [ArkTS 编码规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-coding-style-guide)
- [从 TypeScript 到 ArkTS 的适配规则](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
