# Capitolo 11: Networking e API REST

[← Precedente: Form e Validazione](../02%20-%20Sviluppo%20dell'Interfaccia%20Utente/10_Form_Validazione.md) | [Torna all'Indice](../README.md) | [Prossimo: State Management →](12_State_Management.md)

---

## Descrizione

La maggior parte delle app moderne comunica con server remoti per recuperare e inviare dati. React Native offre diverse opzioni per effettuare chiamate HTTP: dalla Fetch API nativa ad Axios, fino a soluzioni moderne come React Query e SWR per la gestione dello stato asincrono.

In questo capitolo imparerai a effettuare chiamate API REST, gestire autenticazione, implementare caching, gestire errori di rete, e ottimizzare le performance delle chiamate network.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Usare Fetch API per chiamate HTTP
- [ ] Implementare Axios per richieste complesse
- [ ] Gestire autenticazione JWT
- [ ] Implementare refresh token
- [ ] Usare React Query per data fetching
- [ ] Gestire caching e invalidation
- [ ] Implementare retry logic
- [ ] Gestire errori di rete
- [ ] Ottimizzare chiamate API
- [ ] Debuggare network requests

## 📚 Argomenti Teorici

1. [Fetch API](#1-fetch-api)
2. [Axios](#2-axios)
3. [Gestione Errori](#3-error-handling)
4. [Autenticazione JWT](#4-authentication)
5. [Interceptors](#5-interceptors)
6. [React Query](#6-react-query)
7. [SWR](#7-swr)
8. [File Upload](#8-file-upload)
9. [Network Debugging](#9-debugging)
10. [Best Practices](#10-best-practices)

---

## 1. Fetch API

### GET Request

```typescript
const fetchUsers = async () => {
  try {
    const response = await fetch('https://api.example.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
};

// Uso in component
const UsersScreen = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error.message}</Text>;
  
  return (
    <FlatList
      data={users}
      renderItem={({ item }) => <UserItem user={item} />}
    />
  );
};
```

### POST Request

```typescript
const createUser = async (userData: UserData) => {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error('Failed to create user');
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};
```

### PUT/PATCH Request

```typescript
const updateUser = async (userId: string, updates: Partial<User>) => {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(updates)
  });
  
  return response.json();
};
```

### DELETE Request

```typescript
const deleteUser = async (userId: string) => {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'DELETE'
  });
  
  if (!response.ok) {
    throw new Error('Failed to delete user');
  }
  
  return response.json();
};
```

### Headers e Auth

```typescript
const fetchProtectedData = async (token: string) => {
  const response = await fetch('https://api.example.com/protected', {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  
  return response.json();
};
```

---

## 2. Axios

### Installation

```bash
npm install axios
```

### Basic Usage

```typescript
import axios from 'axios';

// GET
const getUsers = async () => {
  const response = await axios.get('https://api.example.com/users');
  return response.data;
};

// POST
const createUser = async (userData: UserData) => {
  const response = await axios.post('https://api.example.com/users', userData);
  return response.data;
};

// PUT/PATCH
const updateUser = async (userId: string, updates: Partial<User>) => {
  const response = await axios.patch(`https://api.example.com/users/${userId}`, updates);
  return response.data;
};

// DELETE
const deleteUser = async (userId: string) => {
  const response = await axios.delete(`https://api.example.com/users/${userId}`);
  return response.data;
};
```

### Axios Instance

```typescript
// api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

export default apiClient;

// Uso
import apiClient from './api/client';

const getUsers = () => apiClient.get('/users');
const createUser = (data: UserData) => apiClient.post('/users', data);
```

### Request Config

```typescript
const response = await axios.get('/users', {
  params: {
    page: 1,
    limit: 10,
    sort: 'name'
  },
  headers: {
    'Authorization': `Bearer ${token}`
  },
  timeout: 5000
});
```

### Response Schema

```typescript
const response = await axios.get('/users');

// response object
{
  data: {},           // Response body
  status: 200,        // HTTP status code
  statusText: 'OK',   // HTTP status message
  headers: {},        // Response headers
  config: {},         // Request config
  request: {}         // XMLHttpRequest
}
```

---

## 3. Error Handling

### Fetch Error Handling

```typescript
const fetchWithErrorHandling = async (url: string) => {
  try {
    const response = await fetch(url);
    
    // Check HTTP status
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || `HTTP ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof TypeError) {
      // Network error
      throw new Error('Network error. Please check your connection.');
    }
    throw error;
  }
};
```

### Axios Error Handling

```typescript
const fetchUsers = async () => {
  try {
    const response = await axios.get('/users');
    return response.data;
  } catch (error) {
    if (axios.isAxiosError(error)) {
      if (error.response) {
        // Server responded with error status
        console.error('Response error:', error.response.status);
        console.error('Error data:', error.response.data);
      } else if (error.request) {
        // Request made but no response
        console.error('No response received');
      } else {
        // Error in request setup
        console.error('Request setup error:', error.message);
      }
    }
    throw error;
  }
};
```

### Custom Error Types

```typescript
class ApiError extends Error {
  constructor(
    public status: number,
    public message: string,
    public data?: any
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

const apiCall = async (url: string) => {
  const response = await fetch(url);
  
  if (!response.ok) {
    const errorData = await response.json();
    throw new ApiError(
      response.status,
      errorData.message,
      errorData
    );
  }
  
  return response.json();
};

// Uso
try {
  const data = await apiCall('/users');
} catch (error) {
  if (error instanceof ApiError) {
    if (error.status === 401) {
      // Handle unauthorized
    } else if (error.status === 404) {
      // Handle not found
    }
  }
}
```

---

## 4. Authentication

### Login Flow

```typescript
interface LoginCredentials {
  email: string;
  password: string;
}

interface AuthResponse {
  accessToken: string;
  refreshToken: string;
  user: User;
}

const login = async (credentials: LoginCredentials): Promise<AuthResponse> => {
  const response = await axios.post('/auth/login', credentials);
  return response.data;
};

// In component
const LoginScreen = () => {
  const handleLogin = async (email: string, password: string) => {
    try {
      const { accessToken, refreshToken, user } = await login({ email, password });
      
      // Store tokens
      await AsyncStorage.setItem('accessToken', accessToken);
      await AsyncStorage.setItem('refreshToken', refreshToken);
      
      // Update auth context
      setUser(user);
      setAuthenticated(true);
    } catch (error) {
      Alert.alert('Login Failed', error.message);
    }
  };
  
  return <LoginForm onSubmit={handleLogin} />;
};
```

### Token Storage

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

const TOKEN_KEY = 'accessToken';
const REFRESH_TOKEN_KEY = 'refreshToken';

export const TokenService = {
  async getAccessToken(): Promise<string | null> {
    return AsyncStorage.getItem(TOKEN_KEY);
  },
  
  async setAccessToken(token: string): Promise<void> {
    return AsyncStorage.setItem(TOKEN_KEY, token);
  },
  
  async getRefreshToken(): Promise<string | null> {
    return AsyncStorage.getItem(REFRESH_TOKEN_KEY);
  },
  
  async setRefreshToken(token: string): Promise<void> {
    return AsyncStorage.setItem(REFRESH_TOKEN_KEY, token);
  },
  
  async clearTokens(): Promise<void> {
    await AsyncStorage.multiRemove([TOKEN_KEY, REFRESH_TOKEN_KEY]);
  }
};
```

### JWT Decoding

```bash
npm install jwt-decode
```

```typescript
import jwtDecode from 'jwt-decode';

interface JWTPayload {
  sub: string;
  exp: number;
  iat: number;
  email: string;
}

const isTokenExpired = (token: string): boolean => {
  try {
    const decoded = jwtDecode<JWTPayload>(token);
    return decoded.exp * 1000 < Date.now();
  } catch {
    return true;
  }
};

// Check before requests
const makeAuthenticatedRequest = async () => {
  const token = await TokenService.getAccessToken();
  
  if (!token || isTokenExpired(token)) {
    // Refresh token or logout
    await refreshAccessToken();
  }
  
  // Make request with valid token
};
```

---

## 5. Interceptors

### Axios Request Interceptor

```typescript
apiClient.interceptors.request.use(
  async (config) => {
    // Add auth token to every request
    const token = await TokenService.getAccessToken();
    
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    console.log('Request:', config.method?.toUpperCase(), config.url);
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

### Axios Response Interceptor

```typescript
apiClient.interceptors.response.use(
  (response) => {
    // Log successful responses
    console.log('Response:', response.status, response.config.url);
    return response;
  },
  async (error) => {
    const originalRequest = error.config;
    
    // Handle 401 Unauthorized
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        // Refresh token
        const newToken = await refreshAccessToken();
        originalRequest.headers.Authorization = `Bearer ${newToken}`;
        
        // Retry original request
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed, logout user
        await logout();
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);
```

### Refresh Token Implementation

```typescript
const refreshAccessToken = async (): Promise<string> => {
  const refreshToken = await TokenService.getRefreshToken();
  
  if (!refreshToken) {
    throw new Error('No refresh token');
  }
  
  try {
    const response = await axios.post('/auth/refresh', {
      refreshToken
    });
    
    const { accessToken } = response.data;
    await TokenService.setAccessToken(accessToken);
    
    return accessToken;
  } catch (error) {
    await TokenService.clearTokens();
    throw error;
  }
};
```

---

## 6. React Query

### Installation

```bash
npm install @tanstack/react-query
```

### Setup

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 3,
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
    }
  }
});

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <Navigation />
    </QueryClientProvider>
  );
};
```

### useQuery

```typescript
import { useQuery } from '@tanstack/react-query';

const fetchUsers = async () => {
  const response = await apiClient.get('/users');
  return response.data;
};

const UsersScreen = () => {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers
  });
  
  if (isLoading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error.message}</Text>;
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <UserItem user={item} />}
      refreshControl={
        <RefreshControl refreshing={isLoading} onRefresh={refetch} />
      }
    />
  );
};
```

### Query with Parameters

```typescript
const fetchUser = async (userId: string) => {
  const response = await apiClient.get(`/users/${userId}`);
  return response.data;
};

const UserDetailScreen = ({ route }) => {
  const { userId } = route.params;
  
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId)
  });
  
  return <UserDetails user={user} />;
};
```

### useMutation

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';

const createUser = async (userData: UserData) => {
  const response = await apiClient.post('/users', userData);
  return response.data;
};

const CreateUserScreen = () => {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      // Invalidate and refetch users query
      queryClient.invalidateQueries({ queryKey: ['users'] });
      
      Alert.alert('Success', 'User created successfully');
    },
    onError: (error) => {
      Alert.alert('Error', error.message);
    }
  });
  
  const handleSubmit = (userData: UserData) => {
    mutation.mutate(userData);
  };
  
  return (
    <UserForm
      onSubmit={handleSubmit}
      isLoading={mutation.isPending}
    />
  );
};
```

### Optimistic Updates

```typescript
const updateUser = async ({ userId, updates }: UpdateUserParams) => {
  const response = await apiClient.patch(`/users/${userId}`, updates);
  return response.data;
};

const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async ({ userId, updates }) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['user', userId] });
    
    // Snapshot previous value
    const previousUser = queryClient.getQueryData(['user', userId]);
    
    // Optimistically update
    queryClient.setQueryData(['user', userId], (old: User) => ({
      ...old,
      ...updates
    }));
    
    return { previousUser };
  },
  onError: (error, variables, context) => {
    // Rollback on error
    queryClient.setQueryData(
      ['user', variables.userId],
      context.previousUser
    );
  },
  onSettled: (data, error, variables) => {
    // Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['user', variables.userId] });
  }
});
```

---

## 7. SWR

### Installation

```bash
npm install swr
```

### Basic Usage

```typescript
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

const UsersScreen = () => {
  const { data, error, isLoading, mutate } = useSWR('/api/users', fetcher);
  
  if (isLoading) return <ActivityIndicator />;
  if (error) return <Text>Error loading users</Text>;
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <UserItem user={item} />}
      refreshControl={
        <RefreshControl refreshing={isLoading} onRefresh={() => mutate()} />
      }
    />
  );
};
```

### Global Config

```typescript
import { SWRConfig } from 'swr';

const App = () => {
  return (
    <SWRConfig
      value={{
        fetcher: (url) => axios.get(url).then(res => res.data),
        revalidateOnFocus: false,
        shouldRetryOnError: true,
        errorRetryCount: 3
      }}
    >
      <Navigation />
    </SWRConfig>
  );
};
```

---

## 8. File Upload

### FormData Upload

```typescript
const uploadImage = async (uri: string) => {
  const formData = new FormData();
  
  formData.append('file', {
    uri,
    type: 'image/jpeg',
    name: 'photo.jpg'
  } as any);
  
  const response = await fetch('https://api.example.com/upload', {
    method: 'POST',
    body: formData,
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  });
  
  return response.json();
};
```

### With Progress

```typescript
import axios from 'axios';

const uploadWithProgress = async (
  uri: string,
  onProgress: (progress: number) => void
) => {
  const formData = new FormData();
  formData.append('file', {
    uri,
    type: 'image/jpeg',
    name: 'photo.jpg'
  } as any);
  
  const response = await axios.post('/upload', formData, {
    headers: {
      'Content-Type': 'multipart/form-data'
    },
    onUploadProgress: (progressEvent) => {
      const percentCompleted = Math.round(
        (progressEvent.loaded * 100) / progressEvent.total
      );
      onProgress(percentCompleted);
    }
  });
  
  return response.data;
};

// In component
const [uploadProgress, setUploadProgress] = useState(0);

const handleUpload = async (uri: string) => {
  try {
    const result = await uploadWithProgress(uri, setUploadProgress);
    console.log('Upload complete:', result);
  } catch (error) {
    console.error('Upload failed:', error);
  }
};
```

---

## 9. Debugging

### React Native Debugger

```typescript
// Enable network inspect in React Native Debugger
if (__DEV__) {
  global.XMLHttpRequest = global.originalXMLHttpRequest || global.XMLHttpRequest;
}
```

### Axios Logger

```typescript
apiClient.interceptors.request.use(request => {
  console.log('Starting Request', {
    url: request.url,
    method: request.method,
    data: request.data
  });
  return request;
});

apiClient.interceptors.response.use(
  response => {
    console.log('Response:', {
      status: response.status,
      data: response.data
    });
    return response;
  },
  error => {
    console.error('Error Response:', {
      status: error.response?.status,
      data: error.response?.data
    });
    return Promise.reject(error);
  }
);
```

### Flipper Network Plugin

```bash
# Flipper automatically shows network requests
# No code changes needed, just open Flipper desktop app
```

---

## 10. Best Practices

### 1. Centralize API Calls

```typescript
// api/users.ts
export const usersApi = {
  getAll: () => apiClient.get('/users'),
  getById: (id: string) => apiClient.get(`/users/${id}`),
  create: (data: UserData) => apiClient.post('/users', data),
  update: (id: string, data: Partial<User>) => apiClient.patch(`/users/${id}`, data),
  delete: (id: string) => apiClient.delete(`/users/${id}`)
};
```

### 2. Type Safety

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

const getUsers = async (): Promise<User[]> => {
  const response = await apiClient.get<ApiResponse<User[]>>('/users');
  return response.data.data;
};
```

### 3. Error Boundaries

```typescript
const ApiErrorBoundary = ({ children }) => {
  return (
    <ErrorBoundary
      fallback={(error) => (
        <View>
          <Text>Something went wrong</Text>
          <Text>{error.message}</Text>
          <Button title="Retry" onPress={() => window.location.reload()} />
        </View>
      )}
    >
      {children}
    </ErrorBoundary>
  );
};
```

### 4. Retry Logic

```typescript
const fetchWithRetry = async (
  url: string,
  maxRetries = 3,
  delay = 1000
): Promise<any> => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url);
      if (response.ok) return await response.json();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
};
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Basic API Integration

Crea app con:
- GET list di users da JSONPlaceholder
- Mostra in FlatList
- Pull to refresh
- Loading e error states

### Esercizio 2: 🟡 CRUD Operations

Implementa CRUD completo:
- Create, Read, Update, Delete
- Form per create/edit
- Confirmation per delete
- Optimistic updates
- Error handling

### Esercizio 3: 🟡 Authentication System

Sistema auth completo:
- Login/Logout
- JWT token management
- Auto-refresh token
- Protected routes
- Persist auth state

### Esercizio 4: 🔴 React Query Integration

App con React Query:
- Multiple queries
- Mutations con optimistic updates
- Cache invalidation
- Pagination
- Infinite scroll

### Esercizio 5: 🔴 Advanced Networking

App completa con:
- Axios interceptors
- Retry logic
- Request cancellation
- File upload con progress
- Offline support
- Network status indicator

---

## 📝 Quiz

1. **Differenza fetch vs axios?**
   - [ ] Sono identici
   - [x] Axios ha più features (interceptors, timeout, auto JSON)
   - [ ] Fetch è deprecato
   - [ ] Axios è più lento

2. **JWT token va salvato in:**
   - [ ] State
   - [ ] Redux
   - [x] AsyncStorage (secure storage meglio)
   - [ ] Props

3. **React Query cache di default:**
   - [ ] Non fa caching
   - [ ] Cache infinita
   - [x] 5 min stale, 10 min cache
   - [ ] 1 ora

4. **Refresh token serve per:**
   - [ ] Refresh UI
   - [x] Ottenere nuovo access token
   - [ ] Logout
   - [ ] Validare form

5. **Interceptor migliore per auth token:**
   - [x] Request interceptor
   - [ ] Response interceptor
   - [ ] Entrambi
   - [ ] Nessuno

**Risposte**: 1-b, 2-c, 3-c, 4-b, 5-a

---

## 🔗 Risorse

- [Fetch API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [Axios Documentation](https://axios-http.com/)
- [React Query Docs](https://tanstack.com/query/latest)
- [SWR Documentation](https://swr.vercel.app/)

---

## ✅ Checklist

- [ ] Fetch API utilizzata
- [ ] Axios configurato
- [ ] Error handling implementato
- [ ] Authentication JWT
- [ ] Interceptors usati
- [ ] React Query o SWR integrato
- [ ] File upload funzionante
- [ ] Network debugging attivo
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🌐 **Fetch API** nativa, **Axios** più features
2. 🔐 **JWT** in AsyncStorage, refresh token per sicurezza
3. 🔄 **Interceptors** per auth e retry
4. ⚡ **React Query** gestisce caching automatico
5. 📤 **FormData** per file upload
6. 🐛 **Flipper** per debug network
7. 🔁 **Retry logic** per reliability
8. 🎯 **Type safety** con TypeScript

---

**Eccezionale! 🎉** Ora sai comunicare con API come un pro. Prossimo: State Management Globale!

[← Precedente: Form e Validazione](../02%20-%20Sviluppo%20dell'Interfaccia%20Utente/10_Form_Validazione.md) | [Torna all'Indice](../README.md) | [Prossimo: State Management →](12_State_Management.md)
