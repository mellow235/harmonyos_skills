# 网络请求

> 🌐 HTTP 通信与数据交互

## 📖 目录

1. [网络概述](#网络概述)
2. [HTTP 请求](#http-请求)
3. [数据解析](#数据解析)
4. [WebSocket](#websocket)
5. [网络工具封装](#网络工具封装)
6. [网络最佳实践](#网络最佳实践)
7. [常见问题](#常见问题)

---

## 网络概述

HarmonyOS 提供完整的网络通信能力，支持 HTTP、WebSocket 等协议。

### 网络能力

```
网络模块
├── HTTP
│   ├── GET 请求
│   ├── POST 请求
│   ├── 文件上传
│   └── 文件下载
├── WebSocket
│   ├── 实时通信
│   └── 长连接
└── 数据解析
    ├── JSON
    └── XML
```

---

## HTTP 请求

### GET 请求

```typescript
import { http } from '@kit.NetworkKit';

async function httpGet(url: string): Promise<string> {
  const httpRequest = http.createHttp();
  
  try {
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.GET,
      header: {
        'Content-Type': 'application/json'
      },
      readTimeout: 15000,
      connectTimeout: 15000
    });

    if (response.responseCode === 200) {
      return response.result as string;
    } else {
      throw new Error(`HTTP Error: ${response.responseCode}`);
    }
  } finally {
    httpRequest.destroy();
  }
}
```

### POST 请求

```typescript
async function httpPost(url: string, data: object): Promise<string> {
  const httpRequest = http.createHttp();
  
  try {
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.POST,
      header: {
        'Content-Type': 'application/json'
      },
      extraData: JSON.stringify(data),
      readTimeout: 15000,
      connectTimeout: 15000
    });

    if (response.responseCode === 200) {
      return response.result as string;
    } else {
      throw new Error(`HTTP Error: ${response.responseCode}`);
    }
  } finally {
    httpRequest.destroy();
  }
}
```

### 请求方法

```typescript
// PUT
async function httpPut(url: string, data: object): Promise<string> {
  const httpRequest = http.createHttp();
  const response = await httpRequest.request(url, {
    method: http.RequestMethod.PUT,
    extraData: JSON.stringify(data)
  });
  return response.result as string;
}

// DELETE
async function httpDelete(url: string): Promise<string> {
  const httpRequest = http.createHttp();
  const response = await httpRequest.request(url, {
    method: http.RequestMethod.DELETE
  });
  return response.result as string;
}
```

### 请求头设置

```typescript
async function requestWithHeaders(url: string): Promise<string> {
  const httpRequest = http.createHttp();
  
  const response = await httpRequest.request(url, {
    method: http.RequestMethod.GET,
    header: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer token123',
      'Accept': 'application/json',
      'X-Custom-Header': 'value'
    }
  });

  return response.result as string;
}
```

---

## 数据解析

### JSON 解析

```typescript
// 解析 JSON
function parseJSON<T>(jsonString: string): T {
  return JSON.parse(jsonString) as T;
}

// 序列化 JSON
function toJSON(data: object): string {
  return JSON.stringify(data);
}

// 使用示例
interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(id: number): Promise<User> {
  const response = await httpGet(`https://api.example.com/users/${id}`);
  return parseJSON<User>(response);
}
```

### 类型安全的数据解析

```typescript
class JsonParser {
  static parse<T>(json: string, validator: (data: any) => data is T): T | null {
    try {
      const data = JSON.parse(json);
      if (validator(data)) {
        return data;
      }
      return null;
    } catch {
      return null;
    }
  }

  static isUser(data: any): data is User {
    return (
      typeof data === 'object' &&
      typeof data.id === 'number' &&
      typeof data.name === 'string' &&
      typeof data.email === 'string'
    );
  }
}

// 使用
const user = JsonParser.parse<User>(response, JsonParser.isUser);
```

---

## WebSocket

### 建立连接

```typescript
import { webSocket } from '@kit.NetworkKit';

class WebSocketClient {
  private ws: webSocket.WebSocket | null = null;
  private url: string;

  constructor(url: string) {
    this.url = url;
  }

  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.ws = webSocket.createWebSocket();
      
      this.ws.on('open', () => {
        console.log('WebSocket connected');
        resolve();
      });

      this.ws.on('message', (data: string | ArrayBuffer) => {
        this.onMessage(data);
      });

      this.ws.on('close', (code: number, reason: string) => {
        console.log('WebSocket closed:', code, reason);
      });

      this.ws.on('error', (error: Error) => {
        console.error('WebSocket error:', error);
        reject(error);
      });

      this.ws.connect(this.url);
    });
  }

  send(data: string): void {
    this.ws?.send(data);
  }

  close(): void {
    this.ws?.close();
  }

  protected onMessage(data: string | ArrayBuffer): void {
    console.log('Received:', data);
  }
}

// 使用
const client = new WebSocketClient('wss://echo.websocket.org');
await client.connect();
client.send('Hello');
```

---

## 网络工具封装

### 完整的 HTTP 客户端

```typescript
import { http } from '@kit.NetworkKit';

interface RequestConfig {
  url: string;
  method?: http.RequestMethod;
  data?: object;
  headers?: Record<string, string>;
  timeout?: number;
}

interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
}

class HttpClient {
  private static instance: HttpClient;
  private baseUrl: string;
  private defaultHeaders: Record<string, string>;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = {
      'Content-Type': 'application/json'
    };
  }

  static getInstance(baseUrl: string): HttpClient {
    if (!this.instance) {
      this.instance = new HttpClient(baseUrl);
    }
    return this.instance;
  }

  setToken(token: string): void {
    this.defaultHeaders['Authorization'] = `Bearer ${token}`;
  }

  async request<T>(config: RequestConfig): Promise<ApiResponse<T>> {
    const httpRequest = http.createHttp();
    
    try {
      const url = config.url.startsWith('http') 
        ? config.url 
        : `${this.baseUrl}${config.url}`;

      const response = await httpRequest.request(url, {
        method: config.method || http.RequestMethod.GET,
        header: {
          ...this.defaultHeaders,
          ...config.headers
        },
        extraData: config.data ? JSON.stringify(config.data) : undefined,
        readTimeout: config.timeout || 15000,
        connectTimeout: config.timeout || 15000
      });

      const result = JSON.parse(response.result as string) as ApiResponse<T>;
      return result;
    } finally {
      httpRequest.destroy();
    }
  }

  async get<T>(url: string): Promise<ApiResponse<T>> {
    return this.request<T>({ url, method: http.RequestMethod.GET });
  }

  async post<T>(url: string, data: object): Promise<ApiResponse<T>> {
    return this.request<T>({ url, method: http.RequestMethod.POST, data });
  }

  async put<T>(url: string, data: object): Promise<ApiResponse<T>> {
    return this.request<T>({ url, method: http.RequestMethod.PUT, data });
  }

  async delete<T>(url: string): Promise<ApiResponse<T>> {
    return this.request<T>({ url, method: http.RequestMethod.DELETE });
  }
}

// 使用
const api = HttpClient.getInstance('https://api.example.com');
api.setToken('your-token');

const users = await api.get<User[]>('/users');
const newUser = await api.post<User>('/users', { name: 'Alice' });
```

---

## 网络最佳实践

### 1. 错误处理

```typescript
async function safeRequest<T>(request: () => Promise<T>): Promise<T | null> {
  try {
    return await request();
  } catch (error) {
    if (error instanceof Error) {
      console.error('Request failed:', error.message);
      // 显示错误提示
    }
    return null;
  }
}
```

### 2. 请求重试

```typescript
async function retryRequest<T>(
  request: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await request();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(1000 * (i + 1));  // 指数退避
    }
  }
  throw new Error('Max retries exceeded');
}
```

### 3. 请求取消

```typescript
class RequestController {
  private abortController: AbortController | null = null;

  async request<T>(url: string): Promise<T> {
    this.abortController = new AbortController();
    
    try {
      const response = await fetch(url, {
        signal: this.abortController.signal
      });
      return await response.json();
    } finally {
      this.abortController = null;
    }
  }

  cancel(): void {
    this.abortController?.abort();
  }
}
```

---

## 常见问题

### Q1: 网络请求超时？

**A:** 调整超时设置：
```typescript
const response = await httpRequest.request(url, {
  readTimeout: 30000,
  connectTimeout: 30000
});
```

### Q2: 如何处理 HTTPS 证书？

**A:** 使用系统默认证书或配置自定义证书。

### Q3: 如何实现请求缓存？

**A:** 使用 Map 缓存响应：
```typescript
const cache = new Map<string, { data: any; timestamp: number }>();

async function cachedRequest<T>(url: string): Promise<T> {
  const cached = cache.get(url);
  if (cached && Date.now() - cached.timestamp < 60000) {
    return cached.data;
  }
  
  const data = await httpGet(url);
  cache.set(url, { data, timestamp: Date.now() });
  return data;
}
```

---

## 📚 延伸阅读

- [HTTP 请求](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-http-request)
- [WebSocket](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-websocket)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
