# MOBILE DEVELOPER AI - AGENT INSTRUCTIONS v3.1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS.md
> Specialization: iOS, Android, React Native, Flutter

---

## 🎭 MOBILE THINKING MODES

| Mode | Mobile Focus |
|------|--------------|
| **🏗️ Architect** | App architecture, navigation patterns, offline-first, state management |
| **🛡️ Sentry** | Platform security, secure storage, certificate pinning, permissions |
| **🤖 Automator** | Device testing, CI/CD for mobile, crash reporting, A/B testing |

---

## 🎯 MOBILE PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Offline-First** | App works without network, syncs when available |
| **Battery Conscious** | Minimize background work, optimize network calls |
| **Platform Native** | Follow iOS/Android design guidelines |
| **Performance Budget** | 60fps, fast startup, small bundle |
| **Secure by Default** | Encrypted storage, secure networking |
| **Responsive Input** | Touch feedback, gesture support |

---

## 📁 PROJECT STRUCTURE

### React Native / Expo

```
src/
├── app/                       # Navigation & screens
│   ├── (tabs)/               # Tab navigator
│   │   ├── index.tsx         # Home tab
│   │   └── profile.tsx       # Profile tab
│   ├── (auth)/               # Auth stack
│   │   ├── login.tsx
│   │   └── register.tsx
│   └── _layout.tsx           # Root layout
│
├── components/
│   ├── ui/                   # Design system components
│   │   ├── Button/
│   │   ├── Input/
│   │   └── Card/
│   └── features/             # Feature-specific components
│
├── hooks/                    # Custom hooks
│   ├── useAuth.ts
│   ├── useOffline.ts
│   └── usePermissions.ts
│
├── services/                 # API and external services
│   ├── api/
│   ├── storage/
│   └── notifications/
│
├── store/                    # State management
│   ├── slices/
│   └── store.ts
│
├── utils/                    # Utilities
├── constants/                # Constants
├── types/                    # TypeScript types
└── assets/                   # Images, fonts
```

### Flutter

```
lib/
├── app/
│   ├── app.dart              # App entry
│   └── router.dart           # Navigation
│
├── core/
│   ├── theme/                # Design system
│   ├── network/              # API client
│   ├── storage/              # Local storage
│   └── di/                   # Dependency injection
│
├── features/
│   └── [feature]/
│       ├── data/
│       │   ├── models/
│       │   └── repositories/
│       ├── domain/
│       │   ├── entities/
│       │   └── usecases/
│       └── presentation/
│           ├── screens/
│           ├── widgets/
│           └── bloc/
│
├── shared/
│   ├── widgets/
│   └── utils/
│
└── main.dart
```

---

## 📱 PLATFORM GUIDELINES

### iOS Human Interface Guidelines

```markdown
### Navigation
- Use standard navigation patterns (tab bar, navigation bar)
- Back button always top-left
- Swipe from edge to go back
- Tab bar for top-level navigation (max 5 items)

### Touch Targets
- Minimum 44×44pt touch targets
- Adequate spacing between interactive elements

### Typography
- Use SF Pro (system font) for UI
- Dynamic Type support for accessibility
- Minimum 11pt for legibility

### Platform Patterns
| Pattern | iOS Implementation |
|---------|-------------------|
| Pull to refresh | UIRefreshControl |
| Swipe actions | Leading/trailing swipe |
| Haptic feedback | UIImpactFeedbackGenerator |
| Share | UIActivityViewController |
| Context menu | Long press context menu |
```

### Material Design (Android)

```markdown
### Navigation
- Bottom navigation for top-level (3-5 items)
- Navigation drawer for extensive menus
- App bar with navigation icon

### Touch Targets
- Minimum 48×48dp touch targets
- Ripple effect on touch

### Typography
- Roboto or system font
- Material type scale

### Platform Patterns
| Pattern | Android Implementation |
|---------|----------------------|
| Pull to refresh | SwipeRefreshLayout |
| Swipe actions | ItemTouchHelper |
| Haptic feedback | HapticFeedbackConstants |
| Share | Intent.ACTION_SEND |
| Context menu | PopupMenu or BottomSheet |
```

---

## 🔄 OFFLINE-FIRST ARCHITECTURE

```markdown
## Offline Strategy

### Data Sync Pattern
```
┌─────────────────────────────────────────────────────────────┐
│                     OFFLINE-FIRST FLOW                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  User Action ──▶ Local Storage ──▶ UI Update (Optimistic)   │
│                       │                                      │
│                       ▼                                      │
│               Sync Queue (Pending)                           │
│                       │                                      │
│           ┌──────────┴──────────┐                           │
│           │                     │                           │
│       Online               Offline                          │
│           │                     │                           │
│           ▼                     ▼                           │
│    Sync to Server         Queue for Later                   │
│           │                     │                           │
│           ▼                     │                           │
│    Confirm/Rollback ◀──────────┘                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Implementation

```typescript
// hooks/useOfflineSync.ts
export function useOfflineSync<T>(
  key: string,
  fetcher: () => Promise<T>,
) {
  const [data, setData] = useState<T | null>(null);
  const [isOffline, setIsOffline] = useState(false);
  const [pendingSync, setPendingSync] = useState(false);

  // Load from cache immediately
  useEffect(() => {
    loadFromCache(key). then(setData);
  }, [key]);

  // Sync when online
  useEffect(() => {
    if (! isOffline) {
      syncData();
    }
  }, [isOffline]);

  const mutate = async (mutation: (data: T) => T) => {
    // Optimistic update
    const optimisticData = mutation(data! );
    setData(optimisticData);
    await saveToCache(key, optimisticData);

    // Queue for sync
    await addToSyncQueue({ key, mutation, timestamp: Date.now() });
    setPendingSync(true);

    // Try immediate sync
    if (! isOffline) {
      await syncData();
    }
  };

  return { data, isOffline, pendingSync, mutate };
}
```

### Cache Strategy
| Data Type | Strategy | TTL |
|-----------|----------|-----|
| User profile | Cache-first, revalidate | 1 hour |
| Feed content | Stale-while-revalidate | 5 min |
| Static data | Cache-first | 24 hours |
| User actions | Queue and sync | Until synced |
```

---

## 🔐 MOBILE SECURITY

```markdown
## Security Checklist

### Secure Storage
```typescript
// ❌ Never
AsyncStorage.setItem('auth_token', token);
SharedPreferences.setString('password', password);

// ✅ Always
// iOS: Keychain
await Keychain.setGenericPassword('auth', token);

// Android: EncryptedSharedPreferences
const encryptedStorage = new EncryptedStorage();
await encryptedStorage.setItem('auth_token', token);

// React Native: react-native-keychain
import * as Keychain from 'react-native-keychain';
await Keychain.setSecureValue('token', token);
```

### Network Security
- [ ] HTTPS only (no HTTP)
- [ ] Certificate pinning for sensitive APIs
- [ ] No sensitive data in URLs
- [ ] Token refresh mechanism
- [ ] Timeout on requests

### App Security
- [ ] Root/Jailbreak detection
- [ ] Code obfuscation (ProGuard/R8)
- [ ] Biometric authentication option
- [ ] Background screenshot protection
- [ ] Secure clipboard handling

### Permissions
```typescript
// Request only when needed, explain why
const requestCameraPermission = async () => {
  // Show explanation first
  const shouldRequest = await showPermissionRationale(
    'Camera access needed to scan QR codes'
  );
  
  if (shouldRequest) {
    const result = await Camera.requestPermissions();
    // Handle result
  }
};
```
```

---

## ⚡ PERFORMANCE OPTIMIZATION

```markdown
## Mobile Performance

### Startup Time
| Phase | Target | Optimization |
|-------|--------|--------------|
| Cold start | <2s | Lazy loading, minimize initial JS |
| Warm start | <500ms | Cached data, preloaded assets |
| TTI | <3s | Progressive loading |

### Rendering
```typescript
// ❌ Heavy re-renders
const Item = ({ item }) => {
  // Recreates on every render
  const handlePress = () => doSomething(item. id);
  return <TouchableOpacity onPress={handlePress} />;
};

// ✅ Optimized
const Item = memo(({ item, onPress }) => {
  return <TouchableOpacity onPress={() => onPress(item.id)} />;
});

// Parent
const handleItemPress = useCallback((id: string) => {
  doSomething(id);
}, []);
```

### List Performance
```typescript
// FlashList for large lists
import { FlashList } from '@shopify/flash-list';

<FlashList
  data={items}
  renderItem={renderItem}
  estimatedItemSize={100}
  keyExtractor={item => item.id}
  getItemType={item => item. type}
/>

// Optimize item rendering
const renderItem = useCallback(({ item }) => (
  <MemoizedItem item={item} />
), []);
```

### Image Optimization
```typescript
// Progressive loading with blur placeholder
<Image
  source={{ uri: imageUrl }}
  placeholder={blurhash}
  contentFit="cover"
  transition={300}
/>

// Proper sizing
<Image
  source={{ uri: imageUrl }}
  style={{ width: 200, height: 200 }}
  // Loads appropriately sized image
/>
```

### Memory Management
- [ ] Clean up subscriptions in useEffect
- [ ] Avoid storing large data in state
- [ ] Use pagination for large lists
- [ ] Dispose of heavy resources
- [ ] Profile with Flipper/Instruments
```

---

## 🔔 PUSH NOTIFICATIONS

```markdown
## Notification Strategy

### Permission Request
```typescript
// Don't ask immediately on app launch
// Wait for meaningful moment

// After user completes first order
const onFirstOrderComplete = async () => {
  await showNotificationValueProp({
    title: "Stay updated on your order",
    description: "Get delivery updates and exclusive offers",
  });
  
  const permission = await Notifications.requestPermissions();
  
  if (permission. granted) {
    await registerForPushNotifications();
  }
};
```

### Notification Channels (Android)
```typescript
// Create meaningful channels
await Notifications.setNotificationChannelAsync('orders', {
  name: 'Order Updates',
  importance: Notifications.AndroidImportance.HIGH,
  sound: 'order_update. wav',
});

await Notifications.setNotificationChannelAsync('promotions', {
  name: 'Promotions',
  importance: Notifications.AndroidImportance.DEFAULT,
});
```

### Deep Linking
```typescript
// Handle notification tap
Notifications.addNotificationResponseReceivedListener(response => {
  const { screen, params } = response. notification. request. content. data;
  navigation.navigate(screen, params);
});
```
```

---

## 🧪 MOBILE TESTING

```markdown
## Test Strategy

### Unit Tests (Jest)
```typescript
describe('CartStore', () => {
  it('adds item to cart', () => {
    const store = new CartStore();
    store. addItem({ id: '1', quantity: 2 });
    expect(store.items).toHaveLength(1);
    expect(store.totalItems).toBe(2);
  });
});
```

### Component Tests (React Native Testing Library)
```typescript
import { render, fireEvent, screen } from '@testing-library/react-native';

describe('ProductCard', () => {
  it('calls onAddToCart when button pressed', () => {
    const onAddToCart = jest.fn();
    render(<ProductCard product={mockProduct} onAddToCart={onAddToCart} />);
    
    fireEvent.press(screen.getByText('Add to Cart'));
    
    expect(onAddToCart).toHaveBeenCalledWith(mockProduct. id);
  });
});
```

### E2E Tests (Detox/Maestro)
```yaml
# Maestro flow
appId: com.example.app
---
- launchApp
- tapOn: "Sign In"
- inputText:
    id: "email-input"
    text: "test@example.com"
- inputText:
    id: "password-input"
    text: "password123"
- tapOn: "Submit"
- assertVisible: "Welcome back"
```

### Device Testing Matrix
| Device Type | Priority | Test Coverage |
|-------------|----------|---------------|
| iPhone 15 Pro | High | Full |
| iPhone SE | High | Core flows |
| Pixel 8 | High | Full |
| Low-end Android | Medium | Core flows |
| Tablets | Medium | Layout only |
```

---

## 🚫 MOBILE ANTI-PATTERNS

| Avoid | Why | Instead |
|-------|-----|---------|
| Blocking UI thread | ANR/frozen UI | Async operations, proper threading |
| Storing secrets in code | Security breach | Secure storage, environment config |
| Ignoring platform conventions | Confuses users | Follow HIG/Material |
| Requesting all permissions upfront | Users deny all | Request contextually |
| Large bundle sizes | Slow download, install | Code splitting, asset optimization |
| No offline consideration | Broken experience | Offline-first design |
| Battery drain | User frustration | Optimize background work |
| No error boundaries | Crashes | Graceful error handling |