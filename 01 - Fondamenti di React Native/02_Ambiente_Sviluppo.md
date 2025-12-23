# Capitolo 2: Configurazione dell'Ambiente di Sviluppo

[← Precedente: Introduzione](01_Introduzione_React_Native.md) | [Torna all'Indice](../README.md) | [Prossimo: JavaScript e TypeScript →](03_JavaScript_TypeScript.md)

---

## Descrizione

In questo capitolo configureremo l'ambiente di sviluppo per React Native. Installeremo tutti gli strumenti necessari per creare, sviluppare e testare applicazioni React Native su iOS e Android. Esploreremo due approcci principali: React Native CLI (maggiore controllo) e Expo (più semplice e veloce).

La configurazione dell'ambiente di sviluppo è un passo fondamentale: una volta completata correttamente, potrai concentrarti interamente sulla scrittura del codice senza preoccuparti di problemi tecnici.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Installare Node.js e gestori di pacchetti (npm/yarn)
- [ ] Comprendere la differenza tra React Native CLI e Expo
- [ ] Configurare l'ambiente di sviluppo Android (JDK, Android Studio, SDK)
- [ ] Configurare l'ambiente di sviluppo iOS (solo macOS)
- [ ] Creare il tuo primo progetto React Native
- [ ] Eseguire l'app su emulatori/simulatori e dispositivi fisici
- [ ] Utilizzare Hot Reload e Fast Refresh
- [ ] Risolvere problemi comuni di configurazione

## 📚 Argomenti Teorici

1. [Prerequisiti e Requisiti di Sistema](#1-prerequisiti-e-requisiti)
2. [Installazione di Node.js e npm/yarn](#2-installazione-nodejs)
3. [React Native CLI vs Expo](#3-react-native-cli-vs-expo)
4. [Configurazione Android](#4-configurazione-android)
5. [Configurazione iOS](#5-configurazione-ios)
6. [Creazione del Primo Progetto](#6-creazione-primo-progetto)
7. [Emulatori e Dispositivi Fisici](#7-emulatori-e-dispositivi)
8. [Hot Reload e Fast Refresh](#8-hot-reload-fast-refresh)
9. [Troubleshooting Comune](#9-troubleshooting)

---

## 1. Prerequisiti e Requisiti

### Requisiti Hardware

**Per sviluppo Android:**
- CPU: Intel Core i5 / AMD Ryzen 5 o superiore
- RAM: Minimo 8 GB (consigliato 16 GB)
- Spazio disco: 20-30 GB liberi
- OS: Windows 10/11, macOS, o Linux (Ubuntu/Debian)

**Per sviluppo iOS:**
- Mac con macOS 12 (Monterey) o superiore
- CPU: Apple Silicon (M1/M2/M3) o Intel Core i5+
- RAM: Minimo 8 GB (consigliato 16 GB)
- Spazio disco: 30-40 GB liberi
- Xcode 14+ installato

### Software da Installare

```
✅ Node.js (v18 LTS o superiore)
✅ Package manager (npm o yarn)
✅ Editor di codice (VS Code consigliato)
✅ Git (per version control)
✅ Android Studio (per Android)
✅ Xcode (per iOS, solo macOS)
```

### Conoscenze Prerequisite

- Utilizzo base del terminale/command line
- Familiarità con editor di codice
- Comprensione di base di JavaScript (Cap. 3)

---

## 2. Installazione Node.js

Node.js è l'ambiente runtime JavaScript necessario per React Native.

### Windows

**Metodo 1: Installer ufficiale**

```bash
# 1. Scarica da https://nodejs.org
# 2. Scegli la versione LTS (Long Term Support)
# 3. Esegui l'installer
# 4. Verifica l'installazione:

node --version
# Output: v18.x.x o superiore

npm --version
# Output: 9.x.x o superiore
```

**Metodo 2: Con Chocolatey**

```bash
# Apri PowerShell come amministratore
choco install nodejs-lts
```

### macOS

**Metodo 1: Con Homebrew (consigliato)**

```bash
# Installa Homebrew (se non già presente)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Installa Node.js
brew install node@18

# Verifica
node --version
npm --version
```

**Metodo 2: Con nvm (Node Version Manager)**

```bash
# Installa nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Riavvia il terminale, poi:
nvm install 18
nvm use 18
nvm alias default 18
```

### Linux (Ubuntu/Debian)

```bash
# Metodo con NodeSource
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verifica
node --version
npm --version
```

### npm vs Yarn

**npm** (incluso con Node.js):
```bash
npm install -g npm@latest  # Aggiorna npm
```

**Yarn** (alternativa più veloce):
```bash
# Installa Yarn
npm install -g yarn

# Verifica
yarn --version
```

**Quale scegliere?**
- **npm**: Standard, sempre disponibile, documentazione ufficiale
- **Yarn**: Più veloce, lockfile più deterministico, migliore per progetti grandi

💡 **Consiglio**: Inizia con npm, passa a Yarn se hai bisogno di maggiore velocità.

---

## 3. React Native CLI vs Expo

Esistono due modi principali per iniziare con React Native:

### React Native CLI

**Descrizione**: Ambiente completo con accesso diretto al codice nativo.

**✅ Vantaggi:**
- Controllo completo sul progetto
- Accesso a tutte le funzionalità native
- Possibilità di scrivere moduli nativi custom
- Nessuna dipendenza da servizi terzi
- Migliore per app complesse

**❌ Svantaggi:**
- Setup più complesso
- Richiede Android Studio e/o Xcode
- Build nativo necessario
- Curva di apprendimento più ripida

**Quando usarlo:**
- App con requisiti nativi specifici
- Integrazione di librerie native personalizzate
- Controllo completo del build process
- App enterprise

### Expo

**Descrizione**: Piattaforma che semplifica lo sviluppo React Native.

**✅ Vantaggi:**
- Setup velocissimo (5 minuti)
- Non serve Android Studio/Xcode per iniziare
- Build in cloud (EAS)
- Hot reload su dispositivo fisico tramite app Expo Go
- Molte API già pronte (camera, location, etc.)
- Over-the-air updates facili

**❌ Svantaggi:**
- Dimensione app più grande (~30-40 MB in più)
- Alcune limitazioni su moduli nativi
- Dipendenza dai servizi Expo
- Meno controllo sul build

**Quando usarlo:**
- Prototipi rapidi
- Principianti
- App che non necessitano moduli nativi custom
- Sviluppo veloce

### Tabella Comparativa

| Caratteristica | React Native CLI | Expo |
|----------------|------------------|------|
| **Setup** | 1-2 ore | 5 minuti |
| **Requisiti** | Android Studio/Xcode | Solo Node.js |
| **Moduli nativi** | Tutti | Limitati (ma tanti) |
| **Dimensione app** | ~20 MB | ~40-50 MB |
| **Build** | Locale | Cloud (EAS) |
| **Curva apprendimento** | Alta | Bassa |
| **OTA Updates** | CodePush | Expo Updates |

### Approccio Ibrido: Expo con Development Builds

Dal 2021, Expo offre **Development Builds** che permettono di:
- Usare Expo ma con moduli nativi custom
- Avere il meglio di entrambi i mondi

```bash
# Puoi sempre "eject" da Expo se necessario
npx expo prebuild
```

### Cosa Useremo in Questo Corso?

**Inizieremo con React Native CLI** per:
- Capire come funziona React Native "sotto il cofano"
- Avere controllo completo
- Imparare il setup Android/iOS

**Poi esploreremo Expo** nel Capitolo 24 per:
- Vedere i vantaggi per sviluppo rapido
- Usare EAS Build e Updates
- Confrontare i due approcci

---

## 4. Configurazione Android

### 4.1 Installazione Java Development Kit (JDK)

React Native richiede JDK 11 o 17.

**Windows/macOS/Linux:**

```bash
# Con Homebrew (macOS)
brew install openjdk@17

# Con apt (Linux)
sudo apt install openjdk-17-jdk

# Windows: Scarica da
# https://adoptium.net/temurin/releases/
```

**Verifica installazione:**

```bash
java -version
# Output: openjdk version "17.x.x"
```

**Imposta JAVA_HOME (importante!):**

**macOS/Linux (~/.bashrc o ~/.zshrc):**
```bash
export JAVA_HOME=$(/usr/libexec/java_home)
```

**Windows (Variabili d'ambiente):**
```
JAVA_HOME = C:\Program Files\OpenJDK\jdk-17
Path += %JAVA_HOME%\bin
```

### 4.2 Installazione Android Studio

1. **Scarica Android Studio** da: https://developer.android.com/studio

2. **Durante l'installazione, assicurati di selezionare:**
   - ✅ Android SDK
   - ✅ Android SDK Platform
   - ✅ Android Virtual Device (AVD)

3. **Primo avvio - SDK Manager:**
   - Apri Android Studio
   - More Actions → SDK Manager
   - SDK Platforms tab:
     - ✅ Android 13.0 (Tiramisu) - API Level 33
     - ✅ Android 12.0 (S) - API Level 31
   - SDK Tools tab:
     - ✅ Android SDK Build-Tools
     - ✅ Android Emulator
     - ✅ Android SDK Platform-Tools
     - ✅ Android SDK Tools (se disponibile)

### 4.3 Configurazione Variabili d'Ambiente Android

**macOS/Linux (~/.bashrc, ~/.zshrc, o ~/.bash_profile):**

```bash
export ANDROID_HOME=$HOME/Library/Android/sdk  # macOS
# export ANDROID_HOME=$HOME/Android/Sdk        # Linux

export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
```

**Windows (Variabili d'ambiente di sistema):**

```
ANDROID_HOME = C:\Users\TuoNome\AppData\Local\Android\Sdk

Path += %ANDROID_HOME%\platform-tools
Path += %ANDROID_HOME%\emulator
Path += %ANDROID_HOME%\tools
Path += %ANDROID_HOME%\tools\bin
```

**Verifica configurazione:**

```bash
# Riavvia il terminale, poi:
echo $ANDROID_HOME
adb --version
emulator -version
```

### 4.4 Creazione Android Virtual Device (AVD)

```bash
# Da terminale
android avdmanager create avd \
  -n Pixel_6_API_33 \
  -k "system-images;android-33;google_apis;x86_64" \
  -d "pixel_6"

# O da Android Studio:
# Tools → Device Manager → Create Device
# - Seleziona Pixel 6
# - API Level 33 (Tiramisu)
# - Finish
```

---

## 5. Configurazione iOS

⚠️ **Nota: Solo per macOS**

### 5.1 Installazione Xcode

1. **Scarica Xcode** dall'App Store (gratuito, ~15 GB)
2. **Installa Command Line Tools:**

```bash
xcode-select --install
```

3. **Accetta la licenza:**

```bash
sudo xcodebuild -license accept
```

### 5.2 Installazione CocoaPods

CocoaPods è il gestore di dipendenze per iOS.

```bash
# Metodo 1: Con gem (Ruby)
sudo gem install cocoapods

# Metodo 2: Con Homebrew
brew install cocoapods

# Verifica
pod --version
```

### 5.3 Configurazione Simulatore iOS

```bash
# Apri Xcode
open -a Xcode

# Vai a: Xcode → Settings → Platforms
# Scarica iOS Simulator per la versione desiderata

# Lista simulatori disponibili
xcrun simctl list devices
```

### 5.4 Configurazione per Dispositivi Fisici

Per testare su iPhone/iPad reale:

1. **Iscrizione Apple Developer** (€99/anno per pubblicare, gratis per testare)
2. **Certificati e Provisioning Profiles** in Xcode
3. **Trust del computer** sul dispositivo

---

## 6. Creazione Primo Progetto

### 6.1 Con React Native CLI

```bash
# Installa React Native CLI globalmente
npm install -g react-native-cli

# Crea nuovo progetto
npx react-native@latest init MyFirstApp

# Con TypeScript
npx react-native@latest init MyFirstApp --template react-native-template-typescript

# Entra nella cartella
cd MyFirstApp
```

**Struttura del progetto creato:**

```
MyFirstApp/
├── android/          # Codice nativo Android
├── ios/              # Codice nativo iOS
├── node_modules/     # Dipendenze npm
├── App.tsx           # Componente principale
├── index.js          # Entry point
├── package.json      # Configurazione npm
├── metro.config.js   # Bundler configuration
└── tsconfig.json     # TypeScript config
```

### 6.2 Esecuzione su Android

```bash
# Metodo 1: Comandi separati (consigliato per debug)
# Terminale 1 - Metro Bundler
npm start

# Terminale 2 - Build e run Android
npm run android
# oppure
npx react-native run-android

# Metodo 2: Tutto insieme
npm run android
```

**Cosa succede:**
1. Metro bundler compila JavaScript
2. Gradle build dell'app Android
3. Installazione sull'emulatore/device
4. App si avvia

### 6.3 Esecuzione su iOS (solo macOS)

```bash
# Prima volta: installa dipendenze CocoaPods
cd ios
pod install
cd ..

# Run iOS
npm run ios
# oppure
npx react-native run-ios

# Specifica simulatore
npx react-native run-ios --simulator="iPhone 15 Pro"
```

### 6.4 Con Expo

```bash
# Installa Expo CLI
npm install -g expo-cli

# Crea progetto
npx create-expo-app MyExpoApp

cd MyExpoApp

# Avvia
npx expo start

# Scansiona QR code con:
# - iOS: Camera app
# - Android: Expo Go app
```

---

## 7. Emulatori e Dispositivi

### 7.1 Android Emulator

**Avvio da Android Studio:**
```bash
# Device Manager → Play button
```

**Avvio da terminale:**
```bash
# Lista AVD disponibili
emulator -list-avds

# Avvia emulatore specifico
emulator -avd Pixel_6_API_33
```

**Comandi ADB utili:**
```bash
# Lista device connessi
adb devices

# Installa APK
adb install app-debug.apk

# Logs
adb logcat

# Screenshot
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png

# Riavvia ADB (se problemi)
adb kill-server
adb start-server
```

### 7.2 iOS Simulator

**Avvio da Xcode:**
```
Xcode → Open Developer Tool → Simulator
```

**Da terminale:**
```bash
# Apri simulator di default
open -a Simulator

# Lista dispositivi
xcrun simctl list devices

# Avvia simulator specifico
xcrun simctl boot "iPhone 15 Pro"
```

**Comandi utili:**
```bash
# Screenshot
xcrun simctl io booted screenshot screenshot.png

# Video recording
xcrun simctl io booted recordVideo video.mp4
# Ctrl+C per fermare

# Reset simulator
xcrun simctl erase all
```

### 7.3 Dispositivi Fisici

**Android:**
1. Abilita "Opzioni sviluppatore" sul dispositivo:
   - Impostazioni → Info telefono
   - Tap 7 volte su "Numero build"
2. Abilita "Debug USB"
3. Connetti via USB
4. Autorizza il computer sul dispositivo
5. Verifica: `adb devices`

**iOS:**
1. Connetti iPhone/iPad via USB
2. Trust del computer sul device
3. In Xcode: Window → Devices and Simulators
4. Seleziona il tuo dispositivo
5. Run da Xcode o React Native CLI

---

## 8. Hot Reload e Fast Refresh

### Fast Refresh (React Native 0.61+)

**Abilitato di default!**

```javascript
// Modifica App.tsx
export default function App() {
  return (
    <View>
      <Text>Hello World!</Text>  // Cambia questo
    </View>
  );
}

// Salva (Cmd+S / Ctrl+S)
// L'app si aggiorna ISTANTANEAMENTE! 🚀
```

**Come funziona:**
- Rileva modifiche ai file
- Re-esegue solo i componenti modificati
- Mantiene lo stato dell'app
- Aggiornamento in ~1 secondo

**Differenze:**

| Feature | Hot Reload (old) | Fast Refresh (new) |
|---------|------------------|-------------------|
| Mantiene stato | ❌ A volte | ✅ Sempre |
| Errori JS | 🔴 Crash | 🟡 Overlay |
| Modifiche file | Uno alla volta | Multiple files |
| Attivazione | Manuale | Automatica |

### Dev Menu

**Android:** Shake device o `Cmd+M` (Mac) / `Ctrl+M` (Windows)
**iOS:** Shake device o `Cmd+D`

**Opzioni utili:**
- 🔄 Reload
- 🐛 Debug
- ⚡ Fast Refresh toggle
- 🔍 Show Inspector
- 📊 Show Performance Monitor

---

## 9. Troubleshooting

### Problema: "Command not found: react-native"

**Soluzione:**
```bash
# Usa npx invece
npx react-native run-android

# Oppure reinstalla
npm install -g react-native-cli
```

### Problema: Build Android fallita

**Soluzione 1: Clean e rebuild**
```bash
cd android
./gradlew clean
cd ..
npm run android
```

**Soluzione 2: Verifica ANDROID_HOME**
```bash
echo $ANDROID_HOME  # Deve puntare alla cartella SDK
```

### Problema: Metro Bundler su porta occupata

**Soluzione:**
```bash
# Kill processo sulla porta 8081
# macOS/Linux
lsof -ti:8081 | xargs kill -9

# Windows
netstat -ano | findstr :8081
taskkill /PID <PID> /F
```

### Problema: "Unable to load script"

**Soluzione:**
```bash
# Riavvia Metro con reset cache
npm start -- --reset-cache
```

### Problema: iOS CocoaPods errori

**Soluzione:**
```bash
cd ios
pod deintegrate
pod install
cd ..
```

### Problema: Xcode "No development team"

**Soluzione:**
1. Apri `ios/MyApp.xcworkspace` in Xcode
2. Seleziona il progetto
3. Signing & Capabilities
4. Seleziona il tuo Apple ID in Team

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Installazione Ambiente

**Obiettivo**: Configurare completamente l'ambiente di sviluppo

**Task**:
1. Installa Node.js e verifica la versione
2. Installa Android Studio e SDK
3. Configura le variabili d'ambiente
4. Verifica che `adb` funzioni
5. (macOS) Installa Xcode e CocoaPods

**Verifica**:
```bash
node --version
npm --version
adb --version
echo $ANDROID_HOME
# macOS: pod --version
```

### Esercizio 2: 🟢 Primo Progetto

**Obiettivo**: Creare ed eseguire la prima app

**Task**:
```bash
# 1. Crea progetto
npx react-native@latest init HelloWorld

# 2. Naviga nella cartella
cd HelloWorld

# 3. Avvia su Android
npm run android

# 4. (macOS) Avvia su iOS
npm run ios
```

**Verifica**: L'app mostra "Welcome to React Native"

### Esercizio 3: 🟡 Modifica e Fast Refresh

**Obiettivo**: Sperimentare con Fast Refresh

**Task**:
1. Apri `App.tsx` nel progetto creato
2. Modifica il testo del componente `<Text>`
3. Salva e osserva Fast Refresh
4. Aggiungi un nuovo componente `<View>`
5. Prova ad inserire un errore intenzionale e vedi l'overlay

**Esempio modifica**:
```typescript
// App.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello React Native!</Text>
      <Text>Questa è la mia prima app! 🚀</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#282c34',
  },
  title: {
    fontSize: 24,
    color: '#61dafb',
    fontWeight: 'bold',
  },
});
```

### Esercizio 4: 🟡 Confronto CLI vs Expo

**Obiettivo**: Capire le differenze pratiche

**Task**:
1. Crea un progetto con React Native CLI (già fatto)
2. Crea un progetto con Expo:
   ```bash
   npx create-expo-app HelloExpo
   cd HelloExpo
   npx expo start
   ```
3. Confronta:
   - Tempo di setup
   - Struttura delle cartelle
   - Dimensione `node_modules`
   - Esperienza di sviluppo

**Scrivi un report breve** (200 parole) sulle differenze osservate.

### Esercizio 5: 🔴 Setup Completo Multi-Piattaforma

**Obiettivo**: Testare l'app su tutti i target possibili

**Task**:
1. Crea un progetto TypeScript
2. Eseguilo su:
   - Emulatore Android
   - Dispositivo Android fisico
   - (macOS) Simulatore iOS
   - (macOS) iPhone/iPad fisico
3. Prendi screenshot da ogni dispositivo
4. Documenta eventuali differenze UI tra piattaforme

---

## 📝 Quiz di Autovalutazione

1. **Qual è il requisito minimo di RAM per sviluppo React Native?**
   - [ ] 4 GB
   - [x] 8 GB
   - [ ] 16 GB
   - [ ] 32 GB

2. **Cosa è Metro Bundler?**
   - [ ] Un emulatore Android
   - [x] Il JavaScript bundler di React Native
   - [ ] Un gestore di pacchetti
   - [ ] Un framework CSS

3. **Differenza principale tra React Native CLI e Expo?**
   - [x] CLI richiede Android Studio/Xcode, Expo no
   - [ ] Expo non può creare app native
   - [ ] CLI è più lento
   - [ ] Expo non supporta iOS

4. **Cosa fa Fast Refresh?**
   - [ ] Ricarica l'intera app
   - [x] Aggiorna solo i componenti modificati mantenendo lo stato
   - [ ] Compila più velocemente
   - [ ] Pulisce la cache

5. **Comando per avviare Metro Bundler?**
   - [x] npm start
   - [ ] npm run metro
   - [ ] npm bundle
   - [ ] react-native start-bundler

**Risposte**: 1-b, 2-b, 3-a, 4-b, 5-a

---

## 🔗 Risorse Aggiuntive

### Documentazione Ufficiale
- [Setting up the development environment](https://reactnative.dev/docs/environment-setup)
- [Running On Device](https://reactnative.dev/docs/running-on-device)
- [Expo Documentation](https://docs.expo.dev/)

### Tool Utili
- [Reactotron](https://github.com/infinitered/reactotron) - Debugging
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)
- [Flipper](https://fbflipper.com/) - Debugging platform di Facebook

### Troubleshooting
- [React Native Common Issues](https://reactnative.dev/docs/troubleshooting)
- [Stack Overflow - React Native tag](https://stackoverflow.com/questions/tagged/react-native)

---

## ✅ Checklist di Completamento

- [ ] Node.js installato e verificato
- [ ] Android Studio installato con SDK
- [ ] Variabili d'ambiente configurate correttamente
- [ ] (macOS) Xcode e CocoaPods installati
- [ ] Primo progetto creato con successo
- [ ] App eseguita su emulatore/simulatore
- [ ] Fast Refresh testato e funzionante
- [ ] Dev Menu esplorato
- [ ] Completati almeno 3 esercizi su 5
- [ ] Quiz superato con almeno 4/5

---

## 📌 Punti Chiave da Ricordare

1. 📦 **Node.js è essenziale** - installa la versione LTS
2. 🤖 **Android Studio include tutto** per sviluppo Android
3. 🍎 **macOS obbligatorio** per sviluppo iOS
4. ⚡ **Fast Refresh è magico** - salva e vedi i cambiamenti istantaneamente
5. 🔧 **React Native CLI = controllo**, Expo = velocità
6. 🎯 **Variabili d'ambiente** sono critiche - configurale bene
7. 📱 **Testa su dispositivi reali** appena possibile
8. 🐛 **Dev Menu (Cmd+D / Cmd+M)** è il tuo migliore amico

---

**Ottimo lavoro! 🎉** Ora hai un ambiente di sviluppo completo e funzionante. Sei pronto per iniziare a scrivere codice React Native. Nel prossimo capitolo approfondiremo JavaScript e TypeScript per React Native!

[← Precedente: Introduzione](01_Introduzione_React_Native.md) | [Torna all'Indice](../README.md) | [Prossimo: JavaScript e TypeScript →](03_JavaScript_TypeScript.md)
