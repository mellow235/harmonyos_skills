# 设备交互

> 📱 系统能力与设备功能

## 📖 目录

1. [设备交互概述](#设备交互概述)
2. [剪贴板](#剪贴板)
3. [屏幕管理](#屏幕管理)
4. [传感器](#传感器)
5. [通知](#通知)
6. [设备信息](#设备信息)
7. [常见问题](#常见问题)

---

## 设备交互概述

HarmonyOS 提供丰富的系统 API，让应用能够与设备硬件和系统功能交互。

### 系统能力

```
设备交互
├── 剪贴板
├── 屏幕管理
│   ├── 屏幕亮度
│   ├── 屏幕方向
│   └── 全屏模式
├── 传感器
│   ├── 加速度计
│   ├── 陀螺仪
│   └── 光线传感器
├── 通知
│   ├── 本地通知
│   └── 通知管理
└── 设备信息
    ├── 设备型号
    ├── 系统版本
    └── 网络状态
```

---

## 剪贴板

### 基础用法

```typescript
import { pasteboard } from '@kit.BasicServicesKit';

// 复制文本
async function copyText(text: string): Promise<void> {
  const pasteData = pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, text);
  const systemPasteboard = pasteboard.getSystemPasteboard();
  await systemPasteboard.setData(pasteData);
}

// 粘贴文本
async function pasteText(): Promise<string> {
  const systemPasteboard = pasteboard.getSystemPasteboard();
  const pasteData = await systemPasteboard.getData();
  return pasteData.getPrimaryText();
}

// 检查剪贴板是否有文本
async function hasText(): Promise<boolean> {
  const systemPasteboard = pasteboard.getSystemPasteboard();
  const hasData = await systemPasteboard.hasData();
  return hasData;
}
```

---

## 屏幕管理

### 屏幕亮度

```typescript
import { brightness } from '@kit.BasicServicesKit';

// 获取屏幕亮度
async function getBrightness(): Promise<number> {
  return await brightness.getValue();
}

// 设置屏幕亮度
async function setBrightness(value: number): Promise<void> {
  await brightness.setValue(value);
}

// 设置保持屏幕常亮
async function keepScreenOn(on: boolean): Promise<void> {
  await brightness.setKeepScreenOn(on);
}
```

### 屏幕方向

```typescript
import { display } from '@kit.ArkUI';

// 获取屏幕方向
async function getOrientation(): Promise<display.Orientation> {
  const displayInfo = await display.getDefaultDisplay();
  return displayInfo.orientation;
}

// 设置屏幕方向
async function setOrientation(orientation: display.Orientation): Promise<void> {
  // 通过 WindowStage 设置
}
```

---

## 传感器

### 加速度计

```typescript
import { sensor } from '@kit.SensorServiceKit';

// 订阅加速度计
function subscribeAccelerometer(callback: (data: sensor.AccelerometerResponse) => void): void {
  sensor.on(sensor.SensorId.ACCELEROMETER, (data) => {
    callback(data);
  }, { interval: 100000000 }); // 100ms
}

// 取消订阅
function unsubscribeAccelerometer(): void {
  sensor.off(sensor.SensorId.ACCELEROMETER);
}

// 使用示例
subscribeAccelerometer((data) => {
  console.log(`x: ${data.x}, y: ${data.y}, z: ${data.z}`);
});
```

### 陀螺仪

```typescript
// 订阅陀螺仪
function subscribeGyroscope(callback: (data: sensor.GyroscopeResponse) => void): void {
  sensor.on(sensor.SensorId.GYROSCOPE, (data) => {
    callback(data);
  }, { interval: 100000000 });
}
```

### 光线传感器

```typescript
// 订阅光线传感器
function subscribeLight(callback: (data: sensor.LightResponse) => void): void {
  sensor.on(sensor.SensorId.AMBIENT_LIGHT, (data) => {
    callback(data);
  }, { interval: 1000000000 }); // 1s
}
```

---

## 通知

### 发送通知

```typescript
import { notificationManager } from '@kit.NotificationKit';

// 请求通知权限
async function requestPermission(): Promise<boolean> {
  try {
    await notificationManager.requestEnableNotification();
    return true;
  } catch {
    return false;
  }
}

// 发送简单通知
async function sendNotification(title: string, content: string): Promise<void> {
  const request: notificationManager.NotificationRequest = {
    id: 1,
    content: {
      notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
      normal: {
        title: title,
        text: content
      }
    }
  };

  await notificationManager.publish(request);
}

// 发送带按钮的通知
async function sendNotificationWithAction(title: string, content: string): Promise<void> {
  const request: notificationManager.NotificationRequest = {
    id: 2,
    content: {
      notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
      normal: {
        title: title,
        text: content
      }
    },
    actionButtons: [
      {
        title: '确认',
        wantAgent: {
          // 点击按钮触发的 Want
        }
      },
      {
        title: '取消'
      }
    ]
  };

  await notificationManager.publish(request);
}
```

### 通知管理

```typescript
// 取消通知
async function cancelNotification(id: number): Promise<void> {
  await notificationManager.cancel(id);
}

// 取消所有通知
async function cancelAllNotifications(): Promise<void> {
  await notificationManager.cancelAll();
}

// 获取通知数量
async function getNotificationCount(): Promise<number> {
  const notifications = await notificationManager.getActiveNotifications();
  return notifications.length;
}
```

---

## 设备信息

### 获取设备信息

```typescript
import { deviceInfo } from '@kit.BasicServicesKit';

// 设备基本信息
function getDeviceInfo(): object {
  return {
    brand: deviceInfo.brand,
    product: deviceInfo.product,
    model: deviceInfo.model,
    hardwareModel: deviceInfo.hardwareModel,
    serial: deviceInfo.serial,
    manufacturer: deviceInfo.manufacturer,
    osName: deviceInfo.osFullName,
    osVersion: deviceInfo.osReleaseType,
    sdkVersion: deviceInfo.sdkApiVersion
  };
}
```

### 网络状态

```typescript
import { connection } from '@kit.NetworkKit';

// 获取网络状态
async function getNetworkState(): Promise<string> {
  const netHandle = await connection.getDefaultNet();
  const netCapabilities = await connection.getNetCapabilities(netHandle);
  
  if (netCapabilities.bearerTypes?.includes(connection.NetBearType.BEARER_WIFI)) {
    return 'WiFi';
  } else if (netCapabilities.bearerTypes?.includes(connection.NetBearType.BEARER_CELLULAR)) {
    return 'Cellular';
  }
  return 'None';
}

// 监听网络变化
function watchNetworkChange(callback: (state: string) => void): void {
  connection.on('netAvailable', () => {
    callback('Available');
  });
  
  connection.on('netUnavailable', () => {
    callback('Unavailable');
  });
}
```

### 电池信息

```typescript
import { batteryInfo } from '@kit.BasicServicesKit';

// 获取电池信息
function getBatteryInfo(): object {
  return {
    level: batteryInfo.batteryLevel,
    charging: batteryInfo.chargingStatus,
    temperature: batteryInfo.batteryTemperature
  };
}

// 监听电池变化
function watchBatteryChange(callback: (info: object) => void): void {
  batteryInfo.onBatteryChange((info) => {
    callback({
      level: info.batteryLevel,
      charging: info.chargingStatus
    });
  });
}
```

---

## 常见问题

### Q1: 如何获取设备唯一标识？

**A:** 使用设备信息组合或 OAID：
```typescript
import { identifier } from '@kit.AdsKit';

async function getOAID(): Promise<string> {
  try {
    const oaid = await identifier.getOAID();
    return oaid;
  } catch {
    return '';
  }
}
```

### Q2: 如何判断设备类型？

**A:** 使用设备信息：
```typescript
function isTablet(): boolean {
  // 根据屏幕尺寸判断
  return false;
}

function isPhone(): boolean {
  return true;
}
```

### Q3: 如何实现摇一摇？

**A:** 使用加速度计：
```typescript
let lastTime = 0;

sensor.on(sensor.SensorId.ACCELEROMETER, (data) => {
  const now = Date.now();
  if (now - lastTime < 300) return;
  
  const acceleration = Math.sqrt(data.x ** 2 + data.y ** 2 + data.z ** 2);
  if (acceleration > 20) {
    lastTime = now;
    console.log('Shake detected!');
  }
});
```

---

## 📚 延伸阅读

- [剪贴板](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-clipboard)
- [传感器](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-sensor)
- [通知](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-notification)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
