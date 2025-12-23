# Capitolo 25: Progetto Completo - Social Media App

[← Precedente: New Architecture](24_New_Architecture.md) | [Torna all'Indice](../README.md)

---

## Descrizione

Costruiamo app social completa applicando tutti concetti: Clean Architecture per codice scalabile, navigation con React Navigation, state management con Redux Toolkit, autenticazione con JWT, real-time chat con WebSockets, camera per foto, geolocation, push notifications, offline-first con AsyncStorage, testing con Jest/Detox, deployment automatizzato.

In questo capitolo costruirai "SocialHub": social media app con profili utente, post con immagini, like/comments, chat real-time, notifiche push, geolocalizzazione, ricerca, offline support, e deployment completo.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Progettare architettura completa
- [ ] Implementare autenticazione JWT
- [ ] Creare feed con infinite scroll
- [ ] Gestire uploads immagini
- [ ] Implementare chat real-time
- [ ] Integrare push notifications
- [ ] Aggiungere geolocation
- [ ] Implementare offline-first
- [ ] Testare app completamente
- [ ] Deploy su stores

## 📚 Argomenti

1. [Project Architecture](#1-architecture)
2. [Authentication System](#2-authentication)
3. [User Profiles](#3-profiles)
4. [Posts Feed](#4-feed)
5. [Real-time Chat](#5-chat)
6. [Notifications](#6-notifications)
7. [Image Upload](#7-images)
8. [Geolocation](#8-geolocation)
9. [Offline Support](#9-offline)
10. [Testing & Deployment](#10-deployment)

---

## 1. Architecture

### Project Structure

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── SignupForm.tsx
│   │   ├── screens/
│   │   │   ├── LoginScreen.tsx
│   │   │   └── SignupScreen.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── store/
│   │   │   └── authSlice.ts
│   │   └── api/
│   │       └── authApi.ts
│   │
│   ├── posts/
│   │   ├── components/
│   │   │   ├── PostCard.tsx
│   │   │   ├── PostForm.tsx
│   │   │   └── CommentList.tsx
│   │   ├── screens/
│   │   │   ├── FeedScreen.tsx
│   │   │   └── PostDetailScreen.tsx
│   │   ├── hooks/
│   │   │   └── usePosts.ts
│   │   ├── store/
│   │   │   └── postsSlice.ts
│   │   └── api/
│   │       └── postsApi.ts
│   │
│   ├── chat/
│   │   ├── components/
│   │   │   ├── ChatBubble.tsx
│   │   │   └── MessageInput.tsx
│   │   ├── screens/
│   │   │   ├── ChatsListScreen.tsx
│   │   │   └── ChatScreen.tsx
│   │   ├── hooks/
│   │   │   └── useChat.ts
│   │   ├── store/
│   │   │   └── chatSlice.ts
│   │   └── services/
│   │       └── websocket.ts
│   │
│   └── profile/
│       ├── components/
│       │   └── ProfileHeader.tsx
│       ├── screens/
│       │   ├── ProfileScreen.tsx
│       │   └── EditProfileScreen.tsx
│       ├── hooks/
│       │   └── useProfile.ts
│       ├── store/
│       │   └── profileSlice.ts
│       └── api/
│           └── profileApi.ts
│
├── shared/
│   ├── components/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Avatar.tsx
│   │   └── LoadingSpinner.tsx
│   ├── hooks/
│   │   ├── useDebounce.ts
│   │   ├── useImagePicker.ts
│   │   └── useLocation.ts
│   ├── utils/
│   │   ├── storage.ts
│   │   ├── validation.ts
│   │   └── formatting.ts
│   └── types/
│       └── index.ts
│
├── navigation/
│   ├── RootNavigator.tsx
│   ├── AuthNavigator.tsx
│   └── MainNavigator.tsx
│
├── store/
│   ├── index.ts
│   └── middleware/
│       └── offlineSync.ts
│
└── services/
    ├── api.ts
    ├── notifications.ts
    └── camera.ts
```

### Core Dependencies

```json
// package.json
{
  "dependencies": {
    "react": "18.2.0",
    "react-native": "0.74.0",
    "@react-navigation/native": "^6.1.9",
    "@react-navigation/stack": "^6.3.20",
    "@react-navigation/bottom-tabs": "^6.5.11",
    "@reduxjs/toolkit": "^2.0.1",
    "react-redux": "^9.0.4",
    "axios": "^1.6.2",
    "socket.io-client": "^4.5.4",
    "@react-native-async-storage/async-storage": "^1.21.0",
    "@react-native-firebase/app": "^19.0.0",
    "@react-native-firebase/messaging": "^19.0.0",
    "react-native-image-picker": "^7.1.0",
    "react-native-geolocation-service": "^5.3.1",
    "react-native-maps": "^1.10.0",
    "react-native-fast-image": "^8.6.3"
  },
  "devDependencies": {
    "@testing-library/react-native": "^12.4.2",
    "detox": "^20.14.6",
    "jest": "^29.7.0"
  }
}
```

---

## 2. Authentication

### Auth Store

```typescript
// features/auth/store/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { authApi } from '../api/authApi';
import { storage } from '../../../shared/utils/storage';

interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  loading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  token: null,
  loading: false,
  error: null
};

export const login = createAsyncThunk(
  'auth/login',
  async ({ email, password }: { email: string; password: string }) => {
    const response = await authApi.login(email, password);
    await storage.setItem('token', response.token);
    await storage.setItem('user', JSON.stringify(response.user));
    return response;
  }
);

export const signup = createAsyncThunk(
  'auth/signup',
  async ({ email, password, name }: { email: string; password: string; name: string }) => {
    const response = await authApi.signup(email, password, name);
    await storage.setItem('token', response.token);
    await storage.setItem('user', JSON.stringify(response.user));
    return response;
  }
);

export const loadUser = createAsyncThunk(
  'auth/loadUser',
  async () => {
    const token = await storage.getItem('token');
    const userStr = await storage.getItem('user');
    if (!token || !userStr) throw new Error('No user stored');
    return { token, user: JSON.parse(userStr) };
  }
);

export const logout = createAsyncThunk(
  'auth/logout',
  async () => {
    await storage.removeItem('token');
    await storage.removeItem('user');
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(login.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Login failed';
      })
      .addCase(signup.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(signup.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(signup.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Signup failed';
      })
      .addCase(loadUser.fulfilled, (state, action) => {
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(logout.fulfilled, (state) => {
        state.user = null;
        state.token = null;
      });
  }
});

export const { clearError } = authSlice.actions;
export default authSlice.reducer;
```

### Login Screen

```typescript
// features/auth/screens/LoginScreen.tsx
import React, { useState } from 'react';
import { View, StyleSheet, KeyboardAvoidingView, Platform } from 'react-native';
import { useAppDispatch, useAppSelector } from '../../../store/hooks';
import { login } from '../store/authSlice';
import { Input } from '../../../shared/components/Input';
import { Button } from '../../../shared/components/Button';
import { Text } from 'react-native';

export const LoginScreen: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const dispatch = useAppDispatch();
  const { loading, error } = useAppSelector(state => state.auth);
  
  const handleLogin = async () => {
    if (!email || !password) {
      alert('Please fill all fields');
      return;
    }
    
    try {
      await dispatch(login({ email, password })).unwrap();
      // Navigation handled by RootNavigator
    } catch (err) {
      alert(err);
    }
  };
  
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={styles.container}
    >
      <View style={styles.form}>
        <Text style={styles.title}>Welcome Back!</Text>
        
        <Input
          placeholder="Email"
          value={email}
          onChangeText={setEmail}
          keyboardType="email-address"
          autoCapitalize="none"
        />
        
        <Input
          placeholder="Password"
          value={password}
          onChangeText={setPassword}
          secureTextEntry
        />
        
        {error && <Text style={styles.error}>{error}</Text>}
        
        <Button
          title="Login"
          onPress={handleLogin}
          loading={loading}
        />
        
        <Button
          title="Don't have an account? Sign up"
          onPress={() => {/* Navigate to signup */}}
          variant="text"
        />
      </View>
    </KeyboardAvoidingView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff'
  },
  form: {
    flex: 1,
    justifyContent: 'center',
    padding: 20
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 30,
    textAlign: 'center'
  },
  error: {
    color: 'red',
    marginBottom: 10
  }
});
```

---

## 3. Profiles

### Profile API

```typescript
// features/profile/api/profileApi.ts
import { api } from '../../../services/api';

export interface Profile {
  id: string;
  name: string;
  email: string;
  bio?: string;
  avatar?: string;
  location?: {
    latitude: number;
    longitude: number;
    address: string;
  };
  postsCount: number;
  followersCount: number;
  followingCount: number;
}

export const profileApi = {
  getProfile: async (userId: string): Promise<Profile> => {
    const response = await api.get(`/users/${userId}`);
    return response.data;
  },
  
  updateProfile: async (updates: Partial<Profile>): Promise<Profile> => {
    const response = await api.put('/users/me', updates);
    return response.data;
  },
  
  uploadAvatar: async (imageUri: string): Promise<string> => {
    const formData = new FormData();
    formData.append('avatar', {
      uri: imageUri,
      type: 'image/jpeg',
      name: 'avatar.jpg'
    } as any);
    
    const response = await api.post('/users/avatar', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    });
    
    return response.data.avatarUrl;
  }
};
```

### Profile Screen

```typescript
// features/profile/screens/ProfileScreen.tsx
import React, { useEffect } from 'react';
import { View, Text, StyleSheet, ScrollView, TouchableOpacity } from 'react-native';
import FastImage from 'react-native-fast-image';
import { useAppDispatch, useAppSelector } from '../../../store/hooks';
import { fetchProfile } from '../store/profileSlice';
import { Button } from '../../../shared/components/Button';

interface Props {
  route: { params: { userId: string } };
  navigation: any;
}

export const ProfileScreen: React.FC<Props> = ({ route, navigation }) => {
  const { userId } = route.params;
  const dispatch = useAppDispatch();
  const { profile, loading } = useAppSelector(state => state.profile);
  const currentUser = useAppSelector(state => state.auth.user);
  
  useEffect(() => {
    dispatch(fetchProfile(userId));
  }, [userId]);
  
  if (loading || !profile) return <Text>Loading...</Text>;
  
  const isOwnProfile = currentUser?.id === userId;
  
  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <FastImage
          source={{ uri: profile.avatar || 'https://via.placeholder.com/150' }}
          style={styles.avatar}
        />
        
        <Text style={styles.name}>{profile.name}</Text>
        {profile.bio && <Text style={styles.bio}>{profile.bio}</Text>}
        
        <View style={styles.stats}>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{profile.postsCount}</Text>
            <Text style={styles.statLabel}>Posts</Text>
          </View>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{profile.followersCount}</Text>
            <Text style={styles.statLabel}>Followers</Text>
          </View>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{profile.followingCount}</Text>
            <Text style={styles.statLabel}>Following</Text>
          </View>
        </View>
        
        {isOwnProfile ? (
          <Button
            title="Edit Profile"
            onPress={() => navigation.navigate('EditProfile')}
          />
        ) : (
          <Button title="Follow" onPress={() => {/* Handle follow */}} />
        )}
      </View>
      
      {/* User's posts grid */}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff'
  },
  header: {
    padding: 20,
    alignItems: 'center'
  },
  avatar: {
    width: 100,
    height: 100,
    borderRadius: 50
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
    marginTop: 10
  },
  bio: {
    fontSize: 14,
    color: '#666',
    marginTop: 5,
    textAlign: 'center'
  },
  stats: {
    flexDirection: 'row',
    marginTop: 20,
    marginBottom: 20
  },
  stat: {
    alignItems: 'center',
    marginHorizontal: 20
  },
  statValue: {
    fontSize: 20,
    fontWeight: 'bold'
  },
  statLabel: {
    fontSize: 12,
    color: '#666'
  }
});
```

---

## 4. Feed

### Posts Slice

```typescript
// features/posts/store/postsSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { postsApi } from '../api/postsApi';

export interface Post {
  id: string;
  authorId: string;
  authorName: string;
  authorAvatar?: string;
  content: string;
  images: string[];
  likes: number;
  liked: boolean;
  comments: number;
  location?: {
    latitude: number;
    longitude: number;
    address: string;
  };
  createdAt: string;
}

interface PostsState {
  items: Post[];
  loading: boolean;
  refreshing: boolean;
  hasMore: boolean;
  page: number;
}

const initialState: PostsState = {
  items: [],
  loading: false,
  refreshing: false,
  hasMore: true,
  page: 1
};

export const fetchPosts = createAsyncThunk(
  'posts/fetchPosts',
  async (page: number) => {
    const response = await postsApi.getPosts(page);
    return response;
  }
);

export const createPost = createAsyncThunk(
  'posts/createPost',
  async ({ content, images, location }: {
    content: string;
    images: string[];
    location?: { latitude: number; longitude: number; address: string };
  }) => {
    const response = await postsApi.createPost(content, images, location);
    return response;
  }
);

export const likePost = createAsyncThunk(
  'posts/likePost',
  async (postId: string) => {
    await postsApi.likePost(postId);
    return postId;
  }
);

const postsSlice = createSlice({
  name: 'posts',
  initialState,
  reducers: {
    resetPosts: (state) => {
      state.items = [];
      state.page = 1;
      state.hasMore = true;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state, action) => {
        if (action.meta.arg === 1) {
          state.refreshing = true;
        } else {
          state.loading = true;
        }
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.loading = false;
        state.refreshing = false;
        
        if (action.meta.arg === 1) {
          state.items = action.payload;
        } else {
          state.items = [...state.items, ...action.payload];
        }
        
        state.page = action.meta.arg + 1;
        state.hasMore = action.payload.length > 0;
      })
      .addCase(fetchPosts.rejected, (state) => {
        state.loading = false;
        state.refreshing = false;
      })
      .addCase(createPost.fulfilled, (state, action) => {
        state.items.unshift(action.payload);
      })
      .addCase(likePost.fulfilled, (state, action) => {
        const post = state.items.find(p => p.id === action.payload);
        if (post) {
          post.liked = !post.liked;
          post.likes += post.liked ? 1 : -1;
        }
      });
  }
});

export const { resetPosts } = postsSlice.actions;
export default postsSlice.reducer;
```

### Feed Screen

```typescript
// features/posts/screens/FeedScreen.tsx
import React, { useEffect, useCallback } from 'react';
import { FlatList, RefreshControl, StyleSheet } from 'react-native';
import { useAppDispatch, useAppSelector } from '../../../store/hooks';
import { fetchPosts, resetPosts } from '../store/postsSlice';
import { PostCard } from '../components/PostCard';
import { LoadingSpinner } from '../../../shared/components/LoadingSpinner';

export const FeedScreen: React.FC = () => {
  const dispatch = useAppDispatch();
  const { items, loading, refreshing, hasMore, page } = useAppSelector(
    state => state.posts
  );
  
  useEffect(() => {
    dispatch(fetchPosts(1));
  }, []);
  
  const handleRefresh = useCallback(() => {
    dispatch(resetPosts());
    dispatch(fetchPosts(1));
  }, []);
  
  const handleLoadMore = useCallback(() => {
    if (!loading && hasMore) {
      dispatch(fetchPosts(page));
    }
  }, [loading, hasMore, page]);
  
  return (
    <FlatList
      data={items}
      renderItem={({ item }) => <PostCard post={item} />}
      keyExtractor={item => item.id}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={handleRefresh} />
      }
      onEndReached={handleLoadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={loading ? <LoadingSpinner /> : null}
      contentContainerStyle={styles.container}
    />
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#f5f5f5'
  }
});
```

### Post Card Component

```typescript
// features/posts/components/PostCard.tsx
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity, Dimensions } from 'react-native';
import FastImage from 'react-native-fast-image';
import { Post } from '../store/postsSlice';
import { useAppDispatch } from '../../../store/hooks';
import { likePost } from '../store/postsSlice';
import { formatDistanceToNow } from 'date-fns';

const { width } = Dimensions.get('window');

interface Props {
  post: Post;
}

export const PostCard: React.FC<Props> = ({ post }) => {
  const dispatch = useAppDispatch();
  
  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <FastImage
          source={{ uri: post.authorAvatar || 'https://via.placeholder.com/40' }}
          style={styles.avatar}
        />
        <View style={styles.headerText}>
          <Text style={styles.authorName}>{post.authorName}</Text>
          <Text style={styles.timestamp}>
            {formatDistanceToNow(new Date(post.createdAt), { addSuffix: true })}
          </Text>
        </View>
      </View>
      
      <Text style={styles.content}>{post.content}</Text>
      
      {post.images.length > 0 && (
        <FastImage
          source={{ uri: post.images[0] }}
          style={styles.image}
          resizeMode={FastImage.resizeMode.cover}
        />
      )}
      
      <View style={styles.actions}>
        <TouchableOpacity
          style={styles.action}
          onPress={() => dispatch(likePost(post.id))}
        >
          <Text style={[styles.actionText, post.liked && styles.liked]}>
            ❤️ {post.likes}
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.action}>
          <Text style={styles.actionText}>💬 {post.comments}</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    marginBottom: 10,
    padding: 15
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 10
  },
  avatar: {
    width: 40,
    height: 40,
    borderRadius: 20
  },
  headerText: {
    marginLeft: 10
  },
  authorName: {
    fontWeight: 'bold',
    fontSize: 16
  },
  timestamp: {
    color: '#666',
    fontSize: 12
  },
  content: {
    fontSize: 14,
    marginBottom: 10
  },
  image: {
    width: width - 30,
    height: width - 30,
    borderRadius: 10
  },
  actions: {
    flexDirection: 'row',
    marginTop: 10
  },
  action: {
    marginRight: 20
  },
  actionText: {
    fontSize: 14
  },
  liked: {
    color: 'red'
  }
});
```

---

## 5. Chat

### WebSocket Service

```typescript
// features/chat/services/websocket.ts
import io, { Socket } from 'socket.io-client';
import { storage } from '../../../shared/utils/storage';

