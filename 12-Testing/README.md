# 测试调试

> 🧪 应用测试与调试工具

## 📖 目录

1. [测试概述](#测试概述)
2. [单元测试](#单元测试)
3. [UI 测试](#ui-测试)
4. [性能调试](#性能调试)
5. [调试工具](#调试工具)
6. [测试最佳实践](#测试最佳实践)
7. [常见问题](#常见问题)

---

## 测试概述

HarmonyOS 提供完整的测试框架，支持单元测试、UI 测试和性能测试。

### 测试类型

```
测试框架
├── 单元测试
│   ├── 函数测试
│   ├── 类测试
│   └── 模块测试
├── UI 测试
│   ├── 组件测试
│   ├── 页面测试
│   └── 交互测试
└── 性能测试
    ├── 启动时间
    ├── 内存使用
    └── 帧率监控
```

---

## 单元测试

### 基础测试

```typescript
// test/utils.test.ets
import { describe, it, expect } from '@ohos/hypium';

export default function utilsTest() {
  describe('UtilsTest', () => {
    it('add', 0, () => {
      const result = add(1, 2);
      expect(result).assertEqual(3);
    });

    it('subtract', 0, () => {
      const result = subtract(5, 3);
      expect(result).assertEqual(2);
    });
  });
}
```

### 断言方法

```typescript
// 相等断言
expect(value).assertEqual(expected);
expect(value).assertNotEqual(unexpected);

// 类型断言
expect(value).assertInstanceOf(Type);

// 空值断言
expect(value).assertNull();
expect(value).assertNotNull();
expect(value).assertUndefined();
expect(value).assertNotUndefined();

// 布尔断言
expect(value).assertTrue();
expect(value).assertFalse();

// 数组断言
expect(array).assertContain(item);
expect(array).assertContainAll(items);

// 异常断言
expect(() => {
  throw new Error('test');
}).assertThrowError();
```

### 测试工具函数

```typescript
// 测试辅助函数
class TestUtils {
  static createMockUser(): User {
    return {
      id: 1,
      name: 'Test User',
      email: 'test@example.com'
    };
  }

  static async waitFor(predicate: () => boolean, timeout: number = 1000): Promise<void> {
    const start = Date.now();
    while (!predicate()) {
      if (Date.now() - start > timeout) {
        throw new Error('Timeout');
      }
      await new Promise(resolve => setTimeout(resolve, 10));
    }
  }
}
```

### Mock 和 Stub

```typescript
// Mock 函数
class MockFunction {
  private calls: any[][] = [];
  private returnValue: any;

  setReturnValue(value: any): void {
    this.returnValue = value;
  }

  call(...args: any[]): any {
    this.calls.push(args);
    return this.returnValue;
  }

  getCallCount(): number {
    return this.calls.length;
  }

  getLastCall(): any[] | undefined {
    return this.calls[this.calls.length - 1];
  }

  reset(): void {
    this.calls = [];
  }
}

// 使用 Mock
const mockApi = new MockFunction();
mockApi.setReturnValue({ success: true });

// 验证调用
expect(mockApi.getCallCount()).assertEqual(1);
expect(mockApi.getLastCall()).assertContain('test');
```

---

## UI 测试

### 组件测试

```typescript
import { describe, it, expect } from '@ohos/hypium';
import { UIContext } from '@ohos.arkui';

export default function componentTest() {
  describe('ComponentTest', () => {
    let uiContext: UIContext;

    beforeAll(async () => {
      // 初始化 UI 上下文
    });

    it('should render text', 0, async () => {
      // 渲染组件
      const component = await TestContext.setUp('pages/Test');
      
      // 查找元素
      const text = component.findComponentByText('Hello');
      
      // 断言
      expect(text).assertNotNull();
    });

    it('should handle click', 0, async () => {
      const component = await TestContext.setUp('pages/Test');
      const button = component.findComponentById('btn');
      
      // 模拟点击
      button.click();
      
      // 验证结果
      const result = component.findComponentByText('Clicked');
      expect(result).assertNotNull();
    });
  });
}
```

### 页面测试

```typescript
export default function pageTest() {
  describe('PageTest', () => {
    it('should navigate to detail', 0, async () => {
      const page = await TestContext.setUp('pages/Home');
      
      // 点击列表项
      const item = page.findComponentById('item-1');
      item.click();
      
      // 验证跳转
      await TestContext.waitForPage('pages/Detail');
      const detailPage = TestContext.getCurrentPage();
      const title = detailPage.findComponentById('title');
      expect(title.getText()).assertEqual('Item 1');
    });
  });
}
```

---

## 性能调试

### 启动时间监控

```typescript
import { hiTraceMeter } from '@ohos.hiTraceMeter';

// 标记开始
hiTraceMeter.startTrace('app-startup', 1);

// 执行代码
// ...

// 标记结束
hiTraceMeter.finishTrace('app-startup', 1);
```

### 内存监控

```typescript
import { memory } from '@ohos.base';

// 获取内存信息
function getMemoryInfo(): object {
  const memInfo = memory.getMemoryInfo();
  return {
    totalMem: memInfo.totalMem,
    freeMem: memInfo.freeMem,
    usedMem: memInfo.totalMem - memInfo.freeMem
  };
}

// 监控内存泄漏
function watchMemoryLeak(): void {
  setInterval(() => {
    const info = getMemoryInfo();
    console.log('Memory usage:', info.usedMem);
    
    if (info.usedMem > THRESHOLD) {
      console.warn('Possible memory leak detected');
    }
  }, 5000);
}
```

### 帧率监控

```typescript
import { observer } from '@ohos.application';

// 监听帧率
function watchFrameRate(): void {
  observer.on('frameRateChange', (frameRate: number) => {
    if (frameRate < 30) {
      console.warn('Low frame rate detected:', frameRate);
    }
  });
}
```

---

## 调试工具

### 日志工具

```typescript
import { hilog } from '@ohos.hilog';

class Logger {
  private static DOMAIN = 0x0001;
  private static TAG = 'MyApp';

  static debug(message: string, ...args: any[]): void {
    hilog.debug(Logger.DOMAIN, Logger.TAG, message, args);
  }

  static info(message: string, ...args: any[]): void {
    hilog.info(Logger.DOMAIN, Logger.TAG, message, args);
  }

  static warn(message: string, ...args: any[]): void {
    hilog.warn(Logger.DOMAIN, Logger.TAG, message, args);
  }

  static error(message: string, ...args: any[]): void {
    hilog.error(Logger.DOMAIN, Logger.TAG, message, args);
  }
}

// 使用
Logger.info('User logged in: %{public}s', username);
Logger.error('Request failed: %{public}s', error.message);
```

### 调试开关

```typescript
class DebugConfig {
  private static isDebug: boolean = true;

  static enable(): void {
    this.isDebug = true;
  }

  static disable(): void {
    this.isDebug = false;
  }

  static log(message: string): void {
    if (this.isDebug) {
      console.log(`[DEBUG] ${message}`);
    }
  }

  static assert(condition: boolean, message: string): void {
    if (this.isDebug && !condition) {
      throw new Error(`Assertion failed: ${message}`);
    }
  }
}
```

### 错误边界

```typescript
@Component
struct ErrorBoundary {
  @State hasError: boolean = false;
  @State error: Error | null = null;

  aboutToAppear() {
    // 注册全局错误处理
    globalThis.onerror = (message: string, source: string, lineno: number, colno: number, error: Error) => {
      this.hasError = true;
      this.error = error;
      Logger.error('Uncaught error:', error);
    };
  }

  build() {
    if (this.hasError) {
      Column() {
        Text('Something went wrong')
          .fontSize(20)
        Text(this.error?.message || '')
          .fontSize(14)
          .fontColor('#999')
        Button('Retry')
          .onClick(() => {
            this.hasError = false;
            this.error = null;
          })
      }
      .padding(20)
    } else {
      // 正常内容
      slot();
    }
  }
}
```

---

## 测试最佳实践

### 1. 测试覆盖率

```typescript
// 确保关键路径被测试
describe('UserService', () => {
  // 正常流程
  it('should create user successfully', 0, () => { });
  
  // 异常流程
  it('should handle duplicate email', 0, () => { });
  
  // 边界条件
  it('should handle empty input', 0, () => { });
  
  // 并发场景
  it('should handle concurrent requests', 0, () => { });
});
```

### 2. 测试命名规范

```typescript
// ✅ 描述性命名
it('should return user when valid id is provided', 0, () => { });
it('should throw error when user not found', 0, () => { });

// ❌ 模糊命名
it('test1', 0, () => { });
it('user test', 0, () => { });
```

### 3. 测试隔离

```typescript
describe('TestSuite', () => {
  beforeEach(() => {
    // 每个测试前重置状态
    resetDatabase();
    clearCache();
  });

  afterEach(() => {
    // 清理资源
    closeConnections();
  });

  it('test1', 0, () => { });
  it('test2', 0, () => { });
});
```

### 4. 持续集成

```yaml
# build-test.yml
name: Test Pipeline
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Tests
        run: hdc shell aa test -b com.example.app -s TestRunner
```

---

## 常见问题

### Q1: 测试超时怎么办？

**A:** 增加超时时间或使用异步测试：
```typescript
it('async test', 0, async () => {
  const result = await asyncOperation();
  expect(result).assertNotNull();
}, 5000); // 5秒超时
```

### Q2: 如何测试 UI 组件？

**A:** 使用 UI 测试框架：
```typescript
it('should render button', 0, async () => {
  const page = await TestContext.setUp('pages/Test');
  const button = page.findComponentById('btn');
  expect(button).assertNotNull();
  expect(button.getText()).assertEqual('Click Me');
});
```

### Q3: 如何模拟网络请求？

**A:** 使用 Mock：
```typescript
const mockHttp = {
  get: (url: string) => Promise.resolve({ data: 'mock' })
};

// 注入 Mock
const service = new UserService(mockHttp);
```

---

## 📚 延伸阅读

- [测试框架](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-test-framework)
- [单元测试](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-unit-test)
- [UI 测试](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-ui-test)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
