---
name: harmonyos-skill
description: Comprehensive HarmonyOS 6.0 (API 12) development skill. Covers ArkTS language, ArkUI declarative framework, state management, components, layouts, animations, routing, storage, networking, device APIs, permissions, testing, 3D graphics, AI/ML, concurrency (TaskPool/Worker/Sendable), runtime/GC, native extensions (NAPI), security, version differences (API 9-12), project engineering, app architecture (MVVM), and quality checklists. Use this skill when writing any HarmonyOS application code.
license: MIT
compatibility: opencode claude cline cursor windsurf agent copilot
metadata:
  version: "2.0.0"
  sdk: "HarmonyOS 6.0 (API 12)"
  language: ArkTS
---

# HarmonyOS 6.0 Comprehensive Development Skill

## Core Concept: What is ArkTS?

ArkTS is **NOT** a superset of TypeScript. It is a **strictly constrained subset of TypeScript** with ArkUI declarative UI extensions:

```
ArkTS = (TypeScript 约束子集) + (ArkUI 声明式扩展)

  TypeScript 语言规范
      │
      ├── ArkTS 移除大量 TS 特性（严格约束子集）
      │   ├── 禁止 any/unknown、for...in、解构、函数重载...
      │   ├── 强制静态类型 + AOT 编译
      │   └── = 更小、更确定的语法面
      │
      └── ArkTS 扩展自身独有特性
          ├── 声明式 UI（struct + build()）
          ├── 状态管理装饰器（@State、@Prop...）
          ├── 并发模型（@Sendable、TaskPool）
          └── 平台 API 集成
```

## ArkTS 6.0 (API 12) Key Features

### New in ArkTS 6.0 (API 12)
- **@Sendable 函数**: Functions can be decorated with `@Sendable` for cross-thread reference passing (not just classes)
- **LongTask**: Execute tasks exceeding 3 minutes via `taskpool.LongTask`
- **SendableLruCache**: Shared LRU cache for concurrent scenarios
- **Sendable System Objects**: `SendablePreferences`, `SendableImage`, `SendableContext`, `SendableColorSpaceManager`, `SendableResourceManager`
- **State Management V2**: `@ComponentV2` / `@Local` (replaces `@State`) / `@Param` (replaces `@Prop`) / `@Once`
- **More Sendable native objects** across system kits

### ArkTS 5.0 (API 11) Features
- **@Sendable classes**: Reference-passing between threads
- **Shared containers**: `collections.Array`, `collections.Map`, `collections.Set`
- **AsyncLock / ConditionVariable**: Thread synchronization primitives
- **Strict mode**: `use strict` enforced, `eval()`/`with()` banned
- **Circular dependency detection**

### .ets Compilation Mode
| compatibleSdkVersion | Mode | TS Syntax Behavior |
|---------------------|------|-------------------|
| `< 10` (e.g. 9) | Compatibility Mode | ⚠️ Warning, compilation succeeds |
| `>= 10` (e.g. 12) | Standard Mode | ❌ Error, compilation fails |

## ArkTS vs TypeScript: Complete Differences

### Disabled TS Features (API 10+, Standard Mode)
| Category | TS Feature | ArkTS Alternative |
|----------|-----------|------------------|
| **Types** | `any`/`unknown` | Concrete type or `Object` |
| **Types** | Index signature `[key: string]` | `Record<K,V>` or `Map` |
| **Types** | Intersection type `&` | Interface inheritance |
| **Types** | Conditional type | Overload-style declarations |
| **Types** | Mapped type | Explicit property declarations |
| **Types** | `this` type | Explicit parameter typing |
| **Variables** | `var` | `let`/`const` |
| **Variables** | Destructuring `const {a}=obj` | Direct property access |
| **Variables** | `as const` | Literal type annotation |
| **Objects** | `delete obj.prop` | Assign `undefined` |
| **Objects** | Runtime property addition | Full type declaration upfront |
| **Objects** | Structural typing | Class inheritance or interface impl |
| **Functions** | Function overloads | Optional params + union types |
| **Functions** | Nested function declarations | Lambda expressions |
| **Functions** | Generators `function*` | Arrays/iterators |
| **Classes** | `#` private fields | `private` keyword |
| **Classes** | Class expressions `const C = class{}` | Named classes |
| **Classes** | `implements` clause | `extends` inheritance |
| **Classes** | Field declarations in `constructor` | Top-level class field declarations |
| **Loops** | `for...in` | `for...of` or `Object.keys()` |
| **Symbols** | `Symbol()` | String constants |
| **Modules** | `require()` | `import` statements |
| **Modules** | `export = ...` | `export default` |
| **Global** | `globalThis` | Avoid global objects |
| **Security** | `eval()`/`with()` (API 11+) | Logical alternatives |