class WebSocketService {
  private socket: Socket | null = null;
  
  async connect() {
    const token = await storage.getItem('token');
    
    this.socket = io('https://api.socialhub.com', {
      auth: { token },
      transports: ['websocket']
    });
    
    this.socket.on('connect', () => {
      console.log('WebSocket connected');
    });
    
    this.socket.on('disconnect', () => {
      console.log('WebSocket disconnected');
    });
  }
  
  disconnect() {
    if (this.socket) {
      this.socket.disconnect();
      this.socket = null;
    }
  }
  
  onMessage(callback: (message: any) => void) {
    if (this.socket) {
      this.socket.on('message', callback);
    }
  }
  
  sendMessage(chatId: string, content: string) {
    if (this.socket) {
      this.socket.emit('message', { chatId, content });
    }
  }
  
  joinChat(chatId: string) {
    if (this.socket) {
      this.socket.emit('join', chatId);
    }
  }
  
  leaveChat(chatId: string) {
    if (this.socket) {
      this.socket.emit('leave', chatId);
    }
  }
}

export const wsService = new WebSocketService();
```

### Chat Screen

```typescript
// features/chat/screens/ChatScreen.tsx
import React, { useEffect, useState, useRef } from 'react';
import {
  View,
  FlatList,
  KeyboardAvoidingView,
  Platform,
  StyleSheet
} from 'react-native';
import { useAppSelector } from '../../../store/hooks';
import { ChatBubble } from '../components/ChatBubble';
import { MessageInput } from '../components/MessageInput';
import { wsService } from '../services/websocket';

