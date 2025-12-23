# 01ter. Expo EAS Advanced - Guida Completa Production

[← Torna all'Indice](README.md)

---

## Parte 1: EAS Build Deep Dive

---

## 1. Setup EAS Account e Progetto

**📚 Teoria: EAS (Expo Application Services)**

```
EAS = Suite di servizi cloud per build, update e submit di app Expo

Componenti:
1. EAS Build   → Build cloud iOS/Android
2. EAS Update  → Over-The-Air updates
3. EAS Submit  → Auto-submit a App Store/Play Store
4. EAS Metadata → Gestione metadata store

Vantaggi vs build locale:
✅ Build iOS senza Mac
✅ Gestione certificates automatica
✅ Build reproducibili e cachable
✅ Team collaboration (shared builds)
✅ CI/CD ready
```

### Setup Iniziale

**Step 1: Installa EAS CLI**

```bash
# Global installation (recommended)
$ npm install -g eas-cli

# Verifica installazione
$ eas --version
# Output: eas-cli/x.x.x

# Oppure: usa npx (no global install)
$ npx eas-cli@latest --version
```

**Step 2: Login a Expo Account**

```bash
$ eas login

# Inserisci credentials:
# Email: your-email@example.com
# Password: ********

# Verifica login
$ eas whoami
# Output: Logged in as your-username
```

**Step 3: Configura Progetto per EAS**

```bash
# Naviga a progetto Expo esistente
$ cd my-expo-app

# Inizializza EAS configuration
$ eas build:configure

# Output:
✔ Select platform: all (iOS and Android)
✔ Generated eas.json
✔ Linked to project: @your-username/my-expo-app

# Viene creato file eas.json!
```

### File: eas.json Explained

```json
/**
 * eas.json - EAS Build Configuration
 * Location: project root
 */
{
  "cli": {
    "version": ">= 5.0.0"  // Minimum EAS CLI version required
  },
  "build": {
    // Profile: development
    "development": {
      "developmentClient": true,  // Build Development Client (custom Expo Go)
      "distribution": "internal",  // Internal distribution (TestFlight, Direct download)
      "ios": {
        "simulator": false        // Build for real device, not simulator
      }
    },
    
    // Profile: preview
    "preview": {
      "distribution": "internal",  // Internal testing
      "ios": {
        "simulator": false
      },
      "android": {
        "buildType": "apk"        // APK (easier to distribute), not AAB
      }
    },
    
    // Profile: production
    "production": {
      "distribution": "store",    // App Store/Play Store distribution
      "ios": {
        "autoIncrement": true     // Auto-increment build number
      },
      "android": {
        "buildType": "app-bundle" // AAB (required by Play Store)
      }
    }
  },
  
  "submit": {
    "production": {
      // Auto-submit configuration (vedi sezione Submit)
    }
  }
}
```

**🔑 Key Concepts:**

```
Build Profile = Named configuration (development, preview, production)

Puoi creare profiles custom:
{
  "build": {
    "staging": {
      "extends": "production",
      "distribution": "internal",
      "env": {
        "API_URL": "https://staging-api.example.com"
      }
    }
  }
}

Build command:
$ eas build --platform ios --profile staging
```

---

## 2. iOS Build senza Mac

**📚 Teoria: iOS Build Cloud**

```
Tradizionalmente: Build iOS richiede Mac + Xcode

EAS Build: Build iOS nel cloud, funziona su Windows/Linux!

Flow:
1. EAS: Legge progetto + eas.json
2. EAS: Provision certificate + provisioning profile (automatic!)
3. EAS Cloud: Build con Xcode su macchine cloud
4. Output: IPA file + TestFlight upload (opzionale)
```

### Primo Build iOS

```bash
# Build Development
$ eas build --platform ios --profile development

# EAS workflow:
✔ Checking project configuration
✔ Ensuring project is configured for building
✔ Compressing project files
✔ Uploading to EAS Build
✔ Starting build...

Build URL: https://expo.dev/accounts/your-username/projects/my-app/builds/build-id

# Aspetta ~15-20 minuti...
# Al termine: Download IPA file
```

**Output Build:**

```
Build completo:
1. IPA file (iOS App Package)
   - Download da EAS website
   - Installa con TestFlight o direct install (se device registrato)

2. Build artifacts:
   - Xcode logs
   - Build size report
   - Certificate info
```

### Gestione Certificates Automatica

**EAS gestisce automaticamente certificates iOS!**

```bash
# First build: EAS crea certificates per te
$ eas build --platform ios

# EAS asks:
? Set up credentials for your iOS app? (Y/n)
> Yes

# EAS:
✔ Created Distribution Certificate
✔ Created Provisioning Profile
✔ Registered UDID (se device connected)
✔ Uploaded certificates to EAS

# Certificates stored nel cloud, riusabili da team!
```

**Credentials management:**

```bash
# View credentials
$ eas credentials

# Output:
iOS Credentials for @your-username/my-app:
├── Distribution Certificate (expires: 2026-01-01)
├── Push Notification Key
└── Provisioning Profiles:
    ├── Development (Ad Hoc)
    └── Production (App Store)

# Remove credentials (force regeneration)
$ eas credentials --platform ios --clear

# Use local credentials (if you have .p12 file)
$ eas build --platform ios --local-credentials
```

### TestFlight Distribution

```bash
# Build + Auto-upload a TestFlight
$ eas build --platform ios --profile production --auto-submit

# EAS:
1. Build IPA
2. Upload a App Store Connect
3. Submit a TestFlight
4. Send email: "Build ready for testing"

# Testers ricevono notifica TestFlight!
```

**Manual TestFlight upload:**

```bash
# 1. Build
$ eas build --platform ios --profile production

# 2. Download IPA da EAS dashboard

# 3. Upload con EAS Submit
$ eas submit --platform ios --path ./path/to/app.ipa

# Oppure: usa Transporter app (macOS)
```

---

## 3. Android Build e Signing

**📚 Teoria: Android Signing**

```
Android richiede:
1. Keystore file (.jks) → Contiene key per firmare app
2. Key alias + passwords → Proteggono keystore

⚠️ IMPORTANTE: Perdi keystore = non puoi più aggiornare app su Play Store!

EAS: Genera keystore automaticamente e lo archivia nel cloud (safe!)
```

### Primo Build Android

```bash
# Build Development
$ eas build --platform android --profile development

# EAS workflow:
✔ Generating Keystore
✔ Compressing project
✔ Uploading to EAS Build
✔ Starting build...

Build URL: https://expo.dev/...

# Aspetta ~10-15 minuti...
# Output: APK o AAB file
```

### APK vs App Bundle (AAB)

```typescript
/**
 * APK = Android Package (file singolo, installa direttamente)
 * AAB = Android App Bundle (ottimizzato, richiesto da Play Store)
 */

// eas.json
{
  "build": {
    "preview": {
      "android": {
        "buildType": "apk"  // ✅ Per testing interno (installa direct)
      }
    },
    "production": {
      "android": {
        "buildType": "app-bundle"  // ✅ Per Play Store (required!)
      }
    }
  }
}
```

**APK Distribution (Internal Testing):**

```bash
# Build APK
$ eas build --platform android --profile preview

# Download APK da EAS dashboard
# Invia APK a testers via email/link
# Testers: Install APK (abilita "Install Unknown Apps")

# ⚡ APK = Instant install, no store needed!
```

**AAB Distribution (Play Store):**

```bash
# Build AAB
$ eas build --platform android --profile production

# Download AAB da EAS dashboard

# Upload a Play Store:
$ eas submit --platform android --path ./app.aab

# Oppure: manuale upload su Play Console
```

### Keystore Management

```bash
# View keystore info
$ eas credentials --platform android

# Output:
Android Credentials for @your-username/my-app:
└── Keystore:
    ├── Key Alias: my-app-key
    ├── SHA1: xx:xx:xx:...
    └── SHA256: yy:yy:yy:...

# Download keystore (backup!)
$ eas credentials --platform android
# Select: Download Keystore

# ⚠️ BACKUP KEYSTORE in safe place!
# Se perdi keystore = non puoi aggiornare app su Play Store
```

**Use existing keystore:**

```bash
# Se hai già keystore da progetto precedente:
$ eas credentials --platform android

# Select:
> Set up a new Android Keystore
> Use an existing Keystore

# Upload .jks file + provide passwords
```

---

## 4. OTA Updates con EAS Update

**📚 Teoria: Over-The-Air Updates**

```
OTA Update = Update JavaScript/assets senza app store review

Come funziona:
1. User apre app → Check for updates (automatic)
2. Se update disponibile → Download in background
3. Al prossimo app restart → Usa nuovo JS bundle

Limiti:
❌ Non puoi aggiornare native code (Swift/Kotlin)
❌ Non puoi modificare Info.plist/AndroidManifest
✅ Puoi aggiornare JS, styles, images, logic

Perfetto per:
✅ Bug fixes
✅ UI tweaks
✅ Business logic changes
✅ Content updates
```

### Setup EAS Update

```bash
# 1. Installa expo-updates
$ npx expo install expo-updates

# 2. Configure EAS Update
$ eas update:configure

# Output:
✔ Created eas.json "update" section
✔ Updated app.json with channel configuration

# 3. Verifica app.json
```

**app.json configuration:**

```json
{
  "expo": {
    "name": "My App",
    "runtimeVersion": "1.0.0",  // ⚠️ IMPORTANTE: Compatibilità native
    "updates": {
      "url": "https://u.expo.dev/your-project-id"  // EAS Update endpoint
    }
  }
}
```

**🔑 Runtime Version:**

```
runtimeVersion = Compatibility ID tra native code e JS updates

Regola:
- Stesso runtimeVersion → Update compatible
- Diverso runtimeVersion → Update non applicato (require new build)

Best practice:
- Increment runtimeVersion quando modifichi native code
- Usa "1.0.0" → "1.0.1" → "1.1.0" semantic versioning
```

### Pubblicare Update

```bash
# Publish update a channel "production"
$ eas update --branch production --message "Fix login bug"

# EAS:
✔ Compressing JavaScript bundle
✔ Uploading assets (images, fonts)
✔ Publishing update...

Update URL: https://expo.dev/...
Branch: production
Runtime version: 1.0.0

# ✅ Update live! Users riceveranno update al prossimo app restart
```

**Workflow completo:**

```bash
# Development flow:
$ npm start  # Test locally

# Quando ready:
$ eas update --branch staging --message "Feature X implementation"

# Test su staging build:
# (build con channel="staging" in eas.json)

# Se OK:
$ eas update --branch production --message "Release v1.2.3"

# 🎉 Production users ricevono update!
```

---

## 5. Deployment Strategies

### Branches e Channels

**📚 Teoria: Branches vs Channels**

```
Branch = Stream of updates (es. "production", "staging", "feature-x")
Channel = Link tra Build e Branch

Workflow:
1. Build → Assigned to Channel (es. "production-channel")
2. Channel → Points to Branch (es. "production" branch)
3. EAS Update → Publish to Branch
4. Users con quella Build → Ricevono update da quel Branch

Example:
Build A → Channel "production" → Branch "production" → Users get updates
Build B → Channel "staging" → Branch "staging" → Testers get updates
```

**eas.json configuration:**

```json
{
  "build": {
    "production": {
      "distribution": "store",
      "channel": "production"  // ← This build listens to "production" branch
    },
    "staging": {
      "distribution": "internal",
      "channel": "staging"  // ← This build listens to "staging" branch
    }
  }
}
```

**Workflow example:**

```bash
# 1. Build production (assigned to "production" channel)
$ eas build --platform all --profile production

# 2. Publish update to "production" branch
$ eas update --branch production --message "v1.0.1 bug fixes"

# Result: Users con production build ricevono update!

# 3. Build staging (assigned to "staging" channel)
$ eas build --platform all --profile staging

# 4. Publish update to "staging" branch
$ eas update --branch staging --message "Testing feature X"

# Result: Solo staging testers ricevono update!
```

### Rollback Strategy

```bash
# View update history
$ eas update:list --branch production

# Output:
Updates for branch "production":
├── 2024-01-15 | v1.0.3 | Fix payment bug
├── 2024-01-14 | v1.0.2 | Update UI
└── 2024-01-10 | v1.0.1 | Initial release

# Rollback to previous update
$ eas update:rollback --branch production

# Select update:
> v1.0.2 | Update UI

# ✅ Republished v1.0.2 as latest
# Users ricevono v1.0.2 al prossimo restart!
```

### A/B Testing con Channels

```json
// eas.json - Multiple channels for A/B testing
{
  "build": {
    "production-a": {
      "distribution": "store",
      "channel": "production-variant-a"
    },
    "production-b": {
      "distribution": "store",
      "channel": "production-variant-b"
    }
  }
}
```

```bash
# Build variant A
$ eas build --profile production-a

# Build variant B
$ eas build --profile production-b

# Publish different updates:
$ eas update --branch production-variant-a --message "New UI design"
$ eas update --branch production-variant-b --message "Old UI (control)"

# Distribute 50% users → Build A, 50% → Build B
# Measure metrics → Choose winner!
```

---

## 6. Creare Config Plugins Custom

**📚 Teoria: Config Plugins Anatomy**

```
Config Plugin = JavaScript function che modifica native configuration

Signature:
const withMyPlugin = (config, options) => {
  // Modify config.ios or config.android
  return config;
};

Helpers disponibili (da @expo/config-plugins):
- withInfoPlist → Modify iOS Info.plist
- withAndroidManifest → Modify AndroidManifest.xml
- withEntitlementsPlist → Modify entitlements
- withAppDelegate → Modify AppDelegate.m/mm
- withMainActivity → Modify MainActivity.java/kt
```

### Esempio 1: Custom URL Scheme Plugin

```javascript
/**
 * File: plugins/withCustomUrlScheme.js
 * 
 * Plugin per aggiungere custom URL scheme (myapp://path)
 */
const { withInfoPlist, withAndroidManifest } = require('@expo/config-plugins');

/**
 * iOS: Modifica Info.plist
 */
function withCustomUrlSchemeIOS(config, { scheme }) {
  return withInfoPlist(config, (config) => {
    // Ottieni Info.plist object
    const infoPlist = config.modResults;
    
    // Aggiungi CFBundleURLTypes
    if (!infoPlist.CFBundleURLTypes) {
      infoPlist.CFBundleURLTypes = [];
    }
    
    infoPlist.CFBundleURLTypes.push({
      CFBundleURLSchemes: [scheme],
      CFBundleURLName: `com.${config.ios?.bundleIdentifier}.urlscheme`,
    });
    
    return config;
  });
}

/**
 * Android: Modifica AndroidManifest.xml
 */
function withCustomUrlSchemeAndroid(config, { scheme }) {
  return withAndroidManifest(config, (config) => {
    const manifest = config.modResults.manifest;
    
    // Trova MainActivity
    const mainActivity = manifest.application[0].activity.find(
      (activity) => activity.$['android:name'] === '.MainActivity'
    );
    
    if (!mainActivity['intent-filter']) {
      mainActivity['intent-filter'] = [];
    }
    
    // Aggiungi intent-filter per URL scheme
    mainActivity['intent-filter'].push({
      action: [{ $: { 'android:name': 'android.intent.action.VIEW' } }],
      category: [
        { $: { 'android:name': 'android.intent.category.DEFAULT' } },
        { $: { 'android:name': 'android.intent.category.BROWSABLE' } },
      ],
      data: [
        { 
          $: { 
            'android:scheme': scheme,
            'android:host': '*',
          } 
        },
      ],
    });
    
    return config;
  });
}

/**
 * Main plugin function
 */
const withCustomUrlScheme = (config, options = {}) => {
  const { scheme } = options;
  
  if (!scheme) {
    throw new Error('withCustomUrlScheme: "scheme" option is required');
  }
  
  // Apply iOS modifications
  config = withCustomUrlSchemeIOS(config, { scheme });
  
  // Apply Android modifications
  config = withCustomUrlSchemeAndroid(config, { scheme });
  
  return config;
};

module.exports = withCustomUrlScheme;
```

**Uso del plugin:**

```json
// app.json
{
  "expo": {
    "plugins": [
      ["./plugins/withCustomUrlScheme.js", { "scheme": "myapp" }]
    ]
  }
}
```

```bash
# Test plugin
$ npx expo prebuild --clean

# Verifica:
# ios/MyApp/Info.plist → CFBundleURLSchemes includes "myapp"
# android/app/src/main/AndroidManifest.xml → intent-filter with scheme="myapp"

# Test deep link:
# iOS: myapp://profile/123
# Android: myapp://profile/123
```

### Esempio 2: Custom Permissions Plugin

```javascript
/**
 * File: plugins/withCustomPermissions.js
 * 
 * Plugin per aggiungere permissions custom
 */
const { withInfoPlist, withAndroidManifest } = require('@expo/config-plugins');

function withCustomPermissionsIOS(config, { permissions }) {
  return withInfoPlist(config, (config) => {
    const infoPlist = config.modResults;
    
    // Aggiungi usage descriptions
    if (permissions.bluetooth) {
      infoPlist.NSBluetoothAlwaysUsageDescription = 
        permissions.bluetooth.message || 'App needs Bluetooth access';
    }
    
    if (permissions.backgroundLocation) {
      infoPlist.NSLocationAlwaysUsageDescription = 
        permissions.backgroundLocation.message || 'App needs background location';
      infoPlist.UIBackgroundModes = ['location'];
    }
    
    return config;
  });
}

function withCustomPermissionsAndroid(config, { permissions }) {
  return withAndroidManifest(config, (config) => {
    const manifest = config.modResults.manifest;
    
    if (!manifest['uses-permission']) {
      manifest['uses-permission'] = [];
    }
    
    // Aggiungi permissions
    if (permissions.bluetooth) {
      manifest['uses-permission'].push({
        $: { 'android:name': 'android.permission.BLUETOOTH' }
      });
      manifest['uses-permission'].push({
        $: { 'android:name': 'android.permission.BLUETOOTH_ADMIN' }
      });
    }
    
    if (permissions.backgroundLocation) {
      manifest['uses-permission'].push({
        $: { 'android:name': 'android.permission.ACCESS_BACKGROUND_LOCATION' }
      });
    }
    
    return config;
  });
}

const withCustomPermissions = (config, options = {}) => {
  const { permissions = {} } = options;
  
  config = withCustomPermissionsIOS(config, { permissions });
  config = withCustomPermissionsAndroid(config, { permissions });
  
  return config;
};

module.exports = withCustomPermissions;
```

**Uso:**

```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "./plugins/withCustomPermissions.js",
        {
          "permissions": {
            "bluetooth": {
              "message": "We need Bluetooth to connect to devices"
            },
            "backgroundLocation": {
              "message": "We track your runs in background"
            }
          }
        }
      ]
    ]
  }
}
```

### Pubblicare Plugin su npm

```bash
# 1. Create package
$ mkdir expo-config-plugin-custom-urls
$ cd expo-config-plugin-custom-urls
$ npm init -y

# 2. Create plugin file
$ touch app.plugin.js

# 3. Update package.json
{
  "name": "expo-config-plugin-custom-urls",
  "version": "1.0.0",
  "main": "app.plugin.js",
  "peerDependencies": {
    "@expo/config-plugins": "^7.0.0"
  }
}

# 4. Publish
$ npm publish

# 5. Use in other projects
$ npm install expo-config-plugin-custom-urls

// app.json
{
  "plugins": [
    ["expo-config-plugin-custom-urls", { "scheme": "myapp" }]
  ]
}
```

---

## 7. Environment Variables e Secrets

**📚 Teoria: Environment Management**

```
Problem: Diversi environments (dev, staging, prod) con diverse configs

Solution:
1. app.config.js → Dynamic configuration
2. EAS Secrets → Store sensitive data nel cloud
3. expo-constants → Read environment at runtime
```

### app.config.js (Dynamic Config)

```javascript
/**
 * File: app.config.js
 * Sostituisce app.json per config dinamica
 */
module.exports = ({ config }) => {
  const environment = process.env.APP_ENV || 'development';
  
  // Different configs per environment
  const configs = {
    development: {
      apiUrl: 'http://localhost:3000',
      sentryDsn: null, // No Sentry in dev
    },
    staging: {
      apiUrl: 'https://staging-api.example.com',
      sentryDsn: 'https://xxx@sentry.io/staging',
    },
    production: {
      apiUrl: 'https://api.example.com',
      sentryDsn: 'https://xxx@sentry.io/production',
    },
  };
  
  const envConfig = configs[environment];
  
  return {
    ...config,
    name: environment === 'production' ? 'MyApp' : `MyApp (${environment})`,
    slug: 'my-app',
    version: '1.0.0',
    
    // Extra config (readable via expo-constants)
    extra: {
      apiUrl: envConfig.apiUrl,
      sentryDsn: envConfig.sentryDsn,
      environment,
      eas: {
        projectId: 'your-project-id',
      },
    },
    
    // Conditional plugins
    plugins: [
      'expo-router',
      ...(envConfig.sentryDsn ? ['sentry-expo'] : []),
    ],
  };
};
```

**Read config in app:**

```typescript
// src/config.ts
import Constants from 'expo-constants';

const config = {
  apiUrl: Constants.expoConfig?.extra?.apiUrl as string,
  sentryDsn: Constants.expoConfig?.extra?.sentryDsn as string | null,
  environment: Constants.expoConfig?.extra?.environment as string,
};

export default config;

// Usage:
import config from './config';

fetch(`${config.apiUrl}/users`);
```

### EAS Secrets

```bash
# Store secret in EAS
$ eas secret:create --scope project --name API_KEY --value "sk_live_xxx" --type string

# List secrets
$ eas secret:list

# Output:
Secrets for @your-username/my-app:
├── API_KEY (string) - Created: 2024-01-15
└── STRIPE_KEY (string) - Created: 2024-01-10

# Delete secret
$ eas secret:delete --name API_KEY
```

**Use secret in build:**

```json
// eas.json
{
  "build": {
    "production": {
      "env": {
        "APP_ENV": "production",
        "API_KEY": "$EAS_SECRET(API_KEY)"  // ← Read from EAS Secrets
      }
    },
    "staging": {
      "env": {
        "APP_ENV": "staging",
        "API_KEY": "$EAS_SECRET(API_KEY_STAGING)"
      }
    }
  }
}
```

**Read in app.config.js:**

```javascript
// app.config.js
module.exports = ({ config }) => {
  return {
    ...config,
    extra: {
      apiKey: process.env.API_KEY,  // ← From EAS Secret
    },
  };
};
```

### .env Files (Local Development)

```bash
# Install dotenv
$ npm install dotenv

# Create .env files
$ touch .env.development .env.staging .env.production
```

```bash
# .env.development
APP_ENV=development
API_URL=http://localhost:3000
API_KEY=dev_key_123

# .env.production
APP_ENV=production
API_URL=https://api.example.com
API_KEY=prod_key_xxx
```

```javascript
// app.config.js
require('dotenv').config({
  path: `.env.${process.env.APP_ENV || 'development'}`,
});

module.exports = ({ config }) => {
  return {
    ...config,
    extra: {
      apiUrl: process.env.API_URL,
      apiKey: process.env.API_KEY,
      environment: process.env.APP_ENV,
    },
  };
};
```

**Build with specific env:**

```bash
# Development
$ APP_ENV=development eas build --profile development

# Production
$ APP_ENV=production eas build --profile production
```

---

## 8. CI/CD Integration

**📚 Teoria: Continuous Integration/Deployment**

```
CI/CD automatizza:
1. Testing (unit tests, linting)
2. Building (EAS Build)
3. Deployment (EAS Update, Submit)

Vantaggi:
✅ No manual builds
✅ Consistent process
✅ Fast iterations
✅ Team collaboration
```

### GitHub Actions Workflow

```yaml
# File: .github/workflows/eas-build.yml

name: EAS Build

on:
  push:
    branches: [main]  # Trigger on push to main
  pull_request:
    branches: [main]

jobs:
  build:
    name: Install and build
    runs-on: ubuntu-latest
    
    steps:
      # 1. Checkout code
      - uses: actions/checkout@v3
      
      # 2. Setup Node.js
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm
      
      # 3. Install dependencies
      - name: Install dependencies
        run: npm ci
      
      # 4. Run tests
      - name: Run tests
        run: npm test
      
      # 5. Setup Expo
      - name: Setup Expo
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      
      # 6. Build on EAS
      - name: Build on EAS
        run: eas build --platform all --non-interactive --no-wait
```

**Setup GitHub Secrets:**

```bash
# 1. Generate Expo token
$ eas whoami
$ npx expo login
# Visit: https://expo.dev/accounts/[username]/settings/access-tokens
# Create token → Copy

# 2. Add to GitHub Secrets:
# GitHub repo → Settings → Secrets and variables → Actions
# New repository secret:
# Name: EXPO_TOKEN
# Value: [paste token]
```

### Advanced Workflow: Build + Update + Submit

```yaml
# .github/workflows/release.yml

name: Release App

on:
  push:
    tags:
      - 'v*'  # Trigger on version tags (v1.0.0, v1.0.1)

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm
      
      - name: Install dependencies
        run: npm ci
      
      - name: Setup Expo
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      
      # Build for production
      - name: Build production
        run: |
          eas build --platform all --profile production --non-interactive --wait
      
      # Publish OTA update
      - name: Publish update
        run: |
          eas update --branch production --message "${{ github.event.head_commit.message }}"
      
      # Submit to stores (iOS App Store, Google Play)
      - name: Submit to stores
        run: |
          eas submit --platform all --latest --non-interactive
```

**Trigger release:**

```bash
# Create version tag
$ git tag v1.0.1
$ git push origin v1.0.1

# GitHub Actions:
1. ✔ Build iOS + Android
2. ✔ Publish EAS Update
3. ✔ Submit to App Store + Play Store
4. 🎉 Done!
```

---

## 9. Monitoring e Error Tracking

### Sentry Integration

```bash
# Install Sentry
$ npx expo install sentry-expo
$ npx expo install expo-dev-client  # Required for Sentry
```

**Configuration:**

```javascript
// app.config.js
module.exports = ({ config }) => {
  return {
    ...config,
    plugins: [
      [
        'sentry-expo',
        {
          organization: 'your-org',
          project: 'your-project',
        },
      ],
    ],
    extra: {
      sentryDsn: process.env.SENTRY_DSN,
    },
    hooks: {
      postPublish: [
        {
          file: 'sentry-expo/upload-sourcemaps',
          config: {
            organization: 'your-org',
            project: 'your-project',
            authToken: process.env.SENTRY_AUTH_TOKEN,
          },
        },
      ],
    },
  };
};
```

```typescript
// App.tsx
import * as Sentry from 'sentry-expo';
import Constants from 'expo-constants';

// Initialize Sentry
Sentry.init({
  dsn: Constants.expoConfig?.extra?.sentryDsn,
  enableInExpoDevelopment: false,  // Disable in Expo Go
  debug: __DEV__,
});

export default function App() {
  try {
    // Your app code
  } catch (error) {
    Sentry.Native.captureException(error);
  }
}
```

**Upload sourcemaps:**

```bash
# Build with Sentry
$ SENTRY_AUTH_TOKEN=your-token eas build --platform all --profile production

# EAS automatically uploads sourcemaps to Sentry!
```

---

## 10. Progetto: E-commerce App Production-Ready

### Architecture Overview

```
E-Commerce App Production Setup:
├── Authentication (Clerk/Supabase)
├── Product Catalog (API + local cache)
├── Shopping Cart (Redux/Zustand)
├── Checkout (Stripe payment)
├── Push Notifications (order updates)
└── EAS Build + Update workflow

Tech Stack:
- Expo SDK 50+
- React Native
- TypeScript
- Expo Router (file-based routing)
- Zustand (state management)
- React Query (API caching)
- Stripe (payments)
- EAS Build + Update
```

### Project Structure

```bash
my-ecommerce-app/
├── app/               # Expo Router screens
│   ├── (tabs)/       # Tab navigator
│   │   ├── index.tsx        # Home (product list)
│   │   ├── cart.tsx         # Shopping cart
│   │   └── profile.tsx      # User profile
│   ├── product/[id].tsx     # Product detail
│   ├── checkout.tsx         # Checkout flow
│   └── _layout.tsx          # Root layout
├── components/
│   ├── ProductCard.tsx
│   ├── CartItem.tsx
│   └── PaymentSheet.tsx
├── store/
│   └── cartStore.ts   # Zustand store
├── api/
│   └── products.ts    # API client
├── app.config.js      # Dynamic config
├── eas.json           # EAS configuration
└── package.json
```

### Implementation Highlights

**1. State Management (Zustand)**

```typescript
// store/cartStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image: string;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  total: number;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      
      addItem: (item) => {
        const existingItem = get().items.find((i) => i.id === item.id);
        if (existingItem) {
          set({
            items: get().items.map((i) =>
              i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
            ),
          });
        } else {
          set({ items: [...get().items, { ...item, quantity: 1 }] });
        }
      },
      
      removeItem: (id) => {
        set({ items: get().items.filter((i) => i.id !== id) });
      },
      
      updateQuantity: (id, quantity) => {
        if (quantity === 0) {
          get().removeItem(id);
        } else {
          set({
            items: get().items.map((i) =>
              i.id === id ? { ...i, quantity } : i
            ),
          });
        }
      },
      
      clearCart: () => set({ items: [] }),
      
      get total() {
        return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
      },
    }),
    {
      name: 'cart-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

**2. Checkout con Stripe**

```typescript
// components/PaymentSheet.tsx
import { useStripe } from '@stripe/stripe-react-native';
import { useCartStore } from '../store/cartStore';
import config from '../config';

export function PaymentSheet() {
  const { initPaymentSheet, presentPaymentSheet } = useStripe();
  const { total, clearCart } = useCartStore();
  
  const initializePayment = async () => {
    // 1. Create payment intent on server
    const response = await fetch(`${config.apiUrl}/create-payment-intent`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount: total * 100 }), // cents
    });
    const { clientSecret } = await response.json();
    
    // 2. Initialize payment sheet
    const { error } = await initPaymentSheet({
      paymentIntentClientSecret: clientSecret,
      merchantDisplayName: 'My Store',
    });
    
    if (error) {
      alert('Error initializing payment');
      return;
    }
    
    // 3. Present payment sheet
    const { error: paymentError } = await presentPaymentSheet();
    
    if (paymentError) {
      alert(`Payment failed: ${paymentError.message}`);
    } else {
      alert('Payment successful!');
      clearCart();
    }
  };
  
  return (
    <TouchableOpacity onPress={initializePayment}>
      <Text>Pay ${total.toFixed(2)}</Text>
    </TouchableOpacity>
  );
}
```

**3. Push Notifications (Order Updates)**

```typescript
// hooks/usePushNotifications.ts
import { useEffect, useRef } from 'react';
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import config from '../config';

