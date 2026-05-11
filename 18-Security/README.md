# 安全机制

> 🔒 HarmonyOS 应用安全与代码保护

## 📖 目录

1. [安全概述](#安全概述)
2. [ArkGuard 源码混淆](#arkguard-源码混淆)
3. [字节码混淆](#字节码混淆)
4. [权限安全](#权限安全)
5. [数据安全](#数据安全)
6. [网络安全](#网络安全)
7. [安全最佳实践](#安全最佳实践)
8. [常见问题](#常见问题)

---

## 安全概述

HarmonyOS 提供多层安全防护机制：

```
安全体系
├── 代码安全
│   ├── ArkGuard 源码混淆
│   └── 字节码混淆
├── 权限安全
│   ├── 最小权限原则
│   └── 动态权限申请
├── 数据安全
│   ├── 加密存储
│   └── 安全沙箱
└── 网络安全
    ├── HTTPS 强制
    └── 证书校验
```

---

## ArkGuard 源码混淆

### 开启混淆

```json5
// build-profile.json5
{
  "buildOption": {
    "arkOptions": {
      "obfuscation": {
        "enabled": true,
        "options": {
          "enableStringPropertyObfuscation": true,
          "enableToplevelObfuscation": true,
          "enableExportObfuscation": true,
          "enablePropertyObfuscation": true
        }
      }
    }
  }
}
```

### 保留规则

```json5
// obfuscation-rules.txt
// 保留特定类名
-keep class com.example.MyClass

// 保留特定方法
-keep class com.example.Utils {
    public static *;
}

// 保留特定属性
-keep-property-name id, name, tag

// 保留全局变量
-keep-global-name MY_CONSTANT

// 保留字符串属性
-keep-string-property name, title, description
```

### 混淆配置详解

```json5
{
  "buildOption": {
    "arkOptions": {
      "obfuscation": {
        "enabled": true,
        "options": {
          // 顶层作用域名称混淆
          "enableToplevelObfuscation": true,
          
          // 属性名称混淆
          "enablePropertyObfuscation": true,
          
          // 字符串属性混淆
          "enableStringPropertyObfuscation": true,
          
          // 导出名称混淆（需谨慎）
          "enableExportObfuscation": false,
          
          // 文件混淆
          "enableFileObfuscation": true,
          
          // 压缩
          "compact": true,
          
          // 移除 console
          "removeConsole": true,
          
          // 移除注释
          "removeComments": true
        },
        "keep": {
          // 保留的类
          "class": [
            "com.example.KeepClass",
            "com.example.**/*Model"
          ],
          // 保留的方法
          "method": [
            "com.example.KeepClass#keepMethod"
          ],
          // 保留的属性
          "property": [
            "id",
            "name",
            "tag"
          ]
        }
      }
    }
  }
}
```

---

## 字节码混淆

### 开启字节码混淆

```json5
// build-profile.json5
{
  "buildOption": {
    "arkOptions": {
      "bytecodeObfuscation": {
        "enabled": true
      }
    }
  }
}
```

### 源码混淆 vs 字节码混淆

| 特性 | 源码混淆 | 字节码混淆 |
|------|---------|-----------|
| 保护时机 | 编译前 | 编译后 |
| 保护强度 | 中等 | 强 |
| 逆向难度 | 较难 | 很难 |
| 性能影响 | 无 | 极小 |
| 推荐场景 | 一般应用 | 高安全要求 |

---

## 权限安全

### 权限分类

```typescript
// 权限级别
enum PermissionLevel {
  // 正常权限（安装时授予）
  NORMAL = 'normal',
  
  // 危险权限（运行时申请）
  DANGEROUS = 'dangerous',
  
  // 签名权限（同签名应用）
  SIGNATURE = 'signature',
  
  // 系统权限（预装应用）
  SYSTEM = 'system'
}
```

### 运行时权限申请

```typescript
import { abilityAccessCtrl } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';

class PermissionManager {
  private atManager: abilityAccessCtrl.AtManager;

  constructor() {
    this.atManager = abilityAccessCtrl.createAtManager();
  }

  // 检查权限
  async checkPermission(permission: string): Promise<boolean> {
    try {
      const authStatus = await this.atManager.checkAccessToken(
        await this.getTokenId(), 
        permission
      );
      return authStatus === abilityAccessCtrl.AuthStatus.PERMISSION_GRANTED;
    } catch (err) {
      console.error('Check permission failed:', err);
      return false;
    }
  }

  // 申请权限
  async requestPermission(permissions: string[]): Promise<boolean> {
    try {
      const context = getContext(this);
      const result = await this.atManager.requestPermissionsFromUser(
        context, 
        permissions
      );
      
      return result.authResults.every(status => 
        status === abilityAccessCtrl.AuthStatus.PERMISSION_GRANTED
      );
    } catch (err) {
      console.error('Request permission failed:', err);
      return false;
    }
  }

  private async getTokenId(): Promise<number> {
    const context = getContext(this);
    const applicationInfo = await context.getApplicationInfo();
    return applicationInfo.accessTokenId;
  }
}

// 使用
const permissionManager = new PermissionManager();

async function requestCamera(): Promise<void> {
  const hasPermission = await permissionManager.checkPermission(
    'ohos.permission.CAMERA'
  );
  
  if (!hasPermission) {
    const granted = await permissionManager.requestPermission([
      'ohos.permission.CAMERA'
    ]);
    
    if (!granted) {
      console.warn('Camera permission denied');
      return;
    }
  }
  
  // 使用相机
}
```

### 权限组管理

```typescript
// 常用权限组
const PERMISSION_GROUPS = {
  // 位置
  LOCATION: [
    'ohos.permission.APPROXIMATELY_LOCATION',
    'ohos.permission.LOCATION'
  ],
  
  // 存储
  STORAGE: [
    'ohos.permission.READ_MEDIA',
    'ohos.permission.WRITE_MEDIA'
  ],
  
  // 相机和麦克风
  MEDIA: [
    'ohos.permission.CAMERA',
    'ohos.permission.MICROPHONE'
  ],
  
  // 联系人
  CONTACTS: [
    'ohos.permission.READ_CONTACTS',
    'ohos.permission.WRITE_CONTACTS'
  ]
};

// 批量申请
async function requestMediaPermissions(): Promise<boolean> {
  return permissionManager.requestPermission(PERMISSION_GROUPS.MEDIA);
}
```

---

## 数据安全

### 加密存储

```typescript
import { cryptoFramework } from '@kit.CryptoArchitectureKit';

class SecureStorage {
  // AES 加密
  async encrypt(data: string, key: Uint8Array): Promise<Uint8Array> {
    const generator = cryptoFramework.createSymKeyGenerator('AES128');
    const symKey = await generator.convertKey({ data: key });
    
    const cipher = cryptoFramework.createCipher('AES128|CBC|PKCS7');
    await cipher.init(cryptoFramework.CryptoMode.ENCRYPT_MODE, symKey, {
      iv: new Uint8Array(16)
    });
    
    const input: cryptoFramework.DataBlob = { data: new Uint8Array(Buffer.from(data)) };
    const output = await cipher.doFinal(input);
    return output.data;
  }

  // AES 解密
  async decrypt(encryptedData: Uint8Array, key: Uint8Array): Promise<string> {
    const generator = cryptoFramework.createSymKeyGenerator('AES128');
    const symKey = await generator.convertKey({ data: key });
    
    const decipher = cryptoFramework.createCipher('AES128|CBC|PKCS7');
    await decipher.init(cryptoFramework.CryptoMode.DECRYPT_MODE, symKey, {
      iv: new Uint8Array(16)
    });
    
    const input: cryptoFramework.DataBlob = { data: encryptedData };
    const output = await decipher.doFinal(input);
    return Buffer.from(output.data).toString();
  }
}
```

### 安全首选项

```typescript
import { preferences } from '@kit.ArkData';

class SecurePreferences {
  private store: preferences.Preferences | null = null;

  async init(context: Context): Promise<void> {
    this.store = await preferences.getPreferences(context, 'secure_storage');
  }

  // 存储敏感数据（自动加密）
  async setSecure(key: string, value: string): Promise<void> {
    if (!this.store) return;
    
    // 实际项目中应使用加密后的值
    const encrypted = await this.encrypt(value);
    await this.store.put(key, encrypted);
    await this.store.flush();
  }

  // 读取敏感数据
  async getSecure(key: string): Promise<string | null> {
    if (!this.store) return null;
    
    const encrypted = await this.store.get(key, '');
    if (!encrypted) return null;
    
    return this.decrypt(encrypted as string);
  }

  private async encrypt(value: string): Promise<string> {
    // 实现加密逻辑
    return value; // 占位
  }

  private async decrypt(value: string): Promise<string> {
    // 实现解密逻辑
    return value; // 占位
  }
}
```

---

## 网络安全

### HTTPS 请求

```typescript
import { http } from '@kit.NetworkKit';

async function secureRequest(url: string): Promise<http.HttpResponse> {
  const httpRequest = http.createHttp();
  
  try {
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.GET,
      header: {
        'Content-Type': 'application/json'
      },
      // 证书校验
      caPath: '/path/to/ca.crt',
      // 证书 pinning
      certificatePinning: {
        hashes: ['sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=']
      }
    });
    
    return response;
  } finally {
    httpRequest.destroy();
  }
}
```

### 请求签名

```typescript
import { cryptoFramework } from '@kit.CryptoArchitectureKit';

class RequestSigner {
  // 生成请求签名
  async signRequest(params: Record<string, string>, secret: string): Promise<string> {
    // 1. 参数排序
    const sortedKeys = Object.keys(params).sort();
    const paramString = sortedKeys
      .map(key => `${key}=${params[key]}`)
      .join('&');
    
    // 2. 添加密钥
    const signString = `${paramString}&key=${secret}`;
    
    // 3. HMAC-SHA256 签名
    const mac = cryptoFramework.createMac('SHA256');
    const keyData: cryptoFramework.DataBlob = { 
      data: new Uint8Array(Buffer.from(secret)) 
    };
    await mac.init(keyData);
    
    const input: cryptoFramework.DataBlob = { 
      data: new Uint8Array(Buffer.from(signString)) 
    };
    const output = await mac.doFinal(input);
    
    // 4. Base64 编码
    return btoa(String.fromCharCode(...output.data));
  }

  // 验证响应签名
  async verifyResponse(response: string, signature: string, publicKey: string): Promise<boolean> {
    // RSA 验签
    const verifier = cryptoFramework.createVerify('SHA256|RSA2048|PSS');
    const pubKey = await this.loadPublicKey(publicKey);
    await verifier.init(pubKey);
    
    const data: cryptoFramework.DataBlob = { 
      data: new Uint8Array(Buffer.from(response)) 
    };
    const sig: cryptoFramework.DataBlob = { 
      data: new Uint8Array(Buffer.from(atob(signature))) 
    };
    
    return verifier.verify(data, sig);
  }

  private async loadPublicKey(keyData: string): Promise<cryptoFramework.PubKey> {
    const generator = cryptoFramework.createAsyKeyGenerator('RSA2048|PKCS1');
    return generator.convertPemKey({ data: keyData }, null);
  }
}
```

---

## 安全最佳实践

### 1. 最小权限原则

```typescript
// module.json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:internet_permission_reason"
      },
      {
        "name": "ohos.permission.CAMERA",
        "reason": "$string:camera_permission_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inUse"
        }
      }
    ]
  }
}
```

### 2. 敏感信息保护

```typescript
// ❌ 错误：硬编码密钥
const API_KEY = 'sk-1234567890abcdef';

// ✅ 正确：从安全存储读取
const API_KEY = await SecureStorage.get('api_key');

// ❌ 错误：日志输出敏感信息
console.info(`User password: ${password}`);

// ✅ 正确：脱敏处理
console.info(`User phone: ${phone.slice(0, 3)}****${phone.slice(-4)}`);
```

### 3. 输入验证

```typescript
class InputValidator {
  // 验证手机号
  static isPhone(phone: string): boolean {
    return /^1[3-9]\d{9}$/.test(phone);
  }

  // 验证邮箱
  static isEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  // 验证身份证
  static isIdCard(id: string): boolean {
    return /^\d{17}[\dXx]$/.test(id);
  }

  // 防 XSS
  static sanitizeInput(input: string): string {
    return input
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;');
  }
}
```

### 4. 安全相机

```typescript
import { camera } from '@kit.CameraKit';

class SecureCamera {
  async captureSecurePhoto(): Promise<image.PixelMap> {
    // 使用安全相机模式
    const cameraManager = camera.getCameraManager(getContext(this));
    
    // 设置安全级别
    const captureSession = cameraManager.createCaptureSession();
    await captureSession.beginConfig();
    
    // 配置安全输出
    const profile = {
      format: camera.CameraFormat.CAMERA_FORMAT_JPEG,
      size: { width: 1920, height: 1080 }
    };
    
    const photoOutput = cameraManager.createPhotoOutput(profile);
    await captureSession.addOutput(photoOutput);
    await captureSession.commitConfig();
    
    // 拍照（自动加密存储）
    return new Promise((resolve, reject) => {
      photoOutput.on('photoAvailable', (err, photo) => {
        if (err) {
          reject(err);
        } else {
          resolve(photo);
        }
      });
      
      photoOutput.capture();
    });
  }
}
```

---

## 常见问题

### Q1: 混淆后应用崩溃？

**A:** 
1. 检查保留规则是否完整
2. 确认反射使用的类名已保留
3. 查看混淆映射文件定位问题

### Q2: 权限申请被拒绝？

**A:**
1. 检查 module.json5 是否声明权限
2. 确认权限申请时机（需在用户操作时申请）
3. 处理用户永久拒绝的情况（引导到设置）

### Q3: 如何安全存储密钥？

**A:**
1. 使用系统 KeyStore
2. 密钥分片存储
3. 服务端动态获取
4. 避免硬编码

---

## 📚 延伸阅读

- [ArkGuard 源码混淆](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-arkguard)
- [权限开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/permission-guidelines)
- [安全相机](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/camera-secure-photo)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
