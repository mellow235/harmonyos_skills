# HarmonyOS 6.0 综合开发技能工具

> 🚀 系统化学习鸿蒙应用开发的完整技能树

## 📖 目录结构

```
harmonyos-skill/
├── 01-ArkTS-Basic/          # ArkTS语言基础
├── 02-UI-Development/       # UI开发
├── 03-State-Management/     # 状态管理
├── 04-Components/           # 组件应用
├── 05-Layout/               # 布局设计
├── 06-Animation/            # 动画效果
├── 07-Router/               # 路由管理
├── 08-Storage/              # 数据存储
├── 09-Network/              # 网络请求
├── 10-Device/               # 设备交互
├── 11-Permission/           # 权限管理
├── 12-Testing/              # 测试调试
├── 13-3D-Graphics/          # 3D图形渲染
├── 14-AI-Engine/            # AI引擎
├── 15-Concurrency/          # 并发编程 
├── 16-Runtime-and-GC/       # 运行时与GC 
├── 17-Native-Extension/     # 原生扩展 
├── 18-Security/             # 代码安全 
├── 19-Version-Differences/  # 版本差异 
├── 20-Project-Engineering/  # 项目工程化 
├── 21-App-Architecture/     # 应用架构 
├── 22-Quality-Checklist/    # 质量检查
├── examples/                # 完整示例项目
├── templates/               # 项目模板
├── QUICK-REFERENCE.md       # 快速参考
└── README.md                # 本文件
```



### 基础入门
1. **ArkTS语言基础** → `01-ArkTS-Basic/`
   - ArkTS = TypeScript 严格约束子集 + ArkUI 声明式扩展
   - TS/ArkTS 精确区分、.ets 编译模式
   - 声明式 UI 范式与组件模型

2. **UI开发基础** → `02-UI-Development/`
   - ArkUI框架概述
   - 组件生命周期
   - 页面结构

### 核心技能
3. **组件应用** → `04-Components/`
   - 基础组件
   - 容器组件
   - 媒体组件

4. **布局设计** → `05-Layout/`
   - 线性布局
   - 弹性布局
   - 栅格布局

5. **状态管理** → `03-State-Management/`
   - @State装饰器
   - @Prop/@Link
   - @Provide/@Consume

### 进阶提升
6. **动画效果** → `06-Animation/`
   - 属性动画
   - 转场动画
   - 手势动画

7. **路由管理** → `07-Router/`
   - 页面路由
   - Navigation导航
   - 参数传递

8. **数据存储** → `08-Storage/`
   - 首选项存储
   - 关系型数据库
   - 文件存储

### 高级应用
9. **网络请求** → `09-Network/`
   - HTTP请求
   - WebSocket
   - 数据解析

10. **设备交互** → `10-Device/`
    - 传感器
    - 剪贴板
    - 屏幕管理

11. **权限管理** → `11-Permission/`
    - 权限申请
    - 权限组
    - 安全最佳实践

12. **测试调试** → `12-Testing/`
    - 单元测试
    - UI测试
    - 性能调试

### 高级特性
13. **3D图形渲染** → `13-3D-Graphics/`
    - XComponent 3D渲染
    - 3D变换与动画
    - OpenGL ES开发

14. **AI引擎** → `14-AI-Engine/`
    - HiAI Foundation
    - ML Kit机器学习
    - 语音识别与合成
    - 计算机视觉

### 专业进阶
15. **并发编程** → `15-Concurrency/`
    - TaskPool 任务池
    - Worker 多线程
    - @Sendable 共享对象
    - 共享容器与异步锁

16. **运行时与GC** → `16-Runtime-and-GC/`
    - 方舟运行时架构
    - HPP GC 机制
    - 内存优化策略

17. **原生扩展** → `17-Native-Extension/`
    - Node-API 跨语言交互
    - C/C++ 原生模块
    - Native Sendable 对象

