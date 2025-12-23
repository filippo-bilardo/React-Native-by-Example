# Capitolo 3: JavaScript e TypeScript per React Native

[← Precedente: Ambiente di Sviluppo](02_Ambiente_Sviluppo.md) | [Torna all'Indice](../README.md) | [Prossimo: Fondamenti di React →](04_Fondamenti_React.md)

---

## Descrizione

React Native è costruito su JavaScript (o TypeScript), quindi una solida comprensione di questi linguaggi è fondamentale. In questo capitolo esploreremo le caratteristiche moderne di JavaScript (ES6+) più utilizzate in React Native, e introdurremo TypeScript per scrivere codice più sicuro e manutenibile.

Non serve essere esperti JavaScript per iniziare, ma è importante conoscere i concetti chiave: arrow functions, destructuring, promises, async/await, e moduli. Per chi vuole tipizzazione statica e migliore autocomplete, TypeScript è la scelta ideale.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Utilizzare le moderne feature JavaScript ES6+
- [ ] Comprendere e usare arrow functions
- [ ] Applicare destructuring e spread operator
- [ ] Gestire operazioni asincrone con Promises e async/await
- [ ] Lavorare con moduli ES6 (import/export)
- [ ] Configurare TypeScript in un progetto React Native
- [ ] Definire tipi, interfacce e type aliases
- [ ] Tipizzare componenti React Native
- [ ] Utilizzare generics per codice riutilizzabile

## 📚 Argomenti Teorici

1. [JavaScript Moderno (ES6+)](#1-javascript-moderno-es6)
2. [Variabili: let, const, var](#2-variabili-let-const-var)
3. [Arrow Functions](#3-arrow-functions)
4. [Destructuring](#4-destructuring)
5. [Spread e Rest Operator](#5-spread-e-rest-operator)
6. [Template Literals](#6-template-literals)
7. [Promises e Async/Await](#7-promises-e-asyncawait)
8. [Moduli ES6](#8-moduli-es6)
9. [TypeScript: Introduzione](#9-typescript-introduzione)
10. [TypeScript in React Native](#10-typescript-in-react-native)

---

## 1. JavaScript Moderno (ES6+)

### Cos'è ES6+?

**ECMAScript** (ES) è lo standard su cui si basa JavaScript.

**Timeline:**
- **ES5** (2009): JavaScript "classico"
- **ES6/ES2015** (2015): 🚀 Rivoluzione! Arrow functions, classes, modules
- **ES2016-ES2025**: Aggiunte incrementali annuali

React Native supporta **tutte le feature moderne** grazie a Babel (transpiler).

### Feature ES6+ più usate in React Native

```javascript
// ✅ Usate costantemente
const, let          // Variabili
Arrow functions     // () => {}
Destructuring       // const { name } = user
Spread operator     // ...array
Template literals   // `Hello ${name}`
Modules             // import/export
Promises            // .then() .catch()
Async/await         // async () => await

// ✅ Usate spesso
Classes             // class Component
Default parameters  // function(x = 10)
Object shorthand    // { name } invece di { name: name }

// 🟡 Usate occasionalmente
Optional chaining   // user?.address?.city
Nullish coalescing  // value ?? defaultValue
```

---

## 2. Variabili: let, const, var

### var (❌ Evita)

```javascript
// var ha problemi di scope
var name = 'John';
if (true) {
  var name = 'Jane';  // Sovrascrive la variabile esterna!
}
console.log(name);  // 'Jane' 😱
```

### let (✅ Per variabili che cambiano)

```javascript
let count = 0;
count = 1;  // OK ✅
count = 2;  // OK ✅

// Block scoped
if (true) {
  let message = 'Hello';
}
console.log(message);  // Error: message is not defined ✅
```

### const (✅ Default, usa sempre quando possibile)

```javascript
const API_URL = 'https://api.example.com';
// API_URL = 'other';  // Error ✅

// Con oggetti e array, il CONTENUTO può cambiare
const user = { name: 'John' };
user.name = 'Jane';  // OK ✅
user.age = 30;       // OK ✅
// user = {};         // Error ✅

const numbers = [1, 2, 3];
numbers.push(4);     // OK ✅
// numbers = [];      // Error ✅
```

**Regola d'oro:**
```javascript
// ✅ SEMPRE const di default
const name = 'John';

// ✅ let solo se deve cambiare
let counter = 0;
counter++;

// ❌ MAI var
// var oldStyle = 'no';
```

---

## 3. Arrow Functions

### Sintassi

```javascript
// Funzione tradizionale
function sum(a, b) {
  return a + b;
}

// Arrow function equivalente
const sum = (a, b) => {
  return a + b;
};

// Arrow function con return implicito
const sum = (a, b) => a + b;

// Un solo parametro (parentesi opzionali)
const double = x => x * 2;

// Nessun parametro
const sayHello = () => console.log('Hello');
```

### Quando Usarle in React Native

**✅ Perfette per:**

```javascript
// Event handlers
<Button onPress={() => console.log('Clicked')} />

// Array methods
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);

// Callbacks
fetchUser()
  .then(user => console.log(user))
  .catch(error => console.error(error));

// Componenti funzionali
const MyComponent = () => {
  return <Text>Hello</Text>;
};
```

### Differenza con Funzioni Tradizionali: `this`

```javascript
// Funzione tradizionale - 'this' dipende da come viene chiamata
function Counter() {
  this.count = 0;
  
  setInterval(function() {
    this.count++;  // 'this' è undefined o window ❌
  }, 1000);
}

// Arrow function - 'this' è lessicale (dal contesto)
function Counter() {
  this.count = 0;
  
  setInterval(() => {
    this.count++;  // 'this' è Counter ✅
  }, 1000);
}
```

**In React Native (componenti funzionali) non serve preoccuparsi di `this`!**

---

## 4. Destructuring

### Object Destructuring

```javascript
// ❌ Modo vecchio
const user = { name: 'John', age: 30, city: 'Rome' };
const name = user.name;
const age = user.age;

// ✅ Destructuring
const { name, age } = user;
console.log(name);  // 'John'
console.log(age);   // 30

// Rinominare variabili
const { name: userName, age: userAge } = user;

// Valori di default
const { country = 'Italy' } = user;

// Destructuring annidato
const user = {
  name: 'John',
  address: {
    city: 'Rome',
    zip: '00100'
  }
};
const { address: { city } } = user;
console.log(city);  // 'Rome'
```

**In React Native:**

```javascript
// Props destructuring (molto comune!)
const UserCard = ({ name, age, avatar }) => {
  return (
    <View>
      <Image source={{ uri: avatar }} />
      <Text>{name}</Text>
      <Text>{age} years old</Text>
    </View>
  );
};

// State destructuring
const [count, setCount] = useState(0);
const [user, setUser] = useState(null);
```

### Array Destructuring

```javascript
const colors = ['red', 'green', 'blue'];

// ✅ Destructuring
const [first, second, third] = colors;
console.log(first);   // 'red'
console.log(second);  // 'green'

// Skip elementi
const [, , third] = colors;
console.log(third);  // 'blue'

// Rest operator
const [first, ...rest] = colors;
console.log(rest);  // ['green', 'blue']
```

**Uso fondamentale in React Native:**

```javascript
// useState Hook
const [count, setCount] = useState(0);
//     ^value  ^setter

// useEffect Hook
const [data, setData] = useState([]);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
```

---

## 5. Spread e Rest Operator

Entrambi usano `...` ma hanno scopi diversi.

### Spread Operator (espandere)

```javascript
// Arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];  // [1, 2, 3, 4, 5, 6]

// Objects
const user = { name: 'John', age: 30 };
const updatedUser = { ...user, age: 31 };  // { name: 'John', age: 31 }

// Copiare array/object
const original = [1, 2, 3];
const copy = [...original];  // Copia shallow

// Function arguments
const numbers = [1, 2, 3];
Math.max(...numbers);  // Invece di Math.max(1, 2, 3)
```

**In React Native:**

```javascript
// Aggiornare state immutabilmente
const [items, setItems] = useState([1, 2, 3]);
setItems([...items, 4]);  // Aggiungi elemento

// Aggiornare oggetti
const [user, setUser] = useState({ name: 'John', age: 30 });
setUser({ ...user, age: 31 });  // Aggiorna solo age

// Passare props
const userProps = { name: 'John', age: 30 };
<UserComponent {...userProps} />
// Equivale a: <UserComponent name="John" age={30} />
```

### Rest Operator (raccogliere)

```javascript
// Raccogliere parametri
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4);  // 10

// Destructuring con rest
const [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first);  // 1
console.log(rest);   // [2, 3, 4, 5]

const { name, ...otherProps } = { name: 'John', age: 30, city: 'Rome' };
console.log(name);        // 'John'
console.log(otherProps);  // { age: 30, city: 'Rome' }
```

**In React Native:**

```javascript
// Estrarre alcune props, passare le altre
const Button = ({ title, onPress, ...otherProps }) => {
  return (
    <TouchableOpacity onPress={onPress} {...otherProps}>
      <Text>{title}</Text>
    </TouchableOpacity>
  );
};
```

---

## 6. Template Literals

```javascript
// ❌ Concatenazione stringe vecchio stile
const name = 'John';
const message = 'Hello, ' + name + '!';

// ✅ Template literals
const message = `Hello, ${name}!`;

// Espressioni dentro ${}
const a = 10, b = 20;
console.log(`${a} + ${b} = ${a + b}`);  // "10 + 20 = 30"

// Multi-line strings
const html = `
  <div>
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;

// Con funzioni
const user = { name: 'John', age: 30 };
const greeting = `Hello ${user.name.toUpperCase()}, you are ${user.age} years old`;
```

**In React Native:**

```javascript
const WelcomeScreen = ({ userName, messageCount }) => {
  return (
    <View>
      <Text>Welcome back, {userName}!</Text>
      <Text>You have {messageCount} new message{messageCount !== 1 ? 's' : ''}</Text>
      <Text>{`Last login: ${new Date().toLocaleDateString()}`}</Text>
    </View>
  );
};

// URLs dinamiche
const userId = 123;
const apiUrl = `https://api.example.com/users/${userId}`;
fetch(apiUrl);

// Stili condizionali
const backgroundColor = isActive ? '#007AFF' : '#CCCCCC';
<View style={{ backgroundColor: `${backgroundColor}` }} />
```

---

## 7. Promises e Async/Await

### Promises

```javascript
// Creare una Promise
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = { id: 1, name: 'John' };
      resolve(data);  // Successo
      // reject(new Error('Failed'));  // Errore
    }, 1000);
  });
};

// Usare una Promise
fetchData()
  .then(data => {
    console.log('Success:', data);
    return data.id;
  })
  .then(id => {
    console.log('ID:', id);
  })
  .catch(error => {
    console.error('Error:', error);
  })
  .finally(() => {
    console.log('Completed');
  });
```

### Async/Await (✅ Preferito)

```javascript
// ✅ Più leggibile con async/await
const loadData = async () => {
  try {
    const data = await fetchData();
    console.log('Success:', data);
    
    const moreData = await fetchMoreData(data.id);
    console.log('More data:', moreData);
    
    return moreData;
  } catch (error) {
    console.error('Error:', error);
  } finally {
    console.log('Completed');
  }
};

// Chiamare funzione async
loadData();

// O con await (dentro un'altra async function)
const result = await loadData();
```

**In React Native:**

```javascript
import { useEffect, useState } from 'react';

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`https://api.example.com/users/${userId}`);
        
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error}</Text>;
  
  return <Text>User: {user.name}</Text>;
};
```

### Promise.all (Chiamate Parallele)

```javascript
// Attendere multiple promises
const loadAllData = async () => {
  try {
    const [users, posts, comments] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json()),
      fetch('/api/comments').then(r => r.json()),
    ]);
    
    console.log(users, posts, comments);
  } catch (error) {
    console.error('One of the requests failed:', error);
  }
};
```

---

## 8. Moduli ES6

### Export

```javascript
// math.js

