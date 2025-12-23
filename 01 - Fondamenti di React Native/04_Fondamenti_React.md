# Capitolo 4: Fondamenti di React

[← Precedente: JavaScript e TypeScript](03_JavaScript_TypeScript.md) | [Torna all'Indice](../README.md) | [Prossimo: Componenti Core →](../02%20-%20Sviluppo%20dell'Interfaccia%20Utente/05_Componenti_Core.md)

---

## Descrizione

React è la libreria JavaScript alla base di React Native. In questo capitolo esploreremo i concetti fondamentali di React che sono essenziali per sviluppare applicazioni React Native: componenti funzionali, JSX, props, state, effects, e hooks personalizzati.

Comprendere React è cruciale perché React Native utilizza la stessa filosofia e gli stessi pattern di sviluppo. Una volta padroneggiati questi concetti, sarai in grado di costruire interfacce utente complesse e interattive con facilità.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Comprendere cos'è React e come funziona
- [ ] Creare componenti funzionali con JSX
- [ ] Utilizzare props per passare dati tra componenti
- [ ] Gestire lo state locale con useState
- [ ] Eseguire side effects con useEffect
- [ ] Condividere stato globale con Context API
- [ ] Creare e utilizzare custom hooks
- [ ] Applicare best practices e pattern comuni

## 📚 Argomenti Teorici

