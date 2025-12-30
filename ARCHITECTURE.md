# CHEORY: Sistem Mimarisi
## ARCHITECTURE.md

> Bu doküman, Cheory platformunun teknik mimarisini tanımlar.
> Kod içermez, mimari kararları ve yaklaşımları içerir.

---

## 1. MİMARİ VİZYON

### 1.1 Temel Felsefe

| Prensip | Açıklama |
|---------|----------|
| Write Less, Do More | Minimum kod ile maksimum işlevsellik |
| Single Codebase | Tüm platformlar tek kaynaktan |
| Zero Build | npm yok, webpack yok, transpile yok |
| Progressive Enhancement | Temel çalışsın, gelişmiş özellikler eklensin |
| Offline First | Network olmadan da çalışabilir |

### 1.2 Mimari Stili

**Modüler Monolith** yaklaşımı:
- Tek repo, tek codebase
- Mantıksal modüller fiziksel olarak ayrı
- Modüller arası iletişim event-driven
- Her modül bağımsız test edilebilir

---

## 2. YÜKSEK SEVİYE MİMARİ

```
┌─────────────────────────────────────────────────────────────────┐
│                         PLATFORMS                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │     Web     │  │     PWA     │  │  Extension  │              │
│  │  (Browser)  │  │  (Install)  │  │   (MV3)     │              │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         └────────────────┼────────────────┘                      │
│                          ▼                                       │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    SHELL (Kabuk)                           │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │
│  │  │ Router  │ │  Store  │ │  i18n   │ │  Theme  │          │  │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘          │  │
│  │       └───────────┴───────────┴───────────┘                │  │
│  │                          │                                  │  │
│  │  ┌───────────────────────▼───────────────────────────────┐ │  │
│  │  │                  Core Services                         │ │  │
│  │  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐        │ │  │
│  │  │  │ Auth │ │Entitl│ │Analytics│ │Notif │ │ Sync │        │ │  │
│  │  │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘        │ │  │
│  │  └───────────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌───────────────────────────▼───────────────────────────────┐  │
│  │                    GAME LAYER                              │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Game Loader (Lazy)                      │  │  │
│  │  └─────────────────────────┬───────────────────────────┘  │  │
│  │                            │                               │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐  │  │
│  │  │ Game 1 │ │ Game 2 │ │ Game 3 │ │  ...   │ │Game 12 │  │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌───────────────────────────▼───────────────────────────────┐  │
│  │                   DATA LAYER                               │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │ localStorage│  │  IndexedDB  │  │  Supabase   │        │  │
│  │  │  (Settings) │  │  (Offline)  │  │   (Cloud)   │        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. KATMAN MİMARİSİ

### 3.1 Platform Katmanı (En Üst)

Platform-spesifik adaptörler. Ana kodu değiştirmeden platform farklarını yönetir.

| Platform | Adaptör Görevi |
|----------|----------------|
| Web | Doğrudan çalışır, adaptör minimal |
| PWA | Service worker, manifest, install prompt |
| Extension | chrome.* API'ları, side panel, new tab |
| Android (TWA) | TWA wrapper, Play Billing |
| iOS | App Store wrapper, Apple IAP |

**Yaklaşım:** Platform katmanı sadece "köprü" görevi görür. İş mantığı burada OLMAZ.

### 3.2 Shell Katmanı (Kabuk)

Uygulamanın "iskeleti". Tüm platformlarda aynı çalışır.

| Modül | Sorumluluk |
|-------|------------|
| Router | URL/hash yönetimi, sayfa geçişleri |
| Store | Global state, reactive updates |
| i18n | Dil yönetimi, çeviri lookup |
| Theme | Dark/light mode, CSS variable yönetimi |

**Yaklaşım:** Shell modülleri birbirinden bağımsız. Pub/Sub ile iletişim.

### 3.3 Core Services Katmanı

Platform-agnostik iş servisleri.

| Servis | Sorumluluk |
|--------|------------|
| Auth | Login/logout, session, token yönetimi |
| Entitlement | Tier kontrolü, feature flags |
| Analytics | Event tracking, batching |
| Notification | Push notification yönetimi |
| Sync | Offline queue, conflict resolution |

**Yaklaşım:** Her servis tek bir şey yapar (Single Responsibility).

### 3.4 Game Layer (Oyun Katmanı)

Oyunların yaşadığı izole alan.

| Bileşen | Sorumluluk |
|---------|------------|
| Game Loader | Lazy loading, entitlement check |
| Game Module | Bağımsız oyun paketi |
| Shared Utils | Ortak yardımcı fonksiyonlar |

**Yaklaşım:** Oyunlar birbirini bilmez. Shell üzerinden iletişim.

### 3.5 Data Layer (Veri Katmanı)

Veri kalıcılığı ve senkronizasyon.

| Depo | Kullanım | Sync |
|------|----------|------|
| localStorage | Ayarlar, tercihler | Cihaza özel |
| IndexedDB | Offline veri, queue | Cihaza özel |
| Supabase | Profil, skorlar, abonelik | Cloud |

**Yaklaşım:** Data layer üst katmanlardan soyutlanmış. Repository pattern.

---

## 4. MODÜL İLİŞKİLERİ

### 4.1 Bağımlılık Kuralları

```
YASAL BAĞIMLILIKLAR (Yukarıdan aşağıya):

Platform → Shell → Core Services → Game Layer → Data Layer

YASAK BAĞIMLILIKLAR:

✗ Game → Game (oyunlar birbirini çağıramaz)
✗ Data → Game (data layer oyun bilmez)
✗ Core Service → Platform (servisler platform bilmez)
✗ Aşağıdan yukarı bağımlılık (hiçbir zaman)
```

### 4.2 İletişim Matrisi

| Kimden | Kime | Nasıl |
|--------|------|-------|
| Platform → Shell | Direct import | Adaptör Shell'i başlatır |
| Shell → Shell | Event Bus | Router ↔ Store haberleşmesi |
| Shell → Core | Direct call | Store auth servisini çağırır |
| Core → Core | Event Bus | Auth → Analytics event gönderir |
| Shell → Game | Game Loader | Lazy load + inject |
| Game → Shell | Callback/Event | dispatch, navigate |
| Core → Data | Repository | Auth → Supabase |
| Game → Data | Repository | Game → Score kaydet |

### 4.3 Event Bus Yapısı

Modüller arası loose coupling için merkezi event sistemi.

```
Event Kategorileri:

app:*        → Uygulama yaşam döngüsü
auth:*       → Kimlik doğrulama
game:*       → Oyun olayları
user:*       → Kullanıcı aksiyonları
sync:*       → Senkronizasyon
error:*      → Hata yönetimi

Örnek Event'ler:

app:ready           → Uygulama hazır
auth:login          → Kullanıcı giriş yaptı
auth:logout         → Kullanıcı çıkış yaptı
game:start          → Oyun başladı
game:end            → Oyun bitti
game:action         → Oyuncu hamle yaptı
user:tier-change    → Tier değişti
sync:online         → Online olundu
sync:offline        → Offline olundu
error:critical      → Kritik hata
```

---

## 5. VERİ AKIŞI

### 5.1 Tek Yönlü Veri Akışı (Unidirectional)

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  User   │────▶│ Action  │────▶│  Store  │────▶│  View   │
│ Action  │     │Dispatch │     │ Update  │     │ Render  │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                                     │
                                     ▼
                              ┌─────────────┐
                              │   Persist   │
                              │ (Async)     │
                              └─────────────┘
```

**Neden Tek Yönlü?**
- Debug kolaylığı
- Tahmin edilebilir state
- Zaman-yolculuğu (undo/redo) mümkün

### 5.2 Oyun İçi Veri Akışı

```
┌──────────────────────────────────────────────────────────────┐
│                      GAME MODULE                              │
│                                                               │
│  ┌─────────┐    ┌─────────────┐    ┌─────────────────────┐   │
│  │  User   │───▶│   Action    │───▶│    Game Engine      │   │
│  │  Input  │    │  Validator  │    │                     │   │
│  └─────────┘    └─────────────┘    │  ┌───────────────┐  │   │
│                        │           │  │ Execute Action│  │   │
│                        │           │  └───────┬───────┘  │   │
│                        │           │          │          │   │
│                        │           │  ┌───────▼───────┐  │   │
│                        │           │  │ Calculate     │  │   │
│                        │           │  │ New State     │  │   │
│                        │           │  └───────┬───────┘  │   │
│                        │           │          │          │   │
│                        │           │  ┌───────▼───────┐  │   │
│                        │           │  │ Check Game    │  │   │
│                        │           │  │ Over          │  │   │
│                        │           │  └───────────────┘  │   │
│                        │           └──────────┬──────────┘   │
│                        │                      │               │
│                        │                      ▼               │
│                        │           ┌─────────────────────┐   │
│                        │           │    New State        │   │
│                        │           └──────────┬──────────┘   │
│                        │                      │               │
│  ┌─────────────────────┴──────────────────────┘               │
│  │                                                            │
│  ▼                                                            │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                      VIEW                                │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │ │
│  │  │  Board   │  │  Score   │  │ Controls │              │ │
│  │  └──────────┘  └──────────┘  └──────────┘              │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### 5.3 Auth Akışı

```
┌────────┐     ┌────────────┐     ┌──────────┐     ┌──────────┐
│  User  │────▶│  Auth UI   │────▶│  Auth    │────▶│ Supabase │
│  Click │     │  (Google)  │     │  Service │     │   Auth   │
└────────┘     └────────────┘     └──────────┘     └────┬─────┘
                                                        │
                    ┌───────────────────────────────────┘
                    │
                    ▼
              ┌──────────┐     ┌──────────┐     ┌──────────┐
              │  Token   │────▶│  Store   │────▶│ Entitle- │
              │ Received │     │  Update  │     │  ment    │
              └──────────┘     └──────────┘     └──────────┘
                                                     │
                                                     ▼
                                              ┌──────────────┐
                                              │ UI Reflects  │
                                              │ User State   │
                                              └──────────────┘
```

### 5.4 Sync Akışı (Offline → Online)

```
OFFLINE DURUMU:

┌────────┐     ┌────────────┐     ┌──────────────┐
│ Action │────▶│  Execute   │────▶│  Save to     │
│        │     │  Locally   │     │  IndexedDB   │
└────────┘     └────────────┘     │  + Queue     │
                                  └──────────────┘

ONLINE OLUNCA:

┌──────────────┐     ┌────────────┐     ┌──────────┐
│  Detect      │────▶│  Process   │────▶│ Supabase │
│  Online      │     │  Queue     │     │  Sync    │
└──────────────┘     └────────────┘     └────┬─────┘
                                              │
                          ┌───────────────────┘
                          │
                          ▼
                    ┌──────────────┐     ┌──────────────┐
                    │   Resolve    │────▶│   Clear      │
                    │   Conflicts  │     │   Queue      │
                    └──────────────┘     └──────────────┘
```

---

## 6. OYUN MODÜLÜ MİMARİSİ

### 6.1 Modül Yapısı

Her oyun modülü şu yapıda organize edilir:

```
/games/prisoners-dilemma/
│
├── index.js          → Entry point, export'lar
├── engine.js         → Oyun mantığı (pure functions)
├── view.js           → UI bileşenleri (Web Components)
├── strategies.js     → Bot stratejileri
└── i18n/             → Oyun-özel çeviriler (opsiyonel)
    ├── en.json
    └── tr.json
```

### 6.2 Engine-View Ayrımı

```
┌─────────────────────────────────────────────────────────────┐
│                      ENGINE (Pure)                           │
│                                                              │
│  - State transition fonksiyonları                           │
│  - Payoff hesaplamaları                                     │
│  - Validation kuralları                                     │
│  - Bot stratejileri                                         │
│                                                              │
│  INPUT: State + Action                                       │
│  OUTPUT: New State                                           │
│                                                              │
│  Side Effect: YOK                                            │
│  DOM Access: YOK                                             │
│  Global State: YOK                                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ State
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       VIEW (Impure)                          │
│                                                              │
│  - Web Components                                            │
│  - Event listeners                                           │
│  - DOM manipulation                                          │
│  - Animation                                                 │
│                                                              │
│  INPUT: State                                                │
│  OUTPUT: DOM Updates                                         │
│                                                              │
│  Dispatch: Action'ları Shell'e gönderir                     │
└─────────────────────────────────────────────────────────────┘
```

**Neden Bu Ayrım?**
- Engine test edilebilir (pure functions)
- View değiştirilebilir (farklı UI)
- Mantık tekrar kullanılabilir

### 6.3 Oyun Yaşam Döngüsü

```
┌──────────┐
│  IDLE    │ ← Başlangıç
└────┬─────┘
     │ load()
     ▼
┌──────────┐
│ LOADING  │ ← Lazy load
└────┬─────┘
     │ ready
     ▼
┌──────────┐
│  SETUP   │ ← Oyuncu/bot seçimi, ayarlar
└────┬─────┘
     │ start()
     ▼
┌──────────┐
│ PLAYING  │ ← Ana oyun döngüsü
└────┬─────┘
     │ isGameOver() === true
     ▼
┌──────────┐
│ FINISHED │ ← Sonuç ekranı
└────┬─────┘
     │ restart() veya exit()
     ▼
┌──────────┐     ┌──────────┐
│  SETUP   │ or │   IDLE   │
└──────────┘     └──────────┘
```

---

## 7. STORE MİMARİSİ

### 7.1 State Şeması (Üst Seviye)

```
AppState {
  // Shell State
  router: {
    currentPath: string
    params: object
    history: string[]
  }
  
  // User State
  auth: {
    isAuthenticated: boolean
    user: User | null
    session: Session | null
  }
  
  // Entitlement State
  entitlement: {
    tier: 'guest' | 'free' | 'premium' | 'premium_plus' | 'classroom'
    unlockedGames: string[]
    dailyPlaysRemaining: number
    adsWatchedToday: number
    features: FeatureFlags
  }
  
  // Preferences
  preferences: {
    theme: 'dark' | 'light' | 'system'
    language: 'en' | 'tr'
    sound: boolean
    notifications: boolean
  }
  
  // Active Game State (sadece oyun açıkken)
  activeGame: {
    gameId: string | null
    state: GameState | null
    history: GameState[]
  }
  
  // UI State
  ui: {
    isLoading: boolean
    modal: Modal | null
    toast: Toast | null
  }
  
  // Sync State
  sync: {
    isOnline: boolean
    pendingActions: Action[]
    lastSyncAt: Date | null
  }
}
```

### 7.2 Store Tasarım Prensipleri

| Prensip | Açıklama |
|---------|----------|
| Single Store | Tek global store, parçalanmış state yok |
| Immutable Updates | State asla mutate edilmez |
| Selector Pattern | State okuma fonksiyonlarla |
| Action Creators | Standart action formatı |
| Middleware | Logging, persist, analytics |

### 7.3 Reactive Update Mekanizması

```
Store.subscribe() Yaklaşımı:

┌──────────┐     ┌──────────┐     ┌──────────┐
│Component │     │  Store   │     │Component │
│    A     │     │          │     │    B     │
└────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                 │
     │ subscribe      │                 │ subscribe
     │───────────────▶│◀────────────────│
     │                │                 │
     │                │ dispatch(action)│
     │                │◀────────────────│
     │                │                 │
     │                │ state changes   │
     │                │                 │
     │◀───notify──────│──────notify────▶│
     │                │                 │
     │ re-render      │                 │ re-render
     ▼                │                 ▼
```

---

## 8. PLATFORM ADAPTÖR MİMARİSİ

### 8.1 Adaptör Pattern

```
┌───────────────────────────────────────────────────────────┐
│                    ABSTRACT ADAPTER                        │
│                                                            │
│  interface PlatformAdapter {                               │
│    // Storage                                              │
│    storage: StorageAdapter                                 │
│                                                            │
│    // Navigation                                           │
│    navigate(path): void                                    │
│    getInitialRoute(): string                               │
│                                                            │
│    // Platform features                                    │
│    showNotification(options): void                         │
│    requestPermission(type): Promise                        │
│                                                            │
│    // Lifecycle                                            │
│    onActivate(callback): void                              │
│    onDeactivate(callback): void                            │
│  }                                                         │
└───────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   WebAdapter    │ │   PWAAdapter    │ │ExtensionAdapter │
│                 │ │                 │ │                 │
│ - Basic browser │ │ - Service Worker│ │ - chrome.* APIs │
│ - localStorage  │ │ - Install prompt│ │ - Side panel    │
│ - History API   │ │ - Cache API     │ │ - New tab       │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### 8.2 Storage Adapter

```
StorageAdapter Interface:

┌─────────────────────────────────────────────────────────┐
│  get(key): Promise<value>                                │
│  set(key, value): Promise<void>                          │
│  remove(key): Promise<void>                              │
│  clear(): Promise<void>                                  │
└─────────────────────────────────────────────────────────┘

Implementations:

┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ LocalStorage     │  │ ChromeStorage    │  │ IndexedDB        │
│ Adapter          │  │ Adapter          │  │ Adapter          │
│                  │  │                  │  │                  │
│ Web/PWA için     │  │ Extension için   │  │ Büyük veri için  │
│ Senkron          │  │ chrome.storage   │  │ Async            │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

## 9. SECURITY MİMARİSİ

### 9.1 Güvenlik Katmanları

```
┌─────────────────────────────────────────────────────────────┐
│                    INPUT VALIDATION                          │
│  - Tüm kullanıcı input'ları validate edilir                 │
│  - Schema-based validation                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION                            │
│  - Supabase Auth (JWT)                                       │
│  - Token refresh otomatik                                    │
│  - Session timeout                                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    AUTHORIZATION                             │
│  - Tier-based access control                                 │
│  - Row Level Security (Supabase RLS)                         │
│  - Feature flags                                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    DATA PROTECTION                           │
│  - Minimal data collection                                   │
│  - HTTPS only                                                │
│  - No sensitive data in localStorage                         │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 Extension-Specific Security

| Tehdit | Mitigasyon |
|--------|------------|
| XSS | CSP strict, no innerHTML |
| Code injection | No eval, no remote code |
| Data leakage | Minimal permissions |
| Clickjacking | X-Frame-Options |

---

## 10. PERFORMANS MİMARİSİ

### 10.1 Loading Stratejisi

```
┌─────────────────────────────────────────────────────────────┐
│                    CRITICAL PATH                             │
│  (İlk render için gerekli - inline veya preload)            │
│                                                              │
│  - Shell CSS (< 10KB)                                        │
│  - Router (< 5KB)                                            │
│  - Store (< 5KB)                                             │
│  - Auth check (< 2KB)                                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ First Paint
┌─────────────────────────────────────────────────────────────┐
│                    DEFERRED                                  │
│  (İlk render sonrası lazy load)                             │
│                                                              │
│  - i18n dictionaries                                         │
│  - Theme system                                              │
│  - Analytics                                                 │
│  - Notification service                                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ Interactive
┌─────────────────────────────────────────────────────────────┐
│                    ON-DEMAND                                 │
│  (Kullanıcı aksiyonuyla yüklenir)                           │
│                                                              │
│  - Game modules (her biri ayrı)                              │
│  - Settings panel                                            │
│  - Modals                                                    │
└─────────────────────────────────────────────────────────────┘
```

### 10.2 Caching Stratejisi

| Kaynak | Strateji | TTL |
|--------|----------|-----|
| Shell (HTML/CSS/JS) | Cache-first | 1 hafta |
| Game modules | Cache-first | 1 hafta |
| i18n | Stale-while-revalidate | 1 gün |
| User data | Network-first | - |
| Static assets | Cache-first | 1 ay |

### 10.3 Bundle Budget

| Kategori | Limit |
|----------|-------|
| Critical CSS | < 10KB |
| Critical JS | < 20KB |
| Per-game JS | < 15KB |
| Total initial | < 50KB |
| Total (all games) | < 200KB |

---

## 11. TEST MİMARİSİ

### 11.1 Test Piramidi

```
                    ┌───────────┐
                    │    E2E    │  ← Az sayıda, kritik flow'lar
                    │   Tests   │
                    └─────┬─────┘
                          │
                    ┌─────▼─────┐
                    │Integration│  ← Modül entegrasyonları
                    │   Tests   │
                    └─────┬─────┘
                          │
              ┌───────────▼───────────┐
              │      Unit Tests       │  ← Çok sayıda, hızlı
              │  (Engine functions)   │
              └───────────────────────┘
```

### 11.2 Test Edilebilirlik Prensipleri

| Prensip | Uygulama |
|---------|----------|
| Pure Functions | Engine fonksiyonları side-effect'siz |
| Dependency Injection | Services inject edilebilir |
| Mockable Adapters | Platform adapters mock'lanabilir |
| Deterministic | Aynı input, aynı output |

---

## 12. MİMARİ KARARLAR KAYDI (ADR)

### ADR-001: Vanilla JS Seçimi

**Durum:** Kabul Edildi

**Bağlam:** Framework seçimi gerekiyordu.

**Karar:** Vanilla JS + ES Modules kullanılacak.

**Gerekçe:**
- MV3'te build karmaşıklığı yok
- Tüm platformlarda aynı çalışır
- Minimum bundle size
- Vibe coding uyumu

**Sonuçlar:**
- (+) Basitlik
- (+) Performans
- (-) Bazı kolaylıklardan vazgeçme
- (-) Daha fazla boilerplate

---

### ADR-002: Event-Driven Architecture

**Durum:** Kabul Edildi

**Bağlam:** Modüller arası iletişim yöntemi gerekiyordu.

**Karar:** Merkezi Event Bus kullanılacak.

**Gerekçe:**
- Loose coupling
- Modüller bağımsız geliştirilebilir
- Debug kolaylığı

**Sonuçlar:**
- (+) Esneklik
- (+) Test edilebilirlik
- (-) Event takibi zorlaşabilir
- (-) Implicit dependencies

---

### ADR-003: Single Codebase Multi-Platform

**Durum:** Kabul Edildi

**Bağlam:** Birden fazla platform desteklenmeli.

**Karar:** Tek codebase, platform adaptörleri.

**Gerekçe:**
- Kod tekrarı yok
- Tek kaynak gerçeği
- Bakım kolaylığı

**Sonuçlar:**
- (+) DRY
- (+) Tutarlılık
- (-) Adaptör karmaşıklığı
- (-) Platform-specific özellikler kısıtlı

---

## 13. SONRAKI ADIMLAR

Bu mimari doküman baz alınarak şu dokümanlar oluşturulacak:

| Sıra | Doküman | Detaylandıracağı Alan |
|------|---------|----------------------|
| 1 | FOLDER_STRUCTURE.md | Klasör ağacı, dosya konvansiyonları |
| 2 | CORE_ENGINE_CONTRACT.md | Game module interface detayları |
| 3 | DATA_SCHEMA.md | Supabase tabloları, state şemaları |
| 4 | UI_COMPONENTS.md | Web Component kataloğu |

---

## 14. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |

---

> **Not:** Bu doküman kod içermez. Implementasyon detayları ilgili spec dokümanlarında yer alacaktır.
