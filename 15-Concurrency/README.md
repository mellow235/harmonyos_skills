# 并发编程

> ⚡ ArkTS 多线程与异步编程核心能力

## 📖 目录

1. [并发概述](#并发概述)
2. [异步并发](#异步并发)
3. [TaskPool 多线程](#taskpool-多线程)
4. [Worker 多线程](#worker-多线程)
5. [Sendable 对象](#sendable-对象)
6. [共享容器](#共享容器)
7. [线程间通信](#线程间通信)
8. [并发最佳实践](#并发最佳实践)
9. [常见问题](#常见问题)

---

## 并发概述

ArkTS 提供两种并发策略：

```
并发策略
├── 异步并发（单线程）
│   ├── Promise
│   └── async/await
└── 多线程并发
    ├── TaskPool（推荐）
    └── Worker
```

### 选择指南

| 场景 | 推荐方案 | 说明 |
|------|---------|------|
| 单次 I/O 任务 | async/await | 网络请求、文件读写 |
| 耗时计算任务 | TaskPool | 图像处理、数据解析 |
| 需要长期运行的后台任务 | Worker | 音乐播放、定位服务 |
| 需要与宿主线程频繁通信 | Worker | 实时数据处理 |

---

## 异步并发

### Promise

```typescript
// 创建 Promise
const promise = new Promise<string>((resolve, reject) => {
  setTimeout(() => {
    resolve('Success');
  }, 1000);
});

// 使用
promise.then((result) => {
  console.info(result);
}).catch((error) => {
  console.error(error);
});
```

### async/await

```typescript
async function fetchData(): Promise<string> {
  try {
    const response = await httpRequest.request(url);
    return response.result as string;
  } catch (error) {
    console.error('Request failed:', error);
    throw error;
  }
}

// 调用
async function loadData(): Promise<void> {
  const data = await fetchData();
  console.info(data);
}
```

### Promise.all 并行执行

```typescript
async function loadMultiple(): Promise<void> {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  
  console.info(`Loaded: ${users.length} users, ${posts.length} posts`);
}
```

---

## TaskPool 多线程

### 基本用法

```typescript
import { taskpool } from '@kit.ArkTS';

// 1. 定义并发函数（必须用 @Concurrent 装饰器）
@Concurrent
function calculateSum(numbers: number[]): number {
  let sum = 0;
  for (const num of numbers) {
    sum += num;
  }
  return sum;
}

// 2. 创建任务并执行
async function runTask(): Promise<void> {
  const numbers = [1, 2, 3, 4, 5];
  const task = new taskpool.Task(calculateSum, numbers);
  
  try {
    const result = await taskpool.execute(task) as number;
    console.info(`Result: ${result}`);
  } catch (error) {
    console.error('Task failed:', error);
  }
}
```

### @Concurrent 装饰器约束

```typescript
// ✅ 正确：使用局部变量和入参
@Concurrent
function validFunction(data: number[]): number {
  const local = 10;  // 局部变量
  return data.length + local;
}

// ❌ 错误：访问闭包变量
let globalVar = 10;

@Concurrent
function invalidFunction(): number {
  return globalVar;  // 编译错误！不能访问闭包
}

// ✅ 正确：导入的变量可以使用
import { utils } from './utils';

@Concurrent
function validWithImport(): number {
  return utils.calculate();  // 可以访问导入的模块
}
```

### TaskGroup 批量任务

```typescript
@Concurrent
function processItem(item: string): string {
  return item.toUpperCase();
}

async function processBatch(items: string[]): Promise<string[]> {
  const taskGroup = new taskpool.TaskGroup();
  
  for (const item of items) {
    const task = new taskpool.Task(processItem, item);
    taskGroup.addTask(task);
  }
  
  const results = await taskpool.execute(taskGroup) as string[];
  return results;
}
```

### ArrayBuffer 传输

```typescript
@Concurrent
function processBuffer(buffer: ArrayBuffer): ArrayBuffer {
  const view = new Uint8Array(buffer);
  for (let i = 0; i < view.length; i++) {
    view[i] = view[i] * 2;
  }
  return buffer;
}

async function transferBuffer(): Promise<void> {
  const buffer = new ArrayBuffer(1024);
  const task = new taskpool.Task(processBuffer, buffer);
  
  // 默认转移所有权（原 buffer 不可用）
  task.setTransferList([buffer]);
  
  // 或设置为拷贝（原 buffer 仍可用）
  // task.setCloneList([buffer]);
  
  const result = await taskpool.execute(task) as ArrayBuffer;
  console.info(`Processed buffer size: ${result.byteLength}`);
}
```

### TaskPool 注意事项

| 限制 | 说明 |
|------|------|
| 执行时长 | 普通任务不超过 3 分钟 |
| 数据大小 | 序列化传输限制 16MB |
| 装饰器 | 必须使用 `@Concurrent` |
| 闭包 | 禁止访问闭包变量 |
| UI 操作 | 工作线程不能操作 UI |
| AppStorage | 工作线程不支持 |

---

## Worker 多线程

### 创建 Worker

```typescript
// Index.ets（宿主线程）
import { worker, MessageEvents, ErrorEvent } from '@kit.ArkTS';

@Entry
@Component
struct WorkerDemo {
  private workerInstance: worker.ThreadWorker | null = null;

  aboutToAppear(): void {
    // 创建 Worker
    this.workerInstance = new worker.ThreadWorker('entry/ets/workers/myWorker.ets');
    
    // 监听消息
    this.workerInstance.onmessage = (e: MessageEvents) => {
      console.info('Main thread received:', e.data);
    };
    
    // 监听错误
    this.workerInstance.onAllErrors = (err: ErrorEvent) => {
      console.error('Worker error:', err.message);
    };
  }

  aboutToDisappear(): void {
    // 销毁 Worker
    this.workerInstance?.terminate();
  }

  sendTask(): void {
    this.workerInstance?.postMessage({ type: 'CALCULATE', data: [1, 2, 3, 4, 5] });
  }

  build() {
    Column() {
      Button('Send Task')
        .onClick(() => this.sendTask())
    }
  }
}
```

### Worker 文件

```typescript
// workers/myWorker.ets
import { worker, MessageEvents, ThreadWorkerGlobalScope } from '@kit.ArkTS';

const workerPort: ThreadWorkerGlobalScope = worker.workerPort;

workerPort.onmessage = (e: MessageEvents) => {
  const { type, data } = e.data;
  
  switch (type) {
    case 'CALCULATE':
      const result = calculate(data);
      workerPort.postMessage({ type: 'RESULT', data: result });
      break;
    default:
      workerPort.postMessage({ type: 'ERROR', message: 'Unknown type' });
  }
};

function calculate(numbers: number[]): number {
  return numbers.reduce((sum, num) => sum + num, 0);
}
```

### Worker 配置

```json5
// build-profile.json5
{
  "buildOption": {
    "sourceOption": {
      "workers": [
        "./src/main/ets/workers/myWorker.ets"
      ]
    }
  }
}
```

### Worker vs TaskPool

| 特性 | TaskPool | Worker |
|------|----------|--------|
| 线程管理 | 系统自动管理 | 开发者手动管理 |
| 生命周期 | 自动回收 | 需手动创建/销毁 |
| 适用场景 | 短期耗时任务 | 长期运行任务 |
| 通信方式 | 返回 Promise | 消息传递 |
| 数量限制 | 动态扩缩容 | 最多 64 个 |

---

## Sendable 对象

### 基本概念

Sendable 对象可在 ArkTS 并发实例间通过**引用传递**共享，避免序列化开销。

```typescript
import { taskpool } from '@kit.ArkTS';

// 定义 Sendable 类
@Sendable
class SharedData {
  count: number = 0;
  message: string = 'Hello';

  increment(): void {
    this.count++;
  }
}

@Concurrent
function processData(data: SharedData): void {
  data.increment();
  console.info(`Worker: count = ${data.count}`);
}

async function demo(): Promise<void> {
  const data = new SharedData();
  
  const task = new taskpool.Task(processData, data);
  await taskpool.execute(task);
  
  // data.count 已经被修改！
  console.info(`Main: count = ${data.count}`);
}
```

### Sendable 约束

```typescript
// ✅ 正确
@Sendable
class ValidSendable {
  name: string = '';
  age: number = 0;
  isActive: boolean = true;
}

// ❌ 错误：包含非 Sendable 类型
@Sendable
class InvalidSendable {
  data: any;  // 错误：any 不允许
  callback: () => void;  // 错误：普通函数不允许
}
```

### Sendable 支持类型

| 类型 | 支持 |
|------|------|
| boolean, number, string, bigint | ✅ |
| null, undefined | ✅ |
| @Sendable class | ✅ |
| collections.Array/Map/Set | ✅ |
| ArkTSUtils.locks.AsyncLock | ✅ |
| 普通 Object/Array | ❌ |
| any/unknown | ❌ |

---

## 共享容器

### 使用 @arkts.collections

```typescript
import { collections } from '@kit.ArkTS';

// 创建共享数组
const sharedArray = collections.Array.create<number>(10, 0);

// 使用
sharedArray[0] = 100;
sharedArray.push(200);

// 遍历
for (const item of sharedArray) {
  console.info(item);
}
```

### 线程安全操作

```typescript
import { ArkTSUtils, collections } from '@kit.ArkTS';

@Concurrent
async function safeIncrement(
  arr: collections.Array<number>, 
  lock: ArkTSUtils.locks.AsyncLock
): Promise<void> {
  await lock.lockAsync(() => {
    arr[0]++;
  });
}

async function demo(): Promise<void> {
  const lock = new ArkTSUtils.locks.AsyncLock();
  const arr = collections.Array.create<number>(1, 0);
  
  const tasks: taskpool.Task[] = [];
  for (let i = 0; i < 100; i++) {
    tasks.push(new taskpool.Task(safeIncrement, arr, lock));
  }
  
  await Promise.all(tasks.map(t => taskpool.execute(t)));
  console.info(`Result: ${arr[0]}`);  // 100
}
```

---

## 线程间通信

### 通信方式对比

| 方式 | 数据类型 | 性能 | 适用场景 |
|------|---------|------|----------|
| 序列化传递 | 基础类型、对象字面量 | 低（拷贝） | 小数据量 |
| 引用传递（Sendable） | @Sendable 对象 | 高（共享） | 大数据量 |
| 转移所有权 | ArrayBuffer | 高 | 二进制数据 |

### 序列化传递

```typescript
@Concurrent
function processObject(data: object): object {
  return { ...data, processed: true };
}

// 数据会被序列化拷贝
const result = await taskpool.execute(
  new taskpool.Task(processObject, { name: 'test' })
);
```

### 引用传递

```typescript
@Sendable
class DataModel {
  items: string[] = [];
}

@Concurrent
function addItem(model: DataModel, item: string): void {
  model.items.push(item);  // 直接修改共享对象
}
```

---

## 并发最佳实践

### 1. 合理选择并发方案

```typescript
// ✅ 异步 I/O 用 async/await
async function fetchUser(): Promise<User> {
  const response = await httpRequest.request(url);
  return JSON.parse(response.result as string);
}

// ✅ 耗时计算用 TaskPool
@Concurrent
function heavyCalculation(data: number[]): number {
  // CPU 密集型任务
  return data.reduce((sum, n) => sum + complexMath(n), 0);
}

// ✅ 长期任务用 Worker
// 音乐播放器后台解码
```

### 2. 避免线程阻塞

```typescript
// ❌ 错误：在工作线程执行同步 I/O
@Concurrent
function badPractice(): void {
  const file = fs.openSync(path);  // 阻塞！
  // ...
}

// ✅ 正确：使用异步 API
@Concurrent
async function goodPractice(): Promise<void> {
  const file = await fs.open(path);  // 非阻塞
  // ...
}
```

### 3. 错误处理

```typescript
@Concurrent
async function robustTask(): Promise<Result> {
  try {
    const data = await fetchData();
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error.message };
  }
}
```

### 4. 资源清理

```typescript
@Entry
@Component
struct WorkerPage {
  private worker: worker.ThreadWorker | null = null;

  aboutToAppear(): void {
    this.worker = new worker.ThreadWorker('entry/ets/workers/task.ets');
  }

  aboutToDisappear(): void {
    // 必须清理资源
    this.worker?.terminate();
    this.worker = null;
  }
}
```

---

## 常见问题

### Q1: TaskPool 任务执行超时？

**A:** 普通任务限制 3 分钟，使用 `LongTask` 类型处理长期任务：

```typescript
import { taskpool } from '@kit.ArkTS';

@Concurrent
function longRunningTask(): void {
  // 执行超过 3 分钟的任务
}

const task = new taskpool.Task(longRunningTask);
const longTask = new taskpool.LongTask(task);  // API 12+
```

### Q2: Sendable 对象修改不生效？

**A:** 确保所有属性都是 Sendable 兼容类型，且使用 `@Sendable` 装饰器。

### Q3: Worker 通信数据大小限制？

**A:** 单次消息限制 16MB，大数据量使用 Sendable 引用传递。

### Q4: 如何判断使用 TaskPool 还是 Worker？

**A:** 
- TaskPool：短期、可重用的计算任务
- Worker：长期运行、需要持续通信的任务

---

## 📚 延伸阅读

- [ArkTS 并发概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-concurrency)
- [TaskPool 简介](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/taskpool-introduction)
- [Worker 简介](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/worker-introduction)
- [Sendable 对象](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-sendable)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
