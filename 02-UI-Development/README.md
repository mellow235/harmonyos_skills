# UI 开发

> 🎨 ArkUI 声明式开发框架

## 📖 目录

1. [ArkUI 框架概述](#arkui-框架概述)
2. [页面结构](#页面结构)
3. [组件基础](#组件基础)
4. [样式设置](#样式设置)
5. [事件处理](#事件处理)
6. [自定义组件](#自定义组件)
7. [多设备适配](#多设备适配)
8. [常见问题](#常见问题)
9. [最佳实践](#最佳实践)

---

## ArkUI 框架概述

ArkUI 是 HarmonyOS 的 UI 开发框架，提供声明式和类 Web 两种开发范式。

### 核心特点

| 特性 | 说明 |
|------|------|
| 声明式 UI | 使用 ArkTS 语言描述 UI |
| 组件化 | 丰富的内置组件 |
| 状态驱动 | 数据变化自动更新 UI |
| 响应式 | 自适应不同屏幕尺寸 |
| 高性能 | 渲染优化 |

### 开发范式对比

```
┌─────────────────────────────────────────────────────────┐
│                    ArkUI 开发范式                         │
├─────────────────────┬───────────────────────────────────┤
│    声明式范式         │         类 Web 范式                │
│  (ArkTS + 声明式UI)   │    (JS/TS + HTML + CSS)          │
├─────────────────────┼───────────────────────────────────┤
│  • TypeScript 语法    │  • JS/TS 语法                     │
│  • 声明式组件         │  • HTML 标签                      │
│  • 链式 API          │  • CSS 样式                       │
│  • 状态管理装饰器     │  • 数据绑定                       │
└─────────────────────┴───────────────────────────────────┘
```

---

## 页面结构

### 入口页面

```typescript
// pages/Index.ets
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS';

  build() {
    Column() {
      Text(this.message)
        .fontSize(30)
        .fontWeight(FontWeight.Bold)
      
      Button('Click Me')
        .onClick(() => {
          this.message = 'Clicked!';
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### 组件生命周期

```typescript
@Component
struct LifecycleDemo {
  @State count: number = 0;

  // 组件创建时
  aboutToAppear(): void {
    console.log('Component will appear');
    // 初始化数据、注册监听等
  }

  // 组件销毁时
  aboutToDisappear(): void {
    console.log('Component will disappear');
    // 清理资源、取消监听等
  }

  build() {
    Column() {
      Text(`Count: ${this.count}`)
      Button('Add')
        .onClick(() => this.count++)
    }
  }
}
```

### 页面生命周期

```typescript
@Entry
@Component
struct PageLifecycle {
  // 页面显示时
  onShow(): void {
    console.log('Page shown');
  }

  // 页面隐藏时
  onHide(): void {
    console.log('Page hidden');
  }

  // 页面返回时
  onBackPress(): boolean {
    console.log('Back pressed');
    return false; // 返回 true 拦截返回
  }

  build() {
    Column() {
      Text('Page Lifecycle Demo')
    }
  }
}
```

---

## 组件基础

### 内置组件分类

```
组件库
├── 基础组件
│   ├── Text - 文本
│   ├── Image - 图片
│   ├── Button - 按钮
│   ├── TextInput - 输入框
│   └── LoadingProgress - 加载
├── 容器组件
│   ├── Column - 纵向布局
│   ├── Row - 横向布局
│   ├── Stack - 层叠布局
│   ├── Flex - 弹性布局
│   └── Grid - 网格布局
├── 媒体组件
│   ├── Image - 图片
│   ├── Video - 视频
│   └── XComponent - 渲染组件
└── 动画组件
    ├── Animator - 属性动画
    └── Transition - 转场动画
```

### Text 组件

```typescript
// 基础文本
Text('Hello World')

// 带样式的文本
Text('Styled Text')
  .fontSize(20)
  .fontWeight(FontWeight.Bold)
  .fontColor('#333333')
  .textAlign(TextAlign.Center)
  .maxLines(2)
  .textOverflow({ overflow: TextOverflow.Ellipsis })

// 富文本
Text() {
  Span('Hello ')
    .fontSize(16)
    .fontColor('#000')
  Span('HarmonyOS')
    .fontSize(20)
    .fontColor('#007DFF')
    .fontWeight(FontWeight.Bold)
}
```

### Image 组件

```typescript
// 网络图片
Image('https://example.com/image.jpg')
  .width(200)
  .height(200)
  .objectFit(ImageFit.Cover)
  .borderRadius(10)

// 本地图片
Image($r('app.media.icon'))
  .width(100)
  .height(100)

// 图片加载状态
Image('https://example.com/image.jpg')
  .onComplete(() => {
    console.log('Image loaded');
  })
  .onError(() => {
    console.log('Image load failed');
  })
```

### Button 组件

```typescript
// 基础按钮
Button('Click Me')
  .onClick(() => {
    console.log('Clicked');
  })

// 带样式的按钮
Button('Submit')
  .width(200)
  .height(50)
  .backgroundColor('#007DFF')
  .fontColor('#FFFFFF')
  .fontSize(16)
  .borderRadius(25)
  .type(ButtonType.Capsule)

// 图标按钮
Button() {
  Image($r('app.media.icon'))
    .width(24)
    .height(24)
  Text('Button')
    .fontSize(16)
    .margin({ left: 8 })
}
```

### TextInput 组件

```typescript
// 基础输入框
TextInput({ placeholder: 'Enter text' })
  .width(300)
  .height(50)
  .onChange((value: string) => {
    console.log('Input:', value);
  })

// 密码输入框
TextInput({ placeholder: 'Password' })
  .type(InputType.Password)
  .width(300)
  .height(50)

// 带图标的输入框
TextInput({ placeholder: 'Search' })
  .width(300)
  .height(40)
  .backgroundColor('#F5F5F5')
  .borderRadius(20)
  .padding({ left: 16 })
```

---

## 样式设置

### 尺寸设置

```typescript
Text('Hello')
  .width(100)           // 固定宽度
  .height(50)           // 固定高度
  .width('50%')         // 百分比宽度
  .height('100%')       // 百分比高度
  .minWidth(50)         // 最小宽度
  .maxWidth(200)        // 最大宽度
```

### 边距和内边距

```typescript
Text('Hello')
  .margin(10)                    // 四周相同
  .margin({ top: 10, left: 20 }) // 分别设置
  .padding(8)                    // 四周相同
  .padding({ left: 16, right: 16 }) // 分别设置
```

### 背景和边框

```typescript
Text('Hello')
  .backgroundColor('#F5F5F5')     // 背景色
  .borderRadius(8)                // 圆角
  .border({                       // 边框
    width: 1,
    color: '#CCCCCC',
    style: BorderStyle.Solid
  })
  .shadow({                       // 阴影
    radius: 10,
    color: 'rgba(0,0,0,0.2)',
    offsetX: 2,
    offsetY: 2
  })
```

### 渐变背景

```typescript
Text('Gradient')
  .width(200)
  .height(50)
  .linearGradient({
    direction: GradientDirection.Right,
    colors: [['#FF6B6B', 0.0], ['#4ECDC4', 1.0]]
  })

// 径向渐变
Text('Radial')
  .width(200)
  .height(200)
  .radialGradient({
    center: [100, 100],
    radius: 100,
    colors: [['#FF6B6B', 0.0], ['#4ECDC4', 1.0]]
  })
```

---

## 事件处理

### 点击事件

```typescript
Button('Click')
  .onClick(() => {
    console.log('Button clicked');
  })

Text('Clickable')
  .onClick((event: ClickEvent) => {
    console.log('Clicked at:', event.x, event.y);
  })
```

### 触摸事件

```typescript
Text('Touchable')
  .onTouch((event: TouchEvent) => {
    switch (event.type) {
      case TouchType.Down:
        console.log('Touch down');
        break;
      case TouchType.Move:
        console.log('Touch move');
        break;
      case TouchType.Up:
        console.log('Touch up');
        break;
    }
  })
```

### 手势事件

```typescript
// 点击手势
Text('Tap')
  .gesture(
    TapGesture()
      .onAction(() => {
        console.log('Tapped');
      })
  )

// 长按手势
Text('Long Press')
  .gesture(
    LongPressGesture({ repeat: false })
      .onAction(() => {
        console.log('Long pressed');
      })
  )

// 拖拽手势
Text('Pan')
  .gesture(
    PanGesture()
      .onActionStart(() => {
        console.log('Pan start');
      })
      .onActionUpdate((event: GestureEvent) => {
        console.log('Pan update:', event.offsetX, event.offsetY);
      })
      .onActionEnd(() => {
        console.log('Pan end');
      })
  )
```

---

## 自定义组件

### 基础自定义组件

```typescript
@Component
struct CustomButton {
  @Prop label: string = '';
  @Prop color: string = '#007DFF';
  @Link isPressed: boolean;

  build() {
    Button(this.label)
      .backgroundColor(this.color)
      .fontColor('#FFFFFF')
      .borderRadius(8)
      .onClick(() => {
        this.isPressed = true;
      })
  }
}

// 使用
@Component
struct Parent {
  @State pressed: boolean = false;

  build() {
    Column() {
      CustomButton({
        label: 'Click Me',
        color: '#FF6B6B',
        isPressed: $pressed
      })
    }
  }
}
```

### @Builder 装饰器

```typescript
@Component
struct BuilderDemo {
  @State items: string[] = ['Apple', 'Banana', 'Cherry'];

  @Builder
  ItemCard(item: string) {
    Row() {
      Image($r('app.media.fruit'))
        .width(40)
        .height(40)
      Text(item)
        .fontSize(16)
        .margin({ left: 12 })
    }
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .margin({ bottom: 8 })
  }

  build() {
    Column() {
      ForEach(this.items, (item: string) => {
        this.ItemCard(item);
      })
    }
  }
}
```

### @Styles 装饰器

```typescript
@Styles
function cardStyle() {
  .padding(16)
  .backgroundColor('#FFFFFF')
  .borderRadius(12)
  .shadow({
    radius: 8,
    color: 'rgba(0,0,0,0.1)',
    offsetX: 0,
    offsetY: 2
  })
}

@Component
struct StyleDemo {
  build() {
    Column() {
      Text('Card 1')
        .cardStyle()
      Text('Card 2')
        .cardStyle()
    }
  }
}
```

---

## 多设备适配

### 断点系统

```typescript
// 定义断点
enum Breakpoint {
  SM = 'sm',   // < 600vp
  MD = 'md',   // 600vp - 840vp
  LG = 'lg'    // > 840vp
}

@Component
struct ResponsiveLayout {
  @StorageProp('breakpoint') breakpoint: Breakpoint = Breakpoint.SM;

  build() {
    Column() {
      if (this.breakpoint === Breakpoint.SM) {
        this.buildMobileLayout()
      } else if (this.breakpoint === Breakpoint.MD) {
        this.buildTabletLayout()
      } else {
        this.buildDesktopLayout()
      }
    }
  }

  @Builder
  buildMobileLayout() {
    Column() {
      // 手机布局
    }
  }

  @Builder
  buildTabletLayout() {
    Row() {
      // 平板布局
    }
  }

  @Builder
  buildDesktopLayout() {
    Grid() {
      // 桌面布局
    }
  }
}
```

### 媒体查询

```typescript
@Component
struct MediaQueryDemo {
  @State isMobile: boolean = true;

  build() {
    Column() {
      Text(this.isMobile ? 'Mobile' : 'Desktop')
    }
    .onBreakpointChange((breakpoint: string) => {
      this.isMobile = breakpoint === 'sm';
    })
  }
}
```

---

## 常见问题

### Q1: 如何实现列表滚动？

**A:** 使用 `List` 或 `Scroll` 组件：
```typescript
List() {
  ForEach(this.items, (item: string) => {
    ListItem() {
      Text(item)
    }
  })
}
.scrollBar(BarState.Off)
.edgeEffect(EdgeEffect.Spring)
```

### Q2: 如何处理键盘弹出？

**A:** 使用 `expandSafeArea` 或监听键盘事件：
```typescript
Column() {
  // 内容
}
.expandSafeArea([SafeAreaType.SYSTEM])
```

### Q3: 如何优化渲染性能？

**A:** 
- 使用 `LazyForEach` 替代 `ForEach`
- 避免在 build 中创建对象
- 使用 `@Watch` 监听状态变化

---

## 最佳实践

### 1. 组件职责单一
```typescript
// ✅ 单一职责
@Component
struct UserAvatar {
  @Prop user: User;
  build() {
    Image(this.user.avatar)
      .width(48)
      .height(48)
      .borderRadius(24)
  }
}

// ❌ 职责混乱
@Component
struct UserProfile {
  @Prop user: User;
  build() {
    Column() {
      // 头像 + 名字 + 简介 + 按钮...
    }
  }
}
```

### 2. 使用常量
```typescript
// ✅ 定义常量
const COLORS = {
  primary: '#007DFF',
  secondary: '#6C757D',
  success: '#28A745'
} as const;

// ❌ 魔法字符串
Text('Hello')
  .fontColor('#007DFF')
```

### 3. 合理使用状态
```typescript
// ✅ 最小化状态
@Component
struct Counter {
  @State count: number = 0;  // 只有需要的状态
}

// ❌ 过度状态
@Component
struct Counter {
  @State count: number = 0;
  @State unused: string = '';  // 未使用的状态
}
```

---

## 📚 延伸阅读

- [ArkUI 官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-overview)
- [组件参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts-components)
- [动画指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-animation)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
