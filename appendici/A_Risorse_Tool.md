# Appendice A: Risorse e Tool Utili

Questa appendice raccoglie le risorse più utili per lo sviluppo React Native: librerie essenziali, tool di sviluppo, documentazione ufficiale, community e risorse di apprendimento. Ogni risorsa è categorizzata per facilitare la ricerca.

---

## 📚 Documentazione Ufficiale

### React Native
- **[React Native Docs](https://reactnative.dev/)** - Documentazione ufficiale completa
- **[React Native Blog](https://reactnative.dev/blog)** - Annunci e aggiornamenti
- **[React Native GitHub](https://github.com/facebook/react-native)** - Repository ufficiale
- **[React Native Community](https://github.com/react-native-community)** - Librerie mantenute dalla community

### React
- **[React Docs](https://react.dev/)** - Documentazione React 18+ con hooks
- **[React DevTools](https://react.dev/learn/react-developer-tools)** - Strumenti di debugging

### Expo
- **[Expo Docs](https://docs.expo.dev/)** - Documentazione completa Expo SDK
- **[Expo API Reference](https://docs.expo.dev/versions/latest/)** - Riferimento API dettagliato
- **[EAS Docs](https://docs.expo.dev/eas/)** - Expo Application Services
- **[Expo Snack](https://snack.expo.dev/)** - Playground online per testare codice

### TypeScript
- **[TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)** - Guida completa TypeScript
- **[React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)** - Pattern comuni React+TS

---

## 🛠️ Tool di Sviluppo Essenziali

### Editor e IDE

#### Visual Studio Code (Consigliato)
**Estensioni fondamentali:**
- **[React Native Tools](https://marketplace.visualstudio.com/items?itemName=msjsdiag.vscode-react-native)** - Debugging e IntelliSense
- **[Expo Tools](https://marketplace.visualstudio.com/items?itemName=expo.vscode-expo-tools)** - Supporto completo Expo
- **[ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)** - Linting JavaScript/TypeScript
- **[Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)** - Formattazione codice automatica
- **[Error Lens](https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens)** - Visualizza errori inline
- **[GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)** - Potenzia Git in VS Code
- **[Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag)** - Rinomina tag JSX automaticamente
- **[Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)** - Autocompletamento percorsi
- **[Better Comments](https://marketplace.visualstudio.com/items?itemName=aaron-bond.better-comments)** - Commenti colorati per categoria

**Configurazione VS Code consigliata (settings.json):**
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "javascript.updateImportsOnFileMove.enabled": "always",
  "typescript.updateImportsOnFileMove.enabled": "always",
  "emmet.includeLanguages": {
    "javascript": "javascriptreact"
  }
}
```

#### Altri Editor
- **[Android Studio](https://developer.android.com/studio)** - IDE ufficiale Android (include emulatore)
- **[Xcode](https://developer.apple.com/xcode/)** - IDE ufficiale iOS (solo macOS)
- **[WebStorm](https://www.jetbrains.com/webstorm/)** - IDE JetBrains a pagamento con supporto completo

### Debugging e Testing

#### Debugging
- **[Flipper](https://fbflipper.com/)** - Platform debugger per React Native (network, logs, layout)
- **[React DevTools](https://www.npmjs.com/package/react-devtools)** - Inspect component tree
- **[Reactotron](https://github.com/infinitered/reactotron)** - Desktop app per debugging (Redux, state, API calls)

#### Testing
- **[Jest](https://jestjs.io/)** - Framework di testing (incluso di default)
- **[React Native Testing Library](https://callstack.github.io/react-native-testing-library/)** - Testing componenti
- **[Detox](https://wix.github.io/Detox/)** - End-to-end testing gray box
- **[Maestro](https://maestro.mobile.dev/)** - E2E testing semplice e veloce

### Performance e Profiling
- **[React Native Performance Monitor](https://reactnative.dev/docs/performance)** - FPS counter built-in
- **[why-did-you-render](https://github.com/welldone-software/why-did-you-render)** - Trova re-render inutili
- **[Xcode Instruments](https://developer.apple.com/instruments/)** - Profiling iOS avanzato
- **[Android Studio Profiler](https://developer.android.com/studio/profile)** - Profiling Android (CPU, memoria, rete)

### Build e Deployment
- **[Fastlane](https://fastlane.tools/)** - Automazione build e deploy iOS/Android
- **[EAS Build](https://docs.expo.dev/build/introduction/)** - Build cloud per Expo
- **[App Center](https://appcenter.ms/)** - CI/CD, distribuzione beta, crash analytics
- **[Bitrise](https://www.bitrise.io/)** - CI/CD specializzato mobile

---

## 📦 Librerie Essenziali

### Navigazione

#### React Navigation (⭐ Più Popolare)
```bash
npm install @react-navigation/native @react-navigation/native-stack
npm install react-native-screens react-native-safe-area-context
```
- **[Documentazione](https://reactnavigation.org/)** - Navigation completa e flessibile
- **[Stack Navigator](https://reactnavigation.org/docs/stack-navigator)** - Navigazione stack
- **[Tab Navigator](https://reactnavigation.org/docs/bottom-tab-navigator)** - Tab navigation
- **[Drawer Navigator](https://reactnavigation.org/docs/drawer-navigator)** - Menu laterale

#### Expo Router (File-based, Modern)
```bash
npx expo install expo-router react-native-safe-area-context react-native-screens
```
- **[Documentazione](https://docs.expo.dev/router/introduction/)** - File-based routing (come Next.js)
- Pro: Convenzioni chiare, type-safe, deep linking automatico
- Contro: Solo Expo, community più piccola

#### React Native Navigation (Nativa)
```bash
npm install react-native-navigation
```
- **[Documentazione](https://wix.github.io/react-native-navigation/)** - Navigation 100% nativa
- Pro: Performance eccellenti, animazioni native
- Contro: Setup complesso, curve di apprendimento ripida

### State Management

#### Context API + Hooks (Built-in)
- **[React Context](https://react.dev/reference/react/useContext)** - State globale integrato
- Pro: Zero dipendenze, semplice per app piccole
- Contro: Performance issues con state complessi

#### Zustand (⭐ Consigliato per semplicità)
```bash
npm install zustand
```
- **[Documentazione](https://github.com/pmndrs/zustand)** - State management minimale
- Pro: API semplicissima, TypeScript ottimo, middleware per persist
- Contro: Meno strutturato di Redux per app enormi

**Esempio Zustand con persist:**
```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  clearCart: () => void;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set) => ({
      items: [],
      addItem: (item) => set((state) => ({ 
        items: [...state.items, item] 
      })),
      removeItem: (id) => set((state) => ({ 
        items: state.items.filter(i => i.id !== id) 
      })),
      clearCart: () => set({ items: [] }),
    }),
    {
      name: 'cart-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

#### Redux Toolkit (Enterprise)
```bash
npm install @reduxjs/toolkit react-redux
```
- **[Documentazione](https://redux-toolkit.js.org/)** - State management enterprise
- **[RTK Query](https://redux-toolkit.js.org/rtk-query/overview)** - Data fetching e caching
- Pro: Architettura scalabile, DevTools eccellenti, ecosystem enorme
- Contro: Boilerplate, curve di apprendimento

#### Jotai (Atoms, come Recoil)
```bash
npm install jotai
```
- **[Documentazione](https://jotai.org/)** - Atomic state management
- Pro: Granularità fine, performance ottimali, TypeScript first
- Contro: Paradigma diverso da store classici

### Data Fetching e Caching

#### TanStack Query (React Query) (⭐ Consigliato)
```bash
npm install @tanstack/react-query
```
- **[Documentazione](https://tanstack.com/query/latest)** - Server state management
- Pro: Cache automatico, refetch, optimistic updates, infinite queries
- Contro: Overhead per chiamate semplici

**Esempio con cache e refetch:**
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query con cache automatico
function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: async () => {
      const res = await fetch('https://api.example.com/products');
      return res.json();
    },
    staleTime: 1000 * 60 * 5, // Cache valido per 5 minuti
    gcTime: 1000 * 60 * 10, // Garbage collect dopo 10 minuti
  });
}

// Mutation con invalidazione automatica
function useCreateProduct() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (product: Product) => 
      fetch('https://api.example.com/products', {
        method: 'POST',
        body: JSON.stringify(product),
      }),
    onSuccess: () => {
      // Invalida la cache per forzare refetch
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}
```

#### SWR (Vercel)
```bash
npm install swr
```
- **[Documentazione](https://swr.vercel.app/)** - Data fetching con cache
- Pro: Semplicissimo, validazione automatica, mutation semplice
- Contro: Meno feature di React Query

#### Apollo Client (GraphQL)
```bash
npm install @apollo/client graphql
```
- **[Documentazione](https://www.apollographql.com/docs/react/)** - Client GraphQL completo

### Networking

#### Axios (HTTP Client Popolare)
```bash
npm install axios
```
- **[Documentazione](https://axios-http.com/)** - HTTP client feature-rich
- Pro: Interceptors, auto JSON parsing, cancel requests, progress tracking

**Setup con interceptors per auth:**
```typescript
import axios from 'axios';
import AsyncStorage from '@react-native-async-storage/async-storage';

const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor per aggiungere token
api.interceptors.request.use(
  async (config) => {
    const token = await AsyncStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor per gestire errori globalmente
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Token scaduto, logout
      await AsyncStorage.removeItem('authToken');
      // Naviga a login (usa navigation ref)
    }
    return Promise.reject(error);
  }
);

export default api;
```

#### Fetch API (Built-in)
- **[Documentazione MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)** - Nativa browser
- Pro: Zero dipendenze, moderna (promises, async/await)
- Contro: Meno feature di axios, no interceptors built-in

### UI Component Libraries

#### React Native Paper (Material Design)
```bash
npm install react-native-paper
```
- **[Documentazione](https://reactnativepaper.com/)** - Material Design components
- Pro: Completa, accessibile, theming potente, animazioni smooth
- Contro: Style opinionato (Material)

#### NativeBase
```bash
npm install native-base
```
- **[Documentazione](https://nativebase.io/)** - Cross-platform component library
- Pro: 100+ componenti, dark mode, responsive, accessibility

#### React Native Elements
```bash
npm install @rneui/themed @rneui/base
```
- **[Documentazione](https://reactnativeelements.com/)** - Componenti altamente customizzabili

#### Tamagui (⭐ Performance)
```bash
npm install tamagui @tamagui/config
```
- **[Documentazione](https://tamagui.dev/)** - UI kit ottimizzato per performance
- Pro: Compile-time optimization, cross-platform (web included), style props

### Form Management

#### React Hook Form (⭐ Consigliato)
```bash
npm install react-hook-form
```
- **[Documentazione](https://react-hook-form.com/)** - Form performanti con validazione
- Pro: Minimal re-renders, integrazione Yup/Zod, DevTools
- Contro: Curve di apprendimento iniziale

**Esempio completo con validazione:**
```typescript
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Email non valida'),
  password: z.string().min(8, 'Minimo 8 caratteri'),
});

type LoginForm = z.infer<typeof loginSchema>;

function LoginScreen() {
  const { control, handleSubmit, formState: { errors } } = useForm<LoginForm>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  });

  const onSubmit = (data: LoginForm) => {
    console.log('Login:', data);
  };

  return (
    <View>
      <Controller
        control={control}
        name="email"
        render={({ field: { onChange, onBlur, value } }) => (
          <>
            <TextInput
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              keyboardType="email-address"
              autoCapitalize="none"
            />
            {errors.email && <Text>{errors.email.message}</Text>}
          </>
        )}
      />
      
      <Controller
        control={control}
        name="password"
        render={({ field: { onChange, onBlur, value } }) => (
          <>
            <TextInput
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              secureTextEntry
            />
            {errors.password && <Text>{errors.password.message}</Text>}
          </>
        )}
      />

      <Button title="Login" onPress={handleSubmit(onSubmit)} />
    </View>
  );
}
```

#### Formik
```bash
npm install formik
```
- **[Documentazione](https://formik.org/)** - Form library completa
- Pro: Molto popolare, integrazione Yup
- Contro: Più re-renders di React Hook Form

#### Yup (Validazione)
```bash
npm install yup
```
- **[Documentazione](https://github.com/jquense/yup)** - Schema validation
- Pro: API intuitiva, error messages customizzabili, async validation

#### Zod (TypeScript-first)
```bash
npm install zod
```
- **[Documentazione](https://zod.dev/)** - TypeScript schema validation
- Pro: Type inference automatico, runtime validation, zero dependencies

### Animazioni

#### React Native Reanimated (⭐ Più Potente)
```bash
npx expo install react-native-reanimated
```
- **[Documentazione](https://docs.swmansion.com/react-native-reanimated/)** - Animazioni UI thread
- Pro: Performance native (UI thread), layout animations, gesture integration
- Contro: API complessa, worklet syntax

**Esempio animazione fade in:**
```typescript
import Animated, { 
  useSharedValue, 
  useAnimatedStyle, 
  withTiming,
  withSpring 
} from 'react-native-reanimated';

function FadeInBox() {
  const opacity = useSharedValue(0);
  const scale = useSharedValue(0.8);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ scale: scale.value }],
  }));

  useEffect(() => {
    opacity.value = withTiming(1, { duration: 500 });
    scale.value = withSpring(1);
  }, []);

  return (
    <Animated.View style={[styles.box, animatedStyle]}>
      <Text>Animated!</Text>
    </Animated.View>
  );
}
```

#### React Native Gesture Handler
```bash
npx expo install react-native-gesture-handler
```
- **[Documentazione](https://docs.swmansion.com/react-native-gesture-handler/)** - Gesture system avanzato
- Pro: Performance native, gesture composer, cross-platform

**Esempio pan gesture:**
```typescript
import { GestureDetector, Gesture } from 'react-native-gesture-handler';
import Animated, { useSharedValue, useAnimatedStyle } from 'react-native-reanimated';

function DraggableBox() {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);

  const pan = Gesture.Pan()
    .onChange((event) => {
      translateX.value += event.changeX;
      translateY.value += event.changeY;
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { translateY: translateY.value },
    ],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.box, animatedStyle]} />
    </GestureDetector>
  );
}
```

#### Lottie (Animazioni After Effects)
```bash
npx expo install lottie-react-native
```
- **[Documentazione](https://github.com/lottie-react-native/lottie-react-native)** - Animazioni vettoriali
- Pro: Animazioni designer-made, file piccoli (JSON), cross-platform
- **[LottieFiles](https://lottiefiles.com/)** - Marketplace animazioni gratuite

### Storage Locale

#### AsyncStorage (Key-Value Semplice)
```bash
npx expo install @react-native-async-storage/async-storage
```
- **[Documentazione](https://react-native-async-storage.github.io/async-storage/)** - Storage key-value
- Pro: Semplice, piccoli dati (preferences, settings)
- Contro: Non per grandi quantità di dati

#### Expo SecureStore (Dati Sensibili)
```bash
npx expo install expo-secure-store
```
- **[Documentazione](https://docs.expo.dev/versions/latest/sdk/securestore/)** - Keychain/Keystore encryption
- Pro: Encryption automatico, sicuro per token/password

#### MMKV (⭐ Performance)
```bash
npm install react-native-mmkv
```
- **[Documentazione](https://github.com/mrousavy/react-native-mmkv)** - Storage ultra-veloce
- Pro: 30x più veloce di AsyncStorage, sync API, encryption

**Esempio MMKV:**
```typescript
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

// Salva
storage.set('user.name', 'John Doe');
storage.set('user.age', 30);
storage.set('user.isPremium', true);

// Leggi
const name = storage.getString('user.name'); // "John Doe"
const age = storage.getNumber('user.age'); // 30
const isPremium = storage.getBoolean('user.isPremium'); // true

// Listener per cambiamenti
storage.addOnValueChangedListener((key) => {
  console.log(`Value changed: ${key}`);
});
```

#### WatermelonDB (Database Reattivo)
```bash
npm install @nozbe/watermelondb @nozbe/with-observables
```
- **[Documentazione](https://nozbe.github.io/WatermelonDB/)** - Database ottimizzato mobile
- Pro: Performance eccellenti, reactive, sync con backend, relations

#### Realm (Database NoSQL)
```bash
npm install realm
```
- **[Documentazione](https://www.mongodb.com/docs/realm/)** - Database object-oriented
- Pro: Sync automatico MongoDB Atlas, queries potenti, encryption

### Media e Camera

#### Expo Image Picker
```bash
npx expo install expo-image-picker
```
- **[Documentazione](https://docs.expo.dev/versions/latest/sdk/imagepicker/)** - Selezione foto/video
- Pro: Crop built-in, permissions automatici, multi-select

#### Expo Camera
```bash
npx expo install expo-camera
```
- **[Documentazione](https://docs.expo.dev/versions/latest/sdk/camera/)** - Fotocamera avanzata
- Pro: Barcode scanner, face detection, video recording

#### React Native Image Crop Picker
```bash
npm install react-native-image-crop-picker
```
- **[Documentazione](https://github.com/ivpusic/react-native-image-crop-picker)** - Crop e compressione
- Pro: Crop UI nativa, compressione, crop circolare

#### Fast Image (Ottimizzata)
```bash
npm install react-native-fast-image
```
- **[Documentazione](https://github.com/DylanVann/react-native-fast-image)** - Image component performante
- Pro: Cache automatico, priority loading, preload

### Mappe e Geolocalizzazione

#### React Native Maps
```bash
npm install react-native-maps
```
- **[Documentazione](https://github.com/react-native-maps/react-native-maps)** - Mappe native
- Pro: Google Maps + Apple Maps, marker custom, heatmaps, clustering

#### Expo Location
```bash
npx expo install expo-location
```
- **[Documentazione](https://docs.expo.dev/versions/latest/sdk/location/)** - Geolocalizzazione
- Pro: Background location, geofencing, accuracy control

### Notifiche

#### Expo Notifications
```bash
npx expo install expo-notifications expo-device
```
- **[Documentazione](https://docs.expo.dev/versions/latest/sdk/notifications/)** - Notifiche locali e push
- Pro: Setup semplice, scheduling, badge management

#### React Native Firebase (FCM)
```bash
npm install @react-native-firebase/app @react-native-firebase/messaging
```
- **[Documentazione](https://rnfirebase.io/)** - Firebase Cloud Messaging
- Pro: Topic subscription, data messages, analytics

### Pagamenti

#### Stripe React Native
```bash
npm install @stripe/stripe-react-native
```
- **[Documentazione](https://stripe.com/docs/payments/accept-a-payment?platform=react-native)** - Pagamenti Stripe
- Pro: PCI compliance, Apple Pay, Google Pay, 3D Secure

**Esempio Stripe PaymentSheet:**
```typescript
import { useStripe } from '@stripe/stripe-react-native';

function CheckoutScreen() {
  const { initPaymentSheet, presentPaymentSheet } = useStripe();

  const checkout = async () => {
    // 1. Fetch payment intent dal backend
    const { clientSecret } = await fetch('/create-payment-intent', {
      method: 'POST',
      body: JSON.stringify({ amount: 1000 }),
    }).then(r => r.json());

    // 2. Inizializza payment sheet
    const { error } = await initPaymentSheet({
      merchantDisplayName: 'My Store',
      paymentIntentClientSecret: clientSecret,
      applePay: { merchantCountryCode: 'IT' },
      googlePay: { merchantCountryCode: 'IT', testEnv: true },
    });

    if (error) return;

    // 3. Mostra payment sheet
    const { error: paymentError } = await presentPaymentSheet();

    if (!paymentError) {
      Alert.alert('Success', 'Payment completed!');
    }
  };

  return <Button title="Pay Now" onPress={checkout} />;
}
```

#### React Native IAP (In-App Purchases)
```bash
npm install react-native-iap
```
- **[Documentazione](https://react-native-iap.dooboolab.com/)** - Acquisti in-app iOS/Android

---

## 🎨 Design e Assets

### Design System e UI Kits
- **[React Native Directory](https://reactnative.directory/)** - Database componenti e librerie
- **[Figma](https://www.figma.com/)** - Design UI/UX (plugin per export React Native)
- **[React Native Sketch Elements](https://github.com/rNoz/react-native-sketch-elements)** - Drawing components

### Icone

#### Expo Vector Icons (Built-in Expo)
```bash
npx expo install @expo/vector-icons
```
- **[Directory](https://icons.expo.fyi/)** - 10,000+ icone da 15 famiglie
- Include: Ionicons, MaterialIcons, FontAwesome, Feather, AntDesign

#### React Native Vector Icons
```bash
npm install react-native-vector-icons
```
- **[Documentazione](https://github.com/oblador/react-native-vector-icons)** - Icone vettoriali

### Font Custom

#### Expo Font
```bash
npx expo install expo-font
```
**Esempio caricamento font:**
```typescript
import { useFonts } from 'expo-font';
import * as SplashScreen from 'expo-splash-screen';

SplashScreen.preventAutoHideAsync();

function App() {
  const [fontsLoaded] = useFonts({
    'Poppins-Regular': require('./assets/fonts/Poppins-Regular.ttf'),
    'Poppins-Bold': require('./assets/fonts/Poppins-Bold.ttf'),
  });

  useEffect(() => {
    if (fontsLoaded) {
      SplashScreen.hideAsync();
    }
  }, [fontsLoaded]);

  if (!fontsLoaded) return null;

  return <App />;
}
```

**Dove trovare font:**
- **[Google Fonts](https://fonts.google.com/)** - Font gratuiti
- **[Font Squirrel](https://www.fontsquirrel.com/)** - Font commerciali free

### Immagini e Assets
- **[Unsplash](https://unsplash.com/)** - Foto stock gratuite
- **[Pexels](https://www.pexels.com/)** - Video e foto stock
- **[Undraw](https://undraw.co/)** - Illustrazioni SVG personalizzabili
- **[Flaticon](https://www.flaticon.com/)** - Icone PNG/SVG

### Ottimizzazione Immagini
- **[TinyPNG](https://tinypng.com/)** - Compressione PNG/JPG
- **[Squoosh](https://squoosh.app/)** - Compressione immagini Google
- **[SVGO](https://jakearchibald.github.io/svgomg/)** - Ottimizzazione SVG

---

## 🌐 Backend e API

### Backend as a Service (BaaS)

#### Firebase (Google)
```bash
npm install @react-native-firebase/app
```
- **[Documentazione](https://rnfirebase.io/)** - Suite completa Google
- **Servizi:** Authentication, Firestore, Realtime Database, Storage, Cloud Functions, Analytics, Crashlytics
- Pro: Gratuito fino a limiti generosi, real-time, integrazione Google
- Contro: Vendor lock-in, pricing può salire

#### Supabase (Open Source)
```bash
npm install @supabase/supabase-js
```
- **[Documentazione](https://supabase.com/docs/guides/getting-started/quickstarts/react-native)** - Firebase alternative open source
- **Servizi:** PostgreSQL database, Authentication, Storage, Edge Functions
- Pro: SQL completo, real-time, self-hostable, pricing chiaro
- Contro: Ecosystem più piccolo di Firebase

**Esempio Supabase auth:**
```typescript
import { createClient } from '@supabase/supabase-js';
import 'react-native-url-polyfill/auto';

const supabase = createClient(
  'https://your-project.supabase.co',
  'your-anon-key'
);

// Signup
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123',
});

// Login
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password123',
});

// Fetch data
const { data: products } = await supabase
  .from('products')
  .select('*')
  .order('created_at', { ascending: false });
```

#### Appwrite
```bash
npm install appwrite
```
- **[Documentazione](https://appwrite.io/docs/getting-started-for-react-native)** - BaaS self-hosted
- **Servizi:** Database, Auth, Storage, Functions, Realtime
- Pro: Self-hosted, open source, privacy control

#### AWS Amplify
```bash
npm install aws-amplify @aws-amplify/react-native
```
- **[Documentazione](https://docs.amplify.aws/react-native/)** - AWS ecosystem
- Pro: Scalabilità AWS, integrazione tutti servizi AWS
- Contro: Complessità, pricing può essere alto

### API Mock e Development

#### JSON Server
```bash
npm install -g json-server
```
- **[Documentazione](https://github.com/typicode/json-server)** - REST API mock veloce
```bash
# Crea db.json con dati fake
json-server --watch db.json --port 3000
```

#### MSW (Mock Service Worker)
```bash
npm install msw --save-dev
```
- **[Documentazione](https://mswjs.io/)** - API mocking con interceptors
- Pro: Mock network-level, testing e development

---

## 📊 Analytics e Monitoring

### Analytics

#### Expo Analytics
```bash
npx expo install expo-firebase-analytics
```
- **[Documentazione](https://docs.expo.dev/versions/latest/sdk/firebase-analytics/)** - Google Analytics 4

#### Segment
```bash
npm install @segment/analytics-react-native
```
- **[Documentazione](https://segment.com/docs/connections/sources/catalog/libraries/mobile/react-native/)** - Analytics hub

#### Mixpanel
```bash
npm install mixpanel-react-native
```
- **[Documentazione](https://developer.mixpanel.com/docs/react-native)** - Product analytics

### Crash Reporting

#### Sentry (⭐ Consigliato)
```bash
npx expo install sentry-expo
```
- **[Documentazione](https://docs.sentry.io/platforms/react-native/)** - Error tracking
- Pro: Sourcemaps automatici, breadcrumbs, release tracking, performance monitoring

**Setup Sentry:**
```typescript
import * as Sentry from 'sentry-expo';

