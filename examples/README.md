# 完整示例项目

> 📱 鸿蒙应用开发完整示例

## 示例项目列表

### 1. Todo 待办应用
**难度**: ⭐⭐  
**涉及技术**: 状态管理、列表、组件、动画

```typescript
// pages/TodoList.ets
@Entry
@Component
struct TodoList {
  @State todos: Todo[] = [];
  @State newTodo: string = '';

  build() {
    Column() {
      // 输入区域
      Row() {
        TextInput({ text: this.newTodo })
          .layoutWeight(1)
          .onChange((value: string) => {
            this.newTodo = value;
          })
        Button('Add')
          .onClick(() => {
            if (this.newTodo.trim()) {
              this.todos.push({
                id: Date.now(),
                text: this.newTodo,
                completed: false
              });
              this.newTodo = '';
            }
          })
      }
      .padding(16)

      // 列表
      List({ space: 8 }) {
        ForEach(this.todos, (todo: Todo) => {
          ListItem() {
            this.buildTodoItem(todo)
          }
        }, (todo: Todo) => todo.id.toString())
      }
      .layoutWeight(1)
      .padding({ left: 16, right: 16 })
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  buildTodoItem(todo: Todo) {
    Row() {
      Checkbox()
        .select(todo.completed)
        .onChange((value: boolean) => {
          todo.completed = value;
          this.todos = [...this.todos];
        })
      Text(todo.text)
        .fontSize(16)
        .decoration({ type: todo.completed ? TextDecorationType.LineThrough : TextDecorationType.None })
        .layoutWeight(1)
      Button('Delete')
        .fontSize(12)
        .onClick(() => {
          this.todos = this.todos.filter(t => t.id !== todo.id);
        })
    }
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
  }
}

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}
```

---

### 2. 天气应用
**难度**: ⭐⭐⭐  
**涉及技术**: 网络请求、动画、组件

```typescript
// pages/Weather.ets
@Entry
@Component
struct WeatherPage {
  @State weather: WeatherData | null = null;
  @State loading: boolean = true;

  aboutToAppear() {
    this.loadWeather();
  }

  async loadWeather() {
    try {
      const response = await fetch('https://api.weather.com/current');
      this.weather = await response.json();
    } finally {
      this.loading = false;
    }
  }

  build() {
    Column() {
      if (this.loading) {
        LoadingProgress()
          .width(50)
          .height(50)
      } else if (this.weather) {
        // 温度
        Text(`${this.weather.temp}°C`)
          .fontSize(64)
          .fontWeight(FontWeight.Light)
        
        // 天气描述
        Text(this.weather.description)
          .fontSize(20)
          .fontColor('#666')
        
        // 详细信息
        Row() {
          this.buildInfoItem('Humidity', `${this.weather.humidity}%`)
          this.buildInfoItem('Wind', `${this.weather.wind}km/h`)
          this.buildInfoItem('UV', `${this.weather.uv}`)
        }
        .margin({ top: 40 })
      }
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .backgroundImage($r('app.media.weather_bg'))
    .backgroundImageSize(ImageSize.Cover)
  }

  @Builder
  buildInfoItem(label: string, value: string) {
    Column() {
      Text(value)
        .fontSize(24)
        .fontWeight(FontWeight.Medium)
      Text(label)
        .fontSize(14)
        .fontColor('#999')
    }
    .padding(16)
  }
}

interface WeatherData {
  temp: number;
  description: string;
  humidity: number;
  wind: number;
  uv: number;
}
```

---

### 3. 新闻阅读应用
**难度**: ⭐⭐⭐  
**涉及技术**: 路由、列表、图片、网络

```typescript
// pages/NewsList.ets
@Entry
@Component
struct NewsListPage {
  @State news: NewsItem[] = [];
  @State refreshing: boolean = false;

  aboutToAppear() {
    this.loadNews();
  }

  async loadNews() {
    // 加载新闻数据
  }

  build() {
    Column() {
      // 顶部标题
      Text('News')
        .fontSize(28)
        .fontWeight(FontWeight.Bold)
        .padding(16)

      Refresh({ refreshing: $$this.refreshing }) {
        List({ space: 12 }) {
          ForEach(this.news, (item: NewsItem) => {
            ListItem() {
              this.buildNewsItem(item)
            }
          }, (item: NewsItem) => item.id.toString())
        }
        .padding({ left: 16, right: 16 })
      }
      .onRefreshing(() => {
        this.loadNews().then(() => {
          this.refreshing = false;
        });
      })
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }

  @Builder
  buildNewsItem(item: NewsItem) {
    Row() {
      Image(item.cover)
        .width(100)
        .height(80)
        .borderRadius(8)
        .objectFit(ImageFit.Cover)
      Column() {
        Text(item.title)
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
          .maxLines(2)
          .textOverflow({ overflow: TextOverflow.Ellipsis })
        Text(item.source)
          .fontSize(12)
          .fontColor('#999')
          .margin({ top: 8 })
      }
      .layoutWeight(1)
      .margin({ left: 12 })
    }
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(12)
    .onClick(() => {
      router.pushUrl({
        url: 'pages/Detail',
        params: { id: item.id }
      })
    })
  }
}

interface NewsItem {
  id: string;
  title: string;
  cover: string;
  source: string;
}
```

---

### 4. 个人资料页
**难度**: ⭐⭐  
**涉及技术**: 布局、组件、样式

```typescript
// pages/Profile.ets
@Entry
@Component
struct ProfilePage {
  @State user: User = {
    name: 'Alice',
    avatar: 'https://example.com/avatar.jpg',
    bio: 'HarmonyOS Developer',
    followers: 1234,
    following: 567
  };

  build() {
    Column() {
      // 头部
      Column() {
        Image(this.user.avatar)
          .width(80)
          .height(80)
          .borderRadius(40)
          .margin({ top: 20 })
        Text(this.user.name)
          .fontSize(24)
          .fontWeight(FontWeight.Bold)
          .margin({ top: 12 })
        Text(this.user.bio)
          .fontSize(14)
          .fontColor('#666')
          .margin({ top: 8 })
      }
      .width('100%')
      .padding(20)
      .backgroundColor('#FFFFFF')

      // 统计
      Row() {
        this.buildStatItem('Followers', this.user.followers)
        Divider().height(30)
        this.buildStatItem('Following', this.user.following)
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceAround)
      .padding(20)
      .backgroundColor('#FFFFFF')
      .margin({ top: 12 })

      // 菜单
      Column() {
        this.buildMenuItem('My Posts')
        this.buildMenuItem('My Favorites')
        this.buildMenuItem('Settings')
      }
      .margin({ top: 12 })
      .backgroundColor('#FFFFFF')
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }

  @Builder
  buildStatItem(label: string, count: number) {
    Column() {
      Text(count.toString())
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      Text(label)
        .fontSize(12)
        .fontColor('#999')
    }
  }

  @Builder
  buildMenuItem(title: string) {
    Row() {
      Text(title)
        .fontSize(16)
      Blank()
      Image($r('app.media.arrow_right'))
        .width(16)
        .height(16)
    }
    .padding(16)
    .width('100%')
    .onClick(() => {
      // 跳转
    })
  }
}
```

---

### 5. 购物车应用
**难度**: ⭐⭐⭐  
**涉及技术**: 状态管理、列表、计算

```typescript
// pages/Cart.ets
@Entry
@Component
struct CartPage {
  @State cartItems: CartItem[] = [];
  @State selectedAll: boolean = false;

  get totalPrice(): number {
    return this.cartItems
      .filter(item => item.selected)
      .reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  get selectedCount(): number {
    return this.cartItems.filter(item => item.selected).length;
  }

  build() {
    Column() {
      // 列表
      List({ space: 8 }) {
        ForEach(this.cartItems, (item: CartItem) => {
          ListItem() {
            this.buildCartItem(item)
          }
        }, (item: CartItem) => item.id.toString())
      }
      .layoutWeight(1)
      .padding(16)

      // 底部结算
      Row() {
        Checkbox()
          .select(this.selectedAll)
          .onChange((value: boolean) => {
            this.selectedAll = value;
            this.cartItems.forEach(item => item.selected = value);
          })
        Text('All')
          .margin({ left: 8 })
        Blank()
        Column() {
          Text(`Total: $${this.totalPrice.toFixed(2)}`)
            .fontSize(18)
            .fontWeight(FontWeight.Bold)
          Text(`${this.selectedCount} items selected`)
            .fontSize(12)
            .fontColor('#999')
        }
        Button('Checkout')
          .margin({ left: 16 })
          .onClick(() => {
            // 结算逻辑
          })
      }
      .padding(16)
      .backgroundColor('#FFFFFF')
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  buildCartItem(item: CartItem) {
    Row() {
      Checkbox()
        .select(item.selected)
        .onChange((value: boolean) => {
          item.selected = value;
        })
      Image(item.image)
        .width(80)
        .height(80)
        .borderRadius(8)
      Column() {
        Text(item.name)
          .fontSize(16)
        Text(`$${item.price}`)
          .fontSize(14)
          .fontColor('#E74C3C')
          .margin({ top: 4 })
        Row() {
          Button('-')
            .width(30)
            .height(30)
            .onClick(() => {
              if (item.quantity > 1) item.quantity--;
            })
          Text(item.quantity.toString())
            .margin({ left: 8, right: 8 })
          Button('+')
            .width(30)
            .height(30)
            .onClick(() => {
              item.quantity++;
            })
        }
        .margin({ top: 8 })
      }
      .layoutWeight(1)
      .margin({ left: 12 })
    }
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
  }
}

interface CartItem {
  id: string;
  name: string;
  image: string;
  price: number;
  quantity: number;
  selected: boolean;
}
```

---

## 运行示例

1. 在 DevEco Studio 中创建新项目
2. 复制代码到对应文件
3. 运行项目

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
