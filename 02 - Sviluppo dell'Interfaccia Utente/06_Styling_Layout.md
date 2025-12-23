# Capitolo 6: Styling e Layout

[← Precedente: Componenti Core](05_Componenti_Core.md) | [Torna all'Indice](../README.md) | [Prossimo: Navigazione →](07_Navigazione.md)

---

## Descrizione

Lo styling in React Native è simile a CSS ma con alcune differenze importanti. Invece di CSS tradizionale, utilizzerai JavaScript objects con proprietà camelCase. Il layout è basato principalmente su **Flexbox**, lo stesso sistema usato nel CSS moderno ma con alcune differenze di default.

In questo capitolo imparerai a creare layout responsive, applicare stili in modo efficiente, gestire le differenze tra piattaforme, e creare UI che si adattano a diverse dimensioni dello schermo.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Applicare stili con StyleSheet API
- [ ] Padroneggiare Flexbox per layout
- [ ] Creare UI responsive per diversi schermi
- [ ] Gestire dimensioni e unità di misura
- [ ] Implementare temi (dark/light mode)
- [ ] Utilizzare Platform-specific styling
- [ ] Ottimizzare performance degli stili
- [ ] Usare librerie di styling popolari
- [ ] Creare design systems riutilizzabili

## 📚 Argomenti Teorici

1. [StyleSheet API](#1-stylesheet-api)
2. [Flexbox in React Native](#2-flexbox)
3. [Dimensioni e Unità](#3-dimensioni-unità)
4. [Responsive Design](#4-responsive-design)
5. [Colors e Theming](#5-colors-theming)
6. [Shadows e Elevation](#6-shadows-elevation)
7. [Transform e Positioning](#7-transform-positioning)
8. [Platform-Specific Styling](#8-platform-specific)
9. [Styled Components e Alternative](#9-styled-components)
10. [Best Practices](#10-best-practices)

---

## 1. StyleSheet API

### Inline Styles

Gli inline styles sono il modo più semplice ma MENO efficiente di stilizzare componenti:

```typescript
<View style={{ backgroundColor: 'blue', padding: 20 }}>
  <Text style={{ color: 'white', fontSize: 16 }}>Hello</Text>
</View>
```

❌ **Problema CRITICO**: 
- JavaScript crea un NUOVO oggetto style ad ogni render
- React Native deve processare l'oggetto ogni volta
- Con liste di 1000 elementi, crei 1000 oggetti nuovi ad ogni render
- **Impatto performance**: rendering lento, animazioni lag

### StyleSheet.create (✅ Metodo Consigliato)

`StyleSheet.create` ottimizza gli stili creandoli UNA SOLA VOLTA e riutilizzandoli:

```typescript
import { StyleSheet } from 'react-native';

// Stili definiti FUORI dal componente = creati una volta sola
const styles = StyleSheet.create({
  container: {
    backgroundColor: 'blue',
    padding: 20
  },
  text: {
    color: 'white',
    fontSize: 16  // Numero = pixel (no 'px' come in CSS)
  }
});

// Nel componente: usa riferimento allo style object
<View style={styles.container}>
  <Text style={styles.text}>Hello</Text>
</View>
```

**Vantaggi di StyleSheet.create:**
- ✅ **Performance**: Stili creati una volta, riutilizzati sempre
- ✅ **Validazione**: React Native valida le proprietà a compile-time
- ✅ **Ottimizzazione**: Bridge ottimizza la comunicazione JS ↔ Native
- ✅ **Autocomplete**: TypeScript/IDE suggeriscono proprietà valide
- ✅ **Type-safety**: Errori rilevati durante sviluppo, non in produzione

**Come funziona internamente:**
```typescript
// StyleSheet.create trasforma gli styles in ID numerici
const styles = StyleSheet.create({ box: { width: 100 } });
// Diventa → { box: { __styleId: 42 } }
// React Native memorizza style ID=42 → { width: 100 } lato native
```

### Array di Stili

React Native supporta array di stili per comporre e sovrascrivere proprietà dinamicamente:

```typescript
// Componi stili multipli con array
<View style={[styles.container, styles.bordered]}>
  <Text style={[styles.text, { fontWeight: 'bold' }]}>Hello</Text>
</View>

/**
 * REGOLA DI PRECEDENZA: Gli stili più a DESTRA sovrascrivono quelli a sinistra
 * Esempio: [{ color: 'red' }, { color: 'blue' }] → risultato: color: 'blue'
 */
<View style={[
  styles.box,  // backgroundColor: 'green'
  { backgroundColor: 'red' },  // ✅ SOVRASCRIVE backgroundColor → ora è rosso
  isActive && styles.active  // Applica SOLO se isActive=true
]}>
```

**Pattern comuni con array:**
```typescript
// 1. Stili base + varianti
<Button style={[styles.button, isPrimary && styles.primary]} />

// 2. Responsive: applica stili in base a dimensioni
<View style={[styles.container, isLargeScreen && styles.containerLarge]} />

// 3. Stati multipli combinati
<Touchable style={[
  styles.base,
  isPressed && styles.pressed,
  isDisabled && styles.disabled,
  isFocused && styles.focused
]} />

// 4. Override condizionale dell'ultimo minuto
<Text style={[styles.text, { color: errorColor }]} />
```

### Stili Condizionali

```typescript
<View style={[
  styles.button,
  disabled && styles.disabled,
  variant === 'primary' && styles.primary,
  variant === 'secondary' && styles.secondary
]}>
```

### Composizione Stili

```typescript
const styles = StyleSheet.create({
  base: {
    padding: 10,
    borderRadius: 5
  },
  primary: {
    backgroundColor: '#007AFF',
    color: 'white'
  },
  secondary: {
    backgroundColor: '#5856D6',
    color: 'white'
  }
});

<TouchableOpacity style={[styles.base, styles.primary]}>
```

---

## 2. Flexbox

### Differenze con CSS

React Native usa Flexbox, MA con default diversi dal web CSS. Questo causa confusione iniziale:

| Proprietà | CSS Default | React Native Default | Motivo |
|-----------|-------------|----------------------|---------|
| flexDirection | row | **column** | Mobile UI è verticale |
| alignContent | stretch | **flex-start** | Performance |
| flexShrink | 1 | **0** | Previene shrink inatteso |
| flex | 0 1 auto | **0 0 auto** | Comportamento esplicito |

**📱 Perché questi default?**
- **column**: App mobile sono naturalmente verticali (scroll verticale)
- **flex-start**: Allineare in alto è più comune su mobile
- **flexShrink: 0**: Previene che elementi si riducano accidentalmente

### Container Properties (Flex Container)

Un "flex container" è una View che contiene altri elementi e controlla il loro layout:

```typescript
<View style={{
  // DIREZIONE: Definisce l'asse principale (main axis)
  flexDirection: 'column',  
  // 'column' → main axis verticale (↓), cross axis orizzontale (→)
  // 'row' → main axis orizzontale (→), cross axis verticale (↓)
  // 'column-reverse' → main axis verticale (↑)
  // 'row-reverse' → main axis orizzontale (←)
  
  // ALLINEAMENTO MAIN AXIS: Come i figli si distribuiscono lungo l'asse principale
  justifyContent: 'flex-start',  
  // 'flex-start' → inizio (top se column, left se row)
  // 'flex-end' → fine (bottom se column, right se row)
  // 'center' → centro dell'asse principale
  // 'space-between' → primo elemento all'inizio, ultimo alla fine, spazio uguale tra
  // 'space-around' → spazio uguale ATTORNO ad ogni elemento
  // 'space-evenly' → spazio uguale tra TUTTI gli elementi (inclusi i bordi)
  
  // ALLINEAMENTO CROSS AXIS: Come i figli si allineano sull'asse secondario
  alignItems: 'stretch',  // 'flex-start', 'flex-end', 'center', 'stretch', 'baseline'
  
  // Multi-line alignment
  alignContent: 'flex-start',  // Come justifyContent ma per linee multiple
  
  // Wrap
  flexWrap: 'nowrap',  // 'nowrap', 'wrap', 'wrap-reverse'
  
  // Gap (React Native 0.71+)
  gap: 10,
  rowGap: 10,
  columnGap: 10
}}>
```

### Item Properties

```typescript
<View style={{
  // Crescita
  flex: 1,  // Prende spazio disponibile
  
  // O specificamente:
  flexGrow: 1,    // Quanto cresce
  flexShrink: 1,  // Quanto si restringe
  flexBasis: 100, // Dimensione base
  
  // Self alignment
  alignSelf: 'auto',  // 'auto', 'flex-start', 'flex-end', 'center', 'stretch', 'baseline'
}}>
```

### Esempi Pratici

**Centrare contenuto:**
```typescript
const styles = StyleSheet.create({
  centered: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
});

<View style={styles.centered}>
  <Text>Centered!</Text>
</View>
```

**Layout a colonne:**
```typescript
const styles = StyleSheet.create({
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 15
  }
});

<View style={styles.row}>
  <Text>Left</Text>
  <Text>Right</Text>
</View>
```

**Distribuzione uguale:**
```typescript
<View style={{ flexDirection: 'row' }}>
  <View style={{ flex: 1, backgroundColor: 'red' }}>
    <Text>1/3</Text>
  </View>
  <View style={{ flex: 1, backgroundColor: 'green' }}>
    <Text>1/3</Text>
  </View>
  <View style={{ flex: 1, backgroundColor: 'blue' }}>
    <Text>1/3</Text>
  </View>
</View>
```

**Header, Content, Footer:**
```typescript
<View style={{ flex: 1 }}>
  <View style={styles.header}>
    <Text>Header (fixed height)</Text>
  </View>
  
  <View style={{ flex: 1 }}>
    <Text>Content (takes remaining space)</Text>
  </View>
  
  <View style={styles.footer}>
    <Text>Footer (fixed height)</Text>
  </View>
</View>

const styles = StyleSheet.create({
  header: {
    height: 60,
    backgroundColor: '#007AFF'
  },
  footer: {
    height: 80,
    backgroundColor: '#5856D6'
  }
});
```

---

## 3. Dimensioni e Unità

### Unità Disponibili

React Native usa **density-independent pixels (dp/pt)**.

```typescript
// ❌ NO CSS units
fontSize: '16px'  // Errore!
width: '50%'      // OK solo come stringa
width: 50         // OK - number è in dp

// ✅ Solo numbers o percentuali come string
width: 100        // 100 dp
width: '50%'      // 50% del container
height: '100%'
```

### Fixed Dimensions

```typescript
<View style={{ width: 200, height: 100 }}>
  <Text>Fixed size</Text>
</View>
```

### Percentage

```typescript
<View style={{ width: '80%', height: '50%' }}>
  <Text>Percentage size</Text>
</View>
```

### Flex Dimensions (✅ Consigliato per responsive)

```typescript
<View style={{ flex: 1 }}>
  <Text>Takes all available space</Text>
</View>
```

### Dimensions API

```typescript
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');
// o Dimensions.get('screen') - include status bar

console.log(`Width: ${width}, Height: ${height}`);

<View style={{ width: width * 0.8, height: height * 0.5 }}>
  <Text>80% width, 50% height</Text>
</View>
```

### useWindowDimensions Hook (✅ React to rotations)

```typescript
import { useWindowDimensions } from 'react-native';

const MyComponent = () => {
  const { width, height } = useWindowDimensions();
  
  return (
    <View style={{ width: width * 0.9 }}>
      <Text>Width: {width}</Text>
    </View>
  );
};
```

---

## 4. Responsive Design

### Breakpoints

```typescript
import { useWindowDimensions } from 'react-native';

const MyComponent = () => {
  const { width } = useWindowDimensions();
  
  const isSmall = width < 375;
  const isMedium = width >= 375 && width < 768;
  const isLarge = width >= 768;
  
  return (
    <View style={{
      padding: isSmall ? 10 : isMedium ? 20 : 30
    }}>
      <Text style={{
        fontSize: isSmall ? 14 : isMedium ? 16 : 20
      }}>
        Responsive Text
      </Text>
    </View>
  );
};
```

### Scale Function

```typescript
import { Dimensions } from 'react-native';

const { width: SCREEN_WIDTH } = Dimensions.get('window');
const BASE_WIDTH = 375; // iPhone SE width

const scale = (size: number) => (SCREEN_WIDTH / BASE_WIDTH) * size;

// Uso
<Text style={{ fontSize: scale(16) }}>Scaled text</Text>
```

### Custom Hook per Responsive

```typescript
import { useWindowDimensions } from 'react-native';

type Breakpoint = 'small' | 'medium' | 'large';

const useResponsive = () => {
  const { width, height } = useWindowDimensions();
  
  const breakpoint: Breakpoint = 
    width < 375 ? 'small' :
    width < 768 ? 'medium' : 'large';
  
  const isSmall = breakpoint === 'small';
  const isMedium = breakpoint === 'medium';
  const isLarge = breakpoint === 'large';
  
  const spacing = {
    small: { padding: 10, margin: 5 },
    medium: { padding: 20, margin: 10 },
    large: { padding: 30, margin: 15 }
  }[breakpoint];
  
  return {
    width,
    height,
    breakpoint,
    isSmall,
    isMedium,
    isLarge,
    spacing
  };
};

// Uso
const MyComponent = () => {
  const { isSmall, spacing } = useResponsive();
  
  return (
    <View style={{ padding: spacing.padding }}>
      <Text>Responsive!</Text>
    </View>
  );
};
```

### Orientation

```typescript
const MyComponent = () => {
  const { width, height } = useWindowDimensions();
  const isPortrait = height > width;
  
  return (
    <View style={{
      flexDirection: isPortrait ? 'column' : 'row'
    }}>
      <Text>Adapts to orientation</Text>
    </View>
  );
};
```

---

## 5. Colors e Theming

### Color Formats

```typescript
// Named colors
backgroundColor: 'red'

// Hex
backgroundColor: '#FF0000'
backgroundColor: '#F00'

// RGB
backgroundColor: 'rgb(255, 0, 0)'

// RGBA
backgroundColor: 'rgba(255, 0, 0, 0.5)'

// HSL (React Native 0.64+)
backgroundColor: 'hsl(0, 100%, 50%)'
backgroundColor: 'hsla(0, 100%, 50%, 0.5)'
```

### Color Constants

```typescript
// colors.ts
export const Colors = {
  primary: '#007AFF',
  secondary: '#5856D6',
  success: '#34C759',
  danger: '#FF3B30',
  warning: '#FF9500',
  info: '#5AC8FA',
  
  background: '#FFFFFF',
  surface: '#F2F2F7',
  border: '#C6C6C8',
  
  text: {
    primary: '#000000',
    secondary: '#8E8E93',
    disabled: '#C7C7CC'
  }
};

// Uso
import { Colors } from './colors';

<View style={{ backgroundColor: Colors.primary }}>
```

### Theme Context

```typescript
// ThemeContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeColors {
  background: string;
  surface: string;
  primary: string;
  text: string;
  border: string;
}

const lightColors: ThemeColors = {
  background: '#FFFFFF',
  surface: '#F2F2F7',
  primary: '#007AFF',
  text: '#000000',
  border: '#C6C6C8'
};

const darkColors: ThemeColors = {
  background: '#000000',
  surface: '#1C1C1E',
  primary: '#0A84FF',
  text: '#FFFFFF',
  border: '#38383A'
};

interface ThemeContextType {
  theme: Theme;
  colors: ThemeColors;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [theme, setTheme] = useState<Theme>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const colors = theme === 'light' ? lightColors : darkColors;
  
  return (
    <ThemeContext.Provider value={{ theme, colors, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// Uso nei componenti
const MyComponent = () => {
  const { colors } = useTheme();
  
  return (
    <View style={{ backgroundColor: colors.background }}>
      <Text style={{ color: colors.text }}>Themed text</Text>
    </View>
  );
};
```

### System Theme

```typescript
import { useColorScheme } from 'react-native';

const MyComponent = () => {
  const colorScheme = useColorScheme();  // 'light' | 'dark' | null
  
  const backgroundColor = colorScheme === 'dark' ? '#000' : '#fff';
  
  return (
    <View style={{ backgroundColor }}>
      <Text>Follows system theme</Text>
    </View>
  );
};
```

---

## 6. Shadows e Elevation

### iOS Shadows

```typescript
const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 8,
    
    // iOS shadows
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84
  }
});
```

### Android Elevation

```typescript
const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 8,
    
    // Android elevation
    elevation: 5
  }
});
```

### Cross-Platform Shadow

```typescript
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 8,
    
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84
      },
      android: {
        elevation: 5
      }
    })
  }
});

// O con StyleSheet.create helper
const createShadow = (elevation: number) => {
  return Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: elevation },
      shadowOpacity: 0.25,
      shadowRadius: elevation * 1.5
    },
    android: {
      elevation
    }
  });
};

const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 8,
    ...createShadow(5)
  }
});
```

---

## 7. Transform e Positioning

### Transform

```typescript
<View style={{
  transform: [
    { translateX: 50 },
    { translateY: 100 },
    { rotate: '45deg' },
    { rotateX: '45deg' },
    { rotateY: '45deg' },
    { rotateZ: '45deg' },
    { scale: 1.5 },
    { scaleX: 2 },
    { scaleY: 0.5 },
    { skewX: '45deg' },
    { skewY: '15deg' }
  ]
}}>
  <Text>Transformed</Text>
</View>
```

### Positioning

```typescript
// Position types
<View style={{ position: 'relative' }}>  // Default
<View style={{ position: 'absolute' }}>

// Absolute positioning
<View style={{
  position: 'absolute',
  top: 0,
  left: 0,
  right: 0,
  bottom: 0  // Fills parent
}}>

// Esempio: Badge
<View style={styles.container}>
  <Image source={require('./avatar.png')} />
  <View style={styles.badge}>
    <Text>3</Text>
  </View>
</View>

const styles = StyleSheet.create({
  container: {
    position: 'relative'
  },
  badge: {
    position: 'absolute',
    top: -5,
    right: -5,
    backgroundColor: 'red',
    borderRadius: 10,
    width: 20,
    height: 20,
    justifyContent: 'center',
    alignItems: 'center'
  }
});
```

### Z-Index

```typescript
<View>
  <View style={{ position: 'absolute', zIndex: 1 }}>
    <Text>Front</Text>
  </View>
  <View style={{ position: 'absolute', zIndex: 0 }}>
    <Text>Back</Text>
  </View>
</View>
```

---

## 8. Platform-Specific

### Platform.select

```typescript
import { Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    padding: Platform.select({
      ios: 20,
      android: 15,
      default: 10
    }),
    
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOpacity: 0.3
      },
      android: {
        elevation: 5
      }
    })
  }
});
```

### Platform OS

```typescript
const styles = StyleSheet.create({
  text: {
    fontFamily: Platform.OS === 'ios' ? 'System' : 'Roboto',
    fontSize: Platform.OS === 'ios' ? 17 : 16
  }
});
```

---

## 9. Styled Components

### React Native Extended StyleSheet

```bash
npm install react-native-extended-stylesheet
```

```typescript
import EStyleSheet from 'react-native-extended-stylesheet';

const styles = EStyleSheet.create({
  container: {
    padding: '20rem',  // Relative units
    backgroundColor: '$primaryColor',  // Variables
    width: '90%'
  }
});

// Build styles
EStyleSheet.build({
  $primaryColor: '#007AFF',
  $rem: 16  // Base unit
});
```

### Styled Components

```bash
npm install styled-components
npm install --save-dev @types/styled-components-react-native
```

```typescript
import styled from 'styled-components/native';

const Container = styled.View`
  flex: 1;
  background-color: ${props => props.theme.background};
  padding: 20px;
`;

const Title = styled.Text`
  font-size: 24px;
  font-weight: bold;
  color: ${props => props.theme.text};
`;

const Button = styled.TouchableOpacity<{ primary?: boolean }>`
  background-color: ${props => props.primary ? '#007AFF' : '#5856D6'};
  padding: 15px;
  border-radius: 8px;
`;

// Uso
const App = () => (
  <Container>
    <Title>Hello</Title>
    <Button primary>
      <Text>Primary</Text>
    </Button>
  </Container>
);
```

### Tailwind CSS (NativeWind)

```bash
npm install nativewind
npm install --save-dev tailwindcss
```

```typescript
import { View, Text } from 'react-native';

<View className="flex-1 bg-white p-4">
  <Text className="text-2xl font-bold text-gray-900">
    Tailwind in React Native!
  </Text>
</View>
```

---

## 10. Best Practices

### 1. Usa StyleSheet.create

```typescript
// ❌ Evita
<View style={{ padding: 20, backgroundColor: 'white' }}>

// ✅ Usa
const styles = StyleSheet.create({
  container: { padding: 20, backgroundColor: 'white' }
});
<View style={styles.container}>
```

### 2. Evita Stili Inline Dinamici

```typescript
// ❌ Evita - crea nuovo oggetto ogni render
<View style={{ backgroundColor: isActive ? 'blue' : 'gray' }}>

// ✅ Meglio
<View style={[styles.box, isActive && styles.activeBox]}>
```

### 3. Componenti Styled Riutilizzabili

```typescript
const Card = ({ children, style }) => (
  <View style={[styles.card, style]}>
    {children}
  </View>
);

const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 8,
    padding: 15,
    shadowColor: '#000',
    shadowOpacity: 0.1,
    elevation: 3
  }
});
```

### 4. Design Tokens

```typescript
// tokens.ts
export const Spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32
};

export const FontSize = {
  small: 12,
  medium: 16,
  large: 20,
  xlarge: 24
};

export const BorderRadius = {
  small: 4,
  medium: 8,
  large: 16,
  round: 9999
};

// Uso
import { Spacing, FontSize, BorderRadius } from './tokens';

const styles = StyleSheet.create({
  button: {
    padding: Spacing.md,
    borderRadius: BorderRadius.medium,
    fontSize: FontSize.medium
  }
});
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Card Component

Crea un card component responsive con:
- Shadow per iOS e elevation per Android
- Padding responsive
- Border radius
- Background color dal theme

### Esercizio 2: 🟡 Grid Layout

Implementa una griglia responsive:
- 2 colonne su mobile
- 3 colonne su tablet
- 4 colonne su desktop
- Gap tra gli elementi
- Items di uguale altezza

### Esercizio 3: 🟡 Theme System

Crea un sistema di temi completo:
- Light e Dark theme
- Context per gestire theme
- Toggle button
- Persistenza con AsyncStorage
- Almeno 5 colori per tema

### Esercizio 4: 🔴 Responsive Dashboard

Crea una dashboard che:
- Ha layout diverso per portrait/landscape
- Cards responsive
- Typography scalabile
- Supporta rotazione device
- Breakpoints per mobile/tablet

### Esercizio 5: 🔴 Design System

Implementa un mini design system:
- Spacing tokens
- Color palette
- Typography scale
- Component variants (Button, Card, Input)
- Documentation con Storybook o esempi

---

## 📝 Quiz

1. **Differenza principale Flexbox RN vs CSS?**
   - [x] flexDirection default è 'column'
   - [ ] Non c'è differenza
   - [ ] RN non usa Flexbox
   - [ ] Solo row è supportato

2. **Come creare shadow cross-platform?**
   - [ ] Solo shadowColor
   - [x] shadowColor per iOS, elevation per Android
   - [ ] Solo elevation
   - [ ] Non è possibile

3. **useWindowDimensions vs Dimensions.get()?**
   - [ ] Sono identici
   - [x] useWindowDimensions reagisce a rotazioni
   - [ ] Dimensions è deprecato
   - [ ] useWindowDimensions è più lento

4. **Unità di misura in React Native?**
   - [ ] px, em, rem
   - [ ] Solo percentuali
   - [x] dp (density-independent pixels) e %
   - [ ] Tutte le unità CSS

5. **StyleSheet.create vantaggio principale?**
   - [ ] Necessario per funzionare
   - [x] Performance e validazione
   - [ ] Solo per TypeScript
   - [ ] Nessun vantaggio

**Risposte**: 1-a, 2-b, 3-b, 4-c, 5-b

---

## ✅ Checklist

- [ ] StyleSheet.create utilizzato
- [ ] Flexbox padroneggiato
- [ ] Responsive design implementato
- [ ] Theme system creato
- [ ] Platform differences gestite
- [ ] Shadows corrette per entrambe piattaforme
- [ ] Design tokens definiti
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🎨 **StyleSheet.create** per performance
2. 📐 **Flexbox** con flexDirection: 'column' default
3. 📱 **useWindowDimensions** per responsive
4. 🌗 **Theme system** per dark/light mode
5. 🔧 **Platform.select** per differenze iOS/Android
6. 💫 **Shadow iOS** + **Elevation Android**
7. 📏 **Design tokens** per consistenza
8. ⚡ **Evita inline styles dinamici**

---

**Fantastico! 🎉** Ora sai creare UI bellissime e responsive. Prossimo: Navigazione!

[← Precedente: Componenti Core](05_Componenti_Core.md) | [Torna all'Indice](../README.md) | [Prossimo: Navigazione →](07_Navigazione.md)
