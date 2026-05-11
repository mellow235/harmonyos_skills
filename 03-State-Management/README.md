# 状态管理

> 🔄 数据驱动 UI 更新的核心机制

## 📖 目录

1. [状态管理概述](#状态管理概述)
2. [组件级状态](#组件级状态)
3. [应用级状态](#应用级状态)
4. [状态管理最佳实践](#状态管理最佳实践)
5. [常见问题](#常见问题)

---

## 状态管理概述

状态管理是 ArkUI 的核心特性，通过装饰器实现数据与 UI 的双向绑定。

### 状态分类

```
状态管理
├── 组件级状态
│   ├── @State - 组件内状态
│   ├── @Prop - 父子单向同步
│   ├── @Link - 父子双向同步
│   ├── @Provide/@Consume - 跨组件共享
│   └── @Observed/@ObjectLink - 嵌套对象
├── 应用级状态
│   ├── AppStorage - 应用全局状态
│   ├── PersistentStorage - 持久化状态
│   └── Environment - 环境状态
└── 自定义状态管理
    └── 状态管理库集成
```

---

## 组件级状态

### @State - 组件内状态

```typescript
@Component
struct Counter {
  @State count: number = 0;
  @State message: string = 'Hello';

  build() {
    Column() {
      Text(`Count: ${this.count}`)
        .fontSize(24)
      
      Button('Increment')
        .onClick(() => {
          this.count++;  // 状态变化自动更新 UI
        })
      
      Text(this.message)
        .fontSize(18)
      
      Button('Change Message')
        .onClick(() => {
          this.message = 'World';
        })
    }
  }
}
```

**@State 规则：**
- 只能在 @Component 内使用
- 状态变化触发 build 重新执行
- 支持基础类型和对象类型

### @Prop - 父子单向同步

```typescript
// 父组件
@Component
struct Parent {
  @State title: string = 'Parent Title';

  build() {
    Column() {
      Text(`Parent: ${this.title}`)
      ChildComponent({ title: this.title })
      Button('Change Title')
        .onClick(() => {
          this.title = 'New Title';
        })
    }
  }
}

// 子组件
@Component
struct ChildComponent {
  @Prop title: string = '';  // 单向接收

  build() {
    Text(`Child: ${this.title}`)
      .fontSize(18)
  }
}
```

**@Prop 特点：**
- 父组件数据变化 → 子组件更新
- 子组件修改 → 不影响父组件
- 适合只读场景

### @Link - 父子双向同步

```typescript
// 父组件
@Component
struct Parent {
  @State value: number = 0;

  build() {
    Column() {
      Text(`Parent Value: ${this.value}`)
      ChildComponent({ value: $value })  // 使用 $ 传递引用
    }
  }
}

// 子组件
@Component
struct ChildComponent {
  @Link value: number;  // 双向绑定

  build() {
    Column() {
      Text(`Child Value: ${this.value}`)
      Button('Increment in Child')
        .onClick(() => {
          this.value++;  // 修改会影响父组件
        })
    }
  }
}
```

**@Link 特点：**
- 父子双向同步
- 使用 `$` 传递引用
- 适合表单等交互场景

### @Provide/@Consume - 跨组件共享

```typescript
// 顶层组件
@Component
struct RootComponent {
  @Provide('theme') theme: string = 'light';
  @Provide('user') user: User = { name: 'Alice', role: 'admin' };

  build() {
    Column() {
      MiddleComponent()
      Button('Toggle Theme')
        .onClick(() => {
          this.theme = this.theme === 'light' ? 'dark' : 'light';
        })
    }
  }
}

// 中间组件（无需传递）
@Component
struct MiddleComponent {
  build() {
    Column() {
      DeepComponent()
    }
  }
}

// 深层组件
@Component
struct DeepComponent {
  @Consume('theme') theme: string;
  @Consume('user') user: User;

  build() {
    Column() {
      Text(`Theme: ${this.theme}`)
      Text(`User: ${this.user.name}`)
    }
  }
}
```

**@Provide/@Consume 特点：**
- 跨多层组件共享
- 无需逐层传递
- 使用字符串 key 区分

### @Observed/@ObjectLink - 嵌套对象

```typescript
// 嵌套对象类
@Observed
class Address {
  city: string;
  street: string;

  constructor(city: string, street: string) {
    this.city = city;
    this.street = street;
  }
}

@Observed
class User {
  name: string;
  address: Address;

  constructor(name: string, address: Address) {
    this.name = name;
    this.address = address;
  }
}

// 使用
@Component
struct UserComponent {
  @ObjectLink user: User;

  build() {
    Column() {
      Text(this.user.name)
      Text(this.user.address.city)
    }
  }
}

@Component
struct Parent {
  @State user: User = new User('Alice', new Address('Beijing', 'Main St'));

  build() {
    Column() {
      UserComponent({ user: this.user })
      Button('Change City')
        .onClick(() => {
          this.user.address.city = 'Shanghai';  // 嵌套属性变化也会触发更新
        })
    }
  }
}
```

---

## 应用级状态

### AppStorage - 应用全局状态

```typescript
// 定义全局状态键
const STORAGE_KEYS = {
  USER_INFO: 'userInfo',
  THEME: 'theme',
  LANGUAGE: 'language'
};

// 页面使用
@Entry
@Component
struct HomePage {
  @StorageLink(STORAGE_KEYS.USER_INFO) userInfo: UserInfo = {};
  @StorageLink(STORAGE_KEYS.THEME) theme: string = 'light';

  build() {
    Column() {
      Text(`Welcome, ${this.userInfo.name}`)
      Text(`Theme: ${this.theme}`)
    }
  }
}

// 工具函数
function initAppStorage(): void {
  AppStorage.setOrCreate<UserInfo>(STORAGE_KEYS.USER_INFO, {
    name: '',
    role: 'guest'
  });
  AppStorage.setOrCreate<string>(STORAGE_KEYS.THEME, 'light');
}
```

### PersistentStorage - 持久化状态

```typescript
// 持久化状态（应用重启后保留）
PersistentStorage.persistProp<string>('theme', 'light');
PersistentStorage.persistProp<UserInfo>('userInfo', {});

@Entry
@Component
struct SettingsPage {
  @StorageLink('theme') theme: string = 'light';

  build() {
    Column() {
      Text(`Current Theme: ${this.theme}`)
      Button('Toggle Theme')
        .onClick(() => {
          this.theme = this.theme === 'light' ? 'dark' : 'light';
          // 自动持久化，无需手动保存
        })
    }
  }
}
```

### Environment - 环境状态

```typescript
@Entry
@Component
struct EnvironmentDemo {
  @StorageProp('colorMode') colorMode: string = 'light';
  @StorageProp('fontScale') fontScale: number = 1.0;

  build() {
    Column() {
      Text(`Color Mode: ${this.colorMode}`)
      Text(`Font Scale: ${this.fontScale}`)
    }
  }
}

// 初始化环境状态
function initEnvironment(): void {
  Environment.envProp('colorMode', 'light');
  Environment.envProp('fontScale', 1.0);
}
```

---

## 状态管理最佳实践

### 1. 选择合适的装饰器

```
场景 → 装饰器选择
├── 组件内部状态 → @State
├── 父子单向传递 → @Prop
├── 父子双向绑定 → @Link
├── 跨组件共享 → @Provide/@Consume
├── 嵌套对象 → @Observed/@ObjectLink
├── 全局状态 → AppStorage
└── 持久化状态 → PersistentStorage
```

### 2. 状态最小化

```typescript
// ✅ 最小化状态
@Component
struct UserProfile {
  @State isLoading: boolean = false;
  @Prop user: User;  // 从父组件接收

  build() {
    if (this.isLoading) {
      LoadingProgress()
    } else {
      Text(this.user.name)
    }
  }
}

// ❌ 过度状态
@Component
struct UserProfile {
  @State user: User;
  @State isLoading: boolean = false;
  @State error: string = '';
  @State cache: any = {};  // 不必要的状态
}
```

### 3. 使用 @Watch 监听变化

```typescript
@Component
struct SearchInput {
  @State @Watch('onSearchChange') keyword: string = '';
  @State results: SearchResult[] = [];

  onSearchChange(): void {
    // 关键词变化时触发搜索
    this.performSearch(this.keyword);
  }

  async performSearch(keyword: string): Promise<void> {
    this.results = await searchAPI(keyword);
  }

  build() {
    Column() {
      TextInput({ text: this.keyword })
        .onChange((value: string) => {
          this.keyword = value;
        })
      List() {
        ForEach(this.results, (result: SearchResult) => {
          ListItem() {
            Text(result.title)
          }
        })
      }
    }
  }
}
```

### 4. 状态提升

```typescript
// ✅ 状态提升到共同父组件
@Component
struct TodoApp {
  @State todos: Todo[] = [];
  @State filter: 'all' | 'active' | 'completed' = 'all';

  build() {
    Column() {
      TodoInput({ onAdd: (text: string) => {
        this.todos = [...this.todos, { text, completed: false }];
      }})
      TodoList({
        todos: this.todos,
        filter: this.filter,
        onToggle: (index: number) => {
          this.todos[index].completed = !this.todos[index].completed;
        }
      })
      TodoFilter({
        filter: $filter
      })
    }
  }
}
```

---

## 常见问题

### Q1: @State 对象属性变化不触发更新？

**A:** 使用展开运算符创建新对象：
```typescript
// ❌ 不触发更新
this.user.name = 'New Name';

// ✅ 触发更新
this.user = { ...this.user, name: 'New Name' };
```

### Q2: 如何在组件外访问状态？

**A:** 使用 AppStorage：
```typescript
// 在组件外访问
const theme = AppStorage.get<string>('theme');

// 在组件内访问
@StorageLink('theme') theme: string;
```

### Q3: @Prop 和 @Link 何时使用？

**A:**
- `@Prop`：子组件只读，不需要修改父组件数据
- `@Link`：子组件需要修改，且要同步到父组件

---

## 📚 延伸阅读

- [状态管理概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-state-management-overview)
- [管理组件拥有的状态](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-state-management-components)
- [管理应用拥有的状态](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-state-management-application)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
