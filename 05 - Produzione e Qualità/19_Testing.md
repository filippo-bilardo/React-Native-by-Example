# Capitolo 19: Testing e Quality Assurance

[← Precedente: Moduli Nativi](../04%20-%20Funzionalità%20Native/18_Moduli_Nativi.md) | [Torna all'Indice](../README.md) | [Prossimo: Ottimizzazione Performance →](20_Performance.md)

---

## Descrizione

Il testing è fondamentale per garantire qualità, affidabilità e manutenibilità delle app React Native. Una suite di test completa include unit tests per logica, integration tests per componenti, E2E tests per user flows, e snapshot tests per UI consistency.

In questo capitolo imparerai a configurare Jest, scrivere unit tests, usare React Native Testing Library, implementare E2E testing con Detox, gestire mocking, seguire TDD workflow, misurare code coverage, e integrare testing in CI/CD.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Configurare Jest per React Native
- [ ] Scrivere unit tests efficaci
- [ ] Testare componenti con Testing Library
- [ ] Implementare E2E tests con Detox
- [ ] Creare snapshot tests
- [ ] Mockare dipendenze esterne
- [ ] Seguire TDD workflow
- [ ] Misurare code coverage
- [ ] Integrare testing in CI/CD
- [ ] Testare hooks personalizzati

## 📚 Argomenti Teorici

1. [Jest Setup](#1-jest-setup)
2. [Unit Testing](#2-unit-testing)
3. [Component Testing](#3-component-testing)
4. [Testing Library](#4-testing-library)
5. [Snapshot Testing](#5-snapshot-testing)
6. [Mocking](#6-mocking)
7. [E2E Testing](#7-e2e-testing)
8. [TDD Workflow](#8-tdd)
9. [Code Coverage](#9-coverage)
10. [CI/CD Integration](#10-ci-cd)

---

## 1. Jest Setup

### Installation

```bash
npm install --save-dev jest @testing-library/react-native @testing-library/jest-native
```

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['@testing-library/jest-native/extend-expect'],
  transformIgnorePatterns: [
    'node_modules/(?!(react-native|@react-native|@react-navigation)/)'
  ],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand"
  }
}
```

---

## 2. Unit Testing

### Test Pure Functions

```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

export const multiply = (a: number, b: number): number => a * b;

export const calculateDiscount = (price: number, percentage: number): number => {
  if (percentage < 0 || percentage > 100) {
    throw new Error('Percentage must be between 0 and 100');
  }
  return price - (price * percentage) / 100;
};

// utils/math.test.ts
import { add, multiply, calculateDiscount } from './math';

describe('Math utilities', () => {
  describe('add', () => {
    it('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });
    
    it('should add negative numbers', () => {
      expect(add(-5, -3)).toBe(-8);
    });
    
    it('should handle zero', () => {
      expect(add(0, 5)).toBe(5);
    });
  });
  
  describe('calculateDiscount', () => {
    it('should calculate 10% discount correctly', () => {
      expect(calculateDiscount(100, 10)).toBe(90);
    });
    
    it('should throw error for invalid percentage', () => {
      expect(() => calculateDiscount(100, -10)).toThrow();
      expect(() => calculateDiscount(100, 150)).toThrow();
    });
    
    it('should handle 0% discount', () => {
      expect(calculateDiscount(100, 0)).toBe(100);
    });
  });
});
```

### Test Async Functions

```typescript
// api/users.ts
export const fetchUser = async (id: string): Promise<User> => {
  const response = await fetch(`https://api.example.com/users/${id}`);
  if (!response.ok) {
    throw new Error('Failed to fetch user');
  }
  return response.json();
};

// api/users.test.ts
import { fetchUser } from './users';

describe('fetchUser', () => {
  beforeEach(() => {
    global.fetch = jest.fn();
  });
  
  afterEach(() => {
    jest.restoreAllMocks();
  });
  
  it('should fetch user successfully', async () => {
    const mockUser = { id: '1', name: 'John' };
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser
    });
    
    const user = await fetchUser('1');
    
    expect(user).toEqual(mockUser);
    expect(global.fetch).toHaveBeenCalledWith('https://api.example.com/users/1');
  });
  
  it('should throw error on failed fetch', async () => {
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: false
    });
    
    await expect(fetchUser('1')).rejects.toThrow('Failed to fetch user');
  });
});
```

---

## 3. Component Testing

### Basic Component Test

```typescript
// components/Button.tsx
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

interface ButtonProps {
  title: string;
  onPress: () => void;
  disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({ title, onPress, disabled }) => {
  return (
    <TouchableOpacity
      style={[styles.button, disabled && styles.disabled]}
      onPress={onPress}
      disabled={disabled}
      testID="custom-button"
    >
      <Text style={styles.text}>{title}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8
  },
  disabled: {
    opacity: 0.5
  },
  text: {
    color: 'white',
    textAlign: 'center'
  }
});

// components/Button.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { Button } from './Button';

describe('Button', () => {
  it('should render correctly', () => {
    const { getByText } = render(
      <Button title="Click me" onPress={() => {}} />
    );
    
    expect(getByText('Click me')).toBeTruthy();
  });
  
  it('should call onPress when pressed', () => {
    const onPressMock = jest.fn();
    const { getByTestId } = render(
      <Button title="Click me" onPress={onPressMock} />
    );
    
    fireEvent.press(getByTestId('custom-button'));
    
    expect(onPressMock).toHaveBeenCalledTimes(1);
  });
  
  it('should not call onPress when disabled', () => {
    const onPressMock = jest.fn();
    const { getByTestId } = render(
      <Button title="Click me" onPress={onPressMock} disabled />
    );
    
    fireEvent.press(getByTestId('custom-button'));
    
    expect(onPressMock).not.toHaveBeenCalled();
  });
});
```

---

## 4. Testing Library

### Query Methods

```typescript
import { render, screen, waitFor } from '@testing-library/react-native';

const { getByText } = render(<MyComponent />);
// Throws error if not found

const { queryByText } = render(<MyComponent />);
// Returns null if not found

const { findByText } = render(<MyComponent />);
// Returns promise, waits for element
```

### Test User Input

```typescript
// components/LoginForm.tsx
import React, { useState } from 'react';
import { View, TextInput, Button, Text } from 'react-native';

export const LoginForm: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  
  const handleLogin = () => {
    if (!email || !password) {
      setError('Email and password are required');
      return;
    }
    // Login logic
  };
  
  return (
    <View>
      <TextInput
        testID="email-input"
        value={email}
        onChangeText={setEmail}
        placeholder="Email"
      />
      <TextInput
        testID="password-input"
        value={password}
        onChangeText={setPassword}
        placeholder="Password"
        secureTextEntry
      />
      <Button testID="login-button" title="Login" onPress={handleLogin} />
      {error && <Text testID="error-message">{error}</Text>}
    </View>
  );
};

// components/LoginForm.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('should show error when fields are empty', () => {
    const { getByTestId, queryByTestId } = render(<LoginForm />);
    
    expect(queryByTestId('error-message')).toBeNull();
    
    fireEvent.press(getByTestId('login-button'));
    
    expect(getByTestId('error-message')).toHaveTextContent(
      'Email and password are required'
    );
  });
  
  it('should update input values', () => {
    const { getByTestId } = render(<LoginForm />);
    
    const emailInput = getByTestId('email-input');
    const passwordInput = getByTestId('password-input');
    
    fireEvent.changeText(emailInput, 'test@example.com');
    fireEvent.changeText(passwordInput, 'password123');
    
    expect(emailInput.props.value).toBe('test@example.com');
    expect(passwordInput.props.value).toBe('password123');
  });
});
```

### Test Async Components

```typescript
// components/UserProfile.tsx
import React, { useEffect, useState } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';

interface User {
  id: string;
  name: string;
  email: string;
}

export const UserProfile: React.FC<{ userId: string }> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');
  
  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(() => setError('Failed to load user'))
      .finally(() => setLoading(false));
  }, [userId]);
  
  if (loading) return <ActivityIndicator testID="loading" />;
  if (error) return <Text testID="error">{error}</Text>;
  if (!user) return <Text>No user found</Text>;
  
  return (
    <View testID="user-profile">
      <Text testID="user-name">{user.name}</Text>
      <Text testID="user-email">{user.email}</Text>
    </View>
  );
};

