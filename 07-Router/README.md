# 路由管理

> 🧭 页面导航与路由系统

## 📖 目录

1. [路由概述](#路由概述)
2. [页面路由](#页面路由)
3. [Navigation 导航](#navigation-导航)
4. [参数传递](#参数传递)
5. [路由拦截](#路由拦截)
6. [路由最佳实践](#路由最佳实践)
7. [常见问题](#常见问题)

---

## 路由概述

HarmonyOS 提供两种导航方式：页面路由 (router) 和 Navigation 组件。

### 导航方式对比

| 特性 | 页面路由 | Navigation |
|------|---------|------------|
| 适用场景 | 全屏页面切换 | 应用内导航 |
| 栈管理 | 自动 | 自定义 |
| 转场动画 | 支持 | 支持 |
| 参数传递 | 支持 | 支持 |
| 拦截器 | 不支持 | 支持 |

---

## 页面路由

### 基础用法

```typescript
import { router } from '@kit.ArkUI';

// 跳转到新页面
router.pushUrl({
  url: 'pages/Detail',
  params: { id: '123' }
});

// 替换当前页面
router.replaceUrl({
  url: 'pages/Detail'
});

// 返回上一页
router.back();

// 返回指定页面
router.back({
  url: 'pages/Home'
});
```

### 路由参数

```typescript
// 传递参数
router.pushUrl({
  url: 'pages/Detail',
  params: {
    id: '123',
    title: 'Hello'
  }
});

// 接收参数
@Entry
@Component
struct DetailPage {
  @State id: string = '';
  @State title: string = '';

  aboutToAppear() {
    const params = router.getParams() as Record<string, string>;
    this.id = params.id;
    this.title = params.title;
  }

  build() {
    Column() {
      Text(`ID: ${this.id}`)
      Text(`Title: ${this.title}`)
    }
  }
}
```

---

## Navigation 导航

### 基础用法

```typescript
// 定义路由配置
const routeMap: Map<string, Builder> = new Map([
  ['home', () => HomePage()],
  ['detail', () => DetailPage()],
  ['settings', () => SettingsPage()]
]);

@Entry
@Component
struct MainPage {
  @State currentIndex: number = 0;

  build() {
    Navigation() {
      // 首页内容
      HomePage()
    }
    .navDestination(routeMap)
    .mode(NavigationMode.Stack)
    .title('My App')
  }
}
```

### 页面跳转

```typescript
import { router } from '@kit.ArkUI';

@Component
struct HomePage {
  build() {
    Column() {
      Button('Go to Detail')
        .onClick(() => {
          router.pushUrl({
            url: 'pages/Detail'
          })
        })
    }
  }
}
```

### Navigation + Tabs

```typescript
@Entry
@Component
struct MainPage {
  @State currentIndex: number = 0;

  build() {
    Tabs({ index: this.currentIndex }) {
      TabContent() {
        Navigation() {
          HomePage()
        }
        .title('Home')
      }
      .tabBar(this.buildTab('Home', 0))

      TabContent() {
        Navigation() {
          DiscoverPage()
        }
        .title('Discover')
      }
      .tabBar(this.buildTab('Discover', 1))

      TabContent() {
        Navigation() {
          ProfilePage()
        }
        .title('Profile')
      }
      .tabBar(this.buildTab('Profile', 2))
    }
  }

  @Builder
  buildTab(title: string, index: number) {
    Column() {
      Text(title)
        .fontColor(this.currentIndex === index ? '#007DFF' : '#999')
    }
  }
}
```

---

## 参数传递

### 简单参数

```typescript
// 传递
router.pushUrl({
  url: 'pages/Detail',
  params: { id: '123', name: 'Hello' }
});

// 接收
const params = router.getParams() as Record<string, string>;
```

### 对象参数

```typescript
interface UserInfo {
  id: string;
  name: string;
  avatar: string;
}

// 传递
const user: UserInfo = {
  id: '123',
  name: 'Alice',
  avatar: 'https://example.com/avatar.jpg'
};
router.pushUrl({
  url: 'pages/Profile',
  params: user
});

// 接收
const user = router.getParams() as UserInfo;
```

### 回调参数

```typescript
// 传递回调
router.pushUrl({
  url: 'pages/Select',
  params: {
    onSelect: (result: string) => {
      this.selectedValue = result;
    }
  }
});

// 调用回调
const params = router.getParams() as Record<string, Function>;
params.onSelect('selected value');
```

---

## 路由拦截

### Navigation 拦截

```typescript
@Entry
@Component
struct MainPage {
  build() {
    Navigation() {
      // 内容
    }
    .onWillShow((event: NavigationWillShowEvent) => {
      // 拦截导航
      if (!this.isLoggedIn) {
        event.preventDefault();
        router.pushUrl({ url: 'pages/Login' });
      }
    })
  }
}
```

### 自定义拦截器

```typescript
class RouterInterceptor {
  private static interceptors: Array<(url: string) => boolean> = [];

  static addInterceptor(interceptor: (url: string) => boolean) {
    this.interceptors.push(interceptor);
  }

  static canNavigate(url: string): boolean {
    return this.interceptors.every(interceptor => interceptor(url));
  }
}

// 注册拦截器
RouterInterceptor.addInterceptor((url: string) => {
  if (url.startsWith('pages/admin') && !isAdmin()) {
    return false;
  }
  return true;
});

// 使用
if (RouterInterceptor.canNavigate('pages/admin/settings')) {
  router.pushUrl({ url: 'pages/admin/settings' });
}
```

---

## 路由最佳实践

### 1. 路由常量

```typescript
// routes.ets
export const ROUTES = {
  HOME: 'pages/Home',
  DETAIL: 'pages/Detail',
  SETTINGS: 'pages/Settings',
  LOGIN: 'pages/Login'
} as const;

// 使用
router.pushUrl({ url: ROUTES.DETAIL });
```

### 2. 路由工具类

```typescript
class RouterUtil {
  static push(url: string, params?: Record<string, Object>): void {
    router.pushUrl({ url, params });
  }

  static replace(url: string, params?: Record<string, Object>): void {
    router.replaceUrl({ url, params });
  }

  static back(): void {
    router.back();
  }

  static getParams<T>(): T {
    return router.getParams() as T;
  }
}

// 使用
RouterUtil.push(ROUTES.DETAIL, { id: '123' });
```

### 3. 页面栈管理

```typescript
class StackManager {
  static clearStack(): void {
    router.clear();
  }

  static getStackSize(): number {
    return router.getLength();
  }

  static canGoBack(): boolean {
    return router.getLength() > 1;
  }
}
```

---

## 常见问题

### Q1: 如何实现页面返回刷新？

**A:** 使用 EventEmitter 或 AppStorage：
```typescript
// 页面 A
EventEmitter.on('refresh', () => {
  this.loadData();
});

// 页面 B
router.back();
EventEmitter.emit('refresh');
```

### Q2: 如何处理页面栈溢出？

**A:** 使用 replace 代替 push：
```typescript
// 避免无限 push
router.replaceUrl({ url: 'pages/NewPage' });
```

### Q3: 如何实现深度链接？

**A:** 通过 Want 参数处理：
```typescript
onCreate(want: Want) {
  if (want.uri) {
    // 解析 URI 并导航
    this.handleDeepLink(want.uri);
  }
}
```

---

## 📚 延伸阅读

- [页面路由](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-page-router)
- [Navigation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-navigation)
- [页面传参](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-page-passing-params)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
