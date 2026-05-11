# 动画效果

> 🎬 让界面更生动

## 📖 目录

1. [动画概述](#动画概述)
2. [属性动画](#属性动画)
3. [显式动画](#显式动画)
4. [转场动画](#转场动画)
5. [手势动画](#手势动画)
6. [路径动画](#路径动画)
7. [动画最佳实践](#动画最佳实践)
8. [常见问题](#常见问题)

---

## 动画概述

ArkUI 提供丰富的动画能力，让界面交互更流畅自然。

### 动画类型

```
动画系统
├── 属性动画
│   ├── animateTo - 隐式动画
│   └── animation - 组件动画
├── 显式动画
│   └── Animator
├── 转场动画
│   ├── 页面转场
│   ├── 组件转场
│   └── 共享元素转场
├── 手势动画
│   ├── 拖拽
│   ├── 缩放
│   └── 旋转
└── 路径动画
    └── SVG Path 动画
```

---

## 属性动画

### animateTo - 隐式动画

```typescript
@Component
struct AnimationDemo {
  @State width: number = 100;
  @State height: number = 100;
  @State opacity: number = 1;
  @State angle: number = 0;

  build() {
    Column() {
      Text('Animated Box')
        .width(this.width)
        .height(this.height)
        .opacity(this.opacity)
        .rotate({ angle: this.angle })
        .backgroundColor('#007DFF')
        .borderRadius(8)

      Button('Animate')
        .onClick(() => {
          animateTo({
            duration: 500,
            curve: Curve.EaseInOut,
            onFinish: () => {
              console.log('Animation finished');
            }
          }, () => {
            this.width = 200;
            this.height = 200;
            this.opacity = 0.5;
            this.angle = 180;
          })
        })
    }
  }
}
```

### animation - 组件动画

```typescript
@Component
struct AnimationDemo {
  @State scale: number = 1;
  @State color: string = '#007DFF';

  build() {
    Column() {
      Text('Animated')
        .scale({ x: this.scale, y: this.scale })
        .backgroundColor(this.color)
        .animation({
          duration: 300,
          curve: Curve.FastOutSlowIn
        })
        .onClick(() => {
          this.scale = this.scale === 1 ? 1.5 : 1;
          this.color = this.color === '#007DFF' ? '#FF6B6B' : '#007DFF';
        })
    }
  }
}
```

### 动画曲线

```typescript
// 预设曲线
Curve.Linear              // 线性
Curve.Ease                // 缓入缓出
Curve.EaseIn              // 缓入
Curve.EaseOut             // 缓出
Curve.EaseInOut           // 缓入缓出
Curve.FastOutSlowIn       // 快出慢入
Curve.LinearOutSlowIn     // 线性出慢入
Curve.FastOutLinearIn     // 快出线性入

// 自定义曲线
Curve.CubicBezier(0.25, 0.1, 0.25, 1.0)

// 弹簧曲线
Curve.Spring              // 弹簧
Curve.SpringEstimate      // 弹簧估算
```

---

## 显式动画

### Animator

```typescript
@Component
struct AnimatorDemo {
  @State value: number = 0;
  private animator: Animator = new Animator({
    duration: 1000,
    easing: 'ease-in-out',
    iterations: -1,  // 无限循环
    direction: 'alternate',
    fill: 'forwards'
  });

  aboutToAppear() {
    this.animator.onframe = (progress: number) => {
      this.value = progress * 100;
    };
    this.animator.play();
  }

  aboutToDisappear() {
    this.animator.finish();
  }

  build() {
    Column() {
      Text(`Value: ${this.value}`)
      Progress({ value: this.value, total: 100, type: ProgressType.Linear })
        .width('80%')
    }
  }
}
```

---

## 转场动画

### 页面转场

```typescript
// pages/Index.ets
@Entry
@Component
struct Index {
  build() {
    Column() {
      Button('Go to Detail')
        .onClick(() => {
          router.pushUrl({
            url: 'pages/Detail'
          })
        })
    }
    .pageTransition({
      enter: PageTransitionEnter({
        type: TransitionType.Push,
        duration: 300
      }),
      exit: PageTransitionExit({
        type: TransitionType.Push,
        duration: 300
      })
    })
  }
}
```

### 组件转场

```typescript
@Component
struct TransitionDemo {
  @State show: boolean = false;

  build() {
    Column() {
      Button('Toggle')
        .onClick(() => {
          this.show = !this.show;
        })

      if (this.show) {
        Text('Content')
          .width(200)
          .height(100)
          .backgroundColor('#007DFF')
          .transition({
            opacity: { from: 0, to: 1 },
            scale: { from: { x: 0.8, y: 0.8 }, to: { x: 1, y: 1 } },
            translate: { y: 50 }
          })
      }
    }
  }
}
```

### 共享元素转场

```typescript
// 列表页
@Component
struct ListPage {
  build() {
    List() {
      ForEach(this.items, (item: Item) => {
        ListItem() {
          Image(item.image)
            .sharedTransition(item.id, {
              duration: 300,
              curve: Curve.EaseInOut
            })
        }
      })
    }
  }
}

// 详情页
@Component
struct DetailPage {
  @Prop item: Item;

  build() {
    Column() {
      Image(this.item.image)
        .sharedTransition(this.item.id, {
          duration: 300,
          curve: Curve.EaseInOut
        })
    }
  }
}
```

---

## 手势动画

### 拖拽动画

```typescript
@Component
struct DragDemo {
  @State offsetX: number = 0;
  @State offsetY: number = 0;

  build() {
    Column() {
      Text('Draggable')
        .width(100)
        .height(100)
        .backgroundColor('#007DFF')
        .borderRadius(8)
        .translate({ x: this.offsetX, y: this.offsetY })
        .gesture(
          PanGesture()
            .onActionStart(() => {
              console.log('Drag start');
            })
            .onActionUpdate((event: GestureEvent) => {
              this.offsetX = event.offsetX;
              this.offsetY = event.offsetY;
            })
            .onActionEnd(() => {
              console.log('Drag end');
            })
        )
    }
  }
}
```

### 缩放动画

```typescript
@Component
struct PinchDemo {
  @State scale: number = 1;

  build() {
    Column() {
      Image($r('app.media.image'))
        .width(200)
        .height(200)
        .scale({ x: this.scale, y: this.scale })
        .gesture(
          PinchGesture()
            .onActionUpdate((event: GestureEvent) => {
              this.scale = event.scale;
            })
        )
    }
  }
}
```

### 旋转动画

```typescript
@Component
struct RotationDemo {
  @State angle: number = 0;

  build() {
    Column() {
      Text('Rotatable')
        .width(100)
        .height(100)
        .backgroundColor('#007DFF')
        .rotate({ angle: this.angle })
        .gesture(
          RotationGesture()
            .onActionUpdate((event: GestureEvent) => {
              this.angle = event.angle;
            })
        )
    }
  }
}
```

---

## 路径动画

### SVG Path 动画

```typescript
@Component
struct PathAnimation {
  @State progress: number = 0;

  build() {
    Column() {
      Path()
        .width(200)
        .height(200)
        .commands('M 10 80 C 40 10, 65 10, 95 80 S 150 150, 180 80')
        .stroke('#007DFF')
        .strokeWidth(3)
        .fill('none')
        .strokeDashArray([this.progress * 300, 300])

      Button('Animate')
        .onClick(() => {
          animateTo({
            duration: 1500,
            curve: Curve.EaseInOut
          }, () => {
            this.progress = 1;
          })
        })
    }
  }
}
```

---

## 动画最佳实践

### 1. 选择合适的动画时长

```typescript
// ✅ 快速反馈
animateTo({ duration: 150 }, () => {
  // 状态变化
});

// ✅ 流畅过渡
animateTo({ duration: 300 }, () => {
  // 状态变化
});

// ❌ 过长动画
animateTo({ duration: 1000 }, () => {
  // 用户等待过久
});
```

### 2. 使用合适的曲线

```typescript
// ✅ 进入动画 - 缓出
animateTo({ curve: Curve.EaseOut }, () => { });

// ✅ 退出动画 - 缓入
animateTo({ curve: Curve.EaseIn }, () => { });

// ✅ 弹性效果
animateTo({ curve: Curve.Spring }, () => { });
```

### 3. 避免过度动画

```typescript
// ✅ 关键属性动画
Text('Hello')
  .opacity(this.isVisible ? 1 : 0)
  .animation({ duration: 200 })

// ❌ 多属性同时动画
Text('Hello')
  .opacity(this.opacity)
  .scale({ x: this.scale, y: this.scale })
  .rotate({ angle: this.angle })
  .translate({ x: this.x, y: this.y })
  // 性能影响
```

---

## 常见问题

### Q1: 动画卡顿怎么办？

**A:** 
- 减少同时动画的属性数量
- 使用 `transform` 代替位置属性
- 避免在动画中触发布局计算

### Q2: 如何实现循环动画？

**A:** 使用 Animator 的 iterations 属性：
```typescript
const animator = new Animator({
  iterations: -1,  // 无限循环
  direction: 'alternate'
});
```

### Q3: 如何取消动画？

**A:** 
```typescript
// 隐式动画无法直接取消
// 显式动画使用 finish()
animator.finish();
```

---

## 📚 延伸阅读

- [动画概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-animation-overview)
- [属性动画](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-animation-attribute)
- [转场动画](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-animation-transition)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