interface Message {
  id: string;
  senderId: string;
  content: string;
  timestamp: string;
}

interface Props {
  route: { params: { chatId: string; otherUser: any } };
}

export const ChatScreen: React.FC<Props> = ({ route }) => {
  const { chatId, otherUser } = route.params;
  const [messages, setMessages] = useState<Message[]>([]);
  const flatListRef = useRef<FlatList>(null);
  const currentUser = useAppSelector(state => state.auth.user);
  
  useEffect(() => {
    wsService.connect();
    wsService.joinChat(chatId);
    
    wsService.onMessage((message: Message) => {
      setMessages(prev => [...prev, message]);
      flatListRef.current?.scrollToEnd();
    });
    
    return () => {
      wsService.leaveChat(chatId);
    };
  }, [chatId]);
  
  const handleSend = (content: string) => {
    wsService.sendMessage(chatId, content);
  };
  
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : undefined}
      style={styles.container}
      keyboardVerticalOffset={90}
    >
      <FlatList
        ref={flatListRef}
        data={messages}
        renderItem={({ item }) => (
          <ChatBubble
            message={item}
            isOwn={item.senderId === currentUser?.id}
          />
        )}
        keyExtractor={item => item.id}
        contentContainerStyle={styles.messagesList}
      />
      
      <MessageInput onSend={handleSend} />
    </KeyboardAvoidingView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff'
  },
  messagesList: {
    padding: 10
  }
});
```

---

## 6. Notifications

### Push Notifications Setup

```typescript
// services/notifications.ts
import messaging from '@react-native-firebase/messaging';
import { storage } from '../shared/utils/storage';

export class NotificationService {
  async requestPermission() {
    const authStatus = await messaging().requestPermission();
    const enabled =
      authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
      authStatus === messaging.AuthorizationStatus.PROVISIONAL;
    
    if (enabled) {
      console.log('Authorization status:', authStatus);
      return true;
    }
    return false;
  }
  
  async getToken() {
    const token = await messaging().getToken();
    await storage.setItem('fcmToken', token);
    return token;
  }
  
  onTokenRefresh(callback: (token: string) => void) {
    return messaging().onTokenRefresh(callback);
  }
  
  onMessage(callback: (message: any) => void) {
    return messaging().onMessage(callback);
  }
  
  onNotificationOpenedApp(callback: (message: any) => void) {
    return messaging().onNotificationOpenedApp(callback);
  }
  
  async getInitialNotification() {
    return await messaging().getInitialNotification();
  }
  
  async sendTokenToServer(token: string) {
    // Send FCM token to your backend
    await api.post('/users/fcm-token', { token });
  }
}

export const notificationService = new NotificationService();
```

### App Setup with Notifications

```typescript
// App.tsx
import React, { useEffect } from 'react';
import { notificationService } from './services/notifications';
import { RootNavigator } from './navigation/RootNavigator';

const App: React.FC = () => {
  useEffect(() => {
    setupNotifications();
  }, []);
  
  const setupNotifications = async () => {
    const hasPermission = await notificationService.requestPermission();
    
    if (hasPermission) {
      const token = await notificationService.getToken();
      await notificationService.sendTokenToServer(token);
      
      // Handle foreground messages
      notificationService.onMessage((message) => {
        console.log('Foreground message:', message);
        // Show in-app notification
      });
      
      // Handle notification taps
      notificationService.onNotificationOpenedApp((message) => {
        console.log('Notification opened:', message);
        // Navigate to appropriate screen
      });
      
      // Check if app opened from notification
      const initialNotification = await notificationService.getInitialNotification();
      if (initialNotification) {
        console.log('App opened from notification:', initialNotification);
      }
    }
  };
  
  return <RootNavigator />;
};

export default App;
```

---

## 7. Images

### Image Picker Hook

```typescript
// shared/hooks/useImagePicker.ts
import { useState } from 'react';
import { launchCamera, launchImageLibrary, ImagePickerResponse } from 'react-native-image-picker';

export const useImagePicker = () => {
  const [loading, setLoading] = useState(false);
  
  const pickFromCamera = async (): Promise<string | null> => {
    setLoading(true);
    
    try {
      const result = await launchCamera({
        mediaType: 'photo',
        quality: 0.8,
        maxWidth: 1080,
        maxHeight: 1080
      });
      
      if (result.didCancel) return null;
      if (result.errorCode) throw new Error(result.errorMessage);
      
      return result.assets?.[0]?.uri || null;
    } finally {
      setLoading(false);
    }
  };
  
  const pickFromGallery = async (): Promise<string | null> => {
    setLoading(true);
    
    try {
      const result = await launchImageLibrary({
        mediaType: 'photo',
        quality: 0.8,
        maxWidth: 1080,
        maxHeight: 1080
      });
      
      if (result.didCancel) return null;
      if (result.errorCode) throw new Error(result.errorMessage);
      
      return result.assets?.[0]?.uri || null;
    } finally {
      setLoading(false);
    }
  };
  
  return {
    pickFromCamera,
    pickFromGallery,
    loading
  };
};
```

---

## 8. Geolocation

### Location Hook

```typescript
// shared/hooks/useLocation.ts
import { useState, useEffect } from 'react';
import Geolocation from 'react-native-geolocation-service';
import { PermissionsAndroid, Platform } from 'react-native';

