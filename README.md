
# React Native by Example - Guida Completa alla Creazione di App Mobili Cross-Platform

## Introduzione

Benvenuti in **React Native by Example**, una guida completa e pratica per imparare a sviluppare applicazioni mobili cross-platform di qualità professionale. Questo corso è progettato per portarti dalle basi di React Native fino alla pubblicazione di app complete su App Store e Google Play.

React Native è uno dei framework più popolari per lo sviluppo mobile, utilizzato da aziende come Facebook, Instagram, Airbnb, Tesla e molte altre. La sua capacità di condividere il codice tra iOS e Android, mantenendo performance native e accesso alle funzionalità del dispositivo, lo rende una scelta eccellente per sviluppatori e aziende.

Il corso segue l'approccio **"learning by example"**: ogni concetto è accompagnato da esempi di codice completi e funzionanti, esercizi pratici graduati per difficoltà, e progetti reali che consolidano le competenze acquisite. Che tu sia uno sviluppatore web che vuole espandersi nel mobile, o un programmatore che vuole imparare React Native da zero, troverai un percorso strutturato e progressivo.

## Indice del Corso

### Parte I: Fondamenti di React Native

1. **[Introduzione a React Native](01%20-%20Fondamenti%20di%20React%20Native/01_Introduzione_React_Native.md)**
   - Storia ed evoluzione di React Native
   - Architettura e funzionamento del framework
   - React Native vs altri framework (Flutter, Ionic, Xamarin)
   - Il bridge JavaScript-Native e la New Architecture
   - Quando usare React Native

