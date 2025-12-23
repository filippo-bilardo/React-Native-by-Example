# Capitolo 7: Navigazione tra Schermate

[← Precedente: Styling e Layout](06_Styling_Layout.md) | [Torna all'Indice](../README.md) | [Prossimo: Liste e Performance →](08_Liste_Performance.md)

---

## Descrizione

La navigazione è uno degli aspetti più importanti di un'app mobile. React Navigation è la libreria standard per gestire la navigazione in React Native, supportando vari pattern come Stack, Tab e Drawer navigation.

In questo capitolo imparerai a configurare React Navigation, implementare diversi tipi di navigatori, passare parametri tra schermate, gestire deep linking e creare esperienze di navigazione fluide e intuitive.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Installare e configurare React Navigation
- [ ] Implementare Stack Navigator
- [ ] Creare Tab Navigator (Bottom e Top)
- [ ] Implementare Drawer Navigator
- [ ] Passare parametri tra schermate
- [ ] Gestire navigation state e lifecycle
- [ ] Implementare deep linking
- [ ] Customizzare headers e transitions
- [ ] Gestire authentication flow
- [ ] Ottimizzare performance di navigazione

## 📚 Argomenti Teorici

1. [Installazione React Navigation](#1-installazione)
2. [Stack Navigator](#2-stack-navigator)
3. [Tab Navigator](#3-tab-navigator)
4. [Drawer Navigator](#4-drawer-navigator)
5. [Passare Parametri](#5-parametri)
6. [Nested Navigation](#6-nested-navigation)
7. [Authentication Flow](#7-authentication-flow)
8. [Deep Linking](#8-deep-linking)
9. [Customizzazione](#9-customizzazione)
10. [Best Practices](#10-best-practices)

---

## 1. Installazione

### Pacchetti Base

```bash
# Core
npm install @react-navigation/native

# Dependencies
npm install react-native-screens react-native-safe-area-context

# Per iOS
cd ios && pod install && cd ..
```

### Navigatori Specifici

```bash
# Stack
npm install @react-navigation/native-stack

# Bottom Tabs
npm install @react-navigation/bottom-tabs

# Drawer
npm install @react-navigation/drawer
npm install react-native-gesture-handler react-native-reanimated

# Material Top Tabs
npm install @react-navigation/material-top-tabs react-native-tab-view
```

### Setup App.tsx

```typescript
import { NavigationContainer } from '@react-navigation/native';

const App = () => {
  return (
    <NavigationContainer>
      {/* Navigators qui */}
    </NavigationContainer>
  );
};

export default App;
```

---

## 2. Stack Navigator

### Setup Base

```typescript
import { createNativeStackNavigator } from '@react-navigation/native-stack';

type RootStackParamList = {
  Home: undefined;
  Details: { id: number };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: { backgroundColor: '#007AFF' },
          headerTintColor: '#fff',
          headerTitleStyle: { fontWeight: 'bold' }
        }}
      >
        <Stack.Screen 
          name="Home" 
          component={HomeScreen}
          options={{ title: 'Home' }}
        />
        <Stack.Screen 
          name="Details" 
          component={DetailsScreen}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

### Screen Components

```typescript
import { NativeStackScreenProps } from '@react-navigation/native-stack';

type Props = NativeStackScreenProps<RootStackParamList, 'Home'>;

const HomeScreen = ({ navigation }: Props) => {
  return (
    <View style={styles.container}>
      <Text>Home Screen</Text>
      
      <Button
        title="Go to Details"
        onPress={() => navigation.navigate('Details', { id: 42 })}
      />
    </View>
  );
};

type DetailsProps = NativeStackScreenProps<RootStackParamList, 'Details'>;

const DetailsScreen = ({ route, navigation }: DetailsProps) => {
  const { id } = route.params;
  
  return (
    <View style={styles.container}>
      <Text>Details Screen</Text>
      <Text>ID: {id}</Text>
      
      <Button
        title="Go Back"
        onPress={() => navigation.goBack()}
      />
    </View>
  );
};
```

### Navigation Methods

```typescript
// Navigate to screen
navigation.navigate('Details', { id: 1 });

// Go back
navigation.goBack();

// Go to top of stack
navigation.popToTop();

// Replace current screen
navigation.replace('Login');

// Push (always add new screen, anche se esiste)
navigation.push('Details', { id: 2 });

// Navigate and reset stack
navigation.reset({
  index: 0,
  routes: [{ name: 'Home' }],
});
```

### Screen Options

```typescript
<Stack.Screen
  name="Details"
  component={DetailsScreen}
  options={{
    title: 'Details',
    headerShown: true,
    headerBackTitle: 'Back',
    headerBackTitleVisible: true,
    headerStyle: { backgroundColor: '#007AFF' },
    headerTintColor: '#fff',
    headerRight: () => (
      <Button title="Save" onPress={() => {}} />
    ),
    gestureEnabled: true,
    animation: 'slide_from_right', // iOS
    presentation: 'modal', // iOS
  }}
/>

// Dynamic options
<Stack.Screen
  name="Details"
  component={DetailsScreen}
  options={({ route }) => ({
    title: `Details #${route.params.id}`
  })}
/>
```

### Update Options da Screen

```typescript
const DetailsScreen = ({ navigation }) => {
  useEffect(() => {
    navigation.setOptions({
      title: 'Updated Title',
      headerRight: () => <Button title="Save" onPress={save} />
    });
  }, []);
  
  return <View>{/* ... */}</View>;
};
```

---

## 3. Tab Navigator

### Bottom Tabs

```typescript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import Ionicons from 'react-native-vector-icons/Ionicons';

type TabParamList = {
  Home: undefined;
  Search: undefined;
  Profile: undefined;
};

const Tab = createBottomTabNavigator<TabParamList>();

const App = () => {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={({ route }) => ({
          tabBarIcon: ({ focused, color, size }) => {
            let iconName: string;
            
            if (route.name === 'Home') {
              iconName = focused ? 'home' : 'home-outline';
            } else if (route.name === 'Search') {
              iconName = focused ? 'search' : 'search-outline';
            } else {
              iconName = focused ? 'person' : 'person-outline';
            }
            
            return <Ionicons name={iconName} size={size} color={color} />;
          },
          tabBarActiveTintColor: '#007AFF',
          tabBarInactiveTintColor: 'gray',
          headerShown: false
        })}
      >
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen 
          name="Search" 
          component={SearchScreen}
          options={{
            tabBarBadge: 3  // Notification badge
          }}
        />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
};
```

### Material Top Tabs

```typescript
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tabs';

const Tab = createMaterialTopTabNavigator();

const App = () => {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
        tabBarIndicatorStyle: { backgroundColor: '#007AFF' },
        tabBarStyle: { backgroundColor: 'white' },
        tabBarLabelStyle: { fontSize: 14, fontWeight: 'bold' },
        swipeEnabled: true
      }}
    >
      <Tab.Screen name="Recent" component={RecentScreen} />
      <Tab.Screen name="Popular" component={PopularScreen} />
      <Tab.Screen name="Trending" component={TrendingScreen} />
    </Tab.Navigator>
  );
};
```

---

## 4. Drawer Navigator

### Setup Drawer

```typescript
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Drawer.Navigator
        screenOptions={{
          drawerActiveTintColor: '#007AFF',
          drawerInactiveTintColor: 'gray',
          drawerStyle: {
            backgroundColor: '#fff',
            width: 280
          },
          headerShown: true
        }}
      >
        <Drawer.Screen 
          name="Home" 
          component={HomeScreen}
          options={{
            drawerIcon: ({ color, size }) => (
              <Ionicons name="home" size={size} color={color} />
            ),
            drawerLabel: 'Home'
          }}
        />
        <Drawer.Screen 
          name="Profile" 
          component={ProfileScreen}
          options={{
            drawerIcon: ({ color, size }) => (
              <Ionicons name="person" size={size} color={color} />
            )
          }}
        />
      </Drawer.Navigator>
    </NavigationContainer>
  );
};
```

### Custom Drawer Content

```typescript
import { DrawerContentScrollView, DrawerItemList } from '@react-navigation/drawer';

const CustomDrawerContent = (props: any) => {
  return (
    <DrawerContentScrollView {...props}>
      <View style={styles.header}>
        <Image source={require('./avatar.png')} style={styles.avatar} />
        <Text style={styles.name}>John Doe</Text>
        <Text style={styles.email}>john@example.com</Text>
      </View>
      
      <DrawerItemList {...props} />
      
      <View style={styles.footer}>
        <Button title="Logout" onPress={() => {}} />
      </View>
    </DrawerContentScrollView>
  );
};

<Drawer.Navigator
  drawerContent={props => <CustomDrawerContent {...props} />}
>
```

### Open/Close Drawer

```typescript
const HomeScreen = ({ navigation }) => {
  return (
    <View>
      <Button
        title="Open Drawer"
        onPress={() => navigation.openDrawer()}
      />
      <Button
        title="Toggle Drawer"
        onPress={() => navigation.toggleDrawer()}
      />
    </View>
  );
};
```

---

## 5. Parametri

### Tipizzazione Parametri

```typescript
type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Post: { postId: number; title?: string };
};

// Uso in screen
type Props = NativeStackScreenProps<RootStackParamList, 'Profile'>;

const ProfileScreen = ({ route, navigation }: Props) => {
  const { userId } = route.params;  // TypeScript sa che esiste!
};
```

### Passare Parametri

```typescript
// Navigate con params
navigation.navigate('Profile', { userId: '123' });

// Update params
navigation.setParams({ userId: '456' });

// Merge params (non sovrascrive tutto)
navigation.setParams({ title: 'Updated' });
```

### Default Params

```typescript
<Stack.Screen
  name="Profile"
  component={ProfileScreen}
  initialParams={{ userId: 'guest' }}
/>
```

### Leggere Parametri

```typescript
const ProfileScreen = ({ route }) => {
  // Option 1: Destructure
  const { userId } = route.params;
  
  // Option 2: Direct access
  const userId = route.params?.userId;
  
  // Option 3: Con default
  const userId = route.params?.userId ?? 'guest';
  
  return <Text>User: {userId}</Text>;
};
```

### Callback Parameters

```typescript
// Screen A
const ScreenA = ({ navigation }) => {
  const [selected, setSelected] = useState('');
  
  return (
    <View>
      <Text>Selected: {selected}</Text>
      <Button
        title="Select Item"
        onPress={() => {
          navigation.navigate('ScreenB', {
            onSelect: (item: string) => setSelected(item)
          });
        }}
      />
    </View>
  );
};

// Screen B
const ScreenB = ({ route, navigation }) => {
  const { onSelect } = route.params;
  
  const handleSelect = (item: string) => {
    onSelect(item);
    navigation.goBack();
  };
  
  return (
    <View>
      <Button title="Select Item 1" onPress={() => handleSelect('Item 1')} />
      <Button title="Select Item 2" onPress={() => handleSelect('Item 2')} />
    </View>
  );
};
```

---

## 6. Nested Navigation

### Stack dentro Tab

```typescript
const HomeStack = () => {
  return (
    <Stack.Navigator>
      <Stack.Screen name="HomeList" component={HomeListScreen} />
      <Stack.Screen name="HomeDetails" component={HomeDetailsScreen} />
    </Stack.Navigator>
  );
};

const ProfileStack = () => {
  return (
    <Stack.Navigator>
      <Stack.Screen name="ProfileMain" component={ProfileScreen} />
      <Stack.Screen name="Settings" component={SettingsScreen} />
    </Stack.Navigator>
  );
};

const App = () => {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeStack} />
        <Tab.Screen name="Profile" component={ProfileStack} />
      </Tab.Navigator>
    </NavigationContainer>
  );
};
```

### Navigate in Nested Navigators

```typescript
// Da HomeListScreen, vai a ProfileMain
navigation.navigate('Profile', {
  screen: 'ProfileMain',
  params: { userId: '123' }
});

// Navigate deep
navigation.navigate('Profile', {
  screen: 'Settings',
  params: {
    screen: 'Privacy',
    params: { option: 'notifications' }
  }
});
```

---

## 7. Authentication Flow

### Conditional Navigation

```typescript
import { useAuth } from './AuthContext';

const App = () => {
  const { user, loading } = useAuth();
  
  if (loading) {
    return <SplashScreen />;
  }
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        {user ? (
          // Authenticated
          <>
            <Stack.Screen name="Home" component={HomeScreen} />
            <Stack.Screen name="Profile" component={ProfileScreen} />
          </>
        ) : (
          // Not authenticated
          <>
            <Stack.Screen 
              name="Login" 
              component={LoginScreen}
              options={{ headerShown: false }}
            />
            <Stack.Screen name="Register" component={RegisterScreen} />
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

### Auth Context

```typescript
import { createContext, useContext, useState, useEffect } from 'react';

interface AuthContextType {
  user: User | null;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check persisted auth
    checkAuth().finally(() => setLoading(false));
  }, []);
  
  const login = async (email: string, password: string) => {
    const userData = await api.login(email, password);
    setUser(userData);
  };
  
  const logout = async () => {
    await api.logout();
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be within AuthProvider');
  return context;
};
```

---

## 8. Deep Linking

### Configuration

```typescript
// linking.ts
const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: 'home',
      Profile: 'profile/:userId',
      Post: {
        path: 'post/:postId',
        parse: {
          postId: (postId: string) => parseInt(postId)
        }
      }
    }
  }
};

// App.tsx
<NavigationContainer linking={linking}>
```

### URL Examples

```
myapp://home
myapp://profile/123
myapp://post/456

https://myapp.com/home
https://myapp.com/profile/123
```

### iOS Setup (ios/myapp/Info.plist)

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### Android Setup (android/app/src/main/AndroidManifest.xml)

```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" />
  <data android:host="myapp.com" />
</intent-filter>
```

---

## 9. Customizzazione

### Custom Header

```typescript
<Stack.Screen
  name="Home"
  component={HomeScreen}
  options={{
    header: ({ navigation, route, options }) => (
      <View style={styles.customHeader}>
        <TouchableOpacity onPress={() => navigation.goBack()}>
          <Text>Back</Text>
        </TouchableOpacity>
        <Text style={styles.title}>{options.title}</Text>
        <TouchableOpacity onPress={() => {}}>
          <Text>Save</Text>
        </TouchableOpacity>
      </View>
    )
  }}
/>
```

### Transition Animations

```typescript
// iOS-style (default on iOS)
<Stack.Navigator
  screenOptions={{
    animation: 'slide_from_right'
  }}
>

// Android-style
<Stack.Navigator
  screenOptions={{
    animation: 'fade_from_bottom'
  }}
>

// Altri
// 'default', 'fade', 'slide_from_bottom', 'slide_from_right', 'slide_from_left', 'none'
```

### Custom Transitions (Animated)

```typescript
import { Animated } from 'react-native';

const forFade = ({ current }: any) => ({
  cardStyle: {
    opacity: current.progress
  }
});

<Stack.Navigator
  screenOptions={{
    cardStyleInterpolator: forFade
  }}
>
```

---

## 10. Best Practices

### 1. Tipizzazione Forte

```typescript
// types.ts
export type RootStackParamList = {
  Home: undefined;
  Details: { id: number };
};

export type RootStackScreenProps<T extends keyof RootStackParamList> =
  NativeStackScreenProps<RootStackParamList, T>;

// Screen
import { RootStackScreenProps } from './types';

const DetailsScreen = ({ route }: RootStackScreenProps<'Details'>) => {
  const { id } = route.params;  // Typed!
};
```

### 2. Navigation Ref

```typescript
// navigationRef.ts
import { createNavigationContainerRef } from '@react-navigation/native';

export const navigationRef = createNavigationContainerRef();

export function navigate(name: string, params?: any) {
  if (navigationRef.isReady()) {
    navigationRef.navigate(name as never, params as never);
  }
}

// App.tsx
import { navigationRef } from './navigationRef';

<NavigationContainer ref={navigationRef}>

// Uso ovunque (anche fuori componenti)
import * as RootNavigation from './navigationRef';

RootNavigation.navigate('Home');
```

### 3. Screen Organization

```
src/
  navigation/
    RootNavigator.tsx
    HomeStack.tsx
    ProfileStack.tsx
    types.ts
  screens/
    Home/
      HomeScreen.tsx
      HomeDetailsScreen.tsx
    Profile/
      ProfileScreen.tsx
      SettingsScreen.tsx
```

### 4. Performance

```typescript
// Lazy load screens
const DetailsScreen = React.lazy(() => import('./DetailsScreen'));

<Stack.Screen name="Details">
  {props => (
    <Suspense fallback={<LoadingScreen />}>
      <DetailsScreen {...props} />
    </Suspense>
  )}
</Stack.Screen>

// Prevent re-renders
<Stack.Screen
  name="Home"
  component={React.memo(HomeScreen)}
/>
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Basic Navigation

Crea un'app con:
- Stack Navigator con 3 schermate
- Home → List → Details
- Passa ID da List a Details
- Go back button

### Esercizio 2: 🟡 Tab Navigation

Implementa:
- Bottom Tabs con 4 tab
- Icone per ogni tab
- Badge notifications su una tab
- Ogni tab ha il suo Stack Navigator

### Esercizio 3: 🟡 Auth Flow

Crea authentication flow:
- Login/Register screens
- Main app screens (dopo login)
- Context per auth state
- Persist auth con AsyncStorage
- Splash screen durante check

### Esercizio 4: 🔴 Complex Navigation

App completa con:
- Drawer Navigator principale
- Bottom Tabs dentro Drawer
- Stack per ogni tab
- Deep linking configurato
- Almeno 8 schermate

### Esercizio 5: 🔴 Custom Navigation

Implementa:
- Custom header component
- Custom transitions
- Modal stack per alcune schermate
- Navigation guard (check permissions)
- Navigation analytics tracking

---

## 📝 Quiz

1. **Differenza navigate() vs push()?**
   - [ ] Non c'è differenza
   - [x] push() aggiunge sempre, navigate() va a screen esistente
   - [ ] navigate() è deprecato
   - [ ] push() è solo per Android

2. **Come tornare al top dello stack?**
   - [ ] navigation.goBack()
   - [x] navigation.popToTop()
   - [ ] navigation.reset()
   - [ ] Non è possibile

3. **Come passare callback come parametro?**
   - [ ] Non è possibile
   - [ ] Solo con Redux
   - [x] route.params.callback()
   - [ ] Serve libreria esterna

4. **Deep linking richiede:**
   - [ ] Solo codice JavaScript
   - [x] Configurazione nativa iOS e Android
   - [ ] Solo per web
   - [ ] Non è supportato

5. **Nested navigation: navigate da tab A a tab B?**
   - [ ] Non è possibile
   - [x] navigation.navigate('TabB', { screen: 'Screen' })
   - [ ] Solo con Redux
   - [ ] Serve reset()

**Risposte**: 1-b, 2-b, 3-c, 4-b, 5-b

---

## 🔗 Risorse

- [React Navigation Docs](https://reactnavigation.org/docs/getting-started)
- [TypeScript Guide](https://reactnavigation.org/docs/typescript)
- [Deep Linking Guide](https://reactnavigation.org/docs/deep-linking)

---

## ✅ Checklist

- [ ] React Navigation installato
- [ ] Stack Navigator implementato
- [ ] Tab Navigator creato
- [ ] Parametri passati tra schermate
- [ ] Nested navigation gestita
- [ ] Auth flow implementato
- [ ] Deep linking configurato (optional)
- [ ] TypeScript typing corretto
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🧭 **React Navigation** è lo standard
2. 📚 **Stack** per navigation gerarchica
3. 📑 **Tabs** per sezioni parallele
4. 🎯 **TypeScript** per type-safe navigation
5. 🔗 **Params** tipizzati con ParamList
6. 🔐 **Auth flow** con conditional rendering
7. 🌐 **Deep linking** per URL esterni
8. 🎨 **Customizzazione** header e transitions

---

**Eccellente! 🎉** Ora sai navigare tra schermate come un pro. Prossimo: Liste e Performance!

[← Precedente: Styling e Layout](06_Styling_Layout.md) | [Torna all'Indice](../README.md) | [Prossimo: Liste e Performance →](08_Liste_Performance.md)
