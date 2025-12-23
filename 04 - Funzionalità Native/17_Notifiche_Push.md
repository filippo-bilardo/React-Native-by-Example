# Capitolo 17: Notifiche Push

[← Precedente: Geolocalizzazione e Mappe](16_Geolocalizzazione_Mappe.md) | [Torna all'Indice](../README.md) | [Prossimo: Moduli Nativi e Bridging →](18_Moduli_Nativi.md)

---

## Descrizione

Le notifiche push sono essenziali per mantenere gli utenti coinvolti: messaggi, aggiornamenti, promemoria, promozioni. React Native offre diverse soluzioni: React Native Push Notification per locale, Firebase Cloud Messaging (FCM) per remote, e librerie come Notifee per controllo avanzato.

In questo capitolo imparerai a implementare notifiche locali, integrare Firebase FCM, gestire notifiche in foreground/background, implementare deep linking, personalizzare aspetto notifiche, e ottimizzare delivery.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Implementare notifiche locali
- [ ] Integrare Firebase Cloud Messaging
- [ ] Gestire notifiche in foreground/background
- [ ] Implementare deep linking da notifiche
- [ ] Schedulare notifiche
- [ ] Personalizzare aspetto notifiche
- [ ] Gestire badges e sound
- [ ] Implementare notifiche ricche (immagini, actions)
- [ ] Gestire permission notifiche
- [ ] Ottimizzare delivery e targeting

## 📚 Argomenti Teorici

1. [Local Notifications](#1-local-notifications)
2. [Firebase Setup](#2-firebase-setup)
3. [Remote Notifications](#3-remote-notifications)
4. [Foreground Handling](#4-foreground)
5. [Background Handling](#5-background)
6. [Deep Linking](#6-deep-linking)
7. [Scheduled Notifications](#7-scheduled)
8. [Rich Notifications](#8-rich)
9. [Permissions](#9-permissions)
10. [Best Practices](#10-best-practices)

---

## 1. Local Notifications

### Installation

```bash
npm install react-native-push-notification
npm install @react-native-community/push-notification-ios
```

### iOS Setup

```bash
cd ios && pod install
```

```objective-c
// ios/AppDelegate.mm
#import <UserNotifications/UserNotifications.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
  center.delegate = self;
  
  return YES;
}

// Add these methods
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
       willPresentNotification:(UNNotification *)notification
         withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
{
  completionHandler(UNNotificationPresentationOptionSound | UNNotificationPresentationOptionAlert | UNNotificationPresentationOptionBadge);
}
```

### Android Setup

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

<application>
  <meta-data
    android:name="com.dieam.reactnativepushnotification.notification_foreground"
    android:value="true"/>
</application>
```

### Basic Local Notification

```typescript
import PushNotification from 'react-native-push-notification';

// Configure
PushNotification.configure({
  onNotification: function (notification) {
    console.log('Notification:', notification);
  },
  permissions: {
    alert: true,
    badge: true,
    sound: true
  },
  popInitialNotification: true,
  requestPermissions: true
});

// Create channel (Android)
PushNotification.createChannel(
  {
    channelId: 'default-channel',
    channelName: 'Default Channel',
    channelDescription: 'A default channel',
    soundName: 'default',
    importance: 4,
    vibrate: true
  },
  (created) => console.log(`Channel created: ${created}`)
);

// Send local notification
const sendLocalNotification = () => {
  PushNotification.localNotification({
    channelId: 'default-channel',
    title: 'Hello!',
    message: 'This is a local notification',
    playSound: true,
    soundName: 'default',
    vibrate: true,
    vibration: 300
  });
};

// Component
const NotificationScreen = () => {
  return (
    <View>
      <Button title="Send Notification" onPress={sendLocalNotification} />
    </View>
  );
};
```

### Notification with Actions

```typescript
const sendNotificationWithActions = () => {
  PushNotification.localNotification({
    channelId: 'default-channel',
    title: 'New Message',
    message: 'You have a new message from John',
    actions: ['Reply', 'Dismiss'],
    data: {
      userId: '123',
      messageId: '456'
    }
  });
};

// Handle action
PushNotification.configure({
  onAction: function (notification) {
    console.log('Action:', notification.action);
    
    if (notification.action === 'Reply') {
      // Open reply screen
      navigation.navigate('Reply', {
        messageId: notification.data.messageId
      });
    }
  }
});
```

---

## 2. Firebase Setup

### Installation

```bash
npm install @react-native-firebase/app
npm install @react-native-firebase/messaging
```

### iOS Setup

1. Create Firebase project at [console.firebase.google.com](https://console.firebase.google.com)
2. Download `GoogleService-Info.plist`
3. Add to `ios/` folder
4. Install pods:

```bash
cd ios && pod install
```

5. Enable Push Notifications capability in Xcode
6. Upload APNs certificate to Firebase Console

### Android Setup

1. Download `google-services.json`
2. Place in `android/app/`
3. Edit `android/build.gradle`:

```gradle
buildscript {
  dependencies {
    classpath 'com.google.gms:google-services:4.3.15'
  }
}
```

4. Edit `android/app/build.gradle`:

```gradle
apply plugin: 'com.google.gms.google-services'
```

### Request Permission

```typescript
import messaging from '@react-native-firebase/messaging';

const requestUserPermission = async () => {
  const authStatus = await messaging().requestPermission();
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL;
  
  if (enabled) {
    console.log('Authorization status:', authStatus);
    return true;
  }
  
  return false;
};

// Get FCM token
const getFCMToken = async () => {
  const token = await messaging().getToken();
  console.log('FCM Token:', token);
  // Send token to your server
  await sendTokenToServer(token);
};

// Component
const App = () => {
  useEffect(() => {
    requestUserPermission().then((enabled) => {
      if (enabled) {
        getFCMToken();
      }
    });
  }, []);
  
  return <Navigation />;
};
```

---

## 3. Remote Notifications

### Listen for Messages

```typescript
import messaging from '@react-native-firebase/messaging';

useEffect(() => {
  // Foreground messages
  const unsubscribe = messaging().onMessage(async (remoteMessage) => {
    console.log('Foreground notification:', remoteMessage);
    
    // Show local notification
    PushNotification.localNotification({
      channelId: 'default-channel',
      title: remoteMessage.notification?.title,
      message: remoteMessage.notification?.body,
      data: remoteMessage.data
    });
  });
  
  return unsubscribe;
}, []);
```

### Send Notification from Server

```typescript
// Node.js example
const admin = require('firebase-admin');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

const sendPushNotification = async (token: string, title: string, body: string) => {
  const message = {
    notification: {
      title: title,
      body: body
    },
    data: {
      type: 'message',
      userId: '123'
    },
    token: token
  };
  
  try {
    const response = await admin.messaging().send(message);
    console.log('Successfully sent message:', response);
  } catch (error) {
    console.error('Error sending message:', error);
  }
};
```

### Topic Subscription

```typescript
const subscribeToTopic = async (topic: string) => {
  await messaging().subscribeToTopic(topic);
  console.log(`Subscribed to ${topic}`);
};

const unsubscribeFromTopic = async (topic: string) => {
  await messaging().unsubscribeFromTopic(topic);
  console.log(`Unsubscribed from ${topic}`);
};

// Uso
const SettingsScreen = () => {
  const handleToggleNotifications = async (enabled: boolean) => {
    if (enabled) {
      await subscribeToTopic('news');
      await subscribeToTopic('updates');
    } else {
      await unsubscribeFromTopic('news');
      await unsubscribeFromTopic('updates');
    }
  };
  
  return (
    <Switch
      value={notificationsEnabled}
      onValueChange={handleToggleNotifications}
    />
  );
};
```

---

## 4. Foreground

### Display Notification in Foreground

```typescript
import notifee from '@notifee/react-native';

const displayNotification = async (title: string, body: string) => {
  await notifee.displayNotification({
    title: title,
    body: body,
    android: {
      channelId: 'default',
      smallIcon: 'ic_launcher',
      pressAction: {
        id: 'default'
      }
    },
    ios: {
      sound: 'default'
    }
  });
};

useEffect(() => {
  const unsubscribe = messaging().onMessage(async (remoteMessage) => {
    await displayNotification(
      remoteMessage.notification?.title || 'New Message',
      remoteMessage.notification?.body || ''
    );
  });
  
  return unsubscribe;
}, []);
```

### In-App Notification

```typescript
const InAppNotification = ({ message, onClose }) => {
  return (
    <Animated.View style={styles.container}>
      <View style={styles.notification}>
        <Text style={styles.title}>{message.title}</Text>
        <Text style={styles.body}>{message.body}</Text>
        <TouchableOpacity onPress={onClose}>
          <Icon name="close" size={20} />
        </TouchableOpacity>
      </View>
    </Animated.View>
  );
};

const App = () => {
  const [notification, setNotification] = useState(null);
  
  useEffect(() => {
    const unsubscribe = messaging().onMessage((remoteMessage) => {
      setNotification(remoteMessage.notification);
      
      setTimeout(() => setNotification(null), 5000);
    });
    
    return unsubscribe;
  }, []);
  
  return (
    <>
      <Navigation />
      {notification && (
        <InAppNotification
          message={notification}
          onClose={() => setNotification(null)}
        />
      )}
    </>
  );
};
```

---

## 5. Background

### Background Message Handler

```typescript
// index.js
import messaging from '@react-native-firebase/messaging';

messaging().setBackgroundMessageHandler(async (remoteMessage) => {
  console.log('Message handled in the background!', remoteMessage);
  
  // Process notification data
  if (remoteMessage.data?.type === 'chat') {
    // Save to local database
    await saveChatMessage(remoteMessage.data);
  }
});
```

### Quit State Handler

```typescript
const App = () => {
  useEffect(() => {
    // Check if app was opened from notification
    messaging()
      .getInitialNotification()
      .then((remoteMessage) => {
        if (remoteMessage) {
          console.log('App opened from quit state:', remoteMessage);
          
          // Navigate to specific screen
          if (remoteMessage.data?.screen) {
            navigation.navigate(remoteMessage.data.screen);
          }
        }
      });
  }, []);
  
  return <Navigation />;
};
```

### Background State Handler

```typescript
useEffect(() => {
  const unsubscribe = messaging().onNotificationOpenedApp((remoteMessage) => {
    console.log('App opened from background:', remoteMessage);
    
    // Navigate based on notification data
    if (remoteMessage.data?.userId) {
      navigation.navigate('UserProfile', {
        userId: remoteMessage.data.userId
      });
    }
  });
  
  return unsubscribe;
}, []);
```

---

## 6. Deep Linking

### Configure Deep Links

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<activity android:name=".MainActivity">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" />
  </intent-filter>
</activity>
```

```xml
<!-- ios/YourApp/Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### Handle Deep Links

```typescript
import { Linking } from 'react-native';

const linking = {
  prefixes: ['myapp://'],
  config: {
    screens: {
      Home: 'home',
      Profile: 'profile/:userId',
      Chat: 'chat/:conversationId'
    }
  }
};

const App = () => {
  return (
    <NavigationContainer linking={linking}>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
        <Stack.Screen name="Chat" component={ChatScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

// Send notification with deep link
const sendNotificationWithDeepLink = async (userId: string) => {
  const message = {
    notification: {
      title: 'New Follower',
      body: 'John started following you'
    },
    data: {
      deepLink: `myapp://profile/${userId}`
    },
    token: userToken
  };
  
  await admin.messaging().send(message);
};
```

---

## 7. Scheduled

### Schedule Local Notification

```typescript
const scheduleNotification = (date: Date) => {
  PushNotification.localNotificationSchedule({
    channelId: 'default-channel',
    title: 'Reminder',
    message: 'Don\'t forget to check your tasks!',
    date: date,
    allowWhileIdle: true
  });
};

// Schedule for tomorrow at 9 AM
const scheduleDailyReminder = () => {
  const date = new Date();
  date.setDate(date.getDate() + 1);
  date.setHours(9, 0, 0, 0);
  
  scheduleNotification(date);
};

// Repeating notification
const scheduleRepeatingNotification = () => {
  PushNotification.localNotificationSchedule({
    channelId: 'default-channel',
    title: 'Daily Reminder',
    message: 'Time for your daily check-in!',
    date: new Date(Date.now() + 60 * 1000), // 1 minute from now
    repeatType: 'day', // 'day', 'week', 'month', 'year'
    repeatTime: 1
  });
};
```

### Notifee Scheduled

```typescript
import notifee, { TriggerType, RepeatFrequency } from '@notifee/react-native';

const scheduleNotification = async () => {
  const date = new Date(Date.now());
  date.setHours(9, 0, 0, 0);
  
  const trigger = {
    type: TriggerType.TIMESTAMP,
    timestamp: date.getTime(),
    repeatFrequency: RepeatFrequency.DAILY
  };
  
  await notifee.createTriggerNotification(
    {
      title: 'Good Morning!',
      body: 'Time to start your day',
      android: {
        channelId: 'default'
      }
    },
    trigger
  );
};
```

---

## 8. Rich Notifications

### Notification with Image

```typescript
import notifee from '@notifee/react-native';

const displayImageNotification = async () => {
  await notifee.displayNotification({
    title: 'New Photo',
    body: 'John posted a new photo',
    android: {
      channelId: 'default',
      largeIcon: 'https://example.com/avatar.jpg',
      style: {
        type: AndroidStyle.BIGPICTURE,
        picture: 'https://example.com/photo.jpg'
      }
    },
    ios: {
      attachments: [
        {
          url: 'https://example.com/photo.jpg',
          thumbnailHidden: false
        }
      ]
    }
  });
};
```

### Notification with Actions

```typescript
const displayActionNotification = async () => {
  await notifee.displayNotification({
    title: 'Meeting Reminder',
    body: 'Team meeting starts in 5 minutes',
    android: {
      channelId: 'default',
      actions: [
        {
          title: 'Join',
          pressAction: {
            id: 'join'
          }
        },
        {
          title: 'Snooze',
          pressAction: {
            id: 'snooze'
          }
        }
      ]
    },
    ios: {
      categoryId: 'meeting',
      foregroundPresentationOptions: {
        alert: true,
        badge: true,
        sound: true
      }
    }
  });
};

// Handle action
notifee.onBackgroundEvent(async ({ type, detail }) => {
  const { notification, pressAction } = detail;
  
  if (type === EventType.ACTION_PRESS && pressAction.id === 'join') {
    // Open meeting
    await Linking.openURL('https://meet.example.com/123');
  }
});
```

### Progress Notification

```typescript
const displayProgressNotification = async () => {
  const notificationId = await notifee.displayNotification({
    title: 'Downloading',
    body: 'Download in progress...',
    android: {
      channelId: 'default',
      progress: {
        max: 100,
        current: 0,
        indeterminate: false
      },
      ongoing: true
    }
  });
  
  // Update progress
  for (let i = 0; i <= 100; i += 10) {
    await new Promise(resolve => setTimeout(resolve, 500));
    
    await notifee.displayNotification({
      id: notificationId,
      title: 'Downloading',
      body: `${i}% complete`,
      android: {
        channelId: 'default',
        progress: {
          max: 100,
          current: i
        }
      }
    });
  }
  
  // Complete
  await notifee.displayNotification({
    id: notificationId,
    title: 'Download Complete',
    body: 'File downloaded successfully',
    android: {
      channelId: 'default',
      ongoing: false
    }
  });
};
```

---

## 9. Permissions

### Check Permission

```typescript
import messaging from '@react-native-firebase/messaging';

const checkNotificationPermission = async () => {
  const authStatus = await messaging().hasPermission();
  
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL;
  
  return enabled;
};
```

### Request Permission with Rationale

```typescript
const requestNotificationPermission = async () => {
  const hasPermission = await checkNotificationPermission();
  
  if (hasPermission) {
    return true;
  }
  
  // Show rationale first
  Alert.alert(
    'Enable Notifications',
    'We need notification permission to keep you updated with important messages.',
    [
      {
        text: 'Cancel',
        style: 'cancel'
      },
      {
        text: 'Enable',
        onPress: async () => {
          const authStatus = await messaging().requestPermission();
          return authStatus === messaging.AuthorizationStatus.AUTHORIZED;
        }
      }
    ]
  );
};
```

---

## 10. Best Practices

1. **Request permission at the right time** - when user understands value
2. **Use channels** (Android) for different notification types
3. **Provide opt-out** - let users control notification preferences
4. **Don't spam** - send relevant, timely notifications
5. **Use local notifications** for immediate feedback
6. **Implement deep linking** for better UX
7. **Test on real devices** - notifications don't work on simulators
8. **Handle all states** - foreground, background, quit
9. **Localize messages** - support multiple languages
10. **Monitor metrics** - track open rates, conversions

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Reminder App

App promemoria con:
- Create reminder
- Schedule local notification
- View all reminders
- Mark as done
- Cancel notification

### Esercizio 2: 🟡 Chat App Notifications

Chat con notifiche:
- FCM integration
- New message notifications
- Foreground in-app notification
- Background notification
- Deep link to conversation

### Esercizio 3: 🟡 News App

News app con:
- Topic subscriptions
- Push notifications per categoria
- Rich notifications con immagini
- Action buttons (Read, Save)
- Notification preferences

### Esercizio 4: 🔴 Delivery Tracker

Delivery tracking con:
- Order status notifications
- Progress notifications
- Location-based geofencing notifications
- Scheduled delivery reminder
- Custom sounds per status

### Esercizio 5: 🔴 Social Media App

Social app completa:
- Multiple notification types
- Rich notifications
- Action buttons
- Badge count
- Notification center
- Deep linking
- Topics and targeting

---

## 📝 Quiz

1. **Local notifications sono:**
   - [x] Generate dal device
   - [ ] Inviate da server
   - [ ] Solo iOS
   - [ ] Solo Android

2. **FCM sta per:**
   - [ ] Fast Cloud Messages
   - [x] Firebase Cloud Messaging
   - [ ] Free Cloud Messaging
   - [ ] File Cloud Manager

3. **Background handler serve per:**
   - [ ] UI updates
   - [x] Processare notifiche quando app chiusa
   - [ ] Animazioni
   - [ ] Solo foreground

4. **Deep linking permette:**
   - [ ] Solo navigation
   - [x] Aprire screen specifici da notifica
   - [ ] Solo su iOS
   - [ ] Solo su Android

5. **Channel Android serve per:**
   - [ ] Inviare notifiche
   - [x] Categorizzare notifiche
   - [ ] Solo suoni
   - [ ] Solo immagini

**Risposte**: 1-a, 2-b, 3-b, 4-b, 5-b

---

## 🔗 Risorse

- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [React Native Push Notification](https://github.com/zo0r/react-native-push-notification)
- [Notifee Documentation](https://notifee.app/)
- [FCM HTTP v1 API](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)

---

## ✅ Checklist

- [ ] Local notifications implementate
- [ ] FCM configurato
- [ ] Remote notifications funzionanti
- [ ] Foreground handling
- [ ] Background handling
- [ ] Deep linking
- [ ] Scheduled notifications
- [ ] Rich notifications
- [ ] Permissions gestiti
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📱 **Local** per notifiche immediate device
2. ☁️ **FCM** per notifiche da server
3. 🎯 **Foreground** mostra notifica in-app
4. 📴 **Background** gestisci quando app chiusa
5. 🔗 **Deep linking** apri screen specifici
6. ⏰ **Scheduled** per promemoria
7. 🖼️ **Rich** con immagini e azioni
8. 🔐 **Permissions** richiedi al momento giusto
9. 📊 **Topics** per targeting
10. ✅ **Best practices** non spammare

---

**Eccezionale! 🎉** Ora padroneggi le notifiche push! Prossimo: Moduli Nativi!

[← Precedente: Geolocalizzazione e Mappe](16_Geolocalizzazione_Mappe.md) | [Torna all'Indice](../README.md) | [Prossimo: Moduli Nativi e Bridging →](18_Moduli_Nativi.md)