export function usePushNotifications() {
  useEffect(() => {
    registerForPushNotificationsAsync();
    
    // Listen for notifications
    const subscription = Notifications.addNotificationReceivedListener((notification) => {
      const data = notification.request.content.data;
      if (data.type === 'order_update') {
        // Handle order update
        console.log('Order updated:', data.orderId);
      }
    });
    
    return () => subscription.remove();
  }, []);
}

async function registerForPushNotificationsAsync() {
  if (!Device.isDevice) return;
  
  const { status } = await Notifications.requestPermissionsAsync();
  if (status !== 'granted') return;
  
  const token = (await Notifications.getExpoPushTokenAsync()).data;
  
  // Send token to server
  await fetch(`${config.apiUrl}/users/push-token`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ token }),
  });
}
```

**4. EAS Configuration**

```json
// eas.json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "env": {
        "APP_ENV": "development"
      }
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview",
      "env": {
        "APP_ENV": "staging"
      }
    },
    "production": {
      "distribution": "store",
      "channel": "production",
      "env": {
        "APP_ENV": "production"
      },
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890"
      },
      "android": {
        "serviceAccountKeyPath": "./google-play-service-account.json"
      }
    }
  }
}
```

**5. Deploy Workflow**

```bash
# Development
$ npm start  # Test locally

