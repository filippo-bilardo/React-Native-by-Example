# Capitolo 20: Ottimizzazione Performance

[← Precedente: Testing e QA](19_Testing.md) | [Torna all'Indice](../README.md) | [Prossimo: Build e Deployment →](21_Deployment.md)

---

## Descrizione

Le performance sono cruciali per user experience: app lente = utenti frustrati = disinstallazioni. React Native può essere velocissimo se ottimizzato correttamente. Bisogna profilare, identificare bottleneck, ottimizzare rendering, liste, immagini, bundle size, e monitorare in produzione.

In questo capitolo imparerai a usare profiling tools, detectare memory leaks, ottimizzare rendering, migliorare liste, ridurre bundle size, ottimizzare immagini, implementare lazy loading, e monitorare performance in produzione.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Profilare app con React DevTools
- [ ] Detectare memory leaks
- [ ] Ottimizzare re-rendering
- [ ] Migliorare performance liste
- [ ] Ridurre bundle size
- [ ] Ottimizzare immagini
- [ ] Implementare lazy loading
- [ ] Ottimizzare navigazione
- [ ] Monitorare performance
- [ ] Debuggare performance issues

## 📚 Argomenti Teorici

1. [Profiling Tools](#1-profiling)
2. [Memory Leaks](#2-memory-leaks)
3. [Rendering Optimization](#3-rendering)
4. [List Performance](#4-lists)
5. [Bundle Size](#5-bundle-size)
6. [Image Optimization](#6-images)
7. [Lazy Loading](#7-lazy-loading)
8. [Navigation Performance](#8-navigation)
9. [Performance Monitoring](#9-monitoring)
10. [Best Practices](#10-best-practices)

---

## 1. Profiling

### React DevTools Profiler

```typescript
// Enable profiling
import { enableScreens } from 'react-native-screens';
enableScreens();

// Wrap component to profile
import { Profiler } from 'react';

const onRenderCallback = (
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) => {
  console.log(`${id} ${phase} took ${actualDuration}ms`);
};

const App = () => {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Navigation />
    </Profiler>
  );
};
```

### Performance Monitor

```typescript
import { PerformanceObserver, performance } from 'react-native-performance';

const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});

observer.observe({ entryTypes: ['measure'] });

// Measure operation
performance.mark('operation-start');
// ... perform operation
performance.mark('operation-end');
performance.measure('operation', 'operation-start', 'operation-end');
```

### Flipper Performance Plugin

```bash
# Install Flipper
brew install --cask flipper

# Run app and open Flipper
# Enable Performance plugin
# Monitor:
# - React DevTools
# - Network
# - Logs
# - Layout Inspector
```

---

## 2. Memory Leaks

### Common Causes

```typescript
// ❌ BAD: Event listener not cleaned up
useEffect(() => {
  const subscription = eventEmitter.addListener('event', handler);
  // Missing cleanup!
}, []);

// ✅ GOOD: Cleanup listener
useEffect(() => {
  const subscription = eventEmitter.addListener('event', handler);
  return () => subscription.remove();
}, []);

// ❌ BAD: Timer not cleared
useEffect(() => {
  setInterval(() => {
    updateData();
  }, 1000);
}, []);

// ✅ GOOD: Clear timer
useEffect(() => {
  const interval = setInterval(() => {
    updateData();
  }, 1000);
  
  return () => clearInterval(interval);
}, []);

// ❌ BAD: Async operation after unmount
useEffect(() => {
  fetchData().then(setData);
}, []);

// ✅ GOOD: Check if mounted
useEffect(() => {
  let isMounted = true;
  
  fetchData().then((data) => {
    if (isMounted) {
      setData(data);
    }
  });
  
  return () => {
    isMounted = false;
  };
}, []);
```

### Memory Profiling

```typescript
// Use React Native's memory profiler
import { NativeModules } from 'react-native';

const logMemory = () => {
  if (__DEV__) {
    const memory = performance.memory;
    console.log('Memory:', {
      usedJSHeapSize: `${(memory.usedJSHeapSize / 1048576).toFixed(2)} MB`,
      totalJSHeapSize: `${(memory.totalJSHeapSize / 1048576).toFixed(2)} MB`,
      jsHeapSizeLimit: `${(memory.jsHeapSizeLimit / 1048576).toFixed(2)} MB`
    });
  }
};

// Log memory periodically
useEffect(() => {
  const interval = setInterval(logMemory, 5000);
  return () => clearInterval(interval);
}, []);
```

---

## 3. Rendering

### React.memo

```typescript
// ❌ BAD: Re-renders on every parent update
const ExpensiveComponent = ({ data }) => {
  console.log('Rendering...');
  return <View>{/* Complex rendering */}</View>;
};

// ✅ GOOD: Only re-renders when props change
const ExpensiveComponent = React.memo(({ data }) => {
  console.log('Rendering...');
  return <View>{/* Complex rendering */}</View>;
});

// With custom comparison
const ExpensiveComponent = React.memo(
  ({ data }) => {
    return <View>{/* Complex rendering */}</View>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip render)
    return prevProps.data.id === nextProps.data.id;
  }
);
```

### useMemo and useCallback

```typescript
// ❌ BAD: Recalculates on every render
const Component = ({ items }) => {
  const sortedItems = items.sort((a, b) => a.value - b.value);
  const handlePress = () => console.log('Pressed');
  
  return <List items={sortedItems} onPress={handlePress} />;
};

// ✅ GOOD: Memoize expensive calculations
const Component = ({ items }) => {
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.value - b.value);
  }, [items]);
  
  const handlePress = useCallback(() => {
    console.log('Pressed');
  }, []);
  
  return <List items={sortedItems} onPress={handlePress} />;
};
```

### Avoid Inline Objects and Functions

```typescript
// ❌ BAD: Creates new object/function on every render
<Component
  style={{ margin: 10 }}
  onPress={() => console.log('Pressed')}
/>

// ✅ GOOD: Define outside or use useMemo/useCallback
const styles = StyleSheet.create({
  container: { margin: 10 }
});

const handlePress = useCallback(() => {
  console.log('Pressed');
}, []);

<Component style={styles.container} onPress={handlePress} />
```

---

## 4. Lists

### FlatList Optimization

```typescript
// ✅ Optimized FlatList
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  
  // Performance props
  initialNumToRender={10}
  maxToRenderPerBatch={10}
  updateCellsBatchingPeriod={50}
  windowSize={5}
  
  // Remove clipped subviews (Android)
  removeClippedSubviews={true}
  
  // Memoize render function
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index
  })}
/>

// Memoized item component
const Item = React.memo(({ item, onPress }) => {
  return (
    <TouchableOpacity onPress={() => onPress(item.id)}>
      <Text>{item.title}</Text>
    </TouchableOpacity>
  );
});

const renderItem = useCallback(({ item }) => {
  return <Item item={item} onPress={handlePress} />;
}, [handlePress]);
```

### FlashList

```bash
npm install @shopify/flash-list
```

```typescript
import { FlashList } from '@shopify/flash-list';

// Better performance than FlatList
<FlashList
  data={items}
  renderItem={renderItem}
  estimatedItemSize={80}
  keyExtractor={(item) => item.id}
/>
```

### Virtualization

```typescript
// Use recyclerlistview for complex lists
import { RecyclerListView, DataProvider, LayoutProvider } from 'recyclerlistview';

const dataProvider = new DataProvider((r1, r2) => r1.id !== r2.id);
const layoutProvider = new LayoutProvider(
  (index) => 'NORMAL',
  (type, dim) => {
    dim.width = width;
    dim.height = 80;
  }
);

<RecyclerListView
  dataProvider={dataProvider.cloneWithRows(items)}
  layoutProvider={layoutProvider}
  rowRenderer={(type, item) => <Item item={item} />}
/>
```

---

## 5. Bundle Size

### Analyze Bundle

```bash
# Generate bundle report
npx react-native bundle \
  --platform android \
  --dev false \
  --entry-file index.js \
  --bundle-output bundle.js \
  --sourcemap-output bundle.map

# Analyze with source-map-explorer
npm install -g source-map-explorer
source-map-explorer bundle.js bundle.map
```

### Code Splitting

```typescript
// ❌ BAD: Import everything upfront
import LargeLibrary from 'large-library';

// ✅ GOOD: Dynamic import
const loadLargeFeature = async () => {
  const module = await import('large-library');
  return module.default;
};

// Use only when needed
const handleFeatureActivation = async () => {
  const LargeLibrary = await loadLargeFeature();
  // Use library
};
```

### Tree Shaking

```javascript
// ❌ BAD: Imports entire library
import _ from 'lodash';

// ✅ GOOD: Import only what you need
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

// Or use lodash-es (ES modules)
import { debounce, throttle } from 'lodash-es';
```

### Remove Unused Dependencies

```bash
# Find unused dependencies
npx depcheck

# Remove unused
npm uninstall unused-package
```

---

## 6. Images

### Image Optimization

```typescript
// ❌ BAD: Large unoptimized image
<Image
  source={{ uri: 'https://example.com/large-image.jpg' }}
  style={{ width: 100, height: 100 }}
/>

// ✅ GOOD: Optimized image with cache
import FastImage from 'react-native-fast-image';

<FastImage
  source={{
    uri: 'https://example.com/large-image.jpg',
    priority: FastImage.priority.normal,
    cache: FastImage.cacheControl.immutable
  }}
  style={{ width: 100, height: 100 }}
  resizeMode={FastImage.resizeMode.cover}
/>
```

### Image Formats

```typescript
// Use WebP format (smaller size)
<Image source={require('./image.webp')} />

// iOS: Use appropriate sizes (@2x, @3x)
<Image source={require('./icon.png')} />
// Assets: icon.png, icon@2x.png, icon@3x.png
```

### Lazy Load Images

```typescript
const LazyImage = ({ source, style }) => {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsVisible(true);
        observer.disconnect();
      }
    });
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <View ref={ref} style={style}>
      {isVisible ? (
        <FastImage source={source} style={style} />
      ) : (
        <View style={[style, { backgroundColor: '#f0f0f0' }]} />
      )}
    </View>
  );
};
```

---

## 7. Lazy Loading

### Lazy Component Loading

```typescript
import React, { lazy, Suspense } from 'react';

// Lazy load component
const HeavyComponent = lazy(() => import('./HeavyComponent'));

const App = () => {
  const [showHeavy, setShowHeavy] = useState(false);
  
  return (
    <View>
      <Button title="Load Heavy" onPress={() => setShowHeavy(true)} />
      
      {showHeavy && (
        <Suspense fallback={<ActivityIndicator />}>
          <HeavyComponent />
        </Suspense>
      )}
    </View>
  );
};
```

### Route-based Code Splitting

```typescript
// Navigation with lazy loading
const HomeScreen = lazy(() => import('./screens/HomeScreen'));
const ProfileScreen = lazy(() => import('./screens/ProfileScreen'));
const SettingsScreen = lazy(() => import('./screens/SettingsScreen'));

const Stack = createNativeStackNavigator();

const Navigation = () => {
  return (
    <NavigationContainer>
      <Suspense fallback={<LoadingScreen />}>
        <Stack.Navigator>
          <Stack.Screen name="Home" component={HomeScreen} />
          <Stack.Screen name="Profile" component={ProfileScreen} />
          <Stack.Screen name="Settings" component={SettingsScreen} />
        </Stack.Navigator>
      </Suspense>
    </NavigationContainer>
  );
};
```

---

## 8. Navigation

### Screen Optimization

```typescript
// ✅ Optimize navigation
import { enableScreens } from 'react-native-screens';
enableScreens();

// Use native stack
import { createNativeStackNavigator } from '@react-navigation/native-stack';
const Stack = createNativeStackNavigator();

// Freeze inactive screens
import { enableFreeze } from 'react-native-screens';
enableFreeze(true);
```

### Lazy Screen Loading

```typescript
const Stack = createNativeStackNavigator();

<Stack.Navigator screenOptions={{ lazy: true }}>
  <Stack.Screen 
    name="Home" 
    component={HomeScreen}
    options={{ 
      freezeOnBlur: true  // Freeze when not focused
    }}
  />
</Stack.Navigator>
```

---

## 9. Monitoring

### Firebase Performance Monitoring

```bash
npm install @react-native-firebase/perf
```

```typescript
import perf from '@react-native-firebase/perf';

// Measure screen load time
const trace = await perf().startTrace('screen_load');
// Load screen
await trace.stop();

// Measure network request
const metric = await perf().newHttpMetric(url, 'GET');
await metric.start();
// Make request
await metric.stop();

// Custom metric
const customTrace = await perf().startTrace('custom_operation');
customTrace.putAttribute('user_id', userId);
customTrace.putMetric('items_loaded', 42);
await customTrace.stop();
```

### Sentry Performance

```bash
npm install @sentry/react-native
```

```typescript
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'YOUR_DSN',
  enableAutoSessionTracking: true,
  tracesSampleRate: 1.0
});

// Measure transaction
const transaction = Sentry.startTransaction({
  name: 'User Profile Load',
  op: 'navigation'
});

Sentry.getCurrentHub().configureScope((scope) => {
  scope.setSpan(transaction);
});

// Create spans
const span = transaction.startChild({
  op: 'http',
  description: 'Fetch user data'
});

await fetchUserData();

span.finish();
transaction.finish();
```

---

## 10. Best Practices

### Performance Checklist

```typescript
// 1. Enable Hermes
// android/app/build.gradle
project.ext.react = [
  enableHermes: true
]

// 2. Use production build
// npm run build

// 3. Optimize images
// - Use WebP
// - Appropriate resolutions
// - FastImage for caching

// 4. Memoize expensive operations
// - React.memo for components
// - useMemo for calculations
// - useCallback for functions

// 5. Optimize lists
// - FlatList with getItemLayout
// - FlashList for better performance
// - windowSize and initialNumToRender

// 6. Reduce bundle size
// - Code splitting
// - Tree shaking
// - Remove unused deps

// 7. Monitor performance
// - Firebase Performance
// - Sentry
// - React DevTools Profiler

// 8. Profile regularly
// - Before major releases
// - After adding features
// - When users report issues
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 List Optimization

Ottimizza lista con 1000+ items:
- Implement FlatList optimization
- Add getItemLayout
- Memoize render function
- Measure performance improvement

### Esercizio 2: 🟡 Image Gallery

Galleria foto performante:
- Lazy load images
- Use FastImage
- Implement pagination
- Optimize memory usage
- Add loading states

### Esercizio 3: 🟡 Bundle Analysis

Analizza e riduci bundle:
- Generate bundle report
- Identify large dependencies
- Implement code splitting
- Remove unused code
- Measure size reduction

### Esercizio 4: 🔴 Performance Dashboard

Dashboard monitoring:
- Integrate Firebase Performance
- Add custom traces
- Monitor API calls
- Track screen load times
- Set up alerts

### Esercizio 5: 🔴 App Optimization

Ottimizza app esistente:
- Profile with React DevTools
- Fix memory leaks
- Optimize re-renders
- Improve list performance
- Reduce bundle size
- Add performance monitoring
- Achieve 60fps consistently

---

## 📝 Quiz

1. **React.memo serve per:**
   - [x] Evitare re-render inutili
   - [ ] Memory management
   - [ ] Solo performance lists
   - [ ] Bundle size

2. **getItemLayout ottimizza:**
   - [ ] Rendering
   - [x] FlatList scrolling
   - [ ] Bundle size
   - [ ] Memory

3. **Hermes è:**
   - [ ] Library
   - [x] JavaScript engine ottimizzato
   - [ ] Testing tool
   - [ ] Profiler

4. **Code splitting riduce:**
   - [x] Initial bundle size
   - [ ] Memory usage
   - [ ] Re-renders
   - [ ] API calls

5. **FastImage migliora:**
   - [ ] Bundle size
   - [x] Image loading e caching
   - [ ] Memory leaks
   - [ ] Re-renders

**Risposte**: 1-a, 2-b, 3-b, 4-a, 5-b

---

## 🔗 Risorse

- [React Native Performance](https://reactnative.dev/docs/performance)
- [Hermes Engine](https://hermesengine.dev/)
- [React DevTools Profiler](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)
- [Firebase Performance](https://firebase.google.com/docs/perf-mon)

---

## ✅ Checklist

- [ ] Profiling tools configurati
- [ ] Memory leaks risolti
- [ ] Rendering ottimizzato
- [ ] Liste performanti
- [ ] Bundle size ridotto
- [ ] Immagini ottimizzate
- [ ] Lazy loading implementato
- [ ] Navigation ottimizzata
- [ ] Monitoring configurato
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📊 **Profile** prima di ottimizzare
2. 🧠 **Memory leaks** cleanup listeners
3. ⚛️ **React.memo** evita re-render
4. 📋 **FlatList** con getItemLayout
5. 📦 **Bundle size** code splitting
6. 🖼️ **FastImage** per caching
7. ⏳ **Lazy loading** componenti pesanti
8. 🎯 **Hermes** engine performante
9. 📈 **Monitoring** Firebase/Sentry
10. 🎮 **60fps** target performance

---

**Eccellente! 🎉** App ottimizzata per performance! Prossimo: Build e Deployment!

[← Precedente: Testing e QA](19_Testing.md) | [Torna all'Indice](../README.md) | [Prossimo: Build e Deployment →](21_Deployment.md)