### ArkTS Exclusive Features
| Feature | Description | Available Since |
|---------|------------|-----------------|
| Declarative UI | `@Entry @Component struct` + `build()` | API 9 |
| State decorators | `@State`, `@Prop`, `@Link`, `@Provide/@Consume` | API 9 |
| Nested observation | `@Observed/@ObjectLink` | API 9 |
| AppStorage | Application-global state | API 9 |
| `@Sendable` class | Cross-thread reference passing | API 11 |
| `@Sendable` function | Function-level Sendable | API 12 |
| `@ComponentV2` | New V2 component decorator | API 12 |
| `@Local`/`@Param`/`@Once` | V2 state decorators | API 12 |
| `@Concurrent` | TaskPool concurrent function marker | API 9 |
| `@Builder` | Custom UI builder function | API 9 |
| `@Styles` | Reusable style function | API 9 |
| `@Extend` | Extend existing component styles | API 9 |
| `@CustomDialog` | Custom dialog decorator | API 9 |
| `@Watch` | State change watcher | API 9 |

## Module Quick Reference

### 01-ArkTS-Basic
```typescript
// Variables: let for mutable, const for immutable
let count: number = 0;
const MAX: number = 100;

// Types: string, number, boolean, null, undefined, arrays, tuples, enums, unions
type Direction = 'up' | 'down';
let id: string | number = '001';

// Control flow: if/else, switch, for, for...of, while
for (const item of items) { }
for (let i = 0; i < len; i++) { }

// Interfaces & Classes
interface User { id: number; name: string; readonly createdAt: Date; }
class Person {
  private name: string;
  constructor(name: string) { this.name = name; }
  greet(): string { return `Hi, ${this.name}`; }
}

// Generics
function identity<T>(arg: T): T { return arg; }
interface ApiResponse<T> { code: number; data: T; }

// High-performance practices
const arr: Float32Array = new Float32Array([1, 2, 3]);  // TypedArray > number[]
const set: Set<string> = new Set(['a', 'b']);            // Set O(1) > Array.includes O(n)
const map: Map<number, number> = new Map();              // Map > sparse array
```

### 02-UI-Development
```typescript
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS';

  // Lifecycle
  aboutToAppear(): void { /* init */ }
  aboutToDisappear(): void { /* cleanup */ }
  onPageShow(): void { }
  onPageHide(): void { }
  onBackPress(): boolean { return false; }

  build() {
    Column() {
      Text(this.message).fontSize(30).fontWeight(FontWeight.Bold)
      Button('Click').onClick(() => { this.message = 'Clicked!'; })
    }
    .width('100%').height('100%').justifyContent(FlexAlign.Center)
  }
}
```

### 03-State-Management
```typescript
// @State - internal component state
@State count: number = 0;

// @Prop - parent→child one-way
@Prop title: string;

// @Link - parent↔child two-way (use $)
@Link value: number;       // Parent: Child({ value: $count })

// @Provide/@Consume - cross-level sharing
@Provide('theme') theme: string = 'light';  // ancestor
@Consume('theme') theme: string;            // descendant

// @Observed/@ObjectLink - nested object observation
@Observed class Address { city: string = ''; }
@ObjectLink address: Address;

// @Watch - state change listener
@State @Watch('onChange') name: string = '';
onChange(): void { }

// AppStorage - global state
@StorageLink('key') value: string;
AppStorage.setOrCreate<string>('key', 'default');

// PersistentStorage - persisted state
PersistentStorage.persistProp<string>('theme', 'light');

// V2 State Management (API 12+)
@ComponentV2
struct NewStyle {
  @Local count: number = 0;    // replaces @State
  @Param title: string = '';   // replaces @Prop
  @Param @Once value: number;  // replaces @Link (init-only)
}
```

### 04-Components
- **Basic**: `Text`, `Span`, `Image`, `Button`, `TextInput`, `TextArea`, `Toggle`, `Slider`, `Progress`, `LoadingProgress`
- **Container**: `Column`, `Row`, `Stack`, `Flex`, `Grid`, `List`, `Scroll`, `Tabs`, `Swiper`
- **Media**: `Image`, `Video`, `XComponent`
- **Navigation**: `Navigation`, `NavRouter`, `TabContent`
- **Dialog**: `AlertDialog`, `CustomDialog`, `Toast`, `ActionSheet`

