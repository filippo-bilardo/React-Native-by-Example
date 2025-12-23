# Capitolo 16: Geolocalizzazione e Mappe

[← Precedente: Fotocamera e Galleria](15_Fotocamera_Galleria.md) | [Torna all'Indice](../README.md) | [Prossimo: Notifiche Push →](17_Notifiche_Push.md)

---

## Descrizione

La geolocalizzazione e le mappe sono essenziali per app di navigazione, delivery, social network, fitness, e-commerce. React Native offre librerie per ottenere posizione GPS, mostrare mappe interattive, calcolare distanze, geocoding, e tracking posizione in background.

In questo capitolo imparerai a implementare geolocalizzazione, integrare Google Maps e Apple Maps, mostrare markers e poligoni, calcolare percorsi, implementare tracking in tempo reale, e ottimizzare batteria.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Ottenere posizione GPS utente
- [ ] Gestire permessi location
- [ ] Mostrare mappa interattiva
- [ ] Aggiungere markers e callouts
- [ ] Disegnare polylines e polygons
- [ ] Implementare geocoding e reverse geocoding
- [ ] Calcolare distanze tra coordinate
- [ ] Implementare location tracking
- [ ] Ottimizzare consumo batteria
- [ ] Gestire regioni e geofencing

## 📚 Argomenti Teorici

1. [Geolocation](#1-geolocation)
2. [React Native Maps](#2-maps)
3. [Markers](#3-markers)
4. [Polylines and Polygons](#4-shapes)
5. [Geocoding](#5-geocoding)
6. [Distance Calculation](#6-distance)
7. [Directions API](#7-directions)
8. [Location Tracking](#8-tracking)
9. [Geofencing](#9-geofencing)
10. [Best Practices](#10-best-practices)

---

## 1. Geolocation

### Installation

```bash
npm install @react-native-community/geolocation
```

### iOS Setup

```xml
<!-- ios/YourApp/Info.plist -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>We need your location to show nearby places</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>We need your location for tracking features</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>We need your location for tracking features</string>
```

### Android Setup

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

### Get Current Position

```typescript
import Geolocation from '@react-native-community/geolocation';

interface Position {
  latitude: number;
  longitude: number;
}

const getCurrentPosition = (): Promise<Position> => {
  return new Promise((resolve, reject) => {
    Geolocation.getCurrentPosition(
      (position) => {
        resolve({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude
        });
      },
      (error) => {
        console.error('Geolocation error:', error);
        reject(error);
      },
      {
        enableHighAccuracy: true,
        timeout: 15000,
        maximumAge: 10000
      }
    );
  });
};

// Uso in component
const LocationScreen = () => {
  const [position, setPosition] = useState<Position | null>(null);
  const [loading, setLoading] = useState(false);
  
  const handleGetLocation = async () => {
    setLoading(true);
    try {
      const pos = await getCurrentPosition();
      setPosition(pos);
    } catch (error) {
      Alert.alert('Error', 'Could not get location');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <View>
      <Button title="Get Location" onPress={handleGetLocation} />
      {loading && <ActivityIndicator />}
      {position && (
        <Text>
          Lat: {position.latitude.toFixed(6)}{'\n'}
          Lon: {position.longitude.toFixed(6)}
        </Text>
      )}
    </View>
  );
};
```

### Watch Position

```typescript
const useWatchPosition = () => {
  const [position, setPosition] = useState<Position | null>(null);
  const [error, setError] = useState<string | null>(null);
  const watchId = useRef<number | null>(null);
  
  useEffect(() => {
    watchId.current = Geolocation.watchPosition(
      (position) => {
        setPosition({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude
        });
        setError(null);
      },
      (error) => {
        setError(error.message);
      },
      {
        enableHighAccuracy: true,
        distanceFilter: 10, // meters
        interval: 5000, // ms
        fastestInterval: 2000
      }
    );
    
    return () => {
      if (watchId.current !== null) {
        Geolocation.clearWatch(watchId.current);
      }
    };
  }, []);
  
  return { position, error };
};

// Uso
const TrackingScreen = () => {
  const { position, error } = useWatchPosition();
  
  return (
    <View>
      {error && <Text>Error: {error}</Text>}
      {position && (
        <Text>
          Current: {position.latitude.toFixed(6)}, {position.longitude.toFixed(6)}
        </Text>
      )}
    </View>
  );
};
```

---

## 2. Maps

### Installation

```bash
npm install react-native-maps
```

### iOS Setup

```bash
cd ios && pod install
```

### Android Setup

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<application>
  <meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="YOUR_GOOGLE_MAPS_API_KEY"/>
</application>
```

### Basic Map

```typescript
import MapView, { PROVIDER_GOOGLE } from 'react-native-maps';

const MapScreen = () => {
  const [region, setRegion] = useState({
    latitude: 37.78825,
    longitude: -122.4324,
    latitudeDelta: 0.0922,
    longitudeDelta: 0.0421
  });
  
  return (
    <MapView
      provider={PROVIDER_GOOGLE}
      style={styles.map}
      initialRegion={region}
      showsUserLocation
      showsMyLocationButton
      onRegionChangeComplete={setRegion}
    />
  );
};

const styles = StyleSheet.create({
  map: {
    ...StyleSheet.absoluteFillObject
  }
});
```

### Map Types

```typescript
const [mapType, setMapType] = useState<'standard' | 'satellite' | 'hybrid' | 'terrain'>('standard');

<MapView
  mapType={mapType}
  style={styles.map}
/>

<View style={styles.mapTypeButtons}>
  <Button title="Standard" onPress={() => setMapType('standard')} />
  <Button title="Satellite" onPress={() => setMapType('satellite')} />
  <Button title="Hybrid" onPress={() => setMapType('hybrid')} />
</View>
```

### Animate to Region

```typescript
const mapRef = useRef<MapView>(null);

const animateToUserLocation = async () => {
  const position = await getCurrentPosition();
  
  mapRef.current?.animateToRegion({
    latitude: position.latitude,
    longitude: position.longitude,
    latitudeDelta: 0.01,
    longitudeDelta: 0.01
  }, 1000);
};

<MapView ref={mapRef} />
```

---

## 3. Markers

### Basic Marker

```typescript
import { Marker } from 'react-native-maps';

interface Place {
  id: string;
  name: string;
  latitude: number;
  longitude: number;
}

const MapWithMarkers = () => {
  const places: Place[] = [
    { id: '1', name: 'Place 1', latitude: 37.78825, longitude: -122.4324 },
    { id: '2', name: 'Place 2', latitude: 37.79025, longitude: -122.4344 }
  ];
  
  return (
    <MapView style={styles.map}>
      {places.map(place => (
        <Marker
          key={place.id}
          coordinate={{
            latitude: place.latitude,
            longitude: place.longitude
          }}
          title={place.name}
          description="Tap for more info"
        />
      ))}
    </MapView>
  );
};
```

### Custom Marker

```typescript
<Marker coordinate={coordinate}>
  <View style={styles.customMarker}>
    <Image source={require('./marker-icon.png')} style={styles.markerImage} />
    <Text style={styles.markerText}>10</Text>
  </View>
</Marker>

const styles = StyleSheet.create({
  customMarker: {
    alignItems: 'center'
  },
  markerImage: {
    width: 40,
    height: 40
  },
  markerText: {
    color: 'white',
    fontWeight: 'bold',
    fontSize: 12,
    position: 'absolute',
    top: 10
  }
});
```

### Marker with Callout

```typescript
import { Callout } from 'react-native-maps';

<Marker coordinate={coordinate}>
  <Callout onPress={() => navigation.navigate('PlaceDetail', { placeId })}>
    <View style={styles.callout}>
      <Text style={styles.calloutTitle}>{place.name}</Text>
      <Text>{place.address}</Text>
      <Image source={{ uri: place.image }} style={styles.calloutImage} />
      <Text style={styles.calloutLink}>View Details →</Text>
    </View>
  </Callout>
</Marker>
```

### Clustered Markers

```bash
npm install react-native-maps-super-cluster
```

```typescript
import ClusteredMapView from 'react-native-maps-super-cluster';

const ClusteredMap = () => {
  const [markers, setMarkers] = useState([
    { location: { latitude: 37.78825, longitude: -122.4324 }, id: '1' },
    // ... more markers
  ]);
  
  return (
    <ClusteredMapView
      data={markers}
      initialRegion={region}
      renderMarker={(data) => (
        <Marker
          key={data.id}
          coordinate={data.location}
        />
      )}
      renderCluster={(cluster, onPress) => (
        <Marker
          coordinate={cluster.coordinate}
          onPress={onPress}
        >
          <View style={styles.clusterContainer}>
            <Text style={styles.clusterText}>{cluster.pointCount}</Text>
          </View>
        </Marker>
      )}
    />
  );
};
```

---

## 4. Shapes

### Polyline (Route)

```typescript
import { Polyline } from 'react-native-maps';

const RouteMap = () => {
  const route = [
    { latitude: 37.78825, longitude: -122.4324 },
    { latitude: 37.78925, longitude: -122.4344 },
    { latitude: 37.79025, longitude: -122.4364 }
  ];
  
  return (
    <MapView style={styles.map}>
      <Polyline
        coordinates={route}
        strokeColor="#0000FF"
        strokeWidth={4}
        lineDashPattern={[1]}
      />
    </MapView>
  );
};
```

### Polygon (Area)

```typescript
import { Polygon } from 'react-native-maps';

const AreaMap = () => {
  const area = [
    { latitude: 37.78825, longitude: -122.4324 },
    { latitude: 37.78925, longitude: -122.4344 },
    { latitude: 37.79025, longitude: -122.4364 },
    { latitude: 37.78825, longitude: -122.4324 }
  ];
  
  return (
    <MapView style={styles.map}>
      <Polygon
        coordinates={area}
        fillColor="rgba(0, 200, 0, 0.3)"
        strokeColor="#00C000"
        strokeWidth={2}
      />
    </MapView>
  );
};
```

### Circle (Radius)

```typescript
import { Circle } from 'react-native-maps';

<Circle
  center={{
    latitude: 37.78825,
    longitude: -122.4324
  }}
  radius={500} // meters
  fillColor="rgba(0, 0, 255, 0.2)"
  strokeColor="rgba(0, 0, 255, 0.5)"
  strokeWidth={2}
/>
```

---

## 5. Geocoding

### Installation

```bash
npm install react-native-geocoding
```

### Setup

```typescript
import Geocoding from 'react-native-geocoding';

Geocoding.init('YOUR_GOOGLE_MAPS_API_KEY');
```

### Address to Coordinates

```typescript
const geocodeAddress = async (address: string) => {
  try {
    const json = await Geocoding.from(address);
    const location = json.results[0].geometry.location;
    
    return {
      latitude: location.lat,
      longitude: location.lng
    };
  } catch (error) {
    console.error('Geocoding error:', error);
    return null;
  }
};

// Uso
const SearchScreen = () => {
  const [address, setAddress] = useState('');
  
  const handleSearch = async () => {
    const location = await geocodeAddress(address);
    if (location) {
      mapRef.current?.animateToRegion({
        ...location,
        latitudeDelta: 0.01,
        longitudeDelta: 0.01
      });
    }
  };
  
  return (
    <View>
      <TextInput
        value={address}
        onChangeText={setAddress}
        placeholder="Enter address"
      />
      <Button title="Search" onPress={handleSearch} />
    </View>
  );
};
```

### Coordinates to Address

```typescript
const reverseGeocode = async (latitude: number, longitude: number) => {
  try {
    const json = await Geocoding.from(latitude, longitude);
    const address = json.results[0].formatted_address;
    return address;
  } catch (error) {
    console.error('Reverse geocoding error:', error);
    return null;
  }
};

// Uso
const [address, setAddress] = useState('');

const handleMapPress = async (e: any) => {
  const { latitude, longitude } = e.nativeEvent.coordinate;
  const addr = await reverseGeocode(latitude, longitude);
  if (addr) {
    setAddress(addr);
  }
};

<MapView onPress={handleMapPress} />
```

---

## 6. Distance

### Haversine Formula

```typescript
const calculateDistance = (
  lat1: number,
  lon1: number,
  lat2: number,
  lon2: number
): number => {
  const R = 6371; // Earth radius in km
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);
  
  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);
  
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  const distance = R * c;
  
  return distance;
};

const toRad = (degrees: number): number => {
  return degrees * (Math.PI / 180);
};

// Uso
const distance = calculateDistance(
  37.78825, -122.4324,
  37.79025, -122.4364
);
console.log(`Distance: ${distance.toFixed(2)} km`);
```

### Distance with Library

```bash
npm install geolib
```

```typescript
import { getDistance, getPreciseDistance } from 'geolib';

const distance = getDistance(
  { latitude: 37.78825, longitude: -122.4324 },
  { latitude: 37.79025, longitude: -122.4364 }
);

console.log(`Distance: ${distance} meters`);
```

---

## 7. Directions

### Google Directions API

```typescript
const getDirections = async (
  origin: { latitude: number; longitude: number },
  destination: { latitude: number; longitude: number }
) => {
  const apiKey = 'YOUR_GOOGLE_MAPS_API_KEY';
  const url = `https://maps.googleapis.com/maps/api/directions/json?origin=${origin.latitude},${origin.longitude}&destination=${destination.latitude},${destination.longitude}&key=${apiKey}`;
  
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    if (data.routes.length > 0) {
      const points = decodePolyline(data.routes[0].overview_polyline.points);
      return points;
    }
  } catch (error) {
    console.error('Directions error:', error);
  }
  
  return [];
};

const decodePolyline = (encoded: string) => {
  const points: { latitude: number; longitude: number }[] = [];
  let index = 0;
  let lat = 0;
  let lng = 0;
  
  while (index < encoded.length) {
    let b;
    let shift = 0;
    let result = 0;
    
    do {
      b = encoded.charCodeAt(index++) - 63;
      result |= (b & 0x1f) << shift;
      shift += 5;
    } while (b >= 0x20);
    
    const dlat = (result & 1) !== 0 ? ~(result >> 1) : result >> 1;
    lat += dlat;
    
    shift = 0;
    result = 0;
    
    do {
      b = encoded.charCodeAt(index++) - 63;
      result |= (b & 0x1f) << shift;
      shift += 5;
    } while (b >= 0x20);
    
    const dlng = (result & 1) !== 0 ? ~(result >> 1) : result >> 1;
    lng += dlng;
    
    points.push({
      latitude: lat / 1e5,
      longitude: lng / 1e5
    });
  }
  
  return points;
};

// Uso
const NavigationScreen = () => {
  const [route, setRoute] = useState([]);
  
  const handleGetDirections = async () => {
    const origin = { latitude: 37.78825, longitude: -122.4324 };
    const destination = { latitude: 37.79025, longitude: -122.4364 };
    
    const points = await getDirections(origin, destination);
    setRoute(points);
  };
  
  return (
    <MapView style={styles.map}>
      {route.length > 0 && (
        <Polyline
          coordinates={route}
          strokeColor="#0000FF"
          strokeWidth={4}
        />
      )}
    </MapView>
  );
};
```

---

## 8. Tracking

### Background Location

```bash
npm install react-native-background-geolocation
```

```typescript
import BackgroundGeolocation from 'react-native-background-geolocation';

const startTracking = () => {
  BackgroundGeolocation.ready({
    desiredAccuracy: BackgroundGeolocation.DESIRED_ACCURACY_HIGH,
    distanceFilter: 10,
    stopTimeout: 5,
    debug: false,
    logLevel: BackgroundGeolocation.LOG_LEVEL_VERBOSE,
    stopOnTerminate: false,
    startOnBoot: true
  }).then((state) => {
    BackgroundGeolocation.start();
  });
  
  BackgroundGeolocation.onLocation((location) => {
    console.log('[location]', location);
    // Send to server or save locally
  });
};

const stopTracking = () => {
  BackgroundGeolocation.stop();
};
```

### Live Tracking

```typescript
const LiveTrackingScreen = () => {
  const [positions, setPositions] = useState<Position[]>([]);
  const [isTracking, setIsTracking] = useState(false);
  
  useEffect(() => {
    if (isTracking) {
      const watchId = Geolocation.watchPosition(
        (position) => {
          const newPos = {
            latitude: position.coords.latitude,
            longitude: position.coords.longitude
          };
          
          setPositions(prev => [...prev, newPos]);
        },
        (error) => console.error(error),
        { enableHighAccuracy: true, distanceFilter: 10 }
      );
      
      return () => Geolocation.clearWatch(watchId);
    }
  }, [isTracking]);
  
  return (
    <View style={{ flex: 1 }}>
      <MapView style={styles.map}>
        <Polyline
          coordinates={positions}
          strokeColor="#FF0000"
          strokeWidth={3}
        />
        {positions.length > 0 && (
          <Marker coordinate={positions[positions.length - 1]} />
        )}
      </MapView>
      
      <Button
        title={isTracking ? 'Stop Tracking' : 'Start Tracking'}
        onPress={() => setIsTracking(!isTracking)}
      />
    </View>
  );
};
```

---

## 9. Geofencing

### Setup Geofence

```typescript
import BackgroundGeolocation from 'react-native-background-geolocation';

const addGeofence = (id: string, latitude: number, longitude: number, radius: number) => {
  BackgroundGeolocation.addGeofence({
    identifier: id,
    radius: radius,
    latitude: latitude,
    longitude: longitude,
    notifyOnEntry: true,
    notifyOnExit: true
  }).then(() => {
    console.log('Geofence added');
  });
};

BackgroundGeolocation.onGeofence((geofence) => {
  console.log('[geofence]', geofence.action, geofence.identifier);
  
  if (geofence.action === 'ENTER') {
    // User entered geofence
    PushNotification.localNotification({
      title: 'Welcome!',
      message: `You've entered ${geofence.identifier}`
    });
  } else if (geofence.action === 'EXIT') {
    // User exited geofence
    PushNotification.localNotification({
      title: 'Goodbye!',
      message: `You've left ${geofence.identifier}`
    });
  }
});
```

---

## 10. Best Practices

1. **Request appropriate permissions** - only what you need (WhenInUse vs Always)
2. **Use appropriate accuracy** - HIGH for navigation, LOW for weather
3. **Implement distance filter** - avoid unnecessary updates
4. **Cache location data** - reduce API calls
5. **Handle offline scenarios** - queue location updates
6. **Optimize battery** - stop tracking when not needed
7. **Test on real devices** - GPS works differently on emulators
8. **Respect user privacy** - be transparent about location usage
9. **Implement fallback** - handle GPS not available
10. **Monitor battery usage** - adjust settings based on battery level

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Current Location

App per mostrare posizione:
- Get current location
- Show on map
- Display address
- Calculate distance from fixed point
- Show accuracy

### Esercizio 2: 🟡 Places Finder

App per trovare luoghi:
- Search places by name
- Show markers on map
- Custom markers per categoria
- Info callout
- Directions to place

### Esercizio 3: 🟡 Running Tracker

Fitness tracker:
- Track running route
- Show polyline
- Calculate distance
- Calculate speed
- Show statistics

### Esercizio 4: 🔴 Delivery App

App delivery con:
- Real-time courier tracking
- Multiple stops markers
- Optimized route polyline
- ETA calculation
- Geofencing for delivery zones

### Esercizio 5: 🔴 Location-based Social

Social app con:
- Show friends on map
- Cluster markers
- Check-in feature
- Nearby places
- Geofencing notifications
- Background location tracking

---

## 📝 Quiz

1. **Geolocation accuracy HIGH significa:**
   - [ ] Più veloce
   - [x] Più preciso ma più batteria
   - [ ] Solo WiFi
   - [ ] Solo GPS

2. **Polyline è usato per:**
   - [ ] Marker
   - [x] Percorso/route
   - [ ] Area
   - [ ] Cerchio

3. **Geocoding converte:**
   - [x] Indirizzo in coordinate
   - [ ] Coordinate in metri
   - [ ] Metri in chilometri
   - [ ] Tempo in distanza

4. **Background location richiede:**
   - [ ] Solo permission WhenInUse
   - [x] Permission Always
   - [ ] Nessuna permission
   - [ ] Solo su iOS

5. **Geofencing è utile per:**
   - [ ] Calcolare distanze
   - [ ] Mostrare mappe
   - [x] Notifiche quando entri/esci area
   - [ ] Geocoding

**Risposte**: 1-b, 2-b, 3-a, 4-b, 5-c

---

## 🔗 Risorse

- [React Native Maps](https://github.com/react-native-maps/react-native-maps)
- [Geolocation Docs](https://github.com/react-native-community/react-native-geolocation)
- [Background Geolocation](https://github.com/transistorsoft/react-native-background-geolocation)
- [Google Maps API](https://developers.google.com/maps/documentation)

---

## ✅ Checklist

- [ ] Geolocation implementato
- [ ] Map integration funzionante
- [ ] Markers custom
- [ ] Polylines per routes
- [ ] Geocoding/reverse geocoding
- [ ] Distance calculation
- [ ] Directions API
- [ ] Location tracking
- [ ] Permissions gestiti
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📍 **Geolocation** per posizione GPS
2. 🗺️ **React Native Maps** per mappe interattive
3. 📌 **Markers** per punti interesse
4. 🛣️ **Polyline** per percorsi
5. 🔄 **Geocoding** indirizzo ↔ coordinate
6. 📏 **Distance** con Haversine
7. 🧭 **Directions API** per navigazione
8. 🏃 **Tracking** per movimento
9. 📍 **Geofencing** per aree
10. 🔋 **Battery** ottimizza consumo

---

**Fantastico! 🎉** Ora padroneggi mappe e geolocalizzazione! Prossimo: Notifiche Push!

[← Precedente: Fotocamera e Galleria](15_Fotocamera_Galleria.md) | [Torna all'Indice](../README.md) | [Prossimo: Notifiche Push →](17_Notifiche_Push.md)
