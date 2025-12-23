# 02. Expo - Introduzione e Architettura

[← Precedente: Panorama Mobile](01_Panorama_Mobile_Moderno.md) | [Torna all'Indice](README.md) | [Prossimo: Setup Ambiente →](03_Setup_Ambiente_Expo.md)

---

## Descrizione

Ora che comprendiamo il panorama dello sviluppo mobile, entriamo nel mondo **Expo**. Cos'è esattamente Expo? Come si integra con React Native? Quali problemi risolve e quali limitazioni introduce?

In questo capitolo esploreremo l'architettura di Expo, il suo ecosistema (SDK, EAS, Expo Go), e capiremo quando e perché sceglierlo rispetto a React Native CLI puro.

---

## 🎯 Obiettivi di Apprendimento

- [ ] Comprendere cos'è Expo e la sua relazione con React Native
- [ ] Conoscere l'architettura e i componenti dell'ecosistema Expo
- [ ] Capire le differenze tra Expo e React Native CLI
- [ ] Identificare i vantaggi e le limitazioni di Expo
- [ ] Sapere quando scegliere Expo vs React Native CLI
- [ ] Conoscere il modello di business e pricing di Expo

---

## 📚 Contenuti

1. [Cos'è Expo](#1-cosè-expo)
2. [Architettura dell'Ecosistema Expo](#2-architettura-dellecosistema-expo)
3. [Expo vs React Native CLI](#3-expo-vs-react-native-cli)
4. [Quando Usare Expo](#4-quando-usare-expo)
5. [Modello di Business e Pricing](#5-modello-di-business-e-pricing)

---

## 1. Cos'è Expo

**📚 Teoria: Expo in 3 Definizioni**

### Definizione 1: Framework e Toolchain

```
Expo = React Native + Developer Experience++

React Native bare:
- Setup manuale Xcode/Android Studio
- Configurazione build manuale  
- Native modules da installare manualmente
- Build locale (serve Mac per iOS)

Expo:
- Setup automatico con create-expo-app
- Build configurate out-of-the-box
- 50+ native modules pre-installati (Expo SDK)
- Build cloud (EAS Build, iOS senza Mac!)
```

**Expo è un layer sopra React Native che rimuove la complessità native.**

### Definizione 2: Piattaforma di Servizi Cloud

```
Expo Application Services (EAS):

┌─────────────────────────────────┐
│        EAS Build                │  ← Cloud build per iOS/Android
├─────────────────────────────────┤
│        EAS Update               │  ← OTA updates (senza app store)
├─────────────────────────────────┤
│        EAS Submit               │  ← Submit automatico a stores
└─────────────────────────────────┘
```

**Expo offre infrastruttura cloud per build, deployment e updates.**

### Definizione 3: Ecosistema e Community

- **Expo SDK**: Librerie native pre-configurate (Camera, Location, Notifications, etc.)
- **Expo Router**: File-based navigation (come Next.js)
- **Expo Go**: App per testare su device senza build
- **Expo Snack**: Playground online per React Native
- **Community**: Forum, Discord, documentazione eccellente

---

## 2. Architettura dell'Ecosistema Expo

**📚 Teoria: I Pilastri di Expo**

```
Ecosistema Expo:

┌─────────────────────────────────────────────────┐
│                   TUA APP                       │
│            (React + TypeScript)                 │
└────────────────┬────────────────────────────────┘
                 │
    ┌────────────┴───────────┐
    │                        │
┌───▼───────────────┐   ┌────▼──────────────┐
│   Expo SDK        │   │  React Native     │
│  (API Native)     │   │  (Core Framework) │
└───────────────────┘   └───────────────────┘
         │                       │
         └───────────┬───────────┘
                     │
         ┌───────────▼────────────┐
         │  Native Platform       │
         │  (iOS / Android)       │
         └────────────────────────┘

EAS Services (Cloud):
┌────────────────────┐
│    EAS Build       │ ← Build remoto
├────────────────────┤
│    EAS Update      │ ← OTA updates
├────────────────────┤
│    EAS Submit      │ ← Submit stores
└────────────────────┘
```

### A) Expo SDK

**Collezione di 50+ librerie native pre-configurate:**

```typescript
/**
 * EXPO SDK: API Native Ready-to-Use
 */
import * as Camera from 'expo-camera';           // 📷 Camera + video
import * as Location from 'expo-location';       // 📍 GPS + geofencing
import * as Notifications from 'expo-notifications'; // 🔔 Push notifications
import * as ImagePicker from 'expo-image-picker'; // 🖼️ Gallery picker
import * as FileSystem from 'expo-file-system';  // 📁 File operations
import * as Contacts from 'expo-contacts';       // 👥 Address book
import * as Calendar from 'expo-calendar';       // 📅 Calendar events
import * as SecureStore from 'expo-secure-store';// 🔐 Encrypted storage
// ... 50+ moduli disponibili!

/**
 * ESEMPIO: Usare Camera (zero configurazione native!)
 */
const App = () => {
  const [permission, requestPermission] = Camera.useCameraPermissions();
  
  const takePicture = async () => {
    if (!permission?.granted) {
      // Request permission
      await requestPermission();
    }
    
    // Use camera (API cross-platform)
    const result = await Camera.launchCameraAsync({
      mediaTypes: Camera.MediaTypeOptions.Images,
      quality: 0.8,
    });
    
    if (!result.canceled) {
      console.log('Photo URI:', result.assets[0].uri);
    }
  };
  
  return (
    <Button title="Take Photo" onPress={takePicture} />
  );
};
```

**🎯 Key Point**: Con Expo SDK, API native complesse (Camera, Location) sono 1-2 righe di codice, senza toccare Xcode/Android Studio!

### B) EAS (Expo Application Services)

**Cloud platform per sviluppo mobile professionale**

```
EAS = Expo Application Services
Platform-as-a-Service per React Native/Expo apps

┌─────────────────────────────────────────┐
│           EAS PLATFORM                  │
├─────────────────────────────────────────┤
│  EAS Build   │ Cloud build system       │
│  EAS Submit  │ App store automation     │
│  EAS Update  │ OTA update system        │
│  EAS Metadata│ Store listing management │
└─────────────────────────────────────────┘
```

**Pensa a EAS come "Vercel/Netlify per mobile apps"**
- Zero infrastruttura da gestire
- Push code → Get built app
- Continuous deployment automatico
- Rollback updates in un click

**EAS risolve i pain points più grandi del mobile development:**
1. ❌ "Serve un Mac per build iOS" → ✅ Build cloud su macOS
2. ❌ "Setup Xcode/Android Studio è un incubo" → ✅ Zero setup native
3. ❌ "App store review richiede giorni" → ✅ OTA updates istantanei
4. ❌ "CI/CD per mobile è complesso" → ✅ Build automatiche integrate

**Architettura EAS:**

```
Developer Workflow con EAS:

┌─────────────┐
│  Developer  │
│  git push   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────┐
│   EAS Build Triggers        │
│   (webhook da GitHub)       │
└──────┬──────────────────────┘
       │
       ▼
┌─────────────────────────────┐
│  Cloud Build Machines       │
│  • macOS (for iOS)          │
│  • Linux (for Android)      │
│                             │
│  Auto install:              │
│  - npm dependencies         │
│  - CocoaPods (iOS)          │
│  - Android SDK              │
│  - Certificates             │
└──────┬──────────────────────┘
       │
       ▼
┌─────────────────────────────┐
│  Build Artifacts            │
│  • IPA (iOS)                │
│  • APK/AAB (Android)        │
│  • Source maps              │
└──────┬──────────────────────┘
       │
       ├──► EAS Submit ──► App Store / Google Play
       │
       └──► Developer (download link)
```


#### EAS Build

```bash
# Build iOS senza avere un Mac!
$ eas build --platform ios --profile production

# Expo cloud fa:
# 1. Setup macOS machine automaticamente
# 2. Installa dipendenze (npm, CocoaPods)
# 3. Compila app con Xcode
# 4. Genera IPA file
# 5. Ti manda link download

# Tempo: ~15-20 minuti
# Costo: Free tier = 30 build/mese
```

#### EAS Update

```bash
# Publish update Over-The-Air (senza app store review!)
$ eas update --branch production

# User apre app → download automatico nuovo JS bundle
# Tempo deploy: ~5 minuti (vs giorni con app store review)
# Perfetto per: bug fixes, contenuti, UI tweaks
# ⚠️ NON per: native code changes (serve rebuild)
```

#### EAS Submit

```bash
# Submit automatico a App Store e Google Play
$ eas submit --platform ios
$ eas submit --platform android

# Expo gestisce:
# - Upload binari
# - Metadata (descrizione, screenshots)
# - Certificates e provisioning
```

### C) Expo Go

**App per testing rapido su device reale**

Expo Go è l'app mobile (disponibile su App Store e Google Play) che trasforma il tuo smartphone in un device di test istantaneo, senza bisogno di build native.

#### Come Funziona

```
Flusso Development con Expo Go:

Developer Machine                    Device (Expo Go App)
┌─────────────────┐                 ┌──────────────────┐
│  1. Edit code   │                 │                  │
│     in VSCode   │                 │   Expo Go App    │
└────────┬────────┘                 │   (in attesa)    │
         │                          └──────────────────┘
         ▼
┌─────────────────┐
│  2. Metro       │
│     Bundler     │  ◄─── File watcher
│     rebuilds    │       (auto-rebuild)
└────────┬────────┘
         │
         ▼ (WiFi/LAN)
┌─────────────────┐
│  3. Push update │ ───────────────► ┌──────────────────┐
│     to device   │                  │  App hot-reloads │
└─────────────────┘                  │  in 1-2 seconds  │
                                     └──────────────────┘

Tempo totale: Save → Refresh = 1-3 secondi!
```

**Esempio Pratico:**

```typescript
// App.tsx
import { Text, View } from 'react-native';

export default function App() {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Hello from Expo Go!</Text>
      {/* Cambia questo testo, salva → app si aggiorna subito su device */}
    </View>
  );
}
```

**Steps:**
1. `npx expo start` su laptop
2. Scansiona QR code con Expo Go app
3. Edit `App.tsx` → salva → vedi cambiamenti istantaneamente!

#### Architettura Tecnica

```
Expo Go contiene:

┌────────────────────────────────────────┐
│         EXPO GO APP                    │
├────────────────────────────────────────┤
│  • React Native runtime (embedded)     │
│  • Tutto l'Expo SDK (50+ modules)      │
│  • Metro bundler client                │
│  • JavaScript engine (Hermes/JSC)      │
│  • Bridge per native modules           │
└────────────────────────────────────────┘

Quando carichi la tua app:
- Expo Go scarica solo il tuo JavaScript bundle
- Usa i native modules già embedded nell'app
- Esegue il tuo codice JS nell'engine esistente

Risultato: No rebuild nativa, instant feedback!
```

#### Vantaggi di Expo Go

**✅ Perfetta per:**

1. **Prototipazione Velocissima**
```typescript
// Puoi testare idee in minuti, non ore
import { Camera } from 'expo-camera';
import { Location } from 'expo-location';

// Instant test di features native complesse
// senza compilare nulla!
```

2. **Demo a Stakeholders**
```bash
# Scenario: Meeting con cliente
$ npx expo start
# Cliente scansiona QR → vede app subito
# Zero "funziona sulla mia macchina" issues
```

3. **UI Development**
```typescript
// Iterazione rapida su styling e layout
<View style={{ 
  backgroundColor: 'blue', // Cambia → salva → vedi subito
  padding: 20 
}}>
```

4. **Testing Multi-Device**
```
Stesso QR code scansionato da:
- iPhone 15 Pro
- Pixel 8
- iPad Air
- Samsung Galaxy Tab

Tutti testano la stessa versione istantaneamente!
```

5. **Cross-Platform Verification**
```typescript
// Testa comportamento iOS vs Android al volo
import { Platform } from 'react-native';

const behavior = Platform.select({
  ios: 'iOS behavior',
  android: 'Android behavior',
});

// Salva → vedi differenze su entrambi i device simultaneamente
```

#### Limitazioni di Expo Go

**❌ NON supporta:**

1. **Native Modules Custom**
```bash
# Questa libreria Bluetooth NON funziona in Expo Go
$ npm install react-native-ble-plx

# Error: Module not found
# Expo Go non ha questo module embedded
```

2. **Background Tasks Avanzati**
```typescript
// Background audio/video processing complesso
// Background location tracking continuo
// Push notification handlers custom

// Richiedono native configuration non in Expo Go
```

3. **Deep Native Customizations**
```xml
<!-- Modifiche a Info.plist (iOS) -->
<!-- Custom Android permissions -->
<!-- Native code modificato -->

<!-- Expo Go usa configurazione standard -->
```

4. **Third-Party SDK Proprietari**
```typescript
// Firebase (alcune features)
// Stripe SDK custom
// Analytics SDK specifici

// Se richiedono setup native, non funzionano
```

#### Quando Passare a Development Builds

**Decision Tree:**

```
La tua app usa solo Expo SDK modules?
  ├─ SÌ → Continua con Expo Go (zero setup!)
  └─ NO ↓

Serve libreria con native code non in Expo SDK?
  ├─ SÌ → Development Build necessaria
  └─ NO ↓

Servono configurazioni native custom?
  ├─ SÌ → Development Build necessaria
  └─ NO → Expo Go sufficiente

Development Build = Custom Expo Go con le TUE dipendenze
```

**Soluzione: Development Builds**

```bash
# Quando Expo Go non basta:
# 1. Installa native module custom
$ npm install react-native-ble-plx

# 2. Crea Development Build (una volta)
$ eas build --profile development --platform ios

# 3. Installa custom Expo Go su device
# Ora funziona come Expo Go MA con i tuoi native modules!

# 4. Continua hot reload come prima
$ npx expo start --dev-client

# ✅ Best of both worlds:
# - Hot reload rapido (come Expo Go)
# - Native modules custom (come RN CLI)
```

#### Confronto: Expo Go vs Development Build vs Production Build

```
┌─────────────────┬──────────────┬─────────────────┬─────────────────┐
│                 │   Expo Go    │ Development     │ Production      │
│                 │              │ Build           │ Build           │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ Setup time      │ 0 min        │ 15-20 min       │ 20-30 min       │
│                 │ (download)   │ (first time)    │                 │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ Hot reload      │ ✅ Sì        │ ✅ Sì          │ ❌ No           │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ Native modules  │ ⚠️ Solo SDK  │ ✅ Tutti       │ ✅ Tutti        │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ Config native   │ ❌ No        │ ✅ Sì          │ ✅ Sì           │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ App size        │ ~30 MB       │ ~50-80 MB       │ ~20-40 MB       │
│                 │ (embedded)   │ (debug symbols) │ (optimized)     │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ Performance     │ Good         │ Good            │ Best            │
├─────────────────┼──────────────┼─────────────────┼─────────────────┤
│ Uso             │ Development  │ Development +   │ App Store       │
│                 │ puro         │ custom modules  │ release         │
└─────────────────┴──────────────┴─────────────────┴─────────────────┘
```

#### Best Practices

**1. Start con Expo Go sempre**
```typescript
// Inizia progetto con Expo Go
// Aggiungi Development Build solo quando serve
// Premature optimization is evil!
```

**2. Usa Expo Go per UI/UX**
```typescript
// Tutto il lavoro di design/layout/styling
// può essere fatto in Expo Go
// Velocità massima!
```

**3. Development Build quando integri native**
```bash
# Solo quando aggiungi:
# - Librerie native third-party
# - Config plugins custom
# - Native code modifications
```

**4. Testing Matrix**
```
Expo Go:
- Sviluppo rapido features
- UI iterations
- Logic testing

Development Build:
- Integration testing con native modules
- Beta testing
- QA pre-release

Production Build:
- App store submissions
- Public releases
```

#### Tips Avanzati

**Condivisione Rapida:**
```bash
# Pubblica app per testing remoto
$ npx expo publish

# Genera link condivisibile
# Chiunque con Expo Go può testare
# No need to be on same network!
```

**Debugging in Expo Go:**
```typescript
// Shake device per aprire dev menu
// - Enable fast refresh
// - Toggle performance monitor
// - Open React DevTools
// - Inspect element tree

// Console logs visibili in terminal:
console.log('Debug info:', data);
```

**Multiple Projects:**
```bash
# Expo Go può switchare tra progetti facilmente
# Recently opened list
# Scan new QR code → instant switch
# Zero juggling tra build!
```

---

## 3. Expo vs React Native CLI

**📚 Teoria: Confronto Approfondito**

### Setup Iniziale

**React Native CLI:**
```bash
# 1. Installa dependencies native
# macOS (per iOS):
$ brew install watchman
$ brew install cocoapods
# Installa Xcode da App Store (15+ GB)
# Installa Xcode Command Line Tools
$ xcode-select --install

# Android:
# Installa Android Studio (3+ GB)
# Setup Android SDK, emulators, ANDROID_HOME env var
# Configura Java JDK

# 2. Crea progetto
$ npx react-native init MyApp

# 3. Installa dipendenze iOS
$ cd MyApp/ios && pod install && cd ..

# 4. Run (richiede emulator/device)
$ npx react-native run-ios      # Solo su macOS
$ npx react-native run-android  # Richiede Android Studio setup

# Tempo setup: 2-4 ore (per developer nuovo)
```

**Expo:**
```bash
# 1. Installa Expo CLI
$ npm install -g expo-cli

# 2. Crea progetto
$ npx create-expo-app MyApp

# 3. Run
$ cd MyApp && npx expo start

# Scansiona QR code con Expo Go app → app running su device!
# Tempo setup: 5 minuti
```

**⚡ Differenza setup: 2-4 ore vs 5 minuti!**

### Native Modules

**React Native CLI:**
```bash
# Installa libreria con native code (es. Camera)
$ npm install react-native-camera

# iOS: Link manualmente native module
$ cd ios && pod install && cd ..

# Android: Configura manualmente in:
# - android/settings.gradle
# - android/app/build.gradle
# - MainApplication.java

# Rebuild app nativa
$ npx react-native run-ios
$ npx react-native run-android

# Tempo: 15-30 minuti + debugging
```

**Expo (con Expo SDK):**
```bash
# Installa libreria Expo SDK
$ npx expo install expo-camera

# Zero configurazione native necessaria!
# Restart dev server
$ npx expo start

# Tempo: 1 minuto
```

**Expo (con custom native module):**
```bash
# Per librerie NON in Expo SDK, serve Development Build
$ npx expo install react-native-ble-plx  # Bluetooth library

# Configura in app.json (config plugin se disponibile)
# Crea development build
$ eas build --profile development --platform ios

# Tempo: 15-20 minuti (solo prima volta)
# Poi: hot reload come Expo Go!
```

### Build e Deploy

**React Native CLI:**
```bash
# iOS Build:
# 1. Apri Xcode
# 2. Configura certificates e provisioning profiles manualmente
# 3. Archive → Upload to App Store Connect
# ⚠️ Richiede Mac + Apple Developer account ($99/anno)

# Android Build:
# 1. Genera keystore per signing
# 2. Configura in build.gradle
# 3. Run: ./gradlew assembleRelease
# 4. Upload APK/AAB manualmente a Play Console

# Tempo: 1-2 ore (prima volta), 30-60 min (successive)
```

**Expo (con EAS):**
```bash
# iOS + Android Build:
$ eas build --platform all

# Expo gestisce:
# - Certificates automaticamente
# - Signing
# - Upload to stores
# ✅ iOS build SENZA Mac!

# Submit:
$ eas submit --platform all

# Tempo: 20-30 minuti (automatico, zero click)
```

### Over-The-Air Updates

**React Native CLI:**
```bash
# Opzione 1: Usa CodePush (Microsoft)
$ npm install react-native-code-push
# Configura manualmente native (iOS + Android)
# Setup CodePush account

# Opzione 2: Custom OTA solution
# (complesso, sconsigliato)
```

**Expo (con EAS Update):**
```bash
# Setup (una volta):
$ eas update:configure

# Publish update:
$ eas update --branch production

# User riceve update automaticamente al prossimo launch
# Tempo: 2 minuti
```

### Confronto Tabella

| Feature | React Native CLI | Expo |
|---------|------------------|------|
| **Setup time** | 2-4 ore | 5 minuti |
| **Serve Mac per iOS?** | ✅ Sì | ❌ No (EAS Build cloud) |
| **Native modules setup** | ⚠️ Manuale | ✅ Automatico (SDK) |
| **Custom native code** | ✅ Pieno accesso | ✅ Con Dev Builds |
| **Hot reload** | ✅ Sì | ✅ Sì |
| **Testing su device** | ⚠️ Richiede build | ✅ Expo Go istantaneo |
| **Build complexity** | 🔴 Alta | 🟢 Bassa |
| **OTA updates** | ⚠️ CodePush (manuale) | ✅ EAS Update (native) |
| **App size** | 🟢 ~10-20 MB | ⚠️ ~30-40 MB (SDK) |
| **Ejecting** | N/A | ✅ Possibile (bare workflow) |
| **Learning curve** | 🔴 Ripida | 🟢 Dolce |
| **Production apps** | ✅ Facebook, Instagram | ✅ Expo Go, OpenAI, Brex |

---

## 4. Quando Usare Expo

**📚 Decision Tree: Expo vs React Native CLI**

```
Il tuo progetto richiede:
- Moduli Bluetooth/NFC custom molto specifici?
- Background tasks complessi (es. audio player)?
- Heavy customization native (es. modifiche a UIKit)?
- Integrazione con SDK native proprietari?
  ├─ SÌ → Considera React Native CLI (o Expo bare workflow)
  └─ NO ↓

Hai team con esperienza native (Swift/Kotlin)?
  ├─ SÌ → Puoi scegliere entrambi
  └─ NO → EXPO (avoid native headaches)

Serve build iOS ma non hai Mac?
  ├─ SÌ → EXPO (EAS Build cloud)
  └─ NO → Entrambi OK

Vuoi deploy rapidi con OTA updates?
  ├─ SÌ → EXPO (EAS Update native)
  └─ NO → Entrambi OK

Priorità: time-to-market veloce?
  ├─ SÌ → EXPO
  └─ NO → Entrambi OK

Default choice → EXPO per 80% dei progetti!
```

### ✅ Usa Expo quando:

1. **Startup/MVP**: Time-to-market è critico
2. **Team JavaScript**: Zero esperienza native
3. **API standard**: Camera, Location, Notifications sufficienti
4. **No Mac**: Vuoi build iOS senza Mac
5. **Frequent updates**: Necessità di OTA updates veloci
6. **Prototyping**: Rapid iteration con Expo Go

### ⚠️ Considera React Native CLI quando:

1. **Native modules specifici**: Librerie native custom non in Expo SDK
2. **App size critica**: Ogni MB conta (es. mercati emergenti con 2G/3G)
3. **Background tasks advanced**: Audio/video recording complessi
4. **Integrazione native pesante**: SDK proprietari aziendali
5. **Team native esperto**: Hai già infra e skill native

### 🎯 Hybrid Approach: Expo Bare Workflow

**Best of both worlds!**

```bash
# Start with Expo
$ npx create-expo-app MyApp

# Eject to bare workflow (mantieni EAS services!)
$ npx expo prebuild

# Ora hai:
# ✅ ios/ e android/ folders (come RN CLI)
# ✅ Accesso completo a native code
# ✅ EAS Build/Update ancora funzionanti
# ✅ Expo SDK modules still available

# Quando serve:
# - Aggiungi native module non-Expo
# - Modifica Info.plist / AndroidManifest.xml direttamente
# - Integra SDK third-party con setup native
```

---

## 5. Modello di Business e Pricing

**📚 Teoria: Come Funziona il Business di Expo**

### Expo è Open Source

```
MIT License:
✅ SDK completamente free e open source
✅ CLI tools free
✅ Expo Go app free
✅ Documentazione free
✅ Community support free

Puoi usare Expo senza pagare nulla!
```

### EAS Services (Freemium Model)

**Free Tier:**
```
EAS Build:
- 30 builds/mese (iOS + Android combined)
- Shared build machines
- Build time: ~15-20 minuti

EAS Update:
- Unlimited OTA updates
- Unlimited users

EAS Submit:
- Unlimited submissions

Perfetto per: Side projects, MVP, small teams
```

**Production Plan ($99/mese):**
```
EAS Build:
- Unlimited builds
- Priority queue (faster builds)
- Larger build machines

+ Features:
- Team collaboration
- Preview comments
- Rollback updates
- Analytics

Per: Professional apps, growing startups
```

**Enterprise Plan ($2999+/mese):**
```
- Self-hosted EAS
- SLA guarantees
- Dedicated support
- Custom integrations
- SSO/SAML

Per: Fortune 500, enterprise apps
```

### Confronto Costi

**Scenario: E-commerce app production**

**Con React Native CLI:**
```
- Mac Mini per iOS builds: $700
- Apple Developer: $99/anno
- Google Play: $25 one-time
- CodePush (Microsoft) account: $0-$40/mese
- CI/CD (GitHub Actions, CircleCI): $50-$200/mese

Total Year 1: ~$1200-$3000
Complessità: 🔴 Alta (manage infra)
```

**Con Expo EAS:**
```
- Apple Developer: $99/anno
- Google Play: $25 one-time
- EAS Production: $99/mese = $1188/anno

Total Year 1: ~$1312
Complessità: 🟢 Zero infra management
```

**Verdict**: Expo costa leggermente più ma **risparmia centinaia di ore** di setup e maintenance infra.

---

## 💡 Case Studies

### Airbnb (RN CLI)

```
- Started with React Native CLI (2016-2018)
- Moved BACK to native (2018)
- Ragione: Performance issues, native customization needs
- Context: App molto complessa, team grande native

Lezione: Per app massive enterprise, RN può diventare limitante
```

### Expo Go App (Expo)

```
- Self-hosting su Expo
- Milioni di developer users
- Zero problemi di performance
- Rapid iteration con EAS Update

Lezione: Expo production-ready per app complesse
```

### Your Project?

```
Se sei qui, probabilmente Expo è la scelta giusta!
- 80% progetti mobile = Expo sufficiente
- 15% progetti = Expo bare workflow
- 5% progetti = RN CLI necessario (edge cases)
```

---

## ✅ Checklist di Completamento

- [ ] Comprendo cos'è Expo (framework + platform + SDK)
- [ ] Conosco i componenti dell'ecosistema (SDK, EAS, Expo Go)
- [ ] Capisco differenze Expo vs React Native CLI
- [ ] So quando scegliere Expo vs CLI
- [ ] Conosco il modello di pricing EAS
- [ ] Sono pronto a iniziare con setup Expo

---

## 📌 Punti Chiave da Ricordare

1. 🚀 **Expo = React Native semplificato** + servizi cloud + 50+ API native
2. ⚡ **Setup: 5 minuti** vs 2-4 ore (RN CLI)
3. ☁️ **EAS Build**: iOS senza Mac, Android senza hassle
4. 🔄 **EAS Update**: OTA updates in 2 minuti
5. 📱 **Expo Go**: Testing su device senza build
6. 💰 **Pricing**: Free tier generoso, $99/mese per production
7. ✅ **Default choice**: Usa Expo per 80% dei progetti
8. 🔧 **Bare workflow**: Sempre possibile "eject" se serve native access

---

[Prossimo: 03. Setup Ambiente Expo →](03_Setup_Ambiente_Expo.md)

[← Precedente: Panorama Mobile](01_Panorama_Mobile_Moderno.md) | [Torna all'Indice](README.md)
