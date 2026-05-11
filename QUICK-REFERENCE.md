# 快速参考卡片

> 📋 常用 API 和代码片段速查

## 🎨 UI 组件速查

### Text 文本

```typescript
Text('内容')
  .fontSize(16)
  .fontWeight(FontWeight.Bold)
  .fontColor('#333')
  .textAlign(TextAlign.Center)
  .maxLines(2)
  .textOverflow({ overflow: TextOverflow.Ellipsis })
```

### Image 图片

```typescript
Image($r('app.media.icon'))
  .width(100)
  .height(100)
  .objectFit(ImageFit.Cover)
  .borderRadius(8)
```

### Button 按钮

```typescript
Button('点击')
  .width(200)
  .height(50)
  .backgroundColor('#007DFF')
  .fontColor('#FFF')
  .borderRadius(25)
  .onClick(() => { })
```

### TextInput 输入框

```typescript
TextInput({ placeholder: '请输入' })
  .width(300)
  .height(50)
  .onChange((value: string) => { })
```

---

## 📐 布局速查

### Column 纵向布局

```typescript
Column() {
  // 子组件
}
.width('100%')
.height('100%')
.justifyContent(FlexAlign.Center)
.alignItems(HorizontalAlign.Center)
```

### Row 横向布局

```typescript
Row() {
  // 子组件
}
.width('100%')
.height(50)
.justifyContent(FlexAlign.SpaceBetween)
```

### Flex 弹性布局

```typescript
Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap }) {
  // 子组件
}
.width('100%')
```

### Stack 层叠布局

```typescript
Stack({ alignContent: Alignment.Center }) {
  // 子组件
}
.width(200)
.height(200)
```

---

## 🔄 状态管理速查

### @State 组件状态

```typescript
@State count: number = 0;
@State message: string = 'Hello';
@State isVisible: boolean = true;
```

### @Prop 父子单向

```typescript
@Prop title: string;
@Prop data: Item;
```

### @Link 父子双向

```typescript
@Link value: number;
@Link selected: boolean;
```

### @Provide/@Consume 跨组件

```typescript
// 祖先组件
@Provide('theme') theme: string = 'light';

// 后代组件
@Consume('theme') theme: string;
```

### @Watch 监听变化

```typescript
@State @Watch('onValueChange') value: number = 0;

onValueChange(): void {
  // 值变化时触发
}
```

---

## 🎬 动画速查

### 属性动画

```typescript
// 隐式动画
animateTo({
  duration: 300,
  curve: Curve.EaseInOut
}, () => {
  this.width = 200;
})

// 组件动画
Text('Hello')
  .opacity(this.visible ? 1 : 0)
  .animation({ duration: 300 })
```

### 转场动画

```typescript
Text('Content')
  .transition({
    opacity: { from: 0, to: 1 },
    scale: { from: { x: 0.8, y: 0.8 } }
  })
```

---

## 🧭 路由速查

### 页面跳转

```typescript
import { router } from '@kit.ArkUI';

// 跳转
router.pushUrl({ url: 'pages/Detail' });

// 带参数
router.pushUrl({
  url: 'pages/Detail',
  params: { id: '123' }
});

// 返回
router.back();

// 替换
router.replaceUrl({ url: 'pages/Home' });
```

### 获取参数

```typescript
const params = router.getParams() as Record<string, string>;
```

---

## 💾 存储速查

### 首选项

```typescript
import { preferences } from '@kit.ArkData';

const store = await preferences.getPreferences(context, 'store');
await store.put('key', 'value');
await store.flush();
const value = await store.get('key', '');
```

### 数据库

```typescript
import { relationalStore } from '@kit.ArkData';

const rdbStore = await relationalStore.getRdbStore(context, config);
await rdbStore.insert('table', values);
const resultSet = await rdbStore.query(predicates, columns);
```

---

## 🌐 网络速查

### HTTP 请求

```typescript
import { http } from '@kit.NetworkKit';

const httpRequest = http.createHttp();
const response = await httpRequest.request(url, {
  method: http.RequestMethod.GET,
  header: { 'Content-Type': 'application/json' }
});
```

---

## ⚡ 并发速查

### TaskPool