1. [Cos'è React](#1-cosè-react)
2. [Componenti Funzionali](#2-componenti-funzionali)
3. [JSX - JavaScript XML](#3-jsx-javascript-xml)
4. [Props - Proprietà dei Componenti](#4-props-proprietà)
5. [State con useState](#5-state-con-usestate)
6. [Effetti Collaterali con useEffect](#6-effetti-con-useeffect)
7. [Context API](#7-context-api)
8. [Custom Hooks](#8-custom-hooks)
9. [Rules of Hooks](#9-rules-of-hooks)
10. [Best Practices](#10-best-practices)

---

## 1. Cos'è React

### Definizione

**React** è una libreria JavaScript per costruire interfacce utente, creata da Facebook nel 2013.

**Filosofia principale:**
```
UI = f(state)
```

L'interfaccia utente è una funzione dello stato. Quando lo stato cambia, React aggiorna automaticamente l'UI.

### Concetti Chiave

**1. Componenti**
```javascript
// Un componente è una funzione che ritorna UI
const Welcome = () => {
  return <Text>Hello World!</Text>;
};
```

**2. Declarative (Dichiarativo)**
```javascript
// Dici COSA vuoi, non COME ottenerlo
{isLoggedIn ? <Dashboard /> : <Login />}
```

**3. Component-Based (Basato su Componenti)**
```javascript
<App>
  <Header />
  <MainContent>
    <Sidebar />
    <Content />
  </MainContent>
  <Footer />
</App>
```

**4. Learn Once, Write Anywhere**
- React per web
- React Native per mobile
- React Native Web per web da codice mobile

### Virtual DOM (in React web)

React usa un Virtual DOM per efficienza, ma **React Native non ha un DOM** - renderizza componenti nativi direttamente!

```
React (web):    JSX → Virtual DOM → Real DOM
React Native:   JSX → Bridge → Native Views
```

---

## 2. Componenti Funzionali

### Anatomia di un Componente

```typescript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

// 1. Definizione del componente (funzione)
const Greeting = () => {
  // 2. Logica (opzionale)
  const name = 'John';
  
  // 3. Return JSX
  return (
    <View style={styles.container}>
      <Text>Hello, {name}!</Text>
    </View>
  );
};

// 4. Stili (opzionale)
const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
});

// 5. Export
export default Greeting;
```

### Componenti con Props

```typescript
interface GreetingProps {
  name: string;
  age?: number;
}

const Greeting: React.FC<GreetingProps> = ({ name, age }) => {
  return (
    <View>
      <Text>Hello, {name}!</Text>
      {age && <Text>You are {age} years old</Text>}
    </View>
  );
};

// Uso
<Greeting name="John" age={30} />
```

### Componenti Nidificati

```typescript
const UserInfo = () => (
  <View>
    <Text>User Information</Text>
  </View>
);

const UserAvatar = () => (
  <Image source={{ uri: 'https://...' }} />
);

const UserCard = () => {
  return (
    <View>
      <UserAvatar />
      <UserInfo />
    </View>
  );
};
```

---

## 3. JSX - JavaScript XML

### Cos'è JSX

JSX è una sintassi che sembra HTML ma è JavaScript.

```jsx
// JSX
const element = <Text>Hello World</Text>;

// Si trasforma in:
const element = React.createElement(Text, null, 'Hello World');
```

### Regole JSX

**1. Un Solo Elemento Root**
```jsx
// ❌ Errore - multiple root
return (
  <Text>Hello</Text>
  <Text>World</Text>
);

// ✅ OK - un root
return (
  <View>
    <Text>Hello</Text>
    <Text>World</Text>
  </View>
);

// ✅ OK - Fragment (non aggiunge nodo)
return (
  <>
    <Text>Hello</Text>
    <Text>World</Text>
  </>
);
```

**2. Chiudere Tutti i Tag**
```jsx
// ❌ Errore in JSX
<Image src="...">

// ✅ Self-closing tag
<Image source={{ uri: '...' }} />
```

**3. camelCase per Attributi**
```jsx
// HTML: class, onclick
// JSX: className (web), style, onPress

<View style={styles.container}>
  <TouchableOpacity onPress={handlePress}>
    <Text>Click me</Text>
  </TouchableOpacity>
</View>
```

### Espressioni JavaScript in JSX

```jsx
const App = () => {
  const name = 'John';
  const age = 30;
  const isAdult = age >= 18;
  
  return (
    <View>
      {/* Variabili */}
      <Text>{name}</Text>
      
      {/* Espressioni */}
      <Text>{2 + 2}</Text>
      <Text>{name.toUpperCase()}</Text>
      
      {/* Conditional rendering */}
      {isAdult && <Text>You are an adult</Text>}
      {isAdult ? <Text>Adult</Text> : <Text>Minor</Text>}
      
      {/* Chiamate a funzioni */}
      <Text>{getName()}</Text>
      
      {/* Array mapping */}
      {items.map(item => (
        <Text key={item.id}>{item.name}</Text>
      ))}
    </View>
  );
};
```

### Conditional Rendering

```jsx
const LoginStatus = ({ isLoggedIn, user }) => {
  // 1. If-else con variabile
  let content;
  if (isLoggedIn) {
    content = <Text>Welcome, {user.name}!</Text>;
  } else {
    content = <Text>Please log in</Text>;
  }
  return <View>{content}</View>;
  
  // 2. Ternary operator
  return (
    <View>
      {isLoggedIn ? (
        <Text>Welcome, {user.name}!</Text>
      ) : (
        <Text>Please log in</Text>
      )}
    </View>
  );
  
  // 3. && operator
  return (
    <View>
      {isLoggedIn && <Text>Welcome, {user.name}!</Text>}
    </View>
  );
  
  // 4. Early return
  if (!isLoggedIn) {
    return <Text>Please log in</Text>;
  }
  return <Text>Welcome, {user.name}!</Text>;
};
```

### Liste e Keys

```jsx
const TodoList = ({ todos }) => {
  return (
    <View>
      {todos.map(todo => (
        <View key={todo.id}>  {/* Key è importante! */}
          <Text>{todo.title}</Text>
        </View>
      ))}
    </View>
  );
};

// ⚠️ Evita index come key se l'ordine può cambiare
// ❌ Non ideale
{todos.map((todo, index) => <View key={index}>...</View>)}

// ✅ Usa ID univoci
{todos.map(todo => <View key={todo.id}>...</View>)}
```

---

## 4. Props - Proprietà

### Passare Props

```typescript
interface ButtonProps {
  title: string;
  onPress: () => void;
  color?: string;
  disabled?: boolean;
}

const CustomButton: React.FC<ButtonProps> = (props) => {
  return (
    <TouchableOpacity
      onPress={props.onPress}
      disabled={props.disabled}
      style={{ backgroundColor: props.color || '#007AFF' }}
    >
      <Text>{props.title}</Text>
    </TouchableOpacity>
  );
};

// Uso
<CustomButton
  title="Click Me"
  onPress={() => console.log('Pressed')}
  color="#FF3B30"
/>
```

### Props Destructuring (Consigliato)

```typescript
// ✅ Più pulito
const CustomButton: React.FC<ButtonProps> = ({
  title,
  onPress,
  color = '#007AFF',  // Default value
  disabled = false
}) => {
  return (
    <TouchableOpacity
      onPress={onPress}
      disabled={disabled}
      style={{ backgroundColor: color }}
    >
      <Text>{title}</Text>
    </TouchableOpacity>
  );
};
```

### Children Prop

```typescript
interface CardProps {
  title: string;
  children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ title, children }) => {
  return (
    <View style={styles.card}>
      <Text style={styles.title}>{title}</Text>
      <View style={styles.content}>
        {children}
      </View>
    </View>
  );
};

// Uso
<Card title="User Info">
  <Text>Name: John</Text>
  <Text>Age: 30</Text>
  <Button title="Edit" onPress={() => {}} />
</Card>
```

### Props Drilling Problem

```jsx
// ❌ Props drilling - passare props attraverso molti livelli
<App>
  <Dashboard user={user}>
    <Profile user={user}>
      <UserInfo user={user} />
    </Profile>
  </Dashboard>
</App>

// ✅ Soluzione: Context API (vedi sezione 7)
```

---

## 5. State con useState

### Cos'è lo State?

Lo **state** sono dati che possono cambiare nel tempo e causano re-render del componente.

```typescript
import { useState } from 'react';

const Counter = () => {
  // Dichiarazione state
  const [count, setCount] = useState(0);
  //     ^valore  ^funzione setter  ^valore iniziale
  
  return (
    <View>
      <Text>Count: {count}</Text>
      <Button title="+" onPress={() => setCount(count + 1)} />
    </View>
  );
};
```

### Multiple State Variables

```typescript
const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const handleLogin = async () => {
    setLoading(true);
    setError(null);
    
    try {
      await login(email, password);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <View>
      <TextInput value={email} onChangeText={setEmail} />
      <TextInput value={password} onChangeText={setPassword} secureTextEntry />
      {error && <Text style={{ color: 'red' }}>{error}</Text>}
      <Button title="Login" onPress={handleLogin} disabled={loading} />
    </View>
  );
};
```

### State con Oggetti

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

const Profile = () => {
  const [user, setUser] = useState<User>({
    name: 'John',
    age: 30,
    email: 'john@example.com'
  });
  
  // ❌ NON mutare direttamente
  // user.name = 'Jane';  // Non funziona!
  
  // ✅ Crea nuovo oggetto con spread
  const updateName = (newName: string) => {
    setUser({
      ...user,
      name: newName
    });
  };
  
  // ✅ Aggiorna multipli campi
  const updateUser = () => {
    setUser({
      ...user,
      name: 'Jane',
      age: 31
    });
  };
  
  return <View>...</View>;
};
```

### State con Array

```typescript
const TodoList = () => {
  const [todos, setTodos] = useState<string[]>([]);
  
  // Aggiungere elemento
  const addTodo = (text: string) => {
    setTodos([...todos, text]);
    // o: setTodos(prev => [...prev, text]);
  };
  
  // Rimuovere elemento
  const removeTodo = (index: number) => {
    setTodos(todos.filter((_, i) => i !== index));
  };
  
  // Aggiornare elemento
  const updateTodo = (index: number, newText: string) => {
    setTodos(todos.map((todo, i) => 
      i === index ? newText : todo
    ));
  };
  
  // Pulire array
  const clearTodos = () => {
    setTodos([]);
  };
  
  return <View>...</View>;
};
```

### Functional Updates

```typescript
const Counter = () => {
  const [count, setCount] = useState(0);
  
  // ❌ Potenziale problema con operazioni rapide multiple
  const incrementThrice = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // count aumenta solo di 1! 😱
  };
  
  // ✅ Usa functional update
  const incrementThriceCorrect = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    // count aumenta di 3 ✅
  };
  
  return <View>...</View>;
};
```

---

## 6. Effetti con useEffect

### Cos'è useEffect?

`useEffect` esegue **side effects** (operazioni che interagiscono con l'esterno del componente).

```typescript
import { useEffect } from 'react';

useEffect(() => {
  // Codice da eseguire (effect)
  
  return () => {
    // Cleanup (opzionale)
  };
}, [dependencies]);  // Array di dipendenze
```

### Effect senza Dipendenze

```typescript
// ⚠️ Esegue ad OGNI render!
useEffect(() => {
  console.log('Component rendered');
});
```

### Effect con Array Vuoto

```typescript
// ✅ Esegue UNA VOLTA al mount
useEffect(() => {
  console.log('Component mounted');
  fetchInitialData();
}, []);  // Array vuoto = solo al mount
```

### Effect con Dipendenze

```typescript
const UserProfile = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState(null);
  
  // ✅ Esegue quando userId cambia
  useEffect(() => {
    const fetchUser = async () => {
      const data = await fetch(`/api/users/${userId}`).then(r => r.json());
      setUser(data);
    };
    
    fetchUser();
  }, [userId]);  // Ri-esegue quando userId cambia
  
  return <View>...</View>;
};
```

### Cleanup Function

```typescript
const Timer = () => {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    // Setup
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Cleanup - viene chiamato quando:
    // 1. Componente viene smontato
    // 2. Prima di ri-eseguire l'effect
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  return <Text>{seconds} seconds</Text>;
};
```

### Multiple Effects

```typescript
const UserDashboard = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  // Effect 1: Fetch user
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  // Effect 2: Fetch posts
  useEffect(() => {
    fetchUserPosts(userId).then(setPosts);
  }, [userId]);
  
  // Effect 3: Document title (web)
  useEffect(() => {
    document.title = user ? `${user.name} - Dashboard` : 'Dashboard';
  }, [user]);
  
  return <View>...</View>;
};
```

### Esempi Comuni

```typescript
// 1. Fetch data
useEffect(() => {
  const fetchData = async () => {
    const response = await fetch('/api/data');
    const data = await response.json();
    setState(data);
  };
  fetchData();
}, []);

// 2. Subscribe to events
useEffect(() => {
  const subscription = someAPI.subscribe(data => {
    setState(data);
  });
  
  return () => subscription.unsubscribe();
}, []);

// 3. Keyboard listener (React Native)
useEffect(() => {
  const keyboardDidShow = Keyboard.addListener('keyboardDidShow', () => {
    setKeyboardVisible(true);
  });
  
  const keyboardDidHide = Keyboard.addListener('keyboardDidHide', () => {
    setKeyboardVisible(false);
  });
  
  return () => {
    keyboardDidShow.remove();
    keyboardDidHide.remove();
  };
}, []);

// 4. LocalStorage sync (web)
useEffect(() => {
  localStorage.setItem('user', JSON.stringify(user));
}, [user]);
```

---

## 7. Context API

### Problema: Props Drilling

```jsx
// Props drilling - difficile da mantenere
<App>
  <Dashboard theme={theme} user={user}>
    <Sidebar theme={theme} user={user}>
      <UserInfo theme={theme} user={user} />
    </Sidebar>
  </Dashboard>
</App>
```

### Soluzione: Context

Context permette di condividere dati senza passare props manualmente attraverso ogni livello.

### Creare un Context

```typescript
// contexts/ThemeContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

// 1. Crea il context
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// 2. Provider component
export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [theme, setTheme] = useState<Theme>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// 3. Custom hook per usare il context
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};
```

### Usare il Context

```typescript
// App.tsx - Wrap con Provider
import { ThemeProvider } from './contexts/ThemeContext';

const App = () => {
  return (
    <ThemeProvider>
      <Dashboard />
    </ThemeProvider>
  );
};

// Qualsiasi componente può accedere al theme
const Header = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <View style={{ backgroundColor: theme === 'light' ? '#fff' : '#000' }}>
      <Button title="Toggle Theme" onPress={toggleTheme} />
    </View>
  );
};

const Sidebar = () => {
  const { theme } = useTheme();
  
  return (
    <View style={{ backgroundColor: theme === 'light' ? '#f0f0f0' : '#333' }}>
      <Text>Sidebar</Text>
    </View>
  );
};
```

### Context Complesso (Auth Example)

```typescript
// contexts/AuthContext.tsx
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextType {
  user: User | null;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  signup: (email: string, password: string) => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check if user is logged in on mount
    checkAuth();
  }, []);
  
  const checkAuth = async () => {
    try {
      const token = await AsyncStorage.getItem('token');
      if (token) {
        const userData = await fetchUserData(token);
        setUser(userData);
      }
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };
  
  const login = async (email: string, password: string) => {
    const { user, token } = await loginAPI(email, password);
    await AsyncStorage.setItem('token', token);
    setUser(user);
  };
  
  const logout = async () => {
    await AsyncStorage.removeItem('token');
    setUser(null);
  };
  
  const signup = async (email: string, password: string) => {
    const { user, token } = await signupAPI(email, password);
    await AsyncStorage.setItem('token', token);
    setUser(user);
  };
  
  return (
    <AuthContext.Provider value={{ user, loading, login, logout, signup }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

---

## 8. Custom Hooks

### Cos'è un Custom Hook?

Un **custom hook** è una funzione che usa altri hooks e permette di riutilizzare logica.

**Regole:**
- Nome deve iniziare con `use`
- Può usare altri hooks
- Può ritornare qualsiasi valore

### Esempio: useFetch

```typescript
// hooks/useFetch.ts
import { useState, useEffect } from 'react';

interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error('Failed to fetch');
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  };
  
  useEffect(() => {
    fetchData();
  }, [url]);
  
  return { data, loading, error, refetch: fetchData };
}

// Uso
const UserList = () => {
  const { data: users, loading, error, refetch } = useFetch<User[]>('/api/users');
  
  if (loading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <View>
      {users?.map(user => <Text key={user.id}>{user.name}</Text>)}
      <Button title="Refresh" onPress={refetch} />
    </View>
  );
};
```

### Esempio: useDebounce

```typescript
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Uso - Search input
const SearchScreen = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API call solo dopo 500ms dall'ultima digitazione
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <TextInput
      value={searchTerm}
      onChangeText={setSearchTerm}
      placeholder="Search..."
    />
  );
};
```

### Esempio: useToggle

```typescript
// hooks/useToggle.ts
import { useState } from 'react';

function useToggle(initialValue: boolean = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue);
  
  const toggle = () => setValue(prev => !prev);
  
  return [value, toggle];
}

// Uso
const Settings = () => {
  const [darkMode, toggleDarkMode] = useToggle(false);
  const [notifications, toggleNotifications] = useToggle(true);
  
  return (
    <View>
      <Switch value={darkMode} onValueChange={toggleDarkMode} />
      <Switch value={notifications} onValueChange={toggleNotifications} />
    </View>
  );
};
```

### Esempio: useLocalStorage (web) / useAsyncStorage (RN)

```typescript
// hooks/useAsyncStorage.ts (React Native)
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

function useAsyncStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(initialValue);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const loadStoredValue = async () => {
      try {
        const item = await AsyncStorage.getItem(key);
        if (item) {
          setStoredValue(JSON.parse(item));
        }
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };
    
    loadStoredValue();
  }, [key]);
  
  const setValue = async (value: T) => {
    try {
      setStoredValue(value);
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue, loading] as const;
}

// Uso
const App = () => {
  const [theme, setTheme, loading] = useAsyncStorage<'light' | 'dark'>('theme', 'light');
  
  if (loading) return <Text>Loading...</Text>;
  
  return (
    <View>
      <Text>Current theme: {theme}</Text>
      <Button title="Toggle" onPress={() => setTheme(theme === 'light' ? 'dark' : 'light')} />
    </View>
  );
};
```

---

## 9. Rules of Hooks

### Regole Fondamentali

**1. Chiama hooks solo al top level**
```typescript
// ❌ NON in condizioni
if (condition) {
  const [state, setState] = useState(0);  // Errore!
}

// ❌ NON in loop
for (let i = 0; i < 10; i++) {
  useEffect(() => {});  // Errore!
}

// ✅ Al top level
const MyComponent = () => {
  const [state, setState] = useState(0);  // OK
  useEffect(() => {}, []);                 // OK
  
  if (condition) {
    // Usa state qui dentro
  }
};
```

**2. Chiama hooks solo in React Functions**
```typescript
// ✅ In componenti React
const MyComponent = () => {
  const [state, setState] = useState(0);
  return <View />;
};

// ✅ In custom hooks
function useCustomHook() {
  const [state, setState] = useState(0);
  return state;
}

// ❌ In funzioni normali
function regularFunction() {
  const [state, setState] = useState(0);  // Errore!
}
```

### ESLint Plugin

```bash
# Installa plugin per verificare le regole
npm install eslint-plugin-react-hooks --save-dev
```

```json
// .eslintrc.json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

---

## 10. Best Practices

### 1. Componentizzazione

```typescript
// ❌ Componente monolitico
const UserProfile = () => {
  return (
    <View>
      {/* 300 linee di codice */}
    </View>
  );
};

// ✅ Componenti piccoli e riutilizzabili
const UserAvatar = ({ uri }) => <Image source={{ uri }} />;
const UserInfo = ({ name, email }) => (
  <View>
    <Text>{name}</Text>
    <Text>{email}</Text>
  </View>
);
const UserProfile = ({ user }) => (
  <View>
    <UserAvatar uri={user.avatar} />
    <UserInfo name={user.name} email={user.email} />
  </View>
);
```

### 2. Props Destructuring

```typescript
// ❌
const Button = (props) => {
  return <Text>{props.title}</Text>;
};

// ✅
const Button = ({ title, onPress, disabled = false }) => {
  return <TouchableOpacity onPress={onPress} disabled={disabled}>
    <Text>{title}</Text>
  </TouchableOpacity>;
};
```

### 3. Early Returns

```typescript
const UserProfile = ({ user }) => {
  // ✅ Early return per casi speciali
  if (!user) return <Text>Loading...</Text>;
  if (user.error) return <Text>Error!</Text>;
  
  // Caso normale
  return (
    <View>
      <Text>{user.name}</Text>
    </View>
  );
};
```

### 4. Memoizzazione (Performance)

```typescript
import { useMemo, useCallback } from 'react';

const ExpensiveComponent = ({ items, onItemClick }) => {
  // useMemo - memorizza il risultato di un calcolo
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);  // Ricalcola solo se items cambia
  
  // useCallback - memorizza una funzione
  const handleClick = useCallback((id: string) => {
    onItemClick(id);
  }, [onItemClick]);
  
  return (
    <FlatList
      data={sortedItems}
      renderItem={({ item }) => (
        <TouchableOpacity onPress={() => handleClick(item.id)}>
          <Text>{item.name}</Text>
        </TouchableOpacity>
      )}
    />
  );
};
```

### 5. Separazione Logica e UI

```typescript
// ✅ Custom hook per logica
const useUserData = (userId: string) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading };
};

// Componente solo per UI
const UserProfile = ({ userId }) => {
  const { user, loading } = useUserData(userId);
  
  if (loading) return <LoadingSpinner />;
  return <UserCard user={user} />;
};
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Counter App

**Obiettivo**: Praticare useState

**Task**: Crea un'app con:
- Display del contatore
- Bottone per incrementare
- Bottone per decrementare
- Bottone per resettare

```typescript
const Counter = () => {
  // Il tuo codice qui
};
```

### Esercizio 2: 🟡 Todo List

**Obiettivo**: State con array e map

**Task**: Crea una todo list con:
- Input per aggiungere todo
- Lista di todo
- Checkbox per completare
- Bottone per eliminare

### Esercizio 3: 🟡 Fetch Data

**Obiettivo**: useEffect e async

**Task**: Fetcha e mostra users da:
`https://jsonplaceholder.typicode.com/users`

- Mostra loading state
- Gestisci errori
- Bottone per refresh

### Esercizio 4: 🔴 Custom useFetch Hook

**Obiettivo**: Creare custom hook riutilizzabile

**Task**: Implementa `useFetch<T>` hook con:
- Tipizzazione TypeScript
- Loading, error, data states
- Refetch function
- Cleanup se componente unmount durante fetch

### Esercizio 5: 🔴 Theme Context

**Obiettivo**: Context API completo

**Task**: Implementa sistema di temi:
- ThemeContext con Provider
- useTheme custom hook
- Toggle tra light/dark
- Persistenza con AsyncStorage
- Applica theme a tutta l'app

---

## 📝 Quiz di Autovalutazione

1. **Cosa ritorna un componente React?**
   - [ ] String
   - [x] JSX
   - [ ] Object
   - [ ] Array

2. **Differenza tra props e state?**
   - [x] Props sono immutabili, state è mutabile
   - [ ] Props sono per dati, state per UI
   - [ ] Non c'è differenza
   - [ ] State è deprecato

3. **Quando esegue useEffect con array vuoto `[]`?**
   - [ ] Ad ogni render
   - [x] Solo al mount
   - [ ] Solo all'unmount
   - [ ] Mai

4. **A cosa serve la key in una lista?**
   - [ ] Per styling
   - [x] Per identificare elementi univocamente
   - [ ] Per performance
   - [ ] È opzionale

5. **Quando usare Context invece di props?**
   - [ ] Sempre
   - [ ] Mai
   - [x] Per dati globali usati da molti componenti
   - [ ] Solo con TypeScript

**Risposte**: 1-b, 2-a, 3-b, 4-b, 5-c

---

## 🔗 Risorse Aggiuntive

- [React Documentation](https://react.dev/)
- [React Native Documentation](https://reactnative.dev/)
- [React Hooks](https://react.dev/reference/react)
- [Thinking in React](https://react.dev/learn/thinking-in-react)

---

## ✅ Checklist di Completamento

- [ ] Comprensione di componenti funzionali
- [ ] Padronanza JSX
- [ ] Props utilizzate correttamente
- [ ] useState per gestire state locale
- [ ] useEffect per side effects
- [ ] Context API per state globale
- [ ] Custom hooks creati
- [ ] Rules of Hooks rispettate
- [ ] Almeno 3 esercizi completati

---

## 📌 Punti Chiave

1. 🧩 **Componenti** sono building blocks
2. 📝 **JSX** sembra HTML ma è JavaScript
3. ⬇️ **Props** passano dati ai componenti
4. 🔄 **State** causa re-render quando cambia
5. ⚡ **useEffect** per side effects
6. 🌐 **Context** per state globale
7. 🎣 **Custom hooks** riutilizzano logica
8. 📏 **Rules of Hooks** vanno sempre rispettate

---

**Fantastico! 🎉** Ora hai una solida base di React. Sei pronto per esplorare i componenti specifici di React Native!

[← Precedente: JavaScript e TypeScript](03_JavaScript_TypeScript.md) | [Torna all'Indice](../README.md) | [Prossimo: Componenti Core →](../02%20-%20Sviluppo%20dell'Interfaccia%20Utente/05_Componenti_Core.md)
