# Capitolo 23: Pattern Avanzati e Architetture

[← Precedente: Monitoring e Analytics](../05%20-%20Produzione%20e%20Qualità/22_Monitoring.md) | [Torna all'Indice](../README.md) | [Prossimo: New Architecture →](24_New_Architecture.md)

---

## Descrizione

Quando le app crescono, serve architettura solida: pattern per organizzare codice, separare concerns, facilitare testing e manutenzione. Clean Architecture, MVVM, MVI sono approcci battle-tested. Advanced hooks, HOCs, render props, compound components offrono riusabilità. Design patterns risolvono problemi comuni.

In questo capitolo imparerai architetture scalabili (Clean Architecture, MVVM, MVI), advanced React patterns (custom hooks, HOCs, render props, compound components), dependency injection, design patterns (Factory, Singleton, Observer, Strategy), e best practices per code organization.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Implementare Clean Architecture
- [ ] Applicare MVVM pattern
- [ ] Usare MVI pattern
- [ ] Creare advanced hooks
- [ ] Implementare HOCs
- [ ] Usare render props
- [ ] Creare compound components
- [ ] Implementare dependency injection
- [ ] Applicare design patterns
- [ ] Organizzare codice scalabile

## 📚 Argomenti Teorici

1. [Clean Architecture](#1-clean-architecture)
2. [MVVM Pattern](#2-mvvm)
3. [MVI Pattern](#3-mvi)
4. [Advanced Hooks](#4-advanced-hooks)
5. [Higher-Order Components](#5-hocs)
6. [Render Props](#6-render-props)
7. [Compound Components](#7-compound-components)
8. [Dependency Injection](#8-dependency-injection)
9. [Design Patterns](#9-design-patterns)
10. [Code Organization](#10-code-organization)

---

## 1. Clean Architecture

### Architecture Layers

```
┌─────────────────────────────────────┐
│     Presentation (UI)               │
│  - Components, Screens, Hooks       │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Domain (Business Logic)         │
│  - Use Cases, Entities, Interfaces  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Data (External)                 │
│  - API, Database, Storage           │
└─────────────────────────────────────┘
```

### Entities

```typescript
// domain/entities/User.ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  createdAt: Date;
}

// domain/entities/Post.ts
export interface Post {
  id: string;
  authorId: string;
  content: string;
  images: string[];
  likes: number;
  comments: number;
  createdAt: Date;
}
```

### Use Cases

```typescript
// domain/usecases/GetUserUseCase.ts
import { User } from '../entities/User';
import { UserRepository } from '../repositories/UserRepository';

export class GetUserUseCase {
  constructor(private userRepository: UserRepository) {}
  
  async execute(userId: string): Promise<User> {
    if (!userId) {
      throw new Error('User ID is required');
    }
    
    return await this.userRepository.getById(userId);
  }
}

// domain/usecases/CreatePostUseCase.ts
import { Post } from '../entities/Post';
import { PostRepository } from '../repositories/PostRepository';

export class CreatePostUseCase {
  constructor(private postRepository: PostRepository) {}
  
  async execute(content: string, authorId: string): Promise<Post> {
    if (!content.trim()) {
      throw new Error('Content cannot be empty');
    }
    
    const post: Partial<Post> = {
      content,
      authorId,
      images: [],
      likes: 0,
      comments: 0,
      createdAt: new Date()
    };
    
    return await this.postRepository.create(post);
  }
}
```

### Repositories (Interfaces)

```typescript
// domain/repositories/UserRepository.ts
import { User } from '../entities/User';

export interface UserRepository {
  getById(id: string): Promise<User>;
  getAll(): Promise<User[]>;
  create(user: Partial<User>): Promise<User>;
  update(id: string, user: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;
}

// data/repositories/UserRepositoryImpl.ts
import { UserRepository } from '../../domain/repositories/UserRepository';
import { User } from '../../domain/entities/User';
import { ApiClient } from '../api/ApiClient';

export class UserRepositoryImpl implements UserRepository {
  constructor(private apiClient: ApiClient) {}
  
  async getById(id: string): Promise<User> {
    const response = await this.apiClient.get(`/users/${id}`);
    return this.mapToUser(response.data);
  }
  
  async getAll(): Promise<User[]> {
    const response = await this.apiClient.get('/users');
    return response.data.map(this.mapToUser);
  }
  
  async create(user: Partial<User>): Promise<User> {
    const response = await this.apiClient.post('/users', user);
    return this.mapToUser(response.data);
  }
  
  async update(id: string, user: Partial<User>): Promise<User> {
    const response = await this.apiClient.put(`/users/${id}`, user);
    return this.mapToUser(response.data);
  }
  
  async delete(id: string): Promise<void> {
    await this.apiClient.delete(`/users/${id}`);
  }
  
  private mapToUser(data: any): User {
    return {
      id: data.id,
      email: data.email,
      name: data.name,
      avatar: data.avatar,
      createdAt: new Date(data.created_at)
    };
  }
}
```

### Presentation Layer

```typescript
// presentation/screens/UserProfileScreen.tsx
import React, { useEffect, useState } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';
import { GetUserUseCase } from '../../domain/usecases/GetUserUseCase';
import { User } from '../../domain/entities/User';

interface Props {
  userId: string;
  getUserUseCase: GetUserUseCase;
}

export const UserProfileScreen: React.FC<Props> = ({ userId, getUserUseCase }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');
  
  useEffect(() => {
    loadUser();
  }, [userId]);
  
  const loadUser = async () => {
    try {
      setLoading(true);
      const userData = await getUserUseCase.execute(userId);
      setUser(userData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  if (!user) return <Text>User not found</Text>;
  
  return (
    <View>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
    </View>
  );
};
```

---

## 2. MVVM

### Model

```typescript
// models/TodoModel.ts
export interface TodoModel {
  id: string;
  title: string;
  completed: boolean;
  createdAt: Date;
}
```

### ViewModel

```typescript
// viewmodels/TodoViewModel.ts
import { useState, useCallback } from 'react';
import { TodoModel } from '../models/TodoModel';
import { TodoService } from '../services/TodoService';

export const useTodoViewModel = (todoService: TodoService) => {
  const [todos, setTodos] = useState<TodoModel[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const loadTodos = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await todoService.getAll();
      setTodos(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [todoService]);
  
  const addTodo = useCallback(async (title: string) => {
    try {
      const newTodo = await todoService.create(title);
      setTodos(prev => [...prev, newTodo]);
    } catch (err) {
      setError(err.message);
    }
  }, [todoService]);
  
  const toggleTodo = useCallback(async (id: string) => {
    try {
      const todo = todos.find(t => t.id === id);
      if (!todo) return;
      
      const updated = await todoService.update(id, {
        completed: !todo.completed
      });
      
      setTodos(prev => prev.map(t => t.id === id ? updated : t));
    } catch (err) {
      setError(err.message);
    }
  }, [todos, todoService]);
  
  const deleteTodo = useCallback(async (id: string) => {
    try {
      await todoService.delete(id);
      setTodos(prev => prev.filter(t => t.id !== id));
    } catch (err) {
      setError(err.message);
    }
  }, [todoService]);
  
  return {
    todos,
    loading,
    error,
    loadTodos,
    addTodo,
    toggleTodo,
    deleteTodo
  };
};
```

### View

```typescript
// views/TodoListView.tsx
import React, { useEffect } from 'react';
import { View, FlatList, ActivityIndicator } from 'react-native';
import { useTodoViewModel } from '../viewmodels/TodoViewModel';
import { todoService } from '../services/TodoService';

export const TodoListView: React.FC = () => {
  const {
    todos,
    loading,
    error,
    loadTodos,
    addTodo,
    toggleTodo,
    deleteTodo
  } = useTodoViewModel(todoService);
  
  useEffect(() => {
    loadTodos();
  }, [loadTodos]);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <View>
      <FlatList
        data={todos}
        renderItem={({ item }) => (
          <TodoItem
            todo={item}
            onToggle={() => toggleTodo(item.id)}
            onDelete={() => deleteTodo(item.id)}
          />
        )}
        keyExtractor={item => item.id}
      />
      <AddTodoForm onAdd={addTodo} />
    </View>
  );
};
```

---

## 3. MVI

### Model (State)

```typescript
// mvi/models/CounterModel.ts
export interface CounterModel {
  count: number;
  loading: boolean;
  error: string | null;
}

export const initialCounterModel: CounterModel = {
  count: 0,
  loading: false,
  error: null
};
```

### Intent (Actions)

```typescript
// mvi/intents/CounterIntent.ts
export type CounterIntent =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' }
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: number }
  | { type: 'FETCH_ERROR'; payload: string };
```

### Reducer

```typescript
// mvi/reducers/counterReducer.ts
import { CounterModel, initialCounterModel } from '../models/CounterModel';
import { CounterIntent } from '../intents/CounterIntent';

export const counterReducer = (
  state: CounterModel,
  intent: CounterIntent
): CounterModel => {
  switch (intent.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    
    case 'RESET':
      return { ...state, count: 0 };
    
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, count: intent.payload };
    
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: intent.payload };
    
    default:
      return state;
  }
};
```

### View

```typescript
// mvi/views/CounterView.tsx
import React, { useReducer, useCallback } from 'react';
import { View, Text, Button } from 'react-native';
import { initialCounterModel } from '../models/CounterModel';
import { counterReducer } from '../reducers/counterReducer';

export const CounterView: React.FC = () => {
  const [state, dispatch] = useReducer(counterReducer, initialCounterModel);
  
  const fetchCount = useCallback(async () => {
    dispatch({ type: 'FETCH_START' });
    try {
      const response = await fetch('/api/count');
      const data = await response.json();
      dispatch({ type: 'FETCH_SUCCESS', payload: data.count });
    } catch (error) {
      dispatch({ type: 'FETCH_ERROR', payload: error.message });
    }
  }, []);
  
  return (
    <View>
      <Text>Count: {state.count}</Text>
      {state.loading && <Text>Loading...</Text>}
      {state.error && <Text>Error: {state.error}</Text>}
      
      <Button title="+" onPress={() => dispatch({ type: 'INCREMENT' })} />
      <Button title="-" onPress={() => dispatch({ type: 'DECREMENT' })} />
      <Button title="Reset" onPress={() => dispatch({ type: 'RESET' })} />
      <Button title="Fetch" onPress={fetchCount} />
    </View>
  );
};
```

---

## 4. Advanced Hooks

### useDebounce

```typescript
import { useState, useEffect } from 'react';

export const useDebounce = <T,>(value: T, delay: number): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(handler);
  }, [value, delay]);
  
  return debouncedValue;
};

// Usage
const SearchScreen = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return <TextInput value={searchTerm} onChangeText={setSearchTerm} />;
};
```

### usePrevious

```typescript
import { useRef, useEffect } from 'react';

export const usePrevious = <T,>(value: T): T | undefined => {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
};

// Usage
const Counter = ({ count }: { count: number }) => {
  const prevCount = usePrevious(count);
  
  return (
    <View>
      <Text>Current: {count}</Text>
      <Text>Previous: {prevCount}</Text>
    </View>
  );
};
```

### useAsync

```typescript
import { useState, useEffect, useCallback } from 'react';

interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

export const useAsync = <T,>(
  asyncFunction: () => Promise<T>,
  immediate = true
) => {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: false,
    error: null
  });
  
  const execute = useCallback(async () => {
    setState({ data: null, loading: true, error: null });
    
    try {
      const data = await asyncFunction();
      setState({ data, loading: false, error: null });
      return data;
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return { ...state, execute };
};

// Usage
const UserProfile = ({ userId }: { userId: string }) => {
  const { data, loading, error } = useAsync(
    () => fetchUser(userId),
    true
  );
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error.message}</Text>;
  if (!data) return null;
  
  return <UserCard user={data} />;
};
```

---

## 5. HOCs

### withLoading

```typescript
import React from 'react';
import { ActivityIndicator, View } from 'react-native';

interface WithLoadingProps {
  loading: boolean;
}

export const withLoading = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P & WithLoadingProps) => {
    const { loading, ...rest } = props;
    
    if (loading) {
      return (
        <View style={{ flex: 1, justifyContent: 'center' }}>
          <ActivityIndicator size="large" />
        </View>
      );
    }
    
    return <Component {...(rest as P)} />;
  };
};

// Usage
const UserList = ({ users }) => {
  return <FlatList data={users} renderItem={...} />;
};

const UserListWithLoading = withLoading(UserList);

<UserListWithLoading loading={isLoading} users={users} />
```

### withAuth

```typescript
import React from 'react';
import { useAuth } from '../hooks/useAuth';
import { LoginScreen } from '../screens/LoginScreen';

export const withAuth = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P) => {
    const { user, loading } = useAuth();
    
    if (loading) return <ActivityIndicator />;
    if (!user) return <LoginScreen />;
    
    return <Component {...props} user={user} />;
  };
};

// Usage
const ProfileScreen = ({ user }) => {
  return <Text>Welcome, {user.name}!</Text>;
};

export default withAuth(ProfileScreen);
```

---

## 6. Render Props

### Mouse Tracker

```typescript
import React, { useState } from 'react';
import { View, PanResponder } from 'react-native';

interface MousePosition {
  x: number;
  y: number;
}

interface Props {
  children: (position: MousePosition) => React.ReactNode;
}

export const MouseTracker: React.FC<Props> = ({ children }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  const panResponder = PanResponder.create({
    onMoveShouldSetPanResponder: () => true,
    onPanResponderMove: (_, gestureState) => {
      setPosition({ x: gestureState.moveX, y: gestureState.moveY });
    }
  });
  
  return (
    <View {...panResponder.panHandlers} style={{ flex: 1 }}>
      {children(position)}
    </View>
  );
};

// Usage
<MouseTracker>
  {({ x, y }) => (
    <View>
      <Text>X: {x}, Y: {y}</Text>
    </View>
  )}
</MouseTracker>
```

---

## 7. Compound Components

### Accordion

```typescript
import React, { createContext, useContext, useState } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

interface AccordionContextType {
  openId: string | null;
  setOpenId: (id: string | null) => void;
}

const AccordionContext = createContext<AccordionContextType | null>(null);

export const Accordion: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [openId, setOpenId] = useState<string | null>(null);
  
  return (
    <AccordionContext.Provider value={{ openId, setOpenId }}>
      <View>{children}</View>
    </AccordionContext.Provider>
  );
};

interface ItemProps {
  id: string;
  title: string;
  children: React.ReactNode;
}

Accordion.Item = ({ id, title, children }: ItemProps) => {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('Item must be used within Accordion');
  
  const { openId, setOpenId } = context;
  const isOpen = openId === id;
  
  return (
    <View>
      <TouchableOpacity onPress={() => setOpenId(isOpen ? null : id)}>
        <Text>{title}</Text>
      </TouchableOpacity>
      {isOpen && <View>{children}</View>}
    </View>
  );
};

// Usage
<Accordion>
  <Accordion.Item id="1" title="Section 1">
    <Text>Content 1</Text>
  </Accordion.Item>
  <Accordion.Item id="2" title="Section 2">
    <Text>Content 2</Text>
  </Accordion.Item>
</Accordion>
```

---

## 8. Dependency Injection

### DI Container

```typescript
// di/Container.ts
type Constructor<T> = new (...args: any[]) => T;

export class Container {
  private services = new Map<string, any>();
  private singletons = new Map<string, any>();
  
  register<T>(name: string, factory: () => T): void {
    this.services.set(name, factory);
  }
  
  registerSingleton<T>(name: string, factory: () => T): void {
    this.services.set(name, factory);
    this.singletons.set(name, null);
  }
  
  resolve<T>(name: string): T {
    if (this.singletons.has(name)) {
      let instance = this.singletons.get(name);
      if (!instance) {
        const factory = this.services.get(name);
        instance = factory();
        this.singletons.set(name, instance);
      }
      return instance;
    }
    
    const factory = this.services.get(name);
    if (!factory) {
      throw new Error(`Service ${name} not found`);
    }
    
    return factory();
  }
}

// Setup
const container = new Container();

container.registerSingleton('apiClient', () => new ApiClient());
container.registerSingleton('userRepository', () => 
  new UserRepositoryImpl(container.resolve('apiClient'))
);
container.register('getUserUseCase', () => 
  new GetUserUseCase(container.resolve('userRepository'))
);

export default container;
```

### React Context DI

```typescript
// di/DIContext.tsx
import React, { createContext, useContext } from 'react';
import container from './Container';

const DIContext = createContext(container);

export const DIProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <DIContext.Provider value={container}>{children}</DIContext.Provider>;
};

export const useDI = () => {
  const context = useContext(DIContext);
  if (!context) throw new Error('useDI must be used within DIProvider');
  return context;
};

export const useService = <T,>(name: string): T => {
  const container = useDI();
  return container.resolve<T>(name);
};

// Usage
const UserProfile = ({ userId }) => {
  const getUserUseCase = useService<GetUserUseCase>('getUserUseCase');
  
  // Use getUserUseCase...
};
```

---

## 9. Design Patterns

### Singleton

```typescript
export class Logger {
  private static instance: Logger;
  
  private constructor() {}
  
  static getInstance(): Logger {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }
  
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

// Usage
const logger = Logger.getInstance();
logger.log('Application started');
```

### Factory

```typescript
interface Button {
  render(): JSX.Element;
}

class PrimaryButton implements Button {
  constructor(private text: string) {}
  
  render() {
    return <Button title={this.text} color="blue" />;
  }
}

class SecondaryButton implements Button {
  constructor(private text: string) {}
  
  render() {
    return <Button title={this.text} color="gray" />;
  }
}

class ButtonFactory {
  createButton(type: 'primary' | 'secondary', text: string): Button {
    switch (type) {
      case 'primary':
        return new PrimaryButton(text);
      case 'secondary':
        return new SecondaryButton(text);
      default:
        throw new Error('Invalid button type');
    }
  }
}

// Usage
const factory = new ButtonFactory();
const button = factory.createButton('primary', 'Click me');
```

### Observer

```typescript
interface Observer {
  update(data: any): void;
}

class Subject {
  private observers: Observer[] = [];
  
  attach(observer: Observer): void {
    this.observers.push(observer);
  }
  
  detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  
  notify(data: any): void {
    this.observers.forEach(observer => observer.update(data));
  }
}

// Usage
class UserObserver implements Observer {
  update(user: User): void {
    console.log('User updated:', user);
  }
}

const userSubject = new Subject();
const observer = new UserObserver();
userSubject.attach(observer);
userSubject.notify({ id: '1', name: 'John' });
```

---

## 10. Code Organization

### Feature-based Structure

```
src/
  features/
    auth/
      components/
        LoginForm.tsx
        SignupForm.tsx
      screens/
        LoginScreen.tsx
        SignupScreen.tsx
      hooks/
        useAuth.ts
      api/
        authApi.ts
      types/
        auth.types.ts
      index.ts
    
    posts/
      components/
      screens/
      hooks/
      api/
      types/
      index.ts
  
  shared/
    components/
    hooks/
    utils/
    types/
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Clean Architecture

Implementa user management:
- Entities (User)
- Use cases (GetUser, CreateUser)
- Repository interface
- API implementation
- UI screen

### Esercizio 2: 🟡 MVVM Todo App

Todo app con MVVM:
- TodoModel
- TodoViewModel
- TodoView
- Service layer
- Complete CRUD

### Esercizio 3: 🟡 Advanced Hooks

Crea custom hooks:
- useDebounce
- usePrevious
- useAsync
- useLocalStorage
- Test in real components

### Esercizio 4: 🔴 Dependency Injection

DI system completo:
- DI Container
- Service registration
- React integration
- Multiple services
- Singleton pattern

### Esercizio 5: 🔴 Complete Architecture

App completa con:
- Clean Architecture
- MVVM pattern
- Custom hooks
- HOCs
- Compound components
- DI container
- Design patterns
- Scalable structure

---

## 📝 Quiz

1. **Clean Architecture separa:**
   - [x] UI, business logic, data
   - [ ] Solo UI e data
   - [ ] Solo components
   - [ ] Solo services

2. **MVVM sta per:**
   - [ ] Model View Manager
   - [x] Model View ViewModel
   - [ ] Module View Model
   - [ ] Model Visual Manager

3. **HOC è:**
   - [ ] High Order Class
   - [x] Higher-Order Component
   - [ ] Hook Order Component
   - [ ] High Object Component

4. **Render props permette:**
   - [x] Condividere logica via function prop
   - [ ] Solo rendering
   - [ ] Solo styling
   - [ ] Solo events

5. **Singleton garantisce:**
   - [x] Una sola istanza di classe
   - [ ] Multiple istanze
   - [ ] Solo factory
   - [ ] Solo pattern

**Risposte**: 1-a, 2-b, 3-b, 4-a, 5-a

---

## 🔗 Risorse

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [React Patterns](https://reactpatterns.com/)
- [Design Patterns](https://refactoring.guru/design-patterns)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)

---

## ✅ Checklist

- [ ] Clean Architecture implementata
- [ ] MVVM pattern applicato
- [ ] MVI pattern compreso
- [ ] Advanced hooks creati
- [ ] HOCs implementati
- [ ] Render props usati
- [ ] Compound components creati
- [ ] DI implementato
- [ ] Design patterns applicati
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🏗️ **Clean Architecture** separa concerns
2. 📐 **MVVM** separa UI da logic
3. 🔄 **MVI** unidirezionale e predictable
4. 🪝 **Custom hooks** riutilizzabili
5. 🎨 **HOCs** aggiungono comportamenti
6. 🔀 **Render props** condividono logica
7. 🧩 **Compound** componenti flessibili
8. 💉 **DI** decoupling e testability
9. 🎯 **Patterns** soluzioni provate
10. 📁 **Organization** feature-based scalabile

---

**Straordinario! 🎉** Padroneggi pattern e architetture! Prossimo: New Architecture!

[← Precedente: Monitoring e Analytics](../05%20-%20Produzione%20e%20Qualità/22_Monitoring.md) | [Torna all'Indice](../README.md) | [Prossimo: New Architecture →](24_New_Architecture.md)