// Named exports (possono essercene multipli)
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

export const PI = 3.14159;

// Export separato
const multiply = (a, b) => a * b;
export { multiply };

// Default export (uno solo per file)
export default function divide(a, b) {
  return a / b;
}
```

### Import

```javascript
// app.js

// Import named exports
import { add, subtract, PI } from './math';
console.log(add(2, 3));      // 5
console.log(PI);             // 3.14159

// Import con alias
import { add as sum } from './math';
console.log(sum(2, 3));      // 5

// Import tutto
import * as Math from './math';
console.log(Math.add(2, 3)); // 5
console.log(Math.PI);        // 3.14159

// Import default
import divide from './math';
console.log(divide(10, 2));  // 5

// Import default + named
import divide, { add, subtract } from './math';
```

**In React Native:**

```javascript
// components/Button.js
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

export const PrimaryButton = ({ title, onPress }) => (
  <TouchableOpacity style={styles.button} onPress={onPress}>
    <Text style={styles.text}>{title}</Text>
  </TouchableOpacity>
);

export const SecondaryButton = ({ title, onPress }) => (
  <TouchableOpacity style={styles.secondaryButton} onPress={onPress}>
    <Text>{title}</Text>
  </TouchableOpacity>
);

const styles = StyleSheet.create({
  button: { backgroundColor: '#007AFF', padding: 15 },
  text: { color: 'white', fontSize: 16 },
  secondaryButton: { backgroundColor: '#ddd', padding: 15 },
});

