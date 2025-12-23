# 07. Expo SDK - Camera, Location e Notifications

[← Precedente: Managed vs Bare Workflow](06_Managed_vs_Bare_Workflow.md) | [Torna all'Indice](README.md) | [Prossimo: Esercizi Pratici →](08_Esercizi_Quiz.md)

---

## Descrizione

L'**Expo SDK** offre oltre 50 moduli per accedere a funzionalità native dei dispositivi mobile senza scrivere codice nativo. In questo capitolo esploreremo 3 APIs fondamentali: **Camera** (foto/video), **Location** (GPS/geolocation) e **Notifications** (push/local). Impareremo pattern comuni (permissions, error handling) e implementeremo esempi pratici completi.

---

## 🎯 Obiettivi di Apprendimento

- [ ] Gestire permissions runtime (iOS/Android)
- [ ] Implementare Camera per foto e video
- [ ] Tracciare posizione utente con Location API
- [ ] Inviare notifiche locali e push
- [ ] Gestire errori e casi edge
- [ ] Comprendere differenze iOS vs Android

---

## 📚 Contenuti

1. [Pattern Comuni: Permissions](#1-pattern-comuni-permissions)
2. [Camera API - Foto e Video](#2-camera-api-foto-e-video)
3. [Location API - GPS e Geofencing](#3-location-api-gps-e-geofencing)
4. [Notifications - Local e Push](#4-notifications-local-e-push)
5. [Error Handling Best Practices](#5-error-handling-best-practices)
6. [Platform Differences](#6-platform-differences)

---

## 1. Pattern Comuni: Permissions

**📚 Teoria: Runtime Permissions**

```
iOS: User può negare permission → chiedi ogni volta
Android: User può negare + "Don't ask again" → user must go to settings

Flow:
1. Check status → (granted | denied | undetermined)
2. Se denied forever → Open settings
3. Se undetermined → Request permission
4. Handle result
```

### Template Reusabile

```typescript
/**
 * Hook per gestire permissions in modo reusabile
 * 
 * Uso:
 * const [status, requestPermission] = usePermission(Camera.useCameraPermissions);
 */
import { useState, useEffect } from 'react';
import { Alert, Linking, Platform } from 'react-native';

type PermissionResponse = {
  status: 'granted' | 'denied' | 'undetermined';
  canAskAgain: boolean; // false su Android se user ha scelto "Don't ask again"
  granted: boolean;
};

type PermissionHook = () => [PermissionResponse | null, () => Promise<PermissionResponse>];

export function usePermissionWithFallback(usePermissionHook: PermissionHook) {
  const [permission, requestPermission] = usePermissionHook();
  
  const requestWithFallback = async (): Promise<PermissionResponse> => {
    // Se già granted, return subito
    if (permission?.granted) {
      return permission;
    }
    
    // Se denied forever (canAskAgain = false), open settings
    if (permission && !permission.canAskAgain && !permission.granted) {
      Alert.alert(
        'Permission Denied',
        'Please enable permission in Settings',
        [
          { text: 'Cancel', style: 'cancel' },
          { 
            text: 'Open Settings', 
            onPress: () => {
              if (Platform.OS === 'ios') {
                Linking.openURL('app-settings:'); // iOS
              } else {
                Linking.openSettings(); // Android
              }
            } 
          },
        ]
      );
      throw new Error('Permission denied forever');
    }
    
    // Request permission
    const result = await requestPermission();
    
    if (!result.granted) {
      throw new Error('Permission denied by user');
    }
    
    return result;
  };
  
  return [permission, requestWithFallback] as const;
}
```

**Uso in componente:**

```typescript
import { Camera } from 'expo-camera';

function MyCameraComponent() {
  const [permission, requestPermission] = usePermissionWithFallback(
    Camera.useCameraPermissions
  );
  
  const handleTakePhoto = async () => {
    try {
      await requestPermission(); // Automatic fallback a settings se denied forever
      // Procedi con camera
    } catch (error) {
      console.log('Permission denied:', error);
    }
  };
  
  return <Button onPress={handleTakePhoto} title="Take Photo" />;
}
```

---

## 2. Camera API - Foto e Video

**📚 Teoria: expo-camera**

```
expo-camera offre:
✅ CameraView component (preview camera)
✅ takePictureAsync() - Scatta foto
✅ recordAsync() - Registra video
✅ Permissions per camera + microphone

Alternative:
- expo-image-picker: Scegli da gallery (più semplice, no preview)
- expo-camera: Full control su camera (flash, zoom, front/back)
```

### Installazione

```bash
$ npx expo install expo-camera
```

### Configurazione Permissions

```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone",
          "recordAudioAndroid": true
        }
      ]
    ]
  }
}
```

### Esempio Completo: Photo Capture

```typescript
/**
 * Componente per scattare foto con preview
 */
import React, { useState, useRef } from 'react';
import { View, Text, TouchableOpacity, Image, StyleSheet, Alert } from 'react-native';
import { CameraView, CameraType, useCameraPermissions } from 'expo-camera';
import * as MediaLibrary from 'expo-media-library';

export default function CameraScreen() {
  // State
  const [facing, setFacing] = useState<CameraType>('back'); // 'back' | 'front'
  const [photo, setPhoto] = useState<string | null>(null); // URI foto scattata
  const cameraRef = useRef<CameraView>(null); // Ref per accedere a camera methods
  
  // Permissions
  const [cameraPermission, requestCameraPermission] = useCameraPermissions();
  const [mediaLibraryPermission, requestMediaLibraryPermission] = 
    MediaLibrary.usePermissions();
  
  // 1. Check permissions
  if (!cameraPermission || !mediaLibraryPermission) {
    // Permissions still loading
    return <View style={styles.container}><Text>Loading...</Text></View>;
  }
  
  if (!cameraPermission.granted || !mediaLibraryPermission.granted) {
    // Permissions not granted
    return (
      <View style={styles.container}>
        <Text style={styles.message}>
          Camera and Media Library permissions required
        </Text>
        <TouchableOpacity 
          style={styles.button}
          onPress={() => {
            requestCameraPermission();
            requestMediaLibraryPermission();
          }}
        >
          <Text style={styles.buttonText}>Grant Permissions</Text>
        </TouchableOpacity>
      </View>
    );
  }
  
  // 2. Take photo
  const takePicture = async () => {
    if (!cameraRef.current) return;
    
    try {
      const photoData = await cameraRef.current.takePictureAsync({
        quality: 0.8, // 0-1, 1 = max quality
        base64: false, // true se vuoi base64 encoding
        exif: true,    // Include EXIF metadata (GPS, date, device)
      });
      
      console.log('Photo taken:', photoData.uri);
      setPhoto(photoData.uri); // Show preview
    } catch (error) {
      Alert.alert('Error', 'Failed to take photo');
      console.error(error);
    }
  };
  
  // 3. Save to gallery
  const saveToGallery = async () => {
    if (!photo) return;
    
    try {
      const asset = await MediaLibrary.createAssetAsync(photo);
      Alert.alert('Success', 'Photo saved to gallery!');
      console.log('Saved asset:', asset);
      setPhoto(null); // Reset
    } catch (error) {
      Alert.alert('Error', 'Failed to save photo');
      console.error(error);
    }
  };
  
  // 4. Toggle camera (front/back)
  const toggleCameraFacing = () => {
    setFacing(current => (current === 'back' ? 'front' : 'back'));
  };
  
  // UI: Photo preview
  if (photo) {
    return (
      <View style={styles.container}>
        <Image source={{ uri: photo }} style={styles.preview} />
        <View style={styles.buttonContainer}>
          <TouchableOpacity style={styles.button} onPress={saveToGallery}>
            <Text style={styles.buttonText}>Save</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.button} onPress={() => setPhoto(null)}>
            <Text style={styles.buttonText}>Retake</Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }
  
  // UI: Camera preview
  return (
    <View style={styles.container}>
      <CameraView 
        style={styles.camera} 
        facing={facing}
        ref={cameraRef}
      >
        <View style={styles.buttonContainer}>
          <TouchableOpacity style={styles.button} onPress={toggleCameraFacing}>
            <Text style={styles.buttonText}>Flip</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.captureButton} onPress={takePicture}>
            <View style={styles.captureButtonInner} />
          </TouchableOpacity>
        </View>
      </CameraView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#000',
  },
  message: {
    textAlign: 'center',
    paddingBottom: 10,
    color: '#fff',
  },
  camera: {
    flex: 1,
  },
  buttonContainer: {
    flex: 1,
    flexDirection: 'row',
    backgroundColor: 'transparent',
    margin: 64,
    justifyContent: 'space-around',
    alignItems: 'flex-end',
  },
  button: {
    alignSelf: 'flex-end',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 10,
  },
  buttonText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#000',
  },
  captureButton: {
    width: 70,
    height: 70,
    borderRadius: 35,
    backgroundColor: '#fff',
    alignSelf: 'flex-end',
    justifyContent: 'center',
    alignItems: 'center',
  },
  captureButtonInner: {
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: '#f00',
  },
  preview: {
    flex: 1,
  },
});
```

**🔑 Key Features:**
- ✅ Permissions check con UI fallback
- ✅ Camera preview con `CameraView`
- ✅ Toggle front/back camera
- ✅ Capture photo con quality config
- ✅ Save to gallery con `MediaLibrary`

### Video Recording

```typescript
/**
 * Registra video
 */
import { CameraView } from 'expo-camera';

function VideoRecorder() {
  const cameraRef = useRef<CameraView>(null);
  const [recording, setRecording] = useState(false);
  
  const startRecording = async () => {
    if (!cameraRef.current) return;
    
    try {
      setRecording(true);
      const videoData = await cameraRef.current.recordAsync({
        maxDuration: 60, // Max 60 secondi
        quality: '720p', // '480p' | '720p' | '1080p' | '4k'
        mute: false,     // false = registra audio
      });
      
      console.log('Video recorded:', videoData.uri);
      setRecording(false);
      
      // Save to gallery
      await MediaLibrary.createAssetAsync(videoData.uri);
    } catch (error) {
      console.error('Recording failed:', error);
      setRecording(false);
    }
  };
  
  const stopRecording = () => {
    if (cameraRef.current) {
      cameraRef.current.stopRecording(); // Stop recording
    }
  };
  
  return (
    <CameraView ref={cameraRef} style={{ flex: 1 }}>
      {recording ? (
        <TouchableOpacity onPress={stopRecording}>
          <Text>⏹ Stop</Text>
        </TouchableOpacity>
      ) : (
        <TouchableOpacity onPress={startRecording}>
          <Text>⏺ Record</Text>
        </TouchableOpacity>
      )}
    </CameraView>
  );
}
```

---

## 3. Location API - GPS e Geofencing

**📚 Teoria: expo-location**

```
expo-location offre:
✅ getCurrentPositionAsync() - Posizione single shot
✅ watchPositionAsync() - Tracking real-time
✅ startGeofencingAsync() - Notifica quando user entra/esce area
✅ Reverse geocoding (lat/lng → address)
✅ Forward geocoding (address → lat/lng)

Permissions:
- iOS: "When In Use" vs "Always" (background)
- Android: FINE vs COARSE, BACKGROUND
```

### Installazione

```bash
$ npx expo install expo-location
```

### Configurazione Permissions

```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow $(PRODUCT_NAME) to use your location.",
          "isIosBackgroundLocationEnabled": true,
          "isAndroidBackgroundLocationEnabled": true
        }
      ]
    ],
    "ios": {
      "infoPlist": {
        "NSLocationWhenInUseUsageDescription": "We need location to show nearby places",
        "NSLocationAlwaysUsageDescription": "We need background location for tracking"
      }
    },
    "android": {
      "permissions": [
        "ACCESS_FINE_LOCATION",
        "ACCESS_COARSE_LOCATION",
        "ACCESS_BACKGROUND_LOCATION"
      ]
    }
  }
}
```

### Esempio: Current Location

```typescript
/**
 * Get user current position (single shot)
 */
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, ActivityIndicator } from 'react-native';
import * as Location from 'expo-location';

export default function LocationScreen() {
  const [location, setLocation] = useState<Location.LocationObject | null>(null);
  const [address, setAddress] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  
  const getCurrentLocation = async () => {
    try {
      setLoading(true);
      
      // 1. Request permission
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        alert('Permission to access location was denied');
        return;
      }
      
      // 2. Get current position
      const position = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.Balanced, // Balanced, Low, High, BestForNavigation
        // maxAge: 10000, // Use cached location if < 10s old
        // timeout: 5000,  // Max wait time
      });
      
      console.log('Position:', position);
      setLocation(position);
      
      // 3. Reverse geocoding (lat/lng → address)
      const addresses = await Location.reverseGeocodeAsync({
        latitude: position.coords.latitude,
        longitude: position.coords.longitude,
      });
      
      if (addresses.length > 0) {
        const addr = addresses[0];
        const formatted = `${addr.street || ''}, ${addr.city || ''}, ${addr.region || ''}, ${addr.country || ''}`;
        setAddress(formatted);
      }
      
    } catch (error) {
      console.error('Location error:', error);
      alert('Failed to get location');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <View style={styles.container}>
      <TouchableOpacity style={styles.button} onPress={getCurrentLocation} disabled={loading}>
        <Text style={styles.buttonText}>Get Current Location</Text>
      </TouchableOpacity>
      
      {loading && <ActivityIndicator size="large" color="#007AFF" />}
      
      {location && (
        <View style={styles.infoContainer}>
          <Text style={styles.title}>📍 Coordinates:</Text>
          <Text>Latitude: {location.coords.latitude.toFixed(6)}</Text>
          <Text>Longitude: {location.coords.longitude.toFixed(6)}</Text>
          <Text>Accuracy: ±{location.coords.accuracy?.toFixed(0)}m</Text>
          <Text>Altitude: {location.coords.altitude?.toFixed(0)}m</Text>
          <Text>Speed: {location.coords.speed ? (location.coords.speed * 3.6).toFixed(1) : '0'} km/h</Text>
          
          {address && (
            <>
              <Text style={styles.title}>🏠 Address:</Text>
              <Text>{address}</Text>
            </>
          )}
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 20,
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  infoContainer: {
    marginTop: 20,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 10,
    marginBottom: 5,
  },
});
```

### Esempio: Real-time Tracking

```typescript
/**
 * Track user position continuously (watch)
 */
import { useEffect, useState } from 'react';
import * as Location from 'expo-location';

export default function TrackingScreen() {
  const [location, setLocation] = useState<Location.LocationObject | null>(null);
  const [subscription, setSubscription] = useState<Location.LocationSubscription | null>(null);
  
  useEffect(() => {
    startTracking();
    
    return () => {
      // Cleanup: stop tracking quando component unmounts
      subscription?.remove();
    };
  }, []);
  
  const startTracking = async () => {
    try {
      // Request permissions
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        alert('Permission denied');
        return;
      }
      
      // Start watching position
      const sub = await Location.watchPositionAsync(
        {
          accuracy: Location.Accuracy.High,
          timeInterval: 5000,    // Update ogni 5 secondi
          distanceInterval: 10,  // Update ogni 10 metri
        },
        (newLocation) => {
          console.log('New position:', newLocation.coords);
          setLocation(newLocation); // Update UI
        }
      );
      
      setSubscription(sub); // Save subscription per cleanup
      
    } catch (error) {
      console.error('Tracking error:', error);
    }
  };
  
  const stopTracking = () => {
    if (subscription) {
      subscription.remove();
      setSubscription(null);
      alert('Tracking stopped');
    }
  };
  
  return (
    <View style={{ flex: 1, padding: 20 }}>
      <Text>Tracking Status: {subscription ? '🟢 Active' : '🔴 Stopped'}</Text>
      
      {location && (
        <View>
          <Text>Lat: {location.coords.latitude.toFixed(6)}</Text>
          <Text>Lng: {location.coords.longitude.toFixed(6)}</Text>
          <Text>Speed: {((location.coords.speed || 0) * 3.6).toFixed(1)} km/h</Text>
        </View>
      )}
      
      <TouchableOpacity onPress={stopTracking}>
        <Text>Stop Tracking</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 4. Notifications - Local e Push

**📚 Teoria: expo-notifications**

```
expo-notifications offre:
✅ Local notifications (no server, schedule locale)
✅ Push notifications (da server, via Expo Push API)
✅ Notification handlers (foreground, background, tap)
✅ Badge count management
✅ Sound, vibration, priority

Flow Push Notifications:
1. User: Grant permission
2. App: Get Expo Push Token
3. App → Your Server: Send token
4. Your Server → Expo Push API: Send notification
5. Expo → APNs/FCM → User device: Delivery
```

### Installazione

```bash
$ npx expo install expo-notifications expo-device expo-constants
```

### Configurazione

```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff",
          "sounds": ["./assets/notification-sound.wav"]
        }
      ]
    ],
    "notification": {
      "icon": "./assets/notification-icon.png",
      "color": "#ffffff",
      "androidMode": "default",
      "androidCollapsedTitle": "#{unread_notifications} new notifications"
    }
  }
}
```

### Esempio: Local Notification

```typescript
/**
 * Schedule local notification (no server required)
 */
import React from 'react';
import { View, TouchableOpacity, Text, Platform } from 'react-native';
import * as Notifications from 'expo-notifications';

// Configure notification behavior quando app è in foreground
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,   // Show notification anche in foreground
    shouldPlaySound: true,   // Play sound
    shouldSetBadge: true,    // Update app badge
  }),
});

