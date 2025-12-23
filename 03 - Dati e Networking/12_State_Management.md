# Capitolo 12: State Management Globale

[← Precedente: Networking e API](11_Networking_API.md) | [Torna all'Indice](../README.md) | [Prossimo: Persistenza Dati →](13_Persistenza_Dati.md)

---

## Descrizione

Man mano che un'app cresce, gestire lo state diventa complesso. Passare props attraverso molti livelli (prop drilling) diventa ingestibile. Context API, Redux, Zustand, e altre librerie offrono soluzioni per gestire lo state globale in modo efficiente.

In questo capitolo imparerai a scegliere la soluzione giusta, implementare Context API, usare Redux Toolkit, esplorare alternative moderne come Zustand e Jotai, e ottimizzare performance dello state management.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Utilizzare Context API efficacemente
- [ ] Implementare Redux Toolkit
- [ ] Gestire async actions con Redux Thunk
- [ ] Usare Redux Persist
- [ ] Implementare Zustand
- [ ] Esplorare Jotai e Recoil
- [ ] Scegliere la soluzione giusta
- [ ] Ottimizzare re-renders
- [ ] Debuggare state management
- [ ] Implementare middleware custom

## 📚 Argomenti Teorici

1. [Context API](#1-context-api)
2. [Redux Toolkit](#2-redux-toolkit)
3. [Redux Thunk](#3-redux-thunk)
4. [Redux Persist](#4-redux-persist)
5. [Zustand](#5-zustand)
6. [Jotai](#6-jotai)
7. [Recoil](#7-recoil)
8. [Performance Optimization](#8-optimization)
9. [DevTools](#9-devtools)
10. [Choosing the Right Solution](#10-choosing)

---

## 1. Context API

Context API è la soluzione nativa di React per condividere dati tra componenti senza prop drilling. È ideale per state globale semplice (tema, auth, lingua).

**Quando usare Context API:**
- ✅ State globale semplice (tema, auth, preferenze)
- ✅ Dati che cambiano raramente
- ✅ Evitare prop drilling (props passati attraverso molti livelli)
- ❌ Non per state che cambia frequentemente (causa re-render di tutti i consumer)
- ❌ Non per app complesse con molte azioni (meglio Redux/Zustand)

### Basic Context

Context API richiede 3 step: **Create → Provide → Consume**

```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. DEFINIZIONE TIPI: TypeScript per type-safety
interface User {
  id: string;
  name: string;
  email: string;
}

// Tipo del context: definisce quali dati/funzioni sono disponibili
interface AuthContextType {
  user: User | null;        // Stato: utente corrente
  login: (user: User) => void;  // Azione: login
  logout: () => void;           // Azione: logout
  isAuthenticated: boolean;     // Computed: derivato da user
}

// 2. CREATE CONTEXT: Crea il context con valore iniziale undefined
// undefined forza l'uso dentro AuthProvider (safety check)
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// 3. PROVIDER COMPONENT: Wrap dell'app che fornisce il context
export const AuthProvider = ({ children }: { children: ReactNode }) => {
  // State locale del provider (potrebbe essere Redux store, API, etc.)
  const [user, setUser] = useState<User | null>(null);
  
  // Funzione login: aggiorna lo state
  const login = (userData: User) => {
    setUser(userData);
    // Potresti anche: salvare in AsyncStorage, chiamare API, etc.
  };
  
  // Funzione logout: resetta lo state
  const logout = () => {
    setUser(null);
    // Potresti anche: rimuovere token, navigare a login screen, etc.
  };
  
  // Crea oggetto value con tutti i dati/funzioni da condividere
  // ⚠️ IMPORTANTE: value è ricreato ad ogni render del provider!
  // Se user non cambia, considera useMemo per ottimizzare
  const value = {
    user,
    login,
    logout,
    isAuthenticated: !!user  // !! converte in boolean (null → false, object → true)
  };
  
  // Provider rende value disponibile a tutti i children
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

// 4. CUSTOM HOOK: Astrae useContext per DX migliore e error handling
export const useAuth = () => {
  const context = useContext(AuthContext);
  
  // Error handling: throw se usato fuori dal Provider
  // Previene bugs silenziosi (undefined)
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  
  return context;
};

// 5. USO: Wrap app root con Provider
const App = () => (
  <AuthProvider>
    {/* Tutti i componenti qui dentro possono usare useAuth() */}
    <Navigation />
  </AuthProvider>
);

// 6. CONSUME: Usa il hook in qualsiasi componente nested
const ProfileScreen = () => {
  // Destructure solo ciò che serve (non causa re-render se altro cambia)
  const { user, logout } = useAuth();
  
  return (
    <View>
      <Text>Welcome, {user?.name}</Text>
      <Button title="Logout" onPress={logout} />
    </View>
  );
};
```

**📝 Note importanti:**
- Context NON è ottimizzato per aggiornamenti frequenti (ogni cambio fa re-render di TUTTI i consumer)
- Dividi in multiple contexts se hai dati che cambiano a frequenze diverse
- Provider deve stare più in alto possibile nella component tree
- Custom hook (`useAuth`) è best practice per error handling e autocomplete

### Multiple Contexts

```typescript
// ThemeContext
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// App con multiple providers
const App = () => (
  <AuthProvider>
    <ThemeProvider>
      <LanguageProvider>
        <Navigation />
      </LanguageProvider>
    </ThemeProvider>
  </AuthProvider>
);
```

### Context with Reducer

```typescript
import { createContext, useReducer, useContext } from 'react';

interface State {
  count: number;
  todos: Todo[];
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'ADD_TODO'; payload: Todo }
  | { type: 'REMOVE_TODO'; payload: string };

const initialState: State = {
  count: 0,
  todos: []
};

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'ADD_TODO':
      return { ...state, todos: [...state.todos, action.payload] };
    case 'REMOVE_TODO':
      return {
        ...state,
        todos: state.todos.filter(t => t.id !== action.payload)
      };
    default:
      return state;
  }
};

const AppContext = createContext<{
  state: State;
  dispatch: React.Dispatch<Action>;
} | undefined>(undefined);

export const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
};

export const useAppContext = () => {
  const context = useContext(AppContext);
  if (!context) throw new Error('useAppContext must be within AppProvider');
  return context;
};

// Uso
const Counter = () => {
  const { state, dispatch } = useAppContext();
  
  return (
    <View>
      <Text>Count: {state.count}</Text>
      <Button title="+" onPress={() => dispatch({ type: 'INCREMENT' })} />
      <Button title="-" onPress={() => dispatch({ type: 'DECREMENT' })} />
    </View>
  );
};
```

---

## 2. Redux Toolkit

### Installation

```bash
npm install @reduxjs/toolkit react-redux
```

### Store Setup

```typescript
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import todosReducer from './slices/todosSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    todos: todosReducer
  }
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Create Slice

```typescript
// slices/authSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  loading: boolean;
}

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  loading: false
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    loginStart: (state) => {
      state.loading = true;
    },
    loginSuccess: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
      state.isAuthenticated = true;
      state.loading = false;
    },
    loginFailure: (state) => {
      state.loading = false;
    },
    logout: (state) => {
      state.user = null;
      state.isAuthenticated = false;
    }
  }
});

export const { loginStart, loginSuccess, loginFailure, logout } = authSlice.actions;
export default authSlice.reducer;
```

### Provider Setup

```typescript
// App.tsx
import { Provider } from 'react-redux';
import { store } from './store';

const App = () => {
  return (
    <Provider store={store}>
      <Navigation />
    </Provider>
  );
};
```

### Hooks

```typescript
// hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// Uso nei componenti
import { useAppSelector, useAppDispatch } from './hooks';
import { loginSuccess, logout } from './slices/authSlice';

const ProfileScreen = () => {
  const user = useAppSelector(state => state.auth.user);
  const dispatch = useAppDispatch();
  
  const handleLogout = () => {
    dispatch(logout());
  };
  
  return (
    <View>
      <Text>Welcome, {user?.name}</Text>
      <Button title="Logout" onPress={handleLogout} />
    </View>
  );
};
```

---

## 3. Redux Thunk

### Async Actions

```typescript
// slices/todosSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { apiClient } from '../api';

interface Todo {
  id: string;
  title: string;
  completed: boolean;
}

interface TodosState {
  items: Todo[];
  loading: boolean;
  error: string | null;
}

const initialState: TodosState = {
  items: [],
  loading: false,
  error: null
};

// Async thunk
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async (userId: string) => {
    const response = await apiClient.get(`/users/${userId}/todos`);
    return response.data;
  }
);

export const addTodo = createAsyncThunk(
  'todos/addTodo',
  async (todo: Omit<Todo, 'id'>) => {
    const response = await apiClient.post('/todos', todo);
    return response.data;
  }
);

const todosSlice = createSlice({
  name: 'todos',
  initialState,
  reducers: {
    toggleTodo: (state, action: PayloadAction<string>) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    }
  },
  extraReducers: (builder) => {
    builder
      // Fetch todos
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch todos';
      })
      // Add todo
      .addCase(addTodo.fulfilled, (state, action) => {
        state.items.push(action.payload);
      });
  }
});

export const { toggleTodo } = todosSlice.actions;
export default todosSlice.reducer;
```

### Using Thunks

```typescript
const TodosScreen = () => {
  const { items, loading, error } = useAppSelector(state => state.todos);
  const dispatch = useAppDispatch();
  
  useEffect(() => {
    dispatch(fetchTodos('user123'));
  }, [dispatch]);
  
  const handleAddTodo = (title: string) => {
    dispatch(addTodo({ title, completed: false }));
  };
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <FlatList
      data={items}
      renderItem={({ item }) => (
        <TodoItem
          todo={item}
          onToggle={() => dispatch(toggleTodo(item.id))}
        />
      )}
    />
  );
};
```

---

## 4. Redux Persist

### Installation

```bash
npm install redux-persist
npm install @react-native-async-storage/async-storage
```

### Setup

```typescript
import { configureStore } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import AsyncStorage from '@react-native-async-storage/async-storage';
import authReducer from './slices/authSlice';

const persistConfig = {
  key: 'root',
  storage: AsyncStorage,
  whitelist: ['auth'] // Only persist auth slice
};

const persistedAuthReducer = persistReducer(persistConfig, authReducer);

export const store = configureStore({
  reducer: {
    auth: persistedAuthReducer,
    todos: todosReducer // Not persisted
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
      }
    })
});

