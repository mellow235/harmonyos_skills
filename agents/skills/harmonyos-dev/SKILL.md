---
name: harmonyos-dev
description: Comprehensive HarmonyOS 6.0 development skill covering ArkTS, ArkUI, state management, components, layout, animation, routing, storage, network, device interaction, permissions, testing, 3D graphics, AI engine, and more. Use when developing HarmonyOS apps or answering HarmonyOS development questions.
license: MIT
compatibility: opencode claude cline cursor windsurf agent
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
├── 14-AI-Engine/            # AI Engine
├── 15-Concurrency/          # Concurrency
├── 16-Runtime-and-GC/       # Runtime & GC
├── 17-Native-Extension/     # Native Extension
├── 18-Security/             # Security
├── 19-Version-Differences/  # Version Differences
├── 20-Project-Engineering/  # Project Engineering
├── 21-App-Architecture/     # App Architecture
├── 22-Quality-Checklist/    # Quality Checklist
├── examples/                # Example Projects
└── templates/               # Project Templates
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
      Text(this.message).fontSize(30).fontWeight(FontWeight.Bold)
      Button('Click Me').onClick(() => { this.message = 'Clicked!'; })
    }.width('100%').height('100%').justifyContent(FlexAlign.Center)
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
      Child({ value: $count })
    }
  }
}
@Component
struct Child {
  @Link value: number;
  @Consume('theme') theme: string;
  build() { Button('+').onClick(() => { this.value++; }) }
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
  } finally { request.destroy(); }
}
```
### Router Navigation
```typescript
import { router } from '@kit.ArkUI';
router.pushUrl({ url: 'pages/Detail', params: { id: '123' } });
router.back();
const params = router.getParams() as Record<string, string>;
```
### Preferences Storage
```typescript
import { preferences } from '@kit.ArkData';
async function saveSetting(context: Context, key: string, value: string) {
  const store = await preferences.getPreferences(context, 'myStore');
  await store.put(key, value); await store.flush();
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
  } catch { return false; }
}
```
### 3D Transform
```typescript
Text('3D Card').width(200).height(300)
  .rotate({ x: 0, y: 1, z: 0, angle: 180 }).perspective(1000)
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
### Text-to-Speech
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
const STORE_CONFIG: relationalStore.StoreConfig = { name: 'mydb.db', securityLevel: relationalStore.SecurityLevel.S1 };
const rdbStore = await relationalStore.getRdbStore(context, STORE_CONFIG);
await rdbStore.insert('users', { name: 'Alice', age: 25 });
const predicates = new relationalStore.RdbPredicates('users');
const rs = await rdbStore.query(predicates, ['id', 'name', 'age']);
```
### Animation Recipes
```typescript
// Implicit animation
animateTo({ duration: 300, curve: Curve.EaseInOut }, () => { this.width = 200; });
// Component transition
Text('Content').transition({ opacity: { from: 0, to: 1 }, scale: { from: { x: 0.8, y: 0.8 } } });
// Continuous rotation
const animator = new Animator({ duration: 3000, iterations: -1, curve: Curve.Linear });
animator.onframe = (progress: number) => { this.angle = progress * 360; };
animator.play();
```
### Common Patterns
```typescript
// Lazy loading list
List() { LazyForEach(this.dataSource, (item: Item) => { ListItem() { Text(item.name) } }, (item: Item) => item.id) }.cachedCount(5)
// Pull to refresh
Refresh({ refreshing: $$this.refreshing }) { List() { } }.onRefreshing(() => { this.loadData().then(() => { this.refreshing = false; }); })
// Debounce
function debounce(fn: Function, delay: number) {
  let timer: number;
  return (...args: any[]) => { clearTimeout(timer); timer = setTimeout(() => fn(...args), delay); };
}
```
## Concurrency
### TaskPool
```typescript
import { taskpool } from '@kit.ArkTS';
@Concurrent
function heavyTask(input: number): number {
  let result = 0;
  for (let i = 0; i < input * 1000000; i++) { result += i; }
  return result;
}
const task = new taskpool.Task(heavyTask, 100);
const result = await taskpool.execute(task);
```
### Worker
```typescript
import { worker } from '@kit.ArkTS';
const workerInstance = new worker.ThreadWorker('entry/ets/workers/MyWorker.ets');
workerInstance.onmessage = (event) => { console.log('Worker data:', event.data); };
workerInstance.postMessage({ type: 'process', data: largeArray });
```
## Animation Curves Reference
| Curve | Description |
|-------|-------------|
| `Curve.Linear` | Constant speed |
| `Curve.Ease` | Smooth in/out |
| `Curve.EaseIn` | Slow→Fast |
| `Curve.EaseOut` | Fast→Slow |
| `Curve.EaseInOut` | Slow→Fast→Slow |
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
## Best Practices Quick Reference
1. Minimize state, use `@Prop` for read-only
2. `LazyForEach` for long lists, not `ForEach`
3. Always call `flush()` after preferences writes
4. Destroy HTTP request objects in `finally` blocks
5. Check permissions before accessing resources
6. Single-purpose components
7. Animation duration 150-500ms
8. Read detailed module docs in `NN-ModuleName/` folders
## GUILLOTINE_RULES
- Always use `@Entry` + `@Component struct` for pages
- Use `$` prefix for `@Link` bindings: `@Link value: number` → `<Child value={$value} />`
- Paginate with `router.pushUrl` or `Navigation`
- Import preferences with `@kit.ArkData`, HTTP with `@kit.NetworkKit`
- Test with `@ohos/hypium` framework
- Hide internal implementation with `private`, expose with `public`
