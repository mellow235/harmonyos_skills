# 组件应用

> 🧩 丰富的内置组件库

## 📖 目录

1. [组件分类](#组件分类)
2. [基础组件](#基础组件)
3. [容器组件](#容器组件)
4. [媒体组件](#媒体组件)
5. [导航组件](#导航组件)
6. [弹窗组件](#弹窗组件)
7. [自定义组件](#自定义组件)
8. [常见问题](#常见问题)

---

## 组件分类

```
组件库
├── 基础组件
│   ├── Text - 文本
│   ├── Span - 行内文本
│   ├── Image - 图片
│   ├── Button - 按钮
│   ├── TextInput - 输入框
│   ├── Textarea - 多行输入
│   ├── Toggle - 开关
│   ├── Slider - 滑动条
│   ├── Progress - 进度条
│   └── LoadingProgress - 加载
├── 容器组件
│   ├── Column - 纵向布局
│   ├── Row - 横向布局
│   ├── Stack - 层叠布局
│   ├── Flex - 弹性布局
│   ├── Grid - 网格布局
│   ├── List - 列表
│   ├── Scroll - 滚动
│   ├── Tabs - 标签页
│   └── Swiper - 轮播
├── 媒体组件
│   ├── Image - 图片
│   ├── Video - 视频
│   └── XComponent - 渲染
├── 导航组件
│   ├── Navigation - 导航
│   ├── NavRouter - 路由
│   └── TabContent - 标签内容
└── 弹窗组件
    ├── AlertDialog - 警告弹窗
    ├── Dialog - 自定义弹窗
    ├── Toast - 提示
    └── ActionSheet - 操作菜单
```

---

## 基础组件

### Text - 文本组件

```typescript
// 基础用法
Text('Hello World')

// 完整配置
Text('Styled Text')
  .fontSize(20)
  .fontWeight(FontWeight.Bold)
  .fontColor('#333333')
  .fontFamily('HarmonyOS Sans')
  .textAlign(TextAlign.Center)
  .textDecoration({ type: TextDecorationType.Underline })
  .textOverflow({ overflow: TextOverflow.Ellipsis })
  .maxLines(2)
  .lineHeight(28)
  .letterSpacing(1)

// 富文本
Text() {
  Span('Hello ')
    .fontSize(16)
  Span('HarmonyOS')
    .fontSize(20)
    .fontWeight(FontWeight.Bold)
    .fontColor('#007DFF')
    .onClick(() => {
      console.log('Clicked');
    })
}
```

### Image - 图片组件

```typescript
// 网络图片
Image('https://example.com/image.jpg')
  .width(200)
  .height(200)
  .objectFit(ImageFit.Cover)
  .borderRadius(10)
  .alt($r('app.media.placeholder'))

// 本地图片
Image($r('app.media.logo'))
  .width(100)
  .height(100)

// SVG 图片
Image($r('app.media.icon.svg'))
  .width(24)
  .height(24)
  .fillColor('#007DFF')

// 图片加载状态
Image('https://example.com/image.jpg')
  .onComplete(() => {
    console.log('Load complete');
  })
  .onError(() => {
    console.log('Load error');
  })
  .onFinish(() => {
    console.log('Decode finish');
  })
```

### Button - 按钮组件

```typescript
// 基础按钮
Button('Click Me')
  .onClick(() => {
    console.log('Clicked');
  })

// 按钮类型
Button('Normal')
  .type(ButtonType.Normal)
  .width(200)
  .height(50)

Button('Capsule')
  .type(ButtonType.Capsule)
  .width(200)
  .height(50)

Button('Circle')
  .type(ButtonType.Circle)
  .width(60)
  .height(60)

// 带图标按钮
Button() {
  Image($r('app.media.icon'))
    .width(20)
    .height(20)
  Text('Button')
    .fontSize(16)
    .margin({ left: 8 })
}
.buttonStyle()
```

### TextInput - 输入组件

```typescript
// 基础输入框
TextInput({ placeholder: 'Enter text' })
  .width(300)
  .height(50)
  .onChange((value: string) => {
    console.log('Value:', value);
  })

// 类型设置
TextInput({ placeholder: 'Password' })
  .type(InputType.Password)

TextInput({ placeholder: 'Email' })
  .type(InputType.Email)

TextInput({ placeholder: 'Number' })
  .type(InputType.Number)

// 带图标
TextInput({ placeholder: 'Search' })
  .width(300)
  .height(40)
  .backgroundColor('#F5F5F5')
  .borderRadius(20)
  .prefixIcon($r('app.media.search'))
  .suffixIcon($r('app.media.clear'))
```

### Toggle - 开关组件

```typescript
// 基础开关
Toggle({ type: ToggleType.Switch, isOn: false })
  .onChange((isOn: boolean) => {
    console.log('Toggle:', isOn);
  })

// 带标签
Row() {
  Text('Enable Notifications')
  Blank()
  Toggle({ type: ToggleType.Switch, isOn: this.isEnabled })
    .onChange((isOn: boolean) => {
      this.isEnabled = isOn;
    })
}
.width('100%')
.padding(16)
```

### Slider - 滑动条

```typescript
// 基础滑动条
Slider({ value: 50, min: 0, max: 100, step: 1 })
  .width('100%')
  .onChange((value: number) => {
    console.log('Value:', value);
  })

// 带样式
Slider({ value: this.volume, min: 0, max: 100 })
  .width('80%')
  .blockColor('#007DFF')
  .trackColor('#E5E5E5')
  .selectedColor('#007DFF')
  .showSteps(true)
  .showTips(true)
```

### Progress - 进度条

```typescript
// 线性进度条
Progress({ value: 60, total: 100, type: ProgressType.Linear })
  .width('100%')
  .height(8)
  .color('#007DFF')
  .backgroundColor('#E5E5E5')

// 环形进度条
Progress({ value: 75, total: 100, type: ProgressType.Ring })
  .width(100)
  .height(100)
  .color('#007DFF')

// 月牙进度条
Progress({ value: 50, total: 100, type: ProgressType.ScaleRing })
  .width(100)
  .height(100)
```

---

## 容器组件

### Column - 纵向布局

```typescript
Column() {
  Text('Item 1')
  Text('Item 2')
  Text('Item 3')
}
.width('100%')
.height('100%')
.justifyContent(FlexAlign.SpaceBetween)
.alignItems(HorizontalAlign.Center)
.padding(16)
```

### Row - 横向布局

```typescript
Row() {
  Text('Left')
  Blank()  // 弹性空白
  Text('Right')
}
.width('100%')
.height(50)
.padding({ left: 16, right: 16 })
```

### List - 列表组件

```typescript
List({ space: 8 }) {
  ForEach(this.items, (item: Item) => {
    ListItem() {
      this.buildItem(item)
    }
  }, (item: Item) => item.id.toString())
}
.width('100%')
.height('100%')
.divider({
  strokeWidth: 1,
  color: '#E5E5E5'
})
.scrollBar(BarState.Off)

@Builder
buildItem(item: Item) {
  Row() {
    Image(item.avatar)
      .width(48)
      .height(48)
      .borderRadius(24)
    Column() {
      Text(item.name)
        .fontSize(16)
        .fontWeight(FontWeight.Medium)
      Text(item.desc)
        .fontSize(14)
        .fontColor('#666')
    }
    .margin({ left: 12 })
  }
  .padding(12)
  .backgroundColor('#FFFFFF')
  .borderRadius(8)
}
```

### Tabs - 标签页

```typescript
@State currentIndex: number = 0;

Tabs({ barPosition: BarPosition.Bottom }) {
  TabContent() {
    HomePage()
  }.tabBar(this.buildTab('Home', $r('app.media.home'), 0))

  TabContent() {
    DiscoverPage()
  }.tabBar(this.buildTab('Discover', $r('app.media.discover'), 1))

  TabContent() {
    ProfilePage()
  }.tabBar(this.buildTab('Profile', $r('app.media.profile'), 2))
}
.onChange((index: number) => {
  this.currentIndex = index;
})

@Builder
buildTab(title: string, icon: Resource, index: number) {
  Column() {
    Image(this.currentIndex === index ? icon : icon)
      .width(24)
      .height(24)
      .fillColor(this.currentIndex === index ? '#007DFF' : '#999')
    Text(title)
      .fontSize(12)
      .fontColor(this.currentIndex === index ? '#007DFF' : '#999')
  }
}
```

### Swiper - 轮播组件

```typescript
Swiper() {
  ForEach(this.banners, (banner: Banner) => {
    Image(banner.image)
      .width('100%')
      .height(200)
      .borderRadius(12)
  })
}
.autoPlay(true)
.interval(3000)
.indicator(Indicator.dot()
  .color('#FFFFFF')
  .selectedColor('#007DFF'))
.loop(true)
```

---

## 弹窗组件

### AlertDialog - 警告弹窗

```typescript
AlertDialog.show({
  title: 'Confirm',
  message: 'Are you sure?',
  primaryButton: {
    value: 'OK',
    action: () => {
      console.log('Confirmed');
    }
  },
  secondaryButton: {
    value: 'Cancel',
    action: () => {
      console.log('Cancelled');
    }
  }
})
```

### 自定义弹窗

```typescript
@CustomDialog
struct CustomDialogComponent {
  controller: CustomDialogController;
  @State input: string = '';

  build() {
    Column() {
      Text('Custom Dialog')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      TextInput({ text: this.input })
        .onChange((value: string) => {
          this.input = value;
        })
      
      Row() {
        Button('Cancel')
          .onClick(() => {
            this.controller.close();
          })
        Button('Confirm')
          .onClick(() => {
            this.controller.close();
          })
      }
    }
    .padding(24)
    .width(300)
  }
}

// 使用
dialogController: CustomDialogController = new CustomDialogController({
  builder: CustomDialogComponent(),
  autoCancel: true,
  alignment: DialogAlignment.Center
});
```

### Toast - 提示

```typescript
import { promptAction } from '@kit.ArkUI';

// 简单提示
promptAction.showToast({
  message: 'Operation successful',
  duration: 2000
});
```

---

## 常见问题

### Q1: 如何实现下拉刷新？

**A:** 使用 Refresh 组件：
```typescript
@State isRefreshing: boolean = false;

Refresh({ refreshing: $$this.isRefreshing }) {
  List() {
    // 列表内容
  }
}
.onRefreshing(() => {
  // 刷新数据
  setTimeout(() => {
    this.isRefreshing = false;
  }, 1500);
})
```

### Q2: 如何实现上拉加载更多？

**A:** 监听列表滚动到底部：
```typescript
List() {
  ForEach(this.items, (item: Item) => {
    ListItem() {
      // 列表项
    }
  })
  
  if (this.hasMore) {
    ListItem() {
      LoadingProgress()
        .width(30)
        .height(30)
    }
    .onAppear(() => {
      this.loadMore();
    })
  }
}
```

### Q3: 如何优化长列表性能？

**A:** 使用 LazyForEach：
```typescript
List() {
  LazyForEach(this.dataSource, (item: Item) => {
    ListItem() {
      // 列表项
    }
  }, (item: Item) => item.id)
}
.cachedCount(5)  // 缓存数量
```

---

## 📚 延伸阅读

- [组件参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts-components)
- [组件通用属性](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-ts-component-common-attributes)
- [组件通用事件](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-ts-component-common-events)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