// App.js
import React from 'react';
import { View } from 'react-native';
import { PrimaryButton, SecondaryButton } from './components/Button';

export default function App() {
  return (
    <View>
      <PrimaryButton title="Login" onPress={() => console.log('Login')} />
      <SecondaryButton title="Cancel" onPress={() => console.log('Cancel')} />
    </View>
  );
}
```

---

## 9. TypeScript: Introduzione

### Perché TypeScript?

**Problemi con JavaScript:**

```javascript
// JavaScript - errore solo a runtime! 😱
function greet(name) {
  return 'Hello ' + name.toUpperCase();
}

greet(123);  // Runtime error: name.toUpperCase is not a function
```

**Con TypeScript:**

```typescript
// TypeScript - errore PRIMA di eseguire! ✅
function greet(name: string): string {
  return 'Hello ' + name.toUpperCase();
}

greet(123);  // ❌ Compile error: Argument of type 'number' is not assignable to parameter of type 'string'
```

**Vantaggi:**
- ✅ Errori rilevati durante lo sviluppo
- ✅ Autocomplete migliore nell'editor
- ✅ Refactoring più sicuro
- ✅ Documentazione nel codice (i tipi)
- ✅ Migliore per team grandi

### Tipi Base

```typescript
// Primitivi
let name: string = 'John';
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Array
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ['John', 'Jane'];