18. **代码安全** → `18-Security/`
    - ArkGuard 代码混淆
    - 数据加密与网络安全
    - 应用签名与完整性

19. **版本差异** → `19-Version-Differences/`
    - API 9~12 语法演进
    - .ets 兼容性模式
    - 版本迁移检查清单

### 工程与架构
20. **项目工程化** → `20-Project-Engineering/`
    - DevEco Studio 项目创建
    - build-profile / module 配置
    - 多模块 HAP+HSP 架构
    - 代码混淆与构建打包
    - 签名与发布流程

21. **应用架构** → `21-App-Architecture/`
    - MVVM 完整分层架构
    - 页面生命周期管理
    - 全局状态管理方案
    - 导航架构（Router/Navigation/Tab）
    - 模块化拆分策略
    - 错误处理与设计模式

### 质量保障
22. **质量检查清单** → `22-Quality-Checklist/`
    - ArkTS 语法合规 13 项检查
    - API 导入规范检查
    - 组件/状态管理/工程配置规范
    - 性能与安全自检
    - 新项目/页面/组件的交付清单

## 📚 核心模块关联图

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Application)                     │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ 路由管理  │  │ 数据存储  │  │ 网络请求  │  │ 设备交互  │    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
│       │             │             │             │           │
├───────┴─────────────┴─────────────┴─────────────┴───────────┤
│                    专业进阶层 (Advanced)                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ 并发编程  │  │ 运行时GC  │  │ 原生扩展  │  │ 代码安全  │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
├─────────────────────────────────────────────────────────────┤
│                    工程与架构层 (Engineering)                  │
│  ┌──────────────────────┐  ┌──────────────────────────────┐  │
│  │  20-项目工程化        │  │  21-应用架构                  │  │
│  │  构建/打包/签名/发布   │  │  MVVM/导航/模块化/设计模式     │  │
│  └──────────────────────┘  └──────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                      智能层 (Intelligence)                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           3D图形 + AI引擎 + 计算机视觉 + NLP          │   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      框架层 (Framework)                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              UI开发 + 状态管理 + 动画效果              │   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      基础层 (Foundation)                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ ArkTS    │  │  组件库   │  │  布局系统  │  │  权限管理  │    │
│  │ (TS子集  │  │          │  │          │  │          │    │
│  │  +扩展)  │  │          │  │          │  │          │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
├─────────────────────────────────────────────────────────────┤
│                    版本兼容层 (Compatibility)                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │    19-版本差异：API 9~12 演进 + .ets 兼容性模式       │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 🔧 快速开始

### 环境准备
1. 安装 DevEco Studio 5.0+
2. 配置 HarmonyOS SDK
3. 创建 ArkTS 项目

### 第一个应用
```typescript
// EntryAbility.ets
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { window } from '@kit.ArkUI';

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onCreate');
  }

  onDestroy(): void {
    hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onDestroy');
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onWindowStageCreate');
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        hilog.error(0x0000, 'testTag', 'Failed to load content. Cause: %{public}s', JSON.stringify(err));
        return;
      }
      hilog.info(0x0000, 'testTag', 'Succeeded in loading content.');
    });
  }
}
```

```typescript
// pages/Index.ets
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS';

  build() {
    Row() {
      Column() {
        Text(this.message)
          .fontSize(30)
          .fontWeight(FontWeight.Bold)
      }
      .width('100%')
    }
    .height('100%')
  }
}
```

## 📋 代码示例索引

