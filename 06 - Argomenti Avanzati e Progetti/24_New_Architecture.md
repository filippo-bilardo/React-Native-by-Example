# Capitolo 24: New Architecture (Fabric e TurboModules)

[← Precedente: Pattern Avanzati](23_Pattern_Avanzati.md) | [Torna all'Indice](../README.md) | [Prossimo: Progetto Completo →](25_Progetto_Completo.md)

---

## Descrizione

React Native New Architecture rivoluziona performance e developer experience: Fabric (nuovo rendering system) e TurboModules (nuovi native modules) eliminano bridge asincrono, abilitano rendering sincrono, riducono overhead. JSI (JavaScript Interface) permette comunicazione diretta JS ↔ Native. Codegen genera type-safe interfaces. Hermes engine ottimizzato.

In questo capitolo imparerai cos'è New Architecture, come funziona JSI, Fabric rendering engine, TurboModules system, Codegen, migrazione da Old Architecture, compatibility layer, e best practices.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Comprendere New Architecture
- [ ] Usare JSI
- [ ] Lavorare con Fabric
- [ ] Creare TurboModules
- [ ] Usare Codegen
- [ ] Migrare app esistenti
- [ ] Gestire compatibility
- [ ] Ottimizzare performance
- [ ] Debuggare New Arch
- [ ] Applicare best practices

## 📚 Argomenti Teorici

1. [New Architecture Overview](#1-overview)
2. [JSI - JavaScript Interface](#2-jsi)
3. [Fabric Renderer](#3-fabric)
4. [TurboModules](#4-turbomodules)
5. [Codegen](#5-codegen)
6. [Migration Guide](#6-migration)
7. [Compatibility Layer](#7-compatibility)
8. [Performance Benefits](#8-performance)
9. [Debugging](#9-debugging)
10. [Best Practices](#10-best-practices)

---

## 1. Overview

### Old vs New Architecture

```
OLD ARCHITECTURE:
┌─────────┐         ┌────────┐         ┌────────┐
│   JS    │ ←────→  │ Bridge │ ←────→  │ Native │
└─────────┘  async  └────────┘  async  └────────┘
  - Async bridge communication
  - JSON serialization overhead
  - No type safety
  - Performance bottlenecks

NEW ARCHITECTURE:
┌─────────┐                     ┌────────┐
│   JS    │ ←─────────────────→ │ Native │
└─────────┘     JSI (sync)      └────────┘
  - Direct synchronous calls via JSI
  - No serialization overhead
  - Type-safe with Codegen
  - Better performance
```

### Key Components

```typescript
// New Architecture components:
// 1. JSI - JavaScript Interface
//    Direct C++ <-> JS communication
// 2. Fabric - New rendering system
//    Synchronous layout, better concurrency
// 3. TurboModules - New native modules
//    Lazy loading, type-safe, sync calls
// 4. Codegen - Code generation
//    Type-safe specs from Flow/TypeScript
```

---

## 2. JSI

### What is JSI

```cpp
// JSI allows calling C++ from JS and vice versa

// C++ side
namespace facebook {
namespace jsi {
  
class HostObject {
public:
  virtual Value get(Runtime&, const PropNameID& name) = 0;
  virtual void set(Runtime&, const PropNameID& name, const Value& value) = 0;
};

}
}

// Example: Synchronous native method
class MyHostObject : public facebook::jsi::HostObject {
public:
  Value get(Runtime& runtime, const PropNameID& name) override {
    auto propName = name.utf8(runtime);
    
    if (propName == "multiply") {
      return Function::createFromHostFunction(
        runtime,
        name,
        2,
        [](Runtime& runtime,
           const Value& thisValue,
           const Value* arguments,
           size_t count) -> Value {
          int a = arguments[0].asNumber();
          int b = arguments[1].asNumber();
          return Value(a * b);
        }
      );
    }
    
    return Value::undefined();
  }
};
```

### Installing JSI in Native Module

```javascript
// JavaScript side
import { NativeModules } from 'react-native';

// Old way (async):
// const result = await NativeModules.MyModule.multiply(5, 3);

// New way (sync with JSI):
const result = global.multiply(5, 3); // Returns 15 immediately
console.log(result); // 15 - synchronous!
```

---

## 3. Fabric

### Fabric Renderer

```typescript
// Fabric enables:
// 1. Synchronous layout
// 2. Concurrent rendering
// 3. Better error handling

// Example: Shadow tree operations
// Old Architecture: async, serialized
View.setNativeProps({ style: { backgroundColor: 'red' } }); // Async

// New Architecture: sync, direct
// No setNativeProps needed - state updates are synchronous
```

### Component Descriptors

```typescript
// Component descriptor defines native component

// MyView.ts - JavaScript side
import { requireNativeComponent } from 'react-native';

interface MyViewProps {
  color: string;
  size: number;
}

export default requireNativeComponent<MyViewProps>('MyView');
```

```cpp
// MyViewComponentDescriptor.h - C++ side
#pragma once

#include <react/renderer/components/view/ConcreteViewShadowNode.h>
#include <react/renderer/core/ConcreteComponentDescriptor.h>

namespace facebook {
namespace react {

class MyViewShadowNode final : public ConcreteViewShadowNode<
    ComponentName,
    MyViewProps> {
public:
  using ConcreteViewShadowNode::ConcreteViewShadowNode;
};

}
}
```

---

## 4. TurboModules

### Creating TurboModule

```typescript
// NativeCalculator.ts - Spec file
import { TurboModule, TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  add(a: number, b: number): number;
  multiply(a: number, b: number): Promise<number>;
  getConstants(): {
    PI: number;
    E: number;
  };
}

export default TurboModuleRegistry.getEnforcing<Spec>('Calculator');
```

```java
// Android implementation
package com.myapp;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Promise;

public class CalculatorModule extends ReactContextBaseJavaModule {
  
  public CalculatorModule(ReactApplicationContext context) {
    super(context);
  }
  
  @Override
  public String getName() {
    return "Calculator";
  }
  
  @ReactMethod(isBlockingSynchronousMethod = true)
  public double add(double a, double b) {
    return a + b; // Synchronous!
  }
  
  @ReactMethod
  public void multiply(double a, double b, Promise promise) {
    promise.resolve(a * b);
  }
  
  @Override
  public Map<String, Object> getConstants() {
    Map<String, Object> constants = new HashMap<>();
    constants.put("PI", Math.PI);
    constants.put("E", Math.E);
    return constants;
  }
}
```

```objc
// iOS implementation
#import "Calculator.h"

@implementation Calculator

RCT_EXPORT_MODULE()

RCT_EXPORT_SYNCHRONOUS_TYPED_METHOD(NSNumber *, add:(double)a b:(double)b)
{
  return @(a + b);
}

RCT_EXPORT_METHOD(multiply:(double)a
                  b:(double)b
                  resolve:(RCTPromiseResolveBlock)resolve
                  reject:(RCTPromiseRejectBlock)reject)
{
  resolve(@(a * b));
}

- (NSDictionary *)constantsToExport
{
  return @{
    @"PI": @(M_PI),
    @"E": @(M_E)
  };
}

@end
```

### Using TurboModule

```typescript
// App.tsx
import Calculator from './specs/NativeCalculator';

const App = () => {
  // Synchronous call
  const sum = Calculator.add(5, 3); // Returns 8 immediately
  
  // Async call
  const multiply = async () => {
    const result = await Calculator.multiply(5, 3);
    console.log(result); // 15
  };
  
  // Constants
  const pi = Calculator.getConstants().PI;
  
  return (
    <View>
      <Text>5 + 3 = {sum}</Text>
      <Text>PI = {pi}</Text>
    </View>
  );
};
```

---

## 5. Codegen

### Flow Types

```javascript
// NativeMyModule.js
// @flow

import type { TurboModule } from 'react-native/Libraries/TurboModule/RCTExport';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  +getUser: (id: string) => Promise<{
    id: string,
    name: string,
    email: string,
  }>;
  +saveUser: (user: {
    id: string,
    name: string,
    email: string,
  }) => Promise<void>;
}

export default (TurboModuleRegistry.get<Spec>('MyModule'): ?Spec);
```

### TypeScript Types

```typescript
// NativeMyModule.ts
import { TurboModule, TurboModuleRegistry } from 'react-native';

export interface User {
  id: string;
  name: string;
  email: string;
}

export interface Spec extends TurboModule {
  getUser(id: string): Promise<User>;
  saveUser(user: User): Promise<void>;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule');
```

### Generating Code

```json
// package.json
{
  "scripts": {
    "codegen": "react-native codegen"
  },
  "codegenConfig": {
    "name": "MyAppSpec",
    "type": "modules",
    "jsSrcsDir": "src/specs",
    "android": {
      "javaPackageName": "com.myapp.specs"
    }
  }
}
```

```bash
# Run codegen
npm run codegen

# Generated files:
# Android: android/app/build/generated/source/codegen/
# iOS: ios/build/generated/ios/
```

---

## 6. Migration

### Enable New Architecture

```bash
# Android - android/gradle.properties
newArchEnabled=true

# iOS - ios/Podfile
ENV['RCT_NEW_ARCH_ENABLED'] = '1'

# Reinstall pods
cd ios && pod install
```

### Update Dependencies

```json
// package.json
{
  "dependencies": {
    "react-native": "^0.74.0", // 0.68+ supports New Arch
    "react": "^18.2.0"
  }
}
```

### Migrate Native Modules

```typescript
// Old way
import { NativeModules } from 'react-native';
const { MyModule } = NativeModules;

// New way - create spec
// NativeMyModule.ts
import { TurboModule, TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  myMethod(param: string): Promise<string>;
}

export default TurboModuleRegistry.get<Spec>('MyModule');
```

### Migrate Native Components

```typescript
// Old way
import { requireNativeComponent } from 'react-native';
const MyView = requireNativeComponent('MyView');

// New way - create spec
// MyViewNativeComponent.ts
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';
import { ViewProps } from 'react-native';

interface NativeProps extends ViewProps {
  color: string;
  size: number;
}

export default codegenNativeComponent<NativeProps>('MyView');
```

---

## 7. Compatibility

### Backward Compatibility

```typescript
// Check if New Architecture is enabled
import { Platform } from 'react-native';

const isNewArchEnabled = () => {
  return (
    Platform.OS === 'android' &&
    global.nativeFabricUIManager != null
  ) || (
    Platform.OS === 'ios' &&
    global.__turboModuleProxy != null
  );
};

// Use appropriate implementation
const MyModule = isNewArchEnabled()
  ? require('./specs/NativeMyModule').default
  : require('./legacy/MyModuleLegacy').default;
```

### Interop Layer

```typescript
// ModuleWrapper.ts - Works with both architectures
import { TurboModuleRegistry, NativeModules } from 'react-native';

interface ModuleInterface {
  myMethod(param: string): Promise<string>;
}

class ModuleWrapper implements ModuleInterface {
  private module: any;
  
  constructor() {
    // Try TurboModule first, fallback to legacy
    try {
      this.module = TurboModuleRegistry.get<ModuleInterface>('MyModule');
    } catch {
      this.module = NativeModules.MyModule;
    }
  }
  
  async myMethod(param: string): Promise<string> {
    return this.module.myMethod(param);
  }
}

export default new ModuleWrapper();
```

---

## 8. Performance

### Synchronous Calls

```typescript
// Old Architecture - async
const start = Date.now();
NativeModules.Calculator.add(5, 3).then(result => {
  console.log('Time:', Date.now() - start); // ~10-50ms
});

// New Architecture - sync
const start = Date.now();
const result = Calculator.add(5, 3);
console.log('Time:', Date.now() - start); // <1ms
```

### Rendering Performance

```typescript
// Fabric enables concurrent rendering
import { unstable_enableLogBox } from 'react-native';

// Test with large lists
const HeavyList = () => {
  const data = Array.from({ length: 10000 }, (_, i) => i);
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <Text>{item}</Text>}
      // Fabric handles concurrent rendering better
    />
  );
};
```

### Benchmarking

```typescript
// PerformanceTest.tsx
import { performance } from 'perf_hooks';

const measurePerformance = (fn: () => void, iterations: number) => {
  const start = performance.now();
  
  for (let i = 0; i < iterations; i++) {
    fn();
  }
  
  const end = performance.now();
  return (end - start) / iterations;
};

// Compare Old vs New
const oldArchTime = measurePerformance(() => {
  // Old architecture call
}, 1000);

const newArchTime = measurePerformance(() => {
  // New architecture call
}, 1000);

console.log(`Speedup: ${oldArchTime / newArchTime}x`);
```

---

## 9. Debugging

### Enable Debug Mode

```bash
# Android
adb shell setprop debug.fb.react.enableDebugMode true

# iOS - In Xcode scheme
# Edit Scheme > Run > Arguments > Environment Variables
# Add: OS_ACTIVITY_MODE = disable
```

### Fabric Debugging

```typescript
// Log shadow tree
import { UIManager } from 'react-native';

if (__DEV__) {
  // @ts-ignore
  global.__dumpHierarchy = () => {
    UIManager.dumpHierarchy();
  };
}

// In dev console:
__dumpHierarchy();
```

### TurboModule Debugging

```typescript
// Enable TurboModule logs
// Android - android/app/src/main/AndroidManifest.xml
<application>
  <meta-data
    android:name="com.facebook.react.turbomodule.enableLogging"
    android:value="true" />
</application>

// Check logs
adb logcat | grep TurboModule
```

---

## 10. Best Practices

### Type Safety

```typescript
// Always use TypeScript/Flow with Codegen
// Bad - no types
const module = TurboModuleRegistry.get('MyModule');

// Good - typed
import MyModule from './specs/NativeMyModule';
const result = MyModule.myMethod('test'); // Type-checked
```

### Lazy Loading

```typescript
// TurboModules are lazy-loaded by default
// Only loaded when first used

import Calculator from './specs/NativeCalculator';

// Module not loaded yet

const App = () => {
  const handlePress = () => {
    // Module loaded here on first use
    const result = Calculator.add(5, 3);
  };
  
  return <Button title="Calculate" onPress={handlePress} />;
};
```

### Error Handling

```typescript
// Handle both sync and async errors
const safeCalculate = (a: number, b: number) => {
  try {
    // Sync TurboModule call
    return Calculator.add(a, b);
  } catch (error) {
    console.error('Sync error:', error);
    return 0;
  }
};

const safeMultiply = async (a: number, b: number) => {
  try {
    // Async TurboModule call
    return await Calculator.multiply(a, b);
  } catch (error) {
    console.error('Async error:', error);
    return 0;
  }
};
```

### Testing

```typescript
// Mock TurboModules in tests
jest.mock('./specs/NativeCalculator', () => ({
  __esModule: true,
  default: {
    add: jest.fn((a, b) => a + b),
    multiply: jest.fn((a, b) => Promise.resolve(a * b)),
    getConstants: jest.fn(() => ({ PI: 3.14, E: 2.71 }))
  }
}));

// Test component using TurboModule
test('calculates sum', () => {
  const { getByText } = render(<CalculatorScreen />);
  const sum = getByText(/5 \+ 3 = 8/);
  expect(sum).toBeTruthy();
});
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Enable New Architecture

Enable New Arch in app:
- Update gradle.properties
- Update Podfile
- Install dependencies
- Verify activation
- Test app

### Esercizio 2: 🟡 Create TurboModule

Build Calculator TurboModule:
- TypeScript spec
- Android implementation
- iOS implementation
- Sync methods
- Test in app

### Esercizio 3: 🟡 Migrate Native Module

Migrate existing module:
- Create spec file
- Update native code
- Add backward compatibility
- Test both architectures
- Measure performance

### Esercizio 4: 🔴 Custom Fabric Component

Build native Fabric component:
- Component spec
- Shadow node
- View manager
- Props with Codegen
- Use in React

### Esercizio 5: 🔴 Performance Comparison

Benchmark Old vs New:
- Same operations both archs
- Measure timing
- Compare results
- Analyze improvements
- Document findings

---

## 📝 Quiz

1. **JSI permette:**
   - [x] Chiamate sincrone JS ↔ Native
   - [ ] Solo async calls
   - [ ] Solo UI updates
   - [ ] Solo networking

2. **Fabric è:**
   - [ ] Un framework UI
   - [x] Nuovo rendering engine
   - [ ] State management
   - [ ] Build tool

3. **TurboModules sono:**
   - [x] Lazy-loaded e type-safe
   - [ ] Sempre loaded
   - [ ] Senza types
   - [ ] Solo async

4. **Codegen genera:**
   - [x] Type-safe interfaces da specs
   - [ ] Solo JavaScript
   - [ ] Solo native code
   - [ ] Solo tests

5. **New Architecture migliora:**
   - [x] Performance e type safety
   - [ ] Solo UI
   - [ ] Solo networking
   - [ ] Solo state

**Risposte**: 1-a, 2-b, 3-a, 4-a, 5-a

---

## 🔗 Risorse

- [New Architecture Docs](https://reactnative.dev/docs/new-architecture-intro)
- [TurboModules Guide](https://reactnative.dev/docs/turbo-native-modules-introduction)
- [Fabric Components](https://reactnative.dev/docs/fabric-native-components-introduction)
- [Migration Guide](https://reactnative.dev/docs/new-architecture-app-intro)

---

## ✅ Checklist

- [ ] New Architecture compresa
- [ ] JSI concetti chiari
- [ ] Fabric rendering capito
- [ ] TurboModule creato
- [ ] Codegen usato
- [ ] App migrata
- [ ] Compatibility gestita
- [ ] Performance misurata
- [ ] Debugging techniques note
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🚀 **JSI** elimina bridge overhead
2. ⚡ **Sync calls** molto più veloci
3. 🎨 **Fabric** rendering sincrono
4. 📦 **TurboModules** lazy-loaded
5. 🔒 **Type-safe** con Codegen
6. 🔄 **Migration** graduale possibile
7. 🔧 **Compatibility** layer importante
8. 📊 **Performance** significativamente migliore
9. 🐛 **Debugging** nuovi tools
10. ✅ **Best practices** per adozione

---

**Fantastico! 🎉** Padroneggi New Architecture! Prossimo: Progetto Completo!

[← Precedente: Pattern Avanzati](23_Pattern_Avanzati.md) | [Torna all'Indice](../README.md) | [Prossimo: Progetto Completo →](25_Progetto_Completo.md)
