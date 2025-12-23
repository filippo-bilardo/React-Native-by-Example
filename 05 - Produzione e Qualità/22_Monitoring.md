# Capitolo 22: Monitoring e Analytics

[← Precedente: Build e Deployment](21_Deployment.md) | [Torna all'Indice](../README.md) | [Prossimo: Pattern Avanzati →](../06%20-%20Argomenti%20Avanzati%20e%20Progetti/23_Pattern_Avanzati.md)

---

## Descrizione

Una volta in produzione, monitorare l'app è fondamentale: crash reports per fixare bugs, analytics per capire utenti, performance monitoring per ottimizzare, A/B testing per decisioni data-driven. Senza monitoring, navighi al buio. Con monitoring, hai dati concreti per migliorare.

In questo capitolo imparerai a implementare crash reporting con Sentry e Firebase Crashlytics, integrare analytics (Firebase, Amplitude, Mixpanel), monitorare performance, tracciare user behavior, implementare A/B testing, gestire error boundaries, e creare logging strategies.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Implementare crash reporting
- [ ] Integrare analytics
- [ ] Monitorare performance
- [ ] Tracciare user behavior
- [ ] Implementare A/B testing
- [ ] Gestire error boundaries
- [ ] Creare logging system
- [ ] Analizzare dati utente
- [ ] Ottimizzare basandosi su dati
- [ ] Implementare dashboards

## 📚 Argomenti Teorici

1. [Crash Reporting](#1-crash-reporting)
2. [Firebase Analytics](#2-firebase-analytics)
3. [Advanced Analytics](#3-advanced-analytics)
4. [Performance Monitoring](#4-performance)
5. [User Behavior Tracking](#5-user-behavior)
6. [A/B Testing](#6-ab-testing)
7. [Error Boundaries](#7-error-boundaries)
8. [Logging](#8-logging)
9. [Dashboards](#9-dashboards)
10. [Best Practices](#10-best-practices)

---

## 1. Crash Reporting

### Sentry Setup

```bash
npm install @sentry/react-native
npx @sentry/wizard -i reactNative -p ios android
```

```typescript
// index.js
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  environment: __DEV__ ? 'development' : 'production',
  enableAutoSessionTracking: true,
  tracesSampleRate: 1.0,
  beforeSend(event, hint) {
    // Filter out events
    if (event.exception) {
      // Add context
      event.tags = { ...event.tags, custom: 'value' };
    }
    return event;
  }
});

// Catch React errors
const App = Sentry.wrap(() => {
  return <Navigation />;
});

AppRegistry.registerComponent(appName, () => App);
```

### Capture Errors

```typescript
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error);
}

// Add breadcrumbs
Sentry.addBreadcrumb({
  category: 'navigation',
  message: 'User navigated to Profile',
  level: 'info'
});

// Set user context
Sentry.setUser({
  id: user.id,
  email: user.email,
  username: user.username
});

// Add extra context
Sentry.setContext('order', {
  orderId: '123',
  items: 5,
  total: 99.99
});

// Set tags
Sentry.setTag('feature', 'checkout');
Sentry.setTag('version', '1.2.0');
```

### Firebase Crashlytics

```bash
npm install @react-native-firebase/app
npm install @react-native-firebase/crashlytics
```

```typescript
import crashlytics from '@react-native-firebase/crashlytics';

// Enable crashlytics
crashlytics().setCrashlyticsCollectionEnabled(true);

// Log error
crashlytics().recordError(error);

// Log message
crashlytics().log('User clicked checkout button');

// Set user identifier
crashlytics().setUserId(userId);

// Set custom keys
crashlytics().setAttribute('credits', String(userCredits));
crashlytics().setAttribute('level', String(userLevel));

// Force crash (testing)
if (__DEV__) {
  crashlytics().crash();
}
```

---

## 2. Firebase Analytics

### Setup

```bash
npm install @react-native-firebase/analytics
```

```typescript
import analytics from '@react-native-firebase/analytics';

// Log event
await analytics().logEvent('purchase', {
  items: [
    {
      item_id: 'P12345',
      item_name: 'Organic Apple',
      price: 1.99
    }
  ],
  value: 1.99,
  currency: 'USD'
});

// Screen view
await analytics().logScreenView({
  screen_name: 'ProductDetails',
  screen_class: 'ProductDetailsScreen'
});

// Set user properties
await analytics().setUserProperty('favorite_category', 'electronics');
await analytics().setUserId(userId);
```

### Common Events

```typescript
// App open
await analytics().logAppOpen();

// Sign up
await analytics().logSignUp({
  method: 'email'
});

// Login
await analytics().logLogin({
  method: 'google'
});

// Purchase
await analytics().logPurchase({
  value: 99.99,
  currency: 'USD',
  items: [{
    item_id: '123',
    item_name: 'Product',
    price: 99.99,
    quantity: 1
  }]
});

// Add to cart
await analytics().logAddToCart({
  items: [{
    item_id: 'P123',
    item_name: 'Product',
    price: 29.99
  }]
});

// Search
await analytics().logSearch({
  search_term: 'laptop'
});

// Share
await analytics().logShare({
  content_type: 'product',
  item_id: 'P123'
});
```

---

## 3. Advanced Analytics

### Amplitude

```bash
npm install @amplitude/analytics-react-native
```

```typescript
import { init, track, Identify, identify } from '@amplitude/analytics-react-native';

// Initialize
init('YOUR_API_KEY');

// Track event
track('Button Clicked', {
  buttonName: 'checkout',
  screen: 'cart'
});

// Identify user
const identifyObj = new Identify();
identifyObj.set('email', 'user@example.com');
identifyObj.set('plan', 'premium');
identifyObj.add('totalPurchases', 1);
identify(identifyObj);

// Revenue tracking
import { revenue } from '@amplitude/analytics-react-native';

const revenueEvent = revenue({
  price: 9.99,
  quantity: 1,
  productId: 'premium_plan'
});
track(revenueEvent);
```

### Mixpanel

```bash
npm install mixpanel-react-native
```

```typescript
import { Mixpanel } from 'mixpanel-react-native';

const mixpanel = new Mixpanel('YOUR_TOKEN');
await mixpanel.init();

// Track event
mixpanel.track('Video Played', {
  videoId: '123',
  duration: 120,
  quality: 'HD'
});

// Set user properties
mixpanel.identify(userId);
mixpanel.getPeople().set({
  $email: 'user@example.com',
  $name: 'John Doe',
  plan: 'premium'
});

// Increment property
mixpanel.getPeople().increment('videos_watched', 1);

// Track revenue
mixpanel.getPeople().trackCharge(9.99, {
  product: 'Premium Plan'
});
```

### Custom Analytics Wrapper

```typescript
// analytics/index.ts
import firebaseAnalytics from '@react-native-firebase/analytics';
import { track as amplitudeTrack } from '@amplitude/analytics-react-native';
import { Mixpanel } from 'mixpanel-react-native';

class Analytics {
  private static instance: Analytics;
  private mixpanel: Mixpanel;
  
  private constructor() {
    this.mixpanel = new Mixpanel('TOKEN');
    this.mixpanel.init();
  }
  
  static getInstance(): Analytics {
    if (!Analytics.instance) {
      Analytics.instance = new Analytics();
    }
    return Analytics.instance;
  }
  
  async trackEvent(eventName: string, properties?: object) {
    // Firebase
    await firebaseAnalytics().logEvent(eventName, properties);
    
    // Amplitude
    amplitudeTrack(eventName, properties);
    
    // Mixpanel
    this.mixpanel.track(eventName, properties);
  }
  
  async setUserId(userId: string) {
    await firebaseAnalytics().setUserId(userId);
    this.mixpanel.identify(userId);
  }
  
  async setUserProperties(properties: object) {
    for (const [key, value] of Object.entries(properties)) {
      await firebaseAnalytics().setUserProperty(key, String(value));
    }
    this.mixpanel.getPeople().set(properties);
  }
}

export default Analytics.getInstance();

// Usage
import analytics from './analytics';

analytics.trackEvent('Purchase', {
  amount: 99.99,
  currency: 'USD',
  productId: '123'
});
```

---

## 4. Performance

### Firebase Performance

```bash
npm install @react-native-firebase/perf
```

```typescript
import perf from '@react-native-firebase/perf';

// Trace
const trace = await perf().startTrace('custom_trace');
trace.putAttribute('userId', userId);
trace.putMetric('items_loaded', 42);

// Perform operation
await loadData();

await trace.stop();

// HTTP metric
const metric = await perf().newHttpMetric(url, 'GET');
await metric.start();

const response = await fetch(url);

metric.setHttpResponseCode(response.status);
metric.setResponseContentType(response.headers.get('Content-Type'));

await metric.stop();
```

### Screen Performance

```typescript
const ScreenTracker = ({ children, screenName }) => {
  useEffect(() => {
    const startTrace = async () => {
      const trace = await perf().startTrace(`screen_${screenName}`);
      
      return async () => {
        await trace.stop();
      };
    };
    
    const cleanup = startTrace();
    
    return () => {
      cleanup.then((fn) => fn());
    };
  }, [screenName]);
  
  return <>{children}</>;
};

// Usage
<ScreenTracker screenName="ProductDetails">
  <ProductDetailsScreen />
</ScreenTracker>
```

---

## 5. User Behavior

### Navigation Tracking

```typescript
import { useNavigationState } from '@react-navigation/native';

const useNavigationAnalytics = () => {
  const routeNameRef = useRef<string>();
  const navigationRef = useNavigationContainerRef();
  
  return {
    navigationRef,
    onReady: () => {
      routeNameRef.current = navigationRef.getCurrentRoute()?.name;
    },
    onStateChange: async () => {
      const previousRouteName = routeNameRef.current;
      const currentRouteName = navigationRef.getCurrentRoute()?.name;
      
      if (previousRouteName !== currentRouteName) {
        await analytics.trackEvent('screen_view', {
          screen_name: currentRouteName,
          previous_screen: previousRouteName
        });
      }
      
      routeNameRef.current = currentRouteName;
    }
  };
};

// Usage
const App = () => {
  const { navigationRef, onReady, onStateChange } = useNavigationAnalytics();
  
  return (
    <NavigationContainer
      ref={navigationRef}
      onReady={onReady}
      onStateChange={onStateChange}
    >
      <Stack.Navigator>
        {/* Screens */}
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

### User Session Tracking

```typescript
const useSessionTracking = () => {
  const sessionStartRef = useRef<number>(Date.now());
  
  useEffect(() => {
    const appState = AppState.addEventListener('change', (state) => {
      if (state === 'active') {
        sessionStartRef.current = Date.now();
        analytics.trackEvent('session_start');
      } else if (state === 'background') {
        const sessionDuration = Date.now() - sessionStartRef.current;
        analytics.trackEvent('session_end', {
          duration: sessionDuration
        });
      }
    });
    
    return () => appState.remove();
  }, []);
};
```

---

## 6. AB Testing

### Firebase Remote Config

```bash
npm install @react-native-firebase/remote-config
```

```typescript
import remoteConfig from '@react-native-firebase/remote-config';

// Setup
await remoteConfig().setDefaults({
  new_feature_enabled: false,
  welcome_message: 'Welcome!'
});

// Fetch
await remoteConfig().fetchAndActivate();

// Get values
const newFeatureEnabled = remoteConfig().getBoolean('new_feature_enabled');
const welcomeMessage = remoteConfig().getString('welcome_message');

// Use in component
const FeatureToggle = ({ featureName, children }) => {
  const [enabled, setEnabled] = useState(false);
  
  useEffect(() => {
    remoteConfig().fetchAndActivate().then(() => {
      setEnabled(remoteConfig().getBoolean(featureName));
    });
  }, [featureName]);
  
  return enabled ? <>{children}</> : null;
};

// Usage
<FeatureToggle featureName="new_checkout_flow">
  <NewCheckoutFlow />
</FeatureToggle>
```

### Optimizely

```bash
npm install @optimizely/react-sdk
```

```typescript
import { OptimizelyProvider, useExperiment } from '@optimizely/react-sdk';

const App = () => {
  return (
    <OptimizelyProvider
      optimizely={optimizely}
      user={{ id: userId }}
    >
      <Navigation />
    </OptimizelyProvider>
  );
};

// Use experiment
const CheckoutButton = () => {
  const [variation] = useExperiment('checkout_button_color');
  
  const buttonColor = variation === 'treatment' ? 'green' : 'blue';
  
  return (
    <Button
      style={{ backgroundColor: buttonColor }}
      title="Checkout"
    />
  );
};
```

---

## 7. Error Boundaries

### React Error Boundary

```typescript
import React from 'react';
import * as Sentry from '@sentry/react-native';

interface Props {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Log to Sentry
    Sentry.captureException(error, {
      contexts: {
        react: {
          componentStack: errorInfo.componentStack
        }
      }
    });
  }
  
  handleReset = () => {
    this.setState({ hasError: false, error: undefined });
  };
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <View style={styles.container}>
          <Text style={styles.title}>Oops! Something went wrong</Text>
          <Text style={styles.message}>{this.state.error?.message}</Text>
          <Button title="Try Again" onPress={this.handleReset} />
        </View>
      );
    }
    
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

---

## 8. Logging

### Custom Logger

```typescript
// utils/logger.ts
enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3
}

class Logger {
  private level: LogLevel = __DEV__ ? LogLevel.DEBUG : LogLevel.ERROR;
  
  debug(message: string, ...args: any[]) {
    if (this.level <= LogLevel.DEBUG) {
      console.log(`[DEBUG] ${message}`, ...args);
    }
  }
  
  info(message: string, ...args: any[]) {
    if (this.level <= LogLevel.INFO) {
      console.info(`[INFO] ${message}`, ...args);
      // Send to analytics
      analytics.trackEvent('log_info', { message });
    }
  }
  
  warn(message: string, ...args: any[]) {
    if (this.level <= LogLevel.WARN) {
      console.warn(`[WARN] ${message}`, ...args);
      // Send to analytics
      analytics.trackEvent('log_warn', { message });
    }
  }
  
  error(message: string, error?: Error, ...args: any[]) {
    if (this.level <= LogLevel.ERROR) {
      console.error(`[ERROR] ${message}`, error, ...args);
      // Send to Sentry
      if (error) {
        Sentry.captureException(error, {
          tags: { message }
        });
      }
    }
  }
}

export default new Logger();

// Usage
import logger from './utils/logger';

logger.debug('Component mounted');
logger.info('User logged in', { userId });
logger.warn('API rate limit approaching');
logger.error('Failed to fetch data', error);
```

---

## 9. Dashboards

### Custom Analytics Dashboard

```typescript
const AnalyticsDashboard = () => {
  const [metrics, setMetrics] = useState({
    activeUsers: 0,
    totalSessions: 0,
    avgSessionDuration: 0,
    crashFreeRate: 0
  });
  
  useEffect(() => {
    // Fetch from your analytics API
    fetchAnalytics().then(setMetrics);
  }, []);
  
  return (
    <ScrollView>
      <Card>
        <Text>Active Users</Text>
        <Text>{metrics.activeUsers}</Text>
      </Card>
      
      <Card>
        <Text>Total Sessions</Text>
        <Text>{metrics.totalSessions}</Text>
      </Card>
      
      <Card>
        <Text>Avg Session Duration</Text>
        <Text>{Math.round(metrics.avgSessionDuration / 60)}m</Text>
      </Card>
      
      <Card>
        <Text>Crash-Free Rate</Text>
        <Text>{(metrics.crashFreeRate * 100).toFixed(2)}%</Text>
      </Card>
    </ScrollView>
  );
};
```

---

## 10. Best Practices

1. **Privacy First**: Get user consent before tracking
2. **Meaningful Events**: Track actions that matter
3. **Consistent Naming**: Use naming conventions
4. **Context Rich**: Add relevant properties
5. **Error Context**: Include user actions before crash
6. **Performance Impact**: Minimize tracking overhead
7. **Data Retention**: Follow GDPR/privacy laws
8. **Test Tracking**: Verify events in dev
9. **Document Events**: Maintain event catalog
10. **Regular Review**: Analyze data weekly

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Basic Tracking

Implementa tracking base:
- Screen view tracking
- Button click events
- User login/logout events
- Firebase Analytics integration

### Esercizio 2: 🟡 Crash Reporting

Setup crash reporting:
- Sentry integration
- Error boundaries
- Custom error logging
- Test crash reporting

### Esercizio 3: 🟡 User Behavior

Track user behavior:
- Session tracking
- Navigation flow
- Feature usage
- Custom events
- User properties

### Esercizio 4: 🔴 A/B Testing

Implementa A/B testing:
- Remote Config setup
- Feature flags
- Experiment tracking
- Results analysis

### Esercizio 5: 🔴 Complete Monitoring

Sistema monitoring completo:
- Multiple analytics providers
- Crash reporting
- Performance monitoring
- Custom dashboard
- Automated reports
- Alert system

---

## 📝 Quiz

1. **Sentry è per:**
   - [x] Crash reporting
   - [ ] Solo analytics
   - [ ] Solo performance
   - [ ] A/B testing

2. **Firebase Analytics traccia:**
   - [ ] Solo crashes
   - [x] Eventi e user behavior
   - [ ] Solo performance
   - [ ] Solo errori

3. **Error boundaries catturano:**
   - [x] Errori React rendering
   - [ ] Tutti gli errori
   - [ ] Solo network errors
   - [ ] Solo syntax errors

4. **A/B testing serve per:**
   - [ ] Bug fixing
   - [x] Testare varianti features
   - [ ] Performance
   - [ ] Crash reporting

5. **Remote Config permette:**
   - [x] Modificare app senza deploy
   - [ ] Solo colori
   - [ ] Solo testi
   - [ ] Native code changes

**Risposte**: 1-a, 2-b, 3-a, 4-b, 5-a

---

## 🔗 Risorse

- [Sentry Documentation](https://docs.sentry.io/platforms/react-native/)
- [Firebase Analytics](https://firebase.google.com/docs/analytics)
- [Amplitude](https://www.docs.developers.amplitude.com/)
- [Mixpanel](https://developer.mixpanel.com/docs)

---

## ✅ Checklist

- [ ] Crash reporting implementato
- [ ] Analytics integrato
- [ ] Performance monitoring attivo
- [ ] User behavior tracciato
- [ ] A/B testing configurato
- [ ] Error boundaries implementati
- [ ] Logging system creato
- [ ] Dashboard configurato
- [ ] Privacy compliance verificata
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 💥 **Crash reporting** fondamentale per stabilità
2. 📊 **Analytics** per capire utenti
3. ⚡ **Performance** monitorare sempre
4. 👤 **User behavior** guida decisioni
5. 🧪 **A/B testing** per ottimizzare
6. 🛡️ **Error boundaries** catturano errori React
7. 📝 **Logging** per debugging
8. 🎯 **Context** arricchisci eventi
9. 🔒 **Privacy** GDPR compliance
10. 📈 **Dashboard** visualizza metriche chiave

---

**Fantastico! 🎉** Hai completato la Parte V - Produzione e Qualità! Prossimo: Pattern Avanzati!

[← Precedente: Build e Deployment](21_Deployment.md) | [Torna all'Indice](../README.md) | [Prossimo: Pattern Avanzati →](../06%20-%20Argomenti%20Avanzati%20e%20Progetti/23_Pattern_Avanzati.md)