# Staging
$ eas build --platform all --profile preview
$ eas update --branch preview --message "Testing checkout flow"

# Production
$ eas build --platform all --profile production
$ eas submit --platform all --latest
$ eas update --branch production --message "v1.2.0 - Improved checkout"
```

---

## Appendice A: Troubleshooting EAS

### Common Issues

**1. Build Failed: "Invalid Bundle Identifier"**

```
Error: Bundle identifier "com.yourname.app" is already used

Solution:
- Change slug in app.json: "slug": "my-app-unique"
- Or specify explicit bundleIdentifier:
  "ios": {
    "bundleIdentifier": "com.yourcompany.myapp"
  }
```

**2. iOS Build: "No Provisioning Profiles"**

```
Solution:
$ eas credentials
> Select platform: iOS
> Set up new credentials
> Follow prompts
```

**3. Update Not Received**

```
Check:
1. Runtime version matches:
   - App runtime: "1.0.0"
   - Update runtime: "1.0.0"
   
2. Channel configuration:
   - eas.json: "channel": "production"
   - Update published to: "production" branch
   
3. User restarted app (updates apply on restart)
```

---

## Appendice B: EAS CLI Reference

```bash
# Build
$ eas build --platform ios|android|all
$ eas build --profile development|preview|production
$ eas build --non-interactive  # No prompts
$ eas build --wait  # Wait for build to complete
$ eas build --clear-cache  # Clear build cache

