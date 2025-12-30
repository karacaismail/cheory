# MATCHING PENNIES
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `matching-pennies` |
| Versiyon | 1.0.0 |
| Kategori | zero-sum |
| Zorluk | easy |
| Tier | free |
| Durum | approved |

---

## 1. ÖZET

### 1.1 Kısa Açıklama

| Dil | Açıklama |
|-----|----------|
| EN | A pure zero-sum game - one player's gain is another's loss. Can you be unpredictable? |
| TR | Saf sıfır toplamlı oyun - birinin kazancı diğerinin kaybı. Tahmin edilemez olabilir misin? |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| Sıfır Toplamlı Oyun | Bir tarafın kazancı = diğerinin kaybı |
| Karışık Strateji | Tahmin edilemezliğin önemi |
| Randomizasyon | Neden bazen rastgele davranmalıyız |
| Blöf | Rakibin beklentisini boşa çıkarma |
| Oyun Teorisi Temeli | En saf oyun teorisi örneği |

### 1.3 Gerçek Hayat Bağlantısı

Bu oyun gerçek hayatta şu durumlara benzer:

- **Penaltı Atışları:** Kaleci sola mı sağa mı atlayacak?
- **Saklambaç:** Arayan nereye bakacak?
- **Poker Blöfü:** Blöf mü yapıyorsun, gerçek mi?
- **Tenis Servisi:** Forehand'e mi backhand'e mi?
- **Güvenlik:** Devriye hangi yoldan geçecek?

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. Her oyuncu bir madeni para gösterir
2. Yazı veya Tura seçilir
3. Eşleşen (Matcher) - Aynı olmasını ister
4. Eşleşmeyen (Mismatcher) - Farklı olmasını ister
5. Eşleşirse Matcher kazanır, farklıysa Mismatcher kazanır
```

### 2.2 Hikaye Bağlamı

```
İki oyuncu: Biri "Eşleştirici" (Matcher), biri "Kaçırıcı" (Mismatcher).

- Matcher: İkisi de aynı tarafı gösterirse kazanır
- Mismatcher: Farklı taraflar gösterilirse kazanır

Tek bir kazanan olabilir - bu saf rekabet!
```

### 2.3 Oyuncu Bilgisi

| Alan | Değer |
|------|-------|
| Minimum Oyuncu | 2 |
| Maximum Oyuncu | 2 |
| Optimal Oyuncu | 2 |
| Bot Desteği | Evet |
| Eşzamanlı Hamle | Evet |

### 2.4 Rol Ataması

```
Oyuncu 1: Matcher (Eşleştirici) - Aynı olsun ister
Oyuncu 2: Mismatcher (Kaçırıcı) - Farklı olsun ister

veya

Rastgele atama yapılabilir.
```

### 2.5 Kazanma Koşulu

```
Tek tur:
- Seçimler aynıysa → Matcher kazanır (+1, -1)
- Seçimler farklıysa → Mismatcher kazanır (-1, +1)

Toplam skor: Her zaman 0 (sıfır toplamlı)
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `CHOOSE` | Para tarafı seçimi | Henüz seçim yapılmamışsa |

### 3.2 Eylem Detayları

#### CHOOSE

```
Action {
  type: 'CHOOSE'
  playerId: string
  payload: {
    choice: 'heads' | 'tails'
  }
}
```

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi

```
                      Mismatcher (Oyuncu 2)
                   Yazı (H)     Tura (T)
Matcher   Yazı (H)  (+1, -1)    (-1, +1)
(Oyuncu 1) Tura (T)  (-1, +1)    (+1, -1)
```

| Sonuç | Matcher | Mismatcher | Açıklama |
|-------|---------|------------|----------|
| H-H | +1 | -1 | Eşleşti! Matcher kazandı |
| H-T | -1 | +1 | Eşleşmedi! Mismatcher kazandı |
| T-H | -1 | +1 | Eşleşmedi! Mismatcher kazandı |
| T-T | +1 | -1 | Eşleşti! Matcher kazandı |

### 4.2 Oyun Teorisi Analizi

```
Saf strateji Nash dengesi: YOK!

Karışık strateji Nash dengesi:
- Her iki oyuncu %50 Yazı, %50 Tura oynamalı
- Beklenen değer: 0 (her iki taraf için)

Bu oyun "strictly competitive" - çıkarlar tamamen zıt.
```