interface Location {
  latitude: number;
  longitude: number;
}

export const useLocation = () => {
  const [location, setLocation] = useState<Location | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const requestPermission = async (): Promise<boolean> => {
    if (Platform.OS === 'android') {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    }
    return true;
  };
  
  const getCurrentLocation = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const hasPermission = await requestPermission();
      if (!hasPermission) {
        throw new Error('Location permission denied');
      }
      
      Geolocation.getCurrentPosition(
        (position) => {
          setLocation({
            latitude: position.coords.latitude,
            longitude: position.coords.longitude
          });
          setLoading(false);
        },
        (err) => {
          setError(err.message);
          setLoading(false);
        },
        { enableHighAccuracy: true, timeout: 15000, maximumAge: 10000 }
      );
    } catch (err) {
      setError(err.message);
      setLoading(false);
    }
  };
  
  return {
    location,
    loading,
    error,
    getCurrentLocation
  };
};
```

---

## 9. Offline

### Offline Middleware

```typescript
// store/middleware/offlineSync.ts
import { Middleware } from '@reduxjs/toolkit';
import NetInfo from '@react-native-community/netinfo';
import { storage } from '../../shared/utils/storage';

const OFFLINE_QUEUE_KEY = 'offlineQueue';

export const offlineSyncMiddleware: Middleware = store => next => action => {
  // Check if action is async thunk
  if (action.type.endsWith('/pending')) {
    NetInfo.fetch().then(state => {
      if (!state.isConnected) {
        // Queue action for later
        queueAction(action);
      }
    });
  }
  
  return next(action);
};

