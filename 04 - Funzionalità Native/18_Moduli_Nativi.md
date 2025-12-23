# Capitolo 18: Moduli Nativi e Bridging

[← Precedente: Notifiche Push](17_Notifiche_Push.md) | [Torna all'Indice](../README.md) | [Prossimo: Testing e Quality Assurance →](../05%20-%20Produzione%20e%20Qualità/19_Testing.md)

---

## Descrizione

Quando React Native non offre API per una funzionalità nativa specifica, puoi creare moduli nativi personalizzati per accedere a SDK nativi iOS e Android. Il bridging permette comunicazione bidirezionale tra JavaScript e codice nativo, aprendo infinite possibilità.

In questo capitolo imparerai a creare moduli nativi per iOS (Swift/Objective-C) e Android (Java/Kotlin), implementare native UI components, gestire callbacks e promises, usare Turbo Modules per performance, e seguire best practices per bridging.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Creare moduli nativi iOS
- [ ] Creare moduli nativi Android
- [ ] Esporre metodi a JavaScript
- [ ] Gestire callbacks e promises
- [ ] Implementare native UI components
- [ ] Inviare eventi da native a JS
- [ ] Usare Turbo Modules
- [ ] Integrare SDK nativi
- [ ] Debuggare codice nativo
- [ ] Pubblicare moduli nativi

## 📚 Argomenti Teorici

1. [Native Modules Basics](#1-basics)
2. [iOS Module Creation](#2-ios-module)
3. [Android Module Creation](#3-android-module)
4. [Methods and Callbacks](#4-methods)
5. [Promises](#5-promises)
6. [Events](#6-events)
7. [Native UI Components](#7-ui-components)
8. [Turbo Modules](#8-turbo-modules)
9. [Third-Party SDKs](#9-sdks)
10. [Best Practices](#10-best-practices)

---

## 1. Basics

### When to Create Native Modules

- Accesso a API native non disponibili in React Native
- Performance critiche (elaborazione immagini, encryption)
- Integrazione SDK di terze parti (payment, analytics)
- Funzionalità specifiche di piattaforma

### Bridge Architecture

```
┌─────────────────────────────────┐
│   JavaScript (React Native)     │
│   - Business Logic              │
│   - UI Components               │
└──────────────┬──────────────────┘
               │
      ┌────────▼────────┐
      │  Bridge (JSON)  │
      └────────┬────────┘
               │
┌──────────────▼──────────────────┐
│   Native Code                   │
│   - iOS: Swift/Obj-C            │
│   - Android: Java/Kotlin        │
└─────────────────────────────────┘
```

### Project Structure

```
ios/
  YourModule.h
  YourModule.m
  YourModule.swift
  
android/
  app/src/main/java/com/yourapp/
    YourModule.java
    YourPackage.java
```

---

## 2. iOS Module

### Create Objective-C Module

```objective-c
// ios/CalendarModule.h
#import <React/RCTBridgeModule.h>

@interface CalendarModule : NSObject <RCTBridgeModule>
@end

// ios/CalendarModule.m
#import "CalendarModule.h"
#import <EventKit/EventKit.h>

@implementation CalendarModule

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(createEvent:(NSString *)title
                  location:(NSString *)location
                  startDate:(NSDate *)startDate
                  endDate:(NSDate *)endDate
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  EKEventStore *store = [[EKEventStore alloc] init];
  
  [store requestAccessToEntityType:EKEntityTypeEvent completion:^(BOOL granted, NSError *error) {
    if (!granted) {
      reject(@"permission_denied", @"Calendar permission denied", error);
      return;
    }
    
    EKEvent *event = [EKEvent eventWithEventStore:store];
    event.title = title;
    event.location = location;
    event.startDate = startDate;
    event.endDate = endDate;
    event.calendar = [store defaultCalendarForNewEvents];
    
    NSError *saveError = nil;
    [store saveEvent:event span:EKSpanThisEvent error:&saveError];
    
    if (saveError) {
      reject(@"event_creation_failed", @"Failed to create event", saveError);
    } else {
      resolve(event.eventIdentifier);
    }
  }];
}

@end
```

### Create Swift Module

```swift
// ios/DeviceInfo.swift
import Foundation

@objc(DeviceInfo)
class DeviceInfo: NSObject {
  
  @objc
  func getDeviceInfo(_ callback: RCTResponseSenderBlock) {
    let device = UIDevice.current
    
    let info: [String: Any] = [
      "model": device.model,
      "systemName": device.systemName,
      "systemVersion": device.systemVersion,
      "name": device.name,
      "batteryLevel": device.batteryLevel
    ]
    
    callback([info])
  }
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return true
  }
}

// ios/DeviceInfo.m (Bridge file)
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(DeviceInfo, NSObject)

RCT_EXTERN_METHOD(getDeviceInfo:(RCTResponseSenderBlock)callback)

@end
```

### Use in JavaScript

```typescript
import { NativeModules } from 'react-native';

const { CalendarModule, DeviceInfo } = NativeModules;

// Create calendar event
const createEvent = async () => {
  try {
    const eventId = await CalendarModule.createEvent(
      'Meeting',
      'Office',
      new Date(),
      new Date(Date.now() + 3600000)
    );
    console.log('Event created:', eventId);
  } catch (error) {
    console.error('Error:', error);
  }
};

// Get device info
DeviceInfo.getDeviceInfo((info) => {
  console.log('Device:', info);
});
```

---

## 3. Android Module

### Create Module

```java
// android/app/src/main/java/com/yourapp/CalendarModule.java
package com.yourapp;

import android.Manifest;
import android.content.pm.PackageManager;
import androidx.core.app.ActivityCompat;

import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

public class CalendarModule extends ReactContextBaseJavaModule {
  
  public CalendarModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }
  
  @Override
  public String getName() {
    return "CalendarModule";
  }
  
  @ReactMethod
  public void createEvent(String title, String location, 
                         double startDate, double endDate,
                         Promise promise) {
    try {
      // Check permission
      if (ActivityCompat.checkSelfPermission(
          getReactApplicationContext(),
          Manifest.permission.WRITE_CALENDAR
      ) != PackageManager.PERMISSION_GRANTED) {
        promise.reject("permission_denied", "Calendar permission denied");
        return;
      }
      
      // Create event logic
      ContentResolver cr = getReactApplicationContext().getContentResolver();
      ContentValues values = new ContentValues();
      
      values.put(CalendarContract.Events.DTSTART, (long)startDate);
      values.put(CalendarContract.Events.DTEND, (long)endDate);
      values.put(CalendarContract.Events.TITLE, title);
      values.put(CalendarContract.Events.EVENT_LOCATION, location);
      values.put(CalendarContract.Events.CALENDAR_ID, 1);
      values.put(CalendarContract.Events.EVENT_TIMEZONE, TimeZone.getDefault().getID());
      
      Uri uri = cr.insert(CalendarContract.Events.CONTENT_URI, values);
      long eventId = Long.parseLong(uri.getLastPathSegment());
      
      promise.resolve(String.valueOf(eventId));
      
    } catch (Exception e) {
      promise.reject("event_creation_failed", e.getMessage());
    }
  }
}
```

### Create Package

```java
// android/app/src/main/java/com/yourapp/CalendarPackage.java
package com.yourapp;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CalendarPackage implements ReactPackage {
  
  @Override
  public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();
    modules.add(new CalendarModule(reactContext));
    return modules;
  }
  
  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }
}
```

### Register Package

```java
// android/app/src/main/java/com/yourapp/MainApplication.java
@Override
protected List<ReactPackage> getPackages() {
  List<ReactPackage> packages = new PackageList(this).getPackages();
  packages.add(new CalendarPackage()); // Add this line
  return packages;
}
```

---

## 4. Methods

### Synchronous Methods (iOS)

```objective-c
RCT_EXPORT_SYNCHRONOUS_TYPED_METHOD(NSString *, getDeviceName)
{
  return [[UIDevice currentDevice] name];
}
```

```typescript
const deviceName = CalendarModule.getDeviceName();
```

### Methods with Multiple Arguments

```objective-c
RCT_EXPORT_METHOD(multiply:(double)a
                  withB:(double)b
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  NSNumber *result = @(a * b);
  resolve(result);
}
```

```typescript
const result = await CalendarModule.multiply(5, 3);
```

### Callback Methods

```java
@ReactMethod
public void fetchData(Callback successCallback, Callback errorCallback) {
  try {
    String data = "Sample data";
    successCallback.invoke(data);
  } catch (Exception e) {
    errorCallback.invoke(e.getMessage());
  }
}
```

```typescript
CalendarModule.fetchData(
  (data) => console.log('Success:', data),
  (error) => console.error('Error:', error)
);
```

---

## 5. Promises

### iOS Promise

```objective-c
RCT_REMAP_METHOD(asyncOperation,
                 asyncOperationWithResolver:(RCTPromiseResolveBlock)resolve
                 rejecter:(RCTPromiseRejectBlock)reject)
{
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // Simulate async work
    sleep(2);
    
    if (/* success */) {
      resolve(@"Operation completed");
    } else {
      NSError *error = [NSError errorWithDomain:@"com.yourapp" 
                                           code:500 
                                       userInfo:@{NSLocalizedDescriptionKey: @"Operation failed"}];
      reject(@"operation_failed", @"The operation failed", error);
    }
  });
}
```

### Android Promise

```java
@ReactMethod
public void asyncOperation(Promise promise) {
  new Thread(() -> {
    try {
      // Simulate async work
      Thread.sleep(2000);
      promise.resolve("Operation completed");
    } catch (Exception e) {
      promise.reject("operation_failed", e);
    }
  }).start();
}
```

### Use Promises

```typescript
const performOperation = async () => {
  try {
    const result = await CalendarModule.asyncOperation();
    console.log('Result:', result);
  } catch (error) {
    console.error('Error:', error);
  }
};
```

---

## 6. Events

### iOS Event Emitter

```objective-c
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>
@end

@implementation CalendarModule

RCT_EXPORT_MODULE();

- (NSArray<NSString *> *)supportedEvents {
  return @[@"onEventAdded", @"onEventDeleted"];
}

- (void)sendEventToJS {
  [self sendEventWithName:@"onEventAdded" 
                     body:@{@"eventId": @"123", @"title": @"New Event"}];
}

@end
```

### Android Event Emitter

```java
import com.facebook.react.modules.core.DeviceEventManagerModule;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;

public class CalendarModule extends ReactContextBaseJavaModule {
  
  private void sendEvent(String eventName, WritableMap params) {
    getReactApplicationContext()
      .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
      .emit(eventName, params);
  }
  
  public void notifyEventAdded(String eventId, String title) {
    WritableMap params = Arguments.createMap();
    params.putString("eventId", eventId);
    params.putString("title", title);
    
    sendEvent("onEventAdded", params);
  }
}
```

### Listen to Events

```typescript
import { NativeEventEmitter, NativeModules } from 'react-native';

const { CalendarModule } = NativeModules;
const calendarEmitter = new NativeEventEmitter(CalendarModule);

useEffect(() => {
  const subscription = calendarEmitter.addListener('onEventAdded', (event) => {
    console.log('New event:', event);
  });
  
  return () => {
    subscription.remove();
  };
}, []);
```

---

## 7. UI Components

### iOS Custom View

```swift
// ios/CustomView.swift
import UIKit

class CustomView: UIView {
  
  @objc var backgroundColor: UIColor = .white {
    didSet {
      self.backgroundColor = backgroundColor
    }
  }
  
  @objc var onCustomEvent: RCTDirectEventBlock?
  
  override init(frame: CGRect) {
    super.init(frame: frame)
    setupView()
  }
  
  private func setupView() {
    let button = UIButton(type: .system)
    button.setTitle("Native Button", for: .normal)
    button.addTarget(self, action: #selector(handleTap), for: .touchUpInside)
    addSubview(button)
  }
  
  @objc private func handleTap() {
    if let onCustomEvent = onCustomEvent {
      onCustomEvent(["message": "Button tapped"])
    }
  }
}

// ios/CustomViewManager.swift
@objc(CustomViewManager)
class CustomViewManager: RCTViewManager {
  
  override func view() -> UIView! {
    return CustomView()
  }
  
  override static func requiresMainQueueSetup() -> Bool {
    return true
  }
}

// ios/CustomViewManager.m
#import <React/RCTViewManager.h>

@interface RCT_EXTERN_MODULE(CustomViewManager, RCTViewManager)

RCT_EXPORT_VIEW_PROPERTY(backgroundColor, UIColor)
RCT_EXPORT_VIEW_PROPERTY(onCustomEvent, RCTDirectEventBlock)

@end
```

### Android Custom View

```java
// CustomView.java
package com.yourapp;

import android.content.Context;
import android.graphics.Color;
import android.widget.Button;
import android.widget.LinearLayout;

import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.uimanager.events.RCTEventEmitter;

public class CustomView extends LinearLayout {
  
  public CustomView(Context context) {
    super(context);
    setupView();
  }
  
  private void setupView() {
    Button button = new Button(getContext());
    button.setText("Native Button");
    button.setOnClickListener(v -> {
      onCustomEvent();
    });
    addView(button);
  }
  
  public void setBackgroundColor(int color) {
    super.setBackgroundColor(color);
  }
  
  private void onCustomEvent() {
    WritableMap event = Arguments.createMap();
    event.putString("message", "Button tapped");
    
    ReactContext reactContext = (ReactContext) getContext();
    reactContext.getJSModule(RCTEventEmitter.class)
        .receiveEvent(getId(), "onCustomEvent", event);
  }
}

// CustomViewManager.java
package com.yourapp;

import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.annotations.ReactProp;

public class CustomViewManager extends SimpleViewManager<CustomView> {
  
  @Override
  public String getName() {
    return "CustomView";
  }
  
  @Override
  protected CustomView createViewInstance(ThemedReactContext reactContext) {
    return new CustomView(reactContext);
  }
  
  @ReactProp(name = "backgroundColor", customType = "Color")
  public void setBackgroundColor(CustomView view, int color) {
    view.setBackgroundColor(color);
  }
}
```

### Use Custom Component

```typescript
import { requireNativeComponent, ViewStyle } from 'react-native';

interface CustomViewProps {
  style?: ViewStyle;
  backgroundColor?: string;
  onCustomEvent?: (event: any) => void;
}

const CustomView = requireNativeComponent<CustomViewProps>('CustomView');

const App = () => {
  return (
    <CustomView
      style={{ width: 200, height: 100 }}
      backgroundColor="#FF0000"
      onCustomEvent={(event) => {
        console.log('Event:', event.nativeEvent);
      }}
    />
  );
};
```

---

## 8. Turbo Modules

### Create Turbo Module Spec

```typescript
// NativeCalendar.ts
import { TurboModule, TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  createEvent(
    title: string,
    location: string,
    startDate: number,
    endDate: number
  ): Promise<string>;
  
  getEvents(): Promise<Array<Object>>;
}

export default TurboModuleRegistry.get<Spec>('CalendarModule') as Spec | null;
```

### iOS Turbo Module

```objective-c
// ios/CalendarModuleTurbo.h
#import <React/RCTBridgeModule.h>
#import <ReactCommon/RCTTurboModule.h>

@interface CalendarModuleTurbo : NSObject <RCTBridgeModule, RCTTurboModule>
@end
```

---

## 9. SDKs

### Integrate Payment SDK (Stripe)

```typescript
// Create native module wrapper
import { NativeModules } from 'react-native';

const { StripeModule } = NativeModules;

export const initializeStripe = (publishableKey: string) => {
  StripeModule.initialize(publishableKey);
};

export const createPaymentMethod = async (cardDetails: CardDetails) => {
  return await StripeModule.createPaymentMethod(cardDetails);
};
```

---

## 10. Best Practices

1. **Use TypeScript** for type safety in JS interface
2. **Handle errors properly** with promises and error codes
3. **Document your module** with clear API docs
4. **Test thoroughly** on both platforms
5. **Minimize bridge calls** for performance
6. **Use constants** for error codes and event names
7. **Implement proper cleanup** in destructors
8. **Version your module** semantically
9. **Provide example app** for testing
10. **Consider Turbo Modules** for better performance

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Device Info Module

Crea modulo per info device:
- Battery level
- Device model
- OS version
- Network type
- Available storage

### Esercizio 2: 🟡 Biometric Authentication

Modulo per autenticazione:
- Check biometric availability
- Authenticate with fingerprint/FaceID
- Handle success/failure
- Fallback to PIN

### Esercizio 3: 🟡 Custom Toast

Native toast component:
- iOS e Android implementation
- Customizable duration
- Different styles (success, error, info)
- Position (top, center, bottom)

### Esercizio 4: 🔴 Audio Recorder

Audio recorder module:
- Start/stop recording
- Pause/resume
- Get audio level
- Save to file
- Playback

### Esercizio 5: 🔴 Payment SDK Integration

Integra Stripe/PayPal:
- Initialize SDK
- Create payment method
- Process payment
- Handle 3D Secure
- Error handling completo

---

## 📝 Quiz

1. **Native modules permettono:**
   - [ ] Solo iOS
   - [ ] Solo Android
   - [x] Accesso a API native su entrambe piattaforme
   - [ ] Solo UI components

2. **Promise in native module serve per:**
   - [ ] Callback multipli
   - [x] Operazioni asincrone
   - [ ] Solo errori
   - [ ] Sincronizzazione

3. **Eventi permettono:**
   - [x] Comunicazione da native a JS
   - [ ] Solo da JS a native
   - [ ] Solo su iOS
   - [ ] Solo su Android

4. **Turbo Modules offrono:**
   - [ ] Meno performance
   - [x] Migliori performance
   - [ ] Solo per UI
   - [ ] Solo callbacks

5. **requireNativeComponent è per:**
   - [ ] Moduli
   - [x] UI components native
   - [ ] Eventi
   - [ ] Promises

**Risposte**: 1-c, 2-b, 3-a, 4-b, 5-b

---

## 🔗 Risorse

- [Native Modules Docs](https://reactnative.dev/docs/native-modules-intro)
- [iOS Native Modules](https://reactnative.dev/docs/native-modules-ios)
- [Android Native Modules](https://reactnative.dev/docs/native-modules-android)
- [Turbo Modules](https://reactnative.dev/docs/the-new-architecture/pillars-turbomodules)

---

## ✅ Checklist

- [ ] Native module iOS creato
- [ ] Native module Android creato
- [ ] Metodi esposti a JS
- [ ] Promises implementate
- [ ] Eventi implementati
- [ ] UI component nativo creato
- [ ] SDK integrato
- [ ] Error handling completo
- [ ] Testing su device
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🔗 **Bridge** connette JS e Native
2. 🍎 **iOS** usa Objective-C o Swift
3. 🤖 **Android** usa Java o Kotlin
4. ⚡ **Promises** per async operations
5. 📡 **Eventi** da Native a JS
6. 🎨 **UI Components** nativi custom
7. 🚀 **Turbo Modules** per performance
8. 📦 **SDKs** integra librerie native
9. 🧪 **Test** su device reali
10. 📚 **Documenta** API chiaramente

---

**Straordinario! 🎉** Hai completato la Parte IV - Funzionalità Native! Prossimo: Testing e Quality!

[← Precedente: Notifiche Push](17_Notifiche_Push.md) | [Torna all'Indice](../README.md) | [Prossimo: Testing e Quality Assurance →](../05%20-%20Produzione%20e%20Qualità/19_Testing.md)
