# Capitolo 14: GraphQL e Apollo Client

[← Precedente: Persistenza Dati](13_Persistenza_Dati.md) | [Torna all'Indice](../README.md) | [Prossimo: Fotocamera e Galleria →](../04%20-%20Funzionalità%20Native/15_Fotocamera_Galleria.md)

---

## Descrizione

GraphQL è un linguaggio di query per API che permette al client di richiedere esattamente i dati necessari. Apollo Client è la libreria più popolare per integrare GraphQL in React Native, offrendo caching intelligente, gestione dello state, optimistic UI, e molto altro.

In questo capitolo imparerai i fondamenti di GraphQL, come integrare Apollo Client, eseguire queries e mutations, gestire cache e subscriptions, implementare optimistic updates, e ottimizzare performance.

## 🎯 Obiettivi di Apprendimento

Alla fine di questo capitolo sarai in grado di:

- [ ] Comprendere GraphQL schema e queries
- [ ] Configurare Apollo Client
- [ ] Eseguire queries con useQuery
- [ ] Eseguire mutations con useMutation
- [ ] Gestire cache Apollo
- [ ] Implementare optimistic updates
- [ ] Usare subscriptions real-time
- [ ] Gestire pagination
- [ ] Implementare local state
- [ ] Ottimizzare performance queries

## 📚 Argomenti Teorici

1. [GraphQL Basics](#1-graphql-basics)
2. [Apollo Client Setup](#2-apollo-setup)
3. [Queries](#3-queries)
4. [Mutations](#4-mutations)
5. [Cache Management](#5-cache)
6. [Optimistic Updates](#6-optimistic)
7. [Subscriptions](#7-subscriptions)
8. [Pagination](#8-pagination)
9. [Local State](#9-local-state)
10. [Performance](#10-performance)

---

## 1. GraphQL Basics

### Schema Example

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  createdAt: String!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
}

type Query {
  users: [User!]!
  user(id: ID!): User
  posts(limit: Int, offset: Int): [Post!]!
  post(id: ID!): Post
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String, email: String): User!
  deleteUser(id: ID!): Boolean!
  
  createPost(title: String!, content: String!, authorId: ID!): Post!
  updatePost(id: ID!, title: String, content: String): Post!
  deletePost(id: ID!): Boolean!
}

type Subscription {
  postAdded: Post!
  commentAdded(postId: ID!): Comment!
}
```

### Query Example

```graphql
# Get all users with their posts
query GetUsers {
  users {
    id
    name
    email
    posts {
      id
      title
    }
  }
}

# Get specific user
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
      content
      comments {
        id
        text
        author {
          name
        }
      }
    }
  }
}

# Get posts with pagination
query GetPosts($limit: Int!, $offset: Int!) {
  posts(limit: $limit, offset: $offset) {
    id
    title
    author {
      name
    }
    createdAt
  }
}
```

### Mutation Example

```graphql
mutation CreateUser($name: String!, $email: String!) {
  createUser(name: $name, email: $email) {
    id
    name
    email
  }
}

mutation UpdatePost($id: ID!, $title: String!) {
  updatePost(id: $id, title: $title) {
    id
    title
    content
  }
}
```

---

## 2. Apollo Setup

### Installation

```bash
npm install @apollo/client graphql
```

### Client Configuration

```typescript
import { ApolloClient, InMemoryCache, HttpLink } from '@apollo/client';

const httpLink = new HttpLink({
  uri: 'https://api.example.com/graphql'
});

export const client = new ApolloClient({
  link: httpLink,
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          posts: {
            merge(existing, incoming) {
              return incoming;
            }
          }
        }
      }
    }
  })
});
```

### Provider Setup

```typescript
import { ApolloProvider } from '@apollo/client';
import { client } from './apollo/client';

const App = () => {
  return (
    <ApolloProvider client={client}>
      <Navigation />
    </ApolloProvider>
  );
};
```

### Auth Link

```typescript
import { ApolloClient, InMemoryCache, HttpLink, ApolloLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';
import AsyncStorage from '@react-native-async-storage/async-storage';

const httpLink = new HttpLink({
  uri: 'https://api.example.com/graphql'
});

const authLink = setContext(async (_, { headers }) => {
  const token = await AsyncStorage.getItem('token');
  
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : ''
    }
  };
});

export const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
});
```

---

## 3. Queries

### Basic Query

```typescript
import { gql, useQuery } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

const UsersScreen = () => {
  const { data, loading, error, refetch } = useQuery(GET_USERS);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error.message}</Text>;
  
  return (
    <FlatList
      data={data.users}
      renderItem={({ item }) => (
        <View>
          <Text>{item.name}</Text>
          <Text>{item.email}</Text>
        </View>
      )}
      refreshControl={
        <RefreshControl refreshing={loading} onRefresh={refetch} />
      }
    />
  );
};
```

### Query with Variables

```typescript
const GET_USER = gql`
  query GetUser($userId: ID!) {
    user(id: $userId) {
      id
      name
      email
      posts {
        id
        title
        content
      }
    }
  }
`;

const UserDetailScreen = ({ route }) => {
  const { userId } = route.params;
  
  const { data, loading, error } = useQuery(GET_USER, {
    variables: { userId }
  });
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error.message}</Text>;
  
  const user = data.user;
  
  return (
    <ScrollView>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
      
      <Text style={styles.heading}>Posts</Text>
      {user.posts.map(post => (
        <View key={post.id}>
          <Text>{post.title}</Text>
          <Text>{post.content}</Text>
        </View>
      ))}
    </ScrollView>
  );
};
```

### Lazy Query

```typescript
import { useLazyQuery } from '@apollo/client';

const SEARCH_USERS = gql`
  query SearchUsers($query: String!) {
    searchUsers(query: $query) {
      id
      name
      email
    }
  }
`;

const SearchScreen = () => {
  const [search, setSearch] = useState('');
  const [searchUsers, { data, loading }] = useLazyQuery(SEARCH_USERS);
  
  const handleSearch = () => {
    if (search.length > 2) {
      searchUsers({ variables: { query: search } });
    }
  };
  
  return (
    <View>
      <TextInput
        value={search}
        onChangeText={setSearch}
        onSubmitEditing={handleSearch}
        placeholder="Search users..."
      />
      
      {loading && <ActivityIndicator />}
      
      {data && (
        <FlatList
          data={data.searchUsers}
          renderItem={({ item }) => <UserItem user={item} />}
        />
      )}
    </View>
  );
};
```

### Polling

```typescript
const { data, startPolling, stopPolling } = useQuery(GET_POSTS, {
  pollInterval: 5000 // Poll every 5 seconds
});

useEffect(() => {
  startPolling(5000);
  
  return () => {
    stopPolling();
  };
}, []);
```

---

## 4. Mutations

### Basic Mutation

```typescript
import { gql, useMutation } from '@apollo/client';

const CREATE_USER = gql`
  mutation CreateUser($name: String!, $email: String!) {
    createUser(name: $name, email: $email) {
      id
      name
      email
    }
  }
`;

const CreateUserScreen = () => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  const [createUser, { loading, error }] = useMutation(CREATE_USER, {
    onCompleted: (data) => {
      Alert.alert('Success', `User ${data.createUser.name} created!`);
      navigation.goBack();
    },
    onError: (error) => {
      Alert.alert('Error', error.message);
    }
  });
  
  const handleSubmit = () => {
    createUser({
      variables: { name, email }
    });
  };
  
  return (
    <View>
      <TextInput
        value={name}
        onChangeText={setName}
        placeholder="Name"
      />
      <TextInput
        value={email}
        onChangeText={setEmail}
        placeholder="Email"
      />
      <Button
        title="Create User"
        onPress={handleSubmit}
        disabled={loading}
      />
      {error && <Text>Error: {error.message}</Text>}
    </View>
  );
};
```

### Update Query After Mutation

```typescript
const [createPost] = useMutation(CREATE_POST, {
  update(cache, { data: { createPost } }) {
    cache.modify({
      fields: {
        posts(existingPosts = []) {
          const newPostRef = cache.writeFragment({
            data: createPost,
            fragment: gql`
              fragment NewPost on Post {
                id
                title
                content
              }
            `
          });
          return [newPostRef, ...existingPosts];
        }
      }
    });
  }
});
```

### Refetch Queries

```typescript
const [deletePost] = useMutation(DELETE_POST, {
  refetchQueries: [
    { query: GET_POSTS },
    { query: GET_USER, variables: { userId } }
  ]
});
```

---

## 5. Cache

### Read from Cache

```typescript
const user = client.readFragment({
  id: 'User:1',
  fragment: gql`
    fragment UserData on User {
      id
      name
      email
    }
  `
});
```

### Write to Cache

```typescript
client.writeFragment({
  id: 'User:1',
  fragment: gql`
    fragment UserData on User {
      id
      name
      email
    }
  `,
  data: {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    __typename: 'User'
  }
});
```

### Cache Policies

```typescript
const { data } = useQuery(GET_USER, {
  variables: { userId },
  fetchPolicy: 'cache-first' // Options: cache-first, network-only, cache-only, no-cache, cache-and-network
});
```

---

## 6. Optimistic Updates

### Optimistic Response

```typescript
const UPDATE_POST = gql`
  mutation UpdatePost($id: ID!, $title: String!) {
    updatePost(id: $id, title: $title) {
      id
      title
      content
    }
  }
`;

const [updatePost] = useMutation(UPDATE_POST, {
  optimisticResponse: ({ id, title }) => ({
    __typename: 'Mutation',
    updatePost: {
      __typename: 'Post',
      id,
      title,
      content: 'Loading...' // Will be replaced by server response
    }
  })
});

// Uso
updatePost({
  variables: { id: '1', title: 'New Title' }
});
```

### Optimistic UI Example

```typescript
const TOGGLE_TODO = gql`
  mutation ToggleTodo($id: ID!) {
    toggleTodo(id: $id) {
      id
      completed
    }
  }
`;

const TodoItem = ({ todo }) => {
  const [toggleTodo] = useMutation(TOGGLE_TODO, {
    optimisticResponse: {
      __typename: 'Mutation',
      toggleTodo: {
        __typename: 'Todo',
        id: todo.id,
        completed: !todo.completed
      }
    }
  });
  
  return (
    <TouchableOpacity onPress={() => toggleTodo({ variables: { id: todo.id } })}>
      <Text style={todo.completed && styles.completed}>
        {todo.title}
      </Text>
    </TouchableOpacity>
  );
};
```

---

## 7. Subscriptions

### WebSocket Setup

```bash
npm install graphql-ws
```

```typescript
import { ApolloClient, InMemoryCache, HttpLink, split } from '@apollo/client';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { getMainDefinition } from '@apollo/client/utilities';
import { createClient } from 'graphql-ws';

const httpLink = new HttpLink({
  uri: 'https://api.example.com/graphql'
});

const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://api.example.com/graphql'
  })
);

const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  httpLink
);

export const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache()
});
```

### Using Subscriptions

```typescript
import { gql, useSubscription } from '@apollo/client';

const POST_ADDED = gql`
  subscription PostAdded {
    postAdded {
      id
      title
      content
      author {
        name
      }
    }
  }
`;

const PostsScreen = () => {
  const { data: postsData } = useQuery(GET_POSTS);
  
  const { data: newPostData } = useSubscription(POST_ADDED, {
    onSubscriptionData: ({ client, subscriptionData }) => {
      Alert.alert('New Post', subscriptionData.data.postAdded.title);
    }
  });
  
  return (
    <FlatList
      data={postsData?.posts}
      renderItem={({ item }) => <PostItem post={item} />}
    />
  );
};
```

---

## 8. Pagination

### Offset-based Pagination

```typescript
const GET_POSTS = gql`
  query GetPosts($limit: Int!, $offset: Int!) {
    posts(limit: $limit, offset: $offset) {
      id
      title
      content
    }
  }
`;

const PostsScreen = () => {
  const [page, setPage] = useState(0);
  const limit = 10;
  
  const { data, loading, fetchMore } = useQuery(GET_POSTS, {
    variables: { limit, offset: 0 }
  });
  
  const loadMore = () => {
    fetchMore({
      variables: {
        offset: (page + 1) * limit
      },
      updateQuery: (prev, { fetchMoreResult }) => {
        if (!fetchMoreResult) return prev;
        return {
          posts: [...prev.posts, ...fetchMoreResult.posts]
        };
      }
    });
    setPage(page + 1);
  };
  
  return (
    <FlatList
      data={data?.posts}
      renderItem={({ item }) => <PostItem post={item} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={loading && <ActivityIndicator />}
    />
  );
};
```

### Cursor-based Pagination

```typescript
const GET_POSTS = gql`
  query GetPosts($after: String) {
    posts(first: 10, after: $after) {
      edges {
        node {
          id
          title
          content
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

const PostsScreen = () => {
  const { data, loading, fetchMore } = useQuery(GET_POSTS);
  
  const loadMore = () => {
    if (!data?.posts.pageInfo.hasNextPage) return;
    
    fetchMore({
      variables: {
        after: data.posts.pageInfo.endCursor
      }
    });
  };
  
  return (
    <FlatList
      data={data?.posts.edges.map(edge => edge.node)}
      renderItem={({ item }) => <PostItem post={item} />}
      onEndReached={loadMore}
    />
  );
};
```

---

## 9. Local State

### Reactive Variables

```typescript
import { makeVar, useReactiveVar } from '@apollo/client';

export const cartItemsVar = makeVar<CartItem[]>([]);

// Add to cart
export const addToCart = (item: CartItem) => {
  const currentItems = cartItemsVar();
  cartItemsVar([...currentItems, item]);
};

// Remove from cart
export const removeFromCart = (itemId: string) => {
  const currentItems = cartItemsVar();
  cartItemsVar(currentItems.filter(item => item.id !== itemId));
};

// Uso in component
const CartScreen = () => {
  const cartItems = useReactiveVar(cartItemsVar);
  
  return (
    <FlatList
      data={cartItems}
      renderItem={({ item }) => (
        <View>
          <Text>{item.name}</Text>
          <Button
            title="Remove"
            onPress={() => removeFromCart(item.id)}
          />
        </View>
      )}
    />
  );
};
```

---

## 10. Performance

### Fragment Colocation

```typescript
const POST_FRAGMENT = gql`
  fragment PostData on Post {
    id
    title
    content
    author {
      id
      name
    }
  }
`;

const GET_POSTS = gql`
  ${POST_FRAGMENT}
  query GetPosts {
    posts {
      ...PostData
    }
  }
`;

const PostItem = ({ post }) => {
  // Component uses PostData fragment
  return (
    <View>
      <Text>{post.title}</Text>
      <Text>{post.content}</Text>
      <Text>By {post.author.name}</Text>
    </View>
  );
};
```

### Batch Queries

```typescript
import { BatchHttpLink } from '@apollo/client/link/batch-http';

const batchLink = new BatchHttpLink({
  uri: 'https://api.example.com/graphql',
  batchMax: 5,
  batchInterval: 20
});
```

---

## 💻 Esercizi Pratici

### Esercizio 1: 🟢 Basic GraphQL App

App con queries e mutations:
- List users
- User detail
- Create user
- Update user
- Delete user

### Esercizio 2: 🟡 Blog App with Apollo

Blog completo:
- Posts list con pagination
- Post detail con comments
- Create/edit/delete post
- Add comments
- Cache management

### Esercizio 3: 🟡 Optimistic UI

Todo app con:
- Add todo (optimistic)
- Toggle todo (optimistic)
- Delete todo (optimistic)
- Rollback su errore
- Loading states

### Esercizio 4: 🔴 Real-time Chat

Chat app con subscriptions:
- Messages list
- Send message
- Real-time updates
- Typing indicator
- Online status

### Esercizio 5: 🔴 E-commerce GraphQL

App e-commerce completa:
- Products catalog
- Search and filters
- Cart (local state)
- Orders
- User profile
- Optimistic updates
- Pagination

---

## 📝 Quiz

1. **GraphQL vs REST:**
   - [ ] GraphQL è più lento
   - [x] GraphQL permette di richiedere esattamente i dati necessari
   - [ ] REST è più moderno
   - [ ] Sono identici

2. **Apollo Cache è:**
   - [ ] Disabled by default
   - [x] Intelligente e normalizzato
   - [ ] Solo in-memory
   - [ ] Solo per queries

3. **Optimistic response serve per:**
   - [ ] Performance server
   - [x] Update UI immediato prima di risposta server
   - [ ] Cache invalidation
   - [ ] Debugging

4. **Subscriptions usano:**
   - [ ] HTTP
   - [x] WebSocket
   - [ ] REST
   - [ ] Polling

5. **Reactive variables sono:**
   - [ ] Solo per GraphQL server
   - [x] Local state Apollo senza schema
   - [ ] Deprecated
   - [ ] Solo per subscriptions

**Risposte**: 1-b, 2-b, 3-b, 4-b, 5-b

---

## 🔗 Risorse

- [GraphQL Official Docs](https://graphql.org/)
- [Apollo Client Docs](https://www.apollographql.com/docs/react/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [Apollo React Native Guide](https://www.apollographql.com/docs/react/integrations/react-native/)

---

## ✅ Checklist

- [ ] Apollo Client configurato
- [ ] Queries implementate
- [ ] Mutations funzionanti
- [ ] Cache gestita
- [ ] Optimistic updates implementati
- [ ] Subscriptions configurate
- [ ] Pagination implementata
- [ ] Local state gestito
- [ ] Performance ottimizzate
- [ ] 3+ esercizi completati

---

## 📌 Punti Chiave

1. 📊 **GraphQL** richiedi solo dati necessari
2. 🚀 **Apollo Client** con cache intelligente
3. 🔍 **useQuery** per fetch dati
4. ✏️ **useMutation** per modifiche
5. 💾 **Cache** automatico e normalizzato
6. ⚡ **Optimistic UI** per UX immediata
7. 🔄 **Subscriptions** per real-time
8. 📄 **Pagination** con fetchMore
9. 🎯 **Reactive vars** per local state
10. 🛠️ **Fragments** per riusabilità

---

**Straordinario! 🎉** Hai completato la Parte III - Dati e Networking! Prossimo: Funzionalità Native!

[← Precedente: Persistenza Dati](13_Persistenza_Dati.md) | [Torna all'Indice](../README.md) | [Prossimo: Fotocamera e Galleria →](../04%20-%20Funzionalità%20Native/15_Fotocamera_Galleria.md)