export const persistor = persistStore(store);
```

### Provider with Persist

```typescript
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import { store, persistor } from './store';

const App = () => {
  return (
    <Provider store={store}>
      <PersistGate loading={<SplashScreen />} persistor={persistor}>
        <Navigation />
      </PersistGate>
    </Provider>
  );
};
```

---

## 5. Zustand

### Installation

```bash
npm install zustand
```

### Create Store

```typescript
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthStore {
  user: User | null;
  isAuthenticated: boolean;
  login: (user: User) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  isAuthenticated: false,
  login: (user) => set({ user, isAuthenticated: true }),
  logout: () => set({ user: null, isAuthenticated: false })
}));

// Uso
const ProfileScreen = () => {
  const { user, logout } = useAuthStore();
  
  return (
    <View>
      <Text>Welcome, {user?.name}</Text>
      <Button title="Logout" onPress={logout} />
    </View>
  );
};
```

### Async Actions

```typescript
interface TodosStore {
  todos: Todo[];
  loading: boolean;
  fetchTodos: () => Promise<void>;
  addTodo: (todo: Todo) => void;
}

export const useTodosStore = create<TodosStore>((set, get) => ({
  todos: [],
  loading: false,
  
  fetchTodos: async () => {
    set({ loading: true });
    try {
      const response = await apiClient.get('/todos');
      set({ todos: response.data, loading: false });
    } catch (error) {
      console.error(error);
      set({ loading: false });
    }
  },
  
  addTodo: (todo) => set((state) => ({
    todos: [...state.todos, todo]
  }))
}));
```

### Persist with Zustand

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const useAuthStore = create(
  persist<AuthStore>(
    (set) => ({
      user: null,
      isAuthenticated: false,
      login: (user) => set({ user, isAuthenticated: true }),
      logout: () => set({ user: null, isAuthenticated: false })
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => AsyncStorage)
    }
  )
);
```

