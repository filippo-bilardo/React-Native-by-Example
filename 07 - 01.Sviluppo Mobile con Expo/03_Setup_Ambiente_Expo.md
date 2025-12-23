# 03. Setup Ambiente Expo - Guida Completa

[← Precedente: Expo Introduzione](02_Expo_Introduzione.md) | [Torna all'Indice](README.md) | [Prossimo: Primo Progetto →](04_Primo_Progetto_Expo.md)

---

## Descrizione

È il momento di **configurare l'ambiente di sviluppo Expo** sul tuo computer. In questo capitolo vedremo passo-passo l'installazione di tutti i tool necessari, dalla configurazione iniziale alla creazione del primo progetto funzionante.

A differenza di React Native CLI che richiede Xcode/Android Studio, Expo permette di iniziare con setup minimale e testare su device reale tramite Expo Go, senza emulatori o build native.

---

## 🎯 Obiettivi di Apprendimento

- [ ] Installare Node.js e npm/yarn correttamente
- [ ] Configurare Expo CLI e creare account Expo
- [ ] Installare e configurare VS Code con extensions Expo
- [ ] Testare setup con Expo Go su device reale
- [ ] Comprendere opzioni di testing (Expo Go vs Emulators vs Device)
- [ ] Risolvere problemi comuni di setup

---

## 📚 Contenuti

1. [Prerequisiti Sistema](#1-prerequisiti-sistema)
2. [Installazione Node.js e Package Manager](#2-installazione-nodejs-e-package-manager)
3. [Installazione Expo CLI](#3-installazione-expo-cli)
4. [Setup Editor: VS Code](#4-setup-editor-vs-code)
5. [Setup Device Testing: Expo Go](#5-setup-device-testing-expo-go)
6. [Opzionale: Setup Emulators](#6-opzionale-setup-emulators)
7. [Verifica Setup Completo](#7-verifica-setup-completo)
8. [Troubleshooting Comune](#8-troubleshooting-comune)

---

## 1. Prerequisiti Sistema

**📚 Requisiti Minimi**

### Hardware

```
Minimum (per development basic):
- CPU: Dual-core 2.0 GHz+
- RAM: 8 GB
- Disk: 10 GB spazio libero
- Network: WiFi (per Expo Go sync)

Recommended (per workflow fluido):
- CPU: Quad-core 2.5 GHz+ (i5/i7 o equivalente)
- RAM: 16 GB
- Disk: 20 GB SSD
- Network: WiFi 5 GHz o cavo ethernet
```

### Sistema Operativo

```
Supportati:
✅ macOS 10.15+ (Catalina o newer)
✅ Windows 10/11 (64-bit)
✅ Linux (Ubuntu 18.04+, Fedora, Arch, etc.)

Note:
- macOS: Best experience (se serve iOS development futuro)
- Windows: Ottimo per Android, iOS via EAS Build cloud
- Linux: Perfetto per development, iOS via EAS Build
```

### Device per Testing

```
Opzioni (in ordine di preferenza):

1. Device Fisico + Expo Go ⭐ CONSIGLIATO
   - iOS: iPhone/iPad con iOS 13+
   - Android: Smartphone con Android 5.0+ (API 21+)
   - Zero setup, instant reload
   
2. Emulators (opzionale)
   - iOS Simulator (solo su macOS, requires Xcode)
   - Android Emulator (richiede Android Studio)
   - Più lento di device fisico
   
3. EAS Build + TestFlight/Internal Testing
   - Per test su device senza Expo Go
   - Richiede build cloud (15-20 min)
```

**🎯 Per iniziare**: Serve solo laptop + smartphone. Zero emulators necessari!

---

## 2. Installazione Node.js e Package Manager

**📚 Teoria: Node.js e npm/yarn**

```
Node.js = JavaScript runtime (V8 engine)
Serve per:
- Eseguire JavaScript fuori dal browser
- Gestire dependencies (npm/yarn)
- Run Metro bundler (dev server Expo)

npm = Node Package Manager (bundled con Node)
yarn = Alternative package manager (by Facebook, optional)
```

### A) Installazione Node.js

**Versione consigliata: LTS (Long Term Support)**

#### macOS

```bash
# Opzione 1: Homebrew (recommended)
$ brew install node

# Verifica installazione
$ node --version
# Output: v20.x.x (o 18.x.x minimum)

$ npm --version
# Output: v10.x.x

# Opzione 2: Download da nodejs.org
# https://nodejs.org/en/download/
# Scarica .pkg installer e segui wizard
```

#### Windows

```powershell
# Opzione 1: Winget (Windows 11)
> winget install OpenJS.NodeJS.LTS

# Opzione 2: Chocolatey
> choco install nodejs-lts

# Opzione 3: Download da nodejs.org
# https://nodejs.org/en/download/
# Scarica .msi installer (64-bit) e installa

# Verifica (apri nuovo PowerShell/CMD):
> node --version
# Output: v20.x.x

> npm --version
# Output: v10.x.x
```

#### Linux (Ubuntu/Debian)

```bash
# NodeSource repository (recommended, versione latest)
$ curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
$ sudo apt-get install -y nodejs

# Verifica
$ node --version
# v20.x.x

$ npm --version
# v10.x.x

# Alternative: nvm (Node Version Manager)
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
$ source ~/.bashrc
$ nvm install --lts
$ nvm use --lts
```

### B) Installazione Yarn (Opzionale ma Consigliato)

```bash
# Yarn 1.x (Classic) - stable, Expo-tested
$ npm install -g yarn

# Verifica
$ yarn --version
# 1.22.x

# Oppure: Yarn 3+ (Berry) - modern ma richiede config
$ corepack enable
$ yarn set version stable
```

**💡 Consiglio**: Usa **yarn** per progetti Expo. È più veloce di npm e gestisce meglio monorepos.

---

## 3. Installazione Expo CLI

**📚 Teoria: Expo CLI**

```
Expo CLI = Command-line tool per:
- Creare progetti Expo
- Startare dev server (Metro bundler)
- Build & deploy con EAS
- Manage configurations

Due modalità:
1. Global install (classic, deprecated)
2. npx (moderno, recommended)
```

### Installazione (npx - Metodo Moderno)

```bash
# NON serve installare expo-cli globalmente!
# Usa npx per always latest version:

$ npx expo --version
# Scarica e esegue latest Expo CLI
# Output: 50.x.x (o versione corrente)

# Per creare progetto:
$ npx create-expo-app MyApp

# Per start dev server:
$ cd MyApp
$ npx expo start
```

**✅ Vantaggi npx:**
- Always latest Expo version
- Zero global pollution
- Meno conflitti tra progetti

### (Opzionale) Installazione Globale

```bash
# Se preferisci comando `expo` diretto:
$ npm install -g expo-cli

# Oppure con yarn:
$ yarn global add expo-cli

# Verifica:
$ expo --version
# 6.x.x

# Uso:
$ expo init MyApp  # Old syntax
$ create-expo-app MyApp  # New syntax (same)
```

**⚠️ Note**: Expo team raccomanda `npx` per avoid version conflicts.

### Creazione Account Expo (Necessario per EAS)

```bash
# Registrati online:
# https://expo.dev/signup

# Oppure da CLI:
$ npx expo register

# Inserisci:
# - Username (es: johndoe)
# - Email
# - Password

# Login da CLI:
$ npx expo login

# Verifica:
$ npx expo whoami
# Output: johndoe
```

**🎯 Account serve per:**
- EAS Build/Update/Submit
- Expo Go project sync
- Team collaboration
- Free tier: unlimited projects

---

## 4. Setup Editor: VS Code

**📚 Teoria: VS Code è l'editor raccomandato per Expo**

```
Alternative valide:
- WebStorm (JetBrains, paid ma potente)
- Sublime Text
- Vim/Neovim con plugins

Ma VS Code è:
✅ Free e open source
✅ Ottima integration Expo/React Native
✅ Extensions ecosystem enorme
✅ IntelliSense TypeScript perfetto
```

### Installazione VS Code

```bash
# macOS (Homebrew):
$ brew install --cask visual-studio-code

# Windows (Winget):
> winget install Microsoft.VisualStudioCode

# Linux (Ubuntu):
$ sudo snap install code --classic

# O download manuale:
# https://code.visualstudio.com/download
```

### Extensions Essenziali Expo

Installa queste extensions da VS Code Marketplace (Ctrl+Shift+X / Cmd+Shift+X):

#### 1. **React Native Tools** (Microsoft)

```
Extension ID: msjsdiag.vscode-react-native

Features:
- IntelliSense per React Native APIs
- Debug support
- Execute commands (run-ios, run-android)
- Integrated terminal
```

#### 2. **ES7+ React/Redux/React-Native snippets**

```
Extension ID: dsznajder.es7-react-js-snippets

Snippets:
- rnf → React Native functional component
- rnfs → React Native functional component + StyleSheet
- useState → const [state, setState] = useState()
- useEffect → useEffect(() => {}, [])
```

#### 3. **Prettier - Code Formatter**

```
Extension ID: esbenp.prettier-vscode

Auto-format code on save
Configurazione (.prettierrc):
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 80
}
```

#### 4. **ESLint**

```
Extension ID: dbaeumer.vscode-eslint

Linting real-time per:
- Syntax errors
- Code style issues
- Best practices violations
```

#### 5. **Path Intellisense**

```
Extension ID: christian-kohler.path-intellisense

Autocomplete per import paths:
import Button from './components/Button'
                   ^ autocomplete!
```

#### 6. **GitLens** (Opzionale ma Utile)

```
Extension ID: eamodio.gitlens

Git superpowers:
- Blame annotations inline
- Commit history
- Compare branches
```

### VS Code Settings per Expo

Crea `.vscode/settings.json` nel workspace:

```json
{
  // Format on save
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  
  // Auto import
  "typescript.updateImportsOnFileMove.enabled": "always",
  "javascript.updateImportsOnFileMove.enabled": "always",
  
  // Exclude folders from search
  "search.exclude": {
    "**/node_modules": true,
    "**/ios/Pods": true,
    "**/android/.gradle": true,
    "**/.expo": true
  },
  
  // File associations
  "files.associations": {
    "*.tsx": "typescriptreact"
  },
  
  // Emmet (JSX support)
  "emmet.includeLanguages": {
    "javascript": "javascriptreact",
    "typescript": "typescriptreact"
  }
}
```

### Keyboard Shortcuts Utili

```
macOS:
- Cmd+P: Quick file open
- Cmd+Shift+P: Command palette
- Cmd+B: Toggle sidebar
- Ctrl+`: Toggle terminal
- Cmd+D: Select next occurrence
- Option+Shift+F: Format document

Windows/Linux:
- Ctrl+P: Quick file open
- Ctrl+Shift+P: Command palette
- Ctrl+B: Toggle sidebar
- Ctrl+`: Toggle terminal
- Ctrl+D: Select next occurrence
- Shift+Alt+F: Format document
```

---

## 5. Setup Device Testing: Expo Go

**📚 Teoria: Expo Go App**

```
Expo Go = Container app per testare Expo projects

Come funziona:
1. Dev avvia `npx expo start` → Metro bundler running
2. Expo Go scansiona QR code (o inserisce URL)
3. App si carica within Expo Go
4. Hot reload automatico su file save

Limitations:
❌ Solo Expo SDK modules (no custom native)
❌ No background tasks advanced
✅ Perfetto per 90% development workflow
```

### Installazione Expo Go

#### iOS (iPhone/iPad)

```
1. Apri App Store
2. Cerca "Expo Go"
3. Install (free app)
4. Apri app → Accept permissions (camera, network)

Requisiti:
- iOS 13.0 o superiore
- WiFi sulla stessa rete del dev machine
```

#### Android

```
1. Apri Google Play Store
2. Cerca "Expo Go"
3. Install (free app)
4. Apri app → Accept permissions

Requisiti:
- Android 5.0 (API 21) o superiore
- WiFi sulla stessa rete del dev machine

Alternative:
- Download APK da: https://expo.dev/tools#client
- Install via USB con adb
```

### Collegare Device a Dev Machine

**Metodo 1: Scansione QR Code (Consigliato)**

```bash
# Start dev server:
$ npx expo start

# Output:
┌─────────────────────────────────────────────┐
│                                             │
│   ████ ████ █████  ███  ████                │
│   █    ▒    █  █  █   █ ▒  █                │
│   ███  ████ ████  █▒▒▒█ ████                │
│   █    ▒    █     █   █ ▒  █                │
│   ████ ████ █     █   █ ▒  █                │
│                                             │
│   QR code qui →  [███████████]              │
│                                             │
└─────────────────────────────────────────────┘

› Metro waiting on exp://192.168.1.10:8081
› Scan QR code with Expo Go (iOS) or Camera (Android)
```

**iOS**: Apri Expo Go app → Tap "Scan QR Code" → Point camera at QR  
**Android**: Apri Camera app → Point at QR → Tap notification "Open with Expo Go"

**Metodo 2: Inserimento URL Manuale**

```bash
# Ottieni URL da output:
exp://192.168.1.10:8081

# Su Expo Go app:
# → Tap "Enter URL manually"
# → Inserisci URL
# → Connect
```

**Metodo 3: Tunnel Mode (se WiFi diverso)**

```bash
# Usa ngrok tunnel (più lento ma bypassa network issues):
$ npx expo start --tunnel

# Expo crea tunnel pubblico:
exp://abc123xyz.ngrok.io

# Funziona anche con cellular data!
```

### Verifica Connessione

```bash
# Device connesso, dovresti vedere:
› Opening exp://192.168.1.10:8081 on iPhone di Mario

# Logs in real-time:
 LOG  App mounted
 LOG  Fetching data...

# Su device: App running in Expo Go!
# Modifica file → Cmd+S → App reload automatico in 1-2 sec
```

---

## 6. Opzionale: Setup Emulators

**📚 Quando Servono Emulators**

```
Usa emulators se:
- Non hai device fisico disponibile
- Vuoi testare su multiple screen sizes
- Debugging specifico OS/version

Ma considera:
⚠️ Emulators sono più lenti di device reali
⚠️ Consumano molta RAM/CPU
⚠️ iOS Simulator richiede macOS + Xcode (15+ GB)
✅ Expo Go su device reale è instant e sufficient per 90% casi
```

### A) iOS Simulator (solo macOS)

```bash
# 1. Installa Xcode da App Store (15+ GB, 1-2 ore download)
# 2. Apri Xcode → Preferences → Components
# 3. Download iOS simulators (es. iOS 17.0)

# 4. Installa Xcode Command Line Tools:
$ xcode-select --install

# 5. Test simulator:
$ open -a Simulator

# 6. Run Expo project su simulator:
$ npx expo start
# → Press 'i' per launch iOS Simulator
# → Expo Go app si installa automaticamente su simulator
```

**Pros:**
- Fast (più veloce di Android Emulator)
- Touch gestures simulati bene
- Debug tools integrati

**Cons:**
- Solo su macOS
- 15+ GB disk space
- RAM-hungry (4-8 GB)

### B) Android Emulator (tutti OS)

```bash
# 1. Installa Android Studio:
# Download da: https://developer.android.com/studio

# 2. Open Android Studio → More Actions → SDK Manager
# 3. Install:
#    - Android SDK Platform 33 (Android 13)
#    - Android SDK Build-Tools
#    - Android Emulator
#    - Intel HAXM (Windows/Linux) o Apple Hypervisor (macOS)

# 4. Create Virtual Device:
# Tools → Device Manager → Create Device
# → Select: Pixel 5 (common device)
# → Select system image: Android 13 (API 33)
# → Finish

# 5. Start emulator:
$ ~/Android/Sdk/emulator/emulator -avd Pixel_5_API_33

# 6. Run Expo:
$ npx expo start
# → Press 'a' per launch Android
# → Expo Go si installa automaticamente
```

**Pros:**
- Cross-platform (Windows/Mac/Linux)
- Multiple devices/versions
- Google Play Services included

**Cons:**
- Slow (anche con HAXM/KVM)
- RAM-hungry (4-8 GB)
- Disk space (10+ GB per emulator)

### C) Expo Dev Client su Emulator

```bash
# Per testare custom native modules:

# Build dev client per emulator:
$ eas build --profile development --platform android

# Download .apk
# Drag & drop su emulator

# Oppure build locale:
$ npx expo run:android  # Richiede Android SDK
$ npx expo run:ios      # Richiede Xcode (macOS)
```

---

## 7. Verifica Setup Completo

**📚 Checklist Finale**

```bash
# 1. Node.js installato
$ node --version
# ✅ v18.x.x o v20.x.x

# 2. npm/yarn installato
$ npm --version
# ✅ v9.x.x o v10.x.x

$ yarn --version
# ✅ v1.22.x (opzionale)

# 3. Expo CLI funzionante
$ npx expo --version
# ✅ 50.x.x

# 4. Account Expo logged in
$ npx expo whoami
# ✅ your-username

# 5. Expo Go installato su device
# ✅ Apri Expo Go app → vedi homepage

# 6. VS Code + extensions
# ✅ Apri VS Code → Extensions tab → verifica installate

# 7. Test end-to-end (prossimo capitolo!)
```

**Se tutti i check passano → SETUP COMPLETO! 🎉**

---

## 8. Troubleshooting Comune

### Problema 1: `expo: command not found`

```bash
# Causa: Expo CLI non installato o PATH issue

# Soluzione A: Usa npx (recommended)
$ npx expo --version

# Soluzione B: Install globalmente
$ npm install -g expo-cli
$ expo --version
```

### Problema 2: Expo Go non si connette

```bash
# Causa: Firewall o network diverse

# Soluzione 1: Verifica stessa WiFi
# Dev machine e device sulla stessa rete WiFi

# Soluzione 2: Disable firewall temporaneamente
# macOS: System Preferences → Security & Privacy → Firewall → Off
# Windows: Settings → Windows Defender Firewall → Turn off

# Soluzione 3: Usa tunnel
$ npx expo start --tunnel
# Più lento ma bypassa firewall
```

### Problema 3: Metro bundler errore "port 8081 already in use"

```bash
# Causa: Altra istanza Metro running

# Soluzione 1: Kill process
# macOS/Linux:
$ lsof -i :8081
$ kill -9 <PID>

# Windows:
> netstat -ano | findstr :8081
> taskkill /PID <PID> /F

# Soluzione 2: Usa altra porta
$ npx expo start --port 8082
```

### Problema 4: Node version troppo vecchia

```bash
# Verifica versione:
$ node --version
# v14.x.x ← Troppo vecchio! Serve v18+ per Expo 50+

# Soluzione: Update Node
# macOS:
$ brew upgrade node

# Windows:
# Download latest da nodejs.org e reinstalla

# Linux:
$ nvm install --lts
$ nvm use --lts
```

### Problema 5: Permission denied su npm install -g

```bash
# Causa: npm global folder needs sudo (bad practice!)

# Soluzione: Cambia npm prefix
$ mkdir ~/.npm-global
$ npm config set prefix '~/.npm-global'
$ echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
$ source ~/.bashrc

# Ora:
$ npm install -g expo-cli  # No sudo needed!
```

---

## ✅ Checklist di Completamento

- [ ] Node.js v18+ installato e funzionante
- [ ] npm o yarn installato
- [ ] Expo CLI accessibile via npx
- [ ] Account Expo creato e logged in
- [ ] VS Code installato con extensions Expo
- [ ] Expo Go installato su device iOS/Android
- [ ] Test connessione device ↔ dev machine riuscito

---

## 📌 Punti Chiave da Ricordare

1. ⚡ **Setup minimale**: Solo Node + Expo CLI + Expo Go su phone
2. 🚫 **No Xcode/Android Studio** necessari per iniziare
3. 📱 **Device reale > Emulator** per development Expo
4. 🔄 **npx expo** > global install (always latest version)
5. 🆓 **Tutto free**: Node, Expo, VS Code, Expo Go
6. ⏱️ **Tempo setup**: 15-30 minuti totale
7. 🔧 **Troubleshooting**: 90% problemi = firewall o WiFi diverso

---

[Prossimo: 04. Primo Progetto Expo →](04_Primo_Progetto_Expo.md)

[← Precedente: Expo Introduzione](02_Expo_Introduzione.md) | [Torna all'Indice](README.md)
