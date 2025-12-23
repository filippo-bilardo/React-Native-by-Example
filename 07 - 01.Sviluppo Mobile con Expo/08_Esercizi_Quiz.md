# 08. Esercizi Pratici e Quiz - Expo Fondamenti

[← Precedente: Expo SDK APIs](07_Expo_SDK_APIs.md) | [Torna all'Indice](README.md)

---

## Descrizione

In questo capitolo troverai **esercizi pratici** di difficoltà crescente per consolidare le conoscenze acquisite sul modulo Expo Fondamenti. Ogni esercizio include obiettivi, requisiti e suggerimenti. Completa il modulo con un **quiz finale** per autovalutare la tua comprensione.

---

## 🎯 Obiettivi

- [ ] Applicare conoscenze teoriche in progetti pratici
- [ ] Integrare multiple Expo SDK APIs
- [ ] Risolvere problemi comuni di sviluppo mobile
- [ ] Autovalutare comprensione tramite quiz

---

## 📚 Contenuti

1. [🟢 Esercizi Base](#esercizi-base)
2. [🟡 Esercizi Intermedi](#esercizi-intermedi)
3. [🔴 Esercizio Avanzato](#esercizio-avanzato)
4. [📝 Quiz Finale](#quiz-finale)
5. [💡 Soluzioni Suggerite](#soluzioni-suggerite)

---

## 🟢 Esercizi Base

### Esercizio 1: Hello Expo World

**Obiettivo**: Creare primo progetto Expo con navigation basic.

**Requisiti:**
- [ ] Setup ambiente Expo (Node.js, Expo CLI, Expo Go)
- [ ] Crea progetto con `create-expo-app`
- [ ] Implementa 2 screens: Home e About
- [ ] Aggiungi React Navigation Stack
- [ ] Home → bottone naviga ad About
- [ ] About → mostra info app (nome, versione da app.json)

**Suggerimenti:**
```typescript
// 1. Create project
$ npx create-expo-app@latest HelloExpo --template blank-typescript

// 2. Install navigation
$ npx expo install @react-navigation/native @react-navigation/native-stack
$ npx expo install react-native-screens react-native-safe-area-context

// 3. Leggi info da app.json
import Constants from 'expo-constants';
const appName = Constants.expoConfig?.name;
const version = Constants.expoConfig?.version;
```

**Tempo stimato**: 30 minuti

---

### Esercizio 2: Image Picker App

**Obiettivo**: App per scegliere foto da gallery e mostrarla.

**Requisiti:**
- [ ] Usa `expo-image-picker` (più semplice di `expo-camera`)
- [ ] Bottone "Pick Image" apre gallery
- [ ] Mostra immagine selezionata in `<Image>`
- [ ] Bottone "Clear" rimuove immagine
- [ ] Gestisci permission denied con Alert

**Suggerimenti:**
```typescript
import * as ImagePicker from 'expo-image-picker';

const pickImage = async () => {
  // Request permission
  const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
  if (status !== 'granted') {
    alert('Permission denied');
    return;
  }
  
  // Launch picker
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    aspect: [4, 3],
    quality: 0.8,
  });
  
  if (!result.canceled) {
    setImage(result.assets[0].uri);
  }
};
```

**Tempo stimato**: 45 minuti

---

### Esercizio 3: Local Notification Reminder

**Obiettivo**: App reminder con notifiche locali.

**Requisiti:**
- [ ] Input text per messaggio reminder
- [ ] Bottone "Remind me in 10 seconds"
- [ ] Schedule local notification con `expo-notifications`
- [ ] Mostra notifica dopo 10 secondi
- [ ] Tapping notifica apre app e mostra messaggio

**Suggerimenti:**
```typescript
import * as Notifications from 'expo-notifications';

// Configure handler
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

// Schedule
await Notifications.scheduleNotificationAsync({
  content: {
    title: 'Reminder',
    body: message,
    data: { message }, // Pass data
  },
  trigger: { seconds: 10 },
});
```

**Tempo stimato**: 1 ora

---

## 🟡 Esercizi Intermedi

### Esercizio 4: Weather App con Location

**Obiettivo**: App meteo che usa posizione utente e API esterna.

**Requisiti:**
- [ ] Get current location con `expo-location`
- [ ] Fetch weather da OpenWeatherMap API (free tier)
- [ ] Mostra: temperatura, condizioni, città, icona meteo
- [ ] Loading state mentre fetch
- [ ] Error handling per permission denied / API error

**API Setup:**
```typescript
// 1. Sign up: https://openweathermap.org/api
// 2. Get free API key
// 3. Endpoint:
const API_KEY = 'YOUR_API_KEY';
const url = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`;
```

**Suggerimenti:**
- Usa `useState` per location, weather, loading, error
- Usa `useEffect` per get location on mount
- Gestisci states: loading → success / error

**Tempo stimato**: 2 ore

---

### Esercizio 5: Photo Gallery App

**Obiettivo**: App galleria con camera e storage locale.

**Requisiti:**
- [ ] Screen "Gallery": mostra lista foto (FlatList)
- [ ] FAB button "+" apre camera
- [ ] Scatta foto e salva in AsyncStorage
- [ ] Ogni foto: timestamp, URI (base64 o file system)
- [ ] Tap foto → full screen preview
- [ ] Long press → delete foto (con conferma)

**Suggerimenti:**
```typescript
// Storage
import AsyncStorage from '@react-native-async-storage/async-storage';

// Save photos array
const photos = [{ id: '1', uri: 'file://...', timestamp: Date.now() }];
await AsyncStorage.setItem('photos', JSON.stringify(photos));

// Load on mount
const stored = await AsyncStorage.getItem('photos');
const photos = stored ? JSON.parse(stored) : [];
```

**Tempo stimato**: 3 ore

---

### Esercizio 6: Fitness Tracker con Geolocation

**Obiettivo**: App tracking corsa con distanza e mappa.

**Requisiti:**
- [ ] Bottone "Start Run" inizia tracking location
- [ ] Usa `watchPositionAsync` per track real-time
- [ ] Calcola distanza percorsa (formula Haversine)
- [ ] Mostra: distanza (km), tempo (mm:ss), velocità media (km/h)
- [ ] Bottone "Stop" ferma tracking e salva sessione
- [ ] Screen "History": lista sessioni passate

**Formula Distanza (Haversine):**
```typescript
function getDistance(lat1: number, lon1: number, lat2: number, lon2: number): number {
  const R = 6371; // Earth radius in km
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLon = (lon2 - lon1) * Math.PI / 180;
  const a = 
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c; // Distance in km
}
```

**Tempo stimato**: 4 ore

---

## 🔴 Esercizio Avanzato

### Esercizio 7: Social Photo Sharing App (Progetto Completo)

**Obiettivo**: App social per condividere foto con notifiche push.

**Requisiti Completi:**

**1. Authentication (fake, no backend):**
- [ ] Login screen con username (AsyncStorage)
- [ ] Logout button

**2. Feed Screen:**
- [ ] FlatList di post (foto + username + caption + timestamp)
- [ ] Pull-to-refresh
- [ ] FAB "+" per creare post

**3. Create Post:**
- [ ] Scatta foto con camera
- [ ] Input caption
- [ ] Bottone "Post" salva in AsyncStorage
- [ ] Return to feed

**4. Profile Screen:**
- [ ] Mostra username
- [ ] Grid delle proprie foto
- [ ] Tap foto → full screen

**5. Push Notifications:**
- [ ] Quando user crea post → send push a tutti (simulate: schedule local notification per altri "user")
- [ ] Notification: "New post from [username]"
- [ ] Tap notifica → apre Feed

**6. Navigation:**
- [ ] Tab Navigator: Feed | Profile
- [ ] Stack Navigator per Create Post, Photo Detail

**Architettura Dati (AsyncStorage):**
```typescript
interface User {
  id: string;
  username: string;
}

interface Post {
  id: string;
  userId: string;
  username: string;
  photoUri: string;
  caption: string;
  timestamp: number;
}

// Keys:
// '@current_user' → User
// '@posts' → Post[]
```

**Suggerimenti:**
- Usa Context API per user state globale
- Usa `useReducer` per posts management
- Implementa pull-to-refresh con `RefreshControl`

**Tempo stimato**: 8-10 ore (progetto completo)

**Bonus:**
- [ ] Like feature (heart icon, count likes)
- [ ] Comments (tap post → comments screen)
- [ ] Dark mode toggle
- [ ] Offline support (detect network con `expo-network`)

---

## 📝 Quiz Finale

### Domande Teoriche

**1. Cos'è Expo?**
- [ ] A) Un framework JavaScript per web
- [ ] B) Un set di tools e services per React Native
- [ ] C) Un linguaggio di programmazione
- [ ] D) Un database mobile

<details>
<summary>Risposta</summary>
✅ B - Expo è un framework, platform e ecosystem per React Native che semplifica sviluppo, build e deploy di app mobile.
</details>

---

**2. Qual è la principale differenza tra Managed e Bare workflow?**
- [ ] A) Managed è gratuito, Bare è a pagamento
- [ ] B) Managed non ha ios/android folders, Bare sì
- [ ] C) Managed funziona solo su iOS
- [ ] D) Non c'è differenza

<details>
<summary>Risposta</summary>
✅ B - Managed workflow non ha native folders (generate on build), Bare workflow ha ios/ e android/ folders nel progetto per full control.
</details>

---

**3. Quando serve un Development Build invece di Expo Go?**
- [ ] A) Mai, Expo Go è sempre sufficiente
- [ ] B) Solo per app production
- [ ] C) Quando serve un native module non incluso in Expo Go
- [ ] D) Solo su Android

<details>
<summary>Risposta</summary>
✅ C - Development Build è necessario quando usi native modules non presenti in Expo Go (es. react-native-ble-plx per Bluetooth).
</details>

---

**4. Cosa fa il comando `expo prebuild`?**
- [ ] A) Build app production
- [ ] B) Genera ios/ e android/ folders
- [ ] C) Installa dependencies
- [ ] D) Run app su emulator

<details>
<summary>Risposta</summary>
✅ B - `expo prebuild` genera native folders (ios/, android/) da app.json configuration e Config Plugins.
</details>

---

**5. Come funzionano i Config Plugins?**
- [ ] A) Modificano codice JavaScript
- [ ] B) Modificano native configuration automaticamente
- [ ] C) Installano dependencies
- [ ] D) Fanno build più veloci

<details>
<summary>Risposta</summary>
✅ B - Config Plugins sono script JavaScript che modificano native configuration (Info.plist, AndroidManifest.xml) in modo declarativo.
</details>

---

**6. Qual è il vantaggio principale di EAS Build?**
- [ ] A) Build iOS senza Mac
- [ ] B) Build gratuiti illimitati
- [ ] C) Build più veloci
- [ ] D) Non servono permissions

<details>
<summary>Risposta</summary>
✅ A - EAS Build permette di buildare app iOS nel cloud senza possedere un Mac (e gestisce certificates automaticamente).
</details>

---

**7. Quale permission è necessaria per `expo-camera`?**
- [ ] A) Solo CAMERA
- [ ] B) CAMERA e MICROPHONE (per video)
- [ ] C) CAMERA, MICROPHONE e STORAGE
- [ ] D) Nessuna permission

<details>
<summary>Risposta</summary>
✅ B - `expo-camera` richiede CAMERA permission (foto) e MICROPHONE permission (per registrare video con audio).
</details>

---

**8. Differenza tra `getCurrentPositionAsync` e `watchPositionAsync`?**
- [ ] A) Nessuna differenza
- [ ] B) getCurrentPosition è più accurato
- [ ] C) getCurrentPosition single shot, watchPosition continuous tracking
- [ ] D) watchPosition funziona solo su Android

<details>
<summary>Risposta</summary>
✅ C - `getCurrentPositionAsync` get posizione una volta, `watchPositionAsync` continua a tracciare posizione real-time.
</details>

---

**9. Come si inviano Push Notifications con Expo?**
- [ ] A) Directly from app
- [ ] B) Via Expo Push API da server
- [ ] C) Solo con Firebase
- [ ] D) Non è possibile

<details>
<summary>Risposta</summary>
✅ B - Push notifications richiedono: 1) Get Expo Push Token in app, 2) Invia token a tuo server, 3) Server invia push via Expo Push API.
</details>

---

**10. Cosa sono le Local Notifications?**
- [ ] A) Notifiche che funzionano solo su localhost
- [ ] B) Notifiche schedulate localmente senza server
- [ ] C) Notifiche solo per development
- [ ] D) Notifiche che non richiedono permissions

<details>
<summary>Risposta</summary>
✅ B - Local Notifications sono schedulate direttamente nell'app senza server (es. reminders, alarms). Push Notifications richiedono server.
</details>

---

### Domande Pratiche (Codice)

**11. Quale codice request permission per Camera?**

A)
```typescript
import { Camera } from 'expo-camera';
Camera.getPermissions();
```

B)
```typescript
import { Camera } from 'expo-camera';
const { status } = await Camera.requestCameraPermissionsAsync();
```

C)
```typescript
import * as Permissions from 'expo-permissions';
await Permissions.askAsync(Permissions.CAMERA);
```

<details>
<summary>Risposta</summary>
✅ B - Metodo corretto in Expo SDK 50+: `Camera.requestCameraPermissionsAsync()`.  
Opzione C è deprecata (vecchia API expo-permissions).
</details>

---

**12. Come schedule una notifica tra 1 ora?**

A)
```typescript
await Notifications.scheduleNotificationAsync({
  content: { title: 'Reminder' },
  trigger: { seconds: 3600 },
});
```

B)
```typescript
await Notifications.scheduleNotificationAsync({
  content: { title: 'Reminder' },
  trigger: { hours: 1 },
});
```

C)
```typescript
setTimeout(() => {
  Notifications.presentNotificationAsync({ title: 'Reminder' });
}, 3600000);
```

<details>
<summary>Risposta</summary>
✅ A - Usa `trigger: { seconds: 3600 }` (1 ora = 3600 secondi).  
Opzione B non esiste (`hours` non è valido).  
Opzione C con setTimeout NON funziona (app potrebbe essere chiusa).
</details>

---

**13. Come get current location?**

A)
```typescript
const location = await Location.getCurrentPosition();
```

B)
```typescript
const location = await Location.getCurrentPositionAsync();
```

C)
```typescript
const location = Location.getLocation();
```

<details>
<summary>Risposta</summary>
✅ B - Metodo corretto: `Location.getCurrentPositionAsync()` (async suffix!).
</details>

---

**14. Come navigare a uno screen con React Navigation?**

A)
```typescript
navigation.push('Details', { itemId: 42 });
```

B)
```typescript
navigation.navigate('Details', { itemId: 42 });
```

C)
```typescript
navigation.goTo('Details', { itemId: 42 });
```

<details>
<summary>Risposta</summary>
✅ B - `navigation.navigate('ScreenName', params)` è il metodo standard.  
`push` esiste ma aggiunge sempre nuova screen (no reuse).  
`goTo` non esiste.
</details>

---

**15. Come aprire Settings app se permission denied forever?**

A)
```typescript
import { Settings } from 'react-native';
Settings.open();
```

B)
```typescript
import { Linking } from 'react-native';
await Linking.openSettings();
```

C)
```typescript
import * as Application from 'expo-application';
Application.openSettings();
```

<details>
<summary>Risposta</summary>
✅ B - `Linking.openSettings()` apre Settings app (Android).  
iOS: `Linking.openURL('app-settings:')`.
</details>

---

### Punteggio Quiz

**Calcola il tuo punteggio:**
- 13-15 corrette: 🏆 **Esperto Expo** - Ottima padronanza!
- 10-12 corrette: ⭐ **Competente** - Buona comprensione, ripassa alcuni concetti
- 7-9 corrette: 📚 **In Apprendimento** - Ripassa capitoli e rifai esercizi
- <7 corrette: 🔄 **Riprova** - Rileggi modulo e pratica con esercizi base

---

## 💡 Soluzioni Suggerite

### Soluzione Esercizio 2: Image Picker App

<details>
<summary>Mostra Soluzione</summary>

```typescript
import React, { useState } from 'react';
import { View, Image, TouchableOpacity, Text, StyleSheet, Alert } from 'react-native';
import * as ImagePicker from 'expo-image-picker';

export default function ImagePickerApp() {
  const [imageUri, setImageUri] = useState<string | null>(null);
  
  const pickImage = async () => {
    try {
      // Request permission
      const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
      if (status !== 'granted') {
        Alert.alert('Permission Denied', 'Camera roll permission is required');
        return;
      }
      
      // Launch picker
      const result = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: ImagePicker.MediaTypeOptions.Images,
        allowsEditing: true,
        aspect: [4, 3],
        quality: 0.8,
      });
      
      if (!result.canceled) {
        setImageUri(result.assets[0].uri);
      }
    } catch (error) {
      console.error('Error picking image:', error);
      Alert.alert('Error', 'Failed to pick image');
    }
  };
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Image Picker App</Text>
      
      {imageUri ? (
        <Image source={{ uri: imageUri }} style={styles.image} />
      ) : (
        <View style={styles.placeholder}>
          <Text>No image selected</Text>
        </View>
      )}
      
      <TouchableOpacity style={styles.button} onPress={pickImage}>
        <Text style={styles.buttonText}>Pick Image</Text>
      </TouchableOpacity>
      
      {imageUri && (
        <TouchableOpacity 
          style={[styles.button, styles.clearButton]} 
          onPress={() => setImageUri(null)}
        >
          <Text style={styles.buttonText}>Clear</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  image: {
    width: 300,
    height: 300,
    borderRadius: 10,
    marginBottom: 20,
  },
  placeholder: {
    width: 300,
    height: 300,
    borderRadius: 10,
    backgroundColor: '#f0f0f0',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    width: 200,
    alignItems: 'center',
    marginBottom: 10,
  },
  clearButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

</details>

---

### Soluzione Esercizio 4: Weather App

<details>
<summary>Mostra Soluzione</summary>

```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, ActivityIndicator, Image } from 'react-native';
import * as Location from 'expo-location';

const API_KEY = 'YOUR_OPENWEATHERMAP_API_KEY'; // Get from openweathermap.org

interface WeatherData {
  name: string;
  main: {
    temp: number;
    feels_like: number;
    humidity: number;
  };
  weather: Array<{
    main: string;
    description: string;
    icon: string;
  }>;
}

export default function WeatherApp() {
  const [weather, setWeather] = useState<WeatherData | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    getWeather();
  }, []);
  
  const getWeather = async () => {
    try {
      // 1. Get location permission
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        setError('Location permission denied');
        setLoading(false);
        return;
      }
      
      // 2. Get current position
      const location = await Location.getCurrentPositionAsync();
      const { latitude, longitude } = location.coords;
      
      // 3. Fetch weather
      const url = `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${API_KEY}&units=metric`;
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error('Weather API error');
      }
      
      const data: WeatherData = await response.json();
      setWeather(data);
      setLoading(false);
      
    } catch (err: any) {
      console.error('Error:', err);
      setError(err.message || 'Failed to get weather');
      setLoading(false);
    }
  };
  
  if (loading) {
    return (
      <View style={styles.container}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Getting weather...</Text>
      </View>
    );
  }
  
  if (error) {
    return (
      <View style={styles.container}>
        <Text style={styles.errorText}>❌ {error}</Text>
      </View>
    );
  }
  
  if (!weather) return null;
  
  const iconUrl = `https://openweathermap.org/img/wn/${weather.weather[0].icon}@4x.png`;
  
  return (
    <View style={styles.container}>
      <Text style={styles.city}>{weather.name}</Text>
      <Image source={{ uri: iconUrl }} style={styles.icon} />
      <Text style={styles.temp}>{Math.round(weather.main.temp)}°C</Text>
      <Text style={styles.description}>{weather.weather[0].description}</Text>
      <View style={styles.details}>
        <Text>Feels like: {Math.round(weather.main.feels_like)}°C</Text>
        <Text>Humidity: {weather.main.humidity}%</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#4A90E2',
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  loadingText: {
    marginTop: 10,
    color: '#fff',
    fontSize: 16,
  },
  errorText: {
    color: '#fff',
    fontSize: 18,
    textAlign: 'center',
  },
  city: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#fff',
    marginBottom: 20,
  },
  icon: {
    width: 150,
    height: 150,
  },
  temp: {
    fontSize: 64,
    fontWeight: 'bold',
    color: '#fff',
  },
  description: {
    fontSize: 24,
    color: '#fff',
    textTransform: 'capitalize',
    marginTop: 10,
  },
  details: {
    marginTop: 30,
    alignItems: 'center',
  },
});
```

</details>

---

## ✅ Checklist Finale Modulo

Completa questa checklist per verificare di aver coperto tutti gli argomenti:

### Teoria
- [ ] Comprendo panorama mobile (Native, Hybrid, Cross-platform)
- [ ] Conosco architettura Expo (SDK, EAS, Expo Go)
- [ ] So configurare ambiente di sviluppo
- [ ] Capisco differenze Managed vs Bare workflow
- [ ] So quando usare Expo Go vs Development Builds
- [ ] Comprendo Config Plugins

### Pratica
- [ ] Ho creato almeno un progetto Expo
- [ ] Ho usato Camera o ImagePicker API
- [ ] Ho implementato Location tracking
- [ ] Ho schedulato notifiche locali
- [ ] Ho gestito permissions correttamente
- [ ] Ho fatto build con EAS (opzionale ma consigliato)

### Esercizi
- [ ] Completato almeno 3 esercizi base (🟢)
- [ ] Completato almeno 1 esercizio intermedio (🟡)
- [ ] (Opzionale) Completato esercizio avanzato (🔴)
- [ ] Quiz: punteggio ≥ 10/15

---

## 🎓 Prossimi Passi

Hai completato **Expo Fondamenti**! Continua con:

1. **[Capitolo 01ter - Expo EAS Advanced]** (se disponibile)
   - EAS Build avanzato (profiles, credentials)
   - EAS Update deployment strategies
   - Config Plugins custom development
   - Monorepo setup
   - CI/CD integration

2. **[Capitolo 02 - Ambiente di Sviluppo React Native]**
   - React Native CLI setup (alternativa a Expo)
   - Native modules linking
   - Xcode e Android Studio deep dive

3. **Progetti Personali**
   - Crea app portfolio usando Expo
   - Pubblica su Expo Snack per condividere
   - Contribuisci a progetti open-source Expo

---

## 📚 Risorse Aggiuntive

- 📖 [Expo Documentation](https://docs.expo.dev)
- 🎮 [Expo Snack](https://snack.expo.dev) - Online playground
- 💬 [Expo Discord](https://chat.expo.dev) - Community support
- 🐙 [Expo GitHub](https://github.com/expo/expo) - Source code
- 📺 [Expo YouTube Channel](https://www.youtube.com/c/expo) - Tutorials

---

**Congratulazioni! 🎉 Hai completato il modulo Expo Fondamenti!**

[← Precedente: Expo SDK APIs](07_Expo_SDK_APIs.md) | [Torna all'Indice](README.md)
