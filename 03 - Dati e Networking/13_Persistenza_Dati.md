# Capitolo 13: Persistenza Dati Locale

[← Precedente: State Management](12_State_Management.md) | [Torna all'Indice](../README.md) | [Prossimo: GraphQL e Apollo →](14_GraphQL_Apollo.md)

---

## Descrizione

Le app moderne necessitano di salvare dati localmente: preferenze utente, cache, dati offline. React Native offre diverse soluzioni: AsyncStorage per key-value storage, SQLite per database relazionali, Realm per database NoSQL object-oriented, e WatermelonDB per reactive databases.

In questo capitolo imparerai a scegliere la soluzione giusta, implementare AsyncStorage, usare SQLite, esplorare Realm e WatermelonDB, gestire migrazioni, e implementare strategie di caching e offline-first.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Utilizzare AsyncStorage efficacemente
- [ ] Implementare SQLite database
- [ ] Gestire Realm database
- [ ] Usare WatermelonDB
- [ ] Implementare migrazioni database
- [ ] Gestire caching intelligente
- [ ] Implementare offline-first
- [ ] Sincronizzare dati online/offline
- [ ] Ottimizzare performance storage
- [ ] Gestire encryption dati

## 📚 Argomenti Teorici

1. [AsyncStorage](#1-asyncstorage)
2. [SQLite](#2-sqlite)
3. [Realm](#3-realm)
4. [WatermelonDB](#4-watermelondb)
5. [Migrations](#5-migrations)
6. [Caching Strategies](#6-caching)
7. [Offline-First](#7-offline-first)
8. [Data Sync](#8-sync)
9. [Encryption](#9-encryption)
10. [Best Practices](#10-best-practices)

---

## 1. AsyncStorage

### Installation

```bash
npm install @react-native-async-storage/async-storage
```

### Basic Usage

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Store data
const storeData = async (key: string, value: string) => {
  try {
    await AsyncStorage.setItem(key, value);
  } catch (error) {
    console.error('Error storing data:', error);
  }
};

// Retrieve data
const getData = async (key: string): Promise<string | null> => {
  try {
    const value = await AsyncStorage.getItem(key);
    return value;
  } catch (error) {
    console.error('Error getting data:', error);
    return null;
  }
};

// Remove data
const removeData = async (key: string) => {
  try {
    await AsyncStorage.removeItem(key);
  } catch (error) {
    console.error('Error removing data:', error);
  }
};

// Clear all data
const clearAll = async () => {
  try {
    await AsyncStorage.clear();
  } catch (error) {
    console.error('Error clearing storage:', error);
  }
};
```

### Storing Objects

```typescript
const storeObject = async (key: string, value: object) => {
  try {
    const jsonValue = JSON.stringify(value);
    await AsyncStorage.setItem(key, jsonValue);
  } catch (error) {
    console.error('Error storing object:', error);
  }
};

const getObject = async <T,>(key: string): Promise<T | null> => {
  try {
    const jsonValue = await AsyncStorage.getItem(key);
    return jsonValue != null ? JSON.parse(jsonValue) : null;
  } catch (error) {
    console.error('Error getting object:', error);
    return null;
  }
};

// Uso
interface User {
  id: string;
  name: string;
  email: string;
}

const saveUser = async (user: User) => {
  await storeObject('user', user);
};

const loadUser = async (): Promise<User | null> => {
  return await getObject<User>('user');
};
```

### Multiple Operations

```typescript
// Store multiple items
const storeMultiple = async (items: [string, string][]) => {
  try {
    await AsyncStorage.multiSet(items);
  } catch (error) {
    console.error('Error storing multiple:', error);
  }
};

// Get multiple items
const getMultiple = async (keys: string[]) => {
  try {
    const values = await AsyncStorage.multiGet(keys);
    return values;
  } catch (error) {
    console.error('Error getting multiple:', error);
    return [];
  }
};

// Remove multiple items
const removeMultiple = async (keys: string[]) => {
  try {
    await AsyncStorage.multiRemove(keys);
  } catch (error) {
    console.error('Error removing multiple:', error);
  }
};

// Uso
await storeMultiple([
  ['username', 'john'],
  ['email', 'john@example.com'],
  ['theme', 'dark']
]);

const values = await getMultiple(['username', 'email']);
// values = [['username', 'john'], ['email', 'john@example.com']]
```

### Custom Hook

```typescript
import { useState, useEffect } from 'react';

export const useAsyncStorage = <T,>(
  key: string,
  initialValue: T
): [T, (value: T) => Promise<void>, boolean] => {
  const [storedValue, setStoredValue] = useState<T>(initialValue);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const loadStoredValue = async () => {
      try {
        const item = await AsyncStorage.getItem(key);
        const value = item ? JSON.parse(item) : initialValue;
        setStoredValue(value);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };
    
    loadStoredValue();
  }, [key, initialValue]);
  
  const setValue = async (value: T) => {
    try {
      setStoredValue(value);
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue, loading];
};

// Uso
const SettingsScreen = () => {
  const [theme, setTheme, loading] = useAsyncStorage('theme', 'light');
  
  if (loading) return <ActivityIndicator />;
  
  return (
    <View>
      <Text>Current theme: {theme}</Text>
      <Button
        title="Toggle Theme"
        onPress={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      />
    </View>
  );
};
```

---

## 2. SQLite

### Installation

```bash
npm install react-native-sqlite-storage
```

### Setup Database

```typescript
import SQLite from 'react-native-sqlite-storage';

SQLite.enablePromise(true);

const DATABASE_NAME = 'myapp.db';

export const openDatabase = async () => {
  return SQLite.openDatabase({
    name: DATABASE_NAME,
    location: 'default'
  });
};

// Initialize tables
export const initDatabase = async () => {
  const db = await openDatabase();
  
  await db.executeSql(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      email TEXT UNIQUE NOT NULL,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  `);
  
  await db.executeSql(`
    CREATE TABLE IF NOT EXISTS todos (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      user_id INTEGER,
      title TEXT NOT NULL,
      completed BOOLEAN DEFAULT 0,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
      FOREIGN KEY (user_id) REFERENCES users(id)
    )
  `);
};
```

### CRUD Operations

```typescript
interface User {
  id?: number;
  name: string;
  email: string;
}

// Create
export const createUser = async (user: User): Promise<number> => {
  const db = await openDatabase();
  const [result] = await db.executeSql(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    [user.name, user.email]
  );
  return result.insertId;
};

// Read
export const getUsers = async (): Promise<User[]> => {
  const db = await openDatabase();
  const [results] = await db.executeSql('SELECT * FROM users');
  
  const users: User[] = [];
  for (let i = 0; i < results.rows.length; i++) {
    users.push(results.rows.item(i));
  }
  return users;
};

export const getUserById = async (id: number): Promise<User | null> => {
  const db = await openDatabase();
  const [results] = await db.executeSql(
    'SELECT * FROM users WHERE id = ?',
    [id]
  );
  
  if (results.rows.length > 0) {
    return results.rows.item(0);
  }
  return null;
};

// Update
export const updateUser = async (id: number, user: Partial<User>) => {
  const db = await openDatabase();
  
  const fields: string[] = [];
  const values: any[] = [];
  
  if (user.name) {
    fields.push('name = ?');
    values.push(user.name);
  }
  if (user.email) {
    fields.push('email = ?');
    values.push(user.email);
  }
  
  values.push(id);
  
  await db.executeSql(
    `UPDATE users SET ${fields.join(', ')} WHERE id = ?`,
    values
  );
};

// Delete
export const deleteUser = async (id: number) => {
  const db = await openDatabase();
  await db.executeSql('DELETE FROM users WHERE id = ?', [id]);
};
```

### Transactions

```typescript
export const createUserWithTodos = async (
  user: User,
  todos: string[]
) => {
  const db = await openDatabase();
  
  await db.transaction(async (tx) => {
    // Insert user
    const [userResult] = await tx.executeSql(
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [user.name, user.email]
    );
    
    const userId = userResult.insertId;
    
    // Insert todos
    for (const title of todos) {
      await tx.executeSql(
        'INSERT INTO todos (user_id, title) VALUES (?, ?)',
        [userId, title]
      );
    }
  });
};
```

---

## 3. Realm

### Installation

```bash
npm install realm
```

### Define Schema

```typescript
import Realm from 'realm';

class User extends Realm.Object {
  static schema = {
    name: 'User',
    primaryKey: 'id',
    properties: {
      id: 'string',
      name: 'string',
      email: 'string',
      age: 'int?',
      createdAt: { type: 'date', default: Date.now }
    }
  };
}

class Todo extends Realm.Object {
  static schema = {
    name: 'Todo',
    primaryKey: 'id',
    properties: {
      id: 'string',
      title: 'string',
      completed: { type: 'bool', default: false },
      user: 'User',
      createdAt: { type: 'date', default: Date.now }
    }
  };
}
```

### Open Realm

```typescript
let realm: Realm;

export const openRealm = async () => {
  if (!realm) {
    realm = await Realm.open({
      schema: [User, Todo],
      schemaVersion: 1
    });
  }
  return realm;
};
```

### CRUD Operations

```typescript
// Create
export const createUser = async (userData: {
  id: string;
  name: string;
  email: string;
}) => {
  const realm = await openRealm();
  
  realm.write(() => {
    realm.create('User', userData);
  });
};

// Read
export const getUsers = async () => {
  const realm = await openRealm();
  return realm.objects('User');
};

export const getUserById = async (id: string) => {
  const realm = await openRealm();
  return realm.objectForPrimaryKey('User', id);
};

// Update
export const updateUser = async (
  id: string,
  updates: Partial<User>
) => {
  const realm = await openRealm();
  
  realm.write(() => {
    const user = realm.objectForPrimaryKey('User', id);
    if (user) {
      Object.assign(user, updates);
    }
  });
};

// Delete
export const deleteUser = async (id: string) => {
  const realm = await openRealm();
  
  realm.write(() => {
    const user = realm.objectForPrimaryKey('User', id);
    if (user) {
      realm.delete(user);
    }
  });
};
```

### Queries

```typescript
// Filter
const activeUsers = realm.objects('User').filtered('age > 18');

// Sort
const sortedUsers = realm.objects('User').sorted('name');

// Complex queries
const results = realm.objects('Todo')
  .filtered('completed = false AND user.age > 20')
  .sorted('createdAt', true);

// Listeners
const users = realm.objects('User');

users.addListener((collection, changes) => {
  console.log(`${changes.insertions.length} insertions`);
  console.log(`${changes.modifications.length} modifications`);
  console.log(`${changes.deletions.length} deletions`);
});

// Remove listener
users.removeAllListeners();
```

---

## 4. WatermelonDB

### Installation

```bash
npm install @nozbe/watermelondb
```

### Define Models

```typescript
import { Model } from '@nozbe/watermelondb';
import { field, date, readonly } from '@nozbe/watermelondb/decorators';

export class User extends Model {
  static table = 'users';
  
  @field('name') name!: string;
  @field('email') email!: string;
  @field('age') age!: number;
  @readonly @date('created_at') createdAt!: Date;
  @readonly @date('updated_at') updatedAt!: Date;
}

export class Todo extends Model {
  static table = 'todos';
  
  @field('title') title!: string;
  @field('completed') completed!: boolean;
  @field('user_id') userId!: string;
  @readonly @date('created_at') createdAt!: Date;
  @readonly @date('updated_at') updatedAt!: Date;
}
```

### Schema

```typescript
import { appSchema, tableSchema } from '@nozbe/watermelondb';

export const schema = appSchema({
  version: 1,
  tables: [
    tableSchema({
      name: 'users',
      columns: [
        { name: 'name', type: 'string' },
        { name: 'email', type: 'string', isIndexed: true },
        { name: 'age', type: 'number' },
        { name: 'created_at', type: 'number' },
        { name: 'updated_at', type: 'number' }
      ]
    }),
    tableSchema({
      name: 'todos',
      columns: [
        { name: 'title', type: 'string' },
        { name: 'completed', type: 'boolean' },
        { name: 'user_id', type: 'string', isIndexed: true },
        { name: 'created_at', type: 'number' },
        { name: 'updated_at', type: 'number' }
      ]
    })
  ]
});
```

### Database Setup

```typescript
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { schema } from './schema';
import { User, Todo } from './models';

const adapter = new SQLiteAdapter({
  schema,
  dbName: 'myapp'
});

export const database = new Database({
  adapter,
  modelClasses: [User, Todo]
});
```

### CRUD Operations

```typescript
// Create
const createUser = async (name: string, email: string) => {
  await database.write(async () => {
    await database.get<User>('users').create((user) => {
      user.name = name;
      user.email = email;
      user.age = 25;
    });
  });
};

// Read
const getUsers = async () => {
  return await database.get<User>('users').query().fetch();
};

// Update
const updateUser = async (user: User, updates: Partial<User>) => {
  await database.write(async () => {
    await user.update((u) => {
      if (updates.name) u.name = updates.name;
      if (updates.email) u.email = updates.email;
    });
  });
};

// Delete
const deleteUser = async (user: User) => {
  await database.write(async () => {
    await user.markAsDeleted();
  });
};
```

### Queries

```typescript
import { Q } from '@nozbe/watermelondb';

// Simple query
const activeUsers = await database
  .get<User>('users')
  .query(Q.where('age', Q.gt(18)))
  .fetch();

// Complex query
const results = await database
  .get<Todo>('todos')
  .query(
    Q.where('completed', false),
    Q.where('user_id', userId),
    Q.sortBy('created_at', Q.desc)
  )
  .fetch();
```

---

## 5. Migrations

### SQLite Migration

```typescript
const migrateDatabase = async (db: SQLite.DatabaseConnection) => {
  const [result] = await db.executeSql(
    'SELECT version FROM schema_version'
  );
  
  const currentVersion = result.rows.item(0).version;
  
  if (currentVersion < 2) {
    await db.executeSql('ALTER TABLE users ADD COLUMN phone TEXT');
    await db.executeSql('UPDATE schema_version SET version = 2');
  }
  
  if (currentVersion < 3) {
    await db.executeSql(`
      CREATE TABLE IF NOT EXISTS categories (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL
      )
    `);
    await db.executeSql('UPDATE schema_version SET version = 3');
  }
};
```

### Realm Migration

```typescript
const realm = await Realm.open({
  schema: [User, Todo],
  schemaVersion: 2,
  migration: (oldRealm, newRealm) => {
    if (oldRealm.schemaVersion < 2) {
      const oldObjects = oldRealm.objects('User');
      const newObjects = newRealm.objects('User');
      
      for (let i = 0; i < oldObjects.length; i++) {
        newObjects[i].phone = '';
      }
    }
  }
});
```

### WatermelonDB Migration

```typescript
import { schemaMigrations } from '@nozbe/watermelondb/Schema/migrations';

export const migrations = schemaMigrations({
  migrations: [
    {
      toVersion: 2,
      steps: [
        {
          type: 'add_column',
          table: 'users',
          column: { name: 'phone', type: 'string' }
        }
      ]
    },
    {
      toVersion: 3,
      steps: [
        {
          type: 'create_table',
          table: {
            name: 'categories',
            columns: [
              { name: 'name', type: 'string' },
              { name: 'created_at', type: 'number' },
              { name: 'updated_at', type: 'number' }
            ]
          }
        }
      ]
    }
  ]
});
```

---

## 6. Caching

### API Response Cache

```typescript
interface CacheEntry<T> {
  data: T;
  timestamp: number;
}

class Cache {
  private cache = new Map<string, CacheEntry<any>>();
  private maxAge = 5 * 60 * 1000; // 5 minutes
  
  set<T>(key: string, data: T): void {
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }
  
  get<T>(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) return null;
    
    const age = Date.now() - entry.timestamp;
    if (age > this.maxAge) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data;
  }
  
  invalidate(key: string): void {
    this.cache.delete(key);
  }
  
  clear(): void {
    this.cache.clear();
  }
}

export const apiCache = new Cache();

// Uso
const fetchUsers = async () => {
  const cacheKey = 'users';
  const cached = apiCache.get(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const response = await apiClient.get('/users');
  apiCache.set(cacheKey, response.data);
  
  return response.data;
};
```

### Persistent Cache

```typescript
const CACHE_KEY_PREFIX = 'cache:';

const persistentCache = {
  async set<T>(key: string, data: T, maxAge = 3600000): Promise<void> {
    const cacheEntry = {
      data,
      timestamp: Date.now(),
      maxAge
    };
    await AsyncStorage.setItem(
      `${CACHE_KEY_PREFIX}${key}`,
      JSON.stringify(cacheEntry)
    );
  },
  
  async get<T>(key: string): Promise<T | null> {
    const item = await AsyncStorage.getItem(`${CACHE_KEY_PREFIX}${key}`);
    
    if (!item) return null;
    
    const { data, timestamp, maxAge } = JSON.parse(item);
    const age = Date.now() - timestamp;
    
    if (age > maxAge) {
      await this.invalidate(key);
      return null;
    }
    
    return data;
  },
  
  async invalidate(key: string): Promise<void> {
    await AsyncStorage.removeItem(`${CACHE_KEY_PREFIX}${key}`);
  },
  
  async clear(): Promise<void> {
    const keys = await AsyncStorage.getAllKeys();
    const cacheKeys = keys.filter(k => k.startsWith(CACHE_KEY_PREFIX));
    await AsyncStorage.multiRemove(cacheKeys);
  }
};
```

---

## 7. Offline-First

### Network State

```typescript
import NetInfo from '@react-native-community/netinfo';

export const useNetworkState = () => {
  const [isConnected, setIsConnected] = useState(true);
  
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsConnected(state.isConnected ?? false);
    });
    
    return unsubscribe;
  }, []);
  
  return isConnected;
};
```

### Offline Queue

```typescript
interface QueuedRequest {
  id: string;
  url: string;
  method: string;
  data?: any;
  timestamp: number;
}

class OfflineQueue {
  private queue: QueuedRequest[] = [];
  private processing = false;
  
  async add(request: Omit<QueuedRequest, 'id' | 'timestamp'>): Promise<void> {
    const queuedRequest: QueuedRequest = {
      ...request,
      id: Date.now().toString(),
      timestamp: Date.now()
    };
    
    this.queue.push(queuedRequest);
    await this.persist();
  }
  
  async process(): Promise<void> {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    
    while (this.queue.length > 0) {
      const request = this.queue[0];
      
      try {
        await apiClient.request({
          url: request.url,
          method: request.method,
          data: request.data
        });
        
        this.queue.shift();
        await this.persist();
      } catch (error) {
        console.error('Failed to process queued request:', error);
        break;
      }
    }
    
    this.processing = false;
  }
  
  private async persist(): Promise<void> {
    await AsyncStorage.setItem('offline_queue', JSON.stringify(this.queue));
  }
  
  async load(): Promise<void> {
    const data = await AsyncStorage.getItem('offline_queue');
    if (data) {
      this.queue = JSON.parse(data);
    }
  }
}

export const offlineQueue = new OfflineQueue();

// Uso
const createTodo = async (todo: Todo) => {
  if (!isConnected) {
    await offlineQueue.add({
      url: '/todos',
      method: 'POST',
      data: todo
    });
    return;
  }
  
  await apiClient.post('/todos', todo);
};

// Process queue when online
useEffect(() => {
  if (isConnected) {
    offlineQueue.process();
  }
}, [isConnected]);
```

---

## 8. Sync

### Last Sync Timestamp

```typescript
const getLastSync = async (): Promise<number> => {
  const timestamp = await AsyncStorage.getItem('last_sync');
  return timestamp ? parseInt(timestamp) : 0;
};

const setLastSync = async (timestamp: number): Promise<void> => {
  await AsyncStorage.setItem('last_sync', timestamp.toString());
};

const syncData = async () => {
  const lastSync = await getLastSync();
  
  const response = await apiClient.get('/sync', {
    params: { since: lastSync }
  });
  
  // Update local database
  await updateLocalDatabase(response.data);
  
  // Update last sync timestamp
  await setLastSync(Date.now());
};
```

---

## 9. Encryption

### Encrypted Storage

```bash
npm install react-native-encrypted-storage
```

```typescript
import EncryptedStorage from 'react-native-encrypted-storage';

const storeSecureData = async (key: string, value: string) => {
  await EncryptedStorage.setItem(key, value);
};

const getSecureData = async (key: string) => {
  return await EncryptedStorage.getItem(key);
};

// Uso per token JWT
await storeSecureData('access_token', token);
```

---

## 10. Best Practices

1. **Use AsyncStorage for:** User preferences, small data, simple key-value
2. **Use SQLite for:** Complex queries, relational data, large datasets
3. **Use Realm for:** Object-oriented data, reactive updates, offline-first
4. **Use WatermelonDB for:** High performance, large datasets, sync
5. **Always handle errors** in storage operations
6. **Implement migrations** for schema changes
7. **Cache API responses** intelligently
8. **Implement offline queue** for write operations
9. **Use encryption** for sensitive data
10. **Test storage on device**, not just simulator

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 AsyncStorage Settings

App settings con:
- Theme (light/dark)
- Language
- Notifications enabled
- Custom hook useSettings
- Persist and load

### Esercizio 2: 🟡 SQLite Todo App

Todo app con SQLite:
- Create/Read/Update/Delete
- Categories
- Search and filter
- Export to JSON

### Esercizio 3: 🟡 Realm Notes App

Notes app con Realm:
- Create/edit notes
- Rich text
- Categories and tags
- Real-time sync
- Search

### Esercizio 4: 🔴 Offline-First App

App offline-first completa:
- Local database (SQLite o Realm)
- API cache
- Offline queue
- Network state detection
- Sync when online
- Conflict resolution

### Esercizio 5: 🔴 WatermelonDB Social App

Social app con WatermelonDB:
- Users, posts, comments
- Complex queries
- Optimistic updates
- Server sync
- Performance optimization

---

## 📝 Quiz

1. **AsyncStorage è:**
   - [ ] Database relazionale
   - [x] Key-value storage
   - [ ] NoSQL database
   - [ ] In-memory cache

2. **SQLite supporta:**
   - [ ] Solo key-value
   - [x] Query SQL complesse
   - [ ] Solo JavaScript
   - [ ] Solo numeri

3. **Realm è:**
   - [ ] SQL database
   - [x] Object database
   - [ ] Key-value storage
   - [ ] Memory storage

4. **WatermelonDB è ottimizzato per:**
   - [ ] Small data
   - [x] Large datasets
   - [ ] Solo Android
   - [ ] Solo iOS

5. **Encryption è importante per:**
   - [ ] Performance
   - [ ] Cache
   - [x] Dati sensibili (token, password)
   - [ ] UI state

**Risposte**: 1-b, 2-b, 3-b, 4-b, 5-c

---

## 🔗 Risorse

- [AsyncStorage Docs](https://react-native-async-storage.github.io/async-storage/)
- [SQLite Tutorial](https://github.com/andpor/react-native-sqlite-storage)
- [Realm Documentation](https://realm.io/docs/javascript/latest/)
- [WatermelonDB Docs](https://nozbe.github.io/WatermelonDB/)

---

## ✅ Checklist

- [ ] AsyncStorage implementato
- [ ] SQLite o Realm usato
- [ ] Migrazioni gestite
- [ ] Caching implementato
- [ ] Offline-first funzionante
- [ ] Sync implementato
- [ ] Encryption per dati sensibili
- [ ] Error handling completo
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 💾 **AsyncStorage** per dati semplici
2. 🗄️ **SQLite** per query complesse
3. 📦 **Realm** per object database
4. ⚡ **WatermelonDB** per performance
5. 🔄 **Migrazioni** per schema changes
6. 🎯 **Cache** API responses
7. 📴 **Offline-first** con queue
8. 🔐 **Encryption** per sicurezza

---

**Magnifico! 🎉** Ora padroneggi la persistenza dati. Prossimo: GraphQL e Apollo Client!

[← Precedente: State Management](12_State_Management.md) | [Torna all'Indice](../README.md) | [Prossimo: GraphQL e Apollo →](14_GraphQL_Apollo.md)