// components/UserProfile.test.tsx
import React from 'react';
import { render, waitFor } from '@testing-library/react-native';
import { UserProfile } from './UserProfile';
import { fetchUser } from '../api/users';

jest.mock('../api/users');

describe('UserProfile', () => {
  it('should show loading state initially', () => {
    (fetchUser as jest.Mock).mockImplementation(
      () => new Promise(() => {}) // Never resolves
    );
    
    const { getByTestId } = render(<UserProfile userId="1" />);
    
    expect(getByTestId('loading')).toBeTruthy();
  });
  
  it('should display user data when loaded', async () => {
    const mockUser = { id: '1', name: 'John Doe', email: 'john@example.com' };
    (fetchUser as jest.Mock).mockResolvedValue(mockUser);
    
    const { getByTestId } = render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(getByTestId('user-profile')).toBeTruthy();
    });
    
    expect(getByTestId('user-name')).toHaveTextContent('John Doe');
    expect(getByTestId('user-email')).toHaveTextContent('john@example.com');
  });
  
  it('should show error message on failure', async () => {
    (fetchUser as jest.Mock).mockRejectedValue(new Error('Network error'));
    
    const { getByTestId } = render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(getByTestId('error')).toHaveTextContent('Failed to load user');
    });
  });
});
```

---

## 5. Snapshot Testing

### Create Snapshot

```typescript
// components/Card.test.tsx
import React from 'react';
import { render } from '@testing-library/react-native';
import { Card } from './Card';

