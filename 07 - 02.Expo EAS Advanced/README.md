# Capitolo 01ter: Expo - EAS e Workflow Avanzati

[← Torna al Corso React Native](../README.md) | [Prerequisito: 01bis Fondamenti](../01bis%20-%20Sviluppo%20Mobile%20con%20Expo/README.md)

---

## 📱 Introduzione al Modulo EAS Advanced

Dopo aver appreso i fondamenti di Expo nel modulo 01bis, è tempo di esplorare i **workflow avanzati per produzione**: EAS Build per build professionali, EAS Update per deployment OTA, Config Plugins custom, e best practices per app production-ready.

Questo modulo è dedicato a developer che vogliono portare le loro app Expo in **produzione** con processi professionali, automazione CI/CD, e deployment strategies robuste.

---

## 🎯 Obiettivi del Modulo

Al termine di questo modulo sarai in grado di:

- ✅ Configurare EAS Build per iOS e Android con profiles custom
- ✅ Gestire certificates e provisioning automaticamente
- ✅ Implementare deployment OTA con EAS Update
- ✅ Creare Config Plugins custom per native configuration
- ✅ Setup CI/CD pipeline con GitHub Actions
- ✅ Integrare monitoring e error tracking (Sentry)
- ✅ Gestire environment variables e secrets
- ✅ Deployare app production-ready su App Store e Play Store

---

## 📚 Contenuto del Capitolo Unico

Questo modulo è strutturato come **capitolo unico avanzato** (~1800 righe) con sezioni progressive:

### Parte 1: EAS Build Deep Dive
1. **[Setup EAS Account e Progetto](#1-setup-eas-account-e-progetto)**
   - Creazione account EAS
   - Configurazione eas.json
   - Build profiles (development, preview, production)

2. **[iOS Build senza Mac](#2-ios-build-senza-mac)**
   - Gestione certificates automatica
   - Provisioning profiles
   - Build e distribuzione TestFlight

3. **[Android Build e Signing](#3-android-build-e-signing)**
   - Keystore generation automatico
   - App Bundle vs APK
   - Internal testing e produzione

### Parte 2: EAS Update (OTA)
4. **[OTA Updates con EAS Update](#4-ota-updates-con-eas-update)**
   - Setup EAS Update
   - Publish updates
   - Branches e channels strategy

5. **[Deployment Strategies](#5-deployment-strategies)**
   - Staging vs Production
   - Rollback mechanism
   - A/B testing con channels

### Parte 3: Config Plugins Advanced
6. **[Creare Config Plugins Custom](#6-creare-config-plugins-custom)**
   - Anatomy di un plugin
   - Modificare Info.plist e AndroidManifest
   - Publish plugin su npm

### Parte 4: Production Best Practices
7. **[Environment Variables e Secrets](#7-environment-variables-e-secrets)**
   - expo-constants e app.config.js
   - EAS Secrets management
   - Different envs (dev, staging, prod)

8. **[CI/CD Integration](#8-cicd-integration)**
   - GitHub Actions workflow
   - Automated builds e deployments
   - Testing automation

9. **[Monitoring e Error Tracking](#9-monitoring-e-error-tracking)**
   - Sentry integration
   - Crashlytics alternative
   - Performance monitoring

### Parte 5: Progetto Completo
10. **[Progetto: E-commerce App Production-Ready](#10-progetto-e-commerce-app-production-ready)**
    - Architettura completa
    - Authentication flow
    - Payment integration
    - Push notifications
    - EAS Build + Update workflow

### Appendici
- **[A. Troubleshooting EAS](#appendice-a-troubleshooting-eas)**
- **[B. EAS CLI Reference](#appendice-b-eas-cli-reference)**
- **[C. Migration Guide](#appendice-c-migration-guide)**

---

## 🎓 Prerequisiti

**Conoscenze richieste:**
- ✅ Modulo 01bis - Expo Fondamenti completato
- ✅ Esperienza React Native base (Capitoli 01-05)
- ✅ Familiarità con Git e GitHub
- ✅ Conoscenze base CLI e terminal

**Account necessari:**
- Expo account (gratuito)
- Apple Developer account ($99/anno) - per iOS production
- Google Play Console account ($25 one-time) - per Android production
- GitHub account (per CI/CD)

---

## ⏱️ Durata Stimata

**Totale: 6-8 ore** suddivise in:
- Parte 1-2 (EAS Build + Update): 2-3 ore
- Parte 3 (Config Plugins): 1-2 ore
- Parte 4 (Production): 2-3 ore
- Parte 5 (Progetto): 3-4 ore (pratica autonoma)

---

## 📖 Come Usare Questo Modulo

1. **Lettura sequenziale**: Segui l'ordine delle sezioni
2. **Pratica hands-on**: Ogni sezione ha esempi eseguibili
3. **Progetto finale**: Applica tutto nel progetto completo
4. **Reference**: Usa appendici come documentazione rapida

---

## 🚀 Quick Start

Se hai già esperienza e vuoi vedere subito un esempio completo:

```bash
# 1. Setup EAS
$ npm install -g eas-cli
$ eas login
$ eas build:configure

# 2. First build
$ eas build --platform all --profile development

# 3. Setup OTA updates
$ eas update:configure
$ eas update --branch production

# Continua con il modulo per dettagli!
```

---

## 📚 Risorse Ufficiali

- 📖 [EAS Documentation](https://docs.expo.dev/eas/)
- 🏗️ [EAS Build Reference](https://docs.expo.dev/build/introduction/)
- 🔄 [EAS Update Reference](https://docs.expo.dev/eas-update/introduction/)
- 🔌 [Config Plugins](https://docs.expo.dev/guides/config-plugins/)
- 💬 [Expo Discord](https://chat.expo.dev) - Community support

---

[Inizia il Modulo: 01. Setup EAS Account e Progetto →](#contenuto-completo)

---

# Contenuto Completo

[Il contenuto completo segue nelle sezioni successive...]