```typescript
// Text with Span
Text() { Span('Hello ').fontColor('#000'); Span('World').fontColor('#007DFF'); }
// Image
Image($r('app.media.icon')).width(100).height(100).objectFit(ImageFit.Cover).borderRadius(8)
// Button
Button('Submit').width(200).height(50).backgroundColor('#007DFF').borderRadius(25).type(ButtonType.Capsule)
// TextInput
TextInput({ placeholder: 'Enter' }).width(300).onChange((v: string) => { })
// LoadingProgress
LoadingProgress().width(50).height(50).color('#007DFF')
```

### 05-Layout
```typescript
// Column (vertical)
Column() { /* children */ }.width('100%').justifyContent(FlexAlign.Center).alignItems(HorizontalAlign.Center)
// Row (horizontal)
Row() { /* children */ }.width('100%').justifyContent(FlexAlign.SpaceBetween)
// Stack (overlay)
Stack({ alignContent: Alignment.Center }) { /* children */ }.width(200).height(200)
// Flex
Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap }) { /* children */ }
// Grid
Grid() { GridItem() { Text('A') }; GridItem() { Text('B') } }.columnsTemplate('1fr 1fr').rowsTemplate('1fr 1fr')
// Responsive GridRow/GridCol
GridRow() { GridCol({ span: { sm: 12, md: 6, lg: 4 } }) { Text('Responsive') } }
// List
List() { LazyForEach(this.data, (item: Item) => { ListItem() { Text(item.name) } }, (item) => item.id) }.cachedCount(5)
```

### 06-Animation
```typescript
// Implicit animation
animateTo({ duration: 300, curve: Curve.EaseInOut }, () => { this.width = 200; this.opacity = 0.5; })

// Component animation
Text('Animate').opacity(this.visible ? 1 : 0).animation({ duration: 300 })

// Transition animation
Text('Content').transition({ opacity: { from: 0, to: 1 }, scale: { from: { x: 0.8, y: 0.8 } } })

// Keyframe animation
animateTo({ duration: 1000, iterations: 3, curve: Curve.Spring }, () => { this.scale = { x: 1.2, y: 1.2 }; })

// Continuous rotation
const animator = new Animator({ duration: 3000, iterations: -1, curve: Curve.Linear });
animator.onframe = (p: number) => { this.angle = p * 360; }; animator.play();

// 3D Flip
Text('Card').rotate({ x: 0, y: 1, z: 0, angle: this.rotateY }).perspective(1000)

// Animation Curves: Linear, Ease, EaseIn, EaseOut, EaseInOut, FastOutSlowIn, Spring, CubicBezier(x1,y1,x2,y2)
```

### 07-Router
```typescript
import { router } from '@kit.ArkUI';

// Push with params
router.pushUrl({ url: 'pages/Detail', params: { id: '123' } });
// Back
router.back();
// Replace
router.replaceUrl({ url: 'pages/Home' });
// Get params in target
const params = router.getParams() as Record<string, string>;

// Navigation component (recommended for complex apps)
Navigation() { NavRouter() { Text('Go to Detail').onClick(() => { this.nav.pushPath({ name: 'Detail' }); }) } }
```

### 08-Storage
```typescript
import { preferences } from '@kit.ArkData';
// Preferences (key-value)
const store = await preferences.getPreferences(context, 'myStore');
await store.put('key', 'value'); await store.flush();
const val = await store.get('key', 'default') as string;

import { relationalStore } from '@kit.ArkData';
// Relational DB
const config: relationalStore.StoreConfig = { name: 'db.db', securityLevel: relationalStore.SecurityLevel.S1 };
const rdb = await relationalStore.getRdbStore(context, config);
await rdb.insert('users', { name: 'Alice', age: 25 });
const pred = new relationalStore.RdbPredicates('users'); pred.equalTo('id', 1);
const rs = await rdb.query(pred, ['id', 'name']);

// File Storage
import { fileIo } from '@kit.CoreFileKit';
const file = fileIo.openSync(filePath, fileIo.OpenMode.READ_WRITE | fileIo.OpenMode.CREATE);
fileIo.writeSync(file.fd, 'data');

// Distributed KV Store (cross-device)
import { distributedKVStore } from '@kit.DistributedServiceKit';
```

### 09-Network
```typescript
import { http } from '@kit.NetworkKit';
async function fetchData(url: string): Promise<string> {
  const req = http.createHttp();
  try {
    const resp = await req.request(url, { method: http.RequestMethod.GET, header: { 'Content-Type': 'application/json' }, readTimeout: 15000 });
    return resp.result as string;
  } finally { req.destroy(); }
}

import { webSocket } from '@kit.NetworkKit';
// WebSocket
const ws = webSocket.createWebSocket();
ws.on('message', (data: string | ArrayBuffer) => { });
ws.connect('wss://example.com/ws', () => { ws.send('Hello'); });

// Data parsing
interface ApiResponse { code: number; data: Record<string, Object>; }
const json: ApiResponse = JSON.parse(responseString);
```

### 10-Device
```typescript
import { pasteboard } from '@kit.BasicServicesKit';
// Clipboard
await pasteboard.getSystemPasteboard().setData(pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, 'text'));
const pd = await pasteboard.getSystemPasteboard().getData(); const text = pd.getPrimaryText();

import { sensor } from '@kit.SensorServiceKit';
// Sensors
sensor.on(sensor.SensorId.ACCELEROMETER, (data) => { console.log(`x:${data.x}, y:${data.y}, z:${data.z}`); }, { interval: 100000000 });

import { notificationManager } from '@kit.NotificationKit';
// Notification
await notificationManager.requestEnableNotification();
await notificationManager.publish({ id: Date.now(), content: { notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT, normal: { title: 'Title', text: 'Body' } } });

import { screen } from '@kit.ArkUI';
// Screen info
const screens = await screen.getAllScreens(); const dpi = screens[0].dpi;

import { window } from '@kit.ArkUI';
// Window management
const win = window.findWindow('myWindow'); await win.setWindowSystemBarEnable([]);

import { brightness } from '@kit.BrightnessKit';
import { bluetooth } from '@kit.ConnectivityKit';
import { wifiManager } from '@kit.ConnectivityKit';
import { geoLocationManager } from '@kit.LocationKit';
```

### 11-Permission
```typescript
import { abilityAccessCtrl } from '@kit.AbilityKit';
import { bundleManager } from '@kit.AbilityKit';

// Check and request permission
async function checkAndRequest(perm: string): Promise<boolean> {
  const atMgr = abilityAccessCtrl.createAtManager();
  try {
    const status = await atMgr.checkAccessToken(getContext().tokenID, perm);
    if (status === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) return true;
    const result = await atMgr.requestPermissionsFromUser(getContext(), [perm]);
    return result.authResults[0] === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
  } catch { return false; }
}

// Common permissions: ohos.permission.CAMERA, ohos.permission.MICROPHONE, ohos.permission.LOCATION, ohos.permission.READ_MEDIA
```

### 12-Testing
```typescript
import { describe, it, expect } from '@ohos/hypium';
import { UIAbility } from '@kit.AbilityKit';

describe('MyTest', () => {
  it('should pass', 0, () => {
    expect(1 + 1).assertEqual(2);
    expect(true).assertTrue();
    expect('hello').assertContain('ell');
  });
});

// UI testing
import { Driver } from '@ohos.UiTest';
const driver = await Driver.create();
await driver.findComponent({ text: 'Submit' }).click();
```

### 13-3D-Graphics
```typescript
import { XComponent } from '@kit.ArkUI';

// XComponent for 3D rendering
XComponent({ type: 'surface', controller: this.xcController })
  .onLoad(() => { /* OpenGL ES context ready */ })
  .width('100%').height('100%')

// 3D transforms on any component
Text('3D')
  .rotate({ x: 0, y: 1, z: 0, angle: 45 })
  .perspective(1000)
  .scale({ x: 1.5, y: 1.5 })

// OpenGL ES integration via native C++ (see 17-Native-Extension)
```

### 14-AI-Engine
```typescript
import { faceDetect } from '@kit.MLKit';
// Face detection
const detector = faceDetect.createFaceDetector({ maxFaceNum: 10, isPose: true, isSmile: true });
const faces = await detector.detect(imagePath);

import { textToSpeech } from '@kit.HiAIFoundation';
// Text-to-Speech
const tts = textToSpeech.createSynthesizer({ language: 'zh-CN', voice: 'female' });
await tts.speak('Hello');

import { speechRecognizer } from '@kit.HiAIFoundation';
// Speech recognition
const recognizer = speechRecognizer.createRecognizer({ language: 'zh-CN' });
recognizer.on('result', (text: string) => { });
await recognizer.start();

import { textAnalyze } from '@kit.MLKit';
// NLP: sentiment analysis, entity extraction
// Image classification, object detection via ML Kit
```