describe('Card', () => {
  it('should match snapshot', () => {
    const { toJSON } = render(
      <Card title="Test Card" description="This is a test" />
    );
    
    expect(toJSON()).toMatchSnapshot();
  });
  
  it('should match snapshot with different props', () => {
    const { toJSON } = render(
      <Card
        title="Another Card"
        description="Different content"
        imageUrl="https://example.com/image.jpg"
      />
    );
    
    expect(toJSON()).toMatchSnapshot();
  });
});
```

### Update Snapshots

```bash
# Update all snapshots
npm test -- -u

# Update snapshots interactively
npm test -- --watch
# Press 'u' to update failing snapshots
```

---

## 6. Mocking

### Mock Native Modules

```typescript
// __mocks__/react-native-push-notification.ts
export default {
  configure: jest.fn(),
  localNotification: jest.fn(),
  cancelAllLocalNotifications: jest.fn()
};

// __tests__/notifications.test.ts
import PushNotification from 'react-native-push-notification';

jest.mock('react-native-push-notification');

describe('Notifications', () => {
  it('should send notification', () => {
    sendNotification('Test', 'Message');
    
    expect(PushNotification.localNotification).toHaveBeenCalledWith({
      title: 'Test',
      message: 'Message'
    });
  });
});
```

### Mock Navigation

```typescript
// __mocks__/@react-navigation/native.ts
export const useNavigation = () => ({
  navigate: jest.fn(),
  goBack: jest.fn(),
  dispatch: jest.fn()
});

export const useRoute = () => ({
  params: {}
});

// components/Screen.test.tsx
import { useNavigation } from '@react-navigation/native';

jest.mock('@react-navigation/native');

describe('Screen', () => {
  it('should navigate on button press', () => {
    const navigate = jest.fn();
    (useNavigation as jest.Mock).mockReturnValue({ navigate });
    
    const { getByText } = render(<Screen />);
    
    fireEvent.press(getByText('Go to Details'));
    
    expect(navigate).toHaveBeenCalledWith('Details');
  });
});
```

### Mock AsyncStorage

```typescript
// __mocks__/@react-native-async-storage/async-storage.ts
export default {
  setItem: jest.fn(() => Promise.resolve()),
  getItem: jest.fn(() => Promise.resolve(null)),
  removeItem: jest.fn(() => Promise.resolve()),
  clear: jest.fn(() => Promise.resolve())
};

// utils/storage.test.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

jest.mock('@react-native-async-storage/async-storage');

