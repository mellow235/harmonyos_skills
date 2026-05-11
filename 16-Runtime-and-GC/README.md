# 运行时与GC优化

> ⚙️ ArkTS 运行时原理与内存管理

## 📖 目录

1. [ArkTS 运行时概述](#arkts-运行时概述)
2. [GC 垃圾回收](#gc-垃圾回收)
3. [内存管理](#内存管理)
4. [性能优化](#性能优化)
5. [调试工具](#调试工具)
6. [常见问题](#常见问题)

---

## ArkTS 运行时概述

ArkTS 运行时（ArkRuntime）是 HarmonyOS 上应用的默认语言运行时，支持 ArkTS/TS/JS 的字节码执行。

### 运行时架构

```
ArkTS Runtime
├── Core Subsystem
│   ├── File（字节码文件）
│   ├── Tooling（调试支持）
│   └── Base（系统调用适配）
├── Execution Subsystem
│   ├── 解释器
│   ├── 内联缓存
│   └── 模块化管理
├── Compiler Subsystem
│   ├── Stub 编译器
│   ├── AOT 静态编译
│   └── JIT 动态编译（实验性）
└── Runtime Subsystem
    ├── 内存管理（GC）
    ├── 分析工具（DFX/Profiling）
    ├── 并发管理
    └── 标准库
```

### 执行方式

| 方式 | 说明 | 性能 |
|------|------|------|
| 解释执行 | 直接解释字节码 | 一般 |
| AOT 编译 | 安装时预编译为机器码 | 高 |
| JIT 编译 | 运行时热点代码编译 | 较高 |

---

## GC 垃圾回收

### GC 算法

ArkTS 使用 **HPP GC**（High Performance Partial Garbage Collection）：

```
GC 类型
├── Young GC（年轻代）
│   ├── 触发：年轻代空间满
│   ├── 算法：标记-复制
│   └── 耗时：约 1-2ms
├── Old GC（老年代）
│   ├── 触发：老年代空间阈值
│   ├── 算法：部分整理 + 部分清扫
│   └── 耗时：约 5-10ms
└── Full GC（全量）
    ├── 触发：后台切换/内存不足
    ├── 算法：全量压缩
    └── 耗时：较长
```

### 分代模型

```
堆内存结构
├── LocalHeap（线程私有）
│   ├── YoungSpace（年轻代）
│   │   ├── From Space
│   │   └── To Space
│   ├── OldSpace（老年代）
│   ├── HugeObjectSpace（大对象）
│   ├── NonMovableSpace（不可移动）
│   └── MachineCodeSpace（机器码）
└── SharedHeap（线程共享）
    ├── SharedOldSpace
    ├── SharedHugeObjectSpace
    └── SharedNonMovableSpace
```

### GC 触发策略

| 触发条件 | GC 类型 | 说明 |
|---------|---------|------|
| 年轻代空间满 | Young GC | 最频繁，耗时最短 |
| 老年代阈值 | Old GC | 频率较低 |
| 切换后台 | Full GC | 最大限度回收 |
| 空闲时 | Smart GC | 智能选择 GC 类型 |
| Native 内存压力 | Old GC | 绑定对象过多 |

### Smart GC

```typescript
// Smart GC 自动在以下场景优化：
// 1. 应用冷启动 - 临时提高阈值，避免启动时 GC
// 2. 列表滑动 - 抑制 GC，避免卡顿
// 3. 页面跳转 - 延迟 GC，保证动画流畅
// 4. 超长帧 - 检测到掉帧时调整策略

// 开发者无需手动配置，系统自动处理
```

---

## 内存管理

### 堆内存参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| HeapSize | 448MB | 主线程堆空间 |
| SemiSpaceSize | 2-16MB | 年轻代大小 |
| Worker HeapSize | 768MB | Worker 线程堆空间 |
| SharedHeapSize | 778MB | 共享堆空间 |

### 内存优化建议

```typescript
// 1. 避免内存泄漏
@Entry
@Component
struct MemorySafePage {
  private timer: number = -1;

  aboutToAppear(): void {
    this.timer = setInterval(() => {
      this.fetchData();
    }, 5000);
  }

  aboutToDisappear(): void {
    // ✅ 必须清理定时器
    clearInterval(this.timer);
  }

  async fetchData(): Promise<void> {
    // ...
  }

  build() { /* ... */ }
}
```

```typescript
// 2. 及时释放大对象
function processLargeData(): void {
  let largeBuffer = new ArrayBuffer(100 * 1024 * 1024); // 100MB
  
  try {
    // 处理数据
    processBuffer(largeBuffer);
  } finally {
    // ✅ 显式释放引用
    largeBuffer = null;
  }
}
```

```typescript
// 3. 使用 Sendable 减少内存拷贝
@Sendable
class SharedCache {
  data: collections.Map<string, string> = new collections.Map();
}

// 多个线程共享同一对象，避免重复内存
```

### 内存监控

```typescript
import { hidebug } from '@kit.PerformanceAnalysisKit';

// 获取内存信息
function logMemoryInfo(): void {
  const info = hidebug.getVMHeapMemoryInfo();
  console.info(`Heap used: ${info.usedSize} bytes`);
  console.info(`Heap total: ${info.totalSize} bytes`);
}

// 获取 GC 统计
function logGCStats(): void {
  const stats = hidebug.getGCStats();
  console.info(`GC count: ${stats.gcCount}`);
  console.info(`GC time: ${stats.gcTime} ms`);
}
```

---

## 性能优化

### 1. 减少 GC 压力

```typescript
// ❌ 频繁创建临时对象
function badPractice(): number {
  let sum = 0;
  for (let i = 0; i < 10000; i++) {
    const obj = { value: i };  // 每次循环创建对象
    sum += obj.value;
  }
  return sum;
}

// ✅ 复用对象
function goodPractice(): number {
  let sum = 0;
  let obj = { value: 0 };
  for (let i = 0; i < 10000; i++) {
    obj.value = i;  // 复用同一对象
    sum += obj.value;
  }
  return sum;
}
```

### 2. 对象池模式

```typescript
class ObjectPool<T> {
  private pool: T[] = [];
  private factory: () => T;
  private reset: (obj: T) => void;

  constructor(factory: () => T, reset: (obj: T) => void) {
    this.factory = factory;
    this.reset = reset;
  }

  acquire(): T {
    if (this.pool.length > 0) {
      return this.pool.pop()!;
    }
    return this.factory();
  }

  release(obj: T): void {
    this.reset(obj);
    this.pool.push(obj);
  }
}

// 使用
const particlePool = new ObjectPool(
  () => ({ x: 0, y: 0, velocity: 0 }),
  (p) => { p.x = 0; p.y = 0; p.velocity = 0; }
);
```

### 3. 避免内存碎片

```typescript
// ✅ 预分配数组容量
function processItems(items: number[]): number[] {
  const result = new Array<number>(items.length);  // 预分配
  for (let i = 0; i < items.length; i++) {
    result[i] = items[i] * 2;
  }
  return result;
}

// ❌ 动态扩容
function badProcess(items: number[]): number[] {
  const result: number[] = [];  // 初始容量 0
  for (const item of items) {
    result.push(item * 2);  // 多次扩容，产生碎片
  }
  return result;
}
```

### 4. 使用合适的数据结构

```typescript
// ✅ 查找频繁用 Map/Set
const userMap = new Map<string, User>();
userMap.set('user1', user);
const found = userMap.get('user1');  // O(1)

// ❌ 数组查找
const userList: User[] = [];
const found = userList.find(u => u.id === 'user1');  // O(n)
```

---

## 调试工具

### 1. GC 日志

```typescript
// 开启 GC 日志（调试模式）
// 在 hdc 中执行：
// hdc shell param set persist.ark.gc.log true

// 典型日志格式：
// [ HPP YoungGC ] [ 1.2ms ] [ Reason: ALLOCATION_LIMIT ]
// [ HPP OldGC ] [ 8.5ms ] [ Reason: IDLE ]
```

### 2. Heap Snapshot

```typescript
// 使用 DevEco Studio 的 Profiler 工具
// 1. 连接设备
// 2. 选择应用进程
// 3. 点击 "Heap Snapshot"
// 4. 分析对象引用链
```

### 3. 性能分析

```typescript
import { hiTraceMeter } from '@kit.PerformanceAnalysisKit';

// 标记性能区间
function processData(): void {
  hiTraceMeter.startTrace('processData');
  
  try {
    // 耗时操作
    heavyComputation();
  } finally {
    hiTraceMeter.finishTrace('processData');
  }
}
```

---

## 常见问题

### Q1: 应用内存溢出（OOM）？

**A:** 
1. 检查是否有内存泄漏（未释放的监听器、定时器）
2. 减少大对象同时存活
3. 使用对象池复用对象
4. 考虑使用 Native 层处理大数据

### Q2: GC 导致卡顿？

**A:**
1. 避免在动画/滑动时创建大量对象
2. 使用对象池减少分配
3. 将大任务移到 TaskPool/Worker
4. 检查是否有内存泄漏导致频繁 GC

### Q3: 如何查看 GC 频率？

**A:**
```typescript
// 通过日志分析
// 或使用 hidebug 接口
const stats = hidebug.getGCStats();
console.info(`Young GC: ${stats.youngGCCount}`);
console.info(`Old GC: ${stats.oldGCCount}`);
```

### Q4: SharedHeap 和 LocalHeap 区别？

**A:**
- LocalHeap：每个线程私有，GC 独立
- SharedHeap：所有线程共享，存储 Sendable 对象
- SharedHeap 不能引用 LocalHeap 对象

---

## 📚 延伸阅读

- [ArkTS 运行时概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-runtime-overview)
- [GC 垃圾回收](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/gc-introduction)
- [性能优化最佳实践](https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-performance-optimization)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
