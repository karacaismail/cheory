# CHEORY: Stil Rehberi
## STYLE_GUIDE.md

> Bu doküman, Cheory'nin görsel tasarım sistemini tanımlar.
> Tüm UI geliştirmeleri bu rehbere uymalıdır.

---

## 1. TASARIM FELSEFESİ

### 1.1 Temel Prensipler

| Prensip | Açıklama |
|---------|----------|
| Dark Mode First | Varsayılan tema koyu |
| Glassmorphism | Şeffaf, bulanık cam efekti |
| Bento Grid | Modüler kart tabanlı layout |
| Minimal | Az element, çok işlev |
| Accessible | Herkes için erişilebilir |
| Child-Friendly | Çocuk dostu, oyunsu |

### 1.2 İlham Kaynakları

| Kaynak | Alınan Öğe |
|--------|------------|
| Apple Design | Temizlik, tipografi, boşluk |
| Nicky Case | İnteraktivite, eğlence |
| Duolingo | Gamification, renkler |
| Linear | Dark mode, glassmorphism |

### 1.3 Hedef Kitleye Uyum

| Yaş Grubu | Stil Adaptasyonu |
|-----------|------------------|
| 8-12 | Daha canlı renkler, büyük butonlar, daha fazla animasyon |
| 13-16 | Dengeli, sofistike ama eğlenceli |

---

## 2. DESIGN TOKENS

### 2.1 Renk Sistemi

```css
:root {
  /* ═══════════════════════════════════════════
     RENK PALETİ - DARK MODE (Varsayılan)
     ═══════════════════════════════════════════ */
  
  /* Arka Plan Katmanları */
  --color-bg-base: #050505;           /* En derin arka plan */
  --color-bg-primary: #0a0a0a;        /* Ana arka plan */
  --color-bg-secondary: #141414;      /* Kart arka planı */
  --color-bg-tertiary: #1f1f1f;       /* Elevated elements */
  --color-bg-quaternary: #2a2a2a;     /* Hover states */
  
  /* Cam Efekti (Glassmorphism) */
  --color-glass-bg: rgba(255, 255, 255, 0.03);
  --color-glass-bg-hover: rgba(255, 255, 255, 0.06);
  --color-glass-border: rgba(255, 255, 255, 0.08);
  --color-glass-border-hover: rgba(255, 255, 255, 0.12);
  
  /* Metin Renkleri */
  --color-text-primary: #ffffff;       /* Ana metin */
  --color-text-secondary: #a1a1a1;     /* İkincil metin */
  --color-text-tertiary: #6b6b6b;      /* Üçüncül metin */
  --color-text-muted: #404040;         /* Çok soluk metin */
  --color-text-inverse: #0a0a0a;       /* Açık arka plan üstünde */
  
  /* Marka Renkleri */
  --color-brand-primary: #6366f1;      /* Indigo - Ana marka */
  --color-brand-secondary: #8b5cf6;    /* Violet - İkincil */
  --color-brand-gradient: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  
  /* Aksent Renkleri (Oyun Kategorileri) */
  --color-accent-classic: #6366f1;     /* Klasik oyunlar - Indigo */
  --color-accent-bargaining: #f59e0b;  /* Pazarlık - Amber */
  --color-accent-coordination: #10b981;/* Koordinasyon - Emerald */
  --color-accent-social: #ec4899;      /* Sosyal - Pink */
  --color-accent-zerosum: #ef4444;     /* Sıfır-toplam - Red */
  --color-accent-voting: #3b82f6;      /* Oylama - Blue */
  
  /* Durum Renkleri */
  --color-success: #22c55e;            /* Başarı - Green */
  --color-success-soft: rgba(34, 197, 94, 0.15);
  --color-warning: #f59e0b;            /* Uyarı - Amber */
  --color-warning-soft: rgba(245, 158, 11, 0.15);
  --color-error: #ef4444;              /* Hata - Red */
  --color-error-soft: rgba(239, 68, 68, 0.15);
  --color-info: #3b82f6;               /* Bilgi - Blue */
  --color-info-soft: rgba(59, 130, 246, 0.15);
  
  /* Oyun Renkleri */
  --color-cooperate: #22c55e;          /* İşbirliği - Yeşil */
  --color-defect: #ef4444;             /* İhanet - Kırmızı */
  --color-neutral: #6b7280;            /* Nötr - Gri */
  --color-player-1: #6366f1;           /* Oyuncu 1 - Indigo */
  --color-player-2: #f59e0b;           /* Oyuncu 2 - Amber */
  --color-bot: #8b5cf6;                /* Bot - Violet */
}
```

