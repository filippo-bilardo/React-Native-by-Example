# 05. Expo Go vs Development Builds

[← Precedente: Primo Progetto](04_Primo_Progetto_Expo.md) | [Torna all'Indice](README.md) | [Prossimo: Managed vs Bare Workflow →](06_Managed_vs_Bare_Workflow.md)

---

## Descrizione

Una delle domande più frequenti per chi inizia con Expo è: **"Quando usare Expo Go e quando creare un Development Build?"** In questo capitolo esploreremo in profondità le differenze tra questi due approcci, capiremo le limitazioni di Expo Go, e impareremo quando e come creare Development Builds custom per il tuo progetto.

---

## 🎯 Obiettivi di Apprendimento

- [ ] Comprendere cosa è Expo Go e le sue limitazioni
- [ ] Capire cosa sono i Development Builds
- [ ] Sapere quando usare l'uno vs l'altro
- [ ] Creare un Development Build con EAS Build
- [ ] Configurare custom native modules
- [ ] Gestire multiple Development Builds per team

---

## 📚 Contenuti

1. [Expo Go: Quick Testing Tool](#1-expo-go-quick-testing-tool)
2. [Limitazioni di Expo Go](#2-limitazioni-di-expo-go)
3. [Development Builds: Custom Native Runtime](#3-development-builds-custom-native-runtime)
4. [Quando Usare Expo Go vs Dev Builds](#4-quando-usare-expo-go-vs-dev-builds)
5. [Creare un Development Build](#5-creare-un-development-build)
6. [Workflow con Development Builds](#6-workflow-con-development-builds)

---

## 1. Expo Go: Quick Testing Tool

**📚 Teoria: Cos'è Expo Go**

```
Expo Go = Pre-built app container

Contiene:
✅ Expo SDK completo (50+ modules)
✅ React Native runtime
✅ Metro bundler client
✅ Developer menu

Come funziona:
1. Developer → scrive codice JS/TS
2. Metro bundler → compila JS bundle
3. Expo Go → scarica bundle via WiFi/LAN
4. Expo Go → esegue bundle nel runtime pre-built

Vantaggio: Zero build nativa necessaria!
Limitazione: Solo Expo SDK modules, no custom native code
```

### Architettura Expo Go

```
┌─────────────────────────────────┐
│        Expo Go App              │
│  (Pre-built binary iOS/Android) │
├─────────────────────────────────┤
│   Expo SDK Modules (pre-built)  │
│   - Camera                      │
│   - Location                    │
│   - Notifications               │
│   - FileSystem                  │
│   - ... 50+ modules             │
├─────────────────────────────────┤
│   React Native Runtime          │
│   (JavaScript engine: Hermes)   │
├─────────────────────────────────┤
│   Metro Client                  │
│   (Download & execute JS bundle)│
└─────────────────────────────────┘
           ↓ WiFi/LAN
┌─────────────────────────────────┐
│      Dev Machine                │
│   - Your App Code (JS/TS)       │
│   - Metro Bundler               │
└─────────────────────────────────┘
```

**🎯 Key Point**: Expo Go è un'app **generic container** che può eseguire qualsiasi progetto Expo che usa solo Expo SDK modules.

### Esempio: Codice Compatibile con Expo Go

```typescript
/**
 * ✅ QUESTO CODICE FUNZIONA IN EXPO GO
 * Perché usa solo Expo SDK modules (pre-built in Expo Go)
 */
import React, { useState } from 'react';
import { View, Text, Button, Image } from 'react-native';
import * as ImagePicker from 'expo-image-picker';  // ✅ Expo SDK
import * as Location from 'expo-location';        // ✅ Expo SDK

export default function App() {
  const [image, setImage] = useState<string | null>(null);
  const [location, setLocation] = useState<string>('');

  /**
   * Pick image from gallery
   * ImagePicker è in Expo SDK → funziona in Expo Go
   */
  const pickImage = async () => {
    const result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      quality: 1,
    });
    
    if (!result.canceled) {
      setImage(result.assets[0].uri);
    }
  };

  /**
   * Get current location
   * Location è in Expo SDK → funziona in Expo Go
   */
  const getLocation = async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== 'granted') return;
    
    const loc = await Location.getCurrentPositionAsync({});
    setLocation(`${loc.coords.latitude}, ${loc.coords.longitude}`);
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button title="Pick Image" onPress={pickImage} />
      {image && <Image source={{ uri: image }} style={{ width: 200, height: 200 }} />}
      
      <Button title="Get Location" onPress={getLocation} />
      <Text>{location}</Text>
    </View>
  );
}
```

**Test su Expo Go:**
```bash
$ npx expo start
# → Scan QR code
# → App runs instantly!
# → ImagePicker e Location funzionano! ✅
```

---

## 2. Limitazioni di Expo Go

**📚 Teoria: Cosa NON Può Fare Expo Go**

```
Expo Go contiene SOLO Expo SDK modules.

Se il tuo progetto richiede:
❌ Librerie native third-party (es. react-native-ble-plx per Bluetooth)
❌ Custom native modules (tuo codice Swift/Kotlin)
❌ Config native custom (modifiche a Info.plist, AndroidManifest.xml)
❌ Background tasks advanced (es. background audio player)
❌ Push notifications custom (provider non-FCM)

→ NON funzionerà in Expo Go!
→ Serve Development Build
```

### Esempio: Codice NON Compatibile con Expo Go

```typescript
/**
 * ❌ QUESTO CODICE NON FUNZIONA IN EXPO GO
 * Perché usa libreria native third-party
 */
import React from 'react';
import { View, Button } from 'react-native';
import { BleManager } from 'react-native-ble-plx';  // ❌ NON in Expo SDK

export default function App() {
  const manager = new BleManager();  // ❌ ERROR su Expo Go!

  const scanDevices = () => {
    manager.startDeviceScan(null, null, (error, device) => {
      console.log(device?.name);
    });
  };

  return (
    <View>
      <Button title="Scan Bluetooth" onPress={scanDevices} />
    </View>
  );
}
```

**Cosa succede su Expo Go:**
```bash
$ npx expo start
# → Scan QR code
# → App loading...
# → ERROR: Unable to resolve module 'react-native-ble-plx'
# → Red screen of death 🔴

# Soluzione: Serve Development Build!
```

### Lista Limitazioni Comuni

| Feature | Expo Go | Development Build |
|---------|---------|-------------------|
| **Expo SDK modules** | ✅ Tutti | ✅ Tutti |
| **react-native-ble-plx** (Bluetooth) | ❌ No | ✅ Sì |
| **react-native-background-fetch** | ❌ No | ✅ Sì |
| **react-native-firebase** (analytics/crashlytics) | ❌ No | ✅ Sì |
| **Custom fonts (non-Expo)** | ⚠️ Limitato | ✅ Sì |
| **Modify Info.plist / AndroidManifest** | ❌ No | ✅ Sì |
| **Custom app icon / splash** | ⚠️ Generic | ✅ Custom |
| **Background audio** | ❌ No | ✅ Sì |
| **Code push updates** | ✅ EAS Update | ✅ EAS Update |

---

## 3. Development Builds: Custom Native Runtime

**📚 Teoria: Cosa Sono Development Builds**

```
Development Build = Custom-built Expo Go

È un'app specifica per IL TUO progetto che contiene:
✅ Expo SDK modules (come Expo Go)
✅ TUE librerie native custom
✅ TUE configurazioni native
✅ Metro client (hot reload like Expo Go)

Differenza chiave:
- Expo Go: Generic app per TUTTI i progetti Expo SDK-only
- Dev Build: Custom app per IL TUO progetto specifico

Come Expo Go ma "ejected" con tue customizations!
```

### Architettura Development Build

```
┌─────────────────────────────────┐
│   Your Dev Build App            │
│  (Custom binary per TUO progetto)│
├─────────────────────────────────┤
│   Expo SDK Modules              │
│   + YOUR CUSTOM NATIVE MODULES  │ ← Differenza!
│     - react-native-ble-plx      │
│     - react-native-firebase     │
│     - Your Swift/Kotlin code    │
├─────────────────────────────────┤
│   React Native Runtime          │
│   (Hermes engine)               │
├─────────────────────────────────┤
│   Metro Client                  │
│   (Hot reload like Expo Go!)    │
└─────────────────────────────────┘
           ↓ WiFi/LAN
┌─────────────────────────────────┐
│      Dev Machine                │
│   - Your App Code (JS/TS)       │
│   - Metro Bundler               │
└─────────────────────────────────┘
```

**🔑 Key Insight**: Development Build = Expo Go + tue native customizations, MA mantiene hot reload!

### Vantaggi Development Builds

```
✅ Mantieni fast iteration di Expo Go (hot reload)
✅ Aggiungi librerie native any
✅ Customizza config native
✅ Stesso workflow Expo (EAS Build, EAS Update)
✅ Debug come Expo Go (Chrome DevTools, Flipper)
✅ Team può testare su device senza build locale

Unico svantaggio:
⚠️ Primo build: 15-20 minuti (EAS Build cloud)
⚠️ Rebuild quando cambi native code (non JS!)
```

---

## 4. Quando Usare Expo Go vs Dev Builds

**📚 Decision Tree**

```
Il tuo progetto usa SOLO Expo SDK modules?
(Camera, Location, Notifications, FileSystem, etc.)
├─ SÌ → Expo Go ✅ (instant testing)
└─ NO ↓

Hai librerie native third-party?
(Bluetooth, Firebase, Background tasks, etc.)
├─ SÌ → Development Build 🔧
└─ NO ↓

Devi modificare config native?
(Info.plist, AndroidManifest.xml, entitlements)
├─ SÌ → Development Build 🔧
└─ NO ↓

Vuoi custom app icon/splash in testing?
├─ SÌ → Development Build 🔧
└─ NO → Expo Go ✅
```

### Workflow Consigliato

```
Phase 1: Prototipo (Settimane 1-2)
→ Usa Expo Go
→ Rapid iteration
→ Test UI/UX
→ Validation idea

Phase 2: Development (Settimane 3-8)
→ Aggiungi librerie native se necessario
→ Crea Development Build
→ Team testa con Dev Build
→ Hot reload still works!

Phase 3: Production (Settimana 9+)
→ Production build con EAS Build
→ Deploy a TestFlight / Internal Testing
→ Release a App Store / Google Play
```

### Case Studies

**Caso A: Social Media App**
```
Features:
- Camera (photo/video)
- Location tagging
- Push notifications
- Image upload

Librerie:
✅ expo-camera (Expo SDK)
✅ expo-location (Expo SDK)
✅ expo-notifications (Expo SDK)
✅ expo-image-picker (Expo SDK)

Verdict: Expo Go sufficiente! ✅
```

**Caso B: Fitness Tracker con Bluetooth**
```
Features:
- Bluetooth heart rate monitor
- Background GPS tracking
- Local notifications
- Data sync

Librerie:
❌ react-native-ble-plx (NOT in Expo SDK)
❌ react-native-background-geolocation
✅ expo-notifications (Expo SDK)

Verdict: Development Build necessario! 🔧
```

**Caso C: E-commerce con Firebase Analytics**
```
Features:
- Product catalog
- Cart management
- Firebase Analytics
- Crashlytics

Librerie:
❌ @react-native-firebase/analytics (NOT Expo SDK)
❌ @react-native-firebase/crashlytics
✅ expo-camera (optional, Expo SDK)

Verdict: Development Build per Firebase! 🔧
```

---

## 5. Creare un Development Build

**📚 Step-by-Step Guide**

### Step 1: Setup EAS Account

```bash
# Login Expo account (se non già fatto):
$ npx expo login

# Verifica login:
$ npx expo whoami
# Output: your-username
```

### Step 2: Configure EAS Build

```bash
# Initialize EAS configuration:
$ eas build:configure

# Output:
✔ Select platform › All (iOS and Android)
✔ Created eas.json

# File creato: eas.json
{
  "build": {
    "development": {                      // Development build profile
      "developmentClient": true,         // ← Key: Enable dev client
      "distribution": "internal",        // Internal distribution
      "ios": {
        "simulator": true                // Build per iOS Simulator
      }
    },
    "preview": {                         // Preview build (pre-production)
      "distribution": "internal"
    },
    "production": {                      // Production build
      "autoIncrement": true
    }
  }
}
```

### Step 3: Install expo-dev-client

```bash
# Install development client package:
$ npx expo install expo-dev-client

# Questo package aggiunge:
# - Development menu
# - Error overlay
# - Metro client integration
```

### Step 4: (Opzionale) Aggiungi Custom Native Module

```bash
# Esempio: Install Bluetooth library
$ npm install react-native-ble-plx

# Configure in app.json (se ha config plugin):
{
  "expo": {
    "plugins": [
      [
        "react-native-ble-plx",
        {
          "isBackgroundEnabled": true,
          "modes": ["peripheral", "central"],
          "bluetoothAlwaysPermission": "Allow app to use Bluetooth"
        }
      ]
    ]
  }
}
```

### Step 5: Build Development Build

```bash
# Build per iOS Simulator (macOS only):
$ eas build --profile development --platform ios

# Build per Android Emulator:
$ eas build --profile development --platform android

# Build per iOS Device (physical):
$ eas build --profile development --platform ios --clear-cache

# Build per Android Device:
$ eas build --profile development --platform android

# Output:
✔ Build type: development
✔ Uploading project to EAS Build...
✔ Queued build...

Build details: https://expo.dev/accounts/yourname/projects/yourapp/builds/abc123

Waiting for build to complete...
Build in progress... (ETA: 15-20 minutes)

✔ Build finished!
Download: https://expo.dev/accounts/yourname/projects/yourapp/builds/abc123/download
```

**⏱️ Tempo build:**
- iOS: 15-20 minuti
- Android: 10-15 minuti
- Primo build più lento (dependencies download)
- Builds successivi: cache aiuta (~10 min)

### Step 6: Install Development Build

**iOS Simulator (macOS):**
```bash
# Download .app file dal link EAS
# Drag & drop su iOS Simulator
# Oppure:
$ eas build:run -p ios
```

**Android Emulator:**
```bash
# Download .apk dal link EAS
# Drag & drop su Android Emulator
# Oppure:
$ eas build:run -p android
```

**iOS Device (Physical):**
```bash
# Opzione 1: TestFlight (recommended)
# EAS genera IPA → upload automatico TestFlight
# User installa da TestFlight app

# Opzione 2: Development Provisioning
# Registra device UUID in Apple Developer
# Build con development profile
# Install via Xcode / Configurator
```

**Android Device:**
```bash
# Download .apk da link EAS
# Send via email/Slack/AirDrop
# Open on device → Install
# (Enable "Unknown sources" if needed)
```

---

## 6. Workflow con Development Builds

**📚 Development con Dev Build**

### Day-to-Day Workflow

```bash
# 1. Start Metro bundler:
$ npx expo start --dev-client

# Output:
› Metro waiting on exp://192.168.1.10:8081
› For development builds only
› Scan QR code with your Development Build app

# 2. Open Development Build app su device
# (NON Expo Go, ma TUA app custom!)

# 3. Scan QR code o insert URL

# 4. App loads from Metro (hot reload enabled!)

# 5. Edit code → Save → Hot reload! 🔥
# Exactly like Expo Go workflow!
```

**🎯 Key Point**: Una volta installato Development Build, workflow è identico a Expo Go (hot reload, instant updates), ma con TUE customizations native!

### Quando Serve Rebuild

```
✅ NO rebuild necessario per:
- Modifiche JS/TS code
- Modifiche styles
- Nuovi components React
- Business logic changes
- API integrations
- Nuovi Expo SDK modules (se già inclusi in build)

❌ Rebuild necessario per:
- Nuovo native module installato
- Modifiche a app.json "plugins" config
- Modifiche a Info.plist / AndroidManifest.xml
- Upgrade Expo SDK version
- Modifiche a native code (Swift/Kotlin)

Rebuild command:
$ eas build --profile development --platform all
```

### Team Workflow con Development Builds

```
Scenario: Team di 5 developer

Opzione A: Everyone builds locally
- ❌ Ogni developer serve Mac per iOS
- ❌ Tempo setup: 2-4 ore per developer
- ❌ Build errors per environment differences

Opzione B: EAS Build cloud (RECOMMENDED)
- ✅ 1 developer crea build con EAS
- ✅ Share .apk / TestFlight link al team
- ✅ Everyone installa Development Build
- ✅ Everyone può sviluppare con hot reload
- ✅ New native module? 1 developer rebuilda, reshare

Steps:
1. Developer A: $ eas build --profile development --platform all
2. Developer A: Share download links in Slack
3. Team: Install Development Build
4. Team: $ npx expo start --dev-client (ognuno sul suo branch)
5. Team: Hot reload works for everyone! 🎉
```

### Multiple Development Builds

```bash
# Scenario: Feature branches con diverse native dependencies

# Branch A: Bluetooth feature
# app.json plugins: ["react-native-ble-plx"]
$ git checkout feature/bluetooth
$ eas build --profile development --platform android
# → Dev Build A (con Bluetooth)

# Branch B: Firebase feature
# app.json plugins: ["@react-native-firebase/app"]
$ git checkout feature/firebase
$ eas build --profile development --platform android
# → Dev Build B (con Firebase)

# Developer può avere ENTRAMBE le app installate!
# (Serve configurare different bundle identifiers)
```

---

## ✅ Checklist di Completamento

- [ ] Comprendo differenze Expo Go vs Development Builds
- [ ] So quando serve Development Build
- [ ] Ho configurato EAS Build (eas.json)
- [ ] Ho creato un Development Build di test
- [ ] Ho testato hot reload con Development Build
- [ ] Comprendo quando serve rebuild vs hot reload
- [ ] So come distribuire Development Build al team

---

## 📌 Punti Chiave da Ricordare

1. 📱 **Expo Go**: App generic, solo Expo SDK, instant testing
2. 🔧 **Development Build**: App custom, tue native deps, mantiene hot reload
3. ⚡ **Rebuild**: Solo quando cambi native code/config
4. ☁️ **EAS Build**: Cloud build, no Mac per iOS needed
5. 👥 **Team**: 1 build, share a tutti, everyone hot reloads
6. 🎯 **Start**: Expo Go per proto, Dev Build quando serve native
7. 🚀 **Production**: Stesso EAS workflow per production builds

---

[Prossimo: 06. Managed vs Bare Workflow →](06_Managed_vs_Bare_Workflow.md)

[← Precedente: Primo Progetto](04_Primo_Progetto_Expo.md) | [Torna all'Indice](README.md)