```typescript
import { taskpool } from '@kit.ArkTS';

@Concurrent
function calculate(data: number[]): number {
  return data.reduce((sum, n) => sum + n, 0);
}

const task = new taskpool.Task(calculate, [1, 2, 3]);
const result = await taskpool.execute(task);
```

### Sendable 对象

```typescript
@Sendable
class SharedData {
  count: number = 0;
}

@Concurrent
function process(data: SharedData): void {
  data.count++;
}
```

### Worker

```typescript
import { worker } from '@kit.ArkTS';

const workerInstance = new worker.ThreadWorker('entry/ets/workers/task.ets');
workerInstance.postMessage({ type: 'START' });
workerInstance.onmessage = (e) => {
  console.info('Received:', e.data);
};
```

---

## 📱 设备速查

### 剪贴板

```typescript
import { pasteboard } from '@kit.BasicServicesKit';

// 复制
const data = pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, 'text');
await pasteboard.getSystemPasteboard().setData(data);

// 粘贴
const text = await pasteboard.getSystemPasteboard().getData();
```

### 通知

```typescript
import { notificationManager } from '@kit.NotificationKit';

await notificationManager.publish({
  id: 1,
  content: {
    notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
    normal: { title: '标题', text: '内容' }
  }
});
```

---

## 🔒 权限速查

### 检查权限

```typescript
import { abilityAccessCtrl } from '@kit.AbilityKit';

const atManager = abilityAccessCtrl.createAtManager();
const status = await atManager.checkAccessToken(tokenID, permission);
```

### 申请权限

```typescript
const results = await atManager.requestPermissionsFromUser(context, permissions);
```

---

## 🧪 测试速查

### 断言

```typescript
expect(value).assertEqual(expected);
expect(value).assertTrue();
expect(array).assertContain(item);
```

### Mock

```typescript
const mock = new MockFunction();
mock.setReturnValue(value);
expect(mock.getCallCount()).assertEqual(1);
```

---

## 📝 日志速查

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit';

hilog.debug(0x0000, 'Tag', 'message');
hilog.info(0x0000, 'Tag', 'message');
hilog.warn(0x0000, 'Tag', 'message');
hilog.error(0x0000, 'Tag', 'message');
```

---

## 🎯 常用工具函数

### 防抖

```typescript
function debounce<T extends (...args: any[]) => void>(
  fn: T, 
  delay: number
): (...args: Parameters<T>) => void {
  let timer: number = -1;
  return (...args: Parameters<T>) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

### 节流

```typescript
function throttle<T extends (...args: any[]) => void>(
  fn: T, 
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean = false;
  return (...args: Parameters<T>) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
```

### 生成 ID

```typescript
function generateId(): string {
  return Date.now().toString(36) + Math.random().toString(36).substring(2);
}
```

---

## 📚 资源引用速查

```typescript
// 本地资源
$r('app.media.icon')
$r('app.string.app_name')

// 网络资源
'https://example.com/image.jpg'
```

---

## 🔧 ArkTS 严格约束速查（按生效版本）

| 类别 | 被禁止的 TS 特性 | 生效版本 | 替代方案 |
|------|----------------|----------|---------|
| 类型 | `any` / `unknown` | API 10+ | 具体类型或 `Object` |
| 类型 | `index signature` | API 10+ | `Record<K,V>` 或 `Map` |
| 类型 | 交叉类型 `&` | API 10+ | 接口继承 |
| 类型 | 条件类型 | API 10+ | 重载函数式声明 |
| 类型 | 映射类型 | API 10+ | 显式声明每个属性 |
| 变量 | `var` | API 10+ | `let` / `const` |
| 变量 | 解构赋值 | API 10+ | 直接属性访问 |
| 变量 | `as const` | API 10+ | 字面量类型注解 |
| 对象 | `delete obj.prop` | API 10+ | 赋值 `undefined` |
| 对象 | 运行时添加属性 | API 10+ | 显式声明完整类型 |
| 函数 | 函数重载 | API 10+ | 可选参数 + 联合类型 |
| 函数 | 函数内声明函数 | API 10+ | Lambda 表达式 |
| 函数 | `function*` 生成器 | API 10+ | 数组/迭代器替代 |
| 类 | `#` 私有字段 | API 10+ | `private` 关键字 |
| 类 | 类表达式 | API 10+ | 显式命名类 |
| 类 | `implements` 子句 | API 10+ | `extends` 继承 |
| 遍历 | `for...in` | API 10+ | `for...of` / `Object.keys()` |
| 符号 | `Symbol()` | API 10+ | 字符串常量 |
| 模块 | `require()` | API 10+ | `import` 语句 |
| 模块 | `export = ...` | API 10+ | `export default` |
| 全局 | `globalThis` | API 10+ | 避免全局对象 |
| 安全 | `eval()` / `with()` | API 11+ | 逻辑替代 |
| 并发 | `@Sendable` 类 | **新增** API 11+ | — |
| 并发 | `@Sendable` 函数 | **新增** API 12+ | — |
| 并发 | `LongTask` | **新增** API 12+ | — |

---

## ⚖️ TS vs ArkTS 代码对照速查

```typescript
// ─── 类型 ───
// ❌ TS: any (ArkTS API 10+ 禁止)
let x: any = 123;

// ✅ ArkTS: 联合类型
let x: string | number = 123;

// ─── 类型推断 ───
// ✅ 两者都支持
let count = 10;  // 自动推断 number

// ─── 变量声明 ───
// ❌ TS: var (ArkTS API 10+ 禁止)
var old = 'legacy';

// ✅ ArkTS: let / const
let modern = 'new';
const FIXED = 'immutable';

// ─── 解构 ───
// ❌ TS: 解构 (ArkTS API 10+ 禁止)
const { name, age } = user;

// ✅ ArkTS: 直接访问
const name = user.name;
const age = user.age;

// ─── 遍历 ───
// ❌ TS: for...in (ArkTS API 10+ 禁止)
for (let k in obj) {}

// ✅ ArkTS: Object.keys()
const keys = Object.keys(obj);
for (let i = 0; i < keys.length; i++) {
  const k = keys[i];
  const v = obj[k];
}

// ─── 私有字段 ───
// ❌ TS: #private (ArkTS API 10+ 禁止)
class A { #x = 1; }

// ✅ ArkTS: private 关键字
class A { private x: number = 1; }

// ─── 模块 ───
// ❌ TS: require() (ArkTS API 10+ 禁止)
const lib = require('lib');

// ✅ ArkTS: import
import { lib } from '@kit.SomeKit';
```

---

## 🚀 高性能编码模式速查

```typescript
// ─── const 优先 ───
// ❌ 低效
let MAX: number = 100;
// ✅ 高效 - 编译器常量折叠
const MAX: number = 100;

// ─── TypedArray 替代 Array<number> ───
// ❌ 装箱开销
const arr: number[] = [1, 2, 3];
// ✅ 连续内存
const arr: Float32Array = new Float32Array([1, 2, 3]);

// ─── Map 替代稀疏数组 ───
// ❌ 引擎退化为字典
const a: number[] = []; a[99999] = 1;
// ✅ 语义准确
const m: Map<number, number> = new Map(); m.set(99999, 1);

// ─── Set/Map O(1) 查找 ───
// ❌ O(n) 线性查找
arr.includes(id);
// ✅ O(1)
const set: Set<string> = new Set(arr); set.has(id);

// ─── 提取循环不变量 ───
// ❌ 每次迭代读取
for (let i = 0; i < this.list.length; i++) {}
// ✅ 循环外提取
const len = this.list.length;
for (let i = 0; i < len; i++) {}

// ─── 数组字面量 ───
// ❌
const a: number[] = new Array<number>(); a.push(1);
// ✅
const a: number[] = [1];
```

---

## 📋 .ets 文件编译模式速查

| compatibleSdkVersion | 模式 | TS 语法行为 | 升级建议 |
|---------------------|------|-----------|---------|
| `< 10` (如 9) | 兼容模式 | ⚠️ warning，编译可过 | 先消除所有 warning |
| `>= 10` (如 10/11/12) | 标准模式 | ❌ error，编译失败 | 必须全部修正 |

```json5
// build-profile.json5
{
  "app": {
    "products": [{
      "compatibleSdkVersion": 12,  // ≥10 → 标准模式
      "runtimeOS": "HarmonyOS"
    }]
  }
}
```

---

> 📝 **最后更新**: 2026-05-10
> 📌 **适用版本**: HarmonyOS 全版本 (API 9-12)