export default function LocalNotificationScreen() {
  
  // 1. Request permissions
  const requestPermissions = async () => {
    const { status } = await Notifications.requestPermissionsAsync();
    if (status !== 'granted') {
      alert('Permission denied');
      return false;
    }
    return true;
  };
  
  // 2. Schedule notification
  const scheduleNotification = async () => {
    const hasPermission = await requestPermissions();
    if (!hasPermission) return;
    
    await Notifications.scheduleNotificationAsync({
      content: {
        title: "You've got mail! 📬",
        body: 'Tap to read your new messages',
        data: { screen: 'Messages', messageId: 123 }, // Custom data
        sound: 'default', // o custom sound file
        badge: 1,         // iOS badge number
      },
      trigger: {
        seconds: 5, // Trigger tra 5 secondi
        // Oppure:
        // date: new Date(Date.now() + 10000), // Specific date
        // repeats: true, // Repeat notification
      },
    });
    
    alert('Notification scheduled for 5 seconds from now');
  };
  
  // 3. Schedule repeating notification
  const scheduleDailyNotification = async () => {
    const hasPermission = await requestPermissions();
    if (!hasPermission) return;
    
    await Notifications.scheduleNotificationAsync({
      content: {
        title: 'Daily Reminder ⏰',
        body: "Don't forget to complete your daily tasks!",
      },
      trigger: {
        hour: 9,      // 9:00 AM
        minute: 0,
        repeats: true, // Every day
      },
    });
    
    alert('Daily notification scheduled for 9:00 AM');
  };
  
  // 4. Cancel all notifications
  const cancelAllNotifications = async () => {
    await Notifications.cancelAllScheduledNotificationsAsync();
    alert('All notifications cancelled');
  };
  
  return (
    <View style={{ flex: 1, padding: 20, justifyContent: 'center' }}>
      <TouchableOpacity 
        style={{ backgroundColor: '#007AFF', padding: 15, borderRadius: 10, marginBottom: 10 }}
        onPress={scheduleNotification}
      >
        <Text style={{ color: '#fff', textAlign: 'center' }}>
          Schedule Notification (5s)
        </Text>
      </TouchableOpacity>
      
      <TouchableOpacity 
        style={{ backgroundColor: '#34C759', padding: 15, borderRadius: 10, marginBottom: 10 }}
        onPress={scheduleDailyNotification}
      >
        <Text style={{ color: '#fff', textAlign: 'center' }}>
          Schedule Daily (9 AM)
        </Text>
      </TouchableOpacity>
      
      <TouchableOpacity 
        style={{ backgroundColor: '#FF3B30', padding: 15, borderRadius: 10 }}
        onPress={cancelAllNotifications}
      >
        <Text style={{ color: '#fff', textAlign: 'center' }}>
          Cancel All
        </Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Esempio: Push Notifications

```typescript
/**
 * Setup push notifications con Expo Push Token
 */
import React, { useState, useEffect, useRef } from 'react';
import { Text, View, Platform } from 'react-native';
import * as Device from 'expo-device';
import * as Notifications from 'expo-notifications';
import Constants from 'expo-constants';

export default function PushNotificationScreen() {
  const [expoPushToken, setExpoPushToken] = useState<string | null>(null);
  const notificationListener = useRef<Notifications.Subscription>();
  const responseListener = useRef<Notifications.Subscription>();
  
  useEffect(() => {
    // 1. Get Expo Push Token
    registerForPushNotificationsAsync().then(token => {
      setExpoPushToken(token);
      // 2. Send token to your backend server
      sendTokenToServer(token);
    });
    
    // 3. Listener: notification received (app in foreground)
    notificationListener.current = Notifications.addNotificationReceivedListener(notification => {
      console.log('Notification received:', notification);
      // Update UI, show in-app notification, ecc.
    });
    
    // 4. Listener: notification tapped (app background → foreground)
    responseListener.current = Notifications.addNotificationResponseReceivedListener(response => {
      console.log('Notification tapped:', response);
      const data = response.notification.request.content.data;
      // Navigate to specific screen based on data
      if (data.screen === 'Messages') {
        // navigation.navigate('Messages', { messageId: data.messageId });
      }
    });
    
    return () => {
      // Cleanup
      if (notificationListener.current) {
        Notifications.removeNotificationSubscription(notificationListener.current);
      }
      if (responseListener.current) {
        Notifications.removeNotificationSubscription(responseListener.current);
      }
    };
  }, []);
  
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center', padding: 20 }}>
      <Text style={{ fontSize: 18, marginBottom: 10 }}>Push Notifications Setup</Text>
      <Text>Expo Push Token:</Text>
      <Text selectable style={{ fontSize: 12, color: '#666', marginTop: 10 }}>
        {expoPushToken || 'Loading...'}
      </Text>
      <Text style={{ marginTop: 20, color: '#999', textAlign: 'center' }}>
        Copy this token and use it to send push notifications via Expo Push API
      </Text>
    </View>
  );
}

// Helper: Get Expo Push Token
async function registerForPushNotificationsAsync(): Promise<string | undefined> {
  let token;
  
  // 1. Check if physical device (push non funziona su simulator)
  if (!Device.isDevice) {
    alert('Must use physical device for Push Notifications');
    return;
  }
  
  // 2. Request permissions
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;
  
  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }
  
  if (finalStatus !== 'granted') {
    alert('Failed to get push token for push notification!');
    return;
  }
  
  // 3. Get Expo Push Token
  token = (await Notifications.getExpoPushTokenAsync({
    projectId: Constants.expoConfig?.extra?.eas?.projectId, // Da app.json
  })).data;
  
  console.log('Expo Push Token:', token);
  
  // 4. Android: Set notification channel
  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#FF231F7C',
    });
  }
  
  return token;
}

// Helper: Send token to your backend
async function sendTokenToServer(token: string | undefined) {
  if (!token) return;
  
  try {
    await fetch('https://your-backend.com/api/push-tokens', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        token,
        userId: 'user123', // Your user ID
        platform: Platform.OS,
      }),
    });
    console.log('Token sent to server');
  } catch (error) {
    console.error('Failed to send token:', error);
  }
}
```

### Inviare Push da Server

```bash
# Expo Push API endpoint
POST https://exp.host/--/api/v2/push/send
Content-Type: application/json

{
  "to": "ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]",
  "sound": "default",
  "title": "Hello World",
  "body": "This is a push notification",
  "data": {
    "screen": "Details",
    "itemId": 123
  },
  "badge": 1
}
```

**Node.js example:**

```javascript
// Server-side (Node.js)
const { Expo } = require('expo-server-sdk');

const expo = new Expo();

async function sendPushNotification(expoPushToken, title, body, data) {
  const messages = [{
    to: expoPushToken,
    sound: 'default',
    title,
    body,
    data,
    badge: 1,
  }];
  
  const chunks = expo.chunkPushNotifications(messages);
  
  for (const chunk of chunks) {
    try {
      const ticketChunk = await expo.sendPushNotificationsAsync(chunk);
      console.log('Tickets:', ticketChunk);
    } catch (error) {
      console.error('Error sending push:', error);
    }
  }
}

// Usage:
sendPushNotification(
  'ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]',
  'New Message',
  'You have a new message from John',
  { screen: 'Chat', chatId: '456' }
);
```

---

## 5. Error Handling Best Practices

```typescript
/**
 * Pattern per error handling robusto
 */

// 1. Try-Catch con user-friendly messages
async function safeLocationAccess() {
  try {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== 'granted') {
      throw new Error('PERMISSION_DENIED');
    }
    
    const location = await Location.getCurrentPositionAsync();
    return { success: true, data: location };
    
  } catch (error: any) {
    if (error.message === 'PERMISSION_DENIED') {
      return { 
        success: false, 
        error: 'Location permission is required to use this feature' 
      };
    }
    
    if (error.code === 'E_LOCATION_SERVICES_DISABLED') {
      return { 
        success: false, 
        error: 'Please enable Location Services in Settings' 
      };
    }
    
    return { 
      success: false, 
      error: 'Failed to get location. Please try again.' 
    };
  }
}

// 2. Timeout per operazioni lente
async function safeLocationWithTimeout() {
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('TIMEOUT')), 10000) // 10s timeout
  );
  
  const locationPromise = Location.getCurrentPositionAsync();
  
  try {
    const location = await Promise.race([locationPromise, timeoutPromise]);
    return { success: true, data: location };
  } catch (error: any) {
    if (error.message === 'TIMEOUT') {
      return { success: false, error: 'Location request timed out' };
    }
    return { success: false, error: error.message };
  }
}
```

---

## 6. Platform Differences

| Feature | iOS | Android | Note |
|---------|-----|---------|------|
| **Camera Permission** | Single permission | CAMERA permission | iOS: user può negare sempre |
| **Location "Always"** | 2-step (When In Use → Always) | BACKGROUND_LOCATION permission | iOS: prompt twice |
| **Push Notifications** | Via APNs | Via FCM | Expo gestisce entrambi |
| **Notification Channels** | ❌ Non esiste | ✅ Required Android 8+ | Android: group notifications |
| **Badge Count** | ✅ Supported | ⚠️ Limitato (dipende da launcher) | iOS: native, Android: alcuni launcher |
| **Location Accuracy** | BestForNavigation, Best, Nearest10m, Hundred, Kilometer | HIGH, MEDIUM, LOW | iOS più granulare |

---

## ✅ Checklist di Completamento

- [ ] Ho implementato Camera con photo capture
- [ ] Ho implementato Location tracking
- [ ] Ho implementato Local Notifications
- [ ] Ho implementato Push Notifications
- [ ] Gestisco permissions correttamente
- [ ] Gestisco errori con user-friendly messages
- [ ] Comprendo differenze iOS vs Android

---

## 📌 Punti Chiave da Ricordare

1. 🔐 **Permissions**: Sempre check status → request → handle denial → fallback to settings
2. 📸 **Camera**: `CameraView` per preview + `takePictureAsync` / `recordAsync`
3. 📍 **Location**: `getCurrentPositionAsync` (single) vs `watchPositionAsync` (continuous)
4. 🔔 **Notifications**: Local (no server) vs Push (Expo Push Token + server)
5. 🛠️ **Error Handling**: Try-catch + timeout + user-friendly messages
6. 🍎🤖 **Platform Differences**: iOS APNs, Android FCM/Channels, badge count limitations

---

[Prossimo: 08. Esercizi e Quiz Pratici →](08_Esercizi_Quiz.md)

[← Precedente: Managed vs Bare Workflow](06_Managed_vs_Bare_Workflow.md) | [Torna all'Indice](README.md)