# Update
$ eas update --branch BRANCH_NAME
$ eas update --message "Update description"
$ eas update:list  # List updates
$ eas update:rollback  # Rollback to previous

# Submit
$ eas submit --platform ios|android|all
$ eas submit --latest  # Submit latest build
$ eas submit --path ./app.ipa  # Submit specific file

# Credentials
$ eas credentials  # Manage credentials
$ eas credentials --platform ios|android

# Project
$ eas project:info  # Project information
$ eas project:init  # Initialize project

# Device
$ eas device:create  # Register device for development
$ eas device:list  # List registered devices
```

---

## Appendice C: Migration Guide

### Da Expo Go a Development Builds

```bash
# 1. Install expo-dev-client
$ npx expo install expo-dev-client

# 2. Update eas.json
{
  "build": {
    "development": {
      "developmentClient": true
    }
  }
}

# 3. Build
$ eas build --platform all --profile development

# 4. Install su device
# iOS: TestFlight or direct download
# Android: Download APK

# 5. Run
$ npx expo start --dev-client
```

### Da React Native CLI a Expo

```bash
# 1. Install expo
$ npx install-expo-modules@latest

# 2. Update package.json scripts
{
  "scripts": {
    "start": "expo start",
    "android": "expo run:android",
    "ios": "expo run:ios"
  }
}

# 3. Create app.json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0"
  }
}

# 4. Install Expo modules as needed
$ npx expo install expo-camera expo-location

# 5. Use EAS Build
$ eas build:configure
$ eas build --platform all
```

---

## ✅ Checklist Finale

- [ ] Ho creato account EAS e configurato progetto
- [ ] Ho eseguito primo build iOS senza Mac
- [ ] Ho eseguito primo build Android con signing automatico
- [ ] Ho pubblicato primo OTA update con EAS Update
- [ ] Ho creato almeno un Config Plugin custom
- [ ] Ho configurato environment variables (dev/staging/prod)
- [ ] Ho setup CI/CD con GitHub Actions
- [ ] Ho integrato error tracking (Sentry)
- [ ] Ho deployato app su TestFlight o Play Store Internal Testing
- [ ] Ho compreso workflow completo production

---

## 📌 Punti Chiave

1. 🏗️ **EAS Build**: Build cloud iOS/Android senza Mac, certificates automatici
2. 🔄 **EAS Update**: OTA updates per bug fixes rapidi, no app store review
3. 🔌 **Config Plugins**: Modify native config in modo declarativo e riusabile
4. 🔐 **Secrets Management**: EAS Secrets per API keys, .env per development
5. 🤖 **CI/CD**: GitHub Actions automatizza build/test/deploy
6. 📊 **Monitoring**: Sentry per error tracking production
7. 🚀 **Deploy**: Workflow completo dev → staging → production

---

**Congratulazioni! 🎉 Hai completato il modulo EAS Advanced!**

Ora sei pronto a deployare app Expo production-ready con workflow professionali. Continua con i Capitoli successivi del corso React Native per approfondire altri aspetti dello sviluppo mobile!

[← Torna all'Indice](README.md) | [Torna al Corso](../README.md)
