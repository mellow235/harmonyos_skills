# 数据存储

> 💾 本地数据持久化方案

## 📖 目录

1. [存储概述](#存储概述)
2. [首选项存储](#首选项存储)
3. [关系型数据库](#关系型数据库)
4. [分布式数据管理](#分布式数据管理)
5. [文件存储](#文件存储)
6. [存储最佳实践](#存储最佳实践)
7. [常见问题](#常见问题)

---

## 存储概述

HarmonyOS 提供多种数据存储方案，满足不同场景需求。

### 存储方案对比

| 方案 | 适用场景 | 数据量 | 结构 |
|------|---------|--------|------|
| 首选项 | 配置、设置 | 小 | KV |
| 关系型数据库 | 结构化数据 | 中 | 表 |
| 分布式数据 | 跨设备同步 | 中 | KV |
| 文件存储 | 大文件 | 大 | 文件 |

---

## 首选项存储

### 基础用法

```typescript
import { preferences } from '@kit.ArkData';

// 获取首选项实例
async function getPreferences(context: Context): Promise<preferences.Preferences> {
  return await preferences.getPreferences(context, 'myStore');
}

// 写入数据
async function saveData(context: Context): Promise<void> {
  const store = await getPreferences(context);
  await store.put('username', 'Alice');
  await store.put('age', 25);
  await store.put('isLoggedIn', true);
  await store.flush();  // 持久化
}

// 读取数据
async function loadData(context: Context): Promise<void> {
  const store = await getPreferences(context);
  const username = await store.get('username', '');
  const age = await store.get('age', 0);
  const isLoggedIn = await store.get('isLoggedIn', false);
}

// 删除数据
async function removeData(context: Context): Promise<void> {
  const store = await getPreferences(context);
  await store.delete('username');
  await store.flush();
}
```

### 封装工具类

```typescript
class PreferencesUtil {
  private static store: preferences.Preferences | null = null;

  static async init(context: Context): Promise<void> {
    this.store = await preferences.getPreferences(context, 'appStore');
  }

  static async set(key: string, value: preferences.ValueType): Promise<void> {
    await this.store?.put(key, value);
    await this.store?.flush();
  }

  static async get<T>(key: string, defaultValue: T): Promise<T> {
    const value = await this.store?.get(key, defaultValue);
    return value as T;
  }

  static async remove(key: string): Promise<void> {
    await this.store?.delete(key);
    await this.store?.flush();
  }

  static async clear(): Promise<void> {
    await this.store?.clear();
    await this.store?.flush();
  }
}

// 使用
await PreferencesUtil.set('token', 'abc123');
const token = await PreferencesUtil.get<string>('token', '');
```

---

## 关系型数据库

### 创建数据库

```typescript
import { relationalStore } from '@kit.ArkData';

// 创建数据库
async function createDatabase(context: Context): Promise<relationalStore.RdbStore> {
  const STORE_CONFIG: relationalStore.StoreConfig = {
    name: 'mydb.db',
    securityLevel: relationalStore.SecurityLevel.S1
  };

  return await relationalStore.getRdbStore(context, STORE_CONFIG);
}
```

### 创建表

```typescript
async function createTable(rdbStore: relationalStore.RdbStore): Promise<void> {
  const SQL_CREATE_TABLE = `
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      email TEXT UNIQUE,
      age INTEGER,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
  `;
  await rdbStore.executeSql(SQL_CREATE_TABLE);
}
```

### CRUD 操作

```typescript
// 插入数据
async function insertUser(rdbStore: relationalStore.RdbStore, user: User): Promise<number> {
  const bucket: relationalStore.ValuesBucket = {
    name: user.name,
    email: user.email,
    age: user.age
  };
  return await rdbStore.insert('users', bucket);
}

// 查询数据
async function queryUsers(rdbStore: relationalStore.RdbStore): Promise<User[]> {
  const predicates = new relationalStore.RdbPredicates('users');
  const resultSet = await rdbStore.query(predicates, ['id', 'name', 'email', 'age']);
  
  const users: User[] = [];
  while (resultSet.goToNextRow()) {
    users.push({
      id: resultSet.getLong(resultSet.getColumnIndex('id')),
      name: resultSet.getString(resultSet.getColumnIndex('name')),
      email: resultSet.getString(resultSet.getColumnIndex('email')),
      age: resultSet.getLong(resultSet.getColumnIndex('age'))
    });
  }
  resultSet.close();
  return users;
}

// 更新数据
async function updateUser(rdbStore: relationalStore.RdbStore, id: number, user: User): Promise<number> {
  const bucket: relationalStore.ValuesBucket = {
    name: user.name,
    email: user.email,
    age: user.age
  };
  const predicates = new relationalStore.RdbPredicates('users');
  predicates.equalTo('id', id);
  return await rdbStore.update(bucket, predicates);
}

// 删除数据
async function deleteUser(rdbStore: relationalStore.RdbStore, id: number): Promise<number> {
  const predicates = new relationalStore.RdbPredicates('users');
  predicates.equalTo('id', id);
  return await rdbStore.delete(predicates);
}
```

### 封装 DAO

```typescript
class UserDao {
  private rdbStore: relationalStore.RdbStore;

  constructor(rdbStore: relationalStore.RdbStore) {
    this.rdbStore = rdbStore;
  }

  async createTable(): Promise<void> {
    const sql = `
      CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE,
        age INTEGER
      )
    `;
    await this.rdbStore.executeSql(sql);
  }

  async insert(user: User): Promise<number> {
    return await this.rdbStore.insert('users', user as relationalStore.ValuesBucket);
  }

  async findAll(): Promise<User[]> {
    const predicates = new relationalStore.RdbPredicates('users');
    const resultSet = await this.rdbStore.query(predicates, ['*']);
    // 解析结果...
    return users;
  }

  async findById(id: number): Promise<User | null> {
    const predicates = new relationalStore.RdbPredicates('users');
    predicates.equalTo('id', id);
    const resultSet = await this.rdbStore.query(predicates, ['*']);
    // 解析结果...
    return user;
  }

  async update(id: number, user: User): Promise<number> {
    const predicates = new relationalStore.RdbPredicates('users');
    predicates.equalTo('id', id);
    return await this.rdbStore.update(user as relationalStore.ValuesBucket, predicates);
  }

  async delete(id: number): Promise<number> {
    const predicates = new relationalStore.RdbPredicates('users');
    predicates.equalTo('id', id);
    return await this.rdbStore.delete(predicates);
  }
}
```

---

## 分布式数据管理

### KVStore

```typescript
import { distributedKVStore } from '@kit.ArkData';

// 创建 KVStore
async function createKVStore(context: Context): Promise<distributedKVStore.KVStore> {
  const kvManager = distributedKVStore.createKVManager({
    context: context,
    bundleName: 'com.example.myapp'
  });

  const options: distributedKVStore.Options = {
    createIfMissing: true,
    encrypt: false,
    backup: false,
    autoSync: true,
    kvStoreType: distributedKVStore.KVStoreType.SINGLE_VERSION
  };

  return await kvManager.getKVStore('myStore', options);
}

// 写入数据
async function putData(kvStore: distributedKVStore.KVStore, key: string, value: string): Promise<void> {
  await kvStore.put(key, value);
}

// 读取数据
async function getData(kvStore: distributedKVStore.KVStore, key: string): Promise<string | undefined> {
  const entry = await kvStore.get(key);
  return entry as string;
}

// 监听数据变化
function watchData(kvStore: distributedKVStore.KVStore): void {
  kvStore.on('dataChange', distributedKVStore.SubscribeType.SUBSCRIBE_TYPE_ALL, (data) => {
    console.log('Data changed:', data);
  });
}
```

---

## 文件存储

### 应用文件

```typescript
import { fileIo } from '@kit.CoreFileKit';

// 获取应用文件目录
function getFilesDir(context: Context): string {
  return context.filesDir;
}

// 写入文件
async function writeFile(context: Context, fileName: string, content: string): Promise<void> {
  const filePath = `${context.filesDir}/${fileName}`;
  const file = fileIo.openSync(filePath, fileIo.OpenMode.CREATE | fileIo.OpenMode.WRITE_ONLY);
  fileIo.writeSync(file.fd, content);
  fileIo.closeSync(file);
}

// 读取文件
async function readFile(context: Context, fileName: string): Promise<string> {
  const filePath = `${context.filesDir}/${fileName}`;
  const file = fileIo.openSync(filePath, fileIo.OpenMode.READ_ONLY);
  const stat = fileIo.statSync(filePath);
  const buffer = new ArrayBuffer(stat.size);
  fileIo.readSync(file.fd, buffer);
  fileIo.closeSync(file);
  return String.fromCharCode(...new Uint8Array(buffer));
}

// 删除文件
function deleteFile(context: Context, fileName: string): void {
  const filePath = `${context.filesDir}/${fileName}`;
  fileIo.unlinkSync(filePath);
}
```

### 缓存文件

```typescript
// 获取缓存目录
function getCacheDir(context: Context): string {
  return context.cacheDir;
}

// 清理缓存
function clearCache(context: Context): void {
  const cacheDir = context.cacheDir;
  // 遍历删除缓存文件
}
```

---

## 存储最佳实践

### 1. 选择合适的存储方案

```
场景分析：
├── 用户设置、配置 → 首选项
├── 结构化业务数据 → 关系型数据库
├── 跨设备同步数据 → 分布式 KV
├── 大文件、多媒体 → 文件存储
└── 临时缓存 → 缓存目录
```

### 2. 数据加密

```typescript
// 使用安全级别
const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'secure.db',
  securityLevel: relationalStore.SecurityLevel.S3  // 高安全级别
};
```

### 3. 错误处理

```typescript
async function safeOperation(): Promise<void> {
  try {
    await saveData();
  } catch (error) {
    console.error('Storage error:', error);
    // 降级处理或提示用户
  }
}
```

---

## 常见问题

### Q1: 首选项数据丢失？

**A:** 确保调用 flush()：
```typescript
await store.put('key', 'value');
await store.flush();  // 必须调用
```

### Q2: 数据库升级如何处理？

**A:** 使用版本迁移：
```typescript
const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'mydb.db',
  securityLevel: relationalStore.SecurityLevel.S1
};

// 升级逻辑
if (oldVersion < 2) {
  await rdbStore.executeSql('ALTER TABLE users ADD COLUMN avatar TEXT');
}
```

### Q3: 如何实现数据备份？

**A:** 使用分布式数据的备份功能或手动导出：
```typescript
// 导出数据
const data = await exportData();
await writeFile('backup.json', JSON.stringify(data));
```

---

## 📚 延伸阅读

- [首选项](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-preferences)
- [关系型数据库](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-rdb-store)
- [分布式数据管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-distributed-data)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
