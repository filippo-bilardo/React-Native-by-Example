# Capitolo 15: Fotocamera e Galleria

[← Precedente: GraphQL e Apollo](../03%20-%20Dati%20e%20Networking/14_GraphQL_Apollo.md) | [Torna all'Indice](../README.md) | [Prossimo: Geolocalizzazione e Mappe →](16_Geolocalizzazione_Mappe.md)

---

## Descrizione

L'accesso a fotocamera e galleria è fondamentale per molte app moderne: social network, e-commerce, documenti, avatar utente. React Native offre diverse librerie per gestire foto e video: react-native-image-picker per selezione semplice, react-native-camera per controllo avanzato, expo-image-picker per Expo.

In questo capitolo imparerai a implementare selezione foto dalla galleria, scattare foto con la fotocamera, gestire permessi, ottimizzare immagini, implementare upload, e creare interfacce avanzate come custom camera.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Selezionare foto dalla galleria
- [ ] Scattare foto con la fotocamera
- [ ] Gestire permessi iOS e Android
- [ ] Registrare video
- [ ] Ottimizzare dimensioni immagini
- [ ] Implementare crop e edit
- [ ] Creare custom camera UI
- [ ] Gestire upload multiple immagini
- [ ] Implementare avatar picker
- [ ] Utilizzare librerie image manipulation

## 📚 Argomenti Teorici

1. [Image Picker](#1-image-picker)
2. [Camera Integration](#2-camera)
3. [Permissions](#3-permissions)
4. [Video Recording](#4-video)
5. [Image Optimization](#5-optimization)
6. [Crop and Edit](#6-crop-edit)
7. [Custom Camera](#7-custom-camera)
8. [Multiple Images](#8-multiple)
9. [Upload Images](#9-upload)
10. [Best Practices](#10-best-practices)

---

## 1. Image Picker

### Installation

```bash
npm install react-native-image-picker
```

### iOS Setup

```xml
<!-- ios/YourApp/Info.plist -->
<key>NSPhotoLibraryUsageDescription</key>
<string>We need access to your photo library to let you select images</string>
<key>NSCameraUsageDescription</key>
<string>We need access to your camera to take photos</string>
<key>NSMicrophoneUsageDescription</key>
<string>We need access to your microphone for video recording</string>
```

### Android Setup

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

### Select from Gallery

```typescript
import { launchImageLibrary, Asset } from 'react-native-image-picker';

const selectImage = async () => {
  const result = await launchImageLibrary({
    mediaType: 'photo',
    quality: 0.8,
    maxWidth: 1024,
    maxHeight: 1024,
    selectionLimit: 1
  });
  
  if (result.didCancel) {
    console.log('User cancelled image picker');
    return null;
  }
  
  if (result.errorCode) {
    console.error('ImagePicker Error:', result.errorMessage);
    return null;
  }
  
  return result.assets?.[0];
};

// Uso in component
const ProfileScreen = () => {
  const [imageUri, setImageUri] = useState<string | null>(null);
  
  const handleSelectImage = async () => {
    const image = await selectImage();
    if (image?.uri) {
      setImageUri(image.uri);
    }
  };
  
  return (
    <View>
      {imageUri ? (
        <Image source={{ uri: imageUri }} style={styles.image} />
      ) : (
        <View style={styles.placeholder}>
          <Text>No image selected</Text>
        </View>
      )}
      
      <Button title="Select Image" onPress={handleSelectImage} />
    </View>
  );
};
```

### Take Photo with Camera

```typescript
import { launchCamera } from 'react-native-image-picker';

const takePhoto = async () => {
  const result = await launchCamera({
    mediaType: 'photo',
    quality: 0.8,
    maxWidth: 1024,
    maxHeight: 1024,
    cameraType: 'back',
    saveToPhotos: true
  });
  
  if (result.didCancel) {
    return null;
  }
  
  if (result.errorCode) {
    console.error('Camera Error:', result.errorMessage);
    return null;
  }
  
  return result.assets?.[0];
};

const CameraScreen = () => {
  const [photo, setPhoto] = useState<Asset | null>(null);
  
  const handleTakePhoto = async () => {
    const image = await takePhoto();
    if (image) {
      setPhoto(image);
    }
  };
  
  return (
    <View>
      {photo && (
        <Image source={{ uri: photo.uri }} style={styles.photo} />
      )}
      <Button title="Take Photo" onPress={handleTakePhoto} />
    </View>
  );
};
```

### Options Dialog

```typescript
import { Alert } from 'react-native';

const showImagePickerOptions = () => {
  Alert.alert(
    'Select Image',
    'Choose an option',
    [
      {
        text: 'Take Photo',
        onPress: async () => {
          const photo = await takePhoto();
          if (photo?.uri) {
            setImageUri(photo.uri);
          }
        }
      },
      {
        text: 'Choose from Gallery',
        onPress: async () => {
          const image = await selectImage();
          if (image?.uri) {
            setImageUri(image.uri);
          }
        }
      },
      {
        text: 'Cancel',
        style: 'cancel'
      }
    ]
  );
};
```

---

## 2. Camera

### React Native Vision Camera

```bash
npm install react-native-vision-camera
npx pod-install
```

### Permissions

```typescript
import { Camera, useCameraPermission, useMicrophonePermission } from 'react-native-vision-camera';

const CameraScreen = () => {
  const { hasPermission, requestPermission } = useCameraPermission();
  const { hasPermission: hasMicPermission } = useMicrophonePermission();
  
  useEffect(() => {
    if (!hasPermission) {
      requestPermission();
    }
  }, [hasPermission]);
  
  if (!hasPermission) {
    return <Text>No camera permission</Text>;
  }
  
  return <Camera style={StyleSheet.absoluteFill} />;
};
```

### Basic Camera

```typescript
import { Camera, useCameraDevice } from 'react-native-vision-camera';

const CameraScreen = () => {
  const device = useCameraDevice('back');
  const camera = useRef<Camera>(null);
  
  if (device == null) {
    return <Text>No camera device</Text>;
  }
  
  const takePhoto = async () => {
    if (camera.current) {
      const photo = await camera.current.takePhoto({
        flash: 'auto',
        qualityPrioritization: 'balanced'
      });
      
      console.log('Photo:', photo.path);
      setPhotoUri(`file://${photo.path}`);
    }
  };
  
  return (
    <View style={styles.container}>
      <Camera
        ref={camera}
        style={StyleSheet.absoluteFill}
        device={device}
        isActive={true}
        photo={true}
      />
      
      <View style={styles.controls}>
        <TouchableOpacity onPress={takePhoto} style={styles.captureButton}>
          <View style={styles.captureButtonInner} />
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  controls: {
    position: 'absolute',
    bottom: 40,
    left: 0,
    right: 0,
    alignItems: 'center'
  },
  captureButton: {
    width: 70,
    height: 70,
    borderRadius: 35,
    backgroundColor: 'white',
    justifyContent: 'center',
    alignItems: 'center',
    borderWidth: 4,
    borderColor: '#ddd'
  },
  captureButtonInner: {
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: 'white'
  }
});
```

### Switch Camera

```typescript
const [cameraPosition, setCameraPosition] = useState<'back' | 'front'>('back');
const device = useCameraDevice(cameraPosition);

const switchCamera = () => {
  setCameraPosition(prev => prev === 'back' ? 'front' : 'back');
};

<TouchableOpacity onPress={switchCamera} style={styles.switchButton}>
  <Icon name="camera-reverse" size={30} color="white" />
</TouchableOpacity>
```

### Flash Control

```typescript
const [flash, setFlash] = useState<'off' | 'on' | 'auto'>('auto');

const takePhoto = async () => {
  if (camera.current) {
    const photo = await camera.current.takePhoto({
      flash: flash,
      qualityPrioritization: 'quality'
    });
  }
};

const toggleFlash = () => {
  setFlash(current => {
    if (current === 'off') return 'auto';
    if (current === 'auto') return 'on';
    return 'off';
  });
};

<TouchableOpacity onPress={toggleFlash}>
  <Icon 
    name={flash === 'on' ? 'flash' : flash === 'auto' ? 'flash-auto' : 'flash-off'} 
    size={30} 
    color="white" 
  />
</TouchableOpacity>
```

---

## 3. Permissions

### Check Permissions

```typescript
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';
import { Platform } from 'react-native';

const checkCameraPermission = async () => {
  const permission = Platform.OS === 'ios' 
    ? PERMISSIONS.IOS.CAMERA 
    : PERMISSIONS.ANDROID.CAMERA;
  
  const result = await check(permission);
  
  switch (result) {
    case RESULTS.UNAVAILABLE:
      console.log('Camera not available');
      return false;
    case RESULTS.DENIED:
      console.log('Permission denied');
      return false;
    case RESULTS.GRANTED:
      console.log('Permission granted');
      return true;
    case RESULTS.BLOCKED:
      console.log('Permission blocked');
      return false;
  }
};
```

### Request Permissions

```typescript
const requestCameraPermission = async () => {
  const permission = Platform.OS === 'ios' 
    ? PERMISSIONS.IOS.CAMERA 
    : PERMISSIONS.ANDROID.CAMERA;
  
  const result = await request(permission);
  
  if (result === RESULTS.GRANTED) {
    return true;
  } else if (result === RESULTS.BLOCKED) {
    Alert.alert(
      'Permission Required',
      'Camera permission is required. Please enable it in settings.',
      [
        { text: 'Cancel', style: 'cancel' },
        { text: 'Open Settings', onPress: () => Linking.openSettings() }
      ]
    );
  }
  
  return false;
};
```

### Combined Permission Check

```typescript
const checkAndRequestPermissions = async () => {
  const cameraPermission = Platform.OS === 'ios' 
    ? PERMISSIONS.IOS.CAMERA 
    : PERMISSIONS.ANDROID.CAMERA;
    
  const photoPermission = Platform.OS === 'ios'
    ? PERMISSIONS.IOS.PHOTO_LIBRARY
    : PERMISSIONS.ANDROID.READ_EXTERNAL_STORAGE;
  
  const results = await checkMultiple([cameraPermission, photoPermission]);
  
  const deniedPermissions = Object.keys(results).filter(
    key => results[key] !== RESULTS.GRANTED
  );
  
  if (deniedPermissions.length > 0) {
    const requestResults = await requestMultiple(deniedPermissions);
    
    const stillDenied = Object.keys(requestResults).filter(
      key => requestResults[key] !== RESULTS.GRANTED
    );
    
    return stillDenied.length === 0;
  }
  
  return true;
};
```

---

## 4. Video

### Record Video

```typescript
const recordVideo = async () => {
  const result = await launchCamera({
    mediaType: 'video',
    videoQuality: 'high',
    durationLimit: 30, // seconds
    saveToPhotos: true
  });
  
  if (result.assets?.[0]) {
    return result.assets[0];
  }
  
  return null;
};

const VideoScreen = () => {
  const [videoUri, setVideoUri] = useState<string | null>(null);
  
  const handleRecordVideo = async () => {
    const video = await recordVideo();
    if (video?.uri) {
      setVideoUri(video.uri);
    }
  };
  
  return (
    <View>
      {videoUri && (
        <Video
          source={{ uri: videoUri }}
          style={styles.video}
          controls
          resizeMode="contain"
        />
      )}
      <Button title="Record Video" onPress={handleRecordVideo} />
    </View>
  );
};
```

### Vision Camera Video

```typescript
const [isRecording, setIsRecording] = useState(false);

const startRecording = async () => {
  if (camera.current) {
    setIsRecording(true);
    
    camera.current.startRecording({
      flash: 'off',
      onRecordingFinished: (video) => {
        console.log('Video:', video.path);
        setVideoUri(`file://${video.path}`);
        setIsRecording(false);
      },
      onRecordingError: (error) => {
        console.error(error);
        setIsRecording(false);
      }
    });
  }
};

const stopRecording = async () => {
  if (camera.current && isRecording) {
    await camera.current.stopRecording();
  }
};

<Camera
  ref={camera}
  device={device}
  isActive={true}
  video={true}
  audio={true}
/>

<TouchableOpacity
  onPress={isRecording ? stopRecording : startRecording}
  style={[styles.captureButton, isRecording && styles.recording]}
>
  <Text>{isRecording ? 'Stop' : 'Record'}</Text>
</TouchableOpacity>
```

---

## 5. Optimization

### Resize Image

```bash
npm install react-native-image-resizer
```

```typescript
import ImageResizer from 'react-native-image-resizer';

const resizeImage = async (uri: string) => {
  try {
    const resized = await ImageResizer.createResizedImage(
      uri,
      800, // maxWidth
      600, // maxHeight
      'JPEG',
      80, // quality
      0, // rotation
      undefined,
      false,
      { mode: 'contain', onlyScaleDown: true }
    );
    
    return resized.uri;
  } catch (error) {
    console.error('Resize error:', error);
    return uri;
  }
};

// Uso
const handleSelectImage = async () => {
  const image = await selectImage();
  if (image?.uri) {
    const resizedUri = await resizeImage(image.uri);
    setImageUri(resizedUri);
  }
};
```

### Compress Image

```typescript
const compressImage = async (uri: string, quality: number = 0.8) => {
  const resized = await ImageResizer.createResizedImage(
    uri,
    1920,
    1080,
    'JPEG',
    quality * 100,
    0
  );
  
  // Check file size
  const { size } = await RNFS.stat(resized.uri);
  const sizeInMB = size / (1024 * 1024);
  
  if (sizeInMB > 1 && quality > 0.3) {
    // Compress more
    return compressImage(uri, quality - 0.1);
  }
  
  return resized.uri;
};
```

---

## 6. Crop and Edit

### Image Crop Picker

```bash
npm install react-native-image-crop-picker
```

```typescript
import ImageCropPicker from 'react-native-image-crop-picker';

const selectAndCropImage = async () => {
  try {
    const image = await ImageCropPicker.openPicker({
      width: 400,
      height: 400,
      cropping: true,
      cropperCircleOverlay: true, // For circular crop
      compressImageQuality: 0.8,
      includeBase64: false,
      mediaType: 'photo'
    });
    
    return image.path;
  } catch (error) {
    console.log('Picker cancelled');
    return null;
  }
};

const takeAndCropPhoto = async () => {
  try {
    const image = await ImageCropPicker.openCamera({
      width: 400,
      height: 400,
      cropping: true,
      cropperCircleOverlay: true,
      compressImageQuality: 0.8
    });
    
    return image.path;
  } catch (error) {
    console.log('Camera cancelled');
    return null;
  }
};

// Avatar picker component
const AvatarPicker = ({ currentAvatar, onAvatarChange }) => {
  const handleSelectAvatar = () => {
    Alert.alert(
      'Change Avatar',
      'Choose an option',
      [
        {
          text: 'Take Photo',
          onPress: async () => {
            const uri = await takeAndCropPhoto();
            if (uri) onAvatarChange(uri);
          }
        },
        {
          text: 'Choose from Gallery',
          onPress: async () => {
            const uri = await selectAndCropImage();
            if (uri) onAvatarChange(uri);
          }
        },
        { text: 'Cancel', style: 'cancel' }
      ]
    );
  };
  
  return (
    <TouchableOpacity onPress={handleSelectAvatar}>
      <Image 
        source={currentAvatar ? { uri: currentAvatar } : defaultAvatar}
        style={styles.avatar}
      />
      <View style={styles.editBadge}>
        <Icon name="camera" size={16} color="white" />
      </View>
    </TouchableOpacity>
  );
};
```

---

## 7. Custom Camera

### Full-Featured Camera

```typescript
const CustomCamera = () => {
  const camera = useRef<Camera>(null);
  const [cameraPosition, setCameraPosition] = useState<'back' | 'front'>('back');
  const [flash, setFlash] = useState<'off' | 'on' | 'auto'>('auto');
  const [zoom, setZoom] = useState(1);
  const [isRecording, setIsRecording] = useState(false);
  
  const device = useCameraDevice(cameraPosition);
  
  const takePhoto = async () => {
    if (camera.current) {
      const photo = await camera.current.takePhoto({
        flash: flash,
        qualityPrioritization: 'quality',
        enableShutterSound: true
      });
      
      // Save or process photo
      await processPhoto(`file://${photo.path}`);
    }
  };
  
  return (
    <View style={styles.container}>
      <Camera
        ref={camera}
        style={StyleSheet.absoluteFill}
        device={device}
        isActive={true}
        photo={true}
        video={true}
        audio={true}
        zoom={zoom}
      />
      
      {/* Top Controls */}
      <View style={styles.topControls}>
        <TouchableOpacity onPress={() => navigation.goBack()}>
          <Icon name="close" size={30} color="white" />
        </TouchableOpacity>
        
        <TouchableOpacity onPress={toggleFlash}>
          <Icon name={getFlashIcon(flash)} size={30} color="white" />
        </TouchableOpacity>
      </View>
      
      {/* Bottom Controls */}
      <View style={styles.bottomControls}>
        <TouchableOpacity onPress={() => {}}>
          <Icon name="image" size={30} color="white" />
        </TouchableOpacity>
        
        <TouchableOpacity onPress={takePhoto} style={styles.captureButton}>
          <View style={styles.captureButtonInner} />
        </TouchableOpacity>
        
        <TouchableOpacity onPress={switchCamera}>
          <Icon name="camera-reverse" size={30} color="white" />
        </TouchableOpacity>
      </View>
      
      {/* Zoom Slider */}
      <View style={styles.zoomContainer}>
        <Slider
          value={zoom}
          onValueChange={setZoom}
          minimumValue={1}
          maximumValue={10}
          style={styles.zoomSlider}
        />
      </View>
    </View>
  );
};
```

---

## 8. Multiple Images

### Select Multiple

```typescript
const selectMultipleImages = async () => {
  const result = await launchImageLibrary({
    mediaType: 'photo',
    quality: 0.8,
    selectionLimit: 5 // 0 for unlimited
  });
  
  if (result.assets) {
    return result.assets;
  }
  
  return [];
};

const MultiImagePicker = () => {
  const [images, setImages] = useState<Asset[]>([]);
  
  const handleSelectImages = async () => {
    const selectedImages = await selectMultipleImages();
    setImages(prev => [...prev, ...selectedImages]);
  };
  
  const removeImage = (index: number) => {
    setImages(prev => prev.filter((_, i) => i !== index));
  };
  
  return (
    <View>
      <FlatList
        data={images}
        horizontal
        renderItem={({ item, index }) => (
          <View style={styles.imageContainer}>
            <Image source={{ uri: item.uri }} style={styles.thumbnail} />
            <TouchableOpacity
              onPress={() => removeImage(index)}
              style={styles.removeButton}
            >
              <Icon name="close" size={20} color="white" />
            </TouchableOpacity>
          </View>
        )}
      />
      
      <Button title="Add Images" onPress={handleSelectImages} />
    </View>
  );
};
```

---

## 9. Upload

### Upload Single Image

```typescript
const uploadImage = async (uri: string) => {
  const formData = new FormData();
  
  formData.append('image', {
    uri,
    type: 'image/jpeg',
    name: 'photo.jpg'
  } as any);
  
  try {
    const response = await fetch('https://api.example.com/upload', {
      method: 'POST',
      body: formData,
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });
    
    const data = await response.json();
    return data.imageUrl;
  } catch (error) {
    console.error('Upload error:', error);
    throw error;
  }
};
```

### Upload with Progress

```typescript
import axios from 'axios';

const uploadImageWithProgress = async (
  uri: string,
  onProgress: (progress: number) => void
) => {
  const formData = new FormData();
  formData.append('image', {
    uri,
    type: 'image/jpeg',
    name: 'photo.jpg'
  } as any);
  
  const response = await axios.post(
    'https://api.example.com/upload',
    formData,
    {
      headers: {
        'Content-Type': 'multipart/form-data'
      },
      onUploadProgress: (progressEvent) => {
        const percentCompleted = Math.round(
          (progressEvent.loaded * 100) / progressEvent.total
        );
        onProgress(percentCompleted);
      }
    }
  );
  
  return response.data.imageUrl;
};

// Component
const ImageUpload = () => {
  const [uploadProgress, setUploadProgress] = useState(0);
  const [uploading, setUploading] = useState(false);
  
  const handleUpload = async (uri: string) => {
    setUploading(true);
    
    try {
      const imageUrl = await uploadImageWithProgress(uri, setUploadProgress);
      Alert.alert('Success', 'Image uploaded!');
    } catch (error) {
      Alert.alert('Error', 'Upload failed');
    } finally {
      setUploading(false);
      setUploadProgress(0);
    }
  };
  
  return (
    <View>
      {uploading && (
        <View>
          <ProgressBar progress={uploadProgress / 100} />
          <Text>{uploadProgress}%</Text>
        </View>
      )}
    </View>
  );
};
```

---

## 10. Best Practices

1. **Always check and request permissions** before accessing camera/gallery
2. **Optimize images** before upload to reduce bandwidth
3. **Show loading states** during image processing
4. **Handle errors gracefully** with user-friendly messages
5. **Use appropriate quality settings** based on use case
6. **Implement image caching** for better performance
7. **Respect user privacy** - only request necessary permissions
8. **Test on real devices** - camera/gallery work differently on simulators
9. **Implement proper cleanup** - delete temporary files
10. **Consider offline scenarios** - queue uploads when offline

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Profile Avatar

Implementa avatar picker:
- Seleziona da galleria o fotocamera
- Crop circolare
- Resize a 400x400
- Salva in AsyncStorage
- Mostra loading durante upload

### Esercizio 2: 🟡 Photo Gallery App

App galleria foto:
- Visualizza tutte le foto device
- Grid layout
- Full-screen view
- Delete photo
- Share photo

### Esercizio 3: 🟡 Custom Camera

Camera personalizzata con:
- Switch front/back
- Flash control
- Zoom slider
- Capture button
- Preview captured photo
- Save to gallery

### Esercizio 4: 🔴 Instagram-like Stories

Feature stories tipo Instagram:
- Record video fino a 15 secondi
- Progress indicator
- Tap to pause/resume
- Add text overlay
- Save or share

### Esercizio 5: 🔴 Multi-Image Upload

App per listing prodotti:
- Select multiple images
- Drag to reorder
- Crop each image
- Add captions
- Upload con progress
- Retry failed uploads

---

## 📝 Quiz

1. **react-native-image-picker permette:**
   - [ ] Solo galleria
   - [ ] Solo camera
   - [x] Entrambi galleria e camera
   - [ ] Solo video

2. **Per camera custom UI usare:**
   - [ ] Image Picker
   - [x] Vision Camera
   - [ ] Solo native code
   - [ ] WebView

3. **Permissions vanno richieste:**
   - [x] Prima di accedere a camera/galleria
   - [ ] Dopo aver scattato foto
   - [ ] Solo su Android
   - [ ] Solo su iOS

4. **Crop circolare è utile per:**
   - [ ] Paesaggi
   - [x] Avatar/profilo
   - [ ] Screenshot
   - [ ] Video

5. **FormData è usato per:**
   - [ ] Resize immagini
   - [ ] Crop immagini
   - [x] Upload immagini al server
   - [ ] Salvare in AsyncStorage

**Risposte**: 1-c, 2-b, 3-a, 4-b, 5-c

---

## 🔗 Risorse

- [React Native Image Picker](https://github.com/react-native-image-picker/react-native-image-picker)
- [Vision Camera Docs](https://react-native-vision-camera.com/)
- [Image Crop Picker](https://github.com/ivpusic/react-native-image-crop-picker)
- [Image Resizer](https://github.com/bamlab/react-native-image-resizer)

---

## ✅ Checklist

- [ ] Image picker implementato
- [ ] Camera integration funzionante
- [ ] Permissions gestiti
- [ ] Video recording implementato
- [ ] Resize/compress immagini
- [ ] Crop funzionalità
- [ ] Upload con progress
- [ ] Error handling completo
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📷 **Image Picker** per galleria e camera semplice
2. 🎥 **Vision Camera** per controllo avanzato
3. 🔐 **Permissions** obbligatori iOS/Android
4. ✂️ **Crop** per avatar e editing
5. 📉 **Resize** prima di upload
6. 📤 **FormData** per multipart upload
7. 📊 **Progress** per UX migliore
8. 🎯 **Quality** bilanciare con file size
9. 📱 **Test** su device reali
10. 🧹 **Cleanup** file temporanei

---

**Ottimo lavoro! 🎉** Ora sai gestire foto e video come un pro! Prossimo: Geolocalizzazione!

[← Precedente: GraphQL e Apollo](../03%20-%20Dati%20e%20Networking/14_GraphQL_Apollo.md) | [Torna all'Indice](../README.md) | [Prossimo: Geolocalizzazione e Mappe →](16_Geolocalizzazione_Mappe.md)
