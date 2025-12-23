# Capitolo 5: Componenti Core di React Native

[← Precedente: Fondamenti di React](../01%20-%20Fondamenti%20di%20React%20Native/04_Fondamenti_React.md) | [Torna all'Indice](../README.md) | [Prossimo: Styling e Layout →](06_Styling_Layout.md)

---

## Descrizione

React Native offre una serie di componenti fondamentali che sono i building blocks di ogni applicazione mobile. In questo capitolo esploreremo i componenti core più importanti: View, Text, Image, ScrollView, TextInput, Button, e molti altri.

A differenza del web dove usi `<div>`, `<p>`, `<img>`, in React Native utilizzerai componenti specifici che si traducono in elementi nativi iOS e Android. Padroneggiare questi componenti è essenziale per costruire qualsiasi interfaccia utente.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Utilizzare View come container principale
- [ ] Renderizzare testo con Text e gestire la tipografia
- [ ] Caricare e visualizzare immagini locali e remote
- [ ] Implementare scroll con ScrollView
- [ ] Creare input forms con TextInput
- [ ] Gestire interazioni utente con Button e TouchableOpacity
- [ ] Usare Modal per overlay e dialoghi
- [ ] Implementare Switch, ActivityIndicator e altri componenti UI
- [ ] Gestire la StatusBar dell'app
- [ ] Comprendere le differenze tra iOS e Android

## 📚 Argomenti Teorici

1. [View - Il Container Fondamentale](#1-view)
2. [Text - Renderizzare Testo](#2-text)
3. [Image - Visualizzare Immagini](#3-image)
4. [ScrollView - Contenuto Scrollabile](#4-scrollview)
5. [TextInput - Input dall'Utente](#5-textinput)
6. [Button e Touchable Components](#6-button-touchable)
7. [Modal - Overlay e Dialoghi](#7-modal)
8. [Altri Componenti UI](#8-altri-componenti)
9. [StatusBar](#9-statusbar)
10. [Platform-Specific Components](#10-platform-specific)

---

## 1. View

### Cos'è View?

`View` è il componente container fondamentale in React Native, il building block di base per costruire l'interfaccia utente. È l'equivalente del `<div>` nel web HTML, ma ottimizzato per dispositivi mobili.

**Concetti chiave:**
- **Container universale**: View può contenere qualsiasi altro componente React Native
- **Layout Flexbox**: Di default usa Flexbox per il layout (diverso dal web che usa block)
- **Touch handling**: Gestisce eventi touch nativi del dispositivo
- **Nesting illimitato**: Puoi annidare View infinite volte per creare gerarchie complesse
- **Platform-specific**: Viene renderizzato come `UIView` su iOS e `android.view.ViewGroup` su Android

```typescript
import { View } from 'react-native';

const App = () => {
  return (
    <View>
      {/* View è il contenitore base per tutti gli altri componenti.
          Senza View, non puoi costruire layout complessi in React Native.
          Ogni elemento visibile deve stare dentro una View (o sue sottoclassi) */}
    </View>
  );
};
```

### Proprietà Principali

View supporta molteplici proprietà per gestire layout, interazione e accessibilità. Vediamo le più importanti:

```typescript
<View
  // STILE: View accetta oggetti style per definire aspetto e layout
  style={{
    flex: 1,              // Occupa tutto lo spazio disponibile nel parent
    backgroundColor: '#fff', // Colore di sfondo
    padding: 20           // Spazio interno su tutti i lati
  }}
  
  // TOUCH HANDLING: Sistema di gestione tocchi di React Native (Responder System)
  // onStartShouldSetResponder decide se questa View deve "catturare" il tocco
  onStartShouldSetResponder={() => true}  // true = cattura tutti i tocchi
  
  // onResponderGrant viene chiamato quando la View diventa il responder attivo
  onResponderGrant={() => console.log('Touch started')}
  
  // onResponderRelease viene chiamato quando l'utente rilascia il tocco
  onResponderRelease={() => console.log('Touch ended')}
  
  // ACCESSIBILITY: Fondamentale per rendere l'app usabile con screen readers
  accessible={true}  // Rende l'elemento accessibile agli screen readers
  accessibilityLabel="Main container"  // Descrizione per utenti non vedenti
  accessibilityRole="main"  // Definisce il ruolo semantico dell'elemento
>
  <Text>Content</Text>
</View>
```

**Note importanti:**
- Il Responder System di React Native è diverso dagli eventi DOM del web
- `onStartShouldSetResponder` ritorna `true` per catturare il tocco, `false` per ignorarlo
- Accessibility props sono OBBLIGATORI per app inclusive e conformi agli standard

### View Nidificate

La composizione è il pattern principale di React Native: costruisci UI complesse nidificando View semplici. Questo esempio mostra una Card con struttura header-body-footer:

```typescript
const Card = () => {
  return (
    // View principale: container esterno della card
    <View style={styles.card}>
      
      {/* Header: sezione superiore per titolo/icone */}
      <View style={styles.header}>
        <Text>Card Title</Text>
      </View>
      
      {/* Body: contenuto principale della card */}
      <View style={styles.body}>
        <Text>Card content goes here</Text>
      </View>
      
      {/* Footer: sezione inferiore per azioni/pulsanti */}
      <View style={styles.footer}>
        <Button title="Action" onPress={() => {}} />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  card: {
    // borderRadius crea angoli arrotondati (8 pixel di raggio)
    borderRadius: 8,
    backgroundColor: 'white',
    
    // Shadow su iOS: usa shadowColor, shadowOpacity, shadowOffset, shadowRadius
    shadowColor: '#000',
    shadowOpacity: 0.1,
    
    // Shadow su Android: usa SOLO elevation (valori da 1 a 24)
    elevation: 3,  // 3 = ombra leggera, 24 = ombra molto pronunciata
  },
  header: { 
    padding: 15,
    // borderBottomWidth crea una linea divisoria tra header e body
    borderBottomWidth: 1, 
    borderColor: '#eee'  // Grigio chiaro per la linea
  },
  body: { padding: 15 },
  footer: { 
    padding: 15, 
    borderTopWidth: 1,  // Linea divisoria sopra il footer
    borderColor: '#eee' 
  },
});
```

**Pattern di composizione:**
1. View esterna = container con stili di card (ombra, bordi)
2. View interne = sezioni logiche con stili specifici
3. Separatori visivi con `borderWidth` e `borderColor`
4. Padding per spazi interni, margin per spazi esterni

### SafeAreaView

**Problema:** Dispositivi moderni (iPhone X+, Android con notch) hanno aree "non sicure" dove il contenuto può essere coperto da:
- Notch (tacca superiore)
- Status bar
- Home indicator (barra inferiore iOS)
- Rounded corners

**Soluzione:** SafeAreaView applica automaticamente padding per evitare queste aree.

```typescript
import { SafeAreaView } from 'react-native';

const App = () => {
  return (
    // SafeAreaView garantisce che il contenuto sia visibile
    // aggiungendo padding automatico in base al dispositivo
    <SafeAreaView style={{ flex: 1, backgroundColor: '#fff' }}>
      {/* Questo testo NON sarà mai coperto dal notch o dalla status bar.
          SafeAreaView calcola automaticamente il padding necessario. */}
      <Text>Questo contenuto evita il notch</Text>
    </SafeAreaView>
  );
};
```

**⚠️ Importante:**
- `SafeAreaView` di React Native funziona SOLO su iOS
- Per Android, usa `react-native-safe-area-context` (libreria community)
- Metti `SafeAreaView` come componente root dell'app
- **Non usare** `SafeAreaView` in modal o nested views (causa layout inaspettati)

**Best practice:**
```typescript
import { SafeAreaProvider, SafeAreaView } from 'react-native-safe-area-context';

// Wrap root con Provider
const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={{ flex: 1 }}>
      {/* Content */}
    </SafeAreaView>
  </SafeAreaProvider>
);
```

---

## 2. Text

### Renderizzare Testo

Il componente `Text` è l'UNICO modo per mostrare testo in React Native. A differenza del web, non puoi scrivere testo direttamente dentro una View.

**Differenze chiave con il web:**
- **Obbligatorio**: Tutto il testo DEVE essere in `<Text>` (nel web puoi scrivere testo libero)
- **No ereditarietà CSS**: Gli stili NON vengono ereditati (tranne tra Text nidificati)
- **Font personalizzati**: Devono essere linkati manualmente al progetto
- **Touch support**: Text supporta `onPress` nativamente (no bisogno di button/link)

```typescript
import { Text } from 'react-native';

{/* Testo base: usa font di sistema e colore di default */}
<Text>Hello World!</Text>

{/* Testo stilizzato: applica stili inline */}
<Text style={{ fontSize: 20, color: 'blue' }}>Styled Text</Text>

{/* ❌ ERRORE: Questo NON funziona in React Native */}
<View>
  Hello World!  {/* Genera errore: Text strings must be rendered within <Text> */}
</View>
```

### Proprietà Text

Text supporta molte proprietà per controllo preciso su rendering, interazione e accessibilità:

```typescript
<Text
  // STYLING: Stili inline (meglio usare StyleSheet per performance)
  style={{ fontSize: 16, fontWeight: 'bold', color: '#333' }}
  
  // TRONCAMENTO TESTO: Essenziale per UI responsive
  numberOfLines={2}  // Limita a MASSIMO 2 righe, resto viene troncato
  ellipsizeMode="tail"  // Dove mettere i "..." quando il testo è troppo lungo:
  // 'head' → "...fine del testo"
  // 'middle' → "inizio...fine"
  // 'tail' → "inizio del testo..." (DEFAULT)
  // 'clip' → tronca senza "..." (raramente usato)
  
  // INTERAZIONE: Text può essere touchable senza Touchable wrapper!
  onPress={() => console.log('Text pressed')}  // Singolo tap
  onLongPress={() => console.log('Long press')}  // Pressione lunga
  // Utile per: link, "leggi di più", azioni inline
  
  // SELEZIONE: Abilita copia/incolla nativo
  selectable={true}  
  // iOS: long press → menu con "Copy"
  // Android: long press → selection handles
  // ⚠️ Di default è false per performance
  
  // ACCESSIBILITÀ: Screen reader support
  accessible={true}  // Rende l'elemento accessibile
  accessibilityRole="header"  // Ruolo semantico per screen readers
  // Altri ruoli: 'link', 'button', 'text', 'none'
>
  This is some text that can be pressed and selected
</Text>
```

**💡 Caso d'uso comune: Testo troncato con "Leggi di più"**
```typescript
const [expanded, setExpanded] = useState(false);

<Text numberOfLines={expanded ? undefined : 3}>
  {longText}
  {!expanded && (
    <Text style={{ color: 'blue' }} onPress={() => setExpanded(true)}>
      {' '}Leggi di più
    </Text>
  )}
</Text>
```

### Text Nidificati

**Feature UNICA di React Native**: Text nidificati ereditano stili dal parent (unico caso di ereditarietà stili in RN!)

```typescript
const Article = () => {
  return (
    // Parent Text: definisce stili di base
    <Text style={{ fontSize: 16, color: 'black' }}>
      {/* Testo normale: eredita fontSize e color dal parent */}
      This is 
      
      {/* Nested Text: SOVRASCRIVE fontWeight, EREDITA fontSize e color */}
      <Text style={{ fontWeight: 'bold' }}>bold</Text>
      
      {/* Lo spazio {' '} è necessario per evitare che "bold" e "and" si attacchino */}
      {' '}and this is{' '}
      
      {/* Nested Text: SOVRASCRIVE fontStyle e color, EREDITA fontSize */}
      <Text style={{ fontStyle: 'italic', color: 'blue' }}>blue italic</Text>
    </Text>
  );
};
```

**Perché è utile:**
- Stili inline senza perdere contesto del paragrafo
- Screen readers leggono tutto come un unico blocco
- Selezione testo funziona attraverso i nested Text

**⚠️ Limitazioni:**
- Funziona SOLO con Text dentro Text (non View dentro Text)
- Layout proprietà (margin, padding) NON vengono ereditate
- Solo stili di testo vengono ereditati (color, fontSize, fontWeight, etc.)

**Pattern comune: Rich Text**
```typescript
<Text>
  <Text style={styles.username}>@john</Text>
  {' '}
  <Text style={styles.normal}>replied to</Text>
  {' '}
  <Text style={styles.username}>@jane</Text>
</Text>
```

### Custom Fonts

**Processo completo per aggiungere font personalizzati:**

```typescript
/**
 * STEP 1: Aggiungi file font al progetto
 * 
 * Crea cartella: assets/fonts/
 * Aggiungi files:
 * - Roboto-Regular.ttf
 * - Roboto-Bold.ttf
 * - Roboto-Italic.ttf
 * 
 * ⚠️ Nomi file IMPORTANTI: iOS li usa direttamente
 */

/**
 * STEP 2: Configura react-native.config.js (root del progetto)
 */
// react-native.config.js
module.exports = {
  project: {
    ios: {},
    android: {},
  },
  assets: ['./assets/fonts/'],  // Path alla cartella font
};

/**
 * STEP 3: Link fonts ai progetti nativi
 * 
 * Comando: npx react-native-asset
 * oppure: npx react-native link
 * 
 * Questo copia i font in:
 * - iOS: Info.plist + Xcode project
 * - Android: android/app/src/main/assets/fonts/
 */

/**
 * STEP 4: Usa nel codice
 * 
 * fontFamily deve matchare il NOME del font, non il nome del file!
 * Per trovare il nome: apri .ttf in Font Book (Mac) o proprietà file (Windows)
 */
<Text style={{ fontFamily: 'Roboto-Bold', fontSize: 20 }}>
  Custom Font
</Text>

// Per bold/italic con font custom:
<Text style={{ 
  fontFamily: 'Roboto',  // Font base
  fontWeight: 'bold'      // NON funziona con custom fonts!
}}>
  ❌ Questo NON renderizza Roboto-Bold
</Text>

// ✅ Soluzione: specifica esplicitamente la variante
<Text style={{ fontFamily: 'Roboto-Bold' }}>
  ✅ Questo usa Roboto-Bold.ttf
</Text>
```

**💡 Best Practice: Font Helper**
```typescript
// utils/fonts.ts
export const fonts = {
  regular: 'Roboto-Regular',
  bold: 'Roboto-Bold',
  italic: 'Roboto-Italic',
};

// Usage
<Text style={{ fontFamily: fonts.bold }}>
  Bold Text
</Text>
```

---

## 3. Image

**Differenze fondamentali con img HTML:**
- **Dimensioni obbligatorie**: width e height DEVONO essere specificati (no size naturale)
- **No auto-resize**: Image non si ridimensiona automaticamente in base al contenuto
- **Due tipi di source**: locale (require) vs remoto (uri)
- **Bundling**: immagini locali sono incluse nel bundle app (aumentano dimensione)

### Immagini Locali

Immagini locali usano `require()` e vengono **bundle-ate** nell'app durante build:

```typescript
import { Image } from 'react-native';

// ✅ Immagine locale: bundled nell'app
<Image
  source={require('./assets/logo.png')}  // require è statico (analizzato a compile-time)
  style={{ width: 100, height: 100 }}     // Dimensioni OBBLIGATORIE
/>

/**
 * ❌ ERRORE COMUNE: require dinamico NON funziona!
 * 
 * const imageName = 'logo.png';
 * <Image source={require(`./assets/${imageName}`)} />  // ❌ ERRORE
 * 
 * Motivo: Metro bundler (JS bundler di RN) analizza require() a compile-time
 * Non può risolvere stringhe dinamiche
 */

/**
 * ✅ SOLUZIONE: Mapping statico
 */
const images = {
  logo: require('./assets/logo.png'),
  profile: require('./assets/profile.png'),
  banner: require('./assets/banner.png'),
};

// Uso dinamico:
<Image source={images[imageName]} style={{ width: 100, height: 100 }} />
```

**Vantaggi immagini locali:**
- ✅ Disponibili offline (bundled nell'app)
- ✅ Caricamento istantaneo (no network)
- ✅ Versionate con il codice
- ❌ Aumentano dimensione bundle
- ❌ Non aggiornabili senza update app

### Immagini Remote (URI)

Immagini remote vengono scaricate dal network a runtime:

```typescript
// ✅ Immagine da URL: scaricata on-demand
<Image
  source={{ uri: 'https://example.com/image.jpg' }}  // uri = URL remoto
  style={{ width: 200, height: 200 }}                // Dimensioni ancora OBBLIGATORIE
/>
// ⚠️ Problemi comuni:
// 1. Dimensioni sbagliate → immagine distorta
// 2. URL HTTP (non HTTPS) → bloccato su iOS per security (serve configurazione)
// 3. CORS issues su web version (React Native Web)

/**
 * Headers personalizzati: per API con autenticazione
 */
<Image
  source={{
    uri: 'https://api.example.com/image',
    headers: {
      Authorization: 'Bearer token',  // Token JWT per API protette
      'Custom-Header': 'value'         // Altri headers custom
    },
    // Opzioni aggiuntive:
    method: 'POST',  // Default: GET
    body: 'data',    // Per POST requests
  }}
  style={{ width: 200, height: 200 }}
/>
```

**Caching automatico:**
React Native caches immagini remote automaticamente:
```typescript
// Prima richiesta: scarica dal network
<Image source={{ uri: 'https://example.com/profile.jpg' }} />

// Richieste successive: usa cache (no network)
<Image source={{ uri: 'https://example.com/profile.jpg' }} />  // Istantaneo!

// ⚠️ Cache persiste anche dopo app restart (stored on disk)
// Per invalidare: cambia URL (es: aggiungere timestamp)
<Image source={{ uri: `https://example.com/profile.jpg?t=${Date.now()}` }} />
```

**iOS HTTP workaround** (se devi usare HTTP invece di HTTPS):
```xml
<!-- ios/YourApp/Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

### Proprietà Image

Image supporta molte proprietà per controllo su rendering, loading e interazione:

```typescript
<Image
  source={{ uri: 'https://example.com/image.jpg' }}
  style={{ width: 300, height: 200 }}
  
  /**
   * RESIZE MODE: Come l'immagine si adatta al container
   * 
   * 'cover' (DEFAULT): Mantiene aspect ratio, riempie container, croppa eccesso
   *   Immagine 400x400 in box 300x200 → croppa top/bottom per riempire
   *   Uso: Thumbnail, backgrounds, hero images
   * 
   * 'contain': Mantiene aspect ratio, fit completo, possibili barre vuote
   *   Immagine 400x400 in box 300x200 → scala a 200x200, barre ai lati
   *   Uso: Loghi, icone, quando l'intera immagine è importante
   * 
   * 'stretch': IGNORA aspect ratio, stretcha per riempire esatto
   *   Immagine 400x400 in box 300x200 → distorta a 300x200
   *   Uso: Pattern, sfondi non distorsibili, raramente per foto
   * 
   * 'repeat': Ripete l'immagine come pattern (tile)
   *   Uso: Texture, sfondi pattern
   * 
   * 'center': Centra senza scale, croppa se più grande
   *   Uso: Icone native size
   */
  resizeMode="cover"
  
  // LOADING STATES: Migliora UX durante caricamento
  defaultSource={require('./assets/placeholder.png')}  
  // ⚠️ SOLO iOS! Mostra placeholder mentre carica
  // Android NON supporta questa prop (usa pattern con state)
  
  loadingIndicatorSource={require('./assets/loading.gif')}
  // Mostra GIF/spinner durante loading (entrambe le piattaforme)
  
  // EVENTI LIFECYCLE: Tracking del caricamento
  onLoadStart={() => {
    console.log('Loading started');
    // Mostra skeleton/shimmer
  }}
  
  onLoad={(event) => {
    console.log('Image loaded successfully');
    console.log('Size:', event.nativeEvent.source);  // {width, height, uri}
    // Nascondi skeleton, mostra immagine
  }}
  
  onLoadEnd={() => {
    console.log('Loading ended (success or error)');
    // Chiamato DOPO onLoad o onError
    // Puoi nascondere loader qui
  }}
  
  onError={(error) => {
    console.error('Failed to load:', error.nativeEvent.error);
    // Mostra placeholder di errore o retry button
  }}
  
  onProgress={(event) => {
    const progress = event.nativeEvent.loaded / event.nativeEvent.total;
    console.log('Progress:', `${(progress * 100).toFixed(0)}%`);
    // Mostra progress bar per immagini grandi
  }}
  
  // FADE ANIMATION (solo Android)
  fadeDuration={300}  // Milliseconds per fade-in animation
  // iOS fa sempre fade, Android no (a meno che non specifichi)
  // 0 = no fade, 300 = default smooth
  
  // ACCESSIBILITÀ: Screen reader support
  accessible={true}
  accessibilityLabel="Product photo showing red shoes"
  // VoiceOver/TalkBack legge questa descrizione
  // Essenziale per utenti non vedenti
/>
```

### Esempio Completo con Loading

Pattern comune: mostrare loader mentre l'immagine carica, gestire errori. Questo componente implementa la UX ottimale per immagini di rete:

```typescript
const NetworkImage = ({ uri }: { uri: string }) => {
  // State per tracciare lo stato del caricamento
  const [loading, setLoading] = useState(true);  // Inizia con loading=true
  const [error, setError] = useState(false);      // Nessun errore inizialmente
  
  return (
    <View style={styles.container}>
      {/* Mostra spinner SOLO durante il caricamento */}
      {loading && (
        <ActivityIndicator
          // StyleSheet.absoluteFill posiziona il loader sopra l'immagine
          // Equivale a { position: 'absolute', left: 0, right: 0, top: 0, bottom: 0 }
          style={StyleSheet.absoluteFill}
          size="large"  // "small" o "large" - dimensione dello spinner
        />
      )}
      
      {/* Immagine: sempre renderizzata, ma nascosta durante loading */}
      <Image
        source={{ uri }}
        style={styles.image}
        
        // onLoad: chiamato quando l'immagine è completamente caricata
        onLoad={() => {
          console.log('Image loaded successfully');
          setLoading(false);  // Nasconde il loader
        }}
        
        // onError: chiamato se il caricamento fallisce (404, timeout, etc.)
        onError={(errorEvent) => {
          console.error('Image load failed:', errorEvent.nativeEvent.error);
          setLoading(false);  // Nasconde il loader
          setError(true);     // Mostra messaggio di errore
        }}
      />
      
      {/* Messaggio di errore: visibile solo se error=true */}
      {error && (
        <Text style={{ color: 'red', textAlign: 'center' }}>
          Failed to load image
        </Text>
      )}
    </View>
  );
};

/**
 * FLUSSO DI ESECUZIONE:
 * 1. Component monta → loading=true → mostra ActivityIndicator
 * 2. Image inizia a caricare dal network
 * 3a. SUCCESS: onLoad chiamato → loading=false → nasconde loader, mostra immagine
 * 3b. FAILURE: onError chiamato → loading=false, error=true → nasconde loader, mostra errore
 */
```

### ImageBackground

**ImageBackground** = Image + View combinati. Permette di mettere contenuto sopra un'immagine di sfondo.

**Come funziona internamente:**
```typescript
// ImageBackground è semplicemente:
<View>
  <Image {...props} style={[style, { position: 'absolute' }]} />
  {children}
</View>
```

**Uso pratico:**
```typescript
import { ImageBackground } from 'react-native';

<ImageBackground
  source={{ uri: 'https://example.com/bg.jpg' }}  // Immagine di background
  style={{ width: '100%', height: 200 }}           // Dimensioni del container
  resizeMode="cover"  // Come l'immagine riempie lo spazio
  imageStyle={{      // Stili SOLO per l'immagine (non il container)
    opacity: 0.7,    // Sfondo semi-trasparente
    borderRadius: 10 // Arrotonda SOLO immagine
  }}
>
  {/* Children sono renderizzati SOPRA l'immagine */}
  <View style={{ padding: 20 }}>
    <Text style={{ color: 'white', fontSize: 24, fontWeight: 'bold' }}>
      Text over image
    </Text>
    <Text style={{ color: 'white' }}>
      This content appears on top of the background
    </Text>
  </View>
</ImageBackground>
```

**Pattern comune: Hero section con overlay**
```typescript
<ImageBackground source={heroImage} style={styles.hero}>
  {/* Overlay scuro per leggibilità testo */}
  <View style={styles.overlay} />
  
  {/* Contenuto sopra overlay */}
  <View style={styles.content}>
    <Text style={styles.title}>Welcome</Text>
    <Button title="Get Started" />
  </View>
</ImageBackground>

const styles = StyleSheet.create({
  hero: {
    width: '100%',
    height: 300,
    justifyContent: 'center',  // Centra contenuto verticalmente
    alignItems: 'center',      // Centra contenuto orizzontalmente
  },
  overlay: {
    ...StyleSheet.absoluteFillObject,  // Copre tutto il parent
    backgroundColor: 'rgba(0,0,0,0.5)',  // Nero 50% trasparente
  },
  content: {
    zIndex: 1,  // Sopra l'overlay
    alignItems: 'center',
  },
  title: {
    color: 'white',
    fontSize: 32,
    fontWeight: 'bold',
  },
});
```

**⚠️ Performance Note:**
- ImageBackground è conveniente ma leggermente meno performante di View + Image separati
- Per performance critiche: usa `position: 'absolute'` manualmente

---

## 4. ScrollView

**ScrollView** rende scrollabile un contenuto che eccede le dimensioni dello schermo.

**📚 Teoria: Differenza critica con FlatList**
- **ScrollView**: Renderizza TUTTO il contenuto immediatamente (anche fuori schermo)
- **FlatList**: Renderizza SOLO gli item visibili + buffer (virtualizzazione)

**Quando usare ScrollView:**
- ✅ Contenuto statico e limitato (< 50-100 elementi)
- ✅ Form con pochi campi
- ✅ Dettaglio articolo/post (contenuto eterogeneo)
- ✅ Gallery con poche immagini
- ❌ Liste lunghe (> 100 item) → usa FlatList
- ❌ Liste infinite → usa FlatList con pagination

### ScrollView Base

```typescript
import { ScrollView } from 'react-native';

{/* ScrollView: wrapper che abilita scrolling verticale (default) */}
<ScrollView>
  {/* ⚠️ IMPORTANTE: Tutti questi children sono renderizzati IMMEDIATAMENTE,
      anche se fuori schermo. 
      
      Esempio: 1000 item = 1000 componenti montati in memoria
      Impatto: 
      - Memoria alta (ogni component usa RAM)
      - Mount iniziale lento (tutti i componenti devono montare)
      - Re-render lenti (React deve riconciliare tutti i nodi)
      
      Per liste > 50-100 item, usa FlatList per virtualizzazione */}
  <Text>Item 1</Text>
  <Text>Item 2</Text>
  <Text>Item 3</Text>
  {/* Molti altri items */}
</ScrollView>
```

**Come funziona internamente:**
```
Schermo (visibile):
  ┌─────────────┐
  │ Item 1       │  ← Visibile, renderizzato
  │ Item 2       │  ← Visibile, renderizzato
  │ Item 3       │  ← Visibile, renderizzato
  └─────────────┘
  
  Item 4            ← NON visibile, ma COMUNQUE renderizzato!
  Item 5            ← NON visibile, ma COMUNQUE renderizzato!
  ...
  Item 1000         ← NON visibile, ma COMUNQUE renderizzato!
  
Tutti i 1000 item esistono in memoria contemporaneamente.
```

### Proprietà ScrollView

```typescript
<ScrollView
  /**
   * DIREZIONE SCROLL
   */
  horizontal={false}  
  // false (default): scroll verticale (top-to-bottom)
  // true: scroll orizzontale (left-to-right)
  // ⚠️ NON puoi avere entrambi! Per scroll 2D, annida ScrollView
  
  /**
   * COMPORTAMENTO VISUAL
   */
  showsVerticalScrollIndicator={true}  
  // Mostra scrollbar verticale (quella sottile a destra)
  // iOS: sempre semi-trasparente
  // Android: appare durante scroll, svanisce dopo
  // Setta false per UI pulita (es: carousel)
  
  showsHorizontalScrollIndicator={true}
  // Scrollbar orizzontale (per horizontal={true})
  
  bounces={true}  
  // SOLO iOS: effetto "bounce" quando raggiungi fine/inizio
  // true: scroll oltre il limite, poi torna indietro (elastic)
  // false: scroll si ferma al limite (rigid)
  
  alwaysBounceVertical={false}
  // SOLO iOS: bounce anche se contenuto < viewport height
  // Utile per pull-to-refresh su contenuti corti
  
  /**
   * PAGINAZIONE: Snap behavior
   */
  pagingEnabled={false}  
  // true: scroll "snappa" a pagine intere (width/height del ScrollView)
  // Uso: Onboarding screens, image galleries
  // Esempio: ScrollView width=300 → snappa a 0px, 300px, 600px...
  
  snapToInterval={300}  
  // Snap a intervalli custom (in pixel)
  // Esempio: Card carousel con card width=300
  // snapToInterval={300} fa snappare al centro di ogni card
  
  decelerationRate="normal"  
  // Velocità di decelerazione dopo swipe:
  // 'normal': Decelerazione standard
  // 'fast': Si ferma rapidamente (per pagingEnabled)
  // number: 0-1, più alto = più lento
  
  /**
   * PERFORMANCE OPTIMIZATION
   */
  removeClippedSubviews={true}  
  // Android optimization: rimuove view completamente fuori schermo dal native hierarchy
  // Pros: Riduce memoria per liste medie (~50-200 item)
  // Cons: Può causare glitch visivi durante scroll veloce
  // ⚠️ Per liste lunghe, usa FlatList invece!
  
  /**
   * STYLING
   */
  style={{ flex: 1 }}  
  // Style del ScrollView container ESTERNO
  // Usa per: height, width, backgroundColor del container
  
  contentContainerStyle={{ padding: 20 }}  
  // Style del contenuto INTERNO scrollabile
  // Usa per: padding, alignItems, justifyContent del contenuto
  // ⚠️ NON usare flex: 1 qui, impedisce scrolling!
  
  /**
   * EVENTI SCROLL
   */
  onScroll={(event) => {
    // event.nativeEvent contiene info sullo scroll
    const offsetY = event.nativeEvent.contentOffset.y;  // Posizione scroll Y
    const offsetX = event.nativeEvent.contentOffset.x;  // Posizione scroll X
    const contentHeight = event.nativeEvent.contentSize.height;  // Altezza totale contenuto
    const scrollViewHeight = event.nativeEvent.layoutMeasurement.height;  // Altezza viewport
    
    // Esempio: Detect scroll verso bottom
    if (offsetY + scrollViewHeight >= contentHeight - 20) {
      console.log('Near bottom! Load more...');
    }
    
    // Esempio: Header che cambia con scroll
    if (offsetY > 100) {
      console.log('Scrolled past header, show mini header');
    }
  }}
  
  scrollEventThrottle={16}  
  // Frequenza onScroll events in millisecondi
  // 16ms = 60fps (60 chiamate al secondo)
  // 100ms = 10fps (più performante, meno smooth)
  // ⚠️ Più basso = più eventi = più CPU usage
  
  onScrollBeginDrag={() => console.log('User started dragging')}
  onScrollEndDrag={() => console.log('User stopped dragging')}
  onMomentumScrollBegin={() => console.log('Momentum scroll started')}
  onMomentumScrollEnd={() => console.log('Momentum scroll ended')}
  // Lifecycle completo:
  // 1. User tocca e draggа → onScrollBeginDrag
  // 2. Durante drag → onScroll (multiple times)
  // 3. User rilascia → onScrollEndDrag
  // 4. Momentum continua → onMomentumScrollBegin
  // 5. Durante momentum → onScroll (multiple times)
  // 6. Si ferma → onMomentumScrollEnd
  
  /**
   * CONTROLLO PROGRAMMATICO
   */
  ref={scrollViewRef}  // Ref per metodi imperativi (vedi sezione successiva)
>
  <View style={{ height: 2000 }}>
    <Text>Tall content that requires scrolling</Text>
  </View>
</ScrollView>
```

### ScrollView Orizzontale

**Use cases comuni:**
- Image galleries / carousels
- Chip filters (tag selezionabili)
- Story-style cards (Instagram/Facebook)
- Tab bar custom

```typescript
const HorizontalGallery = () => {
  return (
    <ScrollView
      horizontal  // Abilita scroll orizzontale (left-to-right)
      showsHorizontalScrollIndicator={false}  // Nascondi scrollbar per estetica
      
      // contentContainerStyle: stili del contenuto interno
      contentContainerStyle={{ 
        paddingHorizontal: 10,  // Padding ai lati (primo/ultimo item non toccano bordo)
        gap: 10,  // Spazio tra items (RN 0.71+)
      }}
      
      // Performance: snap a item per UX migliore
      snapToInterval={210}  // 200 (image width) + 10 (marginRight)
      decelerationRate="fast"  // Stop rapido dopo swipe
    >
      {/* Map array di immagini */}
      {images.map((img, index) => (
        <Image
          key={index}  // Key obbligatoria per lists
          source={{ uri: img }}
          style={{ 
            width: 200,      // Fixed width per ogni image
            height: 200, 
            marginRight: 10,  // Spazio tra images
            borderRadius: 10, // Arrotonda corners
          }}
        />
      ))}
    </ScrollView>
  );
};
```

**💡 Pattern avanzato: Carousel con indicatore posizione**
```typescript
const Carousel = ({ items }) => {
  const [currentIndex, setCurrentIndex] = useState(0);
  const scrollViewRef = useRef<ScrollView>(null);
  
  const handleScroll = (event: NativeSyntheticEvent<NativeScrollEvent>) => {
    const offsetX = event.nativeEvent.contentOffset.x;
    const index = Math.round(offsetX / ITEM_WIDTH);
    setCurrentIndex(index);
  };
  
  return (
    <View>
      <ScrollView
        ref={scrollViewRef}
        horizontal
        pagingEnabled  // Snap a pagine intere
        showsHorizontalScrollIndicator={false}
        onScroll={handleScroll}
        scrollEventThrottle={16}
      >
        {items.map((item, index) => (
          <View key={index} style={{ width: SCREEN_WIDTH }}>
            {/* Item content */}
          </View>
        ))}
      </ScrollView>
      
      {/* Pagination dots */}
      <View style={styles.pagination}>
        {items.map((_, index) => (
          <View
            key={index}
            style={[
              styles.dot,
              index === currentIndex && styles.activeDot  // Highlight current
            ]}
          />
        ))}
      </View>
    </View>
  );
};
```

### ScrollView Ref Methods

**Controllo programmatico dello scroll:** uso di ref per metodi imperativi

```typescript
const App = () => {
  // Crea ref con tipo specifico per autocompletion
  const scrollViewRef = useRef<ScrollView>(null);
  
  /**
   * scrollTo: Scrolla a coordinate specifiche
   */
  const scrollToTop = () => {
    scrollViewRef.current?.scrollTo({ 
      y: 0,           // Posizione Y (0 = top)
      animated: true  // true = smooth scroll, false = instant jump
    });
  };
  
  const scrollToPosition = (yOffset: number) => {
    scrollViewRef.current?.scrollTo({ 
      y: yOffset, 
      animated: true 
    });
  };
  
  /**
   * scrollToEnd: Scrolla alla fine del contenuto
   */
  const scrollToBottom = () => {
    scrollViewRef.current?.scrollToEnd({ 
      animated: true 
    });
  };
  
  /**
   * Use case: Scroll to error field in form
   */
  const errorFieldRef = useRef<View>(null);
  
  const scrollToError = () => {
    // measureLayout misura la posizione di un elemento
    errorFieldRef.current?.measureLayout(
      // @ts-ignore
      scrollViewRef.current,  // Relative to ScrollView
      (x, y, width, height) => {
        scrollViewRef.current?.scrollTo({
          y: y - 50,  // Scroll con offset per non nascondere sotto header
          animated: true
        });
      },
      () => console.error('Failed to measure')
    );
  };
  
  return (
    <>
      {/* Bottoni controllo scroll */}
      <View style={styles.controls}>
        <Button title="Scroll to Top" onPress={scrollToTop} />
        <Button title="Scroll to Bottom" onPress={scrollToBottom} />
        <Button title="Scroll to Error" onPress={scrollToError} />
      </View>
      
      <ScrollView ref={scrollViewRef}>
        <View style={{ height: 500 }}>
          <Text>Content...</Text>
        </View>
        
        {/* Error field che vogliamo raggiungere */}
        <View ref={errorFieldRef}>
          <TextInput placeholder="Email" />
          <Text style={{ color: 'red' }}>Invalid email</Text>
        </View>
        
        <View style={{ height: 500 }}>
          <Text>More content...</Text>
        </View>
      </ScrollView>
    </>
  );
};
```

**Altri metodi utili:**
```typescript
// scrollViewRef.current?.flashScrollIndicators();
// Fa "flash" la scrollbar brevemente per indicare che c'è contenuto scrollabile
// Utile dopo mount per mostrare all'utente che può scrollare

useEffect(() => {
  // Flash scrollbar dopo 500ms per attirare attenzione
  setTimeout(() => {
    scrollViewRef.current?.flashScrollIndicators();
  }, 500);
}, []);
```

### ⚠️ Quando NON usare ScrollView

ScrollView renderizza **tutti i children immediatamente**, anche se non visibili. Per liste lunghe questo causa:

**Problemi di performance:**
```
Scenario: Lista di 1000 prodotti

ScrollView:
- Mount: Renderizza 1000 componenti → 5-10 secondi di freeze
- Memoria: 1000 componenti in RAM → ~50-200 MB
- Scroll: Laggy, FPS drops
- Re-render: Se parent re-renderizza, riconcilia 1000 nodi → lento

FlatList (virtualizzata):
- Mount: Renderizza solo ~15 componenti visibili → istantaneo
- Memoria: ~15 componenti in RAM → ~2-5 MB
- Scroll: Smooth 60 FPS
- Re-render: Riconcilia solo item visibili → veloce
```

**Quando DEVI usare FlatList invece:**
- ❌ Liste > 50-100 elementi
- ❌ Liste infinite con pagination
- ❌ Feed sociali (Instagram, Twitter)
- ❌ Contatti/chat lists
- ❌ Product catalogs
- ❌ Qualsiasi lista dove la quantità di item è imprevedibile

**Quando ScrollView è OK:**
- ✅ Contenuto statico e limitato (< 50 elementi)
- ✅ Form con 5-20 campi
- ✅ Article detail page (paragrafi, immagini, commenti limitati)
- ✅ Settings screen con sezioni
- ✅ Profile page con bio + pochi post

**Regola pratica:**
```typescript
// ❌ BAD: Lista dinamica
<ScrollView>
  {products.map(product => <ProductCard key={product.id} {...product} />)}
</ScrollView>

// ✅ GOOD: FlatList con virtualizzazione
<FlatList
  data={products}
  renderItem={({ item }) => <ProductCard {...item} />}
  keyExtractor={item => item.id}
/>
```

📚 Approfondimento FlatList: vedi **Capitolo 8 - Liste e Performance**

---

## 5. TextInput

**TextInput** è il componente per input testuale. A differenza di HTML `<input>`, non ha styling di default.

**📚 Teoria: Controlled vs Uncontrolled Components**

React Native favorisce **controlled components** (input controllato da React state):

```
Controlled (raccomandato):
  User digita "A" → onChange chiamato → setState("A") → Re-render con value="A"
  
  Vantaggio: 
  - React state è "single source of truth"
  - Puoi validare/trasformare input prima di aggiornare state
  - Puoi resettare input programmaticamente
  
Uncontrolled (sconsigliato in RN):
  User digita "A" → Input aggiorna internamente → React non sa nulla
  
  Problema:
  - Devi usare ref per accedere al valore
  - Più difficile validare/trasformare
  - No single source of truth
```

### TextInput Base

```typescript
import { TextInput } from 'react-native';

// State per contenere il testo (controlled component)
const [text, setText] = useState('');

<TextInput
  // VALUE: Rende l'input "controlled" (React controlla il valore)
  value={text}  
  // Se ometti value, diventa "uncontrolled" (input controlla se stesso)
  
  // ON CHANGE TEXT: Chiamato ogni volta che il testo cambia
  onChangeText={setText}  
  // Alternativa: onChangeText={(newText) => setText(newText)}
  // Differenza con web: 
  // - Web: onChange riceve event → event.target.value
  // - RN: onChangeText riceve direttamente la stringa (più conveniente!)
  
  placeholder="Enter text..."  // Testo mostrato quando vuoto
  
  // STYLING: A differenza di HTML, TextInput NON ha stili default!
  // Devi aggiungere border, padding, etc. manualmente
  style={{
    borderWidth: 1,        // Aggiungi border (default = 0)
    borderColor: '#ccc',   // Colore border
    padding: 10,           // Spazio interno
    borderRadius: 5,       // Arrotonda corners
    fontSize: 16,          // Dimensione testo
    // backgroundColor: '#fff',  // Sfondo (trasparente di default)
  }}
/>
```

**⚠️ Gotcha comune: No default styles**
```typescript
// ❌ Questo sembra invisibile (no border!)
<TextInput value={text} onChangeText={setText} />

// ✅ Aggiungi sempre stili base
<TextInput 
  value={text} 
  onChangeText={setText}
  style={{ borderWidth: 1, padding: 10 }}  // Ora è visibile
/>
```

### Proprietà TextInput

```typescript
<TextInput
  /**
   * VALUE MANAGEMENT
   */
  value={text}  
  // Controlled: React state controlla il valore
  // Input mostra SEMPRE ciò che è nello state
  
  onChangeText={(newText) => setText(newText)}  
  // Chiamato ogni volta che il testo cambia
  // newText: stringa completa (non solo il carattere aggiunto)
  
  defaultValue="Initial value"  
  // Uncontrolled: valore iniziale senza state
  // ⚠️ Non puoi usare sia value che defaultValue!
  // Use case: Form dove non serve validazione real-time
  
  /**
   * PLACEHOLDER
   */
  placeholder="Enter your email"
  placeholderTextColor="#999"  
  // Colore placeholder (default: grigio chiaro, varia per piattaforma)
  // iOS default: rgba(0,0,0,0.3)
  // Android default: rgba(0,0,0,0.54)
  
  /**
   * KEYBOARD TYPE: Mostra tastiera specifica per tipo di input
   */
  keyboardType="default"  
  // 'default': Tastiera standard (lettere + numeri + simboli)
  // 'email-address': Tastiera con @ e . prominenti
  // 'numeric': Solo numeri (0-9) - iOS mostra anche simboli
  // 'phone-pad': Numeri + simboli telefono (*, #)
  // 'decimal-pad': Numeri + punto decimale (per prezzi, pesi)
  // 'number-pad': Solo numeri (Android mostra numeri, iOS = numeric)
  // 'url': Tastiera con /, .com shortcuts
  // 'web-search': Return key dice "Search" invece di "Return"
  // ⚠️ Nota: keyboardType è un SUGGERIMENTO, l'utente può ancora switchare tastiera
  
  /**
   * APPEARANCE
   */
  secureTextEntry={false}  
  // true: Nasconde testo con ••• (password fields)
  // - Disabilita copia/incolla per security
  // - iOS: aggiunge icona "mostra password"
  // - Android: comportamento varia per versione
  
  multiline={false}  
  // true: TextInput diventa textarea (multiple righe)
  // - Return key va a capo invece di submit
  // - Altezza cresce automaticamente (o usa numberOfLines per fisso)
  
  numberOfLines={4}  
  // Altezza iniziale per multiline (in righe di testo)
  // iOS: suggerimento iniziale (poi auto-grow)
  // Android: altezza fissa
  
  /**
   * AUTO-CORRECT & CAPITALIZATION
   */
  autoCorrect={true}  
  // Correzione automatica durante digitazione
  // Disabilita per: username, email, codici
  
  autoCapitalize="sentences"  
  // 'none': No auto-capitalize (per email, username)
  // 'sentences': Capitalizza dopo . ! ? (default per testo normale)
  // 'words': Capitalizza Ogni Parola (per nomi propri)
  // 'characters': TUTTO MAIUSCOLO (per codici, targhe)
  
  autoComplete="email"  
  // Suggerimenti autofill dal sistema operativo
  // 'email': Suggerisce email salvate
  // 'password': Integrazione con password manager
  // 'username': Suggerisce usernames
  // 'tel': Numeri telefono
  // 'name': Nome completo
  // 'street-address', 'postal-code', 'cc-number'...
  // 'off': Disabilita autofill
  
  /**
   * RETURN KEY (tasto bottom-right keyboard)
   */
  returnKeyType="done"  
  // Cambia label del return key:
  // 'done': "Done" (finish editing)
  // 'go': "Go" (submit/navigate)
  // 'next': "Next" (vai al prossimo field)
  // 'search': "Search" (per search bars)
  // 'send': "Send" (per chat/messaggi)
  // ⚠️ Label è solo visual, devi implementare l'azione in onSubmitEditing
  
  onSubmitEditing={() => console.log('Submit pressed')}  
  // Chiamato quando user preme return key
  // Use case: 
  // - "Next" → focus su campo successivo
  // - "Done" → chiudi tastiera
  // - "Send" → invia messaggio
  
  blurOnSubmit={true}  
  // true: Chiude tastiera quando premi return
  // false: Tastiera resta aperta
  // Multiline: default = false (return va a capo)
  // Single line: default = true (return chiude tastiera)
  
  /**
   * FOCUS MANAGEMENT
   */
  autoFocus={false}  
  // true: Focus automatico al mount (tastiera appare)
  // Use case: Search screen, Login form
  // ⚠️ Attento: l'utente potrebbe non voler la tastiera subito!
  
  onFocus={() => console.log('Input focused')}  
  // Chiamato quando input riceve focus (tastiera appare)
  // Use case: Mostra suggerimenti, highlight field
  
  onBlur={() => console.log('Input blurred')}  
  // Chiamato quando input perde focus (tastiera si chiude)
  // Use case: Validazione finale, nascondi suggerimenti
  
  /**
   * SELECTION
   */
  selectTextOnFocus={false}  
  // true: Seleziona tutto il testo al focus
  // Use case: Facile editing di interi valori (codici, numeri)
  
  selection={{ start: 0, end: 5 }}  
  // Controlla selezione programmaticamente
  // start: posizione inizio cursore (0-based)
  // end: posizione fine selezione
  // { start: 0, end: text.length } = seleziona tutto
  
  /**
   * VALIDATION & CONSTRAINTS
   */
  maxLength={100}  
  // Limita lunghezza massima input (caratteri)
  // User NON può digitare oltre questo limite
  // Use case: Tweet (280), SMS (160), username (20)
  
  /**
   * STYLING
   */
  style={styles.input}  
  // Stili per TextInput (border, padding, fontSize, color, etc.)
  // ⚠️ Non supporta tutti gli stili di Text!
  // OK: fontSize, color, fontFamily, padding, border, backgroundColor
  // NO: fontWeight, textAlign (varia per piattaforma)
  
  /**
   * EDITABLE
   */
  editable={true}  
  // false: Input readonly (user non può editare)
  // Visual: Appare "grayed out"
  // Use case: Mostra valori non editabili in un form
/>
```

### Form Completo

**Pattern: Focus chain per UX ottimale** - Permette all'utente di navigare tra campi con tasto "Next" sulla tastiera

```typescript
const LoginForm = () => {
  // State per ogni campo (controlled components)
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  // Ref per controllo focus programmatico
  // Serve per spostare focus al campo successivo quando user preme "Next"
  const passwordRef = useRef<TextInput>(null);
  
  const handleSubmit = () => {
    // Validazione prima di submit
    if (!email || !password) {
      Alert.alert('Errore', 'Compila tutti i campi');
      return;
    }
    
    console.log('Login:', { email, password });
    // Qui: chiamata API, navigation, etc.
  };
  
  return (
    <View style={styles.form}>
      {/* CAMPO 1: Email */}
      <TextInput
        value={email}  // Controlled by state
        onChangeText={setEmail}  // Aggiorna state ad ogni carattere
        
        placeholder="Email"
        
        // KEYBOARD CONFIGURATION per email
        keyboardType="email-address"  // Tastiera con @ e . prominenti
        autoCapitalize="none"  // Email sempre lowercase
        autoCorrect={false}  // No autocorrect per email
        autoComplete="email"  // Autofill email salvate nel sistema
        
        // RETURN KEY STRATEGY: "Next" per andare al campo successivo
        returnKeyType="next"  // Label del return key = "Next"
        onSubmitEditing={() => {
          // Quando user preme "Next", sposta focus a password
          passwordRef.current?.focus();
        }}
        blurOnSubmit={false}  // NON chiudere tastiera, vai al prossimo campo
        
        style={styles.input}
      />
      
      {/* CAMPO 2: Password */}
      <TextInput
        ref={passwordRef}  // Ref per ricevere focus da campo precedente
        
        value={password}
        onChangeText={setPassword}
        
        placeholder="Password"
        
        // PASSWORD CONFIGURATION
        secureTextEntry  // Nasconde testo con •••
        // secureTextEntry={true} è equivalente
        
        autoComplete="password"  // Integrazione password manager (1Password, etc.)
        
        // RETURN KEY STRATEGY: "Done" per submit form
        returnKeyType="done"  // Label = "Done" (ultimo campo)
        onSubmitEditing={handleSubmit}  // Preme "Done" → submit!
        blurOnSubmit={true}  // Chiude tastiera dopo submit
        
        style={styles.input}
      />
      
      <Button title="Login" onPress={handleSubmit} />
    </View>
  );
};

const styles = StyleSheet.create({
  form: {
    padding: 20,
    gap: 15,  // Spazio tra campi (RN 0.71+)
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    backgroundColor: '#fff',
  },
});
```

**🔄 Focus Chain Flow:**
```
1. User apre form → Email field ha autoFocus (opzionale)
2. User digita email → preme "Next" sulla tastiera
3. onSubmitEditing → passwordRef.current?.focus()
4. Password field riceve focus, tastiera resta aperta
5. User digita password → preme "Done"
6. onSubmitEditing → handleSubmit() chiamato → form submitted!
7. blurOnSubmit={true} → tastiera si chiude
```

**💡 Best Practice: Validazione Real-time**
```typescript
const [emailError, setEmailError] = useState('');

const validateEmail = (text: string) => {
  setEmail(text);
  // Validazione email
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (text && !emailRegex.test(text)) {
    setEmailError('Email non valida');
  } else {
    setEmailError('');
  }
};

<TextInput
  value={email}
  onChangeText={validateEmail}  // Valida ad ogni carattere
  // ...
/>
{emailError ? <Text style={styles.error}>{emailError}</Text> : null}
```

### Gestire la Tastiera

**Keyboard API** fornisce metodi per controllare e monitorare la tastiera native.

**📚 Teoria: Problema della Tastiera**
```
Problema:
- Tastiera appare e copre input field
- User non vede cosa sta digitando
- Form button nascosto sotto tastiera

Soluzioni:
1. KeyboardAvoidingView → auto-sposta contenuto
2. ScrollView → permetti scroll manuale
3. Keyboard.dismiss() → chiudi tastiera con tap outside
4. Keyboard listeners → animazioni custom
```

```typescript
import { Keyboard } from 'react-native';

/**
 * METODI IMPERATIVI
 */

// Chiudere la tastiera programmaticamente
Keyboard.dismiss();
// Use cases:
// - Tap su overlay/background
// - Submit form
// - Navigation ad altra screen
// - Timer/automatic dismiss

/**
 * PATTERN 1: Dismiss on tap outside
 */
const FormScreen = () => {
  return (
    <TouchableWithoutFeedback onPress={Keyboard.dismiss}>
      <View style={{ flex: 1 }}>
        <TextInput placeholder="Email" />
        <TextInput placeholder="Password" />
        {/* Tap ovunque fuori dagli input → tastiera si chiude */}
      </View>
    </TouchableWithoutFeedback>
  );
};

/**
 * PATTERN 2: Dismiss on scroll
 */
<ScrollView keyboardDismissMode="on-drag">
  {/* keyboardDismissMode:
      'none': Tastiera resta aperta durante scroll
      'on-drag': Chiude appena user inizia a scrollare
      'interactive': (iOS) Tastiera segue il dito, può essere "trascinata via" */}
  <TextInput />
</ScrollView>

/**
 * EVENTI TASTIERA: Monitoring keyboard lifecycle
 */
useEffect(() => {
  // iOS: keyboardWillShow/Hide (prima dell'animazione)
  // Android: keyboardDidShow/Hide (dopo l'animazione)
  
  const keyboardDidShow = Keyboard.addListener(
    'keyboardDidShow',
    (event) => {
      console.log('Keyboard appeared');
      console.log('Height:', event.endCoordinates.height);  // Altezza tastiera in pixel
      console.log('Duration:', event.duration);  // Durata animazione (ms)
      
      // Use case: Adjust UI based on keyboard height
      // Esempio: Sposta button sopra tastiera
      const buttonBottom = event.endCoordinates.height + 10;
      Animated.timing(buttonPosition, {
        toValue: -buttonBottom,
        duration: event.duration,
        useNativeDriver: false,
      }).start();
    }
  );
  
  const keyboardDidHide = Keyboard.addListener(
    'keyboardDidHide',
    () => {
      console.log('Keyboard hidden');
      // Ripristina UI
      Animated.timing(buttonPosition, {
        toValue: 0,
        duration: 250,
        useNativeDriver: false,
      }).start();
    }
  );
  
  // ⚠️ CLEANUP obbligatorio per evitare memory leaks!
  return () => {
    keyboardDidShow.remove();  // Rimuovi listener
    keyboardDidHide.remove();
  };
}, []);

/**
 * PATTERN 3: Show/Hide tracking con state
 */
const [keyboardVisible, setKeyboardVisible] = useState(false);

useEffect(() => {
  const showListener = Keyboard.addListener('keyboardDidShow', () => {
    setKeyboardVisible(true);
  });
  const hideListener = Keyboard.addListener('keyboardDidHide', () => {
    setKeyboardVisible(false);
  });
  
  return () => {
    showListener.remove();
    hideListener.remove();
  };
}, []);

// Conditional rendering based on keyboard
{!keyboardVisible && (
  <View style={styles.footer}>
    <Text>Footer nascosto quando tastiera è aperta</Text>
  </View>
)}
```

### KeyboardAvoidingView

**KeyboardAvoidingView** sposta automaticamente il contenuto quando la tastiera appare, evitando che copra gli input.

**📚 Teoria: Come funziona**
```
Senza KeyboardAvoidingView:
  ┌─────────────┐
  │ Content     │
  │ TextInput   │  ← Coperto dalla tastiera!
  ├─────────────┤
  │█████████████│  ← Tastiera
  └─────────────┘

Con KeyboardAvoidingView:
  ┌─────────────┐
  │ Content     │  ← Spostato verso l'alto
  │ TextInput   │  ← Visibile sopra tastiera
  ├─────────────┤
  │█████████████│  ← Tastiera
  └─────────────┘
```

```typescript
import { KeyboardAvoidingView, Platform } from 'react-native';

<KeyboardAvoidingView
  /**
   * BEHAVIOR: Come il contenuto si adatta alla tastiera
   * 
   * ⚠️ Comportamento DIVERSO per iOS e Android!
   */
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  // 
  // 'padding': Aggiunge padding bottom = keyboard height
  //   Pros: Contenuto resta nella stessa posizione, spazio creato sotto
  //   Cons: Se contenuto è alto, parte superiore può uscire dallo schermo
  //   Uso: Form compatti, chat input
  // 
  // 'height': Riduce height del container
  //   Pros: Forza contenuto interno a ridimensionarsi
  //   Cons: Può causare layout inaspettati
  //   Uso: Android (tastiera overlay diverso)
  // 
  // 'position': Sposta contenuto verso l'alto (translate Y)
  //   Pros: Mantiene dimensioni originali
  //   Cons: Contenuto superiore può uscire dallo schermo
  //   Uso: Raramente, per controllo fine
  // 
  // undefined: Nessun adjustment automatico
  
  style={{ flex: 1 }}  
  // ⚠️ flex: 1 è IMPORTANTE! Permette al container di riempire lo schermo
  
  /**
   * KEYBOARD VERTICAL OFFSET
   * 
   * Aggiusta l'offset se KeyboardAvoidingView non calcola perfettamente
   * (es: header custom, tab bar)
   */
  keyboardVerticalOffset={Platform.OS === 'ios' ? 64 : 0}
  // 64 = altezza navigation bar iOS
  // Calcolo: Header height + Status bar height + Tab bar height (se presente)
  
  /**
   * ENABLED
   */
  enabled={true}  
  // false: Disabilita behavior (utile per conditional logic)
  // Esempio: enabled={Platform.OS === 'ios'} per solo iOS
>
  {/* Il contenuto qui dentro sarà spostato automaticamente */}
  <ScrollView contentContainerStyle={{ flexGrow: 1 }}>
    <View style={{ flex: 1, justifyContent: 'center', padding: 20 }}>
      <TextInput 
        placeholder="Email" 
        style={styles.input}
      />
      <TextInput 
        placeholder="Password" 
        secureTextEntry
        style={styles.input}
      />
      <Button title="Login" onPress={() => {}} />
    </View>
  </ScrollView>
</KeyboardAvoidingView>
```

**💡 Pattern Completo: Form con KeyboardAvoidingView + ScrollView**
```typescript
// Questo pattern gestisce TUTTI i casi:
// - Tastiera copre input ✅
// - Contenuto troppo alto per schermo ✅  
// - Scroll manuale disponibile ✅
// - Focus automatico visibile ✅

const FormScreen = () => {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={{ flex: 1 }}
    >
      <TouchableWithoutFeedback onPress={Keyboard.dismiss}>
        <ScrollView 
          contentContainerStyle={{ flexGrow: 1, padding: 20 }}
          keyboardShouldPersistTaps="handled"  
          // 'handled': Elementi touchable ricevono tap, altri chiudono tastiera
          // 'always': Tap mai chiude tastiera (serve Keyboard.dismiss() esplicito)
          // 'never': (default) Tap chiude sempre tastiera
        >
          <TextInput placeholder="Nome" style={styles.input} />
          <TextInput placeholder="Email" style={styles.input} />
          <TextInput placeholder="Password" style={styles.input} />
          {/* Molti altri campi... */}
          <Button title="Submit" onPress={handleSubmit} />
        </ScrollView>
      </TouchableWithoutFeedback>
    </KeyboardAvoidingView>
  );
};
```

**⚠️ Problemi comuni:**
```typescript
// ❌ Dimenticato flex: 1
<KeyboardAvoidingView behavior="padding">
  // Non funziona! Container non riempie schermo
</KeyboardAvoidingView>

// ✅ Corretto
<KeyboardAvoidingView behavior="padding" style={{ flex: 1 }}>
  // Funziona!
</KeyboardAvoidingView>

// ❌ Behavior sbagliato per piattaforma
<KeyboardAvoidingView behavior="padding">  
  // Android: layout rotto
</KeyboardAvoidingView>

// ✅ Platform-specific
<KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : 'height'}>
  // Funziona su entrambe le piattaforme
</KeyboardAvoidingView>
```

---

## 6. Button e Touchable Components

**📚 Teoria: Gerarchia dei Componenti Touch**

React Native offre diversi componenti per gestire touch/press events, ognuno con pro/contro:

```
Button (built-in, limitato)
  ↓
TouchableOpacity (opacità feedback)
  ↓
TouchableHighlight (background feedback)
  ↓
TouchableWithoutFeedback (no visual feedback)
  ↓
Pressable (moderno, RN 0.63+, più potente) ⭐
```

### Button (Limitato)

**Button** è il componente più semplice, ma **molto limitato** nello styling.

```typescript
import { Button } from 'react-native';

<Button
  title="Press Me"  // Solo testo, NO icon, NO custom layout
  onPress={() => console.log('Pressed')}
  
  // COLOR: l'UNICA proprietà di styling disponibile!
  color="#007AFF"  
  // iOS: Cambia colore del TESTO
  // Android: Cambia colore del BACKGROUND
  // ⚠️ Comportamento DIVERSO per piattaforma!
  
  disabled={false}  // true = button non cliccabile, grayed out
  
  // accessibilityLabel per screen readers
  accessibilityLabel="Tap to submit form"
/>
```

**⚠️ Limitazioni di Button:**
- ❌ Non puoi customizzare font, padding, border
- ❌ Non puoi aggiungere icone o immagini
- ❌ Non puoi cambiare il layout interno
- ❌ Stile nativo varia molto tra iOS/Android
- ❌ Non puoi controllare l'animazione press

**Quando usare Button:**
- ✅ Prototipi rapidi
- ✅ Debug/testing
- ✅ App interne dove l'estetica non è critica
- ❌ App production con brand identity → usa Touchable/Pressable

### TouchableOpacity (⭐ Più usato)

**TouchableOpacity** riduce l'opacità quando premuto. **Scelta più popolare** per button custom.

```typescript
import { TouchableOpacity, Text } from 'react-native';

<TouchableOpacity
  /**
   * EVENTI TOUCH
   */
  onPress={() => console.log('Pressed')}  
  // Chiamato su tap (press + release veloce)
  // ⚠️ onPress è l'evento principale, sempre implementalo!
  
  onLongPress={() => console.log('Long press')}  
  // Chiamato su pressione prolungata (>500ms)
  // Use case: Context menu, delete action, additional options
  
  onPressIn={() => console.log('Press started')}  
  // Chiamato quando dito tocca lo schermo (inizio press)
  
  onPressOut={() => console.log('Press ended')}  
  // Chiamato quando dito si solleva (fine press)
  
  /**
   * PRESS LIFECYCLE:
   * 1. Dito tocca → onPressIn
   * 2. Dito si solleva rapidamente → onPressOut → onPress
   * 3. Dito resta premuto >500ms → onLongPress
   */
  
  /**
   * COMPORTAMENTO
   */
  disabled={false}  
  // true: Non risponde ai touch, opacity ridotta automaticamente
  
  activeOpacity={0.7}  
  // Opacità quando premuto (0 = invisible, 1 = no change)
  // Default: 0.2 (molto trasparente)
  // 0.7: Effetto più subtle (consigliato)
  // 0.5: Effetto medio
  // 0.2: Effetto pronunciato (default)
  
  delayPressIn={0}  
  // Millisecondi di delay prima di onPressIn
  // Utile per evitare accidental press durante scroll
  
  delayPressOut={0}  
  // Delay prima di onPressOut
  
  delayLongPress={500}  
  // Millisecondi per attivare onLongPress (default: 500)
  
  /**
   * HIT AREA: Espandi area touchable oltre i bounds visibili
   */
  hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}  
  // Utile per piccoli buttons (<44x44pt iOS guideline)
  // Esempi: close buttons, icon buttons
  
  pressRetentionOffset={{ top: 20, bottom: 20, left: 20, right: 20 }}  
  // Quanti pixel può uscire il dito mantenendo il press
  // Utile per evitare accidental cancels
  
  /**
   * STYLING: Puoi usare qualsiasi stile View!
   */
  style={{
    backgroundColor: '#007AFF',  // Background color
    padding: 15,                  // Spaziatura interna
    borderRadius: 8,              // Bordi arrotondati
    alignItems: 'center',         // Centra contenuto
    justifyContent: 'center',
    // margin, borderWidth, shadowColor, etc. - TUTTO supportato!
  }}
>
  {/* Children: puoi mettere QUALSIASI componente */}
  <Text style={{ color: 'white', fontSize: 16, fontWeight: '600' }}>
    Custom Button
  </Text>
  {/* Puoi aggiungere icon, immagini, layout complessi */}
</TouchableOpacity>
```

**Vantaggi TouchableOpacity:**
- ✅ Visual feedback automatico (opacity animation)
- ✅ Styling completamente custom
- ✅ Supporta children complessi (text + icons)
- ✅ Hit area configurabile
- ✅ Funziona su tutte le piattaforme identicamente

### TouchableHighlight

**TouchableHighlight** mostra un colore di background quando premuto. Utile per liste.

```typescript
import { TouchableHighlight } from 'react-native';

<TouchableHighlight
  onPress={() => {}}
  
  /**
   * UNDERLAY COLOR: Colore mostrato quando premuto
   */
  underlayColor="#0051a8"  
  // Il background diventa questo colore durante press
  // Tip: Usa un colore più scuro della backgroundColor per effetto "pressed"
  // Esempio: backgroundColor="#007AFF" → underlayColor="#0051a8" (30% darker)
  
  /**
   * ACTIVE OPACITY: Opacità del CONTENUTO durante press
   */
  activeOpacity={0.85}  
  // Default: 0.85 (leggermente trasparente)
  // Combinato con underlayColor crea effetto "press"
  
  onShowUnderlay={() => console.log('Underlay shown')}  
  // Chiamato quando underlay diventa visibile
  
  onHideUnderlay={() => console.log('Underlay hidden')}  
  // Chiamato quando underlay scompare
  
  style={{
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8
  }}
>
  {/* ⚠️ TouchableHighlight può avere UN SOLO child diretto! */}
  <View>  {/* Wrapper obbligatorio se hai più elementi */}
    <Text style={{ color: 'white' }}>Touch Highlight</Text>
  </View>
</TouchableHighlight>
```

**Quando usare TouchableHighlight:**
- ✅ Liste/tabelle (Material Design style)
- ✅ Quando vuoi background color change invece di opacity
- ❌ Buttons standalone (TouchableOpacity è meglio)

**⚠️ Gotcha:**
```typescript
// ❌ ERRORE: Multiple children diretti
<TouchableHighlight>
  <Text>Text 1</Text>
  <Text>Text 2</Text>  // ❌ Errore!
</TouchableHighlight>

// ✅ Soluzione: Wrappa in View
<TouchableHighlight>
  <View>
    <Text>Text 1</Text>
    <Text>Text 2</Text>
  </View>
</TouchableHighlight>
```

### TouchableWithoutFeedback

**TouchableWithoutFeedback** gestisce touch SENZA visual feedback. Raramente usato.

```typescript
import { TouchableWithoutFeedback, View, Text } from 'react-native';

<TouchableWithoutFeedback 
  onPress={() => console.log('Pressed')}
>
  {/* ⚠️ Deve avere ESATTAMENTE un child diretto */}
  <View>
    <Text>No visual feedback</Text>
    {/* Nessuna animazione, opacity, o underlay quando premuto */}
  </View>
</TouchableWithoutFeedback>
```

**Quando usare TouchableWithoutFeedback:**
- ✅ Dismissing keyboards (tap on background)
- ✅ Closing modals (tap on overlay)
- ✅ Custom feedback completamente personalizzato (gestisci tu l'animazione)
- ❌ Buttons normali (sempre usa feedback visivo per UX!)

**Use case comune: Dismiss keyboard**
```typescript
<TouchableWithoutFeedback onPress={Keyboard.dismiss}>
  <View style={{ flex: 1 }}>
    <TextInput placeholder="Email" />
    {/* Tap ovunque → tastiera si chiude, nessun visual feedback */}
  </View>
</TouchableWithoutFeedback>
```

### Pressable (⭐ Moderno, React Native 0.63+)

**Pressable** è il componente touch **più moderno e potente**. Sostituisce tutti i Touchable con API unificata.

**Perché Pressable è migliore:**
- ✅ Style function con stato press (TouchableOpacity non ha questo)
- ✅ Children function con stato press
- ✅ Controllo completo su feedback visivo
- ✅ Hit rect configuration avanzata
- ✅ Press lifecycle events dettagliati
- ✅ Performance migliori (ottimizzato internamente)

```typescript
import { Pressable, Text } from 'react-native';

<Pressable
  /**
   * EVENTI: Stessi di TouchableOpacity + altri
   */
  onPress={() => console.log('Pressed')}
  onLongPress={() => console.log('Long press')}
  onPressIn={() => console.log('Press started')}
  onPressOut={() => console.log('Press ended')}
  
  // ✨ NUOVO: Hover events (per web/desktop)
  onHoverIn={() => console.log('Mouse entered')}  // React Native Web
  onHoverOut={() => console.log('Mouse left')}
  
  disabled={false}
  delayLongPress={500}
  
  /**
   * HIT RECT: Configurazione avanzata area touch
   */
  hitSlop={10}  // Numero = tutti i lati
  // oppure object:
  hitSlop={{ top: 10, bottom: 10, left: 5, right: 5 }}  
  
  pressRetentionOffset={20}  // Quanto può uscire il dito
  
  /**
   * ANDROID RIPPLE: Effetto Material Design (solo Android)
   */
  android_ripple={{
    color: 'rgba(0, 0, 0, 0.1)',  // Colore ripple
    borderless: false,  // true = ripple oltre i bounds
    radius: 150,  // Raggio massimo ripple
  }}
  // Questo crea l'effetto "ripple" nativo di Android
  // iOS ignora questa prop (usa style function invece)
  
  /**
   * ✨ STYLE FUNCTION: Accesso allo stato pressed!
   */
  style={({ pressed }) => [
    // ↑ pressed: boolean che indica se attualmente premuto
    {
      backgroundColor: pressed ? '#0051a8' : '#007AFF',  
      // Cambia colore durante press!
      
      padding: 15,
      borderRadius: 8,
      alignItems: 'center',
      
      // Effetto scale durante press
      transform: [{ scale: pressed ? 0.96 : 1 }],
      
      // Opacity alternativa
      opacity: pressed ? 0.8 : 1,
    },
    // Puoi combinare con stili statici
    styles.button,
  ]}
>
  {/**
    * ✨ CHILDREN FUNCTION: Render basato su stato!
    */}
  {({ pressed }) => (
    // ↑ pressed disponibile anche nei children
    <Text style={{ 
      color: 'white',
      fontWeight: pressed ? 'bold' : '600',  // Bold quando premuto
    }}>
      {pressed ? 'Pressed!' : 'Press Me'}
      {/* Cambia testo dinamicamente */}
    </Text>
  )}
</Pressable>
```

**💡 Pattern avanzato: Button con loading state**
```typescript
const [loading, setLoading] = useState(false);

<Pressable
  onPress={async () => {
    setLoading(true);
    await submitForm();
    setLoading(false);
  }}
  disabled={loading}  // Disabilita durante loading
  style={({ pressed }) => ({
    backgroundColor: loading ? '#ccc' : (pressed ? '#0051a8' : '#007AFF'),
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    opacity: loading ? 0.6 : 1,
  })}
>
  {({ pressed }) => (
    loading ? (
      <ActivityIndicator color="white" />
    ) : (
      <Text style={{ color: 'white' }}>
        {pressed ? 'Submitting...' : 'Submit'}
      </Text>
    )
  )}
</Pressable>
```

**🎯 Quando usare cosa:**
```typescript
// ✅ Pressable: Default choice per button custom (moderno, potente)
<Pressable style={({ pressed }) => ({ opacity: pressed ? 0.5 : 1 })}>
  <Text>Modern Button</Text>
</Pressable>

// ✅ TouchableOpacity: Se supporti RN < 0.63, o preferisci API semplice
<TouchableOpacity activeOpacity={0.7}>
  <Text>Legacy Button</Text>
</TouchableOpacity>

// ✅ TouchableHighlight: Liste, Material Design style
<TouchableHighlight underlayColor="#f0f0f0">
  <Text>List Item</Text>
</TouchableHighlight>

// ✅ Button: Prototipi, debug, casi dove styling non è importante
<Button title="Quick Test" onPress={() => {}} />

// ❌ TouchableWithoutFeedback: Raramente (solo per dismiss keyboard/modal)
```

### Custom Button Component

**Pattern riutilizzabile:** Crea un button component per l'intera app con varianti, loading, disabled states.

```typescript
import React from 'react';
import { 
  TouchableOpacity, 
  Text, 
  StyleSheet, 
  ActivityIndicator,
  ViewStyle,
  TextStyle 
} from 'react-native';

/**
 * TypeScript Interface per props
 */
interface CustomButtonProps {
  title: string;  // Testo button
  onPress: () => void | Promise<void>;  // Handler (può essere async)
  variant?: 'primary' | 'secondary' | 'danger' | 'outline';  // Varianti
  size?: 'small' | 'medium' | 'large';  // Dimensioni
  disabled?: boolean;  // Stato disabled
  loading?: boolean;  // Mostra loading spinner
  fullWidth?: boolean;  // Occupa tutta la larghezza
  icon?: React.ReactNode;  // Icona opzionale
  style?: ViewStyle;  // Stili custom
  textStyle?: TextStyle;  // Stili testo custom
}

/**
 * Custom Button Component
 * 
 * Esempi uso:
 * <CustomButton title="Save" onPress={handleSave} />
 * <CustomButton title="Delete" variant="danger" onPress={handleDelete} />
 * <CustomButton title="Loading" loading onPress={() => {}} />
 */
const CustomButton: React.FC<CustomButtonProps> = ({
  title,
  onPress,
  variant = 'primary',  // Default variant
  size = 'medium',
  disabled = false,
  loading = false,
  fullWidth = false,
  icon,
  style,
  textStyle,
}) => {
  /**
   * Colori per ogni variante
   */
  const getBackgroundColor = () => {
    if (disabled || loading) return '#ccc';  // Gray quando disabled/loading
    
    switch (variant) {
      case 'primary':
        return '#007AFF';  // iOS blue
      case 'secondary':
        return '#5856D6';  // Purple
      case 'danger':
        return '#FF3B30';  // Red
      case 'outline':
        return 'transparent';  // No background
      default:
        return '#007AFF';
    }
  };
  
  /**
   * Dimensioni per ogni size
   */
  const getPadding = () => {
    switch (size) {
      case 'small':
        return { paddingVertical: 8, paddingHorizontal: 16 };
      case 'medium':
        return { paddingVertical: 12, paddingHorizontal: 24 };
      case 'large':
        return { paddingVertical: 16, paddingHorizontal: 32 };
      default:
        return { paddingVertical: 12, paddingHorizontal: 24 };
    }
  };
  
  const getFontSize = () => {
    switch (size) {
      case 'small': return 14;
      case 'medium': return 16;
      case 'large': return 18;
      default: return 16;
    }
  };
  
  /**
   * Handler che supporta async
   */
  const handlePress = async () => {
    if (disabled || loading) return;  // Ignora se disabled/loading
    await onPress();  // Await se onPress è async
  };
  
  return (
    <TouchableOpacity
      onPress={handlePress}
      disabled={disabled || loading}
      activeOpacity={0.7}
      style={[
        styles.button,
        {
          backgroundColor: getBackgroundColor(),
          ...getPadding(),
          // Outline variant: aggiungi border
          ...(variant === 'outline' && {
            borderWidth: 2,
            borderColor: '#007AFF',
          }),
          // Full width
          ...(fullWidth && { width: '100%' }),
        },
        disabled && styles.disabled,  // Opacity ridotta se disabled
        style,  // Stili custom dall'esterno
      ]}
    >
      {/* Loading spinner OPPURE contenuto normale */}
      {loading ? (
        <ActivityIndicator 
          color={variant === 'outline' ? '#007AFF' : 'white'} 
        />
      ) : (
        <>
          {/* Icon opzionale prima del testo */}
          {icon && <View style={styles.icon}>{icon}</View>}
          
          {/* Testo button */}
          <Text
            style={[
              styles.buttonText,
              {
                fontSize: getFontSize(),
                color: variant === 'outline' ? '#007AFF' : 'white',
              },
              textStyle,  // Stili testo custom
            ]}
          >
            {title}
          </Text>
        </>
      )}
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
    flexDirection: 'row',  // Per supportare icon + text
    minHeight: 44,  // iOS Human Interface Guidelines
  },
  buttonText: {
    fontWeight: '600',
    textAlign: 'center',
  },
  disabled: {
    opacity: 0.5,  // Visual feedback per disabled
  },
  icon: {
    marginRight: 8,  // Spazio tra icon e text
  },
});

export default CustomButton;

/**
 * USO NEL CODICE:
 */
// Primary button
<CustomButton 
  title="Save Changes" 
  onPress={handleSave} 
/>

// Danger button
<CustomButton 
  title="Delete Account" 
  variant="danger" 
  onPress={handleDelete} 
/>

// Outline button
<CustomButton 
  title="Cancel" 
  variant="outline" 
  onPress={handleCancel} 
/>

// Loading state
<CustomButton 
  title="Submitting..." 
  loading={isSubmitting}
  onPress={handleSubmit} 
/>

// Con icona
<CustomButton 
  title="Share" 
  icon={<Icon name="share" size={18} color="white" />}
  onPress={handleShare} 
/>

// Full width
<CustomButton 
  title="Continue" 
  fullWidth 
  onPress={handleContinue} 
/>
```

---

## 7. Modal

**Modal** presenta contenuto sopra la view corrente, bloccando l'interazione con il contenuto sottostante.

**📚 Teoria: Modal vs Navigation**
```
Modal:
- Sovrappone contenuto current screen
- Non è parte dello stack di navigazione
- Dismiss con stato interno (visible state)
- Use case: Dialog, alert custom, form veloce, picker

Navigation (Screen):
- Parte dello navigation stack
- Back button automatico
- Deep linking support
- Use case: Nuove pagine, flow complessi
```

### Modal Base

```typescript
import { Modal, View, Text, Button } from 'react-native';

const [modalVisible, setModalVisible] = useState(false);

<>
  {/* Button per aprire modal */}
  <Button title="Open Modal" onPress={() => setModalVisible(true)} />
  
  <Modal
    /**
     * VISIBILITY: Controllato da state
     */
    visible={modalVisible}  
    // true: Modal appare
    // false: Modal nascosto
    // ⚠️ Modal monta/smonta con visible, non solo show/hide
    
    /**
     * ON REQUEST CLOSE: OBBLIGATORIO per Android back button
     */
    onRequestClose={() => setModalVisible(false)}  
    // Android: Chiamato quando user preme hardware back button
    // iOS: Non chiamato (no back button)
    // ⚠️ Se non implementi, back button non fa nulla su Android!
    
    /**
     * ANIMATION TYPE: Come il modal appare/scompare
     */
    animationType="slide"  
    // 'none': No animation (instant)
    // 'slide': Slide dal bottom (default iOS style)
    // 'fade': Fade in/out (smooth, elegant)
    
    /**
     * TRANSPARENT: Background del modal
     */
    transparent={false}  
    // false (default): Background bianco/nero opaco (copre tutto)
    // true: Background trasparente (vedi contenuto sotto)
    //   → Utile per dialog/alert custom con overlay
    
    /**
     * STATUS BAR: Come gestire status bar durante modal
     */
    statusBarTranslucent={false}  // Android: Modal sovrappone status bar
    
    /**
     * PRESENTATION STYLE: Solo iOS
     */
    presentationStyle="fullScreen"  
    // iOS styles:
    // 'fullScreen': Copre tutto lo schermo
    // 'pageSheet': Card style, swipe down to dismiss (iOS 13+)
    // 'formSheet': Centrato su iPad
    // 'overFullScreen': Trasparente sopra contenuto
  >
    {/* Contenuto modal: qualsiasi componente */}
    <View style={{ 
      flex: 1, 
      justifyContent: 'center', 
      alignItems: 'center',
      backgroundColor: 'white',  // Background contenuto
    }}>
      <Text style={{ fontSize: 24, marginBottom: 20 }}>Modal Content</Text>
      <Button 
        title="Close" 
        onPress={() => setModalVisible(false)} 
      />
    </View>
  </Modal>
</>
```

### Modal Trasparente con Overlay

**Pattern più comune:** Dialog/alert custom con background scuro semi-trasparente.

```typescript
const [visible, setVisible] = useState(false);

<Modal
  visible={visible}
  transparent={true}  // ⭐ Chiave per overlay effect
  animationType="fade"  // Fade smooth per dialog
  onRequestClose={() => setVisible(false)}
>
  {/* OVERLAY: Background scuro semi-trasparente */}
  <View style={styles.overlay}>
    {/* MODAL CONTENT: Card centrata */}
    <View style={styles.modalContent}>
      {/* Header */}
      <Text style={styles.title}>Confirm Action</Text>
      
      {/* Body */}
      <Text style={styles.message}>
        Are you sure you want to proceed? This action cannot be undone.
      </Text>
      
      {/* Buttons */}
      <View style={styles.buttons}>
        <Button 
          title="Cancel" 
          onPress={() => setVisible(false)} 
        />
        <Button 
          title="Confirm" 
          onPress={() => {
            console.log('Confirmed');
            setVisible(false);  // Chiudi modal dopo action
          }} 
        />
      </View>
    </View>
  </View>
</Modal>

const styles = StyleSheet.create({
  overlay: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.5)',  // Nero 50% trasparente
    justifyContent: 'center',  // Centra verticalmente
    alignItems: 'center',      // Centra orizzontalmente
    padding: 20,  // Padding ai lati
  },
  modalContent: {
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 20,
    width: '100%',
    maxWidth: 400,  // Max width per tablet/landscape
    // Shadow per elevazione
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,  // Android shadow
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  message: {
    fontSize: 16,
    color: '#666',
    marginBottom: 20,
    lineHeight: 22,
  },
  buttons: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginTop: 10,
  },
});
```

### Bottom Sheet Modal

**Pattern moderno:** Modal che slide dal bottom (Instagram, Twitter style).

```typescript
interface BottomSheetProps {
  visible: boolean;
  onClose: () => void;
  children: React.ReactNode;
}

const BottomSheetModal: React.FC<BottomSheetProps> = ({ 
  visible, 
  onClose, 
  children 
}) => {
  return (
    <Modal
      visible={visible}
      transparent
      animationType="slide"  // Slide dal bottom
      onRequestClose={onClose}
    >
      {/* TouchableWithoutFeedback per chiudere con tap su overlay */}
      <TouchableWithoutFeedback onPress={onClose}>
        <View style={styles.overlay}>
          {/* TouchableWithoutFeedback interno per NON chiudere con tap sul content */}
          <TouchableWithoutFeedback onPress={() => {}}>
            <View style={styles.bottomSheet}>
              {/* Handle bar per visual feedback "draggable" */}
              <View style={styles.handle} />
              
              {/* Contenuto custom */}
              {children}
            </View>
          </TouchableWithoutFeedback>
        </View>
      </TouchableWithoutFeedback>
    </Modal>
  );
};

const styles = StyleSheet.create({
  overlay: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.5)',
    justifyContent: 'flex-end',  // Allinea al bottom
  },
  bottomSheet: {
    backgroundColor: 'white',
    borderTopLeftRadius: 20,  // Arrotonda solo top corners
    borderTopRightRadius: 20,
    padding: 20,
    minHeight: 200,
    maxHeight: '80%',  // Max 80% altezza schermo
  },
  handle: {
    width: 40,
    height: 4,
    backgroundColor: '#ccc',
    borderRadius: 2,
    alignSelf: 'center',  // Centra orizzontalmente
    marginBottom: 10,
  },
});

/**
 * USO:
 */
<BottomSheetModal
  visible={sheetVisible}
  onClose={() => setSheetVisible(false)}
>
  <Text style={{ fontSize: 18, fontWeight: 'bold' }}>Options</Text>
  <Button title="Option 1" onPress={() => {}} />
  <Button title="Option 2" onPress={() => {}} />
  <Button title="Cancel" onPress={() => setSheetVisible(false)} />
</BottomSheetModal>
```

**💡 Pattern avanzato: Modal con keyboard**
```typescript
// Modal che contiene form con TextInput
<Modal visible={visible} animationType="slide">
  <KeyboardAvoidingView 
    behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    style={{ flex: 1 }}
  >
    <View style={{ flex: 1, padding: 20 }}>
      <Text>Enter your name:</Text>
      <TextInput 
        placeholder="Name" 
        style={styles.input}
      />
      <Button title="Submit" onPress={handleSubmit} />
    </View>
  </KeyboardAvoidingView>
</Modal>
```

---

## 8. Altri Componenti UI

### Switch (Toggle)

**Switch** è un toggle on/off, usato per settings e preferenze.

```typescript
import { Switch } from 'react-native';

const [enabled, setEnabled] = useState(false);

<Switch
  /**
   * VALUE: Stato corrente (true = ON, false = OFF)
   */
  value={enabled}  
  // ⚠️ Deve essere controllato da state!
  
  /**
   * ON VALUE CHANGE: Chiamato quando user toggle
   */
  onValueChange={setEnabled}  
  // Riceve il nuovo valore (boolean)
  // Alternativa: (newValue) => setEnabled(newValue)
  
  /**
   * TRACK COLOR: Colore della "track" (lo sfondo)
   */
  trackColor={{ 
    false: '#767577',  // Colore quando OFF (grigio scuro)
    true: '#81b0ff'    // Colore quando ON (blu chiaro)
  }}
  
  /**
   * THUMB COLOR: Colore del "thumb" (il pallino che si muove)
   */
  thumbColor={enabled ? '#f5dd4b' : '#f4f3f4'}
  // Giallo quando ON, bianco quando OFF
  // Tip: Usa colore brand quando ON per consistency
  
  /**
   * iOS SPECIFIC
   */
  ios_backgroundColor="#3e3e3e"  
  // Background quando OFF (solo iOS)
  // Usato perché trackColor.false non è sempre visibile su iOS
  
  disabled={false}  // true = non interagibile
/>
```

**Pattern comune: Settings row**
```typescript
const SettingsRow = ({ label, value, onValueChange }) => (
  <View style={styles.row}>
    <Text style={styles.label}>{label}</Text>
    <Switch
      value={value}
      onValueChange={onValueChange}
      trackColor={{ false: '#767577', true: '#34C759' }}  // iOS green
      thumbColor="#fff"
    />
  </View>
);

const styles = StyleSheet.create({
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 12,
    paddingHorizontal: 16,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  label: {
    fontSize: 16,
    color: '#333',
  },
});

// Uso:
<SettingsRow 
  label="Enable Notifications" 
  value={notificationsEnabled}
  onValueChange={setNotificationsEnabled}
/>
```

### ActivityIndicator (Loading Spinner)

**ActivityIndicator** mostra uno spinner di caricamento nativo.

```typescript
import { ActivityIndicator, View } from 'react-native';

{/* Basic spinner */}
<ActivityIndicator 
  size="small"  // 'small' o 'large' (o numero su Android)
  color="#0000ff"  // Colore spinner
/>

<ActivityIndicator size="large" color="#00ff00" />

{/* ⚠️ Dimensioni:
    iOS: 'small' = 20pt, 'large' = 36pt (solo questi 2 valori)
    Android: 'small' = 16dp, 'large' = 32dp, oppure numero custom (es: 50) */}

{/**
  * PATTERN 1: Loading overlay full screen
  */}
{loading && (
  <View style={[
    StyleSheet.absoluteFill,  // Copre tutto lo schermo
    { 
      backgroundColor: 'rgba(0, 0, 0, 0.3)',  // Background scuro semi-trasparente
      justifyContent: 'center',
      alignItems: 'center',
      zIndex: 999,  // Sopra tutto
    }
  ]}>
    <ActivityIndicator size="large" color="#fff" />
  </View>
)}

{/**
  * PATTERN 2: Loading in button
  */}
<TouchableOpacity 
  onPress={handleSubmit} 
  disabled={loading}
  style={styles.button}
>
  {loading ? (
    <ActivityIndicator size="small" color="white" />
  ) : (
    <Text style={styles.buttonText}>Submit</Text>
  )}
</TouchableOpacity>

{/**
  * PATTERN 3: Loading inline in lista
  */}
<ScrollView>
  {data.map(item => <ItemCard key={item.id} {...item} />)}
  
  {/* Spinner al bottom durante pagination */}
  {loadingMore && (
    <View style={{ paddingVertical: 20 }}>
      <ActivityIndicator size="small" color="#999" />
    </View>
  )}
</ScrollView>

{/**
  * PATTERN 4: Loading center screen
  */}
{loading ? (
  <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
    <ActivityIndicator size="large" color="#007AFF" />
    <Text style={{ marginTop: 10, color: '#666' }}>Loading...</Text>
  </View>
) : (
  <ContentView />
)}
```

### Alert (API, non Component)

**Alert** mostra dialog nativi del sistema operativo.

```typescript
import { Alert } from 'react-native';

/**
 * ALERT SEMPLICE: Solo messaggio
 */
Alert.alert('Success', 'Your changes have been saved');
// iOS: Dialog nativo con "OK" button
// Android: Toast-style o dialog (dipende da versione)

/**
 * ALERT CON BOTTONI: Azioni custom
 */
Alert.alert(
  'Delete Item',  // Titolo
  'Are you sure you want to delete this item? This action cannot be undone.',  // Messaggio
  [
    // Array di bottoni (max 3 su iOS, illimitati su Android)
    {
      text: 'Cancel',  // Label button
      style: 'cancel',  // Stile: 'cancel', 'default', 'destructive'
      // 'cancel': Bold su iOS, dismisses automaticamente
      // 'default': Normal weight
      // 'destructive': Rosso su iOS (per azioni pericolose)
    },
    {
      text: 'Delete',
      onPress: () => {
        console.log('Item deleted');
        // Esegui azione
      },
      style: 'destructive',  // Rosso per enfatizzare pericolo
    },
  ],
  {
    // Opzioni alert (terzo parametro)
    cancelable: true,  // Android: tap fuori dismisses (default: false)
    onDismiss: () => console.log('Alert dismissed'),  // Android only
  }
);

/**
 * PROMPT: Input text (SOLO iOS!)
 */
Alert.prompt(
  'Enter Name',  // Titolo
  'What is your name?',  // Messaggio
  (text) => console.log('Name entered:', text),  // Callback con testo
  'plain-text',  // Tipo input: 'plain-text', 'secure-text', 'login-password'
  'Default Name',  // Testo default
  'default'  // Keyboard type
);

// ⚠️ Alert.prompt NON funziona su Android!
// Per input su Android, usa Modal + TextInput custom

/**
 * PATTERN: Confirm action
 */
const confirmDelete = () => {
  Alert.alert(
    'Confirm Delete',
    'This action cannot be undone',
    [
      { text: 'Cancel', style: 'cancel' },
      { 
        text: 'Delete', 
        style: 'destructive',
        onPress: async () => {
          await deleteItem();
          // Navigation, state update, etc.
        }
      },
    ]
  );
};

/**
 * PATTERN: Error alert
 */
const showError = (error: Error) => {
  Alert.alert(
    'Error',
    error.message || 'An unexpected error occurred',
    [{ text: 'OK' }]
  );
};

/**
 * PATTERN: Success confirmation
 */
const showSuccess = (message: string) => {
  Alert.alert(
    '✅ Success',  // Emoji per visual feedback
    message,
    [{ text: 'OK' }]
  );
};
```

### RefreshControl (Pull-to-Refresh)

**RefreshControl** aggiunge pull-to-refresh a ScrollView/FlatList.

```typescript
import { ScrollView, RefreshControl } from 'react-native';

const [refreshing, setRefreshing] = useState(false);

/**
 * Handler pull-to-refresh
 */
const onRefresh = async () => {
  setRefreshing(true);  // Mostra spinner
  
  try {
    await fetchData();  // Ricarica dati
    // Update state con nuovi dati
  } catch (error) {
    console.error('Refresh failed:', error);
  } finally {
    setRefreshing(false);  // Nascondi spinner
  }
};

<ScrollView
  /**
   * REFRESH CONTROL: Componente per pull-to-refresh
   */
  refreshControl={
    <RefreshControl 
      refreshing={refreshing}  // Stato loading
      onRefresh={onRefresh}    // Handler pull
      
      // COLORS: Colore spinner durante refresh
      colors={['#007AFF', '#5856D6']}  // Android: array colori (animazione)
      tintColor="#007AFF"  // iOS: singolo colore
      
      // TITLE: Testo durante refresh (solo iOS)
      title="Pull to refresh"  // iOS only
      titleColor="#666"  // iOS: colore title
      
      // PROGRESS: Background e view (solo Android)
      progressBackgroundColor="#fff"  // Android: background spinner
      progressViewOffset={0}  // Android: offset da top
    />
  }
>
  {/* Contenuto scrollabile */}
  {data.map(item => (
    <ItemCard key={item.id} {...item} />
  ))}
</ScrollView>

/**
 * USO CON FLATLIST:
 */
<FlatList
  data={data}
  renderItem={({ item }) => <ItemCard {...item} />}
  refreshControl={
    <RefreshControl 
      refreshing={refreshing}
      onRefresh={onRefresh}
      tintColor="#007AFF"
    />
  }
/>

/**
 * PATTERN: Fetch con refresh
 */
const useFetchWithRefresh = (fetchFn) => {
  const [data, setData] = useState([]);
  const [refreshing, setRefreshing] = useState(false);
  
  const refresh = async () => {
    setRefreshing(true);
    try {
      const newData = await fetchFn();
      setData(newData);
    } catch (error) {
      Alert.alert('Error', 'Failed to refresh data');
    } finally {
      setRefreshing(false);
    }
  };
  
  useEffect(() => {
    refresh();  // Initial load
  }, []);
  
  return { data, refreshing, refresh };
};

// Uso:
const { data, refreshing, refresh } = useFetchWithRefresh(fetchPosts);

<FlatList
  data={data}
  renderItem={({ item }) => <Post {...item} />}
  refreshControl={
    <RefreshControl refreshing={refreshing} onRefresh={refresh} />
  }
/>

---

## 9. StatusBar

**StatusBar** controlla la barra di stato del sistema (clock, battery, signal).

**📚 Teoria: StatusBar Rendering**
```
iOS StatusBar:
- Parte del sistema, NOT controllato dall'app
- Puoi solo cambiare STILE (light/dark)
- Non puoi cambiare background color (usa View dietro)
- Sempre visible (hidden è discouraged)

Android StatusBar:
- Più controllo dall'app
- Puoi cambiare background color
- Puoi renderla translucent (overlap con content)
- Puoi nasconderla completamente
```

```typescript
import { StatusBar } from 'react-native';

/**
 * COMPONENT APPROACH: Metti StatusBar in component tree
 */
const App = () => (
  <View style={{ flex: 1 }}>
    <StatusBar
      /**
       * BAR STYLE: Colore di icone/testo
       */
      barStyle="dark-content"  
      // 'default': Nero su iOS, varia su Android
      // 'dark-content': Icone/testo NERO (per background chiari)
      // 'light-content': Icone/testo BIANCO (per background scuri)
      // 
      // ⚠️ Regola d'oro:
      // Background chiaro → barStyle="dark-content"
      // Background scuro → barStyle="light-content"
      
      /**
       * BACKGROUND COLOR: Solo Android!
       */
      backgroundColor="#007AFF"  
      // Android: Colore background della status bar
      // iOS: IGNORATO (usa View dietro SafeAreaView)
      // Tip: Match con header/navigation bar color
      
      /**
       * HIDDEN: Nascondi status bar
       */
      hidden={false}  
      // true: Status bar nascosta
      // false: Visible (default)
      // ⚠️ iOS guidelines scoraggiano hidden=true
      
      /**
       * TRANSLUCENT: Solo Android!
       */
      translucent={false}  
      // false (default): Content inizia SOTTO status bar
      // true: Content può estendersi DIETRO status bar (overlap)
      //   Utile per: Splash screens, hero images, immersive content
      //   ⚠️ Richiede padding manual per evitare content coperto!
      
      /**
       * ANIMATED: Animazione per cambi (solo iOS)
       */
      animated={true}  
      // Transizioni smooth quando cambi barStyle/hidden
      
      /**
       * NETWORK ACTIVITY: Solo iOS
       */
      networkActivityIndicatorVisible={false}  
      // iOS: Mostra spinner network nella status bar
      // Deprecato iOS 13+, usa progress indicator custom
    />
    
    {/* App content */}
  </View>
);

/**
 * IMPERATIVE API: Cambia status bar da codice
 */

// Set bar style
StatusBar.setBarStyle('light-content', true);  
// Parametro 2: animated (solo iOS)

// Set background color (Android)
StatusBar.setBackgroundColor('#000', true);  
// Parametro 2: animated

// Set hidden
StatusBar.setHidden(true, 'fade');  
// Parametro 2: animation type
// iOS: 'fade', 'slide', 'none'
// Android: 'fade', 'slide'

// Set translucent (Android)
StatusBar.setTranslucent(true);

/**
 * PATTERN 1: StatusBar per Screen
 * Ogni screen configura la propria status bar
 */
const HomeScreen = () => (
  <View style={{ flex: 1, backgroundColor: '#fff' }}>
    <StatusBar barStyle="dark-content" backgroundColor="#fff" />
    {/* Content con background chiaro → icone scure */}
  </View>
);

const ProfileScreen = () => (
  <View style={{ flex: 1, backgroundColor: '#000' }}>
    <StatusBar barStyle="light-content" backgroundColor="#000" />
    {/* Content con background scuro → icone chiare */}
  </View>
);

/**
 * PATTERN 2: Dynamic status bar con scroll
 */
const ScrollScreen = () => {
  const [scrollY, setScrollY] = useState(0);
  
  // Cambia status bar quando scrolli oltre header
  const isDark = scrollY > 200;  // Dopo 200px di scroll
  
  return (
    <View style={{ flex: 1 }}>
      <StatusBar barStyle={isDark ? 'dark-content' : 'light-content'} />
      
      <ScrollView
        onScroll={(e) => setScrollY(e.nativeEvent.contentOffset.y)}
        scrollEventThrottle={16}
      >
        {/* Header con background scuro */}
        <View style={{ height: 300, backgroundColor: '#000' }} />
        {/* Content con background chiaro */}
        <View style={{ backgroundColor: '#fff', minHeight: 1000 }} />
      </ScrollView>
    </View>
  );
};

/**
 * PATTERN 3: Safe Area con status bar iOS
 */
import { SafeAreaView } from 'react-native';

const iOSScreen = () => (
  <SafeAreaView style={{ flex: 1, backgroundColor: '#007AFF' }}>
    {/* SafeAreaView background = status bar background su iOS */}
    <StatusBar barStyle="light-content" />  {/* Icone bianche */}
    
    <View style={{ flex: 1, backgroundColor: '#fff' }}>
      {/* Content bianco sotto status bar blu */}
    </View>
  </SafeAreaView>
);

/**
 * PATTERN 4: Fullscreen/Immersive (Android translucent)
 */
const SplashScreen = () => (
  <View style={{ flex: 1 }}>
    <StatusBar 
      translucent  // Content sotto status bar
      backgroundColor="transparent"  // Status bar trasparente
      barStyle="light-content"
    />
    
    <ImageBackground 
      source={splashImage} 
      style={{ flex: 1 }}  // Immagine fullscreen (include status bar area)
    >
      {/* Content */}
    </ImageBackground>
  </View>
);

/**
 * ⚠️ GOTCHA: Multiple StatusBar components
 */
// Se hai più <StatusBar /> nel tree, ULTIMO renderizzato vince
<View>
  <StatusBar barStyle="dark-content" />  {/* Ignorato */}
  <Screen1 />
  <StatusBar barStyle="light-content" />  {/* ← Questo viene usato */}
</View>

// Soluzione: Uno StatusBar per screen

---

## 10. Platform-Specific Code

**📚 Teoria: Perché Platform-Specific Code**
```
React Native è "write once, run anywhere" ma...
iOS e Android hanno:
- Design guidelines diversi (HIG vs Material Design)
- Componenti nativi diversi (UIKit vs Android Views)
- Comportamenti diversi (gestures, animations, keyboard)
- Capabilities diverse (3D Touch, Back button, etc.)

Platform module permette di:
1. Rilevare la piattaforma corrente
2. Usare valori diversi per piattaforma
3. Organizzare codice platform-specific
```

### Platform Module

```typescript
import { Platform } from 'react-native';

/**
 * PLATFORM.OS: Quale sistema operativo
 */
console.log(Platform.OS);  
// Valori possibili:
// 'ios': iPhone, iPad
// 'android': Dispositivi Android
// 'web': React Native Web
// 'windows': Windows UWP (rare)
// 'macos': macOS apps (React Native macOS)

/**
 * CHECK PLATFORM: Conditional logic
 */
if (Platform.OS === 'ios') {
  console.log('Running on iOS');
  // iOS-specific code
  // Esempio: Usa Haptic Feedback
} else if (Platform.OS === 'android') {
  console.log('Running on Android');
  // Android-specific code
  // Esempio: Usa Back button handler
}

/**
 * PLATFORM.VERSION: OS version
 */
console.log(Platform.Version);  
// iOS: String - "14.5", "15.0"
// Android: Number - 30 (Android 11), 31 (Android 12)

// iOS version check
if (Platform.OS === 'ios' && parseFloat(Platform.Version as string) >= 14) {
  console.log('iOS 14 or later');
  // Use features available in iOS 14+
}

// Android version check  
if (Platform.OS === 'android' && Platform.Version >= 30) {
  console.log('Android 11 or later (API 30+)');
  // Use scoped storage, new permissions model
}

/**
 * PLATFORM.SELECT: Valori diversi per piattaforma
 * ⭐ Il modo PIÙ ELEGANTE per platform-specific values
 */

// Esempio 1: Padding diverso
const styles = StyleSheet.create({
  container: {
    padding: Platform.select({
      ios: 20,      // iOS: 20px padding
      android: 10,  // Android: 10px padding
      default: 15,  // Altre piattaforme: 15px (optional)
    }),
  },
});

// Esempio 2: Font family diverso
const styles = StyleSheet.create({
  text: {
    fontFamily: Platform.select({
      ios: 'Helvetica Neue',
      android: 'Roboto',
      default: 'System',
    }),
  },
});

// Esempio 3: Shadow vs Elevation
const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 16,
    ...Platform.select({
      ios: {
        // iOS: Usa shadow properties
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 4,
      },
      android: {
        // Android: Usa elevation
        elevation: 5,
      },
      default: {},
    }),
  },
});

/**
 * PATTERN: Component props platform-specific
 */
const MyComponent = () => {
  const behavior = Platform.select({
    ios: 'padding',
    android: 'height',
  });
  
  return (
    <KeyboardAvoidingView behavior={behavior}>
      {/* Content */}
    </KeyboardAvoidingView>
  );
};

/**
 * PATTERN: Inline conditional
 */
<View style={{
  marginTop: Platform.OS === 'ios' ? 20 : 10,
  // Equivalente a Platform.select ma più conciso per singoli valori
}}>
  <Text>Content</Text>
</View>

/**
 * PATTERN: Complex platform logic
 */
const getHeaderHeight = () => {
  if (Platform.OS === 'ios') {
    // iOS: Considera notch
    const version = parseFloat(Platform.Version as string);
    if (version >= 11) {
      return 88;  // iPhone X+ con notch
    }
    return 64;  // iPhone vecchi
  }
  
  if (Platform.OS === 'android') {
    // Android: Standard Material Design
    return 56;
  }
  
  return 60;  // Default
};

const headerHeight = getHeaderHeight();

/**
 * PLATFORM.IS TV/PAD: Device type (iOS)
 */
console.log(Platform.isPad);  // true su iPad
console.log(Platform.isTVOS);  // true su Apple TV
// ⚠️ Solo iOS! Android returns undefined

// iPad-specific layout
if (Platform.isPad) {
  // Use multi-column layout
} else {
  // Use single column
}
```

### File Extensions (Platform-Specific Files)

**React Native automaticamente carica il file giusto per la piattaforma!**

```typescript
/**
 * STRUTTURA FILE:
 * 
 * components/
 *   Button.tsx           ← Fallback (tutte le piattaforme)
 *   Button.ios.tsx       ← Usato SOLO su iOS
 *   Button.android.tsx   ← Usato SOLO su Android
 *   Button.web.tsx       ← Usato SOLO su web
 */

// Nel codice, importa SENZA estensione piattaforma:
import Button from './components/Button';
// React Native sceglie automaticamente:
// - iOS: Button.ios.tsx (se esiste), altrimenti Button.tsx
// - Android: Button.android.tsx (se esiste), altrimenti Button.tsx
// - Web: Button.web.tsx (se esiste), altrimenti Button.tsx

/**
 * ESEMPIO: Button.ios.tsx
 */
// iOS-specific button con Haptic Feedback
import React from 'react';
import { TouchableOpacity, Text } from 'react-native';
import * as Haptics from 'expo-haptics';  // iOS-only feature

export default function Button({ title, onPress }) {
  const handlePress = () => {
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);  // Haptic!
    onPress();
  };
  
  return (
    <TouchableOpacity onPress={handlePress}>
      <Text>{title}</Text>
    </TouchableOpacity>
  );
}

/**
 * ESEMPIO: Button.android.tsx
 */
// Android-specific button con Ripple effect
import React from 'react';
import { Pressable, Text } from 'react-native';

export default function Button({ title, onPress }) {
  return (
    <Pressable
      onPress={onPress}
      android_ripple={{ color: 'rgba(0,0,0,0.1)' }}  // Android ripple
    >
      <Text>{title}</Text>
    </Pressable>
  );
}

/**
 * QUANDO USARE FILE EXTENSIONS:
 * 
 * ✅ Usa quando:
 * - Logica completamente diversa tra piattaforme
 * - Componenti nativi diversi (iOS UIKit vs Android Views)
 * - Features platform-specific (Haptics, Ripple, etc.)
 * - Codice lungo/complesso per piattaforma
 * 
 * ❌ Non usare quando:
 * - Solo piccole differenze (usa Platform.select)
 * - Stili diversi (usa Platform.select in StyleSheet)
 * - Props diversi (usa Platform.OS inline)
 * 
 * Regola: Se >50% del codice è platform-specific → usa file extensions
 *         Se <50% è diverso → usa Platform.select/OS check
 */
```

**💡 Best Practices:**

```typescript
// ✅ GOOD: Platform.select per stili
const styles = StyleSheet.create({
  container: {
    ...Platform.select({
      ios: { shadowOpacity: 0.3 },
      android: { elevation: 5 },
    }),
  },
});

// ❌ BAD: Troppe conditional inline
const styles = StyleSheet.create({
  container: {
    shadowColor: Platform.OS === 'ios' ? '#000' : undefined,
    shadowOpacity: Platform.OS === 'ios' ? 0.3 : undefined,
    shadowRadius: Platform.OS === 'ios' ? 4 : undefined,
    elevation: Platform.OS === 'android' ? 5 : undefined,
  },
});

// ✅ GOOD: Extract platform logic
const getPlatformStyles = () => {
  if (Platform.OS === 'ios') {
    return {
      shadowColor: '#000',
      shadowOpacity: 0.3,
      shadowRadius: 4,
    };
  }
  return { elevation: 5 };
};

const styles = StyleSheet.create({
  container: {
    ...getPlatformStyles(),
  },
});

// ✅ GOOD: Costanti platform-specific
const SPACING = Platform.select({
  ios: {
    small: 8,
    medium: 16,
    large: 24,
  },
  android: {
    small: 8,
    medium: 16,
    large: 24,
  },
});

const styles = StyleSheet.create({
  container: {
    padding: SPACING.medium,
  },
});
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Profile Card

Crea un componente `ProfileCard` che mostra:
- Avatar (Image)
- Nome (Text)
- Bio (Text multiline)
- Button "Follow"

### Esercizio 2: 🟡 Login Form

Implementa un form con:
- Email input (con keyboard type corretto)
- Password input (secure)
- Remember me (Switch)
- Login button
- Validazione base

### Esercizio 3: 🟡 Image Gallery

Crea una galleria con:
- ScrollView orizzontale
- 5+ immagini da URI
- Loading indicator per ogni immagine
- Tap per aprire Modal fullscreen

### Esercizio 4: 🔴 Custom Modal Dialog

Crea un sistema di dialog riutilizzabile:
- Overlay trasparente
- Animazione slide from bottom
- Titolo, messaggio, 2 bottoni
- Tap outside per chiudere
- TypeScript interfaces per props

### Esercizio 5: 🔴 Platform-Adaptive UI

Crea un componente che:
- Ha shadow su iOS, elevation su Android
- Font diversi per platform
- Button style nativo per platform
- Status bar adattiva al theme

---

## 📝 Quiz

1. **Qual è la differenza tra View e ScrollView?**
   - [x] ScrollView permette scroll, View no
   - [ ] View è deprecato
   - [ ] Non c'è differenza
   - [ ] ScrollView è solo per liste

2. **Come rendere un'immagine responsive?**
   - [ ] Usare width e height fissi
   - [x] Usare resizeMode e dimensioni relative
   - [ ] Non è possibile
   - [ ] Solo con librerie esterne

3. **Qual è il componente migliore per button custom?**
   - [ ] Button
   - [x] TouchableOpacity o Pressable
   - [ ] View con onPress
   - [ ] Text con onPress

4. **TextInput multiline crea:**
   - [ ] Multiple input fields
   - [x] Un textarea
   - [ ] Un array di stringhe
   - [ ] Errore

5. **Modal transparent={true}:**
   - [ ] Rende il modal invisibile
   - [x] Permette di vedere attraverso il background
   - [ ] È deprecato
   - [ ] Solo per iOS

**Risposte**: 1-a, 2-b, 3-b, 4-b, 5-b

---

## 🔗 Risorse

- [React Native Core Components](https://reactnative.dev/docs/components-and-apis)
- [Image Component Docs](https://reactnative.dev/docs/image)
- [TextInput Docs](https://reactnative.dev/docs/textinput)

---

## ✅ Checklist

- [ ] View e SafeAreaView utilizzate correttamente
- [ ] Text styling padroneggiato
- [ ] Image locali e remote caricate
- [ ] ScrollView implementato
- [ ] TextInput forms creati
- [ ] Touchable components utilizzati
- [ ] Modal implementato
- [ ] Platform differences gestite
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📦 **View** è il container base
2. 📝 **Text** per tutto il testo
3. 🖼️ **Image** con resizeMode
4. 📜 **ScrollView** per contenuto scrollabile (non liste lunghe!)
5. ⌨️ **TextInput** con keyboardType appropriato
6. 👆 **TouchableOpacity/Pressable** > Button
7. 🎭 **Modal** per overlay
8. 🔀 **Platform.select** per differenze iOS/Android

---

**Ottimo lavoro! 🎉** Ora conosci i componenti core. Prossimo: Styling e Layout!

[← Precedente: Fondamenti di React](../01%20-%20Fondamenti%20di%20React%20Native/04_Fondamenti_React.md) | [Torna all'Indice](../README.md) | [Prossimo: Styling e Layout →](06_Styling_Layout.md)