### Slices Pattern

```typescript
// authSlice.ts
export const createAuthSlice = (set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null })
});

// todosSlice.ts
export const createTodosSlice = (set) => ({
  todos: [],
  addTodo: (todo) => set((state) => ({ todos: [...state.todos, todo] }))
});

// store.ts
export const useStore = create((set) => ({
  ...createAuthSlice(set),
  ...createTodosSlice(set)
}));
```

---

## 6. Jotai

### Installation

```bash
npm install jotai
```

### Atoms

```typescript
import { atom, useAtom } from 'jotai';

// Primitive atom
const countAtom = atom(0);

// Derived atom
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Write-only atom
const incrementAtom = atom(
  null,
  (get, set) => set(countAtom, get(countAtom) + 1)
);

// Uso
const Counter = () => {
  const [count, setCount] = useAtom(countAtom);
  const [doubleCount] = useAtom(doubleCountAtom);
  
  return (
    <View>
      <Text>Count: {count}</Text>
      <Text>Double: {doubleCount}</Text>
      <Button title="Increment" onPress={() => setCount(c => c + 1)} />
    </View>
  );
};
```

### Async Atoms

```typescript
import { atom } from 'jotai';

const userIdAtom = atom('user123');

const userAtom = atom(async (get) => {
  const userId = get(userIdAtom);
  const response = await fetch(`https://api.example.com/users/${userId}`);
  return response.json();
});