### 2.2 Light Mode Override

```css
[data-theme="light"] {
  /* Arka Plan Katmanları */
  --color-bg-base: #f5f5f5;
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #fafafa;
  --color-bg-tertiary: #f0f0f0;
  --color-bg-quaternary: #e5e5e5;
  
  /* Cam Efekti */
  --color-glass-bg: rgba(0, 0, 0, 0.02);
  --color-glass-bg-hover: rgba(0, 0, 0, 0.04);
  --color-glass-border: rgba(0, 0, 0, 0.06);
  --color-glass-border-hover: rgba(0, 0, 0, 0.1);
  
  /* Metin Renkleri */
  --color-text-primary: #0a0a0a;
  --color-text-secondary: #525252;
  --color-text-tertiary: #737373;
  --color-text-muted: #a3a3a3;
  --color-text-inverse: #ffffff;
  
  /* Durum renkleri aynı kalır */
}
```

### 2.3 Tipografi

```css
:root {
  /* ═══════════════════════════════════════════
     TİPOGRAFİ SİSTEMİ
     ═══════════════════════════════════════════ */
  
  /* Font Ailesi */
  --font-family-sans: system-ui, -apple-system, BlinkMacSystemFont, 
                      'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  --font-family-mono: ui-monospace, SFMono-Regular, 'SF Mono', Menlo, 
                      Consolas, 'Liberation Mono', monospace;
  
  /* Font Boyutları */
  --font-size-2xs: 0.625rem;    /* 10px */
  --font-size-xs: 0.75rem;      /* 12px */
  --font-size-sm: 0.875rem;     /* 14px */
  --font-size-base: 1rem;       /* 16px */
  --font-size-lg: 1.125rem;     /* 18px */
  --font-size-xl: 1.25rem;      /* 20px */
  --font-size-2xl: 1.5rem;      /* 24px */
  --font-size-3xl: 1.875rem;    /* 30px */
  --font-size-4xl: 2.25rem;     /* 36px */
  --font-size-5xl: 3rem;        /* 48px */
  
  /* Font Ağırlıkları */
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  
  /* Satır Yükseklikleri */
  --line-height-tight: 1.25;
  --line-height-snug: 1.375;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.625;
  --line-height-loose: 2;
  
  /* Harf Aralığı */
  --letter-spacing-tight: -0.025em;
  --letter-spacing-normal: 0;
  --letter-spacing-wide: 0.025em;
  --letter-spacing-wider: 0.05em;
}
```

### 2.4 Spacing (Boşluk)

```css
:root {
  /* ═══════════════════════════════════════════
     SPACING SİSTEMİ (8px Base)
     ═══════════════════════════════════════════ */
  
  --space-0: 0;
  --space-px: 1px;
  --space-0-5: 0.125rem;   /* 2px */
  --space-1: 0.25rem;      /* 4px */
  --space-1-5: 0.375rem;   /* 6px */
  --space-2: 0.5rem;       /* 8px */
  --space-2-5: 0.625rem;   /* 10px */
  --space-3: 0.75rem;      /* 12px */
  --space-3-5: 0.875rem;   /* 14px */
  --space-4: 1rem;         /* 16px */
  --space-5: 1.25rem;      /* 20px */
  --space-6: 1.5rem;       /* 24px */
  --space-7: 1.75rem;      /* 28px */
  --space-8: 2rem;         /* 32px */
  --space-9: 2.25rem;      /* 36px */
  --space-10: 2.5rem;      /* 40px */
  --space-11: 2.75rem;     /* 44px */
  --space-12: 3rem;        /* 48px */
  --space-14: 3.5rem;      /* 56px */
  --space-16: 4rem;        /* 64px */
  --space-20: 5rem;        /* 80px */
  --space-24: 6rem;        /* 96px */
}
```

### 2.5 Border Radius

```css
:root {
  /* ═══════════════════════════════════════════
     BORDER RADIUS
     ═══════════════════════════════════════════ */
  
  --radius-none: 0;
  --radius-sm: 0.25rem;    /* 4px */
  --radius-md: 0.5rem;     /* 8px */
  --radius-lg: 0.75rem;    /* 12px */
  --radius-xl: 1rem;       /* 16px */
  --radius-2xl: 1.5rem;    /* 24px */
  --radius-3xl: 2rem;      /* 32px */
  --radius-full: 9999px;   /* Tam yuvarlak */
}
```

### 2.6 Shadows

```css
:root {
  /* ═══════════════════════════════════════════
     GÖLGE SİSTEMİ (Dark Mode Optimized)
     ═══════════════════════════════════════════ */
  
  --shadow-xs: 0 1px 2px rgba(0, 0, 0, 0.5);
  --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.5);
  --shadow-md: 0 4px 8px rgba(0, 0, 0, 0.5);
  --shadow-lg: 0 8px 16px rgba(0, 0, 0, 0.5);
  --shadow-xl: 0 16px 32px rgba(0, 0, 0, 0.5);
  --shadow-2xl: 0 24px 48px rgba(0, 0, 0, 0.5);
  
  /* Glow efektleri */
  --shadow-glow-brand: 0 0 20px rgba(99, 102, 241, 0.4);
  --shadow-glow-success: 0 0 20px rgba(34, 197, 94, 0.4);
  --shadow-glow-error: 0 0 20px rgba(239, 68, 68, 0.4);
  
  /* İç gölge */
  --shadow-inner: inset 0 2px 4px rgba(0, 0, 0, 0.3);
}

[data-theme="light"] {
  --shadow-xs: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 8px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 8px 16px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 16px 32px rgba(0, 0, 0, 0.15);
  --shadow-2xl: 0 24px 48px rgba(0, 0, 0, 0.2);
  --shadow-inner: inset 0 2px 4px rgba(0, 0, 0, 0.06);
}
```

### 2.7 Transitions & Animations

```css
:root {
  /* ═══════════════════════════════════════════
     GEÇİŞ VE ANİMASYON
     ═══════════════════════════════════════════ */
  
  /* Süreler */
  --duration-instant: 50ms;
  --duration-fast: 150ms;
  --duration-normal: 250ms;
  --duration-slow: 350ms;
  --duration-slower: 500ms;
  --duration-slowest: 700ms;
  
  /* Easing fonksiyonları */
  --ease-linear: linear;
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55);
  --ease-elastic: cubic-bezier(0.68, -0.6, 0.32, 1.6);
  
  /* Hazır geçişler */
  --transition-fast: all var(--duration-fast) var(--ease-out);
  --transition-normal: all var(--duration-normal) var(--ease-out);
  --transition-slow: all var(--duration-slow) var(--ease-out);
  --transition-colors: color var(--duration-fast) var(--ease-out),
                       background-color var(--duration-fast) var(--ease-out),
                       border-color var(--duration-fast) var(--ease-out);
  --transition-transform: transform var(--duration-normal) var(--ease-out);
  --transition-opacity: opacity var(--duration-normal) var(--ease-out);
}
```

### 2.8 Z-Index Katmanları

```css
:root {
  /* ═══════════════════════════════════════════
     Z-INDEX KATMANLARI
     ═══════════════════════════════════════════ */
  
  --z-behind: -1;
  --z-base: 0;
  --z-raised: 10;
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-overlay: 300;
  --z-modal: 400;
  --z-popover: 500;
  --z-tooltip: 600;
  --z-toast: 700;
  --z-max: 9999;
}
```

### 2.9 Layout Değerleri

```css
:root {
  /* ═══════════════════════════════════════════
     LAYOUT DEĞERLERİ
     ═══════════════════════════════════════════ */
  
  /* Container genişlikleri */
  --container-sm: 640px;
  --container-md: 768px;
  --container-lg: 1024px;
  --container-xl: 1280px;
  --container-2xl: 1536px;
  
  /* Shell ölçüleri */
  --header-height: 56px;
  --sidebar-width: 240px;
  --sidebar-collapsed: 64px;
  --footer-height: 48px;
  
  /* Side Panel (Extension) */
  --sidepanel-width: 400px;
  --sidepanel-min-width: 320px;
  
  /* Game area */
  --game-max-width: 800px;
  --game-padding: var(--space-4);
  
  /* Touch target (minimum) */
  --touch-target-min: 44px;
}
```

---

## 3. GLASSMORPHISM

### 3.1 Glass Kartı

```css
.glass-card {
  background: var(--color-glass-bg);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-xl);
}

.glass-card:hover {
  background: var(--color-glass-bg-hover);
  border-color: var(--color-glass-border-hover);
}
```

### 3.2 Glass Varyasyonları

```css
/* Hafif Glass */
.glass-subtle {
  background: rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.05);
}

/* Orta Glass */
.glass-medium {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

/* Güçlü Glass */
.glass-strong {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.15);
}

/* Renkli Glass */
.glass-brand {
  background: rgba(99, 102, 241, 0.1);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(99, 102, 241, 0.2);
}
```

### 3.3 Glass Kullanım Kuralları

| Kullanım Yeri | Varyasyon | Blur |
|---------------|-----------|------|
| Kartlar | Medium | 12px |
| Navigation | Strong | 20px |
| Modal overlay | Subtle | 8px |
| Floating buttons | Strong | 16px |
| Tooltips | Medium | 12px |

---

## 4. BENTO GRID

### 4.1 Grid Temeli

```css
.bento-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-4);
  padding: var(--space-4);
}
```

### 4.2 Grid Varyasyonları

```css
/* 2 Sütunlu (Tablet) */
.bento-grid-2 {
  grid-template-columns: repeat(2, 1fr);
}

/* 3 Sütunlu (Desktop) */
.bento-grid-3 {
  grid-template-columns: repeat(3, 1fr);
}

/* 4 Sütunlu (Geniş Desktop) */
.bento-grid-4 {
  grid-template-columns: repeat(4, 1fr);
}

/* Responsive */
@media (max-width: 768px) {
  .bento-grid-2,
  .bento-grid-3,
  .bento-grid-4 {
    grid-template-columns: 1fr;
  }
}

@media (min-width: 769px) and (max-width: 1024px) {
  .bento-grid-3,
  .bento-grid-4 {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

### 4.3 Bento Kartları

```css
/* Temel Kart */
.bento-card {
  background: var(--color-glass-bg);
  backdrop-filter: blur(12px);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-xl);
  padding: var(--space-6);
  transition: var(--transition-normal);
}

.bento-card:hover {
  background: var(--color-glass-bg-hover);
  border-color: var(--color-glass-border-hover);
  transform: translateY(-2px);
}

/* Span varyasyonları */
.bento-card--span-2 {
  grid-column: span 2;
}

.bento-card--span-3 {
  grid-column: span 3;
}

.bento-card--span-row {
  grid-row: span 2;
}

.bento-card--span-full {
  grid-column: 1 / -1;
}

/* Boyut varyasyonları */
.bento-card--sm {
  padding: var(--space-4);
}

.bento-card--lg {
  padding: var(--space-8);
}
```

### 4.4 Örnek Bento Layout

```
┌──────────────────────────────────────────────────┐
│  ┌──────────────────────┐  ┌──────────────────┐  │
│  │                      │  │                  │  │
│  │   Featured Game      │  │   Quick Stats    │  │
│  │   (span-2)           │  │                  │  │
│  │                      │  │                  │  │
│  └──────────────────────┘  └──────────────────┘  │
│                                                   │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  │
│  │ Game 1 │  │ Game 2 │  │ Game 3 │  │ Game 4 │  │
│  └────────┘  └────────┘  └────────┘  └────────┘  │
│                                                   │
│  ┌────────┐  ┌──────────────────────────────┐   │
│  │        │  │                              │   │
│  │ Stats  │  │   Leaderboard (span-2)       │   │
│  │(span-  │  │                              │   │
│  │ row)   │  └──────────────────────────────┘   │
│  │        │  ┌──────────────────────────────┐   │
│  │        │  │   Achievements (span-2)      │   │
│  └────────┘  └──────────────────────────────┘   │
└──────────────────────────────────────────────────┘
```

---

## 5. COMPONENT STİLLERİ

### 5.1 Butonlar

```css
/* Base Button */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  padding: var(--space-2-5) var(--space-4);
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-medium);
  line-height: var(--line-height-tight);
  border-radius: var(--radius-lg);
  border: none;
  cursor: pointer;
  transition: var(--transition-fast);
  min-height: var(--touch-target-min);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Primary */
.btn-primary {
  background: var(--color-brand-gradient);
  color: var(--color-text-primary);
  box-shadow: var(--shadow-glow-brand);
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 0 30px rgba(99, 102, 241, 0.5);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(0);
}

/* Secondary */
.btn-secondary {
  background: var(--color-glass-bg);
  color: var(--color-text-primary);
  border: 1px solid var(--color-glass-border);
  backdrop-filter: blur(8px);
}

.btn-secondary:hover:not(:disabled) {
  background: var(--color-glass-bg-hover);
  border-color: var(--color-glass-border-hover);
}

/* Ghost */
.btn-ghost {
  background: transparent;
  color: var(--color-text-secondary);
}

.btn-ghost:hover:not(:disabled) {
  background: var(--color-glass-bg);
  color: var(--color-text-primary);
}

/* Success / Danger */
.btn-success {
  background: var(--color-success);
  color: white;
}

.btn-danger {
  background: var(--color-error);
  color: white;
}

/* Boyutlar */
.btn-sm {
  padding: var(--space-1-5) var(--space-3);
  font-size: var(--font-size-xs);
  min-height: 32px;
}

.btn-lg {
  padding: var(--space-3) var(--space-6);
  font-size: var(--font-size-base);
  min-height: 52px;
}

/* Icon-only */
.btn-icon {
  padding: var(--space-2);
  width: var(--touch-target-min);
  height: var(--touch-target-min);
}
```

### 5.2 Form Elemanları

```css
/* Input */
.input {
  width: 100%;
  padding: var(--space-3) var(--space-4);
  font-size: var(--font-size-base);
  color: var(--color-text-primary);
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-lg);
  transition: var(--transition-fast);
  min-height: var(--touch-target-min);
}

.input:focus {
  outline: none;
  border-color: var(--color-brand-primary);
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2);
}

.input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.input::placeholder {
  color: var(--color-text-tertiary);
}

/* Select */
.select {
  appearance: none;
  background-image: url("data:image/svg+xml,..."); /* Chevron icon */
  background-repeat: no-repeat;
  background-position: right var(--space-3) center;
  padding-right: var(--space-10);
}

/* Checkbox & Radio */
.checkbox,
.radio {
  width: 20px;
  height: 20px;
  accent-color: var(--color-brand-primary);
}

/* Slider */
.slider {
  width: 100%;
  height: 6px;
  background: var(--color-bg-tertiary);
  border-radius: var(--radius-full);
  appearance: none;
}

.slider::-webkit-slider-thumb {
  width: 20px;
  height: 20px;
  background: var(--color-brand-primary);
  border-radius: var(--radius-full);
  cursor: pointer;
  appearance: none;
}
```

### 5.3 Kartlar

```css
/* Game Card */
.game-card {
  position: relative;
  overflow: hidden;
  background: var(--color-glass-bg);
  backdrop-filter: blur(12px);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-xl);
  padding: var(--space-5);
  cursor: pointer;
  transition: var(--transition-normal);
}

.game-card:hover {
  transform: translateY(-4px);
  border-color: var(--color-brand-primary);
  box-shadow: var(--shadow-glow-brand);
}

.game-card__icon {
  width: 48px;
  height: 48px;
  margin-bottom: var(--space-3);
}

.game-card__title {
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-primary);
  margin-bottom: var(--space-2);
}

.game-card__description {
  font-size: var(--font-size-sm);
  color: var(--color-text-secondary);
  line-height: var(--line-height-relaxed);
}

.game-card__badge {
  position: absolute;
  top: var(--space-3);
  right: var(--space-3);
  padding: var(--space-1) var(--space-2);
  font-size: var(--font-size-2xs);
  font-weight: var(--font-weight-semibold);
  text-transform: uppercase;
  border-radius: var(--radius-md);
}

.game-card__badge--new {
  background: var(--color-success);
  color: white;
}

.game-card__badge--premium {
  background: var(--color-brand-gradient);
  color: white;
}

.game-card__badge--locked {
  background: var(--color-bg-tertiary);
  color: var(--color-text-tertiary);
}
```

### 5.4 Modal

```css
/* Modal Overlay */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-4);
  z-index: var(--z-modal);
  animation: fadeIn var(--duration-fast) var(--ease-out);
}

/* Modal Content */
.modal {
  width: 100%;
  max-width: 480px;
  max-height: 90vh;
  overflow-y: auto;
  background: var(--color-bg-secondary);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-2xl);
  animation: slideUp var(--duration-normal) var(--ease-out);
}

.modal__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-5);
  border-bottom: 1px solid var(--color-glass-border);
}

.modal__title {
  font-size: var(--font-size-xl);
  font-weight: var(--font-weight-semibold);
}

.modal__body {
  padding: var(--space-5);
}

.modal__footer {
  display: flex;
  gap: var(--space-3);
  justify-content: flex-end;
  padding: var(--space-5);
  border-top: 1px solid var(--color-glass-border);
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { 
    opacity: 0;
    transform: translateY(20px);
  }
  to { 
    opacity: 1;
    transform: translateY(0);
  }
}
```

### 5.5 Toast / Notification

```css
.toast-container {
  position: fixed;
  bottom: var(--space-6);
  right: var(--space-6);
  z-index: var(--z-toast);
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}

.toast {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-4);
  background: var(--color-bg-secondary);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  animation: slideIn var(--duration-normal) var(--ease-out);
}

.toast--success {
  border-left: 4px solid var(--color-success);
}

.toast--error {
  border-left: 4px solid var(--color-error);
}

.toast--warning {
  border-left: 4px solid var(--color-warning);
}

.toast--info {
  border-left: 4px solid var(--color-info);
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
```

---

## 6. TİPOGRAFİ STİLLERİ

### 6.1 Başlıklar

```css
.heading-1 {
  font-size: var(--font-size-4xl);
  font-weight: var(--font-weight-bold);
  line-height: var(--line-height-tight);
  letter-spacing: var(--letter-spacing-tight);
  color: var(--color-text-primary);
}

.heading-2 {
  font-size: var(--font-size-3xl);
  font-weight: var(--font-weight-bold);
  line-height: var(--line-height-tight);
  color: var(--color-text-primary);
}

.heading-3 {
  font-size: var(--font-size-2xl);
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-snug);
  color: var(--color-text-primary);
}

.heading-4 {
  font-size: var(--font-size-xl);
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-snug);
  color: var(--color-text-primary);
}

.heading-5 {
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-medium);
  line-height: var(--line-height-normal);
  color: var(--color-text-primary);
}

.heading-6 {
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-medium);
  line-height: var(--line-height-normal);
  color: var(--color-text-primary);
}
```

### 6.2 Metin Stilleri

```css
.text-body {
  font-size: var(--font-size-base);
  line-height: var(--line-height-relaxed);
  color: var(--color-text-secondary);
}

.text-body-sm {
  font-size: var(--font-size-sm);
  line-height: var(--line-height-relaxed);
  color: var(--color-text-secondary);
}

.text-caption {
  font-size: var(--font-size-xs);
  line-height: var(--line-height-normal);
  color: var(--color-text-tertiary);
}

.text-label {
  font-size: var(--font-size-xs);
  font-weight: var(--font-weight-semibold);
  text-transform: uppercase;
  letter-spacing: var(--letter-spacing-wider);
  color: var(--color-text-tertiary);
}

.text-mono {
  font-family: var(--font-family-mono);
  font-size: var(--font-size-sm);
}
```

---

## 7. OYUN-SPESİFİK STİLLER

### 7.1 Oyun Tahtası

```css
.game-board {
  position: relative;
  width: 100%;
  max-width: var(--game-max-width);
  margin: 0 auto;
  padding: var(--space-6);
  background: var(--color-glass-bg);
  border: 1px solid var(--color-glass-border);
  border-radius: var(--radius-2xl);
}
```

### 7.2 Seçim Butonları (PD için)

```css
.choice-btn {
  flex: 1;
  padding: var(--space-6);
  font-size: var(--font-size-xl);
  font-weight: var(--font-weight-bold);
  border-radius: var(--radius-xl);
  border: 2px solid transparent;
  transition: var(--transition-normal);
}

.choice-btn--cooperate {
  background: var(--color-success-soft);
  color: var(--color-success);
  border-color: var(--color-success);
}

.choice-btn--cooperate:hover {
  background: var(--color-success);
  color: white;
  transform: scale(1.02);
}

.choice-btn--defect {
  background: var(--color-error-soft);
  color: var(--color-error);
  border-color: var(--color-error);
}

.choice-btn--defect:hover {
  background: var(--color-error);
  color: white;
  transform: scale(1.02);
}
```

### 7.3 Skor Gösterimi

```css
.score-display {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-4);
  background: var(--color-bg-tertiary);
  border-radius: var(--radius-lg);
}

.score-player {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--space-2);
}

.score-player__name {
  font-size: var(--font-size-sm);
  color: var(--color-text-secondary);
}

.score-player__value {
  font-size: var(--font-size-3xl);
  font-weight: var(--font-weight-bold);
  color: var(--color-text-primary);
}

.score-player--winner {
  color: var(--color-success);
}

.score-player--loser {
  color: var(--color-error);
}

.score-separator {
  font-size: var(--font-size-2xl);
  color: var(--color-text-muted);
}
```

### 7.4 Payoff Matrix Gösterimi

```css
.payoff-matrix {
  display: grid;
  grid-template-columns: auto 1fr 1fr;
  gap: 2px;
  background: var(--color-glass-border);
  border-radius: var(--radius-lg);
  overflow: hidden;
}

.payoff-matrix__cell {
  padding: var(--space-3);
  background: var(--color-bg-secondary);
  text-align: center;
}

.payoff-matrix__header {
  font-weight: var(--font-weight-semibold);
  background: var(--color-bg-tertiary);
}

.payoff-matrix__value {
  font-family: var(--font-family-mono);
  font-size: var(--font-size-sm);
}

.payoff-matrix__highlight {
  background: var(--color-brand-primary);
  color: white;
}
```

---

## 8. ANİMASYONLAR

### 8.1 Giriş Animasyonları

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInScale {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slideInLeft {
  from {
    opacity: 0;
    transform: translateX(-100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
```

### 8.2 Attention Animasyonları

```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes ping {
  75%, 100% {
    transform: scale(2);
    opacity: 0;
  }
}
```

### 8.3 Oyun Animasyonları

```css
@keyframes scoreUp {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
    color: var(--color-success);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes scoreDown {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(0.9);
    color: var(--color-error);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes reveal {
  0% {
    transform: rotateY(180deg);
    opacity: 0;
  }
  100% {
    transform: rotateY(0);
    opacity: 1;
  }
}

@keyframes celebrate {
  0% { transform: scale(1); }
  25% { transform: scale(1.1) rotate(-5deg); }
  50% { transform: scale(1.1) rotate(5deg); }
  75% { transform: scale(1.1) rotate(-5deg); }
  100% { transform: scale(1); }
}
```

### 8.4 Animasyon Utility Classes

```css
.animate-fade-in { animation: fadeIn var(--duration-normal) var(--ease-out); }
.animate-fade-in-up { animation: fadeInUp var(--duration-normal) var(--ease-out); }
.animate-fade-in-down { animation: fadeInDown var(--duration-normal) var(--ease-out); }
.animate-slide-in-right { animation: slideInRight var(--duration-normal) var(--ease-out); }
.animate-slide-in-left { animation: slideInLeft var(--duration-normal) var(--ease-out); }
.animate-pulse { animation: pulse 2s var(--ease-in-out) infinite; }
.animate-bounce { animation: bounce 1s var(--ease-in-out) infinite; }
.animate-spin { animation: spin 1s linear infinite; }

/* Delay utilities */
.animate-delay-100 { animation-delay: 100ms; }
.animate-delay-200 { animation-delay: 200ms; }
.animate-delay-300 { animation-delay: 300ms; }
.animate-delay-500 { animation-delay: 500ms; }
```

---

## 9. RESPONSIVE TASARIM

### 9.1 Breakpoint'ler

```css
/* Mobile First Yaklaşımı */

/* Small (telefon): 0 - 639px (varsayılan) */

/* Medium (tablet): 640px+ */
@media (min-width: 640px) { }

/* Large (laptop): 1024px+ */
@media (min-width: 1024px) { }

/* XLarge (desktop): 1280px+ */
@media (min-width: 1280px) { }

/* 2XLarge (geniş ekran): 1536px+ */
@media (min-width: 1536px) { }
```

### 9.2 Responsive Utilities

```css
/* Gizleme */
.hide-mobile { display: none; }
@media (min-width: 640px) { .hide-mobile { display: initial; } }

.hide-desktop { display: initial; }
@media (min-width: 1024px) { .hide-desktop { display: none; } }

/* Spacing responsive */
@media (max-width: 639px) {
  .p-responsive { padding: var(--space-3); }
  .gap-responsive { gap: var(--space-3); }
}

@media (min-width: 640px) {
  .p-responsive { padding: var(--space-4); }
  .gap-responsive { gap: var(--space-4); }
}

@media (min-width: 1024px) {
  .p-responsive { padding: var(--space-6); }
  .gap-responsive { gap: var(--space-6); }
}
```

### 9.3 Side Panel Uyumu

```css
/* Extension Side Panel (400px max) */
@media (max-width: 420px) {
  .bento-grid {
    grid-template-columns: 1fr;
    gap: var(--space-3);
    padding: var(--space-3);
  }
  
  .bento-card--span-2,
  .bento-card--span-3 {
    grid-column: span 1;
  }
  
  .modal {
    max-width: 100%;
    border-radius: var(--radius-xl);
  }
}
```

---

## 10. ERİŞİLEBİLİRLİK

### 10.1 Focus Stilleri

```css
/* Görünür focus ring */
:focus-visible {
  outline: 2px solid var(--color-brand-primary);
  outline-offset: 2px;
}

/* Focus within for groups */
.focus-within:focus-within {
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.3);
}

/* Skip to content link */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: var(--space-2) var(--space-4);
  background: var(--color-brand-primary);
  color: white;
  z-index: var(--z-max);
}

.skip-link:focus {
  top: 0;
}
```

### 10.2 Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 10.3 High Contrast

```css
@media (prefers-contrast: high) {
  :root {
    --color-glass-border: rgba(255, 255, 255, 0.3);
    --color-text-secondary: #d4d4d4;
  }
  
  .btn {
    border: 2px solid currentColor;
  }
}
```

---

## 11. ICON SİSTEMİ

### 11.1 FontAwesome Kullanımı

```css
/* FA class'ları için alias'lar */
.icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.icon-sm { font-size: 14px; }
.icon-md { font-size: 18px; }
.icon-lg { font-size: 24px; }
.icon-xl { font-size: 32px; }
```

### 11.2 Inline SVG Standartları

```css
.svg-icon {
  width: 1em;
  height: 1em;
  fill: currentColor;
  vertical-align: middle;
}

.svg-icon--stroke {
  fill: none;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}
```

---

## 12. KULLANIM REHBERİ

### 12.1 CSS Dosya Yapısı

```
/styles/
├── main.css              # Import hub
├── /base/
│   ├── reset.css         # CSS reset
│   ├── variables.css     # Design tokens
│   └── typography.css    # Font stilleri
├── /components/
│   ├── buttons.css
│   ├── cards.css
│   ├── forms.css
│   ├── modals.css
│   └── ...
├── /layouts/
│   ├── grid.css          # Bento grid
│   ├── shell.css         # Ana layout
│   └── game.css          # Oyun layout
├── /themes/
│   ├── dark.css          # Dark mode (default)
│   └── light.css         # Light mode
└── /utilities/
    ├── animations.css
    ├── responsive.css
    └── helpers.css
```

### 12.2 Class Naming Convention

```
BEM-inspired ama basitleştirilmiş:

Block:          .game-card
Element:        .game-card__title
Modifier:       .game-card--featured

State:          .is-active, .is-loading, .is-disabled
Utility:        .text-center, .mt-4, .flex

Örnek:
<div class="game-card game-card--featured is-active">
  <h3 class="game-card__title">Title</h3>
  <p class="game-card__description text-secondary">Desc</p>
</div>
```

### 12.3 Dark/Light Mode Geçişi

```javascript
// Theme toggle yaklaşımı (pseudo-kod)
function toggleTheme() {
  const current = document.documentElement.dataset.theme;
  const next = current === 'dark' ? 'light' : 'dark';
  document.documentElement.dataset.theme = next;
  localStorage.setItem('theme', next);
}

// Sistem tercihini dinle
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)');
prefersDark.addEventListener('change', (e) => {
  if (!localStorage.getItem('theme')) {
    document.documentElement.dataset.theme = e.matches ? 'dark' : 'light';
  }
});
```

---

## 13. CHECKLIST

### 13.1 Yeni Component Eklerken

- [ ] Design token'ları kullanıyor mu?
- [ ] Dark/light mode'da çalışıyor mu?
- [ ] Responsive mi?
- [ ] Erişilebilir mi (focus, contrast)?
- [ ] Touch target yeterli mi (44px+)?
- [ ] Animasyonlar reduced-motion'a uyumlu mu?

### 13.2 Design Review

- [ ] Glassmorphism tutarlı mı?
- [ ] Spacing sisteme uygun mu?
- [ ] Tipografi hiyerarşisi doğru mu?
- [ ] Renkler palette'den mi?
- [ ] Gölgeler tutarlı mı?

---

## 14. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |

---

> **Not:** Bu stil rehberi CSS kodları içerir ancak bunlar doğrudan kopyalanacak kod değil, yaklaşım ve standartları gösteren referanslardır.