Sentry.init({
  dsn: 'your-sentry-dsn',
  enableInExpoDevelopment: false,
  debug: __DEV__,
  tracesSampleRate: 1.0, // Performance monitoring
});
```

#### Bugsnag
```bash
npm install @bugsnag/react-native
```
- **[Documentazione](https://docs.bugsnag.com/platforms/react-native/)** - Error monitoring

#### Firebase Crashlytics
```bash
npm install @react-native-firebase/crashlytics
```
- **[Documentazione](https://rnfirebase.io/crashlytics/usage)** - Crash reporting Google

---

## 🎓 Risorse di Apprendimento

### Tutorial e Corsi

#### Gratuiti
- **[React Native Official Tutorial](https://reactnative.dev/docs/tutorial)** - Tutorial ufficiale
- **[Expo Learn](https://docs.expo.dev/tutorial/introduction/)** - Tutorial Expo interattivo
- **[React Native School](https://www.reactnativeschool.com/)** - Video tutorial gratuiti
- **[William Candillon YouTube](https://www.youtube.com/@wcandillon)** - Animazioni avanzate
- **[Notjust.dev YouTube](https://www.youtube.com/@notjustdev)** - Full-stack RN projects

#### A Pagamento
- **[React Native - The Practical Guide](https://www.udemy.com/course/react-native-the-practical-guide/)** - Udemy Maximilian
- **[Zero to Mastery RN](https://zerotomastery.io/courses/learn-react-native/)** - Corso completo
- **[Frontend Masters](https://frontendmasters.com/courses/react-native-v3/)** - RN course Kadira Brock
- **[Egghead.io](https://egghead.io/q/react-native)** - Video bite-sized

### Blog e Newsletter
- **[React Native Blog](https://reactnative.dev/blog)** - Annunci ufficiali
- **[Expo Blog](https://blog.expo.dev/)** - Aggiornamenti Expo
- **[Infinite Red Blog](https://shift.infinite.red/tagged/react-native)** - Deep dive tecnici
- **[React Native Newsletter](https://reactnativenewsletter.com/)** - Newsletter settimanale
- **[This Week in React](https://thisweekinreact.com/)** - React e RN news

### Podcast
- **[React Native Radio](https://reactnativeradio.com/)** - Podcast ufficiale community
- **[The React Native Show](https://www.callstack.com/podcast-react-native-show)** - Callstack podcast

---

## 👥 Community e Supporto

### Forum e Chat
- **[React Native Community Discord](https://discord.gg/react-native-community)** - Discord ufficiale
- **[Expo Discord](https://discord.gg/expo)** - Community Expo
- **[Reactiflux Discord](https://www.reactiflux.com/)** - React generale
- **[Stack Overflow](https://stackoverflow.com/questions/tagged/react-native)** - Q&A tecnico
- **[Reddit r/reactnative](https://www.reddit.com/r/reactnative/)** - Subreddit attivo

### Eventi
- **[React Native EU](https://react-native.eu/)** - Conferenza annuale Europa
- **[Chain React](https://chainreactconf.com/)** - Conferenza USA
- **[App.js Conf](https://appjs.co/)** - Conferenza Expo/RN

---

## 🔧 Utility e Helper

### Generatori e Boilerplate

#### Ignite (⭐ Consigliato)
```bash
npx ignite-cli new MyApp
```
- **[Documentazione](https://github.com/infinitered/ignite)** - Boilerplate by Infinite Red
- Include: Navigation, state management, API layer, generators, Storybook

#### Expo Template
```bash
npx create-expo-app -t
```
- Template ufficiali: blank, tabs, bare-minimum

### Code Generation

#### Plop.js
```bash
npm install --save-dev plop
```
- **[Documentazione](https://plopjs.com/)** - Component generator
- Crea template per componenti, screens, hooks

### Linting e Formatting

#### ESLint Config Expo
```bash
npx expo install --save-dev eslint
```
```json
{
  "extends": ["expo", "prettier"]
}
```

#### Prettier
```bash
npm install --save-dev prettier
```
**.prettierrc:**
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

### TypeScript Helpers

#### Type Guards
```bash
npm install type-fest
```
- **[Documentazione](https://github.com/sindresorhus/type-fest)** - TypeScript utility types

---

## 📱 Device Testing

### Cloud Testing
- **[BrowserStack](https://www.browserstack.com/app-live)** - Real device cloud testing
- **[Sauce Labs](https://saucelabs.com/)** - Automated testing cloud
- **[AWS Device Farm](https://aws.amazon.com/device-farm/)** - Device testing AWS

### Local Emulators
- **[Android Studio Emulator](https://developer.android.com/studio/run/emulator)** - Emulatore Android ufficiale
- **[iOS Simulator](https://developer.apple.com/documentation/xcode/running-your-app-in-simulator-or-on-a-device)** - Simulatore iOS (macOS)
- **[Genymotion](https://www.genymotion.com/)** - Emulatore Android veloce (a pagamento)

---

## 🚀 Deployment e CI/CD

### Continuous Integration

#### GitHub Actions (Gratuito per repo pubblici)
**Workflow esempio (.github/workflows/ci.yml):**
```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run lint
      - run: npm test
      
  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - run: eas build --platform android --non-interactive
```

#### GitLab CI/CD
- **[Documentazione](https://docs.gitlab.com/ee/ci/)** - CI/CD GitLab

#### CircleCI
- **[Documentazione](https://circleci.com/docs/language-javascript/)** - CircleCI per RN

### Beta Distribution
- **[TestFlight](https://developer.apple.com/testflight/)** - Beta testing iOS (Apple)
- **[Google Play Internal Testing](https://support.google.com/googleplay/android-developer/answer/9845334)** - Testing tracks Android
- **[EAS Submit](https://docs.expo.dev/submit/introduction/)** - Submit automatico stores

---

## 🎯 Best Practices e Pattern

### Architecture
- **[React Native Architecture Guide](https://reactnative.dev/architecture/overview)** - New Architecture
- **[Atomic Design](https://atomicdesign.bradfrost.com/)** - Component organization
- **[Feature-Sliced Design](https://feature-sliced.design/)** - Scalable architecture

### Code Quality
- **[Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)** - Style guide popolare
- **[Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)** - Principi clean code

### Security
- **[OWASP Mobile Security](https://owasp.org/www-project-mobile-security/)** - Security best practices
- **[React Native Security Guide](https://reactnative.dev/docs/security)** - Guida ufficiale

---

## 📚 Libri Consigliati

- **"React Native in Action"** - Nader Dabit (Manning)
- **"Fullstack React Native"** - Devin Abbott, Houssein Djirdeh (Fullstack.io)
- **"React Native Cookbook"** - Dan Ward (Packt)
- **"Learning React Native"** - Bonnie Eisenman (O'Reilly)

---

## 🔗 Link Utili Veloci

| Risorsa | URL | Descrizione |
|---------|-----|-------------|
| React Native Docs | [reactnative.dev](https://reactnative.dev) | Documentazione ufficiale |
| Expo Docs | [docs.expo.dev](https://docs.expo.dev) | Documentazione Expo |
| RN Directory | [reactnative.directory](https://reactnative.directory) | Database librerie |
| Can I Use RN | [caniusenativemodules.com](https://caniusenativemodules.com) | Compatibilità moduli |
| React Navigation | [reactnavigation.org](https://reactnavigation.org) | Navigation library |
| React Native Paper | [reactnativepaper.com](https://reactnativepaper.com) | UI components |
| LottieFiles | [lottiefiles.com](https://lottiefiles.com) | Animazioni gratuite |
| Expo Snack | [snack.expo.dev](https://snack.expo.dev) | Playground online |

---

> **💡 Tip:** Aggiungi questa pagina ai preferiti e consultala regolarmente per scoprire nuove librerie e tool che possono migliorare il tuo workflow di sviluppo React Native!

[← Torna al README principale](../README.md)