### 15-Concurrency
```typescript
// --- Async/Await (single-thread) ---
async function fetchUser(id: number): Promise<User> {
  const resp = await api.getUser(id);
  return resp.data;
}

// --- TaskPool (CPU-intensive, recommended) ---
import { taskpool } from '@kit.ArkTS';
@Concurrent
function heavyCalc(input: number): number {
  let r = 0; for (let i = 0; i < input * 1000000; i++) { r += i; } return r;
}
const task = new taskpool.Task(heavyCalc, 100);
const result = await taskpool.execute(task);

// --- LongTask (API 12+, >3 min tasks) ---
const longTask = new taskpool.LongTask(task);
await taskpool.execute(longTask);

// --- Worker (long-running background) ---
import { worker } from '@kit.ArkTS';
const workerInst = new worker.ThreadWorker('entry/ets/workers/MyWorker.ets');
workerInst.onmessage = (e) => { console.log('Worker:', e.data); };
workerInst.postMessage({ type: 'process', data: arr });
workerInst.terminate();

// --- @Sendable class (API 11+, reference passing) ---
@Sendable
class SharedData { count: number = 0; increment(): void { this.count++; } }

// --- @Sendable function (API 12+) ---
@Sendable
function sendableFn(): void { console.info('Sendable function'); }

// --- Shared containers (API 11+) ---
import { collections } from '@kit.ArkTS';
const sharedArr = collections.Array.create<number>(10, 0);
const sharedMap = new collections.Map<string, number>();

// --- AsyncLock (API 11+) ---
import { ArkTSUtils } from '@kit.ArkTS';
const lock = new ArkTSUtils.locks.AsyncLock();
await lock.lockAsync(() => { sharedArr[0]++; });
```

### 16-Runtime-and-GC
```typescript
// Ark Compiler: AOT (Ahead-of-Time) compilation
// HPP GC (High-Performance Partial Garbage Collection):
//   - Concurrent marking (no full STW pause)
//   - Generational collection (young/old gen)
//   - Memory optimization strategies:
//     * Use const for compile-time constant folding
//     * Use TypedArray (Float32Array, Int32Array) for numeric data
//     * Avoid sparse arrays (use Map instead)
//     * Extract loop invariants
//     * Prefer array literals over new Array() + push
//     * Avoid try/catch for business logic

// Memory leak prevention:
//   - Destroy HTTP requests in finally blocks
//   - Unregister sensor/event listeners in aboutToDisappear
//   - Clean up timers/intervals
//   - Use LazyForEach instead of ForEach for large lists
```

### 17-Native-Extension
```typescript
// Node-API (NAPI): C/C++ ↔ ArkTS interop
// Native module structure:
//   entry/src/main/cpp/
//   ├── CMakeLists.txt
//   ├── hello.cpp        // Native implementation
//   └── types/libhello/  // TypeScript declarations

// C++ example (hello.cpp):
// #include "napi/native_api.h"
// static napi_value Add(napi_env env, napi_callback_info info) {
//   size_t argc = 2; napi_value args[2];
//   napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);
//   double a, b; napi_get_value_double(env, args[0], &a); napi_get_value_double(env, args[1], &b);
//   napi_value sum; napi_create_double(env, a + b, &sum); return sum;
// }

// ArkTS import:
// import nativeAdd from 'libhello.so';
// const sum = nativeAdd(1.5, 2.5);

// @Sendable native objects (API 12+): cross-thread C++ objects
```

### 18-Security
```typescript
// ArkGuard - Code obfuscation (build-profile.json5)
// {
//   "arkOptions": {
//     "obfuscation": {
//       "ruleOptions": [ { "enable": true, "files": ["obfuscation-rules.txt"] } ]
//     }
//   }
// }

// Data encryption
import { cryptoFramework } from '@kit.CryptoArchitectureKit';
// AES-GCM encryption, RSA signing, SHA-256 hashing

// App signing: .p7b certificate + .p12 private key
// Network security: always use HTTPS, validate certificates
// Input validation: sanitize all user inputs
// Permission best practices: minimum privilege principle
// Secure storage: use preferences or encrypted DB
```