describe('Storage', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  it('should save user data', async () => {
    await saveUser({ id: '1', name: 'John' });
    
    expect(AsyncStorage.setItem).toHaveBeenCalledWith(
      'user',
      JSON.stringify({ id: '1', name: 'John' })
    );
  });
  
  it('should retrieve user data', async () => {
    (AsyncStorage.getItem as jest.Mock).mockResolvedValue(
      JSON.stringify({ id: '1', name: 'John' })
    );
    
    const user = await getUser();
    
    expect(user).toEqual({ id: '1', name: 'John' });
  });
});
```

---

## 7. E2E Testing

### Detox Setup

```bash
npm install --save-dev detox
detox init
```

### Detox Configuration

```json
// .detoxrc.json
{
  "testRunner": "jest",
  "runnerConfig": "e2e/config.json",
  "apps": {
    "ios": {
      "type": "ios.app",
      "binaryPath": "ios/build/Build/Products/Debug-iphonesimulator/YourApp.app",
      "build": "xcodebuild -workspace ios/YourApp.xcworkspace -scheme YourApp -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build"
    },
    "android": {
      "type": "android.apk",
      "binaryPath": "android/app/build/outputs/apk/debug/app-debug.apk",
      "build": "cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug && cd .."
    }
  },
  "devices": {
    "simulator": {
      "type": "ios.simulator",
      "device": {
        "type": "iPhone 14"
      }
    },
    "emulator": {
      "type": "android.emulator",
      "device": {
        "avdName": "Pixel_4_API_30"
      }
    }
  },
  "configurations": {
    "ios": {
      "device": "simulator",
      "app": "ios"
    },
    "android": {
      "device": "emulator",
      "app": "android"
    }
  }
}
```

### E2E Test Example

```typescript
// e2e/login.test.ts
describe('Login Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });
  
  beforeEach(async () => {
    await device.reloadReactNative();
  });
  
  it('should show login screen', async () => {
    await expect(element(by.id('login-screen'))).toBeVisible();
  });
  
  it('should login successfully', async () => {
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
    
    await waitFor(element(by.id('home-screen')))
      .toBeVisible()
      .withTimeout(5000);
  });
  
  it('should show error on invalid credentials', async () => {
    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');
    await element(by.id('login-button')).tap();
    
    await expect(element(by.text('Invalid credentials'))).toBeVisible();
  });
  
  it('should navigate through tabs', async () => {
    // Login first
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
    
    await waitFor(element(by.id('home-screen'))).toBeVisible();
    
    // Navigate to profile
    await element(by.id('profile-tab')).tap();
    await expect(element(by.id('profile-screen'))).toBeVisible();
    
    // Navigate to settings
    await element(by.id('settings-tab')).tap();
    await expect(element(by.id('settings-screen'))).toBeVisible();
  });
});
```

### Detox Matchers

```typescript
// Visibility
await expect(element(by.id('button'))).toBeVisible();
await expect(element(by.id('button'))).toBeNotVisible();

// Existence
await expect(element(by.id('button'))).toExist();
await expect(element(by.id('button'))).toNotExist();

// Text
await expect(element(by.id('label'))).toHaveText('Hello');
await expect(element(by.id('input'))).toHaveValue('test');

// Actions
await element(by.id('button')).tap();
await element(by.id('input')).typeText('Hello');
await element(by.id('input')).clearText();
await element(by.id('scrollview')).scrollTo('bottom');
```

---

## 8. TDD

### TDD Cycle

1. **Red**: Write failing test
2. **Green**: Write minimal code to pass
3. **Refactor**: Improve code quality

### TDD Example

```typescript
// Step 1: Red - Write failing test
describe('Calculator', () => {
  it('should add two numbers', () => {
    const calculator = new Calculator();
    expect(calculator.add(2, 3)).toBe(5);
  });
});

// Step 2: Green - Make it pass
class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// Step 3: Refactor - Add more tests
describe('Calculator', () => {
  let calculator: Calculator;
  
  beforeEach(() => {
    calculator = new Calculator();
  });
  
  describe('add', () => {
    it('should add positive numbers', () => {
      expect(calculator.add(2, 3)).toBe(5);
    });
    
    it('should add negative numbers', () => {
      expect(calculator.add(-2, -3)).toBe(-5);
    });
    
    it('should handle zero', () => {
      expect(calculator.add(5, 0)).toBe(5);
    });
  });
});
```

---

## 9. Coverage

### Generate Coverage Report

```bash
npm test -- --coverage
```

### Coverage Report Output

```
----------------------|---------|----------|---------|---------|
File                  | % Stmts | % Branch | % Funcs | % Lines |
----------------------|---------|----------|---------|---------|
All files             |   85.71 |    83.33 |   88.89 |   85.71 |
 components           |   90.00 |    87.50 |   92.31 |   90.00 |
  Button.tsx          |   100   |    100   |   100   |   100   |
  Card.tsx            |   95.00 |    87.50 |   100   |   95.00 |
 utils                |   80.00 |    75.00 |   83.33 |   80.00 |
  math.ts             |   100   |    100   |   100   |   100   |
  validation.ts       |   66.67 |    50.00 |   66.67 |   66.67 |
