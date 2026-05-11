# 布局设计

> 📐 灵活的布局系统

## 📖 目录

1. [布局概述](#布局概述)
2. [线性布局](#线性布局)
3. [弹性布局](#弹性布局)
4. [层叠布局](#层叠布局)
5. [网格布局](#网格布局)
6. [栅格布局](#栅格布局)
7. [布局技巧](#布局技巧)
8. [常见问题](#常见问题)

---

## 布局概述

ArkUI 提供多种布局方式，满足不同场景需求。

### 布局类型

```
布局系统
├── 线性布局
│   ├── Column - 纵向
│   └── Row - 横向
├── 弹性布局
│   └── Flex
├── 层叠布局
│   └── Stack
├── 网格布局
│   └── Grid / GridItem
├── 栅格布局
│   └── GridRow / GridCol
└── 列表布局
    └── List / ListItem
```

---

## 线性布局

### Column - 纵向布局

```typescript
// 基础纵向布局
Column() {
  Text('Item 1')
  Text('Item 2')
  Text('Item 3')
}
.width('100%')
.height('100%')

// 对齐方式
Column() {
  Text('Left')
  Text('Center')
  Text('Right')
}
.width('100%')
.alignItems(HorizontalAlign.Start)    // 左对齐
.alignItems(HorizontalAlign.Center)   // 居中
.alignItems(HorizontalAlign.End)      // 右对齐

// 分布方式
Column() {
  Text('Top')
  Text('Middle')
  Text('Bottom')
}
.height('100%')
.justifyContent(FlexAlign.Start)      // 顶部
.justifyContent(FlexAlign.Center)     // 居中
.justifyContent(FlexAlign.End)        // 底部
.justifyContent(FlexAlign.SpaceBetween) // 两端对齐
.justifyContent(FlexAlign.SpaceAround)  // 均匀分布
.justifyContent(FlexAlign.SpaceEvenly)  // 完全均匀
```

### Row - 横向布局

```typescript
// 基础横向布局
Row() {
  Text('Left')
  Text('Center')
  Text('Right')
}
.width('100%')
.height(50)

// 对齐方式
Row() {
  Text('Top')
  Text('Middle')
  Text('Bottom')
}
.width('100%')
.height(100)
.alignItems(VerticalAlign.Top)      // 顶部对齐
.alignItems(VerticalAlign.Center)   // 垂直居中
.alignItems(VerticalAlign.Bottom)   // 底部对齐

// 使用 Blank 填充
Row() {
  Text('Left')
  Blank()  // 弹性空白
  Text('Right')
}
.width('100%')
.padding(16)
```

---

## 弹性布局

### Flex 布局

```typescript
// 基础 Flex
Flex({ direction: FlexDirection.Row }) {
  Text('1')
    .width(50)
    .height(50)
    .backgroundColor('#FF6B6B')
  Text('2')
    .width(50)
    .height(50)
    .backgroundColor('#4ECDC4')
  Text('3')
    .width(50)
    .height(50)
    .backgroundColor('#45B7D1')
}
.width('100%')
.padding(16)

// Flex 方向
Flex({ direction: FlexDirection.Row })        // 水平
Flex({ direction: FlexDirection.Column })     // 垂直
Flex({ direction: FlexDirection.RowReverse }) // 水平反向
Flex({ direction: FlexDirection.ColumnReverse }) // 垂直反向

// 对齐方式
Flex({
  direction: FlexDirection.Row,
  justifyContent: FlexAlign.SpaceBetween,  // 主轴对齐
  alignItems: ItemAlign.Center,            // 交叉轴对齐
  wrap: FlexWrap.Wrap                      // 换行
}) {
  // 子组件
}

// Flex 子项属性
Text('Item')
  .flexGrow(1)      // 扩展比例
  .flexShrink(0)    // 收缩比例
  .flexBasis('30%') // 基础大小
```

---

## 层叠布局

### Stack 布局

```typescript
// 基础层叠
Stack() {
  Image($r('app.media.bg'))
    .width('100%')
    .height(200)
  Text('Overlay Text')
    .fontSize(24)
    .fontColor('#FFFFFF')
}
.width('100%')
.height(200)

// 对齐方式
Stack({ alignContent: Alignment.TopStart })      // 左上
Stack({ alignContent: Alignment.TopCenter })      // 顶部居中
Stack({ alignContent: Alignment.TopEnd })         // 右上
Stack({ alignContent: Alignment.Center })         // 完全居中
Stack({ alignContent: Alignment.BottomStart })    // 左下
Stack({ alignContent: Alignment.BottomCenter })   // 底部居中
Stack({ alignContent: Alignment.BottomEnd })      // 右下

// 层叠顺序（后添加的在上面）
Stack() {
  Text('Layer 1')
    .width(100)
    .height(100)
    .backgroundColor('#FF6B6B')
  Text('Layer 2')
    .width(80)
    .height(80)
    .backgroundColor('#4ECDC4')
  Text('Layer 3')
    .width(60)
    .height(60)
    .backgroundColor('#45B7D1')
}
.width(150)
.height(150)
```

---

## 网格布局

### Grid / GridItem

```typescript
// 基础网格
Grid() {
  ForEach(this.items, (item: Item) => {
    GridItem() {
      Text(item.name)
        .width('100%')
        .height('100%')
        .backgroundColor('#F5F5F5')
        .borderRadius(8)
    }
  })
}
.columnsTemplate('1fr 1fr 1fr')  // 三列等宽
.rowsTemplate('1fr 1fr')         // 两行等高
.columnsGap(8)
.rowsGap(8)
.width('100%')
.height(300)

// 不等宽网格
Grid() {
  GridItem() {
    Text('1')
  }.columnStart(0).columnEnd(1)  // 占2列

  GridItem() {
    Text('2')
  }

  GridItem() {
    Text('3')
  }
}
.columnsTemplate('1fr 1fr 1fr')
.rowsTemplate('1fr 1fr')
```

---

## 栅格布局

### GridRow / GridCol

```typescript
// 响应式栅格
GridRow({ columns: { sm: 4, md: 8, lg: 12 } }) {
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) {
    Text('Left Half')
      .width('100%')
      .height(100)
      .backgroundColor('#FF6B6B')
  }
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) {
    Text('Right Half')
      .width('100%')
      .height(100)
      .backgroundColor('#4ECDC4')
  }
}
.width('100%')

// 偏移
GridCol({
  span: { sm: 4, md: 6, lg: 8 },
  offset: { sm: 0, md: 1, lg: 2 }
}) {
  Text('Content')
}
```

---

## 布局技巧

### 1. 居中对齐

```typescript
// 完全居中
Column() {
  Text('Centered')
}
.width('100%')
.height('100%')
.justifyContent(FlexAlign.Center)
.alignItems(HorizontalAlign.Center)

// 使用 Stack
Stack({ alignContent: Alignment.Center }) {
  Text('Centered')
}
.width('100%')
.height('100%')
```

### 2. 等分布局

```typescript
// 三等分
Row() {
  Text('1')
    .layoutWeight(1)
    .height(50)
    .backgroundColor('#FF6B6B')
  Text('2')
    .layoutWeight(1)
    .height(50)
    .backgroundColor('#4ECDC4')
  Text('3')
    .layoutWeight(1)
    .height(50)
    .backgroundColor('#45B7D1')
}
.width('100%')
```

### 3. 固定 + 弹性布局

```typescript
Row() {
  Text('Fixed')
    .width(100)
    .height(50)
    .backgroundColor('#FF6B6B')
  Text('Flexible')
    .layoutWeight(1)
    .height(50)
    .backgroundColor('#4ECDC4')
}
.width('100%')
```

### 4. 底部固定布局

```typescript
Column() {
  // 内容区域
  Scroll() {
    // 长内容
  }
  .layoutWeight(1)

  // 底部固定
  Row() {
    Button('Submit')
      .width('100%')
  }
  .padding(16)
  .backgroundColor('#FFFFFF')
}
.height('100%')
```

---

## 常见问题

### Q1: 如何实现滚动布局？

**A:** 使用 Scroll 或 List：
```typescript
Scroll() {
  Column() {
    // 内容
  }
}
.scrollable(ScrollDirection.Vertical)
```

### Q2: 如何实现吸顶效果？

**A:** 使用 sticky 属性：
```typescript
List() {
  ListItemGroup({ header: this.buildHeader() }) {
    // 列表项
  }
  .sticky(Sticky.Normal)
}
```

### Q3: 如何处理不同屏幕尺寸？

**A:** 使用断点和栅格系统：
```typescript
@StorageProp('breakpoint') breakpoint: string = 'sm';

build() {
  Column() {
    if (this.breakpoint === 'sm') {
      // 手机布局
    } else {
      // 平板布局
    }
  }
}
```

---

## 📚 延伸阅读

- [布局概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-layout-overview)
- [线性布局](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-layout-linear)
- [弹性布局](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-layout-flex)
- [层叠布局](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-layout-stack)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