### 19-Version-Differences
| Version | API Level | Key Changes |
|---------|-----------|-------------|
| ArkTS 3.0 | API 9 | TS fully compatible, no strict checks |
| ArkTS 4.0 | API 10 | Mandatory static types, ban `any`/`unknown`, compatibleSdkVersion check |
| ArkTS 5.0 | API 11 | `@Sendable` classes, shared containers, AsyncLock, ban `eval()`/`with()` |
| ArkTS 6.0 | API 12 | `@Sendable` functions, LongTask, State Mgmt V2, Sendable system objects |

**Migration path**: API 9 → 10: fix 20+ disabled TS features. API 10 → 11: add Sendable/AsyncLock. API 11 → 12: adopt V2 state, LongTask.

### 20-Project-Engineering
```json5
// build-profile.json5 (project-level)
{
  "app": {
    "products": [{
      "name": "default",
      "compatibleSdkVersion": 12,
      "runtimeOS": "HarmonyOS",
      "buildOption": { "arkOptions": { "obfuscation": { "ruleOptions": [] } } }
    }]
  }
}
```
```
Project structure:
  AppScope/          → app.json5 (app name, icon, vendor)
  entry/             → main module (HAP)
    build-profile.json5  → module config
    src/main/
      ets/           → ArkTS source
        entryability/EntryAbility.ets
        pages/Index.ets
      resources/     → base/element/string.json, media/, profile/
      module.json5   → module metadata
  oh-package.json5   → dependency management
  hvigorfile.ts      → build script
```

### 21-App-Architecture (MVVM)
```
┌─────────────────────────────────────┐
│  View (Pages/Components)            │
│  - Declarative UI rendering         │
│  - User event bindings              │
│  - Consumes ViewModel state         │
├─────────────────────────────────────┤
│  ViewModel (viewmodels/)            │
│  - Holds UI state (@State sources)  │
│  - Calls Service layer              │
│  - Business logic & transformation  │
├─────────────────────────────────────┤
│  Service (services/)                │
│  - Network/DB/cache encapsulation   │
│  - Third-party SDK wrappers         │
├─────────────────────────────────────┤
│  Model (models/)                    │
│  - Data interfaces & types          │
│  - Enums & constants                │
└─────────────────────────────────────┘
```

### 22-Quality-Checklist
**ArkTS syntax (13 checks)**:
- [ ] No `any`/`unknown` types
- [ ] No `var` declarations
- [ ] No `for...in` loops
- [ ] No destructuring assignments
- [ ] No `#` private fields (use `private`)
- [ ] No `Symbol()` usage
- [ ] No function overloads
- [ ] No runtime object property additions
- [ ] No `delete` operator
- [ ] No `eval()`/`with()` (API 11+)
- [ ] No `implements` clause (use `extends`)
- [ ] Class properties initialized at declaration
- [ ] Proper @kit import paths

**Performance**: LazyForEach for long lists, flush() after preferences writes, destroy HTTP in finally, animation 150-500ms

## API Import Path Reference
| Kit | Import Path |
|-----|------------|
| ArkUI | `@kit.ArkUI` |
| Ability | `@kit.AbilityKit` |
| Network | `@kit.NetworkKit` |
| Data (Preferences, RDB) | `@kit.ArkData` |
| Concurrency | `@kit.ArkTS` |
| Device (Sensor) | `@kit.SensorServiceKit` |
| Device (Clipboard) | `@kit.BasicServicesKit` |
| Notification | `@kit.NotificationKit` |
| ML Kit | `@kit.MLKit` |
| HiAI | `@kit.HiAIFoundation` |
| Logging | `@kit.PerformanceAnalysisKit` |
| Crypto | `@kit.CryptoArchitectureKit` |
| File I/O | `@kit.CoreFileKit` |
| Location | `@kit.LocationKit` |
| Bluetooth/WiFi | `@kit.ConnectivityKit` |
| Testing | `@ohos.hypium` |
| UI Test | `@ohos.UiTest` |

## GUILLOTINE_RULES
- Pages use `@Entry @Component struct PageName { }`
- Always use `$` prefix for @Link bindings
- Import from `@kit.*` namespace (not `@ohos.*` except hypium)
- Call `flush()` after every preferences write
- Destroy HTTP request objects in `finally` blocks
- Check permissions before accessing protected resources
- Use `LazyForEach` for long lists instead of `ForEach`
- Animate with `animateTo()` or `.animation()` property
- Use `private` instead of `#` for private fields
- Use `let`/`const` instead of `var`
- Use `for...of` or `Object.keys()` instead of `for...in`
- Use direct property access instead of destructuring
- Always specify types explicitly (no `any`)
