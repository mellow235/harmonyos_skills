# 应用架构与设计模式

> 🏛️ HarmonyOS 应用架构设计与最佳实践

## 📖 目录

1. [MVVM 架构模式](#mvvm-架构模式)
2. [页面生命周期管理](#页面生命周期管理)
3. [全局状态管理方案](#全局状态管理方案)
4. [导航架构设计](#导航架构设计)
5. [模块化拆分策略](#模块化拆分策略)
6. [错误处理架构](#错误处理架构)
7. [设计模式速查](#设计模式速查)

---

## MVVM 架构模式

### 分层职责

```
┌─────────────────────────────────────────────┐
│  View 层 (Pages/Components)                  │
│  ├── 声明式 UI 渲染                          │
│  ├── 用户交互事件绑定                         │
│  └── 消费 ViewModel 状态                     │
├─────────────────────────────────────────────┤
│  ViewModel 层 (viewmodels/)                  │
│  ├── 持有 UI 状态 (@State 数据源)             │
│  ├── 调用 Service 获取数据                    │
│  ├── 业务逻辑处理                             │
│  └── 状态转换与格式化                         │
├─────────────────────────────────────────────┤
│  Service 层 (services/)                      │
│  ├── 网络请求封装                             │
│  ├── 数据库操作封装                           │
│  ├── 缓存策略                                │
│  └── 第三方 SDK 封装                         │
├─────────────────────────────────────────────┤
│  Model 层 (models/)                          │
│  ├── 数据接口定义 (interface)                 │
│  ├── 枚举与常量                               │
│  └── 数据转换工具                             │
└─────────────────────────────────────────────┘
```

### 完整示例：用户列表页

```typescript
// ── models/UserModel.ets ──
export interface UserModel {
  id: string;
  name: string;
  email: string;
  avatar: string;
  role: UserRole;
}

export const enum UserRole {
  Admin = 'ADMIN',
  Member = 'MEMBER',
  Guest = 'GUEST'
}

// ── models/ApiResponse.ets ──
export interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
}

export interface PaginatedData<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
}

// ── services/UserService.ets ──
import { http } from '@kit.NetworkKit';
import { UserModel } from '../models/UserModel';
import { ApiResponse, PaginatedData } from '../models/ApiResponse';

const BASE_URL: string = 'https://api.example.com';
const TIMEOUT: number = 10000;

export class UserService {
  static async fetchUsers(page: number = 1): Promise<PaginatedData<UserModel>> {
    const request = http.createHttp();
    const url: string = `${BASE_URL}/users?page=${page}&pageSize=20`;

    const response = await request.request(url, {
      method: http.RequestMethod.GET,
      header: { 'Content-Type': 'application/json' },
      connectTimeout: TIMEOUT,
      readTimeout: TIMEOUT
    });

    request.destroy();

    const body: Record<string, Object> = JSON.parse(response.result as string);
    const paginated: PaginatedData<UserModel> = {
      items: body['items'] as UserModel[],
      total: body['total'] as number,
      page: body['page'] as number,
      pageSize: body['pageSize'] as number
    };
    return paginated;
  }

  static async deleteUser(id: string): Promise<boolean> {
    const request = http.createHttp();
    const response = await request.request(`${BASE_URL}/users/${id}`, {
      method: http.RequestMethod.DELETE,
      connectTimeout: TIMEOUT,
      readTimeout: TIMEOUT
    });
    request.destroy();
    return response.responseCode === 200;
  }
}

// ── viewmodels/UserListViewModel.ets ──
import { UserModel } from '../models/UserModel';
import { PaginatedData } from '../models/ApiResponse';
import { UserService } from '../services/UserService';

export class UserListViewModel {
  users: UserModel[] = [];
  isLoading: boolean = false;
  isRefreshing: boolean = false;
  isLoadingMore: boolean = false;
  errorMessage: string = '';
  currentPage: number = 1;
  totalCount: number = 0;
  hasMore: boolean = true;

  async loadUsers(): Promise<void> {
    this.isLoading = true;
    this.errorMessage = '';
    this.currentPage = 1;

    try {
      const result: PaginatedData<UserModel> = await UserService.fetchUsers(1);
      this.users = result.items;
      this.totalCount = result.total;
      this.hasMore = this.users.length < this.totalCount;
    } catch (err) {
      this.errorMessage = '加载用户列表失败';
    } finally {
      this.isLoading = false;
    }
  }

  async refreshUsers(): Promise<void> {
    this.isRefreshing = true;
    try {
      const result: PaginatedData<UserModel> = await UserService.fetchUsers(1);
      this.users = result.items;
      this.currentPage = 1;
      this.hasMore = this.users.length < result.total;
    } finally {
      this.isRefreshing = false;
    }
  }

  async loadMoreUsers(): Promise<void> {
    if (!this.hasMore || this.isLoadingMore) {
      return;
    }
    this.isLoadingMore = true;
    try {
      const nextPage: number = this.currentPage + 1;
      const result: PaginatedData<UserModel> = await UserService.fetchUsers(nextPage);
      this.users = this.users.concat(result.items);
      this.currentPage = nextPage;
      this.hasMore = this.users.length < result.total;
    } finally {
      this.isLoadingMore = false;
    }
  }

  async deleteUser(id: string): Promise<void> {
    const success: boolean = await UserService.deleteUser(id);
    if (success) {
      this.users = this.users.filter((u: UserModel) => u.id !== id);
    }
  }
}

// ── pages/UserListPage.ets ──
import { UserListViewModel } from '../viewmodels/UserListViewModel';
import { UserModel, UserRole } from '../models/UserModel';
import { EmptyState } from '../components/EmptyState';

@Entry
@Component
struct UserListPage {
  private viewModel: UserListViewModel = new UserListViewModel();

  aboutToAppear(): void {
    this.viewModel.loadUsers();
  }

  build() {
    Column() {
      // 标题栏
      Row() {
        Text('用户列表').fontSize(20).fontWeight(FontWeight.Bold)
        Blank()
        Text(`${this.viewModel.users.length} 人`).fontSize(14).fontColor('#999')
      }
      .width('100%').height(56).padding({ left: 16, right: 16 })
      .backgroundColor('#FFFFFF')

      // 内容区
      if (this.viewModel.isLoading) {
        LoadingProgress().width(50).height(50)
      } else if (this.viewModel.errorMessage !== '') {
        this.buildErrorState()
      } else if (this.viewModel.users.length === 0) {
        EmptyState({ message: '暂无用户数据' })
      } else {
        List() {
          ForEach(this.viewModel.users, (user: UserModel) => {
            ListItem() { this.buildUserItem(user) }
          }, (user: UserModel) => user.id)

          if (this.viewModel.isLoadingMore) {
            ListItem() {
              Row() {
                LoadingProgress().width(24).height(24)
                Text('加载中...').fontSize(12).fontColor('#999').margin({ left: 8 })
              }.justifyContent(FlexAlign.Center).padding(16)
            }
          }
        }
        .layoutWeight(1)
        .onReachEnd(() => { this.viewModel.loadMoreUsers() })
      }
    }
    .width('100%').height('100%').backgroundColor('#F5F5F5')
  }

  @Builder buildErrorState() {
    Column() {
      Text('加载失败').fontSize(16).margin({ bottom: 16 })
      Button('重试').onClick(() => { this.viewModel.loadUsers() })
    }
    .width('100%').layoutWeight(1)
    .justifyContent(FlexAlign.Center)
  }

  @Builder buildUserItem(user: UserModel) {
    Row() {
      Image(user.avatar).width(48).height(48).borderRadius(24)
      Column() {
        Row() {
          Text(user.name).fontSize(16).fontWeight(FontWeight.Medium)
          if (user.role === UserRole.Admin) {
            Text('管理员').fontSize(10).fontColor('#007DFF')
              .backgroundColor('#E6F2FF').padding({ left: 4, right: 4 })
              .borderRadius(4).margin({ left: 8 })
          }
        }
        Text(user.email).fontSize(12).fontColor('#999').margin({ top: 4 })
      }.layoutWeight(1).margin({ left: 12 }).alignItems(HorizontalAlign.Start)

      Image($r('app.media.delete'))
        .width(24).height(24)
        .onClick(() => { this.viewModel.deleteUser(user.id) })
    }
    .width('100%').padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .margin({ bottom: 8 })
  }
}
```

---

## 页面生命周期管理

### 生命周期顺序

```
页面创建: aboutToAppear → build → onPageShow
页面隐藏: onPageHide
页面返回: onPageHide → aboutToDisappear
页面销毁: aboutToDisappear
```

### 典型用法

```typescript
@Entry
@Component
struct MyPage {
  @State data: string = '';

  aboutToAppear(): void {
    // ✅ 初始化数据、注册监听
    this.loadData();
  }

  onPageShow(): void {
    // ✅ 页面显示时刷新（从其他页面返回时触发）
    this.refreshData();
  }

  onPageHide(): void {
    // ✅ 暂停操作（如暂停视频播放）
    this.pauseOperations();
  }

  aboutToDisappear(): void {
    // ✅ 清理资源、取消监听、销毁定时器
    this.cleanupResources();
  }

  build() {
    Column() {
      Text(this.data)
    }
  }
}
```

---

## 全局状态管理方案

### 方案选择指南

| 场景 | 推荐方案 | 说明 |
|------|---------|------|
| 少量全局配置 | `AppStorage` | 通过 `@StorageLink` 绑定 |
| 用户信息/登录态 | `AppStorage` + `PersistentStorage` | 持久化 + 响应式 |
| 复杂全局状态（多页面） | `@Provide/@Consume` | 跨层级自动注入 |
| 状态管理 V2（API 12+） | `@ComponentV2` + `@Local/@Param` | 新版推荐 |

### AppStorage + PersistentStorage 示例

```typescript
// ── constants/AppConstants.ets ──
export const enum StorageKeys {
  IsLoggedIn = 'is_logged_in',
  UserInfo = 'user_info',
  ThemeMode = 'theme_mode'
}

// ── entryability/EntryAbility.ets ── (初始化)
import { AppStorage, PersistentStorage } from '@kit.ArkUI';

onCreate(): void {
  PersistentStorage.persistProp(StorageKeys.IsLoggedIn, false);
  PersistentStorage.persistProp(StorageKeys.UserInfo, '{}');
}

// ── 任一页面使用 ──
@Entry
@Component
struct SettingsPage {
  @StorageLink(StorageKeys.ThemeMode) themeMode: string = 'light';

  build() {
    Column() {
      Toggle({ type: ToggleType.Switch, isOn: this.themeMode === 'dark' })
        .onChange((isOn: boolean) => {
          this.themeMode = isOn ? 'dark' : 'light';
        })
    }
  }
}
```

---

## 导航架构设计

### 简单应用（2-5 个页面）：Router

```typescript
import { router } from '@kit.ArkUI';

// 跳转
router.pushUrl({ url: 'pages/Detail', params: { id: '123' } });
// 返回
router.back();
// 替换当前页
router.replaceUrl({ url: 'pages/Login' });
// 清空栈后跳转
router.clear();
router.pushUrl({ url: 'pages/Home' });
```

### 复杂应用（5+ 页面）：Navigation

```typescript
@Entry
@Component
struct MainNavigation {
  private navPathStack: NavPathStack = new NavPathStack();

  @Builder pagesMapBuilder(name: string) {
    if (name === 'Home') {
      HomePage()
    } else if (name === 'Detail') {
      DetailPage()
    } else if (name === 'Settings') {
      SettingsPage()
    }
  }

  build() {
    Navigation(this.navPathStack) {
      HomePage()
    }
    .title('My App')
    .mode(NavigationMode.Stack)
    .navDestination(this.pagesMapBuilder)
  }
}

// 子页面跳转
@Component
struct HomePage {
  private navPathStack: NavPathStack = new NavPathStack();

  build() {
    Button('Go to Detail')
      .onClick(() => {
        this.navPathStack.pushPathByName('Detail', { id: '123' });
      })
  }
}
```

### Tab + Navigation 混合架构

```typescript
@Entry
@Component
struct MainTabs {
  @State currentIndex: number = 0;
  private tabs: string[] = ['Home', 'Discover', 'Profile'];

  build() {
    Column() {
      Tabs({ index: this.currentIndex }) {
        TabContent() {
          Navigation(new NavPathStack()) {
            HomePage()
          }.mode(NavigationMode.Stack)
        }.tabBar('首页')

        TabContent() {
          Navigation(new NavPathStack()) {
            DiscoverPage()
          }.mode(NavigationMode.Stack)
        }.tabBar('发现')

        TabContent() {
          Navigation(new NavPathStack()) {
            ProfilePage()
          }.mode(NavigationMode.Stack)
        }.tabBar('我的')
      }
      .barHeight(56)
      .onChange((index: number) => { this.currentIndex = index })
    }
  }
}
```

---

## 模块化拆分策略

### 拆分原则

```
entry/ (主模块 - HAP)
  ├── 只放 App 壳 (EntryAbility + 首页)
  └── 依赖所有 feature 模块

feature_xxx/ (功能模块 - HAP)
  ├── 独立可部署的功能单元
  ├── 自己的 pages/components/viewmodels/services/models
  └── 通过 router 或 Navigation 互通

library_yyy/ (共享库 - HSP)
  ├── 公共组件 / 工具类 / 数据层
  └── 被多个模块依赖
```

### 何时拆分模块

| 条件 | 操作 |
|------|------|
| 团队 > 3 人并行开发 | 按功能拆 HAP |
| 某功能需独立迭代 | 独立 HAP 模块 |
| 代码复用 ≥ 2 个模块 | 提取为 HSP 库 |
| 包体积过大 | 拆 HSP 动态加载 |

---

## 错误处理架构

### 分层错误处理

```typescript
// ── constants/ErrorCodes.ets ──
export const enum ErrorCodes {
  NetworkError = 1001,
  ServerError = 1002,
  AuthExpired = 1003,
  DataParseError = 1004,
  UnknownError = 9999
}

export interface AppError {
  code: number;
  message: string;
  original?: Object;
}

// ── services/ErrorHandler.ets ──
import { hilog } from '@kit.PerformanceAnalysisKit';
import { AppError, ErrorCodes } from '../constants/ErrorCodes';

export class ErrorHandler {
  static handle(error: Object, context: string): AppError {
    hilog.error(0x0000, 'ErrorHandler', '%{public}s: %{public}s',
      context, JSON.stringify(error));

    const errorObj: Record<string, Object> = error as Record<string, Object>;
    if (errorObj['responseCode'] !== undefined) {
      const code: number = errorObj['responseCode'] as number;
      if (code === 401) {
        return { code: ErrorCodes.AuthExpired, message: '登录已过期，请重新登录' };
      }
      if (code >= 500) {
        return { code: ErrorCodes.ServerError, message: '服务器异常，请稍后重试' };
      }
    }

    return {
      code: ErrorCodes.NetworkError,
      message: '网络异常，请检查网络连接',
      original: error
    };
  }
}

// ── viewmodels 中使用 ──
async loadData(): Promise<void> {
  this.isLoading = true;
  try {
    this.data = await DataService.fetch();
  } catch (err) {
    const appError: AppError = ErrorHandler.handle(err, 'loadData');
    this.errorMessage = appError.message;
    if (appError.code === ErrorCodes.AuthExpired) {
      this.navigateToLogin();
    }
  } finally {
    this.isLoading = false;
  }
}
```

---

## 设计模式速查

| 模式 | 场景 | ArkTS 实现 |
|------|------|-----------|
| **单例** | 全局服务、配置管理器 | `static instance: ClassName` + 私有构造 |
| **观察者** | 事件总线、跨组件通知 | `@Watch` 装饰器 |
| **策略** | 多支付方式、多主题 | 接口 + 不同实现类 |
| **适配器** | 第三方 SDK 封装 | Service 层封装 |
| **工厂** | 复杂对象创建 | 静态工厂方法 |
| **Builder** | 复杂 UI 组件构建 | `@Builder` 方法 |

### 单例模式

```typescript
export class ConfigManager {
  private static instanceValue: ConfigManager | null = null;
  private config: Record<string, string> = {};

  private constructor() {}

  static getInstance(): ConfigManager {
    if (ConfigManager.instanceValue === null) {
      ConfigManager.instanceValue = new ConfigManager();
    }
    return ConfigManager.instanceValue;
  }

  get(key: string): string {
    return this.config[key] ?? '';
  }

  set(key: string, value: string): void {
    this.config[key] = value;
  }
}
```

---

## 📚 延伸阅读

- [HarmonyOS 应用模型](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-model-description)
- [Stage 模型开发指导](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/stage-model-development-overview)
- [Navigation 组件](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-navigation)

---

> 📝 **最后更新**: 2026-05-10
> 📌 **适用版本**: HarmonyOS NEXT (API 12)