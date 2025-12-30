# CHEORY: Klasör Yapısı
## FOLDER_STRUCTURE.md

> Bu doküman, Cheory projesinin dosya ve klasör organizasyonunu tanımlar.
> Tüm geliştiriciler bu yapıya uymalıdır.

---

## 1. GENEL PRENSİPLER

### 1.1 Organizasyon Felsefesi

| Prensip | Açıklama |
|---------|----------|
| Flat over Nested | Derin iç içe klasörlerden kaçın (max 3 seviye) |
| Colocation | İlgili dosyalar yan yana |
| Feature-based | Teknik tipe göre değil, özelliğe göre grupla |
| Self-documenting | İsimler açıklayıcı olsun |
| Discoverability | Yeni gelen kolayca anlasın |

### 1.2 Dosya Limitleri

| Kural | Limit | Kapsam |
|-------|-------|--------|
| Toplam dosya | < 20 (core) | Shell + Core Services |
| Oyun başına | 3-5 dosya | Her game modülü |
| Dosya boyutu | < 500 satır | Tek dosya |
| Klasör derinliği | Max 3 seviye | Tüm proje |

---

## 2. KÖKDIZIN YAPISI

```
cheory/
│
├── index.html              # Ana giriş noktası (Web/PWA)
├── manifest.json           # PWA manifest
├── sw.js                   # Service Worker
├── robots.txt              # SEO
├── favicon.ico             # Favicon
│
├── /core/                  # Platform çekirdeği
├── /games/                 # Oyun modülleri
├── /adapters/              # Platform adaptörleri
├── /assets/                # Statik dosyalar
├── /locales/               # Çeviri dosyaları
├── /styles/                # Global stiller
├── /docs/                  # Proje dokümanları
│
├── /extension/             # Chrome Extension özel dosyaları
├── /pwa/                   # PWA özel dosyaları (icons vb.)
│
├── .gitignore
├── README.md
└── LICENSE
```

---

## 3. DETAYLI KLASÖR AÇIKLAMALARI

### 3.1 /core/ - Platform Çekirdeği

Uygulamanın kalbi. Tüm platformlarda ortak çalışan temel modüller.

```
/core/
│
├── shell.js                # Shell başlatıcı, bootstrap
├── router.js               # SPA routing, hash-based navigation
├── store.js                # Global state yönetimi
├── events.js               # Event bus, pub/sub sistemi
│
├── /services/              # Core iş servisleri
│   ├── auth.js             # Authentication
│   ├── entitlement.js      # Tier, feature flags
│   ├── analytics.js        # Event tracking
│   ├── sync.js             # Offline/online sync
│   └── notification.js     # Push notification
│
├── /utils/                 # Yardımcı fonksiyonlar
│   ├── dom.js              # DOM helpers
│   ├── storage.js          # Storage abstraction
│   ├── validation.js       # Input validation
│   ├── format.js           # Date, number formatting
│   └── i18n.js             # Internationalization
│
└── /components/            # Paylaşılan Web Components
    ├── base-component.js   # Temel component sınıfı
    ├── app-header.js       # Üst navigasyon
    ├── app-footer.js       # Alt bilgi
    ├── game-card.js        # Oyun kartı
    ├── modal-dialog.js     # Modal pencere
    ├── toast-message.js    # Bildirim toast
    ├── loading-spinner.js  # Yükleniyor göstergesi
    ├── tier-badge.js       # Premium rozeti
    └── ad-container.js     # Reklam wrapper
```

**Dosya Sayısı:** ~20 dosya (limit dahilinde)

---

### 3.2 /games/ - Oyun Modülleri

Her oyun bağımsız bir alt klasör. Birbirinden izole.

```
/games/
│
├── _loader.js              # Game lazy loader
├── _registry.js            # Oyun listesi, metadata
│
├── /prisoners-dilemma/     # Oyun 1
│   ├── index.js            # Entry point, exports
│   ├── engine.js           # Oyun mantığı (pure)
│   ├── view.js             # UI components
│   ├── strategies.js       # Bot stratejileri
│   └── README.md           # Oyun dokümantasyonu
│
├── /iterated-pd/           # Oyun 2
│   ├── index.js
│   ├── engine.js
│   ├── view.js
│   └── strategies.js
│
├── /stag-hunt/             # Oyun 3
│   └── ...
│
├── /hawk-dove/             # Oyun 4
│   └── ...
│
├── /matching-pennies/      # Oyun 5
│   └── ...
│
├── /battle-of-sexes/       # Oyun 6
│   └── ...
│
├── /public-goods/          # Oyun 7
│   └── ...
│
├── /ultimatum/             # Oyun 8
│   └── ...
│
├── /dictator/              # Oyun 9
│   └── ...
│
├── /trust-game/            # Oyun 10
│   └── ...
│
├── /rps/                   # Oyun 11 (Rock-Paper-Scissors)
│   └── ...
│
└── /voting-lab/            # Oyun 12
    ├── index.js
    ├── engine.js
    ├── view.js
    ├── methods/            # Voting metodları (alt modül)
    │   ├── plurality.js
    │   ├── borda.js
    │   └── irv.js
    └── strategies.js
```

**Her Oyun Dosya Yapısı:**

| Dosya | Amaç | Zorunlu |
|-------|------|---------|
| index.js | Entry, export'lar, metadata | Evet |
| engine.js | Pure oyun mantığı | Evet |
| view.js | Web Components, UI | Evet |
| strategies.js | Bot AI stratejileri | Hayır |
| README.md | Oyun dokümantasyonu | Hayır |

---

### 3.3 /adapters/ - Platform Adaptörleri

Platform-spesifik kod. Core'u değiştirmeden platform farklarını yönetir.

```
/adapters/
│
├── base-adapter.js         # Abstract adapter interface
├── web-adapter.js          # Vanilla web browser
├── pwa-adapter.js          # PWA özel (install prompt vb.)
├── extension-adapter.js    # Chrome extension (chrome.* APIs)
│
└── /storage/               # Storage implementasyonları
    ├── local-storage.js    # localStorage wrapper
    ├── chrome-storage.js   # chrome.storage wrapper
    └── indexed-db.js       # IndexedDB wrapper
```

---

### 3.4 /assets/ - Statik Dosyalar

Binary ve statik kaynaklar.

```
/assets/
│
├── /icons/                 # Uygulama ikonları
│   ├── icon-16.png
│   ├── icon-32.png
│   ├── icon-48.png
│   ├── icon-128.png
│   ├── icon-192.png
│   ├── icon-512.png
│   └── icon.svg            # Vektör kaynak
│
├── /fonts/                 # Font dosyaları (vendor)
│   ├── fa-solid-900.woff2  # FontAwesome solid
│   ├── fa-regular-400.woff2
│   └── fa-brands-400.woff2
│
├── /images/                # Görsel assets
│   ├── logo.svg
│   ├── og-image.png        # Social share
│   └── /games/             # Oyun görselleri
│       ├── pd-thumb.svg
│       └── ...
│
└── /sounds/                # Ses efektleri (opsiyonel)
    ├── click.mp3
    ├── success.mp3
    └── error.mp3
```

---

### 3.5 /locales/ - Çeviri Dosyaları

i18n için JSON çeviri dosyaları.

```
/locales/
│
├── /en/                    # İngilizce
│   ├── common.json         # Genel UI
│   ├── games.json          # Oyun adları, açıklamalar
│   ├── tutorial.json       # Tutorial metinleri
│   └── errors.json         # Hata mesajları
│
└── /tr/                    # Türkçe
    ├── common.json
    ├── games.json
    ├── tutorial.json
    └── errors.json
```

**JSON Yapısı Örneği (common.json):**

```
{
  "app": {
    "name": "Cheory",
    "tagline": "Game Theory Arena"
  },
  "nav": {
    "home": "Home",
    "games": "Games",
    "profile": "Profile",
    "settings": "Settings"
  },
  "actions": {
    "play": "Play",
    "restart": "Restart",
    "exit": "Exit",
    "confirm": "Confirm",
    "cancel": "Cancel"
  },
  "messages": {
    "loading": "Loading...",
    "error": "Something went wrong",
    "offline": "You are offline"
  }
}
```

---

### 3.6 /styles/ - Global Stiller

CSS dosyaları. Modüler, değişken tabanlı.

```
/styles/
│
├── main.css                # Ana entry, import'lar
│
├── /base/                  # Reset ve temel stiller
│   ├── reset.css           # CSS reset
│   ├── typography.css      # Font, heading stilleri
│   └── variables.css       # CSS custom properties
│
├── /components/            # Component stilleri
│   ├── buttons.css
│   ├── cards.css
│   ├── forms.css
│   ├── modals.css
│   └── navigation.css
│
├── /layouts/               # Sayfa layout'ları
│   ├── grid.css            # Bento grid sistemi
│   ├── shell.css           # Ana shell layout
│   └── game.css            # Oyun sayfası layout
│
├── /themes/                # Tema varyasyonları
│   ├── dark.css            # Dark mode (default)
│   └── light.css           # Light mode override
│
└── /vendors/               # 3rd party CSS
    └── fontawesome.css     # FA minimal subset
```

---

### 3.7 /extension/ - Chrome Extension

MV3 extension özel dosyaları.

```
/extension/
│
├── manifest.json           # Extension manifest (MV3)
├── background.js           # Service worker
│
├── /pages/                 # Extension sayfaları
│   ├── sidepanel.html      # Side panel UI
│   ├── newtab.html         # New tab override
│   ├── options.html        # Ayarlar sayfası
│   └── popup.html          # Popup (minimal)
│
└── /scripts/               # Extension-only scripts
    ├── sidepanel.js        # Side panel bootstrap
    ├── newtab.js           # New tab bootstrap
    └── options.js          # Options page logic
```

**manifest.json Yapısı (Özet):**

```
{
  "manifest_version": 3,
  "name": "Cheory",
  "version": "1.0.0",
  "description": "Game Theory Arena",
  
  "permissions": [
    "storage",
    "sidePanel"
  ],
  
  "background": {
    "service_worker": "background.js"
  },
  
  "side_panel": {
    "default_path": "pages/sidepanel.html"
  },
  
  "chrome_url_overrides": {
    "newtab": "pages/newtab.html"
  },
  
  "action": {
    "default_popup": "pages/popup.html"
  },
  
  "options_page": "pages/options.html",
  
  "icons": { ... }
}
```

---

### 3.8 /pwa/ - PWA Özel Dosyalar

PWA için ek kaynaklar.

```
/pwa/
│
├── /icons/                 # PWA ikonları (maskable dahil)
│   ├── icon-maskable-192.png
│   ├── icon-maskable-512.png
│   └── apple-touch-icon.png
│
├── /screenshots/           # Store screenshots
│   ├── screenshot-1.png
│   └── screenshot-2.png
│
└── offline.html            # Offline fallback sayfası
```

---

### 3.9 /docs/ - Proje Dokümanları

Tüm spec ve planlama dokümanları.

```
/docs/
│
├── /specs/                 # Teknik spesifikasyonlar
│   ├── ARCHITECTURE.md
│   ├── FOLDER_STRUCTURE.md     # (Bu dosya)
│   ├── CORE_ENGINE_CONTRACT.md
│   ├── DATA_SCHEMA.md
│   ├── UI_COMPONENTS.md
│   ├── PLATFORM_RULES.md
│   └── API_CONTRACT.md
│
├── /games/                 # Oyun spesifikasyonları
│   ├── GAME_SPEC_TEMPLATE.md
│   ├── 01_PRISONERS_DILEMMA.md
│   ├── 02_ITERATED_PD.md
│   └── ...
│
├── /guides/                # Kılavuzlar
│   ├── STYLE_GUIDE.md
│   ├── I18N_GUIDE.md
│   ├── TESTING_PLAN.md
│   └── CONTRIBUTING.md
│
├── /business/              # İş dokümanları
│   ├── MONETIZATION.md
│   ├── ANALYTICS_EVENTS.md
│   └── PRIVACY_POLICY_SPEC.md
│
└── /planning/              # Planlama
    ├── cheory_master_spec.md
    ├── SPRINT_PLAN.md
    └── CHANGELOG.md
```

---

## 4. İSİMLENDİRME KURALLARI

### 4.1 Klasör İsimleri

| Kural | Örnek | Açıklama |
|-------|-------|----------|
| kebab-case | `/prisoners-dilemma/` | Tire ile ayrılmış küçük harf |
| Tekil | `/game/` değil `/games/` | Çoğul tercih edilir |
| Anlamlı | `/utils/` değil `/u/` | Kısaltma yok |
| İngilizce | `/oyunlar/` değil `/games/` | Kod dili İngilizce |

### 4.2 Dosya İsimleri

| Tür | Format | Örnek |
|-----|--------|-------|
| JavaScript | kebab-case.js | `game-loader.js` |
| CSS | kebab-case.css | `main.css`, `buttons.css` |
| JSON | kebab-case.json | `common.json` |
| Markdown | SCREAMING_CASE.md | `README.md`, `ARCHITECTURE.md` |
| Assets | kebab-case.ext | `icon-128.png` |

### 4.3 Component İsimleri

| Tür | Format | Örnek |
|-----|--------|-------|
| Web Component class | PascalCase | `AppHeader`, `GameCard` |
| Custom element tag | kebab-case | `<app-header>`, `<game-card>` |
| Dosya adı | kebab-case.js | `app-header.js`, `game-card.js` |

### 4.4 Fonksiyon ve Değişken İsimleri

| Tür | Format | Örnek |
|-----|--------|-------|
| Fonksiyon | camelCase | `calculateScore()`, `handleClick()` |
| Değişken | camelCase | `playerScore`, `isGameOver` |
| Sabit | SCREAMING_SNAKE | `MAX_ROUNDS`, `DEFAULT_TIMEOUT` |
| Private | _prefix | `_internalState`, `_helper()` |

---

## 5. IMPORT/EXPORT KURALLARI

### 5.1 Relative vs Absolute Paths

```
TERCIH EDİLEN (Relative):

// /games/prisoners-dilemma/view.js içinde
import { executeAction } from './engine.js';
import { BaseComponent } from '../../core/components/base-component.js';

KAÇINILAN (Magic paths):

// Webpack alias gibi yapılar YOK
import { something } from '@core/utils';  // ✗ Kullanma
```

### 5.2 Export Pattern

```
// Named exports tercih et (tree-shaking için)
export function calculatePayoff() { ... }
export function isGameOver() { ... }

// Default export sadece ana modül için
export default {
  id: 'prisoners-dilemma',
  name: { en: 'Prisoner\'s Dilemma', tr: 'Mahkum İkilemi' },
  engine,
  view
};
```

### 5.3 Barrel Exports

```
// /core/services/index.js (barrel file)
export { AuthService } from './auth.js';
export { EntitlementService } from './entitlement.js';
export { AnalyticsService } from './analytics.js';

// Kullanım
import { AuthService, EntitlementService } from '../services/index.js';
```

---

## 6. ÖZEL DOSYALAR

### 6.1 index.html (Kök)

Ana giriş noktası. Minimal, hızlı yüklenen.

```
Yapısı:
- DOCTYPE, html, head, body
- Critical CSS inline veya preload
- Minimum script (shell bootstrap)
- Loading placeholder
- No blocking resources
```

### 6.2 manifest.json (PWA)

```
Zorunlu alanlar:
- name, short_name
- start_url
- display: standalone
- icons (192, 512)
- theme_color, background_color

Opsiyonel:
- description
- screenshots
- shortcuts
- share_target
```

### 6.3 sw.js (Service Worker)

```
Sorumlulukları:
- Cache stratejileri
- Offline fallback
- Background sync
- Push notification handling

Dikkat:
- Versiyon yönetimi
- Cache invalidation
- Update flow
```

### 6.4 .gitignore

```
# Dependencies (yok ama olursa)
node_modules/

# Build output (yok ama olursa)
dist/
build/

# Environment
.env
.env.local

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db

# Logs
*.log

# Test
coverage/
```

---

## 7. PLATFORM-SPESİFİK YAPILAR

### 7.1 Web/PWA Yapısı

```
Web/PWA için kullanılan dosyalar:

cheory/
├── index.html          ✓
├── manifest.json       ✓
├── sw.js               ✓
├── /core/              ✓ (tamamı)
├── /games/             ✓ (tamamı)
├── /adapters/
│   ├── web-adapter.js  ✓
│   └── pwa-adapter.js  ✓
├── /assets/            ✓ (tamamı)
├── /locales/           ✓ (tamamı)
├── /styles/            ✓ (tamamı)
└── /pwa/               ✓ (tamamı)
```

### 7.2 Chrome Extension Yapısı

```
Extension paketi için kullanılan dosyalar:

cheory/
├── /core/              ✓ (tamamı)
├── /games/             ✓ (tamamı)
├── /adapters/
│   └── extension-adapter.js  ✓
├── /assets/            ✓ (tamamı)
├── /locales/           ✓ (tamamı)
├── /styles/            ✓ (tamamı)
└── /extension/         ✓ (tamamı)
    ├── manifest.json   ✓ (extension manifest)
    └── ...
```

### 7.3 Ortak vs Platform-Özel

| Klasör | Web | PWA | Extension |
|--------|-----|-----|-----------|
| /core/ | Ortak | Ortak | Ortak |
| /games/ | Ortak | Ortak | Ortak |
| /styles/ | Ortak | Ortak | Ortak |
| /locales/ | Ortak | Ortak | Ortak |
| /assets/ | Ortak | Ortak | Ortak |
| /adapters/ | web-adapter | pwa-adapter | extension-adapter |
| /extension/ | - | - | Özel |
| /pwa/ | - | Özel | - |
| index.html | Kullanır | Kullanır | - |
| manifest.json (PWA) | - | Kullanır | - |

---

## 8. GELİŞTİRME WORKFLOW

### 8.1 Yeni Oyun Ekleme

```
1. /games/ altında yeni klasör oluştur:
   /games/new-game-name/

2. Zorunlu dosyaları ekle:
   - index.js
   - engine.js
   - view.js

3. _registry.js'e oyunu ekle

4. /locales/ içine çevirileri ekle:
   - en/games.json'a ekle
   - tr/games.json'a ekle

5. Opsiyonel:
   - strategies.js (bot varsa)
   - README.md (dokümantasyon)
   - /docs/games/ altına spec
```

### 8.2 Yeni Component Ekleme

```
1. Paylaşılan mı, oyun-özel mi karar ver

2. Paylaşılan ise:
   /core/components/new-component.js

3. Oyun-özel ise:
   /games/game-name/view.js içine

4. Stiller:
   /styles/components/new-component.css
   veya inline (Web Component shadow DOM)
```

### 8.3 Yeni Servis Ekleme

```
1. /core/services/ altına yeni dosya:
   /core/services/new-service.js

2. Barrel export'a ekle:
   /core/services/index.js

3. Shell'e bağla (gerekirse):
   /core/shell.js
```

---

## 9. DOSYA BOYUTU REHBERİ

### 9.1 Hedef Boyutlar

| Dosya Türü | Hedef | Maksimum |
|------------|-------|----------|
| JavaScript (per file) | < 200 satır | 500 satır |
| CSS (per file) | < 150 satır | 300 satır |
| JSON (i18n) | < 100 key | 200 key |
| Game module (toplam) | < 500 satır | 800 satır |

### 9.2 Büyük Dosya Stratejisi

Dosya çok büyüdüğünde:

```
1. Fonksiyon bazında böl:
   utils.js → dom-utils.js, format-utils.js

2. Feature bazında böl:
   engine.js → engine-core.js, engine-scoring.js

3. Alt klasör oluştur:
   /voting-lab/methods/plurality.js
```

---

## 10. CHECKLIST

### 10.1 Yeni Dosya Eklerken

- [ ] İsimlendirme kurallarına uygun mu?
- [ ] Doğru klasörde mi?
- [ ] Import path'leri relative mi?
- [ ] Export pattern'e uygun mu?
- [ ] Boyut limitleri içinde mi?

### 10.2 Yeni Klasör Eklerken

- [ ] Derinlik 3'ü geçmiyor mu?
- [ ] kebab-case mi?
- [ ] README.md gerekli mi?
- [ ] .gitkeep gerekli mi (boş klasör)?

### 10.3 Code Review

- [ ] Dosya doğru yerde mi?
- [ ] Gereksiz dosya var mı?
- [ ] Duplicate kod başka dosyada mı?
- [ ] Import/export temiz mi?

---

## 11. ANTİ-PATTERN'LER

### 11.1 Kaçınılması Gerekenler

| Anti-Pattern | Neden Kötü | Alternatif |
|--------------|------------|------------|
| `/src/` klasörü | Gereksiz nesting | Kök dizinde organize et |
| `/lib/` klasörü | npm convention | `/core/utils/` kullan |
| `/helpers/` | Çok genel | Specific isimler kullan |
| Deep nesting | Navigasyon zor | Flat yapı |
| index.js her yerde | Karışıklık | Açık isimler |
| Tek dev dosya | Bakım zor | Modüler böl |

### 11.2 Kötü Örnekler

```
✗ KÖTÜ:
/src/lib/utils/helpers/string/format.js

✓ İYİ:
/core/utils/format.js
```

```
✗ KÖTÜ:
/components/ui/common/shared/Button.js

✓ İYİ:
/core/components/button.js
```

---

## 12. VERSİYON KONTROLÜ

### 12.1 Commit Mesaj Formatı

```
<type>(<scope>): <description>

Örnekler:
feat(game): add Stag Hunt game
fix(auth): resolve token refresh issue
docs(spec): update architecture diagram
style(css): fix button hover state
refactor(store): simplify state structure
```

### 12.2 Branch Stratejisi

```
main            → Production-ready
develop         → Integration branch
feature/*       → Yeni özellikler
bugfix/*        → Hata düzeltmeleri
release/*       → Release hazırlık
```

---

## 13. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |

---

> **Sonraki Adım:** `CORE_ENGINE_CONTRACT.md` - Oyun motoru interface detayları