const queueAction = async (action: any) => {
  const queue = await storage.getItem(OFFLINE_QUEUE_KEY);
  const actions = queue ? JSON.parse(queue) : [];
  actions.push(action);
  await storage.setItem(OFFLINE_QUEUE_KEY, JSON.stringify(actions));
};

export const processOfflineQueue = async (dispatch: any) => {
  const queue = await storage.getItem(OFFLINE_QUEUE_KEY);
  if (!queue) return;
  
  const actions = JSON.parse(queue);
  
  for (const action of actions) {
    try {
      await dispatch(action);
    } catch (err) {
      console.error('Failed to process queued action:', err);
    }
  }
  
  await storage.removeItem(OFFLINE_QUEUE_KEY);
};
```

---

## 10. Deployment

### Environment Config

```bash
# .env.production
API_URL=https://api.socialhub.com
WS_URL=wss://api.socialhub.com
SENTRY_DSN=https://xxx@sentry.io/xxx
```

### Fastlane Config

```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Build and upload to TestFlight"
  lane :beta do
    increment_build_number(xcodeproj: "SocialHub.xcodeproj")
    gym(scheme: "SocialHub")
    pilot(skip_waiting_for_build_processing: true)
  end
  
  lane :release do
    increment_build_number(xcodeproj: "SocialHub.xcodeproj")
    gym(scheme: "SocialHub")
    deliver(force: true)
  end
end

platform :android do
  desc "Build and upload to Play Store"
  lane :beta do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(
      track: 'internal',
      release_status: 'draft'
    )
  end
  
  lane :release do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(
      track: 'production',
      release_status: 'draft'
    )
  end