2. **[Configurazione dell'Ambiente di Sviluppo](01%20-%20Fondamenti%20di%20React%20Native/02_Ambiente_Sviluppo.md)**
   - Installazione di Node.js e package manager (npm/yarn)
   - React Native CLI vs Expo: differenze e scelta
   - Setup ambiente Android (JDK, Android Studio, SDK)
   - Setup ambiente iOS (Xcode, CocoaPods) - Solo macOS
   - Creazione del primo progetto
   - Emulatori, simulatori e dispositivi fisici

3. **[JavaScript e TypeScript per React Native](01%20-%20Fondamenti%20di%20React%20Native/03_JavaScript_TypeScript.md)**
   - JavaScript moderno (ES6+) per React Native
   - Arrow functions, destructuring, spread operator
   - Promises e async/await
   - Introduzione a TypeScript
   - Configurazione TypeScript in React Native
   - Tipi, interfacce e generics

4. **[Fondamenti di React](01%20-%20Fondamenti%20di%20React%20Native/04_Fondamenti_React.md)**
   - Componenti funzionali e JSX
   - Props e comunicazione tra componenti
   - State locale con useState
   - Effetti collaterali con useEffect
   - Context API per lo stato globale
   - Custom Hooks

### Parte II: Sviluppo dell'Interfaccia Utente

5. **[Componenti Core di React Native](02%20-%20Sviluppo%20dell'Interfaccia%20Utente/05_Componenti_Core.md)**
   - View, Text, Image
   - ScrollView e FlatList per liste performanti
   - TextInput e gestione della tastiera
   - Button, TouchableOpacity, Pressable
   - Modal, Alert, Switch
   - ActivityIndicator e StatusBar

6. **[Styling e Layout](02%20-%20Sviluppo%20dell'Interfaccia%20Utente/06_Styling_Layout.md)**
   - Flexbox in React Native
   - StyleSheet API e stili inline
   - Responsive design e dimensioni dello schermo
   - Platform-specific code (iOS vs Android)
   - Temi e variabili di stile
   - Librerie di styling (Styled Components, Tailwind)

7. **[Navigazione tra Schermate](02%20-%20Sviluppo%20dell'Interfaccia%20Utente/07_Navigazione.md)**
   - React Navigation: installazione e setup
   - Stack Navigator per navigazione gerarchica
   - Tab Navigator per menu a schede
   - Drawer Navigator per menu laterale
   - Passaggio di parametri tra schermate
   - Deep linking e navigation state

8. **[Liste e Performance](02%20-%20Sviluppo%20dell'Interfaccia%20Utente/08_Liste_Performance.md)**
   - FlatList: rendering ottimizzato di liste
   - SectionList per liste con sezioni
   - VirtualizedList per casi complessi
   - Ottimizzazione con React.memo e useMemo
   - Pull-to-refresh e infinite scrolling
   - Best practices per liste performanti

9. **[Animazioni e Gestures](02%20-%20Sviluppo%20dell'Interfaccia%20Utente/09_Animazioni_Gestures.md)**
   - Animated API di React Native
   - LayoutAnimation per animazioni semplici
   - React Native Reanimated 2/3
   - Gesture Handler per gesture complesse
   - Animazioni di transizione tra schermate
   - Esempi pratici: swipe, drag, parallax

10. **[Formulari e Validazione](02%20-%20Sviluppo%20dell'Interfaccia%20Utente/10_Form_Validazione.md)**
    - Gestione di form complessi
    - Validazione input lato client
    - React Hook Form per React Native
    - Librerie di validazione (Yup, Zod)
    - Gestione errori e feedback utente
    - Form multi-step

### Parte III: Dati e Networking

11. **[Networking e API REST](03%20-%20Dati%20e%20Networking/11_Networking_API.md)**
    - Fetch API e Axios
    - Chiamate HTTP: GET, POST, PUT, DELETE
    - Gestione degli errori di rete
    - Timeout e retry logic
    - Headers, autenticazione e token
    - Best practices per le chiamate API

12. **[State Management Avanzato](03%20-%20Dati%20e%20Networking/12_State_Management.md)**
    - Redux Toolkit per applicazioni complesse
    - Redux con TypeScript
    - Middleware: thunks e saga
    - Zustand: alternativa leggera a Redux
    - Recoil per state management
    - Scelta del giusto strumento

13. **[Persistenza dei Dati](03%20-%20Dati%20e%20Networking/13_Persistenza_Dati.md)**
    - AsyncStorage per dati semplici
    - SQLite per database relazionali
    - Realm per database NoSQL
    - WatermelonDB per sync e performance
    - Secure Storage per dati sensibili
    - Cache e offline-first strategies

14. **[GraphQL e Client Moderni](03%20-%20Dati%20e%20Networking/14_GraphQL_Apollo.md)**
    - Introduzione a GraphQL
    - Apollo Client in React Native
    - Query, mutations e subscriptions
    - Cache e ottimizzazione
    - Gestione errori e loading states
    - GraphQL Code Generator

### Parte IV: Funzionalità Native

15. **[Accesso alla Fotocamera e Galleria](04%20-%20Funzionalit%C3%A0%20Native/15_Fotocamera_Galleria.md)**
    - React Native Camera
    - Expo Camera API
    - Image Picker per selezione foto
    - Crop e manipolazione immagini
    - Permessi runtime su iOS e Android
    - Upload di immagini

16. **[Geolocalizzazione e Mappe](04%20-%20Funzionalit%C3%A0%20Native/16_Geolocalizzazione_Mappe.md)**
    - Geolocation API
    - React Native Maps
    - Marker, regioni e clustering
    - Direzioni e routing
    - Background location tracking
    - Gestione permessi di localizzazione

17. **[Notifiche Push](04%20-%20Funzionalit%C3%A0%20Native/17_Notifiche_Push.md)**
    - Notifiche locali
    - Firebase Cloud Messaging (FCM)
    - Apple Push Notification Service (APNS)
    - Gestione token e registrazione
    - Deep linking da notifiche
    - Scheduling e notifiche ricorrenti

18. **[Multimedia e Sensori](04%20-%20Funzionalit%C3%A0%20Native/18_Moduli_Nativi.md)**
    - Audio playback e recording
    - Video player
    - Accelerometro e giroscopio
    - Biometria (Face ID, Touch ID)
    - Condivisione contenuti (Share API)
    - In-App Purchases

### Parte V: Produzione e Qualità

19. **[Performance e Ottimizzazione](05%20-%20Produzione%20e%20Qualit%C3%A0/20_Performance.md)**
    - Profiling e debugging performance
    - Hermes JavaScript Engine
    - Bundle size optimization
    - Lazy loading e code splitting
    - Memory leaks e come evitarli
    - Native performance monitoring

20. **[Testing](05%20-%20Produzione%20e%20Qualit%C3%A0/19_Testing.md)**
    - Jest per unit testing
    - Testing componenti con React Native Testing Library
    - Mock di API e moduli nativi
    - Integration testing
    - End-to-end testing con Detox
    - Coverage e CI/CD integration

21. **[Sicurezza](05%20-%20Produzione%20e%20Qualit%C3%A0/21_Deployment.md)**
    - Secure storage di dati sensibili
    - SSL Pinning
    - Code obfuscation
    - Gestione sicura di API keys
    - Prevenzione di reverse engineering
    - Best practices di sicurezza

22. **[Build, Release e Deployment](05%20-%20Produzione%20e%20Qualit%C3%A0/22_Monitoring.md)**
    - Configurazione per production
    - Build Android (APK e AAB)
    - Build iOS (IPA e archiving)
    - Code signing e certificati
    - Pubblicazione su Google Play Store
    - Pubblicazione su Apple App Store
    - Over-the-air updates con CodePush

### Parte VI: Argomenti Avanzati e Progetti

23. **[Moduli Nativi e Bridging](06%20-%20Argomenti%20Avanzati%20e%20Progetti/23_Pattern_Avanzati.md)**
    - Quando servono moduli nativi
    - Creare moduli nativi per Android (Kotlin/Java)
    - Creare moduli nativi per iOS (Swift/Objective-C)
    - Turbo Modules e JSI
    - Fabric Renderer
    - Librerie third-party native

24. **[New Architecture di React Native](06%20-%20Argomenti%20Avanzati%20e%20Progetti/24_New_Architecture.md)**
    - Introduzione alla New Architecture
    - JSI (JavaScript Interface) e Turbo Modules
    - Fabric Renderer e miglioramenti UI
    - Bridgeless Mode
    - Migrazione da Old a New Architecture
    - Compatibilità librerie third-party

25. **[Progetti Completi](06%20-%20Argomenti%20Avanzati%20e%20Progetti/25_Progetto_Completo.md)**
    - Progetto 1: App E-commerce con carrello e pagamenti
    - Progetto 2: Social Network con chat real-time
    - Progetto 3: App Fitness con tracking e grafici
    - Best practices e pattern architetturali
    - Deployment in produzione
    - Manutenzione e aggiornamenti

### Parte VII: Expo
1. **[Sviluppo Mobile con Expo - Fondamenti](07%20-%2001.Sviluppo%20Mobile%20con%20Expo/README.md)**
   - 📱 Panorama dello sviluppo mobile moderno (Native, Hybrid, Cross-platform)
   - 🎯 Introduzione a Expo: architettura, SDK ed ecosystem
   - ⚙️ Setup completo ambiente Expo (Node.js, Expo CLI, VS Code, Expo Go)
   - 🚀 Primo progetto Expo e debugging con Metro
   - 🔄 Expo Go vs Development Builds: quando usare cosa
   - 🛠️ Managed vs Bare Workflow e Config Plugins
   - 📸 Expo SDK APIs: Camera, Location, Notifications (esempi completi)
   - 💪 Esercizi pratici e quiz finale

2. **[Expo EAS Advanced](07%20-%2002.Expo%20EAS%20Advanced/README.md)**
   - 🏗️ EAS Build: build iOS senza Mac e Android automatizzati
   - 🔄 EAS Update: deployment OTA, branches, rollback
   - 🔌 Config Plugins custom: URL schemes e permessi
   - 🔐 Environment variables e EAS Secrets
   - 🤖 CI/CD integration con GitHub Actions
   - 📊 Monitoring con Sentry
   - 🛒 Progetto E-commerce completo (Zustand, Stripe, Push Notifications)
   - 📚 Troubleshooting, CLI reference e migration guides


## Appendici

- **[Appendice A: Risorse e Tool Utili](appendici/A_Risorse_Tool.md)**
- **[Appendice B: Troubleshooting Comune](appendici/B_Troubleshooting.md)**
- **[Appendice C: Migrazione da Versioni Precedenti](appendici/C_Migrazione.md)**
- **[Appendice D: Glossario](appendici/D_Glossario.md)**

## Come Utilizzare Questo Corso

Ogni capitolo è organizzato nel seguente modo:

- **README.md**: Panoramica del capitolo con obiettivi, indice degli argomenti ed esercizi pratici
- **teoria/**: Cartella contenente file markdown con spiegazioni teoriche dettagliate
- **esempi/**: Codice di esempio completo e commentato
- **esercizi/**: Esercizi pratici con difficoltà graduata (🟢 Base, 🟡 Intermedio, 🔴 Avanzato)

### Percorso Consigliato

1. **Per principianti**: Seguire tutti i capitoli in ordine, dall'1 al 25 (con 01bis e 01ter per chi sceglie Expo)
2. **Per sviluppatori React esperti**: Puoi iniziare dal capitolo 5, avendo familiarità con i capitoli 3-4
3. **Per sviluppatori mobile nativi**: Focus su capitoli 1-4, 7, 12-14, 23-24
4. **Per approfondimenti specifici**: Ogni capitolo è autocontenuto e può essere consultato indipendentemente

## Prerequisiti

- Conoscenza base di JavaScript (variabili, funzioni, oggetti, array)
- Familiarità con HTML/CSS è utile ma non obbligatoria
- Nozioni di programmazione ad oggetti
- Computer con macOS (per sviluppo iOS) o Windows/Linux (solo Android)

## Repository e Risorse

- **Codice degli esempi**: Ogni capitolo include codice funzionante e testato
- **Video tutorial**: Link a video dimostrativi per concetti complessi
- **Community**: Link al forum per domande e discussioni
- **Aggiornamenti**: Questo corso viene mantenuto aggiornato con le ultime versioni di React Native

## Licenza

Questo materiale didattico è rilasciato sotto licenza Creative Commons BY-NC-SA 4.0.

---

**Buon apprendimento e buon coding!** 🚀📱

[Inizia dal Capitolo 1: Introduzione a React Native →](01%20-%20Fondamenti%20di%20React%20Native/01_Introduzione_React_Native.md)