# Capitolo 8: Liste e Performance

[← Precedente: Navigazione](07_Navigazione.md) | [Torna all'Indice](../README.md) | [Prossimo: Animazioni e Gestures →](09_Animazioni_Gestures.md)

---

## Descrizione

Le liste sono uno dei componenti più comuni nelle app mobile. React Native offre componenti ottimizzati come FlatList e SectionList che rendono virtualizzate solo le righe visibili, garantendo performance eccellenti anche con migliaia di elementi.

In questo capitolo imparerai a creare liste performanti, implementare pull-to-refresh, infinite scroll, ottimizzare rendering, e gestire grandi dataset senza compromettere l'esperienza utente.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Utilizzare FlatList per liste semplici
- [ ] Implementare SectionList per liste con sezioni
- [ ] Ottimizzare performance con keyExtractor e getItemLayout
- [ ] Implementare pull-to-refresh
- [ ] Creare infinite scroll con pagination
- [ ] Usare React.memo e useCallback efficacemente
- [ ] Gestire empty states e loading states
- [ ] Implementare search e filtering
- [ ] Usare VirtualizedList per casi avanzati
- [ ] Debugging performance issues

## 📚 Argomenti Teorici

1. [FlatList Base](#1-flatlist-base)
2. [Ottimizzazione FlatList](#2-ottimizzazione)
3. [SectionList](#3-sectionlist)
4. [Pull to Refresh](#4-pull-to-refresh)
5. [Infinite Scroll](#5-infinite-scroll)
6. [Search e Filtering](#6-search-filtering)
7. [Empty e Loading States](#7-empty-loading-states)
8. [React.memo e Optimization](#8-react-memo)
9. [VirtualizedList](#9-virtualizedlist)
10. [Performance Best Practices](#10-best-practices)

---

## 1. FlatList Base

### FlatList Semplice

```typescript
import { FlatList, Text, View } from 'react-native';

interface Item {
  id: string;
  title: string;
}

const data: Item[] = [
  { id: '1', title: 'Item 1' },
  { id: '2', title: 'Item 2' },
  { id: '3', title: 'Item 3' },
];

const App = () => {
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => (
        <View style={styles.item}>
          <Text>{item.title}</Text>
        </View>
      )}
      keyExtractor={item => item.id}
    />
  );
};
```

### RenderItem Component

```typescript
interface ItemProps {
  item: Item;
  index: number;
}

const ItemComponent = ({ item, index }: ItemProps) => {
  return (
    <View style={styles.item}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.index}>#{index}</Text>
    </View>
  );
};

<FlatList
  data={data}
  renderItem={({ item, index }) => (
    <ItemComponent item={item} index={index} />
  )}
  keyExtractor={item => item.id}
/>
```

### Horizontal List

```typescript
<FlatList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  horizontal
  showsHorizontalScrollIndicator={false}
  contentContainerStyle={{ paddingHorizontal: 10 }}
/>
```

### Multi-Column

```typescript
<FlatList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  numColumns={2}
  keyExtractor={item => item.id}
  columnWrapperStyle={{ justifyContent: 'space-between' }}
/>
```

### Separator

```typescript
<FlatList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  ItemSeparatorComponent={() => (
    <View style={{ height: 1, backgroundColor: '#ccc' }} />
  )}
/>
```

### Header e Footer

```typescript
<FlatList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  ListHeaderComponent={() => (
    <View style={styles.header}>
      <Text>Header</Text>
    </View>
  )}
  ListFooterComponent={() => (
    <View style={styles.footer}>
      <Text>Footer</Text>
    </View>
  )}
/>
```

---

## 2. Ottimizzazione

### keyExtractor

```typescript
// ❌ Bad - index non è stable
<FlatList
  data={data}
  keyExtractor={(item, index) => index.toString()}
/>

// ✅ Good - usa ID unico
<FlatList
  data={data}
  keyExtractor={item => item.id}
/>
```

### getItemLayout (Per Fixed Height Items)

```typescript
const ITEM_HEIGHT = 80;

<FlatList
  data={data}
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index
  })}
  renderItem={({ item }) => (
    <View style={{ height: ITEM_HEIGHT }}>
      <Text>{item.title}</Text>
    </View>
  )}
/>
```

### removeClippedSubviews

```typescript
<FlatList
  data={data}
  removeClippedSubviews={true}  // Rimuove view off-screen (Android)
  renderItem={({ item }) => <ItemComponent item={item} />}
/>
```

### windowSize

```typescript
<FlatList
  data={data}
  windowSize={10}  // Default 21. Più basso = meno memoria, più scroll lag
  maxToRenderPerBatch={10}  // Quanti item rendere per batch
  updateCellsBatchingPeriod={50}  // Millisecondi tra batches
  initialNumToRender={10}  // Items da rendere inizialmente
  renderItem={({ item }) => <ItemComponent item={item} />}
/>
```

### React.memo per Items

```typescript
interface ItemProps {
  item: Item;
  onPress: (id: string) => void;
}

const ItemComponent = React.memo(({ item, onPress }: ItemProps) => {
  console.log('Rendering:', item.id);  // Should log only once
  
  return (
    <TouchableOpacity onPress={() => onPress(item.id)}>
      <Text>{item.title}</Text>
    </TouchableOpacity>
  );
}, (prevProps, nextProps) => {
  // Custom comparison
  return prevProps.item.id === nextProps.item.id;
});

const App = () => {
  // ❌ Bad - crea nuova funzione ogni render
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => (
        <ItemComponent
          item={item}
          onPress={(id) => console.log(id)}
        />
      )}
    />
  );
  
  // ✅ Good - usa useCallback
  const handlePress = useCallback((id: string) => {
    console.log(id);
  }, []);
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => (
        <ItemComponent item={item} onPress={handlePress} />
      )}
    />
  );
};
```

---

## 3. SectionList

### Basic SectionList

```typescript
import { SectionList } from 'react-native';

interface Item {
  id: string;
  name: string;
}

interface Section {
  title: string;
  data: Item[];
}

const sections: Section[] = [
  {
    title: 'A',
    data: [
      { id: '1', name: 'Apple' },
      { id: '2', name: 'Avocado' }
    ]
  },
  {
    title: 'B',
    data: [
      { id: '3', name: 'Banana' },
      { id: '4', name: 'Blueberry' }
    ]
  }
];

const App = () => {
  return (
    <SectionList
      sections={sections}
      renderItem={({ item }) => (
        <View style={styles.item}>
          <Text>{item.name}</Text>
        </View>
      )}
      renderSectionHeader={({ section }) => (
        <View style={styles.sectionHeader}>
          <Text style={styles.sectionTitle}>{section.title}</Text>
        </View>
      )}
      keyExtractor={item => item.id}
    />
  );
};

const styles = StyleSheet.create({
  sectionHeader: {
    backgroundColor: '#f4f4f4',
    padding: 10
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold'
  },
  item: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee'
  }
});
```

### Sticky Headers

```typescript
<SectionList
  sections={sections}
  stickySectionHeadersEnabled={true}  // Default true su iOS
  renderSectionHeader={({ section }) => (
    <View style={styles.stickyHeader}>
      <Text>{section.title}</Text>
    </View>
  )}
/>
```

### Section Footer

```typescript
<SectionList
  sections={sections}
  renderSectionFooter={({ section }) => (
    <View style={styles.footer}>
      <Text>{section.data.length} items</Text>
    </View>
  )}
/>
```

---

## 4. Pull to Refresh

### RefreshControl

```typescript
import { RefreshControl } from 'react-native';

const App = () => {
  const [refreshing, setRefreshing] = useState(false);
  const [data, setData] = useState<Item[]>([]);
  
  const onRefresh = useCallback(async () => {
    setRefreshing(true);
    
    try {
      const newData = await fetchData();
      setData(newData);
    } catch (error) {
      console.error(error);
    } finally {
      setRefreshing(false);
    }
  }, []);
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <ItemComponent item={item} />}
      refreshControl={
        <RefreshControl
          refreshing={refreshing}
          onRefresh={onRefresh}
          tintColor="#007AFF"  // iOS spinner color
          colors={['#007AFF']}  // Android colors
          title="Pull to refresh"  // iOS
          titleColor="#999"
        />
      }
    />
  );
};
```

---

## 5. Infinite Scroll

### Basic Pagination

```typescript
const App = () => {
  const [data, setData] = useState<Item[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  
  const loadMore = async () => {
    if (loading || !hasMore) return;
    
    setLoading(true);
    
    try {
      const newData = await fetchData(page);
      
      if (newData.length === 0) {
        setHasMore(false);
      } else {
        setData(prev => [...prev, ...newData]);
        setPage(prev => prev + 1);
      }
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };
  
  useEffect(() => {
    loadMore();
  }, []);
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <ItemComponent item={item} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}  // Trigger at 50% from bottom
      ListFooterComponent={() => (
        loading ? <ActivityIndicator /> : null
      )}
    />
  );
};
```

### Custom Hook

```typescript
interface UsePaginationOptions<T> {
  fetchFunction: (page: number) => Promise<T[]>;
  initialPage?: number;
}

const usePagination = <T,>({
  fetchFunction,
  initialPage = 1
}: UsePaginationOptions<T>) => {
  const [data, setData] = useState<T[]>([]);
  const [page, setPage] = useState(initialPage);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  
  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    
    setLoading(true);
    
    try {
      const newData = await fetchFunction(page);
      
      if (newData.length === 0) {
        setHasMore(false);
      } else {
        setData(prev => [...prev, ...newData]);
        setPage(prev => prev + 1);
      }
    } finally {
      setLoading(false);
    }
  }, [page, loading, hasMore, fetchFunction]);
  
  const refresh = useCallback(async () => {
    setRefreshing(true);
    setPage(initialPage);
    setHasMore(true);
    
    try {
      const newData = await fetchFunction(initialPage);
      setData(newData);
      setPage(initialPage + 1);
    } finally {
      setRefreshing(false);
    }
  }, [fetchFunction, initialPage]);
  
  useEffect(() => {
    loadMore();
  }, []);
  
  return { data, loading, refreshing, hasMore, loadMore, refresh };
};

// Uso
const App = () => {
  const { data, loading, refreshing, loadMore, refresh } = usePagination({
    fetchFunction: fetchPosts
  });
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <PostItem item={item} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={refresh} />
      }
      ListFooterComponent={() => loading ? <ActivityIndicator /> : null}
    />
  );
};
```

---

## 6. Search e Filtering

### Search Implementation

```typescript
const App = () => {
  const [data, setData] = useState<Item[]>([]);
  const [filteredData, setFilteredData] = useState<Item[]>([]);
  const [searchQuery, setSearchQuery] = useState('');
  
  useEffect(() => {
    // Load data
    fetchData().then(setData);
  }, []);
  
  useEffect(() => {
    if (searchQuery === '') {
      setFilteredData(data);
    } else {
      const filtered = data.filter(item =>
        item.title.toLowerCase().includes(searchQuery.toLowerCase())
      );
      setFilteredData(filtered);
    }
  }, [searchQuery, data]);
  
  return (
    <View style={{ flex: 1 }}>
      <TextInput
        value={searchQuery}
        onChangeText={setSearchQuery}
        placeholder="Search..."
        style={styles.searchInput}
      />
      
      <FlatList
        data={filteredData}
        renderItem={({ item }) => <ItemComponent item={item} />}
        keyExtractor={item => item.id}
      />
    </View>
  );
};
```

### Debounced Search

```typescript
import { useEffect, useState } from 'react';

const useDebounce = <T,>(value: T, delay: number): T => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
};

const App = () => {
  const [searchQuery, setSearchQuery] = useState('');
  const debouncedQuery = useDebounce(searchQuery, 500);
  const [results, setResults] = useState<Item[]>([]);
  
  useEffect(() => {
    if (debouncedQuery) {
      searchAPI(debouncedQuery).then(setResults);
    } else {
      setResults([]);
    }
  }, [debouncedQuery]);
  
  return (
    <View>
      <TextInput
        value={searchQuery}
        onChangeText={setSearchQuery}
        placeholder="Search..."
      />
      
      <FlatList
        data={results}
        renderItem={({ item }) => <ItemComponent item={item} />}
      />
    </View>
  );
};
```

---

## 7. Empty e Loading States

### ListEmptyComponent

```typescript
<FlatList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  ListEmptyComponent={() => (
    <View style={styles.emptyContainer}>
      <Text style={styles.emptyText}>No items found</Text>
      <Button title="Reload" onPress={reload} />
    </View>
  )}
/>
```

### Complete State Management

```typescript
const App = () => {
  const [data, setData] = useState<Item[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    loadData();
  }, []);
  
  const loadData = async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await fetchData();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) {
    return (
      <View style={styles.centered}>
        <ActivityIndicator size="large" />
      </View>
    );
  }
  
  if (error) {
    return (
      <View style={styles.centered}>
        <Text>Error: {error}</Text>
        <Button title="Retry" onPress={loadData} />
      </View>
    );
  }
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <ItemComponent item={item} />}
      ListEmptyComponent={() => (
        <View style={styles.centered}>
          <Text>No data available</Text>
        </View>
      )}
    />
  );
};
```

---

## 8. React.memo

### Quando Usare React.memo

```typescript
// ❌ Without memo - re-renders on every parent render
const ItemComponent = ({ item }: { item: Item }) => {
  console.log('Rendering:', item.id);
  return <Text>{item.title}</Text>;
};

// ✅ With memo - only re-renders if props change
const ItemComponent = React.memo(({ item }: { item: Item }) => {
  console.log('Rendering:', item.id);
  return <Text>{item.title}</Text>;
});
```

### useCallback per Callbacks

```typescript
const App = () => {
  const [data, setData] = useState<Item[]>([]);
  
  // ❌ Bad - new function every render
  const handleDelete = (id: string) => {
    setData(prev => prev.filter(item => item.id !== id));
  };
  
  // ✅ Good - memoized function
  const handleDelete = useCallback((id: string) => {
    setData(prev => prev.filter(item => item.id !== id));
  }, []);
  
  return (
    <FlatList
      data={data}
      renderItem={({ item }) => (
        <ItemComponent item={item} onDelete={handleDelete} />
      )}
    />
  );
};
```

### useMemo per Computed Data

```typescript
const App = () => {
  const [data, setData] = useState<Item[]>([]);
  const [filter, setFilter] = useState('all');
  
  // ✅ Memoize filtered data
  const filteredData = useMemo(() => {
    console.log('Computing filtered data');
    
    if (filter === 'all') return data;
    return data.filter(item => item.category === filter);
  }, [data, filter]);
  
  return (
    <FlatList
      data={filteredData}
      renderItem={({ item }) => <ItemComponent item={item} />}
    />
  );
};
```

---

## 9. VirtualizedList

### Custom VirtualizedList

```typescript
import { VirtualizedList } from 'react-native';

const getItem = (data: Item[], index: number) => data[index];
const getItemCount = (data: Item[]) => data.length;

<VirtualizedList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  keyExtractor={item => item.id}
  getItemCount={getItemCount}
  getItem={getItem}
  initialNumToRender={10}
  maxToRenderPerBatch={10}
  windowSize={10}
/>
```

---

## 10. Best Practices

### 1. Performance Checklist

- ✅ Usa `keyExtractor` con ID stabili
- ✅ `getItemLayout` per fixed heights
- ✅ `React.memo` per item components
- ✅ `useCallback` per callbacks
- ✅ `removeClippedSubviews` su Android
- ✅ Ottimizza `windowSize` e batch settings
- ✅ Avoid inline functions in renderItem
- ✅ Avoid complex logic in renderItem

### 2. FlatList vs ScrollView

```typescript
// ❌ Don't use ScrollView for long lists
<ScrollView>
  {data.map(item => <ItemComponent key={item.id} item={item} />)}
</ScrollView>

// ✅ Use FlatList
<FlatList
  data={data}
  renderItem={({ item }) => <ItemComponent item={item} />}
  keyExtractor={item => item.id}
/>
```

### 3. Measure Performance

```typescript
import { InteractionManager } from 'react-native';

const ItemComponent = ({ item }: { item: Item }) => {
  useEffect(() => {
    const start = Date.now();
    
    return () => {
      const end = Date.now();
      console.log(`Render time: ${end - start}ms`);
    };
  }, []);
  
  return <View>{/* ... */}</View>;
};
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Simple List

Crea una FlatList con:
- 50 items
- Separator tra items
- Header e Footer
- Horizontal scroll

### Esercizio 2: 🟡 Contacts List

Implementa lista contatti con:
- SectionList alfabetica (A-Z)
- Search bar
- Sticky section headers
- Item press per details

### Esercizio 3: 🟡 Infinite Scroll

Crea feed con:
- Pagination (10 items per pagina)
- Pull-to-refresh
- Loading indicator in footer
- Empty state

### Esercizio 4: 🔴 Optimized Gallery

Gallery di immagini con:
- Grid 3 colonne
- Lazy loading images
- React.memo optimization
- Smooth scrolling (60fps)
- Image caching

### Esercizio 5: 🔴 Advanced Search

App di ricerca con:
- Debounced search
- Multiple filters
- Sorting options
- Results count
- Performance monitoring

---

## 📝 Quiz

1. **Differenza FlatList vs ScrollView?**
   - [ ] Sono identici
   - [x] FlatList virtualizza, ScrollView no
   - [ ] FlatList è deprecato
   - [ ] ScrollView è più veloce

2. **getItemLayout migliora performance?**
   - [x] Sì, se items hanno altezza fissa
   - [ ] No, non serve
   - [ ] Solo su Android
   - [ ] Solo per SectionList

3. **Quando usare React.memo?**
   - [ ] Sempre
   - [ ] Mai
   - [x] Quando item component è puro
   - [ ] Solo per performance issues

4. **onEndReachedThreshold=0.5 significa?**
   - [ ] 50 pixel da fine
   - [x] 50% dello schermo da fine
   - [ ] 5 items da fine
   - [ ] 0.5 secondi delay

5. **keyExtractor dovrebbe usare:**
   - [ ] Index
   - [x] ID unico e stabile
   - [ ] Random number
   - [ ] Non importa

**Risposte**: 1-b, 2-a, 3-c, 4-b, 5-b

---

## 🔗 Risorse

- [FlatList Docs](https://reactnative.dev/docs/flatlist)
- [Performance Overview](https://reactnative.dev/docs/performance)
- [React.memo](https://react.dev/reference/react/memo)

---

## ✅ Checklist

- [ ] FlatList implementata
- [ ] SectionList creata
- [ ] Pull-to-refresh funziona
- [ ] Infinite scroll implementato
- [ ] React.memo utilizzato
- [ ] Search con debounce
- [ ] Empty states gestiti
- [ ] Performance ottimizzate
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📜 **FlatList** per liste lunghe
2. 🔑 **keyExtractor** con ID stabili
3. 📏 **getItemLayout** per fixed heights
4. 💾 **React.memo** per item components
5. 🔄 **useCallback** per callbacks
6. 🔍 **Debounce** per search
7. ♾️ **Pagination** per infinite scroll
8. ⚡ **Optimize** per 60fps

---

**Perfetto! 🎉** Ora sai creare liste ultra-performanti. Prossimo: Animazioni e Gestures!

[← Precedente: Navigazione](07_Navigazione.md) | [Torna all'Indice](../README.md) | [Prossimo: Animazioni e Gestures →](09_Animazioni_Gestures.md)