end
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install pods
        run: cd ios && pod install
      
      - name: Run tests
        run: npm test
      
      - name: Build and deploy
        env:
          FASTLANE_USER: ${{ secrets.FASTLANE_USER }}
          FASTLANE_PASSWORD: ${{ secrets.FASTLANE_PASSWORD }}
        run: cd ios && fastlane beta
  
  android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build and deploy
        env:
          ANDROID_KEYSTORE: ${{ secrets.ANDROID_KEYSTORE }}
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        run: cd android && fastlane beta
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Setup Project

Inizializza SocialHub:
- React Native init
- Install dependencies
- Setup navigation
- Configure Firebase
- Test basic flow

### Esercizio 2: 🟡 Authentication

Implementa auth completo:
- Login/signup screens
- JWT storage
- Redux integration
- Protected routes
- Error handling

### Esercizio 3: 🟡 Posts Feed

Build feed con:
- Infinite scroll
- Pull to refresh
- Like functionality
- Image display
- Optimistic updates

### Esercizio 4: 🔴 Real-time Chat

Chat completo:
- WebSocket connection
- Message sending/receiving
- Typing indicators
- Read receipts
- Offline messages

### Esercizio 5: 🔴 Complete App

Finalize SocialHub:
- All features integrated
- Push notifications
- Image uploads
- Geolocation
- Offline support
- Tests (Jest + Detox)
- Deployment to stores

---

## 📝 Quiz

1. **Clean Architecture separa:**
   - [x] Features con components/screens/api/store
   - [ ] Solo UI
   - [ ] Solo business logic
   - [ ] Nessuna separazione

2. **Redux Toolkit usa:**
   - [ ] Actions manuali
   - [x] createSlice e createAsyncThunk
   - [ ] Solo context
   - [ ] Nessun state management

3. **WebSocket permette:**
   - [x] Comunicazione real-time bidirezionale
   - [ ] Solo HTTP requests
   - [ ] Solo polling
   - [ ] Nessuna comunicazione

4. **Offline-first significa:**
   - [x] App funziona senza connessione
   - [ ] Solo con connessione
   - [ ] Solo cache
   - [ ] Nessun offline

5. **CI/CD automatizza:**
   - [x] Build, test, e deployment
   - [ ] Solo build
   - [ ] Solo test
   - [ ] Nessuna automazione

**Risposte**: 1-a, 2-b, 3-a, 4-a, 5-a

---

## 🔗 Risorse

- [Project on GitHub](https://github.com/example/socialhub)
- [React Native Docs](https://reactnative.dev)
- [Redux Toolkit](https://redux-toolkit.js.org)
- [Socket.io](https://socket.io)

---

## ✅ Checklist

- [ ] Project setup completo
- [ ] Architecture definita
- [ ] Authentication implementata
- [ ] User profiles funzionanti
- [ ] Posts feed con infinite scroll
- [ ] Real-time chat attivo
- [ ] Push notifications configurate
- [ ] Image upload funzionante
- [ ] Geolocation integrata
- [ ] Offline support implementato
- [ ] Tests scritti (Jest + Detox)
- [ ] CI/CD configurato
- [ ] App deployata su stores

---

## 📌 Punti Chiave

1. 🏗️ **Clean Architecture** mantiene codice organizzato
2. 🔐 **JWT Auth** con AsyncStorage persistence
3. 📱 **Redux Toolkit** per state management
4. 📡 **WebSockets** per real-time features
5. 🔔 **Push Notifications** con Firebase
6. 📷 **Image Picker** camera e gallery
7. 📍 **Geolocation** per location features
8. 💾 **Offline-first** con queue middleware
9. 🧪 **Testing** Jest + Detox coverage
10. 🚀 **CI/CD** automated deployment

---

**Complimenti! 🎉** Hai completato SocialHub e padroneggi React Native! Ora sei pronto per build production apps!

[← Precedente: New Architecture](24_New_Architecture.md) | [Torna all'Indice](../README.md)
