# 权限管理

> 🔒 应用权限与安全

## 📖 目录

1. [权限概述](#权限概述)
2. [权限类型](#权限类型)
3. [权限申请](#权限申请)
4. [权限组管理](#权限组管理)
5. [安全最佳实践](#安全最佳实践)
6. [常见问题](#常见问题)

---

## 权限概述

HarmonyOS 采用严格的权限管理机制，保护用户隐私和系统安全。

### 权限分类

```
权限类型
├── system_grant（系统授权）
│   ├── 网络访问
│   ├── 蓝牙
│   └── 振动
└── user_grant（用户授权）
    ├── 相机
    ├── 麦克风
    ├── 位置
    ├── 存储
    └── 通讯录
```

---

## 权限类型

### 系统授权权限

这些权限在应用安装时自动授予：

```json
// module.json5
{
  "requestPermissions": [
    {
      "name": "ohos.permission.INTERNET",
      "reason": "$string:permission_internet_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "always"
      }
    },
    {
      "name": "ohos.permission.GET_NETWORK_INFO",
      "reason": "$string:permission_network_reason"
    },
    {
      "name": "ohos.permission.USE_BLUETOOTH",
      "reason": "$string:permission_bluetooth_reason"
    }
  ]
}
```

### 用户授权权限

这些权限需要用户手动授权：

```json
{
  "requestPermissions": [
    {
      "name": "ohos.permission.CAMERA",
      "reason": "$string:permission_camera_reason",
      "usedScene": {
        "abilities": ["CameraAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.MICROPHONE",
      "reason": "$string:permission_microphone_reason"
    },
    {
      "name": "ohos.permission.READ_MEDIA",
      "reason": "$string:permission_media_reason"
    }
  ]
}
```

---

## 权限申请

### 检查权限

```typescript
import { abilityAccessCtrl } from '@kit.AbilityKit';

async function checkPermission(permission: string): Promise<boolean> {
  try {
    const atManager = abilityAccessCtrl.createAtManager();
    const grantStatus = await atManager.checkAccessToken(
      getContext().tokenID,
      permission
    );
    return grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
  } catch {
    return false;
  }
}
```

### 申请权限

```typescript
import { abilityAccessCtrl } from '@kit.AbilityKit';

async function requestPermission(permissions: string[]): Promise<boolean> {
  try {
    const atManager = abilityAccessCtrl.createAtManager();
    const results = await atManager.requestPermissionsFromUser(
      getContext(),
      permissions
    );
    
    return results.authResults.every(
      result => result === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
    );
  } catch {
    return false;
  }
}

// 使用
async function requestCameraPermission(): Promise<boolean> {
  const hasPermission = await checkPermission('ohos.permission.CAMERA');
  if (hasPermission) {
    return true;
  }
  return await requestPermission(['ohos.permission.CAMERA']);
}
```

### 权限申请工具类

```typescript
class PermissionManager {
  private static atManager = abilityAccessCtrl.createAtManager();

  static async check(permission: string): Promise<boolean> {
    try {
      const grantStatus = await this.atManager.checkAccessToken(
        getContext().tokenID,
        permission
      );
      return grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
    } catch {
      return false;
    }
  }

  static async request(permissions: string[]): Promise<boolean> {
    try {
      const results = await this.atManager.requestPermissionsFromUser(
        getContext(),
        permissions
      );
      return results.authResults.every(
        result => result === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
      );
    } catch {
      return false;
    }
  }

  static async checkAndRequest(permission: string): Promise<boolean> {
    const hasPermission = await this.check(permission);
    if (hasPermission) {
      return true;
    }
    return await this.request([permission]);
  }

  static async checkAndRequestMultiple(permissions: string[]): Promise<boolean> {
    const results = await Promise.all(
      permissions.map(p => this.check(p))
    );
    
    const missingPermissions = permissions.filter(
      (_, index) => !results[index]
    );
    
    if (missingPermissions.length === 0) {
      return true;
    }
    
    return await this.request(missingPermissions);
  }
}
```

---

## 权限组管理

### 权限组定义

```typescript
const PERMISSION_GROUPS = {
  // 相机权限组
  CAMERA: [
    'ohos.permission.CAMERA'
  ],
  
  // 麦克风权限组
  MICROPHONE: [
    'ohos.permission.MICROPHONE'
  ],
  
  // 存储权限组
  STORAGE: [
    'ohos.permission.READ_MEDIA',
    'ohos.permission.WRITE_MEDIA'
  ],
  
  // 位置权限组
  LOCATION: [
    'ohos.permission.APPROXIMATELY_LOCATION',
    'ohos.permission.LOCATION'
  ],
  
  // 通讯录权限组
  CONTACTS: [
    'ohos.permission.READ_CONTACTS',
    'ohos.permission.WRITE_CONTACTS'
  ]
};

// 申请权限组
async function requestPermissionGroup(group: keyof typeof PERMISSION_GROUPS): Promise<boolean> {
  const permissions = PERMISSION_GROUPS[group];
  return await PermissionManager.request(permissions);
}
```

### 权限检查封装

```typescript
class PermissionGuard {
  static async withCameraPermission<T>(action: () => Promise<T>): Promise<T | null> {
    const hasPermission = await PermissionManager.checkAndRequest(
      'ohos.permission.CAMERA'
    );
    
    if (!hasPermission) {
      console.error('Camera permission denied');
      return null;
    }
    
    return await action();
  }

  static async withStoragePermission<T>(action: () => Promise<T>): Promise<T | null> {
    const hasPermission = await PermissionManager.checkAndRequestMultiple([
      'ohos.permission.READ_MEDIA',
      'ohos.permission.WRITE_MEDIA'
    ]);
    
    if (!hasPermission) {
      console.error('Storage permission denied');
      return null;
    }
    
    return await action();
  }

  static async withLocationPermission<T>(action: () => Promise<T>): Promise<T | null> {
    const hasPermission = await PermissionManager.checkAndRequestMultiple([
      'ohos.permission.APPROXIMATELY_LOCATION',
      'ohos.permission.LOCATION'
    ]);
    
    if (!hasPermission) {
      console.error('Location permission denied');
      return null;
    }
    
    return await action();
  }
}

// 使用
await PermissionGuard.withCameraPermission(async () => {
  // 打开相机
});
```

---

## 安全最佳实践

### 1. 最小权限原则

```json
// ✅ 只申请必要的权限
{
  "requestPermissions": [
    {
      "name": "ohos.permission.INTERNET"
    }
  ]
}

// ❌ 申请过多权限
{
  "requestPermissions": [
    {
      "name": "ohos.permission.INTERNET"
    },
    {
      "name": "ohos.permission.CAMERA"
    },
    {
      "name": "ohos.permission.MICROPHONE"
    },
    {
      "name": "ohos.permission.LOCATION"
    }
    // ... 更多不必要的权限
  ]
}
```

### 2. 权限使用说明

```json
{
  "requestPermissions": [
    {
      "name": "ohos.permission.CAMERA",
      "reason": "$string:permission_camera_reason",
      "usedScene": {
        "abilities": ["CameraAbility"],
        "when": "inuse"
      }
    }
  ]
}
```

在 `resources/base/element/string.json` 中添加说明：
```json
{
  "string": [
    {
      "name": "permission_camera_reason",
      "value": "用于拍照和扫码功能"
    }
  ]
}
```

### 3. 权限被拒处理

```typescript
async function handlePermissionDenied(permission: string): Promise<void> {
  // 提示用户
  AlertDialog.show({
    title: '权限被拒绝',
    message: '该功能需要相机权限，请在设置中开启',
    primaryButton: {
      value: '去设置',
      action: () => {
        // 跳转到应用设置页面
        context.startAbility({
          bundleName: 'com.huawei.hmos.settings',
          abilityName: 'com.huawei.hmos.settings.MainAbility'
        });
      }
    },
    secondaryButton: {
      value: '取消'
    }
  });
}
```

### 4. 数据安全

```typescript
import { cryptoFramework } from '@kit.CryptoArchitectureKit';

// 加密数据
async function encrypt(data: string, key: string): Promise<string> {
  // 使用系统加密 API
  return encryptedData;
}

// 解密数据
async function decrypt(encryptedData: string, key: string): Promise<string> {
  // 使用系统解密 API
  return decryptedData;
}
```

---

## 常见问题

### Q1: 权限被拒绝后如何再次申请？

**A:** 引导用户到设置页面：
```typescript
async function goToSettings(): Promise<void> {
  const want = {
    bundleName: 'com.huawei.hmos.settings',
    abilityName: 'com.huawei.hmos.settings.MainAbility',
    parameters: {
      ':ohos:windowWidth': 360,
      ':ohos:windowHeight': 640
    }
  };
  await context.startAbility(want);
}
```

### Q2: 如何判断权限是否被永久拒绝？

**A:** 检查权限状态：
```typescript
async function isPermissionPermanentlyDenied(permission: string): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  try {
    await atManager.requestPermissionsFromUser(getContext(), [permission]);
    return false;
  } catch (error) {
    // 如果抛出异常，可能被永久拒绝
    return true;
  }
}
```

### Q3: 如何在运行时动态申请权限？

**A:** 使用 `requestPermissionsFromUser`：
```typescript
async function dynamicRequest(permissions: string[]): Promise<void> {
  // 先检查
  const allGranted = await Promise.all(
    permissions.map(p => PermissionManager.check(p))
  );
  
  if (allGranted.every(Boolean)) {
    return;
  }
  
  // 申请未授权的权限
  const missing = permissions.filter((_, i) => !allGranted[i]);
  await PermissionManager.request(missing);
}
```

---

## 📚 延伸阅读

- [权限概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-permissions-overview)
- [访问控制](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-access-control)
- [安全指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-security)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