// Uso
import { Suspense } from 'react';

const UserProfile = () => {
  const [user] = useAtom(userAtom);
  
  return (
    <View>
      <Text>{user.name}</Text>
    </View>
  );
};

const App = () => (
  <Suspense fallback={<ActivityIndicator />}>
    <UserProfile />
  </Suspense>
);
```

---

## 7. Recoil

### Installation

```bash
npm install recoil
```

### Atoms

```typescript
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil';

const countState = atom({
  key: 'countState',
  default: 0
});

const doubleCountState = selector({
  key: 'doubleCountState',
  get: ({ get }) => {
    const count = get(countState);
    return count * 2;
  }
});

// Uso
import { RecoilRoot } from 'recoil';

const App = () => (
  <RecoilRoot>
    <Counter />
  </RecoilRoot>
);

const Counter = () => {
  const [count, setCount] = useRecoilState(countState);
  const doubleCount = useRecoilValue(doubleCountState);
  
  return (
    <View>
      <Text>Count: {count}</Text>
      <Text>Double: {doubleCount}</Text>
      <Button title="+" onPress={() => setCount(c => c + 1)} />
    </View>
  );
};
```

---

## 8. Optimization

### React.memo

```typescript
const TodoItem = React.memo(({ todo, onToggle }) => {
  console.log('Rendering TodoItem:', todo.id);
  return (
    <TouchableOpacity onPress={onToggle}>
      <Text>{todo.title}</Text>
    </TouchableOpacity>
  );
});
```

### Selectors

```typescript
// Redux
const selectActiveTodos = (state: RootState) =>
  state.todos.items.filter(t => !t.completed);

const TodosList = () => {
  const activeTodos = useAppSelector(selectActiveTodos);
  // Only re-renders when active todos change
};

// Zustand
const useActiveTodos = () => useTodosStore(
  state => state.todos.filter(t => !t.completed)
);
```

### Memoization

```typescript
import { useMemo } from 'react';

const TodosList = () => {
  const todos = useAppSelector(state => state.todos.items);
  
  const activeTodos = useMemo(
    () => todos.filter(t => !t.completed),
    [todos]
  );
  
  return <FlatList data={activeTodos} />;
};
```

---

## 9. DevTools

### Redux DevTools

```typescript
import { configureStore } from '@reduxjs/toolkit';

export const store = configureStore({
  reducer: rootReducer,
  devTools: __DEV__ // Enable only in development
});
```

### Zustand DevTools

```typescript
import { devtools } from 'zustand/middleware';

