# Appendice B: Troubleshooting Comune

Questa appendice raccoglie i problemi più comuni che si incontrano nello sviluppo React Native, con soluzioni dettagliate e procedure di debug. Ogni problema include sintomi, cause, e soluzioni passo-passo.

---

## 📋 Indice Rapido

- [Setup e Installazione](#setup-e-installazione)
- [Build e Compilazione](#build-e-compilazione)
- [Metro Bundler](#metro-bundler)
- [Dipendenze Native](#dipendenze-native)
- [Performance](#performance)
- [Networking](#networking)
- [Debug](#debug)
- [iOS Specifico](#ios-specifico)
- [Android Specifico](#android-specifico)
- [Expo](#expo)
- [TypeScript](#typescript)

---

## 🛠️ Setup e Installazione

### ❌ Problema: `command not found: npx` o `command not found: npm`

**Sintomi:**
```bash
$ npx react-native init MyApp
bash: npx: command not found
```

**Causa:** Node.js non installato o non nel PATH.

**Soluzione:**

1. **Verifica installazione Node.js:**
```bash
node --version
npm --version
```

2. **Installa/Aggiorna Node.js:**

**macOS (Homebrew):**
```bash
brew install node
```

**Windows:**
- Scarica installer da [nodejs.org](https://nodejs.org)
- Usa `nvm-windows`: `nvm install lts` poi `nvm use lts`

**Linux:**
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Con nvm (consigliato)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
nvm use --lts
```

3. **Riavvia il terminale** dopo l'installazione.

---

### ❌ Problema: `EACCES: permission denied` durante `npm install -g`

**Sintomi:**
```bash
$ npm install -g expo-cli
npm ERR! code EACCES
npm ERR! syscall mkdir
npm ERR! path /usr/local/lib/node_modules/expo-cli
npm ERR! errno -13
```

**Causa:** Permessi insufficienti per scrivere in directory globali npm.

**Soluzione:**

**Opzione 1 - Cambia directory globale npm (consigliato):**
```bash
# Crea directory per moduli globali
mkdir ~/.npm-global

# Configura npm
npm config set prefix '~/.npm-global'

# Aggiungi al PATH (.bashrc, .zshrc, o .bash_profile)
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Ora installa senza sudo
npm install -g expo-cli
```

**Opzione 2 - Usa nvm (migliore per sviluppo):**
```bash
# Installa nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Installa Node via nvm
nvm install --lts
nvm use --lts

# Ora npm globale funziona senza sudo
npm install -g expo-cli
```

**Opzione 3 - Fix permessi (solo Linux/macOS):**
```bash
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) /usr/local/lib/node_modules
```

⚠️ **Non usare `sudo npm install -g`** - crea problemi di permessi!

---

### ❌ Problema: `error Couldn't find a package.json file`

**Sintomi:**
```bash
$ npm install
error Couldn't find a package.json file in "/path/to/dir"
```

**Causa:** Sei nella directory sbagliata o `package.json` non esiste.

**Soluzione:**

1. **Verifica di essere nella directory del progetto:**
```bash
ls -la
# Dovresti vedere package.json
```

2. **Se non c'è package.json, inizializza:**
```bash
# Per nuovo progetto React Native
npx react-native init MyApp
cd MyApp

# Per progetto Expo
npx create-expo-app MyApp
cd MyApp

# Per progetto esistente senza package.json
npm init -y
```

---

## 🏗️ Build e Compilazione

### ❌ Problema: `Build failed` - CocoaPods non installato (iOS)

**Sintomi:**
```bash
error Could not find "Podfile.lock" at ... 
Run CLI with --verbose flag for more details.
Command PhaseScriptExecution failed with a nonzero exit code
```

**Causa:** CocoaPods non installato o non configurato.

**Soluzione:**

1. **Installa CocoaPods:**
```bash
# macOS con Homebrew (consigliato)
brew install cocoapods

# oppure con Ruby gems
sudo gem install cocoapods
```

2. **Installa dipendenze iOS:**
```bash
cd ios
pod install
cd ..
```

3. **Se pod install fallisce, prova:**
```bash
cd ios
# Pulisci cache
rm -rf Pods Podfile.lock
pod cache clean --all

# Reinstalla
pod install --repo-update
cd ..
```

4. **Rebuilda l'app:**
```bash
npx react-native run-ios
```

---

### ❌ Problema: `Command PhaseScriptExecution failed` (iOS)

**Sintomi:**
```bash
❌  error Command PhaseScriptExecution failed with a nonzero exit code
```

**Causa:** Problemi con script di build, spesso permessi o cache.

**Soluzione:**

**Step 1 - Pulisci cache:**
```bash
# Pulisci cache Metro
rm -rf $TMPDIR/metro-*
rm -rf $TMPDIR/haste-map-*

# Pulisci node_modules e reinstalla
rm -rf node_modules
npm install

# Pulisci cache iOS
cd ios
rm -rf build
rm -rf ~/Library/Developer/Xcode/DerivedData
pod deintegrate
pod install
cd ..
```

**Step 2 - Verifica permessi script:**
```bash
# In ios/YourApp.xcodeproj
# Build Phases > Bundle React Native code and images
# Assicurati che lo script esista:
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

**Step 3 - Pulisci e rebuilda:**
```bash
npx react-native run-ios
```

**Step 4 - Se ancora problemi, apri Xcode:**
```bash
open ios/YourApp.xcworkspace
# Product > Clean Build Folder (Shift+Cmd+K)
# Product > Build (Cmd+B)
```

---

### ❌ Problema: `Failed to install the app` (Android)

**Sintomi:**
```bash
error Failed to install the app. 
Make sure you have the Android development environment set up
INSTALL_FAILED_INSUFFICIENT_STORAGE
```

**Causa:** Storage insufficiente nell'emulatore, SDK non configurato, o APK corrotto.

**Soluzione:**

**Caso 1 - Storage insufficiente:**
```bash
# Aumenta storage emulatore
# Android Studio > AVD Manager > Edit > Show Advanced Settings
# Internal Storage: 2000 MB (minimo)

# Oppure pulisci emulatore
adb shell pm clear com.yourapp
```

**Caso 2 - SDK non configurato:**
```bash
# Verifica ANDROID_HOME
echo $ANDROID_HOME
# Dovrebbe essere: /Users/[USERNAME]/Library/Android/sdk (macOS)
#                   /home/[USERNAME]/Android/Sdk (Linux)
#                   C:\Users\[USERNAME]\AppData\Local\Android\Sdk (Windows)

# Se vuoto, aggiungi a .bashrc/.zshrc:
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools

source ~/.zshrc  # o ~/.bashrc
```

**Caso 3 - Pulisci build:**
```bash
cd android
./gradlew clean
cd ..

# Pulisci cache
rm -rf android/app/build
rm -rf $TMPDIR/react-*

# Rebuilda
npx react-native run-android
```

---

### ❌ Problema: `Execution failed for task ':app:installDebug'`

**Sintomi:**
```bash
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: No connected devices!
```

**Causa:** Nessun dispositivo/emulatore connesso o adb non funzionante.

**Soluzione:**

1. **Verifica dispositivi connessi:**
```bash
adb devices
# Dovrebbe listare almeno un dispositivo
```

2. **Se lista vuota:**

**Per emulatore:**
```bash
# Lista emulatori disponibili
emulator -list-avds

# Avvia emulatore
emulator -avd Pixel_5_API_31

# Oppure da Android Studio
# Tools > AVD Manager > Play button
```

**Per dispositivo fisico:**
- Abilita "Developer Options" sul telefono
- Abilita "USB Debugging"
- Connetti USB e accetta prompt

3. **Se adb non riconosce dispositivo:**
```bash
# Restart adb
adb kill-server
adb start-server
adb devices

# Se ancora problemi (macOS/Linux):
sudo adb devices
```

4. **Rebuilda:**
```bash
npx react-native run-android
```

---

## 📦 Metro Bundler

### ❌ Problema: `Unable to resolve module`

**Sintomi:**
```bash
error: Error: Unable to resolve module @react-navigation/native from App.tsx: 
@react-navigation/native could not be found within the project or in these directories:
  node_modules
```

**Causa:** Dipendenza non installata o cache corrotta.

**Soluzione:**

**Step 1 - Installa dipendenza mancante:**
```bash
npm install @react-navigation/native
# oppure
yarn add @react-navigation/native
```

**Step 2 - Pulisci cache Metro:**
```bash
# Restart Metro con cache pulita
npx react-native start --reset-cache

# Oppure:
rm -rf $TMPDIR/metro-*
rm -rf $TMPDIR/haste-map-*
```

**Step 3 - Pulisci e reinstalla node_modules:**
```bash
rm -rf node_modules
rm package-lock.json  # o yarn.lock
npm install
```

**Step 4 - Per iOS, reinstalla pods:**
```bash
cd ios
rm -rf Pods Podfile.lock
pod install
cd ..
```

**Step 5 - Restart completo:**
```bash
# Chiudi Metro (Ctrl+C)
npx react-native start --reset-cache

# In altro terminale
npx react-native run-ios
# oppure
npx react-native run-android
```

---

### ❌ Problema: `Metro has encountered an error: ENOSPC`

**Sintomi:**
```bash
Error: ENOSPC: System limit for number of file watchers reached
```

**Causa:** Limite file watchers troppo basso (Linux).

**Soluzione:**

**Linux:**
```bash
# Aumenta limite temporaneamente
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf

# Applica subito
sudo sysctl -p

# Verifica
cat /proc/sys/fs/inotify/max_user_watches
# Dovrebbe mostrare 524288
```

**macOS:**
```bash
# Installa watchman (gestisce file watching)
brew install watchman

# Verifica
watchman version
```

Poi riavvia Metro:
```bash
npx react-native start --reset-cache
```

---

### ❌ Problema: `Port 8081 already in use`

**Sintomi:**
```bash
error: Metro Bundler error: listen EADDRINUSE: address already in use :::8081
```

**Causa:** Metro già in esecuzione o altra app usa porta 8081.

**Soluzione:**

**Opzione 1 - Killa processo sulla porta 8081:**
```bash
# macOS/Linux
lsof -ti:8081 | xargs kill -9

# Windows
netstat -ano | findstr :8081
taskkill /PID <PID_NUMBER> /F
```

**Opzione 2 - Usa porta diversa:**
```bash
npx react-native start --port 8088

# In app, aggiorna configurazione (solo per debug)
```

**Opzione 3 - Restart completo:**
```bash
# Chiudi tutti i terminali Metro
killall -9 node  # macOS/Linux

# Riavvia
npx react-native start
```

---

## 🔌 Dipendenze Native

### ❌ Problema: `Invariant Violation: Native module cannot be null`

**Sintomi:**
```javascript
ERROR  Invariant Violation: Native module cannot be null.
// oppure
ERROR  TurboModuleRegistry.getEnforcing(...): 'RNCAsyncStorage' could not be found.
```

**Causa:** Modulo nativo non linkato correttamente.

**Soluzione:**

**Per React Native 0.60+ (Autolinking):**

1. **Reinstalla modulo:**
```bash
npm uninstall @react-native-async-storage/async-storage
npm install @react-native-async-storage/async-storage
```

2. **iOS - Reinstalla pods:**
```bash
cd ios
rm -rf Pods Podfile.lock
pod install
cd ..
```

3. **Android - Pulisci build:**
```bash
cd android
./gradlew clean
cd ..
```

4. **Rebuilda app:**
```bash
# Chiudi Metro (Ctrl+C)
npx react-native run-ios
# oppure
npx react-native run-android
```

**Per React Native < 0.60 (Manual linking):**
```bash
npx react-native link @react-native-async-storage/async-storage
```

**Se ancora problemi (iOS):**
```bash
# Verifica Podfile include:
# ios/Podfile dovrebbe avere:
use_react_native!(
  :path => config[:reactNativePath],
  # ... altre configurazioni
)

cd ios
pod install
cd ..
```

---

### ❌ Problema: `NullInjectorError: No provider for...` (dopo aggiornamento)

**Sintomi:**
```bash
ERROR  Invariant Violation: Module AppRegistry is not a registered callable module
```

**Causa:** Versioni incompatibili tra dipendenze o cache corrotta.

**Soluzione:**

**Step 1 - Verifica compatibilità versioni:**
```bash
# Controlla package.json
# react, react-native devono avere versioni compatibili

# Esempio corretto:
# "react": "18.2.0",
# "react-native": "0.73.0"
```

**Step 2 - Pulisci tutto:**
```bash
# Pulisci cache npm
npm cache clean --force

# Rimuovi node_modules e lock files
rm -rf node_modules
rm package-lock.json

# Reinstalla
npm install

# iOS: reinstalla pods
cd ios
rm -rf Pods Podfile.lock
pod install
cd ..

# Android: pulisci build
cd android
./gradlew clean
cd ..

# Pulisci cache Metro
rm -rf $TMPDIR/metro-*
rm -rf $TMPDIR/haste-map-*
```

**Step 3 - Rebuilda:**
```bash
npx react-native start --reset-cache
# In altro terminale
npx react-native run-ios # o run-android
```

---

## ⚡ Performance

### ❌ Problema: App lenta, FPS bassi, UI lag

**Sintomi:**
- Animazioni scattose
- Scroll non fluido
- Delay dopo tap

**Causa:** Re-render eccessivi, operazioni pesanti su JS thread, immagini non ottimizzate.

**Soluzione:**

**Step 1 - Enable Performance Monitor:**
```javascript
// In dev mode
// iOS: Cmd+D > Show Perf Monitor
// Android: Ctrl+M > Show Perf Monitor

// Verifica:
// - JS thread dovrebbe rimanere >50 FPS
// - UI thread dovrebbe rimanere >55 FPS
```

**Step 2 - Trova e ottimizza re-render:**
```javascript
// Installa why-did-you-render
npm install --save-dev @welldone-software/why-did-you-render

// wdyr.js
import React from 'react';

if (process.env.NODE_ENV === 'development') {
  const whyDidYouRender = require('@welldone-software/why-did-you-render');
  whyDidYouRender(React, {
    trackAllPureComponents: true,
  });
}

// index.js
import './wdyr'; // <--- DEVE essere prima di App
import {AppRegistry} from 'react-native';
import App from './App';

AppRegistry.registerComponent('appName', () => App);
```

**Step 3 - Ottimizza componenti:**
```javascript
// ❌ BAD - Re-render ogni volta che parent cambia
function Item({ item }) {
  console.log('Render item:', item.id);
  return <Text>{item.name}</Text>;
}

// ✅ GOOD - Memo previene re-render inutili
const Item = React.memo(({ item }) => {
  console.log('Render item:', item.id);
  return <Text>{item.name}</Text>;
});

// ✅ GOOD - useMemo per calcoli pesanti
function ExpensiveList({ data }) {
  const processedData = useMemo(() => {
    return data.map(item => complexCalculation(item));
  }, [data]); // Solo quando data cambia

  return <FlatList data={processedData} />;
}

// ✅ GOOD - useCallback per funzioni in props
function Parent() {
  const handlePress = useCallback(() => {
    console.log('Pressed');
  }, []); // Funzione stabile, non ricreata

  return <Child onPress={handlePress} />;
}
```

**Step 4 - Ottimizza FlatList:**
```javascript
// ✅ GOOD - Tutte le ottimizzazioni FlatList
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  
  // Performance ottimizzazioni
  initialNumToRender={10}  // Rende solo 10 item iniziali
  maxToRenderPerBatch={10}  // 10 item per batch
  windowSize={5}  // Tiene in memoria 5 viewports
  
  // Evita flash durante scroll
  removeClippedSubviews={true}  // Android
  
  // Ottimizza con getItemLayout se possibile
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
/>
```

**Step 5 - Ottimizza immagini:**
```javascript
// ❌ BAD - Immagini grandi non ottimizzate
<Image source={{ uri: 'https://example.com/huge-image.jpg' }} />

// ✅ GOOD - Usa FastImage
npm install react-native-fast-image

import FastImage from 'react-native-fast-image';

<FastImage
  source={{
    uri: 'https://example.com/image.jpg',
    priority: FastImage.priority.normal,
  }}
  resizeMode={FastImage.resizeMode.cover}
  style={{ width: 200, height: 200 }}
/>
```

**Step 6 - Abilita Hermes (se non già attivo):**

**iOS (ios/Podfile):**
```ruby
use_react_native!(
  :path => config[:reactNativePath],
  :hermes_enabled => true  # <--- Abilita Hermes
)
```

**Android (android/app/build.gradle):**
```gradle
project.ext.react = [
    enableHermes: true,  // <--- Abilita Hermes
]
```

Poi rebuilda:
```bash
cd ios && pod install && cd ..
npx react-native run-ios
npx react-native run-android
```

---

## 🌐 Networking

### ❌ Problema: `Network request failed` su iOS

**Sintomi:**
```javascript
[TypeError: Network request failed]
// Fetch calls fail solo su iOS fisico, emulatore funziona
```

**Causa:** App Transport Security (ATS) blocca HTTP non-sicuro su iOS.

**Soluzione:**

**Per development - Allow HTTP (ios/Info.plist):**
```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
  <!-- RIMUOVI per production! -->
</dict>
```

**Per production - Whitelist domini specifici:**
```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSExceptionDomains</key>
  <dict>
    <key>example.com</key>
    <dict>
      <key>NSExceptionAllowsInsecureHTTPLoads</key>
      <true/>
      <key>NSIncludesSubdomains</key>
      <true/>
    </dict>
  </dict>
</dict>
```

**Soluzione migliore - Usa HTTPS:**
```javascript
// ✅ GOOD
const response = await fetch('https://api.example.com/data');
```

---

### ❌ Problema: CORS errors durante fetch

**Sintomi:**
```javascript
Access to fetch at 'https://api.example.com' from origin 'http://localhost:8081' 
has been blocked by CORS policy
```

**Causa:** Backend non ha headers CORS corretti. **Nota:** Questo NON dovrebbe accadere in app mobile native!

**Soluzione:**

1. **Verifica che NON stai testando in browser web:**
```bash
# React Native app NATIVA non ha problemi CORS
# CORS è solo un problema browser

# Se vedi CORS, probabilmente stai usando:
npx expo start --web  # <-- Expo web mode (browser)
```

2. **Se usi Expo Web, configura backend con CORS:**
```javascript
// Express.js example
const cors = require('cors');
app.use(cors({
  origin: 'http://localhost:8081',
  credentials: true,
}));
```

3. **Per app nativa, usa proxy per development:**
```javascript
// package.json (solo per Metro proxy)
{
  "proxy": "https://api.example.com"
}

// Poi fetch relativo
fetch('/api/endpoint');  // Proxato a https://api.example.com/api/endpoint
```

---

## 🐛 Debug

### ❌ Problema: `console.log` non mostra nulla

**Sintomi:**
- `console.log()` non appare in terminale
- Logs spariscono

**Causa:** Metro non mostra logs o logger overridden.

**Soluzione:**

1. **Apri Remote Debugger:**
```bash
# iOS: Cmd+D > Debug
# Android: Ctrl+M > Debug

# Logs appariranno in Chrome DevTools Console
```

2. **Usa Flipper (consigliato):**
```bash
# Installa Flipper Desktop
# https://fbflipper.com/

# In app già configurato per RN 0.62+
# Logs appariranno automaticamente in Flipper
```

3. **Verifica non ci siano override:**
```javascript
// Cerca nel codice se console è overridden:
console.log = () => {};  // <-- Rimuovi se presente
```

4. **Usa LogBox invece di console per errori:**
```javascript
// Per errori critici
import { LogBox } from 'react-native';

// Ignora warnings specifici (SOLO per development!)
LogBox.ignoreLogs(['Warning: ...']);

// Disabilita tutto (NON consigliato)
LogBox.ignoreAllLogs();
```

---

### ❌ Problema: Breakpoints non funzionano in VS Code

**Sintomi:**
- Breakpoints grigi (non attivi)
- Debugger non si ferma

**Causa:** Launch configuration non corretta.

**Soluzione:**

1. **Installa React Native Tools extension:**
```bash
# VS Code
# Extensions > Search "React Native Tools" > Install
```

2. **Crea/aggiorna .vscode/launch.json:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug iOS",
      "cwd": "${workspaceFolder}",
      "type": "reactnative",
      "request": "launch",
      "platform": "ios"
    },
    {
      "name": "Debug Android",
      "cwd": "${workspaceFolder}",
      "type": "reactnative",
      "request": "launch",
      "platform": "android"
    },
    {
      "name": "Attach to packager",
      "cwd": "${workspaceFolder}",
      "type": "reactnative",
      "request": "attach"
    }
  ]
}
```

3. **Usa debugger:**
```bash
# Start Metro
npx react-native start

# In VS Code: Run > Start Debugging (F5)
# Seleziona "Debug iOS" o "Debug Android"
```

---

## 🍎 iOS Specifico

### ❌ Problema: `Code signing error` su iOS

**Sintomi:**
```bash
error: Signing for "YourApp" requires a development team. 
Select a development team in the Signing & Capabilities editor.
```

**Causa:** Nessun team di sviluppo configurato.

**Soluzione:**

1. **Apri progetto in Xcode:**
```bash
open ios/YourApp.xcworkspace
```

2. **Configura signing:**
- Seleziona target "YourApp" nella sidebar sinistra
- Tab "Signing & Capabilities"
- Spunta "Automatically manage signing"
- Team: Seleziona il tuo Apple ID (o team)

3. **Se non hai Apple Developer Account:**
- Xcode > Preferences > Accounts > + > Add Apple ID
- Usa il tuo normale Apple ID (gratuito per sviluppo su dispositivo)

4. **Build da Xcode o CLI:**
```bash
npx react-native run-ios
```

---

### ❌ Problema: `Xcode version too old`

**Sintomi:**
```bash
error React Native requires Xcode version 14.0 or higher
You have Xcode 13.4.1
```

**Causa:** Xcode obsoleto.

**Soluzione:**

1. **Aggiorna Xcode:**
- App Store > Aggiornamenti > Xcode

2. **Seleziona Xcode Command Line Tools:**
```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

3. **Verifica versione:**
```bash
xcodebuild -version
# Xcode 14.0 o superiore
```

4. **Accetta licenza:**
```bash
sudo xcodebuild -license accept
```

---

## 🤖 Android Specifico

### ❌ Problema: `SDK location not found` (Android)

**Sintomi:**
```bash
* What went wrong:
SDK location not found. Define location with an ANDROID_SDK_ROOT 
environment variable or by setting the sdk.dir path in your project's 
local properties file at '/path/to/project/android/local.properties'.
```

**Causa:** ANDROID_HOME non configurato.

**Soluzione:**

**Step 1 - Trova SDK path:**
```bash
# Dovrebbe essere:
# macOS: /Users/[USERNAME]/Library/Android/sdk
# Linux: /home/[USERNAME]/Android/Sdk
# Windows: C:\Users\[USERNAME]\AppData\Local\Android\Sdk

# Oppure da Android Studio:
# Preferences > Appearance & Behavior > System Settings > Android SDK
# Copia "Android SDK Location"
```

**Step 2 - Imposta variabili ambiente:**

**macOS/Linux (.bashrc o .zshrc):**
```bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin

# Applica
source ~/.zshrc  # o ~/.bashrc
```

**Windows (System Environment Variables):**
```
ANDROID_HOME = C:\Users\[USERNAME]\AppData\Local\Android\Sdk
Path += %ANDROID_HOME%\emulator
Path += %ANDROID_HOME%\platform-tools
Path += %ANDROID_HOME%\tools
Path += %ANDROID_HOME%\tools\bin
```

**Step 3 - Verifica:**
```bash
echo $ANDROID_HOME
adb version
```

**Step 4 - Rebuilda:**
```bash
cd android
./gradlew clean
cd ..
npx react-native run-android
```

---

### ❌ Problema: `Gradle build failed` con errore memoria

**Sintomi:**
```bash
Expiring Daemon because JVM heap space is exhausted
FAILURE: Build failed with an exception.
* What went wrong:
Gradle build daemon disappeared unexpectedly
```

**Causa:** Heap memory insufficiente per Gradle.

**Soluzione:**

**Crea/Modifica android/gradle.properties:**
```properties
# Aumenta memoria Gradle
org.gradle.jvmargs=-Xmx4096m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# Altre ottimizzazioni
org.gradle.parallel=true
org.gradle.configureondemand=true
org.gradle.daemon=true
```

Poi rebuilda:
```bash
cd android
./gradlew clean
cd ..
npx react-native run-android
```

---

## 📱 Expo

### ❌ Problema: `Expo Go app keeps reloading`

**Sintomi:**
- Expo Go si ricarica continuamente
- Loop infinito di reload

**Causa:** Errore JavaScript che crashha app prima che errore sia mostrato.

**Soluzione:**

1. **Controlla Metro bundler output:**
```bash
# Nel terminale dove hai fatto `expo start`
# Cerca errori di sintassi o import
```

2. **Commenta codice recente:**
```javascript
// Commenta l'ultimo codice aggiunto
// Trova quale componente causa il crash
```

3. **Usa Error Boundary:**
```javascript
// ErrorBoundary.tsx
import React from 'react';
import { View, Text, Button } from 'react-native';

class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.log('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', padding: 20 }}>
          <Text style={{ color: 'red', marginBottom: 20 }}>
            Errore: {this.state.error?.toString()}
          </Text>
          <Button title="Reload" onPress={() => this.setState({ hasError: false })} />
        </View>
      );
    }
    return this.props.children;
  }
}

// App.tsx
export default function App() {
  return (
    <ErrorBoundary>
      <YourApp />
    </ErrorBoundary>
  );
}
```

4. **Disabilita Fast Refresh temporaneamente:**
```bash
# Expo Dev Menu (shake device o Cmd+D/Ctrl+M)
# Disable Fast Refresh
```

---

### ❌ Problema: `Expo modules not found after update`

**Sintomi:**
```bash
Error: Cannot find module 'expo-constants'
// dopo aver fatto expo upgrade
```

**Causa:** Dipendenze Expo non aggiornate correttamente.

**Soluzione:**

1. **Usa expo install per reinstallare:**
```bash
# Reinstalla tutti i moduli Expo
npx expo install --fix

# Oppure singolarmente
npx expo install expo-constants expo-camera expo-location
```

2. **Pulisci e reinstalla:**
```bash
rm -rf node_modules
rm package-lock.json  # o yarn.lock
npm install

# iOS
cd ios
rm -rf Pods Podfile.lock
pod install
cd ..

# Restart
npx expo start --clear
```

---

## 📘 TypeScript

### ❌ Problema: `Cannot find module` TypeScript

**Sintomi:**
```typescript
import { MyComponent } from './components/MyComponent';
// Error: Cannot find module './components/MyComponent' or its corresponding type declarations.
```

**Causa:** File .tsx non riconosciuto o tsconfig.json non configurato.

**Soluzione:**

1. **Verifica estensione file:**
```bash
# Assicurati che il file sia .tsx o .ts
mv components/MyComponent.js components/MyComponent.tsx
```

2. **Verifica tsconfig.json:**
```json
{
  "extends": "@react-native/typescript-config/tsconfig.json",
  "compilerOptions": {
    "allowJs": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "isolatedModules": true,
    "jsx": "react-native",
    "lib": ["ES2019", "ES2020.Promise"],
    "moduleResolution": "node",
    "noEmit": true,
    "strict": true,
    "target": "esnext",
    "resolveJsonModule": true
  },
  "exclude": [
    "node_modules",
    "babel.config.js",
    "metro.config.js",
    "jest.config.js"
  ]
}
```

3. **Restart TypeScript server in VS Code:**
```
Cmd+Shift+P (Ctrl+Shift+P) > TypeScript: Restart TS Server
```

4. **Se usa path aliases:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    }
  }
}

// babel.config.js
module.exports = {
  plugins: [
    [
      'module-resolver',
      {
        root: ['./src'],
        alias: {
          '@components': './src/components',
          '@utils': './src/utils',
        },
      },
    ],
  ],
};

// Installa babel plugin
npm install --save-dev babel-plugin-module-resolver
```

---

### ❌ Problema: `Type errors` con librerie third-party

**Sintomi:**
```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';
// Could not find a declaration file for module '@react-native-async-storage/async-storage'
```

**Causa:** Libreria non ha types TypeScript.

**Soluzione:**

1. **Installa @types se disponibili:**
```bash
npm install --save-dev @types/react-native-vector-icons
```

2. **Se @types non esistono, crea custom types:**
```typescript
// types/custom.d.ts
declare module '@react-native-async-storage/async-storage' {
  const AsyncStorage: any;
  export default AsyncStorage;
}

// oppure più tipizzato
declare module '@react-native-async-storage/async-storage' {
  export interface AsyncStorageStatic {
    getItem(key: string): Promise<string | null>;
    setItem(key: string, value: string): Promise<void>;
    removeItem(key: string): Promise<void>;
    clear(): Promise<void>;
  }
  const AsyncStorage: AsyncStorageStatic;
  export default AsyncStorage;
}
```

3. **Aggiungi a tsconfig.json:**
```json
{
  "compilerOptions": {
    "typeRoots": ["./types", "./node_modules/@types"]
  },
  "include": ["src", "types"]
}
```

---

## 🚀 Quick Reference: Comandi Pulizia

Quando tutto il resto fallisce, prova il "**nuclear reset**":

```bash
#!/bin/bash
# reset.sh - Pulizia completa React Native

echo "🧹 Pulizia completa React Native..."

# 1. Pulisci cache Metro
rm -rf $TMPDIR/metro-*
rm -rf $TMPDIR/haste-map-*
rm -rf $TMPDIR/react-*

# 2. Pulisci node_modules
rm -rf node_modules
rm package-lock.json yarn.lock

# 3. Pulisci cache npm/yarn
npm cache clean --force
# o: yarn cache clean

# 4. iOS: Pulisci Pods e build
cd ios
rm -rf Pods Podfile.lock
rm -rf build
rm -rf ~/Library/Developer/Xcode/DerivedData
cd ..

# 5. Android: Pulisci build e gradle
cd android
rm -rf build
rm -rf app/build
./gradlew clean
cd ..

# 6. Reinstalla tutto
npm install

# 7. iOS: Reinstalla Pods
cd ios
pod install
cd ..

echo "✅ Pulizia completata! Ora prova:"
echo "npx react-native start --reset-cache"
echo "npx react-native run-ios (o run-android)"
```

Salva come `reset.sh`, rendi eseguibile (`chmod +x reset.sh`) e lancia (`./reset.sh`).

---

## 🆘 Risorse Utili

- **[React Native Issues](https://github.com/facebook/react-native/issues)** - Issue tracker ufficiale
- **[Stack Overflow](https://stackoverflow.com/questions/tagged/react-native)** - Q&A community
- **[Reactiflux Discord](https://www.reactiflux.com/)** - Chat community React/RN
- **[Expo Forums](https://forums.expo.dev/)** - Forum Expo specifico

---

> **💡 Tip:** Prima di cercare online, prova sempre il "nuclear reset" sopra. Risolve l'80% dei problemi cache-related!

[← Torna al README principale](../README.md)
