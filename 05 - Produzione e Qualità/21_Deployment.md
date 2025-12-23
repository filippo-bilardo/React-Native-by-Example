# Capitolo 21: Build e Deployment

[← Precedente: Ottimizzazione Performance](20_Performance.md) | [Torna all'Indice](../README.md) | [Prossimo: Monitoring e Analytics →](22_Monitoring.md)

---

## Descrizione

Il deployment è il momento cruciale: portare l'app da development a produzione, nelle mani degli utenti. iOS App Store e Google Play Store hanno processi complessi: certificates, provisioning profiles, build configurations, screenshots, compliance. Automatizzare con CI/CD risparmia tempo e riduce errori.

In questo capitolo imparerai a configurare build iOS e Android, gestire certificates, creare release builds, submittere su App Store e Play Store, implementare CI/CD con GitHub Actions, gestire environment variables, e implementare OTA updates con CodePush.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Creare iOS release build
- [ ] Creare Android release build
- [ ] Gestire certificates iOS
- [ ] Configurare signing Android
- [ ] Submittere su App Store
- [ ] Publicare su Play Store
- [ ] Implementare CI/CD pipeline
- [ ] Gestire environment variables
- [ ] Implementare OTA updates
- [ ] Gestire versioning

## 📚 Argomenti Teorici

1. [iOS Build](#1-ios-build)
2. [Android Build](#2-android-build)
3. [iOS Certificates](#3-ios-certificates)
4. [Android Signing](#4-android-signing)
5. [App Store Submission](#5-app-store)
6. [Play Store Publishing](#6-play-store)
7. [CI/CD Pipeline](#7-ci-cd)
8. [Environment Variables](#8-env-vars)
9. [OTA Updates](#9-ota-updates)
10. [Best Practices](#10-best-practices)

---

## 1. iOS Build

### Prerequisites

- Mac with Xcode installed
- Apple Developer Account ($99/year)
- iOS device for testing

### Configure Xcode Project

```bash
# Open Xcode project
cd ios
open YourApp.xcworkspace

# In Xcode:
# 1. Select project in navigator
# 2. Select target
# 3. General tab:
#    - Bundle Identifier: com.yourcompany.yourapp
#    - Version: 1.0.0
#    - Build: 1
# 4. Signing & Capabilities:
#    - Select team
#    - Enable "Automatically manage signing"
```

### Build for Simulator

```bash
npx react-native run-ios
# or
npx react-native run-ios --simulator="iPhone 14 Pro"
```

### Build for Device

```bash
# Build release
npx react-native run-ios --configuration Release --device "Your iPhone"
```

### Create Archive

```bash
# In Xcode:
# 1. Select "Any iOS Device (arm64)" as destination
# 2. Product > Archive
# 3. Wait for build to complete
# 4. Organizer window opens with archive
```

---

## 2. Android Build

### Generate Signing Key

```bash
# Generate keystore
keytool -genkeypair -v -storetype PKCS12 \
  -keystore my-release-key.keystore \
  -alias my-key-alias \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000

# Move to android/app
mv my-release-key.keystore android/app/
```

### Configure Gradle

```gradle
// android/app/build.gradle
android {
  defaultConfig {
    applicationId "com.yourapp"
    versionCode 1
    versionName "1.0.0"
  }
  
  signingConfigs {
    release {
      if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
        storeFile file(MYAPP_RELEASE_STORE_FILE)
        storePassword MYAPP_RELEASE_STORE_PASSWORD
        keyAlias MYAPP_RELEASE_KEY_ALIAS
        keyPassword MYAPP_RELEASE_KEY_PASSWORD
      }
    }
  }
  
  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled true
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
}
```

### Gradle Properties

```properties
# android/gradle.properties
MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=*****
MYAPP_RELEASE_KEY_PASSWORD=*****

# Add to .gitignore
android/gradle.properties
```

### Build Release APK

```bash
cd android
./gradlew assembleRelease

# Output: android/app/build/outputs/apk/release/app-release.apk
```

### Build Release AAB (App Bundle)

```bash
cd android
./gradlew bundleRelease

# Output: android/app/build/outputs/bundle/release/app-release.aab
```

---

## 3. iOS Certificates

### Certificate Types

- **Development Certificate**: For testing on devices
- **Distribution Certificate**: For App Store submission
- **Push Notification Certificate**: For push notifications

### Create Certificates

```bash
# Using Fastlane
gem install fastlane

cd ios
fastlane init

# Create certificates and provisioning profiles
fastlane match development
fastlane match appstore
```

### Manual Certificate Creation

1. Go to [developer.apple.com](https://developer.apple.com)
2. Certificates, Identifiers & Profiles
3. Create Certificate
4. Download and install
5. Create App ID
6. Create Provisioning Profile
7. Download and add to Xcode

### Provisioning Profiles

```bash
# Download provisioning profiles
fastlane match development --readonly
fastlane match appstore --readonly

# Refresh profiles
fastlane match development --force
```

---

## 4. Android Signing

### Keystore Information

```bash
# View keystore info
keytool -list -v -keystore my-release-key.keystore
```

### ProGuard Rules

```proguard
# android/app/proguard-rules.pro

# Keep React Native
-keep class com.facebook.react.** { *; }
-keep class com.facebook.hermes.** { *; }

# Keep app classes
-keep class com.yourapp.** { *; }

# Keep native methods
-keepclasseswithmembernames class * {
    native <methods>;
}
```

### Build Variants

```gradle
// android/app/build.gradle
android {
  flavorDimensions "version"
  
  productFlavors {
    dev {
      dimension "version"
      applicationIdSuffix ".dev"
      versionNameSuffix "-dev"
      resValue "string", "app_name", "YourApp Dev"
    }
    
    staging {
      dimension "version"
      applicationIdSuffix ".staging"
      versionNameSuffix "-staging"
      resValue "string", "app_name", "YourApp Staging"
    }
    
    production {
      dimension "version"
      resValue "string", "app_name", "YourApp"
    }
  }
}
```

---

## 5. App Store

### App Store Connect Setup

1. Go to [appstoreconnect.apple.com](https://appstoreconnect.apple.com)
2. My Apps > + > New App
3. Fill in app information:
   - Platform: iOS
   - Name: Your App Name
   - Primary Language
   - Bundle ID: Select from dropdown
   - SKU: Unique identifier

### Upload Build

```bash
# Using Xcode:
# 1. Archive the app (Product > Archive)
# 2. In Organizer, select archive
# 3. Click "Distribute App"
# 4. Select "App Store Connect"
# 5. Upload
# 6. Wait for processing

# Using Fastlane:
fastlane deliver
```

### App Store Information

```yaml
# Fastlane/Deliverfile
app_identifier "com.yourcompany.yourapp"
username "your@email.com"

# App metadata
name({ "en-US" => "Your App Name" })
description({
  "en-US" => "Your app description..."
})

keywords({
  "en-US" => "keywords, separated, by, commas"
})

# Screenshots
screenshots_path "./fastlane/screenshots"

# Submission
submit_for_review true
automatic_release true
```

### Submit for Review

1. Complete all required information:
   - App Privacy
   - App Information
   - Pricing and Availability
   - Screenshots (all required sizes)
   - App Preview (optional)
2. Click "Submit for Review"
3. Wait for Apple review (typically 1-3 days)

---

## 6. Play Store

### Google Play Console Setup

1. Go to [play.google.com/console](https://play.google.com/console)
2. Create Application
3. Fill in details:
   - Title
   - Short description
   - Full description
   - Graphics (icon, screenshots, feature graphic)
   - Categorization
   - Contact details
   - Privacy policy URL

### Upload Release

```bash
# Create release in Play Console:
# 1. Production > Create new release
# 2. Upload AAB file
# 3. Add release notes
# 4. Review and rollout

# Using Fastlane:
fastlane supply
```

### Fastlane Setup

```ruby
# Fastlane/Appfile
json_key_file("path/to/service-account.json")
package_name("com.yourcompany.yourapp")

# Fastlane/Fastfile
lane :deploy do
  gradle(
    task: "bundle",
    build_type: "Release"
  )
  
  upload_to_play_store(
    track: "production",
    aab: "app/build/outputs/bundle/release/app-release.aab"
  )
end
```

### Internal Testing

```bash
# Upload to internal testing track
fastlane supply --track internal --aab path/to/app.aab

# Add testers via email
# Share opt-in link
```

---

## 7. CI/CD

### GitHub Actions - iOS

```yaml
# .github/workflows/ios.yml
name: iOS Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: macos-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Pods
        run: |
          cd ios
          pod install
      
      - name: Build iOS
        run: |
          xcodebuild -workspace ios/YourApp.xcworkspace \
            -scheme YourApp \
            -configuration Release \
            -sdk iphoneos \
            -archivePath ios/build/YourApp.xcarchive \
            archive
      
      - name: Export IPA
        run: |
          xcodebuild -exportArchive \
            -archivePath ios/build/YourApp.xcarchive \
            -exportPath ios/build \
            -exportOptionsPlist ios/ExportOptions.plist
      
      - name: Upload to TestFlight
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
        run: |
          xcrun altool --upload-app \
            --type ios \
            --file ios/build/YourApp.ipa \
            --apiKey $APP_STORE_CONNECT_API_KEY
```

### GitHub Actions - Android

```yaml
# .github/workflows/android.yml
name: Android Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '11'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Create keystore
        run: |
          echo "${{ secrets.KEYSTORE }}" | base64 -d > android/app/release.keystore
      
      - name: Build AAB
        env:
          MYAPP_RELEASE_STORE_FILE: release.keystore
          MYAPP_RELEASE_KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          MYAPP_RELEASE_STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}
          MYAPP_RELEASE_KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: |
          cd android
          ./gradlew bundleRelease
      
      - name: Upload to Play Store
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.SERVICE_ACCOUNT_JSON }}
          packageName: com.yourcompany.yourapp
          releaseFiles: android/app/build/outputs/bundle/release/app-release.aab
          track: production
```

---

## 8. Env Vars

### React Native Config

```bash
npm install react-native-config
```

```bash
# .env
API_URL=https://api.example.com
API_KEY=your_api_key_here
ENVIRONMENT=production

# .env.development
API_URL=https://dev-api.example.com
API_KEY=dev_api_key
ENVIRONMENT=development
```

```typescript
// Usage
import Config from 'react-native-config';

const apiUrl = Config.API_URL;
const apiKey = Config.API_KEY;
const environment = Config.ENVIRONMENT;

console.log(`Environment: ${environment}`);
console.log(`API URL: ${apiUrl}`);
```

### iOS Configuration

```ruby
# ios/Podfile
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENVFILE'] = '.env.production'
    end
  end
end
```

### Android Configuration

```gradle
// android/app/build.gradle
project.ext.envConfigFiles = [
  debug: ".env.development",
  release: ".env.production"
]

apply from: project(':react-native-config').projectDir.getPath() + "/dotenv.gradle"
```

---

## 9. OTA Updates

### CodePush Setup

```bash
npm install --save react-native-code-push
npm install -g appcenter-cli

appcenter login
appcenter apps create -d MyApp-iOS -o iOS -p React-Native
appcenter apps create -d MyApp-Android -o Android -p React-Native
```

### iOS Configuration

```objective-c
// ios/AppDelegate.mm
#import <CodePush/CodePush.h>

- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
#if DEBUG
  return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index"];
#else
  return [CodePush bundleURL];
#endif
}
```

### Android Configuration

```java
// android/app/src/main/java/com/.../MainApplication.java
@Override
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
    new MainReactPackage(),
    new CodePush(getResources().getString(R.string.CodePushDeploymentKey), getApplicationContext(), BuildConfig.DEBUG)
  );
}
```

### Deploy Update

```typescript
// App.tsx
import codePush from 'react-native-code-push';

const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,
  installMode: codePush.InstallMode.ON_NEXT_RESUME
};

const App = () => {
  return <Navigation />;
};

export default codePush(codePushOptions)(App);

// Deploy
// appcenter codepush release-react -a <username>/MyApp-iOS -d Production
// appcenter codepush release-react -a <username>/MyApp-Android -d Production
```

---

## 10. Best Practices

1. **Versioning**: Use semantic versioning (1.0.0)
2. **Build Numbers**: Auto-increment in CI/CD
3. **Environment Separation**: Dev, Staging, Production
4. **Secrets Management**: Use CI/CD secrets, not in code
5. **Automated Testing**: Run tests before deployment
6. **Staged Rollout**: Start with small percentage
7. **Monitoring**: Set up crash reporting before launch
8. **Backup**: Keep keystores and certificates secure
9. **Documentation**: Document build and deployment process
10. **Rollback Plan**: Have OTA update rollback strategy

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Local Build

Build locale per entrambe piattaforme:
- iOS release build
- Android release APK
- Test on real devices
- Fix signing issues

### Esercizio 2: 🟡 App Store Submission

Submit test app:
- Complete App Store Connect setup
- Add screenshots and metadata
- Submit for review
- Handle rejection (if any)

### Esercizio 3: 🟡 Play Store Publishing

Publish to Play Store:
- Create release in console
- Upload AAB
- Internal testing track
- Add testers
- Production release

### Esercizio 4: 🔴 CI/CD Pipeline

Setup CI/CD completo:
- GitHub Actions workflow
- Automated testing
- Build for both platforms
- Deploy to TestFlight
- Deploy to Play Console

### Esercizio 5: 🔴 OTA Updates

Implementa CodePush:
- Setup CodePush
- Deploy initial release
- Make code change
- Deploy OTA update
- Verify update on device
- Rollback mechanism

---

## 📝 Quiz

1. **App Bundle (AAB) è:**
   - [ ] Solo iOS
   - [x] Android format ottimizzato
   - [ ] Per development
   - [ ] Tool di build

2. **CodePush permette:**
   - [x] OTA updates JavaScript
   - [ ] Native code updates
   - [ ] Solo bug fixes
   - [ ] Store bypass

3. **Provisioning profile serve per:**
   - [x] iOS app signing
   - [ ] Android signing
   - [ ] Entrambe piattaforme
   - [ ] Solo development

4. **CI/CD automatizza:**
   - [ ] Solo testing
   - [ ] Solo build
   - [x] Build, test, deploy
   - [ ] Solo deployment

5. **Semantic versioning format:**
   - [ ] 1.0
   - [x] 1.0.0 (major.minor.patch)
   - [ ] v1.0.0
   - [ ] 1.0.0.0

**Risposte**: 1-b, 2-a, 3-a, 4-c, 5-b

---

## 🔗 Risorse

- [iOS App Distribution](https://developer.apple.com/documentation/xcode/distributing-your-app-for-beta-testing-and-releases)
- [Android Publishing](https://developer.android.com/studio/publish)
- [Fastlane Documentation](https://docs.fastlane.tools/)
- [CodePush](https://github.com/microsoft/react-native-code-push)

---

## ✅ Checklist

- [ ] iOS build configurato
- [ ] Android build configurato
- [ ] Certificates gestiti
- [ ] Signing configurato
- [ ] App Store submission completata
- [ ] Play Store publishing completato
- [ ] CI/CD pipeline funzionante
- [ ] Environment variables gestite
- [ ] OTA updates implementati
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🍎 **iOS** richiede certificates e profiles
2. 🤖 **Android** usa keystore per signing
3. 📦 **AAB** format preferito per Play Store
4. 🏪 **App Store** review 1-3 giorni
5. 🚀 **CI/CD** automatizza deployment
6. 🔐 **Secrets** mai in repository
7. 🌍 **Environments** dev/staging/production
8. ⚡ **CodePush** per OTA updates
9. 📈 **Staged rollout** riduce rischi
10. 🔄 **Versioning** semantico e consistente

---

**Straordinario! 🎉** App deployata con successo! Prossimo: Monitoring e Analytics!

[← Precedente: Ottimizzazione Performance](20_Performance.md) | [Torna all'Indice](../README.md) | [Prossimo: Monitoring e Analytics →](22_Monitoring.md)