### 4.3 Skor Hesaplama

```
function calculatePayoff(matcherChoice, mismatcherChoice):
    if (matcherChoice === mismatcherChoice):
        return { matcher: 1, mismatcher: -1 }  // Eşleşti
    else:
        return { matcher: -1, mismatcher: 1 }  // Eşleşmedi
```

### 4.4 Sıfır Toplam Kontrolü

```
// Her zaman doğrulanmalı:
matcher_score + mismatcher_score === 0
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  // Roller
  matcherId: string
  mismatcherId: string
  
  // Seçimler
  matcherChoice: 'heads' | 'tails' | null
  mismatcherChoice: 'heads' | 'tails' | null
  
  // Sonuç
  matched: boolean | null
  winnerId: string | null
}
```

### 5.2 Örnek Initial State

```
{
  gameId: "game-mp-001",
  moduleId: "matching-pennies",
  
  players: [
    { id: "player-1", type: "human", name: "Eşleştirici", score: 0, role: "matcher" },
    { id: "player-2", type: "bot", name: "Kaçırıcı Bot", score: 0, role: "mismatcher" }
  ],
  
  phase: "playing",
  round: 1,
  maxRounds: 1,
  
  data: {
    matcherId: "player-1",
    mismatcherId: "player-2",
    matcherChoice: null,
    mismatcherChoice: null,
    matched: null,
    winnerId: null
  },
  
  history: [],
  result: null
}
```

### 5.3 Örnek Final State

```
{
  gameId: "game-mp-001",
  moduleId: "matching-pennies",
  
  players: [
    { id: "player-1", type: "human", name: "Eşleştirici", score: 1 },
    { id: "player-2", type: "bot", name: "Kaçırıcı Bot", score: -1 }
  ],
  
  phase: "finished",
  
  data: {
    matcherId: "player-1",
    mismatcherId: "player-2",
    matcherChoice: "heads",
    mismatcherChoice: "heads",
    matched: true,
    winnerId: "player-1"
  },
  
  result: {
    winnerId: "player-1",
    finalScores: { "player-1": 1, "player-2": -1 },
    summary: {
      en: "Matched! Both showed Heads. Matcher wins!",
      tr: "Eşleşti! İkisi de Yazı gösterdi. Eşleştirici kazandı!"
    }
  }
}
```

---

## 6. UI TASARIMI

### 6.1 Main Game Layout

```
+-----------------------------------------------+
|                HEADER                          |
|  Madeni Para Eşleştirme  |  Rol: Eşleştirici  |
+-----------------------------------------------+
|                                               |
|  ROLE EXPLANATION                             |
|  +-------------------------------------------+|
|  | Sen: EŞLEŞTİRİCİ 🎯                       ||
|  | Hedefin: Rakiple AYNI tarafı göster       ||
|  |                                           ||
|  | Rakip: KAÇIRICI 🎭                        ||
|  | Hedefi: Senden FARKLI taraf göstermek     ||
|  +-------------------------------------------+|
|                                               |
|  COINS DISPLAY                                |
|  +-------------------------------------------+|
|  |                                           ||
|  |     🪙 ?           🪙 ?                   ||
|  |    (Sen)         (Rakip)                  ||
|  |                                           ||
|  +-------------------------------------------+|
|                                               |
+-----------------------------------------------+
|  CHOICE BUTTONS                               |
|                                               |
|  +--------------------+  +------------------+ |
|  |     YAZI          |  |      TURA       | |
|  |       👤           |  |       🦅        | |
|  +--------------------+  +------------------+ |
|                                               |
+-----------------------------------------------+
```

### 6.2 Reveal Screen

```
+-----------------------------------------------+
|              SONUÇ AÇIKLANDI!                 |
+-----------------------------------------------+
|                                               |
|     🪙 YAZI         🪙 YAZI                   |
|      (Sen)          (Rakip)                   |
|                                               |
|           ✓ EŞLEŞTİ!                         |
|                                               |
|     🎯 Eşleştirici KAZANDI! 🎯               |
|           +1 puan                             |
|                                               |
+-----------------------------------------------+
|  [Tekrar Oyna]  [Rol Değiştir]  [Çık]       |
+-----------------------------------------------+
```