| 模块 | 示例数量 | 难度 | 路径 |
|------|---------|------|------|
| ArkTS基础 | 15+ | ⭐ | `01-ArkTS-Basic/examples/` |
| UI开发 | 20+ | ⭐⭐ | `02-UI-Development/examples/` |
| 状态管理 | 12+ | ⭐⭐ | `03-State-Management/examples/` |
| 组件应用 | 25+ | ⭐⭐ | `04-Components/examples/` |
| 布局设计 | 10+ | ⭐⭐ | `05-Layout/examples/` |
| 动画效果 | 15+ | ⭐⭐⭐ | `06-Animation/examples/` |
| 路由管理 | 8+ | ⭐⭐ | `07-Router/examples/` |
| 数据存储 | 10+ | ⭐⭐⭐ | `08-Storage/examples/` |
| 网络请求 | 8+ | ⭐⭐⭐ | `09-Network/examples/` |
| 设备交互 | 12+ | ⭐⭐⭐ | `10-Device/examples/` |
| 权限管理 | 6+ | ⭐⭐ | `11-Permission/examples/` |
| 测试调试 | 10+ | ⭐⭐⭐ | `12-Testing/examples/` |

## 🎨 最佳实践

### 代码规范
- 使用 ArkTS 严格模式
- 遵循命名规范（驼峰命名）
- 组件职责单一
- 合理使用状态管理

### 性能优化
- 避免过度渲染
- 使用 LazyForEach 懒加载
- 合理使用缓存
- 优化图片资源

### 安全实践
- 最小权限原则
- 数据加密存储
- 网络请求HTTPS
- 输入验证

## 🔗 官方资源

- [鸿蒙开发文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts)
- [ArkTS API参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkts-api)
- [DevEco Studio](https://developer.huawei.com/consumer/cn/deveco-studio/)
- [鸿蒙开发者社区](https://developer.huawei.com/consumer/cn/community/)

## 📝 更新日志

### v1.4.0 (2026-05-10)
- 🌐 **多工具适配**：创建 `.cursorrules`、`.github/copilot-instructions.md`、`UNIVERSAL-RULES.md`
- 📘 新增多工具接入指南（SKILL.md 底部）
- ✅ 新增 `22-Quality-Checklist/` 质量检查清单模块（13 项语法自检 + 交付清单）

### v1.3.0 (2026-05-10)
- 🔌 **Agent 可调用化**：创建 `.trae/skills/harmonyos-dev/SKILL.md` 标准入口
- 🏗️ 新增 `20-Project-Engineering/` 项目工程化模块（脚手架/构建/打包/签名）
- 🏛️ 新增 `21-App-Architecture/` 应用架构模块（MVVM/导航/生命周期/设计模式）
- 🔧 修复 `templates/README.md` 中错误 API 导入和 `any` 类型使用
- 📐 SKILL.md 包含完整可执行代码模板（Agent 可直接参照生成项目）

### v1.2.0 (2026-05-10)
- 🔄 重大优化：强化 TS/ArkTS 精确区分能力
- 📊 新增 `19-Version-Differences/` 版本差异对照模块
- 📝 重写 `01-ArkTS-Basic/` 核心定位为"TS 约束子集 + ArkUI 扩展"
- 📋 全面版本标注化：QUICK-REFERENCE 中所有约束标注生效 API Level
- 🚀 新增高性能编程实践（const 优先、TypedArray、循环优化）
- 📐 新增 .ets 兼容性模式说明（compatibleSdkVersion 行为差异）

### v1.1.0 (2026-05-03)
- ✨ 新增 `15-Concurrency/` 并发编程模块
- ✨ 新增 `16-Runtime-and-GC/` 运行时与 GC 模块
- ✨ 新增 `17-Native-Extension/` 原生扩展模块
- ✨ 新增 `18-Security/` 代码安全模块
- 🔧 修正 API 导入路径（@kit 命名空间）
- 📝 更新 QUICK-REFERENCE 与最佳实践

### v1.0.0 (2026-05-03)
- ✨ 初始化项目结构
- 📚 添加12个核心模块文档
- 💻 提供基础代码示例
- 📋 创建学习路径指南

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request 来完善这个技能工具。

## 📄 许可证

MIT License

---

> 💡 **提示**：本技能工具基于 HarmonyOS 6.0 (API 12) 版本，请确保开发环境与文档版本匹配。
