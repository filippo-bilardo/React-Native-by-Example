# 06. Managed vs Bare Workflow - Guida Completa

[← Precedente: Expo Go vs Dev Builds](05_Expo_Go_vs_Development_Builds.md) | [Torna all'Indice](README.md) | [Prossimo: Expo SDK APIs →](07_Expo_SDK_APIs.md)

---

## Descrizione

Uno dei concetti più importanti nell'ecosistema Expo è la distinzione tra **Managed Workflow** e **Bare Workflow**. Questa scelta architetturale influenza profondamente come sviluppi, buildi e mantieni la tua app. In questo capitolo esploreremo entrambi gli approcci, capiremo quando usare l'uno vs l'altro, e impareremo come "eject" da managed a bare quando necessario.

---

## 🎯 Obiettivi di Apprendimento

- [ ] Comprendere cos'è il Managed Workflow
- [ ] Comprendere cos'è il Bare Workflow  
- [ ] Capire differenze architetturali tra i due
- [ ] Sapere quando usare managed vs bare
- [ ] Eseguire "prebuild" per generare native folders
- [ ] Configurare Config Plugins per native customizations
- [ ] Migrare da managed a bare workflow

---

## 📚 Contenuti

1. [Managed Workflow: Expo Gestisce Native](#1-managed-workflow-expo-gestisce-native)
2. [Bare Workflow: Tu Gestisci Native](#2-bare-workflow-tu-gestisci-native)
3. [Confronto Approfondito](#3-confronto-approfondito)
4. [Config Plugins: Best of Both Worlds](#4-config-plugins-best-of-both-worlds)
5. [Expo Prebuild: Generate Native Folders](#5-expo-prebuild-generate-native-folders)
6. [Quando Usare Managed vs Bare](#6-quando-usare-managed-vs-bare)
7. [Migrazione da Managed a Bare](#7-migrazione-da-managed-a-bare)

---

## 1. Managed Workflow: Expo Gestisce Native

**📚 Teoria: "Managed" = Expo Gestisce Tutto**

```
Managed Workflow = NO ios/ e android/ folders nel tuo progetto

Struttura progetto managed:
MyApp/
├── App.tsx               ← Il tuo codice
├── app.json              ← Configurazione
├── assets/               ← Images, fonts
├── package.json
├── node_modules/
└── .expo/

⚠️ NOTA: NO ios/ folder, NO android/ folder!

Expo gestisce:
✅ Build configuration (Xcode settings, Gradle config)
✅ Native dependencies linking
✅ Certificates & provisioning
✅ Platform-specific files (Info.plist, AndroidManifest.xml)
✅ App icon & splash screen generation

Tu gestisci:
✅ Business logic (JS/TS)
✅ UI components
✅ State management
✅ API integrations
```

### Come Funziona Managed Workflow

```
Development:
1. Tu scrivi codice → App.tsx, components/
2. Tu configuri app → app.json (declarative config)
3. Expo Go / Dev Build → Esegue tua app (no native folders needed)

Build:
1. Tu: $ eas build --platform ios
2. EAS Build cloud:
   a. Legge app.json configuration
   b. GENERA ios/ folder automaticamente (via "expo prebuild")
   c. Applica configurazioni (icon, splash, permissions)
   d. Compila con Xcode
   e. Upload IPA
3. Tu: Download binary

Deploy:
1. Tu: $ eas submit --platform ios
2. EAS: Upload a App Store Connect automaticamente

Update (OTA):
1. Tu: $ eas update --branch production
2. User: Riceve JS update al prossimo app launch (no app store review!)
```

**🔑 Key Insight**: In managed workflow, native folders (ios/, android/) sono **generated on-demand** durante build, non esistono nel tuo repo!

### Vantaggi Managed Workflow

```
✅ Zero configurazioni native manuali
✅ No Xcode/Android Studio necessari
✅ Upgrade Expo SDK = upgrade automatico native dependencies
✅ Config declarative in app.json (readable, version-controllable)
✅ Build reproducibili (stesso app.json → stesso binary)
✅ Meno merge conflicts (no native files in git)
✅ Onboarding team velocissimo (no native knowledge required)
✅ EAS Build cloud gestisce certificates automaticamente
```

### Limitazioni Managed Workflow

```
❌ Non puoi modificare native files direttamente (no editing Info.plist)
❌ Non puoi aggiungere native modules senza Config Plugin
❌ Dipendi da Expo team per supporto nuove native features
⚠️ Alcune librerie third-party non hanno Config Plugin → serve bare workflow

MA: Config Plugins risolve 90% dei casi (vedi sezione 4)!
```

### Esempio: App Managed Workflow

```typescript
/**
 * Progetto Managed Workflow
 * File: app.json
 */
{
  "expo": {
    "name": "My Managed App",
    "slug": "my-managed-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    
    // Permissions (declarative!)
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "We need camera access to take photos"
      }
    },
    "android": {
      "permissions": ["CAMERA"]
    },
    
    // Plugins per native configuration
    "plugins": [
      "expo-camera",  // Camera Config Plugin
      "expo-location" // Location Config Plugin
    ]
  }
}
```

**Quando buildi:**
```bash
$ eas build --platform ios

# EAS:
# 1. Legge app.json
# 2. Genera Info.plist con NSCameraUsageDescription
# 3. Link expo-camera native module
# 4. Build app con Xcode

# Tu non hai MAI toccato Xcode!
```

---

## 2. Bare Workflow: Tu Gestisci Native

**📚 Teoria: "Bare" = Full Control su Native**

```
Bare Workflow = ios/ e android/ folders nel tuo progetto

Struttura progetto bare:
MyApp/
├── App.tsx               ← Il tuo codice
├── app.json              ← Config (meno usato)
├── ios/                  ← Native iOS project ⭐
│   ├── MyApp.xcodeproj
│   ├── MyApp/
│   │   ├── Info.plist
│   │   └── AppDelegate.m
│   └── Podfile
├── android/              ← Native Android project ⭐
│   ├── app/
│   │   ├── build.gradle
│   │   └── src/
│   │       └── AndroidManifest.xml
│   └── build.gradle
├── package.json
├── node_modules/
└── .expo/

Tu gestisci:
✅ Tutto! (Native + JS)
✅ Modifiche dirette a Info.plist, AndroidManifest.xml
✅ Aggiungi native modules manualmente
✅ Custom native code (Swift, Kotlin)
✅ Build configuration (Xcode settings, Gradle)

Expo fornisce:
✅ Expo SDK modules (still available!)
✅ EAS Build (still works!)
✅ EAS Update (still works!)
✅ Hot reload development
```

### Come Funziona Bare Workflow

```
Development:
1. Tu scrivi codice → App.tsx, components/
2. Tu configuri native → Xcode / Android Studio direttamente
3. Run locale o Dev Build → Esegue app

Build:
Opzione A: Local build
$ npx expo run:ios         # Build locale con Xcode (richiede Mac)
$ npx expo run:android     # Build locale con Android Studio

Opzione B: EAS Build (still recommended!)
$ eas build --platform ios  # EAS cloud build (still works!)

Update (OTA):
$ eas update --branch production  # Still works! JS updates only
```

**🔑 Key Insight**: Bare workflow = React Native CLI tradizionale + Expo SDK modules + (opzionalmente) EAS services.

### Vantaggi Bare Workflow

```
✅ Full control su native code (Info.plist, AndroidManifest, Swift/Kotlin)
✅ Aggiungi ANY native module (anche senza Config Plugin)
✅ Customizza build process (Xcode schemes, Gradle flavors)
✅ Debug native code (Xcode debugger, Android Studio)
✅ Integra SDK proprietari (es. banking SDKs)
✅ Background tasks advanced (audio player, location tracking)
✅ No limiti di Expo (sei su React Native "puro")
```

### Svantaggi Bare Workflow

```
❌ Devi gestire native manually (Xcode/Android Studio skills required)
❌ Upgrade Expo SDK = manual native changes
❌ Build locale richiede Mac (per iOS) e setup complesso
❌ Native files in git = merge conflicts more frequent
❌ Team onboarding slower (need native knowledge)
❌ Certificates & provisioning manual (unless using EAS)
```

---

## 3. Confronto Approfondito

### Tabella Decisionale

| Aspetto | Managed Workflow | Bare Workflow |
|---------|------------------|---------------|
| **Native folders** | ❌ No (generated on build) | ✅ Sì (ios/, android/) |
| **Xcode/Android Studio** | ❌ Non necessari | ✅ Necessari |
| **Native code editing** | ❌ No (usa Config Plugins) | ✅ Sì (full control) |
| **Expo SDK modules** | ✅ Sì | ✅ Sì |
| **EAS Build** | ✅ Sì | ✅ Sì |
| **EAS Update** | ✅ Sì | ✅ Sì (JS only) |
| **Build iOS senza Mac** | ✅ Sì (EAS cloud) | ✅ Sì (EAS cloud) |
| **Custom native modules** | ⚠️ Via Config Plugin | ✅ Diretta |
| **Upgrade Expo SDK** | ✅ Automatico | ⚠️ Manual |
| **Team onboarding** | ✅ Veloce (no native) | ⚠️ Lento (native skills) |
| **Merge conflicts** | ✅ Rari | ⚠️ Frequenti |
| **Complexity** | 🟢 Bassa | 🔴 Alta |
| **Flexibility** | 🟡 Media | ⭐ Massima |

### Workflow Comparison

**Scenario: Aggiungere Camera Permission**

**Managed:**
```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow app to access camera"
        }
      ]
    ]
  }
}

// Build:
$ eas build --platform ios
// → Config Plugin genera automaticamente Info.plist entry
```

**Bare:**
```xml
<!-- ios/MyApp/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>Allow app to access camera</string>

<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
```

```bash
# Build:
$ npx expo run:ios  # Build locale
# Oppure:
$ eas build --platform ios  # EAS cloud build
```

**Verdict**: Managed più semplice per case comuni, Bare per full control.

---

## 4. Config Plugins: Best of Both Worlds

**📚 Teoria: Config Plugins**

```
Config Plugin = File JavaScript che modifica native configuration

Come funziona:
1. Tu: Configuri plugin in app.json
2. expo prebuild: Esegue plugin
3. Plugin: Modifica ios/android folders automaticamente
4. Build: Usa folders modificati

Vantaggi:
✅ Mantieni managed workflow (no ios/android in git)
✅ MA hai customizations native (via plugins)
✅ Configurazione declarative e version-controllable
✅ Riusabile across projects
```

### Esempio: Config Plugin Esistente

```json
// app.json
{
  "expo": {
    "plugins": [
      // Camera plugin (built-in Expo)
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access camera",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access microphone",
          "recordAudioAndroid": true
        }
      ],
      
      // Location plugin
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow location access",
          "isIosBackgroundLocationEnabled": true,
          "isAndroidBackgroundLocationEnabled": true
        }
      ],
      
      // Custom plugin
      "./plugins/my-custom-plugin.js"
    ]
  }
}
```

**Quando buildi:**
```bash
$ eas build --platform all

# EAS:
# 1. Runs "expo prebuild" (genera ios/android folders)
# 2. Executes plugins su folders
# 3. Plugins modificano Info.plist, AndroidManifest, ecc.
# 4. Build with modified configuration
```

### Creare Config Plugin Custom

```javascript
// plugins/my-custom-plugin.js

/**
 * Config Plugin per aggiungere custom URL scheme
 */
const { withInfoPlist, withAndroidManifest } = require('@expo/config-plugins');

const withCustomUrlScheme = (config, { scheme }) => {
  // iOS: Modifica Info.plist
  config = withInfoPlist(config, (config) => {
    config.modResults.CFBundleURLTypes = [
      {
        CFBundleURLSchemes: [scheme],
      },
    ];
    return config;
  });

  // Android: Modifica AndroidManifest.xml
  config = withAndroidManifest(config, (config) => {
    const mainActivity = config.modResults.manifest.application[0].activity.find(
      (activity) => activity.$['android:name'] === '.MainActivity'
    );
    
    mainActivity['intent-filter'].push({
      action: [{ $: { 'android:name': 'android.intent.action.VIEW' } }],
      category: [
        { $: { 'android:name': 'android.intent.category.DEFAULT' } },
        { $: { 'android:name': 'android.intent.category.BROWSABLE' } },
      ],
      data: [{ $: { 'android:scheme': scheme } }],
    });
    
    return config;
  });

  return config;
};

module.exports = withCustomUrlScheme;
```

**Uso:**
```json
// app.json
{
  "expo": {
    "plugins": [
      ["./plugins/my-custom-plugin.js", { "scheme": "myapp" }]
    ]
  }
}
```

**Risultato**: URL `myapp://path` apre la tua app!

---

## 5. Expo Prebuild: Generate Native Folders

**📚 Teoria: expo prebuild**

```
expo prebuild = Comando che genera ios/ e android/ folders

Da dove genera?
1. Template Expo base (React Native + Expo modules setup)
2. app.json configuration
3. Config plugins modifications

Quando viene eseguito?
- Automaticamente da "eas build"
- Manualmente con "npx expo prebuild"
- Prima volta che fai "npx expo run:ios/android"
```

### Comando Prebuild

```bash
# Generate native folders:
$ npx expo prebuild

# Output:
✔ Platform › all (iOS and Android)
› Creating native directories...
› Generating ios/ folder...
› Generating android/ folder...
› Installing CocoaPods...
› Installing node modules...
✔ Config synced

# Ora hai:
MyApp/
├── ios/           ← Generated!
├── android/       ← Generated!
├── App.tsx
└── app.json
```

**⚠️ Attenzione**: Dopo `expo prebuild`, sei **tecnicamente** in bare workflow (hai native folders). Ma puoi ancora rimanere "managed" NON committando ios/android in git!

### .gitignore per "Managed with Prebuild"

```bash
# .gitignore

# Keep managed workflow (don't commit native folders)
ios/
android/

# But commit config
app.json
eas.json
plugins/
```

**Pattern consigliato:**
```
1. ios/android/ in .gitignore (not committed)
2. Run "expo prebuild" durante CI/CD build
3. Build usa folders generated on-the-fly
4. Mantieni managed workflow benefits + config plugin customizations
```

---

## 6. Quando Usare Managed vs Bare

**📚 Decision Tree**

```
Il tuo progetto richiede:

Heavy native customizations?
(Custom native UI, complex native modules, proprietary SDKs)
├─ SÌ → Bare Workflow
└─ NO ↓

Existing React Native CLI project?
├─ SÌ → Bare Workflow (add Expo modules)
└─ NO ↓

Team ha esperienza native (Swift/Kotlin)?
├─ SÌ → Bare Workflow OK
└─ NO → Managed Workflow

Hai Config Plugin per tue dependencies?
├─ SÌ → Managed Workflow ✅
└─ NO ↓

Puoi creare Config Plugin custom?
├─ SÌ → Managed Workflow ✅
└─ NO → Bare Workflow

Vuoi zero headaches native?
├─ SÌ → Managed Workflow ✅
└─ NO → Bare Workflow

Default choice per 80% progetti: Managed Workflow!
```

### Caso d'Uso: Managed Workflow

```
✅ Usa Managed quando:
- Startup/MVP rapid development
- Team JavaScript-only
- Standard features (Camera, Location, Notifications)
- Voglio zero native complexity
- Frequent Expo SDK upgrades
- Small team (<10 developer)

Esempi:
- Social media app
- E-commerce app
- Productivity app
- Fitness tracker (senza custom wearables)
- News/content app
```

### Caso d'Uso: Bare Workflow

```
✅ Usa Bare quando:
- Custom native UI components
- Proprietary native SDKs (banking, medical)
- Background services advanced
- Existing RN CLI codebase
- Team native esperto
- Need full control su build process

Esempi:
- Banking app (con native security SDK)
- AR/VR app (custom rendering)
- IoT device controller (Bluetooth custom protocol)
- White-label app (multiple bundle IDs)
- Enterprise app con native compliance
```

---

## 7. Migrazione da Managed a Bare

**📚 Step-by-Step Migration**

### Step 1: Backup Progetto

```bash
# Commit tutto:
$ git add .
$ git commit -m "Before migration to bare workflow"

# Create branch:
$ git checkout -b migration/bare-workflow
```

### Step 2: Generate Native Folders

```bash
# Run prebuild:
$ npx expo prebuild

# Output genera:
# - ios/ folder
# - android/ folder
# - Installa CocoaPods (iOS)
```

### Step 3: Verifica Build Locale

```bash
# Test iOS (macOS only):
$ npx expo run:ios

# Test Android:
$ npx expo run:android

# Se app runs correttamente → migration successful!
```

### Step 4: Commit Native Folders

```bash
# Rimuovi ios/android da .gitignore
# (Oppure mantienili in .gitignore per "managed with prebuild" pattern)

# Commit:
$ git add ios/ android/
$ git commit -m "Migration to bare workflow: add native folders"
```

### Step 5: Update EAS Configuration

```json
// eas.json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
      // ❌ Rimuovi "ios": { "simulator": true }
      // Bare workflow: build da native folders esistenti
    },
    "production": {}
  }
}
```

### Step 6: Native Customizations

Ora puoi modificare direttamente:

```bash
# iOS:
$ open ios/MyApp.xcworkspace  # Apri Xcode
# Modifica Info.plist, capabilities, signing

# Android:
# Open android/ in Android Studio
# Modifica build.gradle, AndroidManifest.xml
```

### Step 7: Aggiornamenti Futuri

```bash
# Upgrade Expo SDK in bare workflow:
$ npx expo install expo@latest

# Poi: Manual native changes (vedi upgrade guide Expo docs)
# Esempio: Upgrade CocoaPods:
$ cd ios && pod install && cd ..
```

---

## ✅ Checklist di Completamento

- [ ] Comprendo differenze Managed vs Bare workflow
- [ ] So quando usare managed vs bare
- [ ] Capisco cosa sono Config Plugins
- [ ] Ho eseguito expo prebuild per generare native folders
- [ ] So come migrare da managed a bare
- [ ] Comprendo pattern "managed with prebuild" (folders in gitignore)

---

## 📌 Punti Chiave da Ricordare

1. 🎯 **Managed**: NO native folders, Expo gestisce tutto, Config Plugins per customizations
2. 🔧 **Bare**: CON native folders, full control, manual native management
3. 🔌 **Config Plugins**: Best of both worlds (managed + customizations)
4. 🏗️ **expo prebuild**: Genera ios/android on-demand
5. 📋 **Pattern consigliato**: Managed con Config Plugins per 80% progetti
6. 🔄 **Migration**: Sempre possibile da managed → bare (irreversible)
7. ⚡ **EAS services**: Funzionano con ENTRAMBI i workflows

---

[Prossimo: 07. Expo SDK - Camera, Location, Notifications →](07_Expo_SDK_APIs.md)

[← Precedente: Expo Go vs Dev Builds](05_Expo_Go_vs_Development_Builds.md) | [Torna all'Indice](README.md)