export const useStore = create(
  devtools((set) => ({
    // store implementation
  }))
);
```

---

## 10. Choosing

### Comparison Table

| Feature | Context API | Redux Toolkit | Zustand | Jotai |
|---------|-------------|---------------|---------|-------|
| Learning Curve | Low | Medium | Low | Medium |
| Bundle Size | 0 KB | ~12 KB | ~1 KB | ~3 KB |
| Performance | Medium | High | High | High |
| DevTools | No | Yes | Yes | Yes |
| TypeScript | Good | Excellent | Good | Good |
| Async | Manual | Built-in | Manual | Built-in |

### When to Use

**Context API:**
- Small apps
- Simple state
- No complex updates

**Redux Toolkit:**
- Large apps
- Complex state logic
- Time-travel debugging needed
- Team familiar with Redux

**Zustand:**
- Medium apps
- Simple API preferred
- Want minimal boilerplate
- Need good performance

**Jotai/Recoil:**
- Atomic state management
- Derived state important
- React Suspense integration

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Context API Todo App

App con Context:
- Add/Remove/Toggle todos
- Filter (all/active/completed)
- Clear completed
- Persist in AsyncStorage

### Esercizio 2: 🟡 Redux Toolkit Shopping Cart

E-commerce cart con Redux:
- Add/remove items
- Update quantities
- Calculate totals
- Async product fetching
- Redux Persist

### Esercizio 3: 🟡 Zustand Multi-Store

App con multiple stores:
- Auth store
- Theme store
- Settings store
- Todos store
- Persist auth e theme

### Esercizio 4: 🔴 State Management Comparison

Implementa stessa app con:
- Context API
- Redux Toolkit
- Zustand
- Confronta performance
- Documenta differenze

### Esercizio 5: 🔴 Advanced Redux

App completa Redux con:
- Multiple slices
- Async thunks
- Normalized state
- Entity adapters
- Middleware custom
- Time-travel debugging

---

## 📝 Quiz

1. **Context API re-renders:**
   - [ ] Solo componenti che usano il valore
   - [x] Tutti i componenti nel Provider
   - [ ] Nessun componente
   - [ ] Solo root component

2. **Redux Toolkit usa:**
   - [ ] Redux classico
   - [x] Immer per immutability
   - [ ] MobX
   - [ ] Context API

3. **Zustand è:**
   - [ ] Più grande di Redux
   - [x] Più piccolo di Redux
   - [ ] Uguale a Redux
   - [ ] Non si può confrontare

4. **Per async in Zustand:**
   - [x] Async/await nelle actions
   - [ ] createAsyncThunk
   - [ ] Middleware obbligatorio
   - [ ] Non supportato

5. **Redux Persist serve per:**
   - [ ] Cache API calls
   - [x] Salvare state in AsyncStorage
   - [ ] Performance
   - [ ] Debugging

**Risposte**: 1-b, 2-b, 3-b, 4-a, 5-b

---

## 🔗 Risorse

- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [Jotai Documentation](https://jotai.org/)
- [Recoil Documentation](https://recoiljs.org/)

---

## ✅ Checklist

- [ ] Context API implementato
- [ ] Redux Toolkit configurato
- [ ] Async actions gestite
- [ ] Persist implementato
- [ ] Zustand provato
- [ ] Performance ottimizzate
- [ ] DevTools configurati
- [ ] Soluzione giusta scelta
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🎯 **Context API** per state semplice
2. 🔴 **Redux Toolkit** per app grandi
3. 🐻 **Zustand** per semplicità
4. ⚛️ **Jotai/Recoil** per atomic state
5. 💾 **Persist** con AsyncStorage
6. ⚡ **Selectors** per performance
7. 🛠️ **DevTools** per debugging
8. 🎯 **Scegli** in base a esigenze

---

**Fantastico! 🎉** Ora padroneggi lo state management. Prossimo: Persistenza Dati Locale!

[← Precedente: Networking e API](11_Networking_API.md) | [Torna all'Indice](../README.md) | [Prossimo: Persistenza Dati →](13_Persistenza_Dati.md)
