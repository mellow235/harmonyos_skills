# 3D 图形与渲染

> 🎮 HarmonyOS 3D 能力与图形渲染

## 📖 目录

1. [3D 能力概述](#3d-能力概述)
2. [XComponent 3D 渲染](#xcomponent-3d-渲染)
3. [ArkUI 3D 组件](#arkui-3d-组件)
4. [3D 动画](#3d-动画)
5. [图形渲染管线](#图形渲染管线)
6. [常见问题](#常见问题)

---

## 3D 能力概述

HarmonyOS 提供多层次的 3D 图形能力，从底层 OpenGL ES 到高层 ArkUI 3D 组件。

### 3D 技术栈

```
3D 图形能力
├── 高层 API（ArkUI）
│   ├── 3D 组件
│   ├── 3D 变换
│   └── 3D 动画
├── 中层 API
│   ├── XComponent
│   └── NativeWindow
└── 底层 API
    ├── OpenGL ES 3.2
    ├── EGL
    └── NativeImage
```

---

## XComponent 3D 渲染

### 基础用法

```typescript
@Component
struct XComponentDemo {
  private xComponent: XComponentController = new XComponentController();

  build() {
    Column() {
      XComponent({
        id: 'xcomponent',
        type: XComponentType.SURFACE,
        libraryname: 'native_render'
      })
        .width('100%')
        .height(400)
        .onLoad(() => {
          console.log('XComponent loaded');
          // 初始化渲染上下文
          this.xComponent.setXComponentSurfaceSize({
            surfaceWidth: 1080,
            surfaceHeight: 720
          });
        })
        .onDestroy(() => {
          console.log('XComponent destroyed');
        })
    }
  }
}
```

### Native 渲染层 (C/C++)

```cpp
// native_render.cpp
#include <EGL/egl.h>
#include <GLES3/gl3.h>
#include <native_window/external_window.h>

// 初始化 EGL
EGLDisplay display = eglGetDisplay(EGL_DEFAULT_DISPLAY);
eglInitialize(display, nullptr, nullptr);

// 配置 EGL
EGLConfig config;
EGLint numConfigs;
EGLint configAttribs[] = {
    EGL_SURFACE_TYPE, EGL_WINDOW_BIT,
    EGL_RED_SIZE, 8,
    EGL_GREEN_SIZE, 8,
    EGL_BLUE_SIZE, 8,
    EGL_ALPHA_SIZE, 8,
    EGL_DEPTH_SIZE, 24,
    EGL_RENDERABLE_TYPE, EGL_OPENGL_ES3_BIT,
    EGL_NONE
};
eglChooseConfig(display, configAttribs, &config, 1, &numConfigs);

// 创建渲染上下文
EGLint contextAttribs[] = {
    EGL_CONTEXT_CLIENT_VERSION, 3,
    EGL_NONE
};
EGLContext context = eglCreateContext(display, config, EGL_NO_CONTEXT, contextAttribs);

// 渲染循环
void renderFrame() {
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // 绘制 3D 对象
    // ...
    
    eglSwapBuffers(display, surface);
}
```

---

## ArkUI 3D 组件

### 3D 变换

```typescript
@Component
struct Transform3DDemo {
  @State rotateX: number = 0;
  @State rotateY: number = 0;
  @State rotateZ: number = 0;

  build() {
    Column() {
      Text('3D Transform')
        .width(200)
        .height(200)
        .backgroundColor('#007DFF')
        .fontSize(20)
        .fontColor('#FFFFFF')
        .textAlign(TextAlign.Center)
        // 3D 旋转
        .rotate({
          x: 1,
          y: 0,
          z: 0,
          angle: this.rotateX
        })
        .rotate({
          x: 0,
          y: 1,
          z: 0,
          angle: this.rotateY
        })
        .rotate({
          x: 0,
          y: 0,
          z: 1,
          angle: this.rotateZ
        })
        // 透视效果
        .perspective(1000)

      Slider({ value: this.rotateX, min: 0, max: 360 })
        .onChange((value: number) => {
          this.rotateX = value;
        })
    }
  }
}
```

### 3D 翻转卡片

```typescript
@Component
struct FlipCard {
  @State isFlipped: boolean = false;
  @State rotateY: number = 0;

  build() {
    Stack() {
      // 正面
      Text('Front')
        .width(200)
        .height(300)
        .backgroundColor('#007DFF')
        .fontSize(24)
        .fontColor('#FFFFFF')
        .textAlign(TextAlign.Center)
        .borderRadius(12)
        .rotate({
          x: 0,
          y: 1,
          z: 0,
          angle: this.rotateY
        })
        .opacity(this.rotateY < 90 || this.rotateY > 270 ? 1 : 0)

      // 背面
      Text('Back')
        .width(200)
        .height(300)
        .backgroundColor('#FF6B6B')
        .fontSize(24)
        .fontColor('#FFFFFF')
        .textAlign(TextAlign.Center)
        .borderRadius(12)
        .rotate({
          x: 0,
          y: 1,
          z: 0,
          angle: this.rotateY + 180
        })
        .opacity(this.rotateY >= 90 && this.rotateY <= 270 ? 1 : 0)
    }
    .width(200)
    .height(300)
    .perspective(1000)
    .onClick(() => {
      animateTo({
        duration: 600,
        curve: Curve.EaseInOut
      }, () => {
        this.rotateY = this.isFlipped ? 0 : 180;
        this.isFlipped = !this.isFlipped;
      })
    })
  }
}
```

### 3D 轮播

```typescript
@Component
struct Carousel3D {
  @State currentIndex: number = 0;
  private items: string[] = ['Page 1', 'Page 2', 'Page 3', 'Page 4', 'Page 5'];
  private readonly ITEM_ANGLE: number = 360 / 5;
  private readonly RADIUS: number = 200;

  build() {
    Column() {
      Stack() {
        ForEach(this.items, (item: string, index: number) => {
          Text(item)
            .width(150)
            .height(200)
            .backgroundColor(this.getColor(index))
            .fontSize(18)
            .fontColor('#FFFFFF')
            .textAlign(TextAlign.Center)
            .borderRadius(12)
            .rotate({
              x: 0,
              y: 1,
              z: 0,
              angle: index * this.ITEM_ANGLE
            })
            .translate({
              z: this.RADIUS
            })
            .opacity(index === this.currentIndex ? 1 : 0.6)
            .scale({ x: index === this.currentIndex ? 1 : 0.8 })
        })
      }
      .width(300)
      .height(300)
      .perspective(1000)
      .rotate({
        x: 0,
        y: 1,
        z: 0,
        angle: -this.currentIndex * this.ITEM_ANGLE
      })

      Row() {
        Button('Previous')
          .onClick(() => {
            this.currentIndex = (this.currentIndex - 1 + this.items.length) % this.items.length;
          })
        Button('Next')
          .onClick(() => {
            this.currentIndex = (this.currentIndex + 1) % this.items.length;
          })
      }
      .margin({ top: 50 })
    }
  }

  private getColor(index: number): ResourceColor {
    const colors: ResourceColor[] = ['#007DFF', '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4'];
    return colors[index % colors.length];
  }
}
```

---

## 3D 动画

### 弹簧动画

```typescript
@Component
struct SpringAnimation3D {
  @State scale: number = 1;
  @State rotateY: number = 0;

  build() {
    Column() {
      Text('Spring 3D')
        .width(150)
        .height(150)
        .backgroundColor('#007DFF')
        .fontSize(20)
        .fontColor('#FFFFFF')
        .textAlign(TextAlign.Center)
        .borderRadius(12)
        .scale({ x: this.scale, y: this.scale })
        .rotate({ x: 0, y: 1, z: 0, angle: this.rotateY })
        .perspective(800)
        .gesture(
          TapGesture()
            .onAction(() => {
              animateTo({
                duration: 1000,
                curve: Curve.Spring
              }, () => {
                this.scale = this.scale === 1 ? 1.5 : 1;
                this.rotateY = this.rotateY === 0 ? 360 : 0;
              })
            })
        )
    }
  }
}
```

### 连续旋转

```typescript
@Component
struct ContinuousRotation {
  @State angle: number = 0;
  private animator: Animator = new Animator({
    duration: 3000,
    iterations: -1,  // 无限循环
    curve: Curve.Linear
  });

  aboutToAppear() {
    this.animator.onframe = (progress: number) => {
      this.angle = progress * 360;
    };
    this.animator.play();
  }

  aboutToDisappear() {
    this.animator.finish();
  }

  build() {
    Column() {
      Text('Rotating')
        .width(100)
        .height(100)
        .backgroundColor('#FF6B6B')
        .fontSize(16)
        .fontColor('#FFFFFF')
        .textAlign(TextAlign.Center)
        .borderRadius(12)
        .rotate({ x: 1, y: 1, z: 0, angle: this.angle })
        .perspective(500)
    }
  }
}
```

---

## 图形渲染管线

### 渲染流程

```
渲染管线
├── 顶点处理
│   ├── 顶点着色器
│   ├── 图元装配
│   └── 光栅化
├── 片段处理
│   ├── 片段着色器
│   ├── 深度测试
│   └── 模板测试
└── 输出合并
    ├── 混合
    ├── 抖动
    └── 帧缓冲
```

### 着色器示例 (GLSL)

```glsl
// 顶点着色器
#version 300 es
layout(location = 0) in vec3 aPosition;
layout(location = 1) in vec3 aNormal;

uniform mat4 uModelMatrix;
uniform mat4 uViewMatrix;
uniform mat4 uProjectionMatrix;

out vec3 vNormal;
out vec3 vFragPos;

void main() {
    gl_Position = uProjectionMatrix * uViewMatrix * uModelMatrix * vec4(aPosition, 1.0);
    vFragPos = vec3(uModelMatrix * vec4(aPosition, 1.0));
    vNormal = mat3(transpose(inverse(uModelMatrix))) * aNormal;
}
```

```glsl
// 片段着色器
#version 300 es
precision mediump float;

in vec3 vNormal;
in vec3 vFragPos;

uniform vec3 uLightPos;
uniform vec3 uViewPos;
uniform vec3 uObjectColor;

out vec4 FragColor;

void main() {
    // 环境光
    float ambientStrength = 0.1;
    vec3 ambient = ambientStrength * vec3(1.0);
    
    // 漫反射
    vec3 norm = normalize(vNormal);
    vec3 lightDir = normalize(uLightPos - vFragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = diff * vec3(1.0);
    
    // 镜面反射
    float specularStrength = 0.5;
    vec3 viewDir = normalize(uViewPos - vFragPos);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32.0);
    vec3 specular = specularStrength * spec * vec3(1.0);
    
    vec3 result = (ambient + diffuse + specular) * uObjectColor;
    FragColor = vec4(result, 1.0);
}
```

---

## 常见问题

### Q1: 如何加载 3D 模型？

**A:** 使用 XComponent + Native 层加载：
- 支持 glTF 2.0 格式
- 使用 Assimp 库解析模型
- 在 Native 层渲染

### Q2: 3D 性能优化？

**A:**
- 使用 LOD（细节层次）
- 合批渲染
- 纹理压缩
- 遮挡剔除

### Q3: 如何实现 AR 效果？

**A:** 使用 AR Engine：
- 相机标定
- 特征点检测
- 位姿估计
- 虚实融合

---

## 📚 延伸阅读

- [XComponent 组件](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts-xcomponent)
- [OpenGL ES 开发](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/native-rendering)
- [图形渲染指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/graphics-overview)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