----------------------|---------|----------|---------|---------|
```

### View HTML Coverage Report

```bash
npm test -- --coverage
open coverage/lcov-report/index.html
```

---

## 10. CI/CD

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

### Pre-commit Hook

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test -- --coverage"
    }
  }
}
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Todo App Tests

Scrivi tests per Todo app:
- Unit tests per todo reducer
- Component tests per TodoItem
- Integration test per TodoList
- Snapshot test per UI

### Esercizio 2: 🟡 Form Validation Tests

Test completi per form:
- Input validation tests
- Error message tests
- Submit handler tests
- Async validation tests
- Mock API calls

### Esercizio 3: 🟡 Authentication Tests

Test authentication flow:
- Login component tests
- Auth context tests
- Protected route tests
- Token refresh tests
- E2E login flow

### Esercizio 4: 🔴 E-Commerce Tests

Test suite completa:
- Product listing tests
- Cart functionality tests
- Checkout flow E2E
- Payment integration tests
- Coverage > 80%

### Esercizio 5: 🔴 Social App Tests

Test app complessa:
- Feed component tests
- Post creation tests
- Real-time updates tests
- Infinite scroll tests
- E2E user journey
- CI/CD integration

---

## 📝 Quiz

1. **Jest è:**
   - [x] Testing framework JavaScript
   - [ ] Solo per React
   - [ ] Solo per unit tests
   - [ ] Tool per debugging

2. **Testing Library aiuta a:**
   - [ ] Solo snapshot tests
   - [x] Testare componenti come utenti
   - [ ] Solo E2E tests
   - [ ] Performance testing

3. **Detox è usato per:**
   - [ ] Unit tests
   - [ ] Component tests
   - [x] E2E testing
   - [ ] Code coverage

4. **Snapshot tests verificano:**
   - [ ] Solo logica
   - [ ] Solo performance
   - [x] UI consistency
   - [ ] Solo errori

5. **Code coverage misura:**
   - [ ] Performance
   - [x] Percentuale codice testato
   - [ ] Solo bugs
   - [ ] Solo UI

**Risposte**: 1-a, 2-b, 3-c, 4-c, 5-b

---

## 🔗 Risorse

- [Jest Documentation](https://jestjs.io/)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Detox Documentation](https://wix.github.io/Detox/)
- [Testing Best Practices](https://testingjavascript.com/)

---

## ✅ Checklist

- [ ] Jest configurato
- [ ] Unit tests scritti
- [ ] Component tests implementati
- [ ] Testing Library usata
- [ ] Snapshot tests creati
- [ ] Mocking implementato
- [ ] E2E tests con Detox
- [ ] Code coverage > 80%
- [ ] CI/CD integrato
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 🧪 **Jest** framework standard per testing
2. 📦 **Unit tests** per funzioni pure
3. 🎨 **Component tests** per UI
4. 🔍 **Testing Library** testa come utenti
5. 📸 **Snapshots** per UI consistency
6. 🎭 **Mocking** per dipendenze esterne
7. 🤖 **Detox** per E2E testing
8. 🔴🟢♻️ **TDD** migliora design
9. 📊 **Coverage** misura qualità tests
10. 🚀 **CI/CD** automatizza testing

---

**Fantastico! 🎉** Ora padroneggi il testing in React Native! Prossimo: Ottimizzazione Performance!

[← Precedente: Moduli Nativi](../04%20-%20Funzionalità%20Native/18_Moduli_Nativi.md) | [Torna all'Indice](../README.md) | [Prossimo: Ottimizzazione Performance →](20_Performance.md)