### 6.3 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| Coin | `<mp-coin>` | Para animasyonu (flip) |
| Role Badge | `<mp-role-badge>` | Matcher/Mismatcher rozeti |
| Choice Button | `<mp-choice-button>` | Yazı/Tura butonu |
| Match Indicator | `<mp-match-indicator>` | Eşleşme göstergesi |
| Score Display | `<mp-score>` | +1/-1 gösterimi |

### 6.4 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| Coin flip | Reveal başlangıcı | 1000ms |
| Match glow | Eşleşme olunca | 500ms |
| Mismatch shake | Eşleşmeme olunca | 300ms |
| Score pop | Skor gösterilince | 400ms |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `random` | Rastgele | Easy | %50-%50 (optimal!) |
| `heads-bias` | Yazı Yanlısı | Easy | %70 yazı |
| `tails-bias` | Tura Yanlısı | Easy | %70 tura |
| `pattern-detect` | Kalıp Avcısı | Medium | Oyuncunun kalıbını bulmaya çalışır |
| `anti-pattern` | Kalıp Kırıcı | Hard | Kendi kalıbını gizler |

### 7.2 Strateji Detayları

#### random: Rastgele (Nash Optimum)

```
function decide(state, playerId):
    // Bu aslında EN İYİ strateji!
    if (Math.random() < 0.5):
        return 'heads'
    return 'tails'
```

**Özellikler:**
- Matematiksel olarak optimal
- Rakip tarafından sömürülemez
- Beklenen değer: 0

#### pattern-detect: Kalıp Avcısı

```
function decide(state, playerId):
    history = getOpponentHistory()
    
    if (history.length < 3):
        return random()
    
    // Son 5 seçimde ağırlık analizi
    headsCount = count(history.slice(-5), 'heads')
    
    if (headsCount > 3):
        // Rakip yazı yanlısı, ona göre oyna
        if (isMatcher):
            return 'heads'  // Eşleşmeye çalış
        else:
            return 'tails'  // Kaçırmaya çalış
    
    // Benzer analiz diğer durum için...
    return random()
```

### 7.3 Varsayılan Strateji

```
defaultStrategyId: 'random'

Zorluk bazlı:
- Easy: 'heads-bias', 'tails-bias'
- Medium: 'random'
- Hard: 'pattern-detect', 'anti-pattern'
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef |
|------|--------|-------|
| 1 | Roller | Matcher vs Mismatcher açıklaması |
| 2 | Hedef | Senin hedefin ne? |
| 3 | Sıfır Toplam | Birinin kazancı = diğerinin kaybı |
| 4 | Strateji | Neden rastgele oynamalısın? |
| 5 | Dene | İlk seçimi yap |

### 8.2 Tutorial Ayarları

```
{
  isSkippable: true,
  showOnFirstPlay: true,
  estimatedTime: '2 min'
}
```

---

## 9. EDGE CASES

### 9.1 Olası Durumlar

| # | Durum | Beklenen Davranış |
|---|-------|-------------------|
| E1 | Uzun eşleşme serisi | Analiz için göster |
| E2 | Rol değiştirme isteği | Setup'ta izin ver |
| E3 | Çok oyunlu mod | Rolleri dönüşümlü ver |

---

## 10. TEST SENARYOLARI

### 10.1 Given-When-Then Senaryoları

#### Senaryo 1: Matcher Kazanır

```
GIVEN: Oyuncu Matcher rolünde
WHEN: Her iki oyuncu "Yazı" seçer
THEN: Oyuncu (Matcher) +1 puan alır
  AND: Bot (Mismatcher) -1 puan alır
  AND: "Eşleşti!" mesajı gösterilir
```

#### Senaryo 2: Mismatcher Kazanır

```
GIVEN: Oyuncu Mismatcher rolünde
WHEN: Oyuncu "Yazı", Rakip "Tura" seçer
THEN: Oyuncu (Mismatcher) +1 puan alır
  AND: Rakip (Matcher) -1 puan alır
  AND: "Eşleşmedi!" mesajı gösterilir