// Tuple (array con tipi fissi)
let person: [string, number] = ['John', 30];

// Any (evitare quando possibile)
let anything: any = 'hello';
anything = 123;  // OK ma perde type safety

// Unknown (più sicuro di any)
let userInput: unknown;
userInput = 'hello';
userInput = 123;
// let text: string = userInput;  // Error ✅
```

### Interfacce

```typescript
// Definire la forma di un oggetto
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;          // Proprietà opzionale
  readonly role: string; // Readonly
}

const user: User = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  role: 'admin'
};

// user.role = 'user';  // Error: readonly ✅

// Funzione che accetta User
function greetUser(user: User): string {
  return `Hello ${user.name}!`;
}
```

### Type Aliases

```typescript
// Simile a interface, ma più flessibile
type ID = number | string;

type User = {
  id: ID;
  name: string;
  email: string;
};

type AdminUser = User & {
  permissions: string[];
};

// Union types
type Status = 'loading' | 'success' | 'error';

let currentStatus: Status = 'loading';
// currentStatus = 'unknown';  // Error ✅
```

### Funzioni

```typescript
// Tipizzare parametri e return
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Parametri opzionali
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Hello'} ${name}`;
}

// Default parameters
function createUser(name: string, age: number = 18): User {
  return { id: 1, name, email: '', role: 'user' };
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}
```

### Generics

```typescript
// Funzione generica
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);      // num è number
const str = identity<string>('hello'); // str è string

// Array generics
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const firstNumber = getFirstElement([1, 2, 3]);    // number | undefined
const firstName = getFirstElement(['a', 'b', 'c']); // string | undefined

// Interface generica
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: 'John', email: 'john@example.com', role: 'user' },
  status: 200,
  message: 'Success'
};
```

---

## 10. TypeScript in React Native

### Setup TypeScript

```bash
# Nuovo progetto con TypeScript
npx react-native@latest init MyApp --template react-native-template-typescript

# Aggiungere TypeScript a progetto esistente
npm install --save-dev typescript @types/react @types/react-native

# Genera tsconfig.json
npx tsc --init
```

### Tipizzare Componenti

```typescript
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';

// Definire props interface
interface ButtonProps {
  title: string;
  onPress: () => void;
  disabled?: boolean;
  color?: string;
}

// Componente tipizzato
const CustomButton: React.FC<ButtonProps> = ({ 
  title, 
  onPress, 
  disabled = false,
  color = '#007AFF'
}) => {
  return (
    <TouchableOpacity
      style={[styles.button, { backgroundColor: color }]}
      onPress={onPress}
      disabled={disabled}
    >
      <Text style={styles.text}>{title}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    padding: 15,
    borderRadius: 8,
  },
  text: {
    color: 'white',
    fontSize: 16,
    textAlign: 'center',
  },
});

export default CustomButton;
```

### Tipizzare Hooks

```typescript
import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

const UserProfile = () => {
  // TypeScript inferisce il tipo
  const [count, setCount] = useState(0);  // number
  
  // Specificare tipo esplicitamente
  const [user, setUser] = useState<User | null>(null);
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch('https://api.example.com/user/1');
        const data: User = await response.json();
        setUser(data);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, []);
  
  return (
    <View>
      {loading && <Text>Loading...</Text>}
      {user && <Text>{user.name}</Text>}
    </View>
  );
};
```

### Tipizzare Navigation

```typescript
// types/navigation.ts
export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: { section?: string };
};

// Nei componenti
import { NativeStackScreenProps } from '@react-navigation/native-stack';
import { RootStackParamList } from './types/navigation';

type ProfileScreenProps = NativeStackScreenProps<RootStackParamList, 'Profile'>;

const ProfileScreen = ({ route, navigation }: ProfileScreenProps) => {
  const { userId } = route.params;  // TypeScript sa che userId è string ✅
  
  navigation.navigate('Settings', { section: 'privacy' });  // Type-safe ✅
  
  return <View><Text>User ID: {userId}</Text></View>;
};
```

### Utility Types

```typescript
// Partial - rende tutte le proprietà opzionali
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// Equivale a: { id?: number; name?: string; email?: string; }

const updateUser = (id: number, updates: Partial<User>) => {
  // Puoi passare solo le proprietà da aggiornare
};
updateUser(1, { name: 'New Name' });  // OK ✅

// Pick - seleziona solo alcune proprietà
type UserPreview = Pick<User, 'id' | 'name'>;
// Equivale a: { id: number; name: string; }

// Omit - rimuove alcune proprietà
type UserWithoutId = Omit<User, 'id'>;
// Equivale a: { name: string; email: string; }

// Readonly - rende tutto readonly
type ReadonlyUser = Readonly<User>;
// Proprietà non modificabili

// Record - crea oggetto con chiavi e valori tipizzati
type UserRoles = Record<string, string[]>;
const roles: UserRoles = {
  admin: ['read', 'write', 'delete'],
  user: ['read']
};
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 ES6 Basics

**Obiettivo**: Praticare le feature ES6

**Task**: Riscrivi questo codice usando feature moderne:

```javascript
// Codice da migliorare:
var userName = 'John Doe';
var userAge = 30;

function getUserInfo(user) {
  var name = user.name;
  var age = user.age;
  return 'Name: ' + name + ', Age: ' + age;
}

var numbers = [1, 2, 3, 4, 5];
var doubled = [];
for (var i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}
```

**Soluzione attesa**: const/let, destructuring, template literals, arrow functions, map()

### Esercizio 2: 🟡 Async/Await

**Obiettivo**: Gestire operazioni asincrone

**Task**: Crea una funzione che:
1. Fetcha dati da `https://jsonplaceholder.typicode.com/users`
2. Filtra solo utenti con username che inizia con 'C'
3. Ritorna array di nomi

```typescript
const fetchFilteredUsers = async (): Promise<string[]> => {
  // Il tuo codice qui
};
```

### Esercizio 3: 🟡 TypeScript Componente

**Obiettivo**: Tipizzare un componente React Native

**Task**: Crea un componente `UserCard` con TypeScript:

```typescript
interface UserCardProps {
  // Definisci props: name, email, avatar (URL), age (opzionale)
}

const UserCard: React.FC<UserCardProps> = (props) => {
  // Implementa il componente
  // Mostra immagine, nome, email
  // Se age presente, mostra anche quello
};
```

### Esercizio 4: 🔴 Generic Hook

**Obiettivo**: Creare un custom hook generico

**Task**: Crea `useFetch<T>` hook:

```typescript
function useFetch<T>(url: string): {
  data: T | null;
  loading: boolean;
  error: string | null;
} {
  // Implementa il hook
  // Usa useState, useEffect
  // Fetcha dati, gestisci loading ed errori
}

// Uso:
const { data, loading, error } = useFetch<User[]>('/api/users');
```

### Esercizio 5: 🔴 Refactoring con TypeScript

**Obiettivo**: Convertire JavaScript in TypeScript

**Task**: Dato questo codice JS, aggiungi TypeScript completo:

```javascript
// JavaScript
const TodoList = ({ todos, onToggle, onDelete }) => {
  const [filter, setFilter] = useState('all');
  
  const filteredTodos = todos.filter(todo => {
    if (filter === 'completed') return todo.completed;
    if (filter === 'active') return !todo.completed;
    return true;
  });
  
  return (
    <FlatList
      data={filteredTodos}
      renderItem={({ item }) => (
        <TodoItem 
          todo={item}
          onToggle={() => onToggle(item.id)}
          onDelete={() => onDelete(item.id)}
        />
      )}
    />
  );
};
```

Aggiungi:
- Interface per Todo
- Interface per props di TodoList e TodoItem
- Tipizzazione corretta di tutti gli hooks e callbacks

---

## 📝 Quiz di Autovalutazione

1. **Qual è la differenza tra `let` e `const`?**
   - [ ] let è per numeri, const per stringhe
   - [x] const non può essere riassegnato, let sì
   - [ ] Non c'è differenza
   - [ ] let è deprecato

2. **Cosa fa questo codice? `const [a, , c] = [1, 2, 3]`**
   - [ ] Errore di sintassi
   - [x] a=1, c=3 (salta il secondo elemento)
   - [ ] a=1, c=2
   - [ ] a=[1,2,3], c undefined

3. **Differenza tra `...` come spread e rest?**
   - [x] Spread espande, rest raccoglie
   - [ ] Sono identici
   - [ ] Spread per array, rest per oggetti
   - [ ] Rest è deprecato

4. **Quando usare `async/await` invece di `.then()`?**
   - [ ] Mai, .then() è più moderno
   - [ ] Solo per chiamate multiple
   - [x] Sempre, è più leggibile
   - [ ] Solo con TypeScript

5. **Vantaggio principale di TypeScript?**
   - [ ] È più veloce
   - [ ] Ha più feature di JS
   - [x] Rileva errori durante lo sviluppo
   - [ ] È richiesto da React Native

**Risposte**: 1-b, 2-b, 3-a, 4-c, 5-c

---

## 🔗 Risorse Aggiuntive

### JavaScript
- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [JavaScript.info](https://javascript.info/)
- [ES6 Features](http://es6-features.org/)

### TypeScript
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Playground](https://www.typescriptlang.org/play)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

### Corsi
- [Codecademy - Learn JavaScript](https://www.codecademy.com/learn/introduction-to-javascript)
- [TypeScript for Beginners](https://www.youtube.com/watch?v=BwuLxPH8IDs)

---

## ✅ Checklist di Completamento

- [ ] Comprensione di const/let vs var
- [ ] Padronanza arrow functions
- [ ] Destructuring di oggetti e array
- [ ] Spread e rest operator utilizzati correttamente
- [ ] Promises e async/await per operazioni asincrone
- [ ] Import/export moduli ES6
- [ ] TypeScript configurato in un progetto
- [ ] Interfacce e type aliases creati
- [ ] Componenti React Native tipizzati
- [ ] Almeno 3 esercizi completati su 5

---

## 📌 Punti Chiave da Ricordare

1. 🔒 **Usa `const` di default**, `let` solo se cambia, mai `var`
2. ➡️ **Arrow functions** sono perfette per React Native
3. 📦 **Destructuring** rende il codice più pulito
4. 🎯 **Spread operator** per aggiornamenti immutabili
5. ⏳ **Async/await** > Promises per leggibilità
6. 📝 **TypeScript** previene errori e migliora DX
7. 🎨 **Interfacce** documentano la forma dei dati
8. 🔧 **Generics** per codice riutilizzabile

---

**Eccellente! 🎉** Ora hai una solida base di JavaScript moderno e TypeScript per React Native. Nel prossimo capitolo approfondiremo React: componenti, props, state, hooks!

[← Precedente: Ambiente di Sviluppo](02_Ambiente_Sviluppo.md) | [Torna all'Indice](../README.md) | [Prossimo: Fondamenti di React →](04_Fondamenti_React.md)
