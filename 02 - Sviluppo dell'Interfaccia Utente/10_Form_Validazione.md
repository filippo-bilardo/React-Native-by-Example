# Capitolo 10: Formulari e Validazione

[← Precedente: Animazioni e Gestures](09_Animazioni_Gestures.md) | [Torna all'Indice](../README.md) | [Prossimo: Networking →](../03%20-%20Dati%20e%20Networking/11_Networking_API.md)

---

## Descrizione

I form sono presenti in quasi ogni app: login, registrazione, profilo, checkout, e molto altro. Creare form user-friendly con validazione robusta è essenziale per una buona UX. React Hook Form è diventato lo standard per gestire form complessi in modo performante.

In questo capitolo imparerai a creare form accessibili, implementare validazione client-side, gestire errori in modo efficace, usare React Hook Form, e costruire componenti form riutilizzabili.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Creare form controlled e uncontrolled
- [ ] Implementare validazione client-side
- [ ] Usare React Hook Form efficacemente
- [ ] Integrare Yup/Zod per validation schemas
- [ ] Gestire form multi-step
- [ ] Implementare custom input components
- [ ] Gestire file uploads
- [ ] Ottimizzare performance dei form
- [ ] Creare form accessibili
- [ ] Gestire errori e feedback utente

## 📚 Argomenti Teorici

1. [Form Base in React Native](#1-form-base)
2. [Controlled vs Uncontrolled](#2-controlled-uncontrolled)
3. [Validazione Base](#3-validazione-base)
4. [React Hook Form](#4-react-hook-form)
5. [Schema Validation (Yup/Zod)](#5-schema-validation)
6. [Custom Input Components](#6-custom-inputs)
7. [Form Multi-Step](#7-multi-step)
8. [Error Handling](#8-error-handling)
9. [Performance Optimization](#9-optimization)
10. [Best Practices](#10-best-practices)

---

## 1. Form Base

### Simple Form

```typescript
const SimpleForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState<{ email?: string; password?: string }>({});
  
  const handleSubmit = () => {
    const newErrors: typeof errors = {};
    
    if (!email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!password) {
      newErrors.password = 'Password is required';
    } else if (password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Submit form
    console.log({ email, password });
  };
  
  return (
    <View style={styles.container}>
      <TextInput
        value={email}
        onChangeText={setEmail}
        placeholder="Email"
        keyboardType="email-address"
        autoCapitalize="none"
        style={styles.input}
      />
      {errors.email && <Text style={styles.error}>{errors.email}</Text>}
      
      <TextInput
        value={password}
        onChangeText={setPassword}
        placeholder="Password"
        secureTextEntry
        style={styles.input}
      />
      {errors.password && <Text style={styles.error}>{errors.password}</Text>}
      
      <Button title="Submit" onPress={handleSubmit} />
    </View>
  );
};
```

---

## 2. Controlled vs Uncontrolled

### Controlled Inputs (✅ Consigliato)

```typescript
// State controlla il valore
const [value, setValue] = useState('');

<TextInput
  value={value}
  onChangeText={setValue}
/>
```

### Uncontrolled Inputs

```typescript
// Ref per accedere al valore
const inputRef = useRef<TextInput>(null);

const getValue = () => {
  // ⚠️ Non c'è modo diretto di leggere il valore in RN
  // Usa sempre controlled inputs
};

<TextInput ref={inputRef} />
```

---

## 3. Validazione Base

### Validation Functions

```typescript
const validateEmail = (email: string): string | undefined => {
  if (!email) {
    return 'Email is required';
  }
  if (!/\S+@\S+\.\S+/.test(email)) {
    return 'Email is invalid';
  }
  return undefined;
};

const validatePassword = (password: string): string | undefined => {
  if (!password) {
    return 'Password is required';
  }
  if (password.length < 8) {
    return 'Password must be at least 8 characters';
  }
  if (!/[A-Z]/.test(password)) {
    return 'Password must contain uppercase letter';
  }
  if (!/[0-9]/.test(password)) {
    return 'Password must contain number';
  }
  return undefined;
};
```

### Real-time Validation

```typescript
const [email, setEmail] = useState('');
const [emailError, setEmailError] = useState<string>();
const [touched, setTouched] = useState(false);

const handleEmailChange = (text: string) => {
  setEmail(text);
  
  if (touched) {
    const error = validateEmail(text);
    setEmailError(error);
  }
};

const handleBlur = () => {
  setTouched(true);
  const error = validateEmail(email);
  setEmailError(error);
};

<TextInput
  value={email}
  onChangeText={handleEmailChange}
  onBlur={handleBlur}
/>
{touched && emailError && <Text style={styles.error}>{emailError}</Text>}
```

---

## 4. React Hook Form

### Installation

```bash
npm install react-hook-form
```

### Basic Usage

```typescript
import { useForm, Controller } from 'react-hook-form';

interface FormData {
  email: string;
  password: string;
}

const LoginForm = () => {
  const { control, handleSubmit, formState: { errors } } = useForm<FormData>();
  
  const onSubmit = (data: FormData) => {
    console.log(data);
  };
  
  return (
    <View>
      <Controller
        control={control}
        name="email"
        rules={{
          required: 'Email is required',
          pattern: {
            value: /\S+@\S+\.\S+/,
            message: 'Email is invalid'
          }
        }}
        render={({ field: { onChange, onBlur, value } }) => (
          <TextInput
            value={value}
            onChangeText={onChange}
            onBlur={onBlur}
            placeholder="Email"
            keyboardType="email-address"
            autoCapitalize="none"
            style={styles.input}
          />
        )}
      />
      {errors.email && <Text style={styles.error}>{errors.email.message}</Text>}
      
      <Controller
        control={control}
        name="password"
        rules={{
          required: 'Password is required',
          minLength: {
            value: 6,
            message: 'Password must be at least 6 characters'
          }
        }}
        render={({ field: { onChange, onBlur, value } }) => (
          <TextInput
            value={value}
            onChangeText={onChange}
            onBlur={onBlur}
            placeholder="Password"
            secureTextEntry
            style={styles.input}
          />
        )}
      />
      {errors.password && <Text style={styles.error}>{errors.password.message}</Text>}
      
      <Button title="Submit" onPress={handleSubmit(onSubmit)} />
    </View>
  );
};
```

### Default Values

```typescript
const { control, handleSubmit } = useForm<FormData>({
  defaultValues: {
    email: '',
    password: ''
  }
});
```

### Validation Modes

```typescript
const { control } = useForm({
  mode: 'onChange',     // Validate on change
  // mode: 'onBlur',    // Validate on blur
  // mode: 'onSubmit',  // Validate on submit (default)
  // mode: 'onTouched', // Validate on touch + submit
  // mode: 'all',       // Validate on change + blur
});
```

### Watch Values

```typescript
const { watch } = useForm();

const email = watch('email');
const allValues = watch();

useEffect(() => {
  console.log('Email changed:', email);
}, [email]);
```

### Reset Form

```typescript
const { reset } = useForm();

const handleReset = () => {
  reset();  // Reset to default values
  
  // Or reset to specific values
  reset({
    email: 'new@example.com',
    password: ''
  });
};
```

---

## 5. Schema Validation

### Yup

```bash
npm install yup @hookform/resolvers
```

```typescript
import * as yup from 'yup';
import { yupResolver } from '@hookform/resolvers/yup';

const schema = yup.object({
  email: yup
    .string()
    .required('Email is required')
    .email('Email is invalid'),
  
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(/[A-Z]/, 'Password must contain uppercase letter')
    .matches(/[0-9]/, 'Password must contain number'),
  
  confirmPassword: yup
    .string()
    .required('Confirm password is required')
    .oneOf([yup.ref('password')], 'Passwords must match')
}).required();

type FormData = yup.InferType<typeof schema>;

const SignupForm = () => {
  const { control, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: yupResolver(schema)
  });
  
  const onSubmit = (data: FormData) => {
    console.log(data);
  };
  
  return (
    <View>
      <Controller
        control={control}
        name="email"
        render={({ field }) => (
          <TextInput
            value={field.value}
            onChangeText={field.onChange}
            onBlur={field.onBlur}
            placeholder="Email"
          />
        )}
      />
      {errors.email && <Text style={styles.error}>{errors.email.message}</Text>}
      
      {/* Altri campi... */}
      
      <Button title="Sign Up" onPress={handleSubmit(onSubmit)} />
    </View>
  );
};
```

### Zod

```bash
npm install zod @hookform/resolvers
```

```typescript
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Email is invalid'),
  
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[0-9]/, 'Password must contain number'),
  
  age: z
    .number()
    .min(18, 'Must be 18 or older')
    .max(120, 'Invalid age')
});

type FormData = z.infer<typeof schema>;

const { control, handleSubmit, formState: { errors } } = useForm<FormData>({
  resolver: zodResolver(schema)
});
```

---

## 6. Custom Inputs

### Controlled Input Component

```typescript
interface InputProps {
  label: string;
  value: string;
  onChangeText: (text: string) => void;
  onBlur?: () => void;
  error?: string;
  placeholder?: string;
  secureTextEntry?: boolean;
  keyboardType?: KeyboardTypeOptions;
}

const Input: React.FC<InputProps> = ({
  label,
  value,
  onChangeText,
  onBlur,
  error,
  placeholder,
  secureTextEntry,
  keyboardType
}) => {
  return (
    <View style={styles.container}>
      <Text style={styles.label}>{label}</Text>
      <TextInput
        value={value}
        onChangeText={onChangeText}
        onBlur={onBlur}
        placeholder={placeholder}
        secureTextEntry={secureTextEntry}
        keyboardType={keyboardType}
        style={[styles.input, error && styles.inputError]}
      />
      {error && <Text style={styles.error}>{error}</Text>}
    </View>
  );
};

// Uso
<Controller
  control={control}
  name="email"
  render={({ field, fieldState }) => (
    <Input
      label="Email"
      value={field.value}
      onChangeText={field.onChange}
      onBlur={field.onBlur}
      error={fieldState.error?.message}
      placeholder="Enter your email"
      keyboardType="email-address"
    />
  )}
/>
```

### Select/Picker Component

```typescript
import { Picker } from '@react-native-picker/picker';

interface SelectProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
  options: { label: string; value: string }[];
  error?: string;
}

const Select: React.FC<SelectProps> = ({ label, value, onChange, options, error }) => {
  return (
    <View>
      <Text style={styles.label}>{label}</Text>
      <Picker
        selectedValue={value}
        onValueChange={onChange}
        style={styles.picker}
      >
        {options.map(option => (
          <Picker.Item key={option.value} label={option.label} value={option.value} />
        ))}
      </Picker>
      {error && <Text style={styles.error}>{error}</Text>}
    </View>
  );
};

// Uso con React Hook Form
<Controller
  control={control}
  name="country"
  render={({ field, fieldState }) => (
    <Select
      label="Country"
      value={field.value}
      onChange={field.onChange}
      options={[
        { label: 'USA', value: 'us' },
        { label: 'Italy', value: 'it' },
        { label: 'UK', value: 'uk' }
      ]}
      error={fieldState.error?.message}
    />
  )}
/>
```

### Checkbox Component

```typescript
interface CheckboxProps {
  label: string;
  checked: boolean;
  onChange: (checked: boolean) => void;
  error?: string;
}

const Checkbox: React.FC<CheckboxProps> = ({ label, checked, onChange, error }) => {
  return (
    <View>
      <TouchableOpacity
        style={styles.checkboxContainer}
        onPress={() => onChange(!checked)}
      >
        <View style={[styles.checkbox, checked && styles.checkboxChecked]}>
          {checked && <Text style={styles.checkmark}>✓</Text>}
        </View>
        <Text style={styles.checkboxLabel}>{label}</Text>
      </TouchableOpacity>
      {error && <Text style={styles.error}>{error}</Text>}
    </View>
  );
};

// Uso
<Controller
  control={control}
  name="agreeToTerms"
  rules={{ required: 'You must agree to terms' }}
  render={({ field, fieldState }) => (
    <Checkbox
      label="I agree to terms and conditions"
      checked={field.value}
      onChange={field.onChange}
      error={fieldState.error?.message}
    />
  )}
/>
```

---

## 7. Multi-Step Form

### Setup

```typescript
interface FormData {
  // Step 1
  email: string;
  password: string;
  
  // Step 2
  firstName: string;
  lastName: string;
  
  // Step 3
  address: string;
  city: string;
}

const MultiStepForm = () => {
  const [step, setStep] = useState(1);
  const { control, handleSubmit, trigger, getValues } = useForm<FormData>();
  
  const nextStep = async () => {
    let isValid = false;
    
    if (step === 1) {
      isValid = await trigger(['email', 'password']);
    } else if (step === 2) {
      isValid = await trigger(['firstName', 'lastName']);
    }
    
    if (isValid) {
      setStep(prev => prev + 1);
    }
  };
  
  const prevStep = () => {
    setStep(prev => prev - 1);
  };
  
  const onSubmit = (data: FormData) => {
    console.log('Final data:', data);
  };
  
  return (
    <View style={styles.container}>
      {/* Progress Indicator */}
      <View style={styles.progress}>
        <Text>Step {step} of 3</Text>
        <View style={styles.progressBar}>
          <View style={[styles.progressFill, { width: `${(step / 3) * 100}%` }]} />
        </View>
      </View>
      
      {/* Step 1 */}
      {step === 1 && (
        <View>
          <Controller
            control={control}
            name="email"
            rules={{ required: 'Email is required' }}
            render={({ field, fieldState }) => (
              <Input
                label="Email"
                {...field}
                error={fieldState.error?.message}
              />
            )}
          />
          
          <Controller
            control={control}
            name="password"
            rules={{ required: 'Password is required' }}
            render={({ field, fieldState }) => (
              <Input
                label="Password"
                {...field}
                secureTextEntry
                error={fieldState.error?.message}
              />
            )}
          />
        </View>
      )}
      
      {/* Step 2 */}
      {step === 2 && (
        <View>
          <Controller
            control={control}
            name="firstName"
            rules={{ required: 'First name is required' }}
            render={({ field, fieldState }) => (
              <Input label="First Name" {...field} error={fieldState.error?.message} />
            )}
          />
          
          <Controller
            control={control}
            name="lastName"
            rules={{ required: 'Last name is required' }}
            render={({ field, fieldState }) => (
              <Input label="Last Name" {...field} error={fieldState.error?.message} />
            )}
          />
        </View>
      )}
      
      {/* Step 3 */}
      {step === 3 && (
        <View>
          <Controller
            control={control}
            name="address"
            rules={{ required: 'Address is required' }}
            render={({ field, fieldState }) => (
              <Input label="Address" {...field} error={fieldState.error?.message} />
            )}
          />
          
          <Controller
            control={control}
            name="city"
            rules={{ required: 'City is required' }}
            render={({ field, fieldState }) => (
              <Input label="City" {...field} error={fieldState.error?.message} />
            )}
          />
        </View>
      )}
      
      {/* Navigation */}
      <View style={styles.buttons}>
        {step > 1 && (
          <Button title="Previous" onPress={prevStep} />
        )}
        
        {step < 3 ? (
          <Button title="Next" onPress={nextStep} />
        ) : (
          <Button title="Submit" onPress={handleSubmit(onSubmit)} />
        )}
      </View>
    </View>
  );
};
```

---

## 8. Error Handling

### Display Errors

```typescript
const { formState: { errors } } = useForm();

// Single error
{errors.email && (
  <Text style={styles.error}>{errors.email.message}</Text>
)}

// All errors
{Object.keys(errors).length > 0 && (
  <View style={styles.errorContainer}>
    <Text style={styles.errorTitle}>Please fix the following errors:</Text>
    {Object.entries(errors).map(([key, error]) => (
      <Text key={key} style={styles.error}>
        • {error.message}
      </Text>
    ))}
  </View>
)}
```

### Server Errors

```typescript
const { setError } = useForm();

const onSubmit = async (data: FormData) => {
  try {
    await api.login(data);
  } catch (error) {
    if (error.response?.status === 401) {
      setError('email', {
        type: 'server',
        message: 'Invalid email or password'
      });
    } else {
      setError('root', {
        type: 'server',
        message: 'Something went wrong. Please try again.'
      });
    }
  }
};

// Display root error
{errors.root && (
  <Text style={styles.error}>{errors.root.message}</Text>
)}
```

---

## 9. Optimization

### Avoid Re-renders

```typescript
// ✅ Good - field isolation
<Controller
  control={control}
  name="email"
  render={({ field }) => <Input {...field} />}
/>

// ❌ Bad - triggers re-render
const email = watch('email');
<Input value={email} onChange={...} />
```

### useFormContext

```typescript
// FormProvider in parent
const methods = useForm();

<FormProvider {...methods}>
  <Form />
</FormProvider>

// In child component
const ChildComponent = () => {
  const { control } = useFormContext();
  
  return (
    <Controller
      control={control}
      name="field"
      render={({ field }) => <Input {...field} />}
    />
  );
};
```

---

## 10. Best Practices

### 1. Usa Schema Validation

```typescript
// ✅ Good - centralized validation
const schema = yup.object({ ... });
const { control } = useForm({ resolver: yupResolver(schema) });

// ❌ Bad - rules scattered
<Controller rules={{ required: true, minLength: 6 }} ... />
```

### 2. Component Riutilizzabili

```typescript
// Create reusable Input component
const ControlledInput = ({ name, control, ...props }) => (
  <Controller
    control={control}
    name={name}
    render={({ field, fieldState }) => (
      <Input
        {...field}
        error={fieldState.error?.message}
        {...props}
      />
    )}
  />
);

// Uso
<ControlledInput control={control} name="email" label="Email" />
```

### 3. Accessibility

```typescript
<TextInput
  accessibilityLabel="Email address"
  accessibilityHint="Enter your email to login"
  accessible={true}
/>
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Login Form

Crea form di login con:
- Email e password
- Validazione base
- Show/hide password
- Submit button
- Error messages

### Esercizio 2: 🟡 Registration Form

Form di registrazione con:
- React Hook Form
- Yup validation
- Tutti i campi comuni (name, email, password, confirm, age, country)
- Checkbox privacy policy
- Password strength indicator

### Esercizio 3: 🟡 Dynamic Form

Form dinamico con:
- Add/remove fields
- Array fields (hobbies, phones, etc.)
- React Hook Form useFieldArray
- Validazione per ogni field

### Esercizio 4: 🔴 Multi-Step Wizard

Wizard completo con:
- 4 steps
- Progress indicator
- Validation per step
- Back/Next/Submit
- Review finale prima submit
- Save draft in AsyncStorage

### Esercizio 5: 🔴 Advanced Form System

Sistema form completo con:
- Custom Input components library
- Form builder con JSON schema
- Conditional fields
- File upload
- Auto-save draft
- i18n per error messages

---

## 📝 Quiz

1. **Controlled input in React Native:**
   - [x] Usa value e onChangeText
   - [ ] Usa solo ref
   - [ ] Non esiste
   - [ ] Solo con form libraries

2. **React Hook Form usa:**
   - [ ] Redux
   - [ ] Context
   - [x] Uncontrolled components con refs
   - [ ] State management

3. **Yup vs Zod:**
   - [ ] Yup è deprecato
   - [ ] Zod è più lento
   - [x] Entrambi validi, Zod ha migliori TypeScript types
   - [ ] Non si possono usare insieme

4. **Per form multi-step, validation si fa:**
   - [ ] Solo a fine
   - [x] Per ogni step prima di next
   - [ ] Mai
   - [ ] Solo sul submit

5. **useNativeDriver con form:**
   - [ ] Sempre usarlo
   - [x] Non applicabile ai form
   - [ ] Solo per animations
   - [ ] Obbligatorio

**Risposte**: 1-a, 2-c, 3-c, 4-b, 5-b

---

## 🔗 Risorse

- [React Hook Form Docs](https://react-hook-form.com/)
- [Yup Documentation](https://github.com/jquense/yup)
- [Zod Documentation](https://zod.dev/)

---

## ✅ Checklist

- [ ] Form base implementato
- [ ] Validazione funzionante
- [ ] React Hook Form usato
- [ ] Schema validation integrato
- [ ] Custom components creati
- [ ] Multi-step form implementato
- [ ] Error handling completo
- [ ] Accessibilità considerata
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📝 **Controlled inputs** con value + onChangeText
2. 🎯 **React Hook Form** per form complessi
3. ✅ **Schema validation** con Yup o Zod
4. 🧩 **Component riutilizzabili** per inputs
5. 📊 **Multi-step** con trigger per validare steps
6. 🚨 **Error handling** chiaro e user-friendly
7. ⚡ **Performance** con field isolation
8. ♿ **Accessibility** con accessibilityLabel

---

**Complimenti! 🎉** Ora puoi creare form professionali e user-friendly. Hai completato la Parte II!

[← Precedente: Animazioni e Gestures](09_Animazioni_Gestures.md) | [Torna all'Indice](../README.md) | [Prossimo: Networking →](../03%20-%20Dati%20e%20Networking/11_Networking_API.md)