```

#### Senaryo 3: Sıfır Toplam Kontrolü

```
GIVEN: Herhangi bir oyun sonucu
WHEN: Skorlar hesaplanır
THEN: matcher_score + mismatcher_score === 0
```

---

## 11. METRİKLER

### 11.1 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:matching-pennies:start` | Oyun başı | `{ playerRole }` |
| `game:matching-pennies:choice` | Seçim | `{ choice, role }` |
| `game:matching-pennies:complete` | Oyun sonu | `{ matched, winner }` |

### 11.2 Oyun Metrikleri

| Metrik | Açıklama |
|--------|----------|
| Match Rate | Eşleşme oranı (beklenen: %50) |
| Choice Distribution | Yazı/Tura dağılımı (beklenen: %50-%50) |
| Role Win Rate | Matcher mı Mismatcher mı daha çok kazanıyor |

---

## 12. ACHIEVEMENTS

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `mp-first-match` | İlk Eşleşme | Matcher olarak kazan | Matcher win |
| `mp-first-mismatch` | İlk Kaçırma | Mismatcher olarak kazan | Mismatcher win |
| `mp-streak-3` | Seri | 3 oyun üst üste kazan | 3 ardışık win |
| `mp-balanced` | Dengeli | 10 oyunda %45-55 yazı oranı | Balanced choices |
| `mp-unpredictable` | Tahmin Edilemez | Pattern-detect botu yen | Beat pattern bot |

---

## 13. i18n KEYS

### 13.1 İngilizce

```json
"matching-pennies": {
  "name": "Matching Pennies",
  "description": "A zero-sum guessing game",
  "roles": {
    "matcher": "Matcher",
    "mismatcher": "Mismatcher"
  },
  "choices": {
    "heads": "Heads",
    "tails": "Tails"
  },
  "outcomes": {
    "matched": "Matched! Matcher wins!",
    "mismatched": "Mismatched! Mismatcher wins!"
  },
  "tutorial": {
    "zero_sum": "This is a zero-sum game: what you win, your opponent loses!"
  }
}
```

### 13.2 Türkçe

```json
"matching-pennies": {
  "name": "Madeni Para Eşleştirme",
  "description": "Sıfır toplamlı tahmin oyunu",
  "roles": {
    "matcher": "Eşleştirici",
    "mismatcher": "Kaçırıcı"
  },
  "choices": {
    "heads": "Yazı",
    "tails": "Tura"
  },
  "outcomes": {
    "matched": "Eşleşti! Eşleştirici kazandı!",
    "mismatched": "Eşleşmedi! Kaçırıcı kazandı!"
  },
  "tutorial": {
    "zero_sum": "Bu sıfır toplamlı oyun: senin kazancın = rakibin kaybı!"
  }
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/matching-pennies.svg` |
| Coin Heads | `/assets/images/games/coin-heads.svg` |
| Coin Tails | `/assets/images/games/coin-tails.svg` |

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| Free tier | Basit kurallar, temel konsept |
| Tek tur varsayılan | Sadelik, hız |
| Roller sabit | Öğretici açıdan daha net |

### 15.2 Pedagojik Değer

```
Bu oyun şunları öğretir:
1. Sıfır toplamlı oyun kavramı
2. Karışık strateji Nash dengesi
3. Neden bazen rastgele olmak gerekir
4. Pure strateji dengesinin olmayabileceği

Dikkat: Çocuklara "gambling" izlenimi vermemeli!
Para teması yerine renkler de kullanılabilir.
```

### 15.3 Alternatif Temalar

| Tema | Matcher | Mismatcher |
|------|---------|------------|
| Para | Yazı/Tura | Yazı/Tura |
| Renk | Kırmızı/Mavi | Kırmızı/Mavi |
| El | Sağ/Sol | Sağ/Sol |
| Kapı | Sol Kapı/Sağ Kapı | Sol Kapı/Sağ Kapı |

### 15.4 Gelecek İyileştirmeler

- [ ] Çok turlu versiyon
- [ ] Rol değiştirmeli turnuva
- [ ] Pattern analiz gösterimi
- [ ] Alternatif temalar

### 15.5 Referanslar

- Von Neumann & Morgenstern - Game Theory foundations
- Minimax theorem
- Mixed Strategy Nash Equilibrium

---

## 16. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |
