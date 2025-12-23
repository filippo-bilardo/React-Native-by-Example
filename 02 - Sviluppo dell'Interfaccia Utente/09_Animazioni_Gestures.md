# Capitolo 9: Animazioni e Gestures

[← Precedente: Liste e Performance](08_Liste_Performance.md) | [Torna all'Indice](../README.md) | [Prossimo: Form e Validazione →](10_Form_Validazione.md)

---

## Descrizione

Le animazioni e i gestures sono fondamentali per creare un'esperienza utente fluida e intuitiva. React Native offre diverse API per le animazioni: dalla semplice Animated API alle potenti Reanimated 2/3 per animazioni complesse e performanti.

In questo capitolo imparerai a creare animazioni smooth, gestire gestures come swipe e pinch, implementare transizioni, e costruire interazioni touch avanzate che rendono la tua app professionale e piacevole da usare.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Usare Animated API per animazioni base
- [ ] Implementare LayoutAnimation
- [ ] Creare animazioni con Reanimated 2/3
- [ ] Gestire gestures con Gesture Handler
- [ ] Implementare swipe gestures
- [ ] Creare animazioni di transizione
- [ ] Usare PanResponder per gestures custom
- [ ] Ottimizzare animazioni per 60fps
- [ ] Creare animazioni complesse (parallax, carousel, etc.)

## 📚 Argomenti Teorici

1. [Animated API Base](#1-animated-api)
2. [Timing, Spring, Decay](#2-animation-types)
3. [Interpolation](#3-interpolation)
4. [LayoutAnimation](#4-layoutanimation)
5. [Reanimated 2/3](#5-reanimated)
6. [Gesture Handler](#6-gesture-handler)
7. [PanResponder](#7-panresponder)
8. [Animazioni Pratiche](#8-practical-animations)
9. [Performance](#9-performance)
10. [Best Practices](#10-best-practices)

---

## 1. Animated API

### Setup Base

```typescript
import { Animated, View } from 'react-native';

const FadeInView = ({ children }: { children: React.ReactNode }) => {
  const fadeAnim = useRef(new Animated.Value(0)).current;
  
  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 1000,
      useNativeDriver: true  // ⚠️ IMPORTANTE per performance
    }).start();
  }, []);
  
  return (
    <Animated.View style={{ opacity: fadeAnim }}>
      {children}
    </Animated.View>
  );
};
```

### Animated Components

```typescript
// Componenti disponibili
<Animated.View />
<Animated.Text />
<Animated.Image />
<Animated.ScrollView />
<Animated.FlatList />

// Custom component
const AnimatedTouchable = Animated.createAnimatedComponent(TouchableOpacity);
```

### Value Types

```typescript
// Single value
const fadeAnim = new Animated.Value(0);

// XY value (per position)
const position = new Animated.ValueXY({ x: 0, y: 0 });

// Uso
<Animated.View
  style={{
    opacity: fadeAnim,
    transform: position.getTranslateTransform()
  }}
/>
```

---

## 2. Animation Types

### Timing (Lineare/Easing)

```typescript
Animated.timing(fadeAnim, {
  toValue: 1,
  duration: 300,
  easing: Easing.ease,  // Easing.linear, Easing.bezier(), etc.
  useNativeDriver: true
}).start();
```

### Spring (Fisica)

```typescript
Animated.spring(scaleAnim, {
  toValue: 1,
  friction: 3,      // Resistenza (default 7)
  tension: 40,      // Velocità (default 40)
  useNativeDriver: true
}).start();
```

### Decay (Decellerazione)

```typescript
Animated.decay(velocity, {
  velocity: 0.5,
  deceleration: 0.997,
  useNativeDriver: true
}).start();
```

### Callbacks

```typescript
Animated.timing(fadeAnim, {
  toValue: 1,
  duration: 300,
  useNativeDriver: true
}).start(({ finished }) => {
  if (finished) {
    console.log('Animation completed');
  }
});
```

---

## 3. Interpolation

### Basic Interpolation

```typescript
const animValue = new Animated.Value(0);

const opacity = animValue.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 1]
});

const translateY = animValue.interpolate({
  inputRange: [0, 1],
  outputRange: [0, -100]
});

<Animated.View
  style={{
    opacity,
    transform: [{ translateY }]
  }}
/>
```

### Rotation

```typescript
const rotation = animValue.interpolate({
  inputRange: [0, 1],
  outputRange: ['0deg', '360deg']
});

<Animated.View
  style={{
    transform: [{ rotate: rotation }]
  }}
/>
```

### Color Interpolation

```typescript
const backgroundColor = animValue.interpolate({
  inputRange: [0, 1],
  outputRange: ['rgb(255, 0, 0)', 'rgb(0, 255, 0)']
});

<Animated.View style={{ backgroundColor }} />
```

### Extrapolate

```typescript
const scale = animValue.interpolate({
  inputRange: [0, 1],
  outputRange: [1, 2],
  extrapolate: 'clamp'  // 'extend', 'clamp', 'identity'
});
```

---

## 4. LayoutAnimation

### Simple Usage

```typescript
import { LayoutAnimation, UIManager, Platform } from 'react-native';

// Android setup (una volta all'avvio)
if (Platform.OS === 'android') {
  if (UIManager.setLayoutAnimationEnabledExperimental) {
    UIManager.setLayoutAnimationEnabledExperimental(true);
  }
}

const App = () => {
  const [expanded, setExpanded] = useState(false);
  
  const toggleExpand = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
    setExpanded(!expanded);
  };
  
  return (
    <View>
      <TouchableOpacity onPress={toggleExpand}>
        <Text>Toggle</Text>
      </TouchableOpacity>
      
      {expanded && (
        <View style={{ height: 200 }}>
          <Text>Content</Text>
        </View>
      )}
    </View>
  );
};
```

### Presets

```typescript
LayoutAnimation.Presets.easeInEaseOut
LayoutAnimation.Presets.linear
LayoutAnimation.Presets.spring
```

### Custom Config

```typescript
LayoutAnimation.configureNext({
  duration: 300,
  create: {
    type: LayoutAnimation.Types.easeInEaseOut,
    property: LayoutAnimation.Properties.opacity
  },
  update: {
    type: LayoutAnimation.Types.spring,
    springDamping: 0.7
  },
  delete: {
    type: LayoutAnimation.Types.easeInEaseOut,
    property: LayoutAnimation.Properties.opacity
  }
});
```

---

## 5. Reanimated

### Installation

```bash
npm install react-native-reanimated

# iOS
cd ios && pod install

# babel.config.js
module.exports = {
  plugins: ['react-native-reanimated/plugin']  // Deve essere ultimo!
};
```

### Basic Animation

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring
} from 'react-native-reanimated';

const AnimatedBox = () => {
  const offset = useSharedValue(0);
  
  const animatedStyles = useAnimatedStyle(() => {
    return {
      transform: [{ translateX: offset.value }]
    };
  });
  
  const moveRight = () => {
    offset.value = withSpring(100);
  };
  
  return (
    <View>
      <Animated.View style={[styles.box, animatedStyles]} />
      <Button title="Move" onPress={moveRight} />
    </View>
  );
};
```

### Gestures con Reanimated

```typescript
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const DraggableBox = () => {
  const offset = useSharedValue({ x: 0, y: 0 });
  
  const pan = Gesture.Pan()
    .onUpdate((e) => {
      offset.value = {
        x: e.translationX,
        y: e.translationY
      };
    })
    .onEnd(() => {
      offset.value = withSpring({ x: 0, y: 0 });
    });
  
  const animatedStyles = useAnimatedStyle(() => ({
    transform: [
      { translateX: offset.value.x },
      { translateY: offset.value.y }
    ]
  }));
  
  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.box, animatedStyles]} />
    </GestureDetector>
  );
};
```

### Animated Values

```typescript
// Shared value
const progress = useSharedValue(0);

// Read value
console.log(progress.value);

// Set value
progress.value = 1;

// Animate
progress.value = withTiming(1, { duration: 300 });
progress.value = withSpring(1);
progress.value = withDecay({ velocity: 2000 });
```

### Worklets

```typescript
const animatedStyles = useAnimatedStyle(() => {
  'worklet';  // Opzionale, Reanimated lo aggiunge automaticamente
  
  return {
    opacity: progress.value,
    transform: [{ scale: 1 + progress.value }]
  };
});
```

---

## 6. Gesture Handler

### Installation

```bash
npm install react-native-gesture-handler

# iOS
cd ios && pod install

# Wrap app (index.js o App.tsx)
import { GestureHandlerRootView } from 'react-native-gesture-handler';

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      {/* Your app */}
    </GestureHandlerRootView>
  );
}
```

### Tap Gesture

```typescript
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const TapExample = () => {
  const tap = Gesture.Tap()
    .numberOfTaps(2)  // Double tap
    .onStart(() => {
      console.log('Tap started');
    })
    .onEnd(() => {
      console.log('Tap ended');
    });
  
  return (
    <GestureDetector gesture={tap}>
      <View style={styles.box}>
        <Text>Double tap me</Text>
      </View>
    </GestureDetector>
  );
};
```

### Pan Gesture

```typescript
const PanExample = () => {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  
  const pan = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
      translateY.value = event.translationY;
    })
    .onEnd(() => {
      translateX.value = withSpring(0);
      translateY.value = withSpring(0);
    });
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { translateY: translateY.value }
    ]
  }));
  
  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.box, animatedStyle]} />
    </GestureDetector>
  );
};
```

### Pinch Gesture

```typescript
const PinchExample = () => {
  const scale = useSharedValue(1);
  
  const pinch = Gesture.Pinch()
    .onUpdate((event) => {
      scale.value = event.scale;
    })
    .onEnd(() => {
      scale.value = withTiming(1);
    });
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }]
  }));
  
  return (
    <GestureDetector gesture={pinch}>
      <Animated.Image
        source={require('./image.jpg')}
        style={[styles.image, animatedStyle]}
      />
    </GestureDetector>
  );
};
```

### Rotation Gesture

```typescript
const RotationExample = () => {
  const rotation = useSharedValue(0);
  
  const rotate = Gesture.Rotation()
    .onUpdate((event) => {
      rotation.value = event.rotation;
    })
    .onEnd(() => {
      rotation.value = withTiming(0);
    });
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ rotateZ: `${rotation.value}rad` }]
  }));
  
  return (
    <GestureDetector gesture={rotate}>
      <Animated.View style={[styles.box, animatedStyle]} />
    </GestureDetector>
  );
};
```

### Composing Gestures

```typescript
// Simultaneous
const composed = Gesture.Simultaneous(pan, pinch);

// Sequential
const composed = Gesture.Sequential(longPress, pan);

// Exclusive
const composed = Gesture.Exclusive(pan1, pan2);
```

---

## 7. PanResponder

### Basic PanResponder

```typescript
import { PanResponder } from 'react-native';

const DraggableBox = () => {
  const pan = useRef(new Animated.ValueXY()).current;
  
  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onPanResponderMove: Animated.event(
        [null, { dx: pan.x, dy: pan.y }],
        { useNativeDriver: false }  // ⚠️ x,y non supportano native driver
      ),
      onPanResponderRelease: () => {
        Animated.spring(pan, {
          toValue: { x: 0, y: 0 },
          useNativeDriver: false
        }).start();
      }
    })
  ).current;
  
  return (
    <Animated.View
      style={{
        transform: [{ translateX: pan.x }, { translateY: pan.y }]
      }}
      {...panResponder.panHandlers}
    >
      <View style={styles.box} />
    </Animated.View>
  );
};
```

---

## 8. Practical Animations

### Fade In/Out

```typescript
const FadeInOut = ({ visible, children }: { visible: boolean; children: React.ReactNode }) => {
  const opacity = useRef(new Animated.Value(0)).current;
  
  useEffect(() => {
    Animated.timing(opacity, {
      toValue: visible ? 1 : 0,
      duration: 300,
      useNativeDriver: true
    }).start();
  }, [visible]);
  
  return (
    <Animated.View style={{ opacity }}>
      {children}
    </Animated.View>
  );
};
```

### Slide In

```typescript
const SlideIn = ({ children }: { children: React.ReactNode }) => {
  const translateY = useRef(new Animated.Value(100)).current;
  
  useEffect(() => {
    Animated.spring(translateY, {
      toValue: 0,
      useNativeDriver: true
    }).start();
  }, []);
  
  return (
    <Animated.View style={{ transform: [{ translateY }] }}>
      {children}
    </Animated.View>
  );
};
```

### Pulse Animation

```typescript
const Pulse = ({ children }: { children: React.ReactNode }) => {
  const scale = useRef(new Animated.Value(1)).current;
  
  useEffect(() => {
    Animated.loop(
      Animated.sequence([
        Animated.timing(scale, {
          toValue: 1.2,
          duration: 500,
          useNativeDriver: true
        }),
        Animated.timing(scale, {
          toValue: 1,
          duration: 500,
          useNativeDriver: true
        })
      ])
    ).start();
  }, []);
  
  return (
    <Animated.View style={{ transform: [{ scale }] }}>
      {children}
    </Animated.View>
  );
};
```

### Swipeable Card

```typescript
const SwipeableCard = ({ onSwipeLeft, onSwipeRight }: {
  onSwipeLeft: () => void;
  onSwipeRight: () => void;
}) => {
  const translateX = useSharedValue(0);
  const { width } = useWindowDimensions();
  
  const pan = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
    })
    .onEnd((event) => {
      if (Math.abs(event.translationX) > width * 0.3) {
        // Swipe complete
        const direction = event.translationX > 0 ? width : -width;
        translateX.value = withTiming(direction, {}, () => {
          runOnJS(event.translationX > 0 ? onSwipeRight : onSwipeLeft)();
        });
      } else {
        // Return to center
        translateX.value = withSpring(0);
      }
    });
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
    opacity: 1 - Math.abs(translateX.value) / width
  }));
  
  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <Text>Swipe me!</Text>
      </Animated.View>
    </GestureDetector>
  );
};
```

### Parallax Scroll

```typescript
const ParallaxScroll = () => {
  const scrollY = useSharedValue(0);
  
  const scrollHandler = useAnimatedScrollHandler({
    onScroll: (event) => {
      scrollY.value = event.contentOffset.y;
    }
  });
  
  const headerStyle = useAnimatedStyle(() => ({
    transform: [
      { translateY: -scrollY.value * 0.5 },
      { scale: 1 + scrollY.value / 1000 }
    ]
  }));
  
  return (
    <Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16}>
      <Animated.View style={[styles.header, headerStyle]}>
        <Text>Parallax Header</Text>
      </Animated.View>
      
      <View style={{ height: 2000 }}>
        <Text>Content...</Text>
      </View>
    </Animated.ScrollView>
  );
};
```

---

## 9. Performance

### useNativeDriver

```typescript
// ✅ Supportato (runs on native thread)
transform: [{ translateX }, { translateY }, { scale }, { rotate }]
opacity

// ❌ NON supportato
backgroundColor
width, height
left, right, top, bottom
```

### LayoutAnimation vs Animated

```typescript
// ✅ LayoutAnimation - per layout changes
LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
setExpanded(!expanded);

// ✅ Animated - per animazioni continue
Animated.timing(fadeAnim, { ... }).start();
```

### Reanimated Performance

```typescript
// ✅ Reanimated runs on UI thread - 60fps guaranteed
const animatedStyle = useAnimatedStyle(() => {
  return {
    transform: [{ translateX: offset.value }]
  };
});
```

---

## 10. Best Practices

### 1. Sempre useNativeDriver quando possibile

```typescript
// ✅ Good
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  useNativeDriver: true
}).start();

// ❌ Bad (se può usare native driver)
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  useNativeDriver: false
}).start();
```

### 2. Cleanup Animations

```typescript
useEffect(() => {
  const animation = Animated.timing(fadeAnim, { ... });
  animation.start();
  
  return () => {
    animation.stop();  // Cleanup
  };
}, []);
```

### 3. Reanimated per Gestures Complesse

```typescript
// ✅ Use Reanimated + Gesture Handler per performance
import Animated from 'react-native-reanimated';
import { Gesture } from 'react-native-gesture-handler';
```

### 4. Evita Re-renders

```typescript
// ❌ Bad - re-render component
const [opacity, setOpacity] = useState(0);

// ✅ Good - no re-render
const opacity = useRef(new Animated.Value(0)).current;
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Fade & Slide

Crea componente che:
- Fade in + slide up on mount
- Configurable duration
- Callback on complete

### Esercizio 2: 🟡 Swipeable List Item

Implementa swipe-to-delete:
- Swipe left mostra delete button
- Swipe completo elimina item
- Spring back se swipe incompleto
- Colori diversi per direzione

### Esercizio 3: 🟡 Image Viewer

Gallery con:
- Pinch to zoom
- Pan when zoomed
- Double tap to zoom
- Smooth animations

### Esercizio 4: 🔴 Animated Tabs

Tab bar custom con:
- Animated indicator
- Smooth transitions
- Gesture swipe tra tab
- Spring animations

### Esercizio 5: 🔴 Complex Carousel

Carousel con:
- Parallax effect
- Scale inactive items
- Snap to center
- Pagination dots animated
- Smooth 60fps scrolling

---

## 📝 Quiz

1. **useNativeDriver supporta width/height?**
   - [ ] Sì
   - [x] No
   - [ ] Solo su iOS
   - [ ] Serve flag

2. **Reanimated corre su:**
   - [ ] JS thread
   - [x] UI thread
   - [ ] Background thread
   - [ ] Main thread

3. **LayoutAnimation è utile per:**
   - [ ] Gestures
   - [x] Layout changes
   - [ ] Scroll animations
   - [ ] Color changes

4. **Per drag & drop performante usa:**
   - [ ] PanResponder
   - [ ] Animated.event
   - [x] Reanimated + Gesture Handler
   - [ ] LayoutAnimation

5. **Interpolate può mappare:**
   - [ ] Solo numbers
   - [ ] Solo colors
   - [x] Numbers, colors, strings
   - [ ] Solo opacity

**Risposte**: 1-b, 2-b, 3-b, 4-c, 5-c

---

## 🔗 Risorse

- [Animated API](https://reactnative.dev/docs/animated)
- [Reanimated Docs](https://docs.swmansion.com/react-native-reanimated/)
- [Gesture Handler](https://docs.swmansion.com/react-native-gesture-handler/)

---

## ✅ Checklist

- [ ] Animated API utilizzata
- [ ] LayoutAnimation implementata
- [ ] Reanimated installata e usata
- [ ] Gesture Handler configurata
- [ ] Gestures base implementate
- [ ] useNativeDriver usato dove possibile
- [ ] Animazioni smooth (60fps)
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🎬 **Animated API** per animazioni semplici
2. ⚡ **useNativeDriver: true** per performance
3. 🔄 **Reanimated** per animazioni complesse
4. 👆 **Gesture Handler** per gestures performanti
5. 🎨 **LayoutAnimation** per layout changes
6. 📊 **interpolate** per mappare valori
7. 🧹 **Cleanup** animations in useEffect
8. 🚀 **60fps** è l'obiettivo

---

**Straordinario! 🎉** Ora puoi creare UI animate e interattive. Prossimo: Form e Validazione!

[← Precedente: Liste e Performance](08_Liste_Performance.md) | [Torna all'Indice](../README.md) | [Prossimo: Form e Validazione →](10_Form_Validazione.md)
