# 04. Primo Progetto Expo - Hello World

[← Precedente: Setup Ambiente](03_Setup_Ambiente_Expo.md) | [Torna all'Indice](README.md) | [Prossimo: Expo Go vs Dev Builds →](05_Expo_Go_vs_Development_Builds.md)

---

## Descrizione

È il momento di **creare il tuo primo progetto Expo** e vedere tutto il workflow in azione! In questo capitolo creeremo un'app "Hello World" step-by-step, esploreremo la struttura del progetto, modificheremo il codice con hot reload, e capiremo come funziona il development flow Expo.

---

## 🎯 Obiettivi di Apprendimento

- [ ] Creare un nuovo progetto Expo da zero
- [ ] Comprendere la struttura file di un progetto Expo
- [ ] Avviare il dev server Metro bundler
- [ ] Testare app su device con Expo Go
- [ ] Modificare codice e vedere hot reload in azione
- [ ] Comprendere il file app.json e le configurazioni base
- [ ] Navigare tra screens basic con React Navigation

---

## 📚 Contenuti

1. [Creazione Progetto](#1-creazione-progetto)
2. [Struttura File Progetto](#2-struttura-file-progetto)
3. [Avvio Dev Server](#3-avvio-dev-server)
4. [Hello World Component](#4-hello-world-component)
5. [Hot Reload Workflow](#5-hot-reload-workflow)
6. [Configurazione app.json](#6-configurazione-appjson)
7. [Aggiungere Navigazione Base](#7-aggiungere-navigazione-base)

---

## 1. Creazione Progetto

**📚 Teoria: create-expo-app**

```
create-expo-app = CLI tool per scaffolding progetto Expo

Template disponibili:
- blank: Minimal setup (recommended per learning)
- blank-typescript: TypeScript pre-configurato
- tabs: Bottom tab navigation template
- bare-minimum: Bare workflow (native folders)

Noi useremo: blank-typescript (best practice)
```

### Comando Creazione

```bash
# Naviga nella cartella progetti:
$ cd ~/Projects

# Crea nuovo progetto Expo:
$ npx create-expo-app HelloExpo --template blank-typescript

# Output:
✔ Downloaded and extracted project files.
📦 Installing dependencies...

yarn install v1.22.19
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
[3/4] 🔗  Linking dependencies...
[4/4] 🔨  Building fresh packages...
✨ Done in 45.23s.

✅ Your project is ready!

To run your project, navigate to the directory and run:
  cd HelloExpo
  npx expo start

# Tempo: 1-2 minuti (download dependencies)
```

**💡 Cosa è successo:**
1. Download template "blank-typescript" da GitHub
2. Copia file nel folder HelloExpo/
3. Inizializza Git repository
4. Installa dependencies (React Native, Expo SDK, TypeScript)
5. Setup basic configuration

### Struttura Creata

```bash
$ cd HelloExpo
$ tree -L 1

HelloExpo/
├── .expo/              # Expo cache (ignore)
├── .git/               # Git repository
├── assets/             # Images, fonts, icons
├── node_modules/       # Dependencies (ignore)
├── .gitignore          # Git ignore rules
├── App.tsx             # Main app component ⭐
├── app.json            # Expo configuration ⭐
├── babel.config.js     # Babel config (transpiler JS)
├── package.json        # npm dependencies ⭐
├── tsconfig.json       # TypeScript config
└── README.md           # Project docs
```

---

## 2. Struttura File Progetto

**📚 Analisi File Principali**

### A) package.json

```json
{
  "name": "helloexpo",
  "version": "1.0.0",
  "main": "expo/AppEntry.js",  // Entry point (gestito da Expo)
  "scripts": {
    "start": "expo start",     // Dev server
    "android": "expo start --android",  // Launch Android
    "ios": "expo start --ios",          // Launch iOS
    "web": "expo start --web"           // Launch web (se configurato)
  },
  "dependencies": {
    "expo": "~50.0.0",                  // Expo SDK
    "expo-status-bar": "~1.11.0",       // StatusBar component
    "react": "18.2.0",                  // React core
    "react-native": "0.73.0"            // React Native core
  },
  "devDependencies": {
    "@babel/core": "^7.20.0",
    "@types/react": "~18.2.45",         // TypeScript types
    "typescript": "^5.1.3"
  }
}
```

**🎯 Key Points:**
- `expo` version definisce SDK version (50.0.0 = latest al momento)
- `react-native` version è gestita automaticamente da Expo (no manual upgrade!)
- Scripts predefiniti per launch su platforms diverse

### B) app.json

```json
{
  "expo": {
    "name": "HelloExpo",               // App name (displayed)
    "slug": "helloexpo",               // Unique identifier
    "version": "1.0.0",                // App version
    "orientation": "portrait",         // Lock orientation
    "icon": "./assets/icon.png",       // App icon (1024x1024)
    "userInterfaceStyle": "light",     // Light/dark/automatic
    "splash": {                        // Splash screen config
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"                           // Include all assets in bundle
    ],
    "ios": {                           // iOS-specific config
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.helloexpo"
    },
    "android": {                       // Android-specific config
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.yourcompany.helloexpo"
    },
    "web": {                           // Web-specific (optional)
      "favicon": "./assets/favicon.png"
    }
  }
}
```

**🔑 Importanza app.json:**
- Configura metadati app (name, version, icon)
- Platform-specific settings (iOS/Android/Web)
- Build configurations
- Permissions (aggiungeremo dopo per Camera/Location)

### C) App.tsx (Main Component)

```typescript
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      {/* Main content */}
      <Text>Open up App.tsx to start working on your app!</Text>
      
      {/* StatusBar component (Expo SDK) */}
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,                    // Fill entire screen
    backgroundColor: '#fff',
    alignItems: 'center',       // Center horizontally
    justifyContent: 'center',   // Center vertically
  },
});
```

**📚 Componenti usati:**
- `View`: Container (equivalente a `<div>` in web)
- `Text`: Text display (unico modo per mostrare testo in RN)
- `StyleSheet`: API per definire stili (CSS-like)
- `StatusBar` (da Expo SDK): Controlla barra stato (orario, batteria, network)

### D) tsconfig.json (TypeScript)

```json
{
  "extends": "expo/tsconfig.base",  // Inherit Expo defaults
  "compilerOptions": {
    "strict": true,                 // Strict type checking
    "jsx": "react-native"           // JSX for React Native
  }
}
```

---

## 3. Avvio Dev Server

**📚 Teoria: Metro Bundler**

```
Metro = JavaScript bundler per React Native

Cosa fa:
1. Legge file sorgenti (App.tsx, components, etc.)
2. Transpila TypeScript → JavaScript
3. Bundla tutto in single JS file
4. Serve bundle a Expo Go / device
5. Monitora file changes → hot reload

Simile a: Webpack (web), Vite, Parcel
```

### Start Development Server

```bash
# Avvia dev server:
$ npx expo start

# Output:
Starting Metro Bundler
›
› Metro waiting on exp://192.168.1.10:8081
›
› QR Code:
  ┌─────────────────────────────┐
  │  ████ ████ █████  ███  ████ │
  │  █    ▒    █  █  █   █ ▒  █ │
  │  ███  ████ ████  █▒▒▒█ ████ │
  │  █    ▒    █     █   █ ▒  █ │
  │  ████ ████ █     █   █ ▒  █ │
  └─────────────────────────────┘
›
› Press a │ open Android
› Press i │ open iOS simulator
› Press w │ open web
› Press r │ reload app
› Press m │ toggle menu
› Press ? │ show all commands

Logs from devices will appear here:
```

**🎯 Comandi Interattivi:**
- `a`: Launch su Android emulator/device
- `i`: Launch su iOS Simulator (solo macOS)
- `w`: Launch su browser (web support)
- `r`: Reload app manualmente
- `m`: Apri developer menu su device
- `j`: Apri Chrome DevTools (debug JS)
- `Ctrl+C`: Stop server

### Connetti Device con Expo Go

**Metodo 1: Scansiona QR Code**

```bash
# Su device:
1. Apri Expo Go app
2. Tap "Scan QR Code"
3. Point camera al QR nel terminal
4. App si carica (5-10 secondi primo load)

# Terminal mostra:
› Opening on iPhone di Mario...
 LOG  Running "HelloExpo" with {"rootTag":1}

# Su device: Vedi "Open up App.tsx to start working on your app!"
```

**Metodo 2: URL Manuale**

```bash
# Copia URL da terminal:
exp://192.168.1.10:8081

# Su Expo Go:
1. Tap "Enter URL manually"
2. Paste URL
3. Connect
```

**Metodo 3: Development Build (se Expo Go non sufficiente)**

```bash
# Crea development build (vedremo nel Cap. 05):
$ eas build --profile development --platform ios
```

---

## 4. Hello World Component

**📚 Modifichiamo App.tsx con Componenti Base**

### Versione 1: Testo e Stile

```typescript
// App.tsx
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      {/* Title */}
      <Text style={styles.title}>
        Hello, Expo! 👋
      </Text>
      
      {/* Subtitle */}
      <Text style={styles.subtitle}>
        Welcome to React Native with Expo
      </Text>
      
      {/* Description */}
      <Text style={styles.description}>
        Edit App.tsx and save to see changes instantly!
      </Text>
      
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,  // Fill screen
    backgroundColor: '#f0f0f0',  // Light gray background
    alignItems: 'center',         // Horizontal center
    justifyContent: 'center',     // Vertical center
    padding: 20,                  // Padding all sides
  },
  title: {
    fontSize: 32,                 // Font size in pixels
    fontWeight: 'bold',           // Bold text
    color: '#333',                // Dark gray text
    marginBottom: 10,             // Space below
  },
  subtitle: {
    fontSize: 18,
    color: '#666',
    marginBottom: 20,
  },
  description: {
    fontSize: 14,
    color: '#999',
    textAlign: 'center',          // Center text inside Text component
    lineHeight: 20,               // Line height (spacing between lines)
  },
});
```

**Salva file → Guarda device: App aggiornata automaticamente! 🔥**

### Versione 2: Aggiungiamo Button Interattivo

```typescript
// App.tsx
import { StatusBar } from 'expo-status-bar';
import { 
  StyleSheet, 
  Text, 
  View, 
  Button,      // Button component
  Alert        // Alert API
} from 'react-native';

export default function App() {
  /**
   * Handler function per button press
   */
  const handlePress = () => {
    // Mostra native alert dialog
    Alert.alert(
      'Button Pressed!',           // Title
      'You tapped the button',     // Message
      [{ text: 'OK' }]             // Buttons
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello, Expo! 👋</Text>
      
      <Text style={styles.subtitle}>
        Welcome to React Native with Expo
      </Text>
      
      {/* Button component */}
      <Button 
        title="Press Me"           // Button text
        onPress={handlePress}      // Handler function
        color="#007AFF"            // iOS blue
      />
      
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f0f0f0',
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 18,
    color: '#666',
    marginBottom: 30,  // Spazio prima del button
    textAlign: 'center',
  },
});
```

**Testa su device:**
1. Tap "Press Me" button
2. Vedi native alert dialog
3. Tap "OK" → dialog si chiude

### Versione 3: State con useState Hook

```typescript
// App.tsx
import { StatusBar } from 'expo-status-bar';
import { useState } from 'react';  // React hook per state
import { 
  StyleSheet, 
  Text, 
  View, 
  TouchableOpacity  // Touchable button (more customizable than Button)
} from 'react-native';

export default function App() {
  /**
   * State: counter inizia a 0
   * setCounter: funzione per aggiornare counter
   */
  const [counter, setCounter] = useState(0);

  /**
   * Increment counter
   */
  const increment = () => {
    setCounter(counter + 1);  // Update state → re-render
  };

  /**
   * Reset counter
   */
  const reset = () => {
    setCounter(0);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Counter App</Text>
      
      {/* Display counter value */}
      <Text style={styles.counter}>{counter}</Text>
      
      {/* Increment button */}
      <TouchableOpacity 
        style={styles.button}
        onPress={increment}
      >
        <Text style={styles.buttonText}>Increment +1</Text>
      </TouchableOpacity>
      
      {/* Reset button */}
      <TouchableOpacity 
        style={[styles.button, styles.resetButton]}  // Array styles
        onPress={reset}
      >
        <Text style={styles.buttonText}>Reset</Text>
      </TouchableOpacity>
      
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    color: '#333',
  },
  counter: {
    fontSize: 72,              // Large font
    fontWeight: 'bold',
    color: '#007AFF',
    marginBottom: 40,
  },
  button: {
    backgroundColor: '#007AFF',  // iOS blue
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 10,            // Rounded corners
    marginVertical: 10,          // Vertical spacing
    minWidth: 200,               // Minimum width
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#FF3B30',  // iOS red
  },
  buttonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

**🎓 Concetti React:**
- **useState**: Hook per gestire state locale
- **Re-render**: Quando state cambia, component si re-renderizza
- **Event handlers**: Funzioni passate a onPress
- **Style arrays**: `[style1, style2]` per combinare stili

---

## 5. Hot Reload Workflow

**📚 Teoria: Hot Reload vs Fast Refresh**

```
Hot Reload (legacy):
- Reload entire app quando file cambia
- State viene perso
- Lento (3-5 secondi)

Fast Refresh (moderno, default Expo):
- Reload SOLO component modificato
- State viene preservato ⭐
- Velocissimo (1-2 secondi)
```

### Test Fast Refresh

```typescript
// App.tsx - Modifica counter incremento
const increment = () => {
  setCounter(counter + 2);  // ← Cambia +1 a +2
};

// Salva file (Cmd+S / Ctrl+S)
// → Device: App reload in 1 secondo
// → Counter value preservato! (se era 5, rimane 5)
```

**💡 Fast Refresh preserva:**
- ✅ useState state
- ✅ useRef refs
- ✅ Navigation state
- ❌ useEffect side effects (richiamati)
- ❌ Global variables

### Quando Fast Refresh Fallisce

```typescript
// ❌ Syntax error:
const increment = () => {
  setCounter(counter + 2  // Missing closing )
};

// Terminal mostra error:
 ERROR  SyntaxError: Unexpected token (7:2)
 
// Device: Red screen error con stack trace
// Fix error → Save → Auto-reload
```

**Se Fast Refresh non funziona:**
```bash
# Opzione 1: Reload manuale
# Press 'r' nel terminal

# Opzione 2: Shake device → Developer Menu → Reload

# Opzione 3: Restart dev server
# Ctrl+C → npx expo start
```

---

## 6. Configurazione app.json

**📚 Modifichiamo app.json per Personalizzare App**

### Cambio Nome App

```json
{
  "expo": {
    "name": "My Counter App",        // ← Nome visualizzato
    "slug": "my-counter-app",        // ← URL slug (unique)
    "version": "1.0.0",
    // ... rest
  }
}
```

**⚠️ Dopo modifica app.json:**
```bash
# Stop dev server (Ctrl+C)
# Restart:
$ npx expo start

# Shake device → Reload
# → Nome app aggiornato nella home Expo Go
```

### Aggiungi Custom Icon

```json
{
  "expo": {
    "icon": "./assets/icon.png",     // 1024x1024 PNG
    "splash": {
      "image": "./assets/splash.png", // 1242x2436 PNG (iPhone X)
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    // ...
  }
}
```

**Per generare icon:**
1. Design icon 1024x1024 (Figma, Photoshop, Canva)
2. Salva come `assets/icon.png`
3. Restart dev server
4. Rebuild app → Icon aggiornato

### Orientazione Screen

```json
{
  "expo": {
    "orientation": "portrait",  // Lock portrait
    // Opzioni:
    // - "portrait": Solo verticale
    // - "landscape": Solo orizzontale
    // - "default": Auto-rotate (default)
  }
}
```

---

## 7. Aggiungere Navigazione Base

**📚 Teoria: Screens e Navigation**

```
Single screen app = Limitata
Multi-screen app = Navigation library

React Navigation (industry standard):
- Stack navigation (push/pop screens)
- Tab navigation (bottom tabs)
- Drawer navigation (sidebar)

Installation: Expo ha template con navigation pre-configurata
```

### Installazione React Navigation

```bash
# Install core:
$ npx expo install @react-navigation/native

# Install dependencies per Expo:
$ npx expo install react-native-screens react-native-safe-area-context

# Install stack navigator:
$ npx expo install @react-navigation/native-stack
```

### Esempio: Due Screens con Stack Navigation

```typescript
// App.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { 
  StyleSheet, 
  Text, 
  View, 
  TouchableOpacity 
} from 'react-native';

/**
 * Stack navigator instance
 */
const Stack = createNativeStackNavigator();

/**
 * Home Screen component
 */
function HomeScreen({ navigation }: any) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home Screen</Text>
      
      <TouchableOpacity 
        style={styles.button}
        onPress={() => navigation.navigate('Details')}  // Navigate to Details
      >
        <Text style={styles.buttonText}>Go to Details</Text>
      </TouchableOpacity>
    </View>
  );
}

/**
 * Details Screen component
 */
function DetailsScreen({ navigation }: any) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Details Screen</Text>
      
      <TouchableOpacity 
        style={styles.button}
        onPress={() => navigation.goBack()}  // Go back to previous screen
      >
        <Text style={styles.buttonText}>Go Back</Text>
      </TouchableOpacity>
    </View>
  );
}

/**
 * Main App with Navigation
 */
export default function App() {
  return (
    <NavigationContainer>
      {/* Stack Navigator con 2 screens */}
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen 
          name="Home" 
          component={HomeScreen}
          options={{ title: 'Home' }}  // Header title
        />
        <Stack.Screen 
          name="Details" 
          component={DetailsScreen}
          options={{ title: 'Details' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

**🎯 Test Navigation:**
1. Start app → Vedi "Home Screen"
2. Tap "Go to Details" → Slide animation, vedi "Details Screen"
3. Tap "Go Back" OR swipe from left → Torna a Home
4. Header navigation automatico (back button iOS/Android)

---

## ✅ Checklist di Completamento

- [ ] Creato progetto Expo con create-expo-app
- [ ] Compresa struttura file (App.tsx, app.json, package.json)
- [ ] Avviato dev server Metro bundler
- [ ] Testato app su device con Expo Go
- [ ] Modificato codice e visto fast refresh
- [ ] Usato useState per state management
- [ ] Configurato app.json (nome, icon, orientazione)
- [ ] Aggiunto navigation base con React Navigation

---

## 📌 Punti Chiave da Ricordare

1. 🚀 **create-expo-app**: Crea progetto in 1-2 minuti
2. 📱 **Expo Go**: Test su device senza build
3. 🔥 **Fast Refresh**: Modifica codice → reload in 1-2 sec
4. ⚛️ **useState**: Hook per state management React
5. 🎨 **StyleSheet**: CSS-like styling per React Native
6. 🧭 **React Navigation**: Library standard per navigation
7. ⚙️ **app.json**: Configurazione app (nome, icon, permissions)
8. 📦 **Metro**: Bundler JavaScript per React Native

---

[Prossimo: 05. Expo Go vs Development Builds →](05_Expo_Go_vs_Development_Builds.md)

[← Precedente: Setup Ambiente](03_Setup_Ambiente_Expo.md) | [Torna all'Indice](README.md)
