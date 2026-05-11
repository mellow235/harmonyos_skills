# 项目模板

> 📋 快速启动项目的基础模板

## 基础项目模板

### 项目结构

```
my-app/
├── entry/
│   └── src/
│       └── main/
│           ├── ets/
│           │   ├── entryability/
│           │   │   └── EntryAbility.ets
│           │   ├── pages/
│           │   │   ├── Index.ets
│           │   │   └── Detail.ets
│           │   ├── components/
│           │   │   └── CommonComponents.ets
│           │   ├── utils/
│           │   │   ├── Logger.ets
│           │   │   └── Utils.ets
│           │   └── models/
│           │       └── Models.ets
│           └── resources/
│               └── base/
│                   ├── element/
│                   │   └── string.json
│                   ├── media/
│                   │   └── icon.png
│                   └── profile/
│                       └── main_pages.json
└── build-profile.json5
```

---

### EntryAbility.ets

```typescript
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { window } from '@kit.ArkUI';

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, 'EntryAbility', 'onCreate');
  }

  onDestroy(): void {
    hilog.info(0x0000, 'EntryAbility', 'onDestroy');
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    hilog.info(0x0000, 'EntryAbility', 'onWindowStageCreate');
    
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        hilog.error(0x0000, 'EntryAbility', 'Failed to load content: %{public}s', JSON.stringify(err));
        return;
      }
      hilog.info(0x0000, 'EntryAbility', 'Content loaded successfully');
    });
  }

  onWindowStageDestroy(): void {
    hilog.info(0x0000, 'EntryAbility', 'onWindowStageDestroy');
  }

  onForeground(): void {
    hilog.info(0x0000, 'EntryAbility', 'onForeground');
  }

  onBackground(): void {
    hilog.info(0x0000, 'EntryAbility', 'onBackground');
  }
}
```

---

### pages/Index.ets

```typescript
import { router } from '@kit.ArkUI';

@Entry
@Component
struct Index {
  @State message: string = 'Welcome to HarmonyOS';

  build() {
    Column() {
      // 顶部导航
      Row() {
        Text('My App')
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
      }
      .width('100%')
      .height(56)
      .padding({ left: 16, right: 16 })
      .backgroundColor('#FFFFFF')

      // 内容区域
      Column() {
        Text(this.message)
          .fontSize(28)
          .fontWeight(FontWeight.Bold)
          .margin({ bottom: 20 })

        Button('Get Started')
          .width(200)
          .height(50)
          .fontSize(16)
          .backgroundColor('#007DFF')
          .borderRadius(25)
          .onClick(() => {
            router.pushUrl({ url: 'pages/Detail' });
          })
      }
      .layoutWeight(1)
      .justifyContent(FlexAlign.Center)
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }
}
```

---

### pages/Detail.ets

```typescript
import { router } from '@kit.ArkUI';

@Entry
@Component
struct DetailPage {
  @State title: string = 'Detail Page';

  build() {
    Column() {
      // 返回按钮
      Row() {
        Image($r('app.media.back'))
          .width(24)
          .height(24)
          .onClick(() => {
            router.back();
          })
        Text(this.title)
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
          .margin({ left: 12 })
      }
      .width('100%')
      .height(56)
      .padding({ left: 16, right: 16 })
      .backgroundColor('#FFFFFF')

      // 内容
      Column() {
        Text('Detail Content')
          .fontSize(18)
      }
      .layoutWeight(1)
      .justifyContent(FlexAlign.Center)
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }
}
```

---

### components/CommonComponents.ets

```typescript
// 通用按钮组件
@Component
struct PrimaryButton {
  @Prop label: string = '';
  @Prop onClick: () => void = () => {};

  build() {
    Button(this.label)
      .width('100%')
      .height(50)
      .fontSize(16)
      .backgroundColor('#007DFF')
      .borderRadius(25)
      .onClick(this.onClick)
  }
}

// 通用卡片组件
@Component
struct Card {
  @Prop title: string = '';
  @Prop content: string = '';

  build() {
    Column() {
      Text(this.title)
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
      Text(this.content)
        .fontSize(14)
        .fontColor('#666')
        .margin({ top: 8 })
    }
    .width('100%')
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
}

// 导航栏组件
@Component
struct NavBar {
  @Prop title: string = '';
  @Prop showBack: boolean = true;

  build() {
    Row() {
      if (this.showBack) {
        Image($r('app.media.back'))
          .width(24)
          .height(24)
          .onClick(() => {
            router.back();
          })
      }
      Text(this.title)
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .margin({ left: this.showBack ? 12 : 0 })
    }
    .width('100%')
    .height(56)
    .padding({ left: 16, right: 16 })
    .backgroundColor('#FFFFFF')
  }
}
```

---

### utils/Logger.ets

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit';

export class Logger {
  private static DOMAIN: number = 0x0001;
  private static TAG: string = 'MyApp';

  static debug(message: string): void {
    hilog.debug(Logger.DOMAIN, Logger.TAG, '%{public}s', message);
  }

  static info(message: string): void {
    hilog.info(Logger.DOMAIN, Logger.TAG, '%{public}s', message);
  }

  static warn(message: string): void {
    hilog.warn(Logger.DOMAIN, Logger.TAG, '%{public}s', message);
  }

  static error(message: string): void {
    hilog.error(Logger.DOMAIN, Logger.TAG, '%{public}s', message);
  }
}
```

---

### utils/Utils.ets

```typescript
export class Utils {
  static formatDate(date: Date, format: string = 'YYYY-MM-DD'): string {
    const year: number = date.getFullYear();
    const month: string = String(date.getMonth() + 1).padStart(2, '0');
    const day: string = String(date.getDate()).padStart(2, '0');
    return format
      .replace('YYYY', year.toString())
      .replace('MM', month)
      .replace('DD', day);
  }

  static generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substring(2);
  }
}
```

---

### models/Models.ets

```typescript
// 用户模型
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

// API 响应模型
export interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
}

// 分页响应
export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
}

// 通用列表项
export interface ListItem {
  id: string;
  title: string;
  description?: string;
  icon?: string;
}
```

---

### main_pages.json

```json
{
  "src": [
    "pages/Index",
    "pages/Detail"
  ]
}
```

---

### string.json

```json
{
  "string": [
    {
      "name": "app_name",
      "value": "My App"
    },
    {
      "name": "main_desc",
      "value": "Welcome to HarmonyOS"
    }
  ]
}
```

---

## 使用模板

1. 在 DevEco Studio 创建新项目
2. 替换对应文件内容
3. 添加所需资源文件
4. 运行项目

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
