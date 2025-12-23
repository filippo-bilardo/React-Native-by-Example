# Capitolo 1: Introduzione a React Native

[← Torna all'Indice](../README.md) | [Prossimo: Configurazione Ambiente →](02_Ambiente_Sviluppo.md)

---

## Descrizione

Benvenuti nel primo capitolo del corso React Native! In questa lezione esploreremo le fondamenta di React Native: cosa è, come funziona, perché dovresti usarlo e quando è la scelta giusta per il tuo progetto. Capiremo l'architettura del framework, la sua evoluzione e come si posiziona rispetto ad altre soluzioni per lo sviluppo mobile cross-platform.

React Native rappresenta una rivoluzione nel mondo dello sviluppo mobile, permettendo di creare app native per iOS e Android utilizzando JavaScript e React. Nato in Facebook nel 2015, è oggi uno dei framework più popolari al mondo, utilizzato da migliaia di aziende e milioni di sviluppatori.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Comprendere cos'è React Native e come funziona
- [ ] Spiegare l'architettura del framework e il concetto di bridge
- [ ] Confrontare React Native con altri framework cross-platform
- [ ] Valutare quando usare React Native per un progetto
- [ ] Conoscere le aziende e le app che utilizzano React Native
- [ ] Comprendere i vantaggi e i limiti del framework
- [ ] Capire la differenza tra la vecchia architettura e la New Architecture

## 📚 Argomenti Teorici

1. [Storia ed Evoluzione di React Native](#1-storia-ed-evoluzione)
2. [Cos'è React Native](#2-cosè-react-native)
3. [Architettura e Funzionamento](#3-architettura-e-funzionamento)
4. [Il Bridge JavaScript-Native](#4-il-bridge-javascript-native)
5. [La New Architecture](#5-la-new-architecture)
6. [React Native vs Altri Framework](#6-react-native-vs-altri-framework)
7. [Vantaggi di React Native](#7-vantaggi-di-react-native)
8. [Limiti e Svantaggi](#8-limiti-e-svantaggi)
9. [Quando Usare React Native](#9-quando-usare-react-native)
10. [Casi di Studio](#10-casi-di-studio)

---

## 1. Storia ed Evoluzione

### Le Origini (2013-2015)

React Native nasce da un **Hackathon interno a Facebook** nel 2013. Gli sviluppatori volevano portare i benefici di React (già utilizzato per il web) anche sulle piattaforme mobile. L'obiettivo era ambizioso: **"Learn Once, Write Anywhere"** - impara React una volta e scrivilo ovunque.

**Timeline storica:**

- **Agosto 2013**: Prima prototipo interno a Facebook
- **Gennaio 2015**: Facebook annuncia React Native alla conferenza React.js Conf
- **Marzo 2015**: React Native viene rilasciato come open source (solo iOS)
- **Settembre 2015**: Supporto Android
- **2016-2017**: Adozione massiva da parte della community
- **2018**: Rilascio della versione 0.50+ con miglioramenti significativi
- **2020-2021**: Annuncio della New Architecture
- **2022-2024**: Graduale adozione della New Architecture
- **2025**: React Native 0.75+ con New Architecture stabile

### Perché Facebook Ha Creato React Native?

Facebook aveva diverse app mobile (Facebook, Instagram, Messenger) e affrontava problemi comuni:

1. **Team separati** per iOS e Android (duplicazione del lavoro)
2. **Tempi di sviluppo lunghi** per implementare feature su entrambe le piattaforme
3. **Difficoltà nel mantenere** la coerenza tra le piattaforme
4. **Costi elevati** per sviluppatori specializzati

React Native è stata la risposta a questi problemi: **un codice, due piattaforme**.

---

## 2. Cos'è React Native

React Native è un **framework open-source** per lo sviluppo di applicazioni mobile native utilizzando **JavaScript** e **React**.

### Caratteristiche Principali

```
React Native = React + Native APIs
```

**Non è:**
- ❌ Una WebView (come Cordova/PhoneGap)
- ❌ Un'app ibrida con HTML/CSS renderizzato
- ❌ Una semplice wrapper di un'app web

**È:**
- ✅ Un framework che renderizza componenti nativi reali
- ✅ Una soluzione per scrivere logica in JavaScript e UI nativa
- ✅ Un ponte tra JavaScript e codice nativo (Java/Kotlin per Android, Objective-C/Swift per iOS)

### Concetto Fondamentale

```javascript
// Scrivi questo in JavaScript/React:
<View>
  <Text>Hello World!</Text>
</View>

// React Native lo trasforma in:
// iOS: UIView + UILabel
// Android: ViewGroup + TextView
```

React Native **traduce i tuoi componenti React in componenti nativi** della piattaforma.

---

## 3. Architettura e Funzionamento

### Architettura a Tre Livelli

React Native è composto da tre layer principali:

```
┌─────────────────────────────────────┐
│   JavaScript Layer (Your Code)      │
│   - React Components                │
│   - Business Logic                  │
│   - State Management                │
└──────────────┬──────────────────────┘
               │
               │ Bridge (Async)
               │
┌──────────────▼──────────────────────┐
│   Native Modules Layer              │
│   - Platform APIs                   │
│   - Custom Native Code              │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   Native UI Layer                   │
│   - iOS: UIKit                      │
│   - Android: Android Views          │
└─────────────────────────────────────┘
```

### Come Funziona

1. **Tu scrivi codice JavaScript/React**
   ```javascript
   const App = () => {
     return <Text>Hello React Native!</Text>;
   };
   ```

2. **JavaScript viene eseguito in un JavaScript Engine**
   - iOS: JavaScriptCore (JSC)
   - Android: Hermes o JSC
   - Metro Bundler raggruppa il codice

3. **Il Bridge comunica tra JavaScript e Native**
   - Messaggi serializzati in JSON
   - Comunicazione asincrona
   - Batch di operazioni per efficienza

4. **Native UI viene renderizzato**
   - Componenti nativi reali
   - Performance native
   - Look & Feel della piattaforma

---

## 4. Il Bridge JavaScript-Native

### Cos'è il Bridge?

Il **Bridge** è il canale di comunicazione tra il mondo JavaScript e il mondo nativo. È il cuore dell'architettura React Native tradizionale.

```
JavaScript Thread  ←─── Bridge ───→  Native Thread
     (JS)                                (iOS/Android)
```

### Come Funziona

1. **Comunicazione Asincrona**
   ```javascript
   // JavaScript vuole accedere alla camera
   Camera.takePicture() // Invia messaggio al bridge
     .then(photo => {    // Riceve risposta dal bridge
       // Usa la foto
     });
   ```

2. **Serializzazione dei Messaggi**
   ```
   JS: { type: 'takePicture', params: {...} }
     ↓ Serializzato in JSON
   Bridge: "{"type":"takePicture",...}"
     ↓ Deserializzato
   Native: Esegue takePicture()
     ↓ Risultato serializzato
   Bridge: "{"result":"photo_uri"}"
     ↓ Deserializzato
   JS: Riceve il risultato
   ```

3. **Batching per Performance**
   - Non ogni operazione passa immediatamente
   - Le operazioni vengono raggruppate
   - Inviate in batch ogni ~16ms (60 FPS)

### Limiti del Bridge

- **Overhead di serializzazione/deserializzazione**
- **Collo di bottiglia** per operazioni intensive
- **Comunicazione asincrona** può causare ritardi
- **Non ottimale** per animazioni complesse o scroll pesanti

Questi limiti hanno portato allo sviluppo della **New Architecture**.

---

## 5. La New Architecture

### Perché una Nuova Architettura?

La vecchia architettura con il Bridge, pur funzionando bene per la maggior parte dei casi, aveva limitazioni:

- Performance non ottimali per UI complesse
- Impossibilità di chiamate sincrone
- Startup lento dell'app
- Difficoltà nell'integrare codice nativo moderno

### Componenti della New Architecture

La New Architecture introduce due concetti fondamentali:

#### 1. **Turbo Modules** (Sostituto dei Native Modules)

```
JavaScript ←─── JSI ───→ C++ ←───→ Native
           (Direct Call)
```

**Vantaggi:**
- ✅ Chiamate sincrone quando necessario
- ✅ Niente serializzazione JSON
- ✅ Lazy loading dei moduli
- ✅ Performance significativamente migliorate

#### 2. **Fabric** (Nuovo Renderer UI)

**Vecchio sistema:**
```
JS → Bridge → Shadow Tree → Native UI
```

**Nuovo sistema (Fabric):**
```
JS → JSI → C++ Renderer → Native UI
        (Direct, sincronizzato)
```

**Vantaggi:**
- ✅ Rendering sincrono e più veloce
- ✅ Migliore gestione delle priorità
- ✅ Supporto per Concurrent React
- ✅ Migliore interoperabilità con codice nativo

### JavaScript Interface (JSI)

JSI è il cuore della New Architecture:

```cpp
// JSI permette di chiamare direttamente funzioni native da JS
// e viceversa, senza bridge!

// C++ può esporre funzioni a JavaScript
jsi::Function jsFunc = jsi::Function::createFromHostFunction(...);

// JavaScript può chiamare direttamente codice C++/Native
```

### Status e Adozione

- **React Native 0.68+**: New Architecture disponibile (opt-in)
- **React Native 0.70+**: Miglioramenti e stabilità
- **React Native 0.74+**: Raccomandato per nuovi progetti
- **React Native 0.76+**: Default per nuovi progetti

---

## 6. React Native vs Altri Framework

### Confronto tra Framework Cross-Platform

| Caratteristica | React Native | Flutter | Ionic | Xamarin/.NET MAUI |
|----------------|--------------|---------|-------|-------------------|
| **Linguaggio** | JavaScript/TS | Dart | HTML/CSS/JS | C# |
| **UI Rendering** | Componenti Nativi | Canvas (Skia) | WebView | Componenti Nativi |
| **Performance** | ⭐⭐⭐⭐ Native-like | ⭐⭐⭐⭐⭐ Eccellente | ⭐⭐⭐ Buona | ⭐⭐⭐⭐ Native-like |
| **Curva di Apprendimento** | Media (React) | Media (Dart) | Bassa (Web) | Media (C#) |
| **Community** | 🔥 Molto Grande | 🔥 Grande | Media | Media |
| **Hot Reload** | ✅ | ✅ | ❌ (Live Reload) | ✅ |
| **Dimensione App** | ~15-30 MB | ~15-20 MB | ~5-10 MB | ~20-40 MB |
| **Accesso Native** | Buono (Moduli) | Eccellente | Limitato | Eccellente |

### React Native vs Flutter

**React Native:**
- ✅ Usa JavaScript (più sviluppatori disponibili)
- ✅ Componenti nativi reali
- ✅ Ecosistema npm enorme
- ✅ Web con React Native Web
- ❌ Performance leggermente inferiore per animazioni complesse
- ❌ Aggiornamenti alle librerie native possono causare breaking changes

**Flutter:**
- ✅ Performance eccellenti (rendering su canvas)
- ✅ UI consistente su tutte le piattaforme
- ✅ Hot reload velocissimo
- ❌ Dart è meno diffuso di JavaScript
- ❌ Dimensione app più grande
- ❌ Non usa componenti nativi (look personalizzato)

### React Native vs Ionic

**React Native:**
- ✅ Componenti nativi → Look & Feel nativo
- ✅ Performance superiori
- ✅ Accesso diretto alle API native
- ❌ Più complesso da configurare

**Ionic:**
- ✅ Usa tecnologie web standard
- ✅ Più facile per sviluppatori web
- ✅ Dimensione app più piccola
- ❌ Performance inferiori (WebView)
- ❌ Non sembra nativa
- ❌ Limitazioni nell'accesso alle funzionalità native

### Quando Scegliere React Native?

**✅ Scegli React Native se:**
- Hai esperienza con React o JavaScript
- Vuoi riutilizzare codice tra iOS, Android e Web
- Hai bisogno di componenti con look nativo
- Vuoi accesso a migliaia di librerie npm
- Il team conosce già JavaScript
- Hai bisogno di rilasci rapidi e iterazioni veloci

**❌ Considera alternative se:**
- Hai bisogno di performance estreme (giochi 3D → Unity)
- Devi usare pesantemente API native molto specifiche (forse nativo puro)
- Il team conosce solo C# (→ .NET MAUI)
- Preferisci UI pixel-perfect identica su tutte le piattaforme (→ Flutter)

---

## 7. Vantaggi di React Native

### 1. **Cross-Platform con Codice Condiviso**

```
📱 iOS     }
📱 Android } → 70-90% codice condiviso
🌐 Web     }    (con React Native Web)
```

**Benefici:**
- Sviluppo più veloce
- Meno bug (logica condivisa)
- Team più piccoli
- Costi ridotti

### 2. **Componenti Nativi Reali**

```javascript
// Questo non è HTML, sono componenti nativi!
<ScrollView>  {/* UIScrollView su iOS, ScrollView su Android */}
  <Image />   {/* UIImageView su iOS, ImageView su Android */}
  <Text />    {/* UILabel su iOS, TextView su Android */}
</ScrollView>
```

**Risultato:**
- Look & Feel nativo
- Animazioni fluide
- Risposta al tocco immediata

### 3. **Hot Reload & Fast Refresh**

```javascript
// Modifica questo:
<Text>Hello World</Text>

// Salva... e vedi il cambiamento ISTANTANEAMENTE nell'app!
<Text>Hello React Native</Text>

// Senza perdere lo stato dell'app! 🚀
```

**Produttività aumentata del 2-3x** rispetto allo sviluppo nativo tradizionale.

### 4. **Ecosistema JavaScript**

Accesso a **oltre 2 milioni di pacchetti npm**:

```bash
npm install axios           # HTTP client
npm install react-query     # Data fetching
npm install zustand         # State management
npm install date-fns        # Date utilities
# ... e moltissimi altri!
```

### 5. **Community Enorme**

- **122k+ stelle** su GitHub
- Migliaia di librerie dedicate
- Tutorial, corsi, articoli
- Forum attivi (Stack Overflow, Reddit, Discord)
- Conferenze dedicate (React Native EU, Chain React)

### 6. **Supporto di Grandi Aziende**

Utilizzato da:
- **Meta** (Facebook, Instagram, Messenger)
- **Microsoft** (Office, Xbox, Teams mobile)
- **Shopify** (app mobile principale)
- **Tesla** (app mobile)
- **Discord** (app iOS e Android)
- **Walmart**, **Bloomberg**, **Uber Eats**, e centinaia di altre

### 7. **Over-The-Air Updates**

Con CodePush o Expo Updates:

```javascript
// Puoi aggiornare l'app senza passare per lo store!
// Bug fix, nuove features in JavaScript → update immediato
```

**Vantaggi:**
- Fix bug istantanei
- A/B testing semplificato
- No attesa review degli store

### 8. **Sviluppo Web Possibile**

Con React Native Web:

```javascript
// Lo stesso codice React Native può girare sul web!
import { View, Text } from 'react-native';

// Funziona su iOS, Android E browser! 🎉
```

---

## 8. Limiti e Svantaggi

### 1. **Dipendenze Native**

- Ogni libreria che usa codice nativo richiede configurazione
- Aggiornamenti di iOS/Android possono rompere dipendenze
- Alcune librerie non sono mantenute

### 2. **Debugging Complesso**

Devi debuggare tre ambienti:
- JavaScript (Chrome DevTools)
- iOS (Xcode)
- Android (Android Studio / Logcat)

### 3. **Dimensione dell'App**

App minima: **~20-30 MB** (vs ~5 MB per nativo puro)

### 4. **Performance per Casi Estremi**

Per app con:
- Animazioni 3D complesse
- Elaborazione video/audio intensiva
- Giochi ad alte performance

→ Il nativo puro o Unity potrebbero essere migliori

### 5. **Aggiornamenti Frequenti**

- Nuove versioni ogni ~1 mese
- Breaking changes occasionali
- Necessità di mantenere il progetto aggiornato

### 6. **Curva di Apprendimento**

Devi conoscere:
- JavaScript/TypeScript
- React
- Specifiche iOS (Xcode, CocoaPods, provisioning)
- Specifiche Android (Gradle, SDK, manifest)

---

## 9. Quando Usare React Native

### ✅ Casi d'Uso Ideali

1. **App Business/Produttività**
   - CRM, ERP mobile
   - Dashboard aziendali
   - App di gestione

2. **Social Network e Messaggistica**
   - Feed di contenuti
   - Chat e messaggistica
   - Condivisione sociale

3. **E-Commerce**
   - Cataloghi prodotti
   - Carrelli e checkout
   - Gestione ordini

4. **Content App**
   - News reader
   - Blog e magazine
   - Podcast player

5. **App per Eventi**
   - Conferenze
   - Concerti
   - Meetup

6. **App Finanziarie**
   - Banking app
   - Trading platforms
   - Expense trackers

### ❌ Quando Considerare Alternative

1. **Giochi 3D** → Unity, Unreal Engine
2. **App con pesante elaborazione video** → Swift/Kotlin nativo
3. **App con requisiti di performance estremi** → Nativo
4. **App che usano feature iOS/Android ultra-recenti** → Nativo (poi considera RN)

### 🤔 Valutazione

Chiediti:

1. **"L'app ha bisogno di performance native al 100%?"**
   - No → React Native ✅
   - Sì → Valuta nativo o Flutter

2. **"Il team conosce JavaScript/React?"**
   - Sì → React Native ✅
   - No → Training o considera altro

3. **"Serve deployment su iOS E Android?"**
   - Sì → React Native ✅
   - Solo una piattaforma → Valuta nativo

4. **"Budget e tempi sono limitati?"**
   - Sì → React Native ✅
   - Illimitati → Valuta tutte le opzioni

---

## 10. Casi di Studio

### Facebook

- **App**: Facebook, Instagram, Messenger
- **Dettagli**: Parti dell'app (non tutto) in React Native
- **Benefici**: Sviluppo più veloce, codice condiviso

### Microsoft

- **App**: Xbox, Office, Skype
- **Dettagli**: Gran parte dell'UI mobile in React Native
- **Benefici**: Team unificato, rilasci sincronizzati

### Shopify

- **App**: Shopify Mobile (principale app mobile)
- **Dettagli**: Riscritta completamente in React Native nel 2020
- **Risultati**:
  - 🚀 3x più veloce nello sviluppo
  - 👥 Team ridotti
  - 📦 Single codebase per iOS e Android

### Tesla

- **App**: Tesla Mobile App
- **Dettagli**: App per controllare le auto Tesla
- **Funzionalità**: Unlock auto, clima, monitoraggio

### Discord

- **App**: Discord mobile (iOS e Android)
- **Dettagli**: Migrazione da nativo a React Native
- **Risultati**: Velocità di sviluppo aumentata, sincronizzazione feature tra piattaforme

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Ricerca e Analisi

**Obiettivo**: Familiarizzare con l'ecosistema React Native

**Task**:
1. Visita il sito ufficiale di React Native (reactnative.dev)
2. Esplora la documentazione
3. Leggi i "Case Studies" nel blog ufficiale
4. Rispondi a queste domande:
   - Quali sono le ultime versioni di React Native?
   - Quali funzionalità sono state aggiunte nell'ultima major release?
   - Quante librerie trovi su npmjs.com cercando "react-native"?

### Esercizio 2: 🟡 Confronto Framework

**Obiettivo**: Capire quando scegliere React Native

**Task**:
Crea una tabella comparativa per questi scenari, indicando quale framework useresti e perché:

| Scenario | Framework Scelto | Motivazione |
|----------|------------------|-------------|
| App di e-commerce per startup | ? | ? |
| Gioco 2D casual | ? | ? |
| App bancaria enterprise | ? | ? |
| Social network per foto | ? | ? |
| App per fitness con sensori | ? | ? |

**Opzioni**: React Native, Flutter, Nativo (Swift/Kotlin), Ionic, Unity

### Esercizio 3: 🔴 Analisi di App Reali

**Obiettivo**: Riconoscere app React Native "in the wild"

**Task**:
1. Scarica 2-3 di queste app:
   - Facebook
   - Instagram
   - Discord
   - Bloomberg
   - Walmart

2. Analizza:
   - Fluidità delle animazioni
   - Tempo di risposta dell'UI
   - Look & feel (nativo vs custom)

3. Prova a identificare quali parti potrebbero essere React Native

4. Scrivi un report di 300 parole sulle tue osservazioni

### Esercizio 4: 🟢 Esplorare il Codice

**Obiettivo**: Vedere codice React Native reale

**Task**:
1. Visita GitHub e cerca "react-native example"
2. Apri 2-3 repository di app open source
3. Cerca il file `App.js` o `App.tsx`
4. Anche senza capire tutto, prova a identificare:
   - Componenti usati (`<View>`, `<Text>`, etc.)
   - Struttura del codice
   - Gestione dello state

**Repository suggeriti**:
- facebook/react-native (contiene esempi)
- expo/examples
- react-native-community/react-native-maps

---

## 📝 Quiz di Autovalutazione

1. **Cos'è React Native?**
   - [ ] Un framework per creare app web
   - [ ] Un framework per creare app native usando WebView
   - [x] Un framework per creare app native usando JavaScript e componenti nativi
   - [ ] Un linguaggio di programmazione

2. **Qual è il ruolo del Bridge in React Native?**
   - [ ] Collega l'app a internet
   - [x] Permette la comunicazione tra JavaScript e codice nativo
   - [ ] Compila JavaScript in codice nativo
   - [ ] Gestisce la navigazione tra schermate

3. **Quale di queste NON è una caratteristica della New Architecture?**
   - [ ] Turbo Modules
   - [ ] Fabric Renderer
   - [ ] JSI (JavaScript Interface)
   - [x] Faster Bridge

4. **Quando NON dovresti usare React Native?**
   - [ ] Per un'app di e-commerce
   - [x] Per un gioco 3D ad alte performance
   - [ ] Per un social network
   - [ ] Per un'app di news

5. **Qual è un vantaggio principale di React Native rispetto allo sviluppo nativo?**
   - [ ] App più piccole
   - [ ] Performance sempre migliori
   - [x] Condivisione del codice tra iOS e Android
   - [ ] Non serve conoscere JavaScript

**Risposte**: 1-c, 2-b, 3-d, 4-b, 5-c

---

## 🔗 Risorse Aggiuntive

### Documentazione Ufficiale
- [React Native Docs](https://reactnative.dev/)
- [React Native Blog](https://reactnative.dev/blog)
- [New Architecture](https://reactnative.dev/docs/the-new-architecture/landing-page)

### Video e Tutorial
- [React Native Crash Course](https://www.youtube.com/results?search_query=react+native+crash+course)
- [Meta Engineering Blog](https://engineering.fb.com/)

### Articoli Consigliati
- "Why React Native is the Future of Mobile Development"
- "React Native at Scale: Stories from Discord and Shopify"
- "Understanding the React Native Bridge"

### Community
- [React Native Community on GitHub](https://github.com/react-native-community)
- [r/reactnative on Reddit](https://reddit.com/r/reactnative)
- [Reactiflux Discord](https://www.reactiflux.com/)

---

## ✅ Checklist di Completamento

Prima di passare al prossimo capitolo, assicurati di:

- [ ] Aver compreso cos'è React Native e come funziona
- [ ] Conoscere la differenza tra Bridge e New Architecture
- [ ] Saper confrontare React Native con altri framework
- [ ] Aver identificato scenari adatti per React Native
- [ ] Aver esplorato la documentazione ufficiale
- [ ] Aver completato almeno 2 esercizi su 4
- [ ] Aver risposto correttamente ad almeno 4 domande del quiz su 5

---

## 📌 Punti Chiave da Ricordare

1. 🎯 **React Native usa componenti nativi reali**, non WebView
2. 🌉 **Il Bridge connette JavaScript e Native** (architettura vecchia)
3. ⚡ **La New Architecture (JSI + Turbo Modules + Fabric) migliora drasticamente le performance**
4. 🔄 **70-90% del codice è condivisibile** tra iOS e Android
5. 🚀 **Hot Reload accelera lo sviluppo** enormemente
6. 🏢 **Aziende come Meta, Microsoft, Shopify lo usano in produzione**
7. ✅ **Ottimo per la maggior parte delle app**, non per giochi 3D o app con performance estreme
8. 📚 **Ecosistema npm enorme** a disposizione

---

**Congratulazioni! 🎉** Hai completato il primo capitolo. Ora hai una solida comprensione di cos'è React Native, come funziona e quando usarlo. Nel prossimo capitolo configureremo l'ambiente di sviluppo e creeremo la nostra prima app!

[← Torna all'Indice](../README.md) | [Prossimo: Configurazione Ambiente →](02_Ambiente_Sviluppo.md)
