---
name: harmonyos-dev
description: Comprehensive HarmonyOS 6.0 development skill covering ArkTS, ArkUI, state management, components, layout, animation, routing, storage, network, device interaction, permissions, testing, 3D graphics, AI engine, and more. Use when developing HarmonyOS apps or answering HarmonyOS development questions.
license: MIT
compatibility: opencode
metadata:
  version: "1.1.0"
  sdk: "HarmonyOS 6.0 (API 12)"
  language: ArkTS
---

# HarmonyOS 6.0 Development Skill

Comprehensive development reference for HarmonyOS applications using ArkTS and ArkUI.

## Project File Structure

```
harmonyos-skill/
├── 01-ArkTS-Basic/          # ArkTS Language Basics
├── 02-UI-Development/       # UI Development
├── 03-State-Management/     # State Management
├── 04-Components/           # Components Reference
├── 05-Layout/               # Layout Design
├── 06-Animation/            # Animation Effects
├── 07-Router/               # Router & Navigation
├── 08-Storage/              # Data Storage
├── 09-Network/              # Network Requests
├── 10-Device/               # Device Interaction
├── 11-Permission/           # Permission Management
├── 12-Testing/              # Testing & Debugging
├── 13-3D-Graphics/          # 3D Graphics Rendering
├── 14-AI-Engine/            # AI Engine (HiAI, ML Kit, NLP, Speech)
├── 15-Concurrency/          # Concurrency (TaskPool, Worker)
├── 16-Runtime-and-GC/       # Runtime & Garbage Collection
├── 17-Native-Extension/     # Native Extension (C/C++, NAPI)
├── 18-Security/             # Security Best Practices
├── 19-Version-Differences/  # API Version Differences
├── 20-Project-Engineering/  # Project Engineering
├── 21-App-Architecture/     # Application Architecture
├── 22-Quality-Checklist/    # Quality Checklist
├── examples/                # Complete Example Projects
├── templates/               # Project Templates
└── QUICK-REFERENCE.md       # Quick API Reference
```

## Key Concepts

### Decorative Patterns

| Decorator | Scope | Direction | Use Case |
|-----------|-------|-----------|----------|
| `@State` | Component internal | Self | Internal component state |
| `@Prop` | Parent→Child | One-way | Read-only child props |
| `@Link` | Parent↔Child | Two-way | Two-way binding (use `$`) |
| `@Provide/@Consume` | Ancestor↔Descendant | Cross-level | Skip intermediate components |
| `@Observed/@ObjectLink` | Class instances | Nested | Nested object observation |
| `@StorageLink` | App-wide | Global | AppStorage binding |
| `@StorageProp` | App-wide | One-way | Read-only AppStorage |

### Component Categories

- **Basic**: Text, Span, Image, Button, TextInput, Toggle, Slider, Progress, LoadingProgress
- **Container**: Column, Row, Stack, Flex, Grid, List, Scroll, Tabs, Swiper
- **Media**: Image, Video, XComponent
- **Navigation**: Navigation, NavRouter, TabContent
- **Dialog**: AlertDialog, CustomDialog, Toast, ActionSheet

### Layout Types

- **Linear**: `Column()` (vertical), `Row()` (horizontal)
- **Flex**: `Flex({ direction, justifyContent, alignItems, wrap })`
- **Stack**: `Stack({ alignContent })` — overlap layers
- **Grid**: `Grid()` + `GridItem()` — cell-based grid
- **Responsive**: `GridRow()` + `GridCol({ span })` — responsive 12-col

## Quick Code Snippets

### Entry Component

```typescript
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
        .onClick(() => { this.message = 'Clicked!'; })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### State Management

```typescript
@Component
struct Parent {
  @State count: number = 0;
  @Provide('theme') theme: string = 'light';

  build() {
    Column() {
      Text(`Count: ${this.count}`)
      Child({ value: $count })  // @Link with $
    }
  }
}

@Component
struct Child {
  @Link value: number;
  @Consume('theme') theme: string;

  build() {
    Button('+').onClick(() => { this.value++; })
  }
}
```

### HTTP Request

```typescript
import { http } from '@kit.NetworkKit';

async function fetchData(url: string): Promise<string> {
  const request = http.createHttp();
  try {
    const response = await request.request(url, {
      method: http.RequestMethod.GET,
      header: { 'Content-Type': 'application/json' },
      readTimeout: 15000
    });
    return response.result as string;
  } finally {
    request.destroy();
  }
}
```

### Router Navigation

```typescript
import { router } from '@kit.ArkUI';

// Push with params
router.pushUrl({ url: 'pages/Detail', params: { id: '123' } });

// Back
router.back();

// Get params in target page
const params = router.getParams() as Record<string, string>;
```

### Preferences Storage

```typescript
import { preferences } from '@kit.ArkData';

async function saveSetting(context: Context, key: string, value: string) {
  const store = await preferences.getPreferences(context, 'myStore');
  await store.put(key, value);
  await store.flush();
}

async function loadSetting(context: Context, key: string): Promise<string> {
  const store = await preferences.getPreferences(context, 'myStore');
  return await store.get(key, '') as string;
}
```

### Permissions

```typescript
import { abilityAccessCtrl } from '@kit.AbilityKit';

async function checkAndRequest(permission: string): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  try {
    const status = await atManager.checkAccessToken(getContext().tokenID, permission);
    if (status === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) return true;
    const result = await atManager.requestPermissionsFromUser(getContext(), [permission]);
    return result.authResults[0] === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
  } catch {
    return false;
  }
}
```

### Notification

```typescript
import { notificationManager } from '@kit.NotificationKit';

async function sendNotify(title: string, text: string) {
  await notificationManager.requestEnableNotification();
  await notificationManager.publish({
    id: Date.now(),
    content: {
      notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
      normal: { title, text }
    }
  });
}
```

### 3D Transform

```typescript
Text('3D Card')
  .width(200).height(300)
  .rotate({ x: 0, y: 1, z: 0, angle: 180 })
  .perspective(1000)
```

### ML Kit - Face Detection

```typescript
import { faceDetect } from '@kit.MLKit';

async function detectFaces(imagePath: string) {
  const detector = faceDetect.createFaceDetector({
    maxFaceNum: 10, minFaceSize: 0.1,
    isPose: true, isEyeOpen: true, isSmile: true
  });
  return await detector.detect(imagePath);
}
```

### Text-to-Speech (TTS)

```typescript
import { textToSpeech } from '@kit.HiAIFoundation';

async function speakText(text: string) {
  const synthesizer = textToSpeech.createSynthesizer({
    language: 'zh-CN', voice: 'female', speed: 1.0, pitch: 1.0
  });
  await synthesizer.speak(text);
}
```

### Database CRUD

```typescript
import { relationalStore } from '@kit.ArkData';

const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'mydb.db',
  securityLevel: relationalStore.SecurityLevel.S1
};
const rdbStore = await relationalStore.getRdbStore(context, STORE_CONFIG);

// Insert
await rdbStore.insert('users', { name: 'Alice', age: 25 });

// Query
const predicates = new relationalStore.RdbPredicates('users');
const rs = await rdbStore.query(predicates, ['id', 'name', 'age']);

// Update
const bucket: relationalStore.ValuesBucket = { name: 'Bob' };
const updatePredicates = new relationalStore.RdbPredicates('users');
updatePredicates.equalTo('id', 1);
await rdbStore.update(bucket, updatePredicates);

// Delete
const deletePredicates = new relationalStore.RdbPredicates('users');
deletePredicates.equalTo('id', 1);
await rdbStore.delete(deletePredicates);
```

### Animation Recipes

```typescript
// Implicit animation
animateTo({ duration: 300, curve: Curve.EaseInOut }, () => { this.width = 200; });

// Component transition
Text('Content')
  .transition({ opacity: { from: 0, to: 1 }, scale: { from: { x: 0.8, y: 0.8 } } });

// Continuous rotation
const animator = new Animator({ duration: 3000, iterations: -1, curve: Curve.Linear });
animator.onframe = (progress: number) => { this.angle = progress * 360; };
animator.play();

// 3D Flip card
Text('Card')
  .rotate({ x: 0, y: 1, z: 0, angle: this.rotateY })
  .perspective(1000)
```

### Common Patterns

```typescript
// Lazy loading list
List() {
  LazyForEach(this.dataSource, (item: Item) => {
    ListItem() { Text(item.name) }
  }, (item: Item) => item.id)
}.cachedCount(5)

// Pull to refresh
Refresh({ refreshing: $$this.refreshing }) {
  List() { /* items */ }
}.onRefreshing(() => { this.loadData().then(() => { this.refreshing = false; }); })

// Debounce utility
function debounce(fn: Function, delay: number) {
  let timer: number;
  return (...args: any[]) => { clearTimeout(timer); timer = setTimeout(() => fn(...args), delay); };
}
```

### Sensors

```typescript
import { sensor } from '@kit.SensorServiceKit';

// Accelerometer
sensor.on(sensor.SensorId.ACCELEROMETER, (data) => {
  console.log(`x: ${data.x}, y: ${data.y}, z: ${data.z}`);
}, { interval: 100000000 });
```

### Logger

```typescript
import { hilog } from '@ohos.hilog';

hilog.info(0x0001, 'MyApp', 'User logged in: %{public}s', username);
hilog.error(0x0001, 'MyApp', 'Error: %{public}s', error.message);
```

### Clipboard

```typescript
import { pasteboard } from '@kit.BasicServicesKit';

// Copy
const data = pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, 'text');
await pasteboard.getSystemPasteboard().setData(data);

// Paste
const pd = await pasteboard.getSystemPasteboard().getData();
const text = pd.getPrimaryText();
```

## Concurrency (TaskPool & Worker)

### TaskPool (CPU-intensive short tasks)

```typescript
import { taskpool } from '@kit.ArkTS';

@Concurrent
function heavyTask(input: number): number {
  let result = 0;
  for (let i = 0; i < input * 1000000; i++) { result += i; }
  return result;
}

async function runHeavyTask() {
  try {
    const task = new taskpool.Task(heavyTask, 100);
    const result = await taskpool.execute(task);
    console.log('Task result:', result);
  } catch (error) {
    console.error('Task failed:', error);
  }
}
```

### Worker (long-running background tasks)

```typescript
import { worker } from '@kit.ArkTS';

const workerInstance = new worker.ThreadWorker('entry/ets/workers/MyWorker.ets');

workerInstance.onmessage = (event) => {
  console.log('Worker data:', event.data);
};

workerInstance.postMessage({ type: 'process', data: largeArray });
workerInstance.terminate();
```

## Animation Curves Reference

| Curve | Description |
|-------|-------------|
| `Curve.Linear` | Constant speed |
| `Curve.Ease` | Smooth in/out |
| `Curve.EaseIn` | Slow→Fast |
| `Curve.EaseOut` | Fast→Slow |
| `Curve.EaseInOut` | Slow→Fast→Slow |
| `Curve.FastOutSlowIn` | Quick start, slow end |
| `Curve.Spring` | Spring-like bounce |
| `Curve.CubicBezier(x1,y1,x2,y2)` | Custom bezier |

## Storage Comparison

| Type | Data | Use Case |
|------|------|----------|
| Preferences | Key-Value, small | Settings, config |
| Relational DB | Structured tables | App data, entities |
| Distributed KV | Key-Value, sync | Cross-device data |
| File Storage | Large files | Images, logs, exports |

## When to Use This Skill

- Writing any HarmonyOS application code
- Understanding ArkTS/ArkUI APIs
- Implementing state management patterns
- Adding animations or layouts
- Making network requests or storing data
- Handling permissions or device features
- Working with 3D graphics or AI/ML features
- Debugging or testing HarmonyOS apps
- Answering HarmonyOS development questions

## Best Practices Quick Reference

1. **Minimize state** — keep state close to where it's used, use `@Prop` for read-only
2. **LazyForEach** for long lists, not `ForEach`
3. **Always call `flush()`** after preferences writes
4. **Destroy HTTP request objects** in `finally` blocks
5. **Check permissions before** accessing protected resources
6. **Use single-purpose components** — each component should do one thing well
7. **Animation duration 150-500ms** for UI feedback, avoid long animations
8. **Read detailed module docs** in corresponding `NN-ModuleName/` folders

## GUILLOTINE_RULES

- Always use `@Entry` + `@Component struct` for pages
- Use `$` prefix for `@Link` bindings: `@Link value: number` → `<Child value={$value} />`
- Paginate with `router.pushUrl` or `Navigation`
- Import preferences with `@kit.ArkData`, HTTP with `@kit.NetworkKit`
- Test with `@ohos/hypium` framework
- Hide internal implementation with `private`, expose with `public`
