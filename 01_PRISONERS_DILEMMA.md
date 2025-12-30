# PRISONER'S DILEMMA
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `prisoners-dilemma` |
| Versiyon | 1.0.0 |
| Kategori | classic |
| Zorluk | easy |
| Tier | free |
| Durum | approved |

---

## 1. ÖZET

### 1.1 Kısa Açıklama

| Dil | Açıklama |
|-----|----------|
| EN | The classic game of cooperation vs betrayal - will you trust or betray? |
| TR | İşbirliği ve ihanet arasındaki klasik oyun - güvenecek misin yoksa ihanet mi edeceksin? |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| İşbirliği vs Rekabet | İki tarafın da kazanabileceği durumlar olduğunu gösterir |
| Güven | Başkalarına güvenmenin risklerini ve ödüllerini öğretir |
| Nash Dengesi | Rasyonel seçimlerin her zaman en iyi sonucu vermediğini gösterir |
| Sosyal İkilem | Bireysel çıkar ile ortak çıkar arasındaki gerilimi anlatır |

### 1.3 Gerçek Hayat Bağlantısı

Bu oyun gerçek hayatta şu durumlara benzer:

- **Arkadaşlık:** Bir sır paylaştığında arkadaşın onu saklayacak mı yoksa başkalarına söyleyecek mi?
- **Takım Çalışması:** Grup ödevinde herkes eşit çalışacak mı yoksa bazıları kaytaracak mı?
- **Çevre:** Herkes çöpünü çöp kutusuna atacak mı yoksa yere atacak mı?
- **Paylaşım:** Son kalan şekeri kardeşinle paylaşacak mısın?

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. İki oyuncu aynı anda, birbirini görmeden bir seçim yapar
2. Her oyuncu "İşbirliği Yap" veya "İhanet Et" seçeneklerinden birini seçer
3. Seçimler aynı anda açıklanır
4. Seçimlere göre puanlar verilir
5. Tek turda oyun biter ve sonuç gösterilir
```

### 2.2 Oyuncu Bilgisi

| Alan | Değer |
|------|-------|
| Minimum Oyuncu | 2 |
| Maximum Oyuncu | 2 |
| Optimal Oyuncu | 2 |
| Bot Desteği | Evet |
| Eşzamanlı Hamle | Evet |

### 2.3 Tur Yapısı

| Alan | Değer |
|------|-------|
| Tur Sayısı | 1 (tek tur) |
| Varsayılan Tur | 1 |
| Tur Süresi | Süresiz (veya opsiyonel 30 saniye) |

### 2.4 Kazanma Koşulu

```
Tek turda oyun biter. Kazanan, daha yüksek puan alan oyuncudur.
Eşitlik durumunda berabere ilan edilir.

Puan Tablosu:
- İkisi de işbirliği: Her ikisi 3 puan (Win-Win)
- İkisi de ihanet: Her ikisi 1 puan (Lose-Lose)  
- Biri işbirliği, diğeri ihanet: İhanet eden 5 puan, işbirliği yapan 0 puan
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `CHOOSE` | Oyuncu seçimini yapar | Oyuncu henüz seçim yapmamışsa |

### 3.2 Eylem Detayları

#### CHOOSE

```
Action {
  type: 'CHOOSE'
  playerId: string
  payload: {
    choice: 'cooperate' | 'defect'
  }
}
```

**Validation Kuralları:**
- Oyuncu ID'si geçerli olmalı
- Oyuncu daha önce seçim yapmamış olmalı
- choice değeri 'cooperate' veya 'defect' olmalı
- Oyun 'playing' fazında olmalı

**Sonuç:**
- Oyuncunun seçimi state'e kaydedilir
- İki oyuncu da seçim yaptıysa 'reveal' fazına geçilir
- Tek oyuncu seçtiyse diğer oyuncu beklenir

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi

```
                      Oyuncu 2
                 İşbirliği    İhanet
Oyuncu 1  İşbirliği  (3, 3)     (0, 5)
          İhanet     (5, 0)     (1, 1)
```

| Sembol | Değer | Açıklama |
|--------|-------|----------|
| R (Reward) | 3 | Her iki taraf işbirliği yaparsa ödül |
| T (Temptation) | 5 | İhanet edip diğeri işbirliği yaparsa |
| S (Sucker) | 0 | İşbirliği yapıp diğeri ihanet ederse |
| P (Punishment) | 1 | Her iki taraf ihanet ederse ceza |

**Oyun Teorisi Koşulu:** T > R > P > S (5 > 3 > 1 > 0)

### 4.2 Skor Hesaplama

```
Pseudo-kod:

function calculatePayoff(player1Choice, player2Choice):
    matrix = {
        'cooperate-cooperate': { p1: 3, p2: 3 },
        'cooperate-defect':    { p1: 0, p2: 5 },
        'defect-cooperate':    { p1: 5, p2: 0 },
        'defect-defect':       { p1: 1, p2: 1 }
    }
    
    key = player1Choice + '-' + player2Choice
    return matrix[key]
```

### 4.3 Final Skor

```
Tek turlu oyunda:
finalScore = roundScore

Sonuç kategorileri:
- Mutual Cooperation (3-3): "Win-Win" 
- Mutual Defection (1-1): "Lose-Lose"
- Exploitation (5-0 veya 0-5): Bir taraf kazandı
```

### 4.4 Kazanan Belirleme

```
if (player1.score > player2.score):
    winner = player1
    result = 'player1_wins'
else if (player2.score > player1.score):
    winner = player2
    result = 'player2_wins'
else:
    winner = null
    result = 'draw'
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  // Seçimler
  player1Choice: 'cooperate' | 'defect' | null
  player2Choice: 'cooperate' | 'defect' | null
  
  // Ayarlar
  showPayoffMatrix: boolean
  timeLimit: number | null
  
  // Sonuç
  outcome: {
    type: 'mutual_cooperation' | 'mutual_defection' | 'exploitation' | null
    exploiter: string | null
  } | null
}
```

### 5.2 Örnek Initial State

```
{
  gameId: "game-pd-001",
  moduleId: "prisoners-dilemma",
  
  players: [
    { 
      id: "player-1", 
      type: "human", 
      name: "Player 1", 
      score: 0,
      stats: { wins: 0, losses: 0, draws: 0, totalScore: 0 }
    },
    { 
      id: "player-2", 
      type: "bot", 
      name: "Bot", 
      score: 0, 
      strategyId: "random",
      stats: { wins: 0, losses: 0, draws: 0, totalScore: 0 }
    }
  ],
  
  currentPlayerId: null,
  phase: "playing",
  round: 1,
  maxRounds: 1,
  
  data: {
    player1Choice: null,
    player2Choice: null,
    showPayoffMatrix: true,
    timeLimit: null,
    outcome: null
  },
  
  history: [],
  result: null,
  
  createdAt: 1704067200000,
  updatedAt: 1704067200000
}
```

### 5.3 Örnek Final State

```
{
  gameId: "game-pd-001",
  moduleId: "prisoners-dilemma",
  
  players: [
    { id: "player-1", type: "human", name: "Player 1", score: 0 },
    { id: "player-2", type: "bot", name: "Bot", score: 5 }
  ],
  
  currentPlayerId: null,
  phase: "finished",
  round: 1,
  maxRounds: 1,
  
  data: {
    player1Choice: "cooperate",
    player2Choice: "defect",
    showPayoffMatrix: true,
    timeLimit: null,
    outcome: {
      type: "exploitation",
      exploiter: "player-2"
    }
  },
  
  history: [
    {
      round: 1,
      actions: [
        { playerId: "player-1", choice: "cooperate", timestamp: 1704067210000 },
        { playerId: "player-2", choice: "defect", timestamp: 1704067211000 }
      ],
      payoff: { "player-1": 0, "player-2": 5 }
    }
  ],
  
  result: {
    winnerId: "player-2",
    finalScores: { "player-1": 0, "player-2": 5 },
    summary: {
      en: "Bot wins! You cooperated but Bot defected.",
      tr: "Bot kazandı! Sen işbirliği yaptın ama Bot ihanet etti."
    }
  }
}
```

---

## 6. UI TASARIMI

### 6.1 Ekran Listesi

| Ekran | Açıklama | Önem |
|-------|----------|------|
| Setup | Rakip seçimi (bot/zorluk) | Zorunlu |
| Main Game | Seçim ekranı | Zorunlu |
| Waiting | Diğer oyuncu bekleniyor | Zorunlu |
| Reveal | Sonuçların gösterimi | Zorunlu |
| Result | Final skor ve özet | Zorunlu |

### 6.2 Main Game Layout

```
+---------------------------------------------+
|              HEADER                          |
|  Mahkum İkilemi  |  Tur: 1/1                |
+---------------------------------------------+
|                                             |
|           PAYOFF MATRIX                     |
|  +-----------------------------------------+|
|  |          | İşbirliği | İhanet   |       ||
|  |----------+-----------+----------|       ||
|  | İşbirliği|  (3, 3)   |  (0, 5)  |       ||
|  | İhanet   |  (5, 0)   |  (1, 1)  |       ||
|  +-----------------------------------------+|
|                                             |
|           QUESTION                          |
|  "Seçimini yap: İşbirliği mi, İhanet mi?"  |
|                                             |
+---------------------------------------------+
|           CHOICE BUTTONS                    |
|                                             |
|  +-------------------+ +-------------------+|
|  |                   | |                   ||
|  |   İŞBİRLİĞİ      | |    İHANET        ||
|  |                   | |                   ||
|  +-------------------+ +-------------------+|
|                                             |
+---------------------------------------------+
|           OPPONENT STATUS                   |
|  Bot: Düşünüyor...                         |
+---------------------------------------------+
```

### 6.3 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| Game Board | `<pd-game-board>` | Ana oyun alanı wrapper |
| Payoff Matrix | `<pd-payoff-matrix>` | 2x2 matris gösterimi |
| Choice Button | `<pd-choice-button>` | İşbirliği/İhanet butonu |
| Player Card | `<pd-player-card>` | Oyuncu bilgisi ve seçimi |
| Reveal Animation | `<pd-reveal>` | Sonuç açıklama animasyonu |
| Result Summary | `<pd-result-summary>` | Oyun sonu özet |

### 6.4 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| Button hover scale | Hover | 150ms |
| Button click pulse | Click | 200ms |
| Card flip reveal | Reveal phase | 500ms |
| Score count up | Result phase | 800ms |
| Winner celebration | Win | 1000ms |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `always-cooperate` | Her Zaman İşbirliği | Easy | Hep işbirliği yapar |
| `always-defect` | Her Zaman İhanet | Easy | Hep ihanet eder |
| `random` | Rastgele | Easy | %50-%50 rastgele seçer |
| `tit-for-tat` | Kısasa Kısas | Medium | İlk tur işbirliği, sonra taklit |
| `suspicious-tft` | Şüpheci | Hard | İlk tur ihanet, sonra taklit |

### 7.2 Strateji Detayları

#### always-cooperate: Her Zaman İşbirliği

**Algoritma (Pseudo-kod):**
```
function decide(state, playerId):
    return { type: 'CHOOSE', playerId, payload: { choice: 'cooperate' } }
```

**Davranış Özellikleri:**
- Hiç ihanet etmez
- Sömürülebilir
- Çocuklar için güvenli başlangıç

#### always-defect: Her Zaman İhanet

**Algoritma (Pseudo-kod):**
```
function decide(state, playerId):
    return { type: 'CHOOSE', playerId, payload: { choice: 'defect' } }
```

#### random: Rastgele

**Algoritma (Pseudo-kod):**
```
function decide(state, playerId):
    choice = Math.random() > 0.5 ? 'cooperate' : 'defect'
    return { type: 'CHOOSE', playerId, payload: { choice } }
```

### 7.3 Varsayılan Strateji

```
defaultStrategyId: 'random'

Zorluk bazlı öneri:
- Easy: 'always-cooperate' veya 'random'
- Medium: 'tit-for-tat'
- Hard: 'suspicious-tft' veya 'always-defect'
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef | Etkileşim |
|------|--------|-------|-----------|
| 1 | Hoş Geldin | Oyunu tanıt | wait |
| 2 | Hikaye | Mahkum senaryosu | wait |
| 3 | Seçenekler | İşbirliği ve ihanet | highlight |
| 4 | Matris | Puanları açıkla | highlight |
| 5 | Dene | İlk seçimi yaptır | click |
| 6 | Sonuç | Sonucu açıkla | wait |

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
| E1 | Oyuncu çok hızlı iki kez tıklarsa | İlk tıklama geçerli, ikincisi ignore |
| E2 | Sayfa yenilenirse (mid-game) | State localStorage'dan restore |
| E3 | Bot yanıt vermezse | Timeout sonrası random seçim |
| E4 | Her iki oyuncu aynı anda seçerse | İkisi de kabul, reveal'e geç |
| E5 | Süre limiti dolarsa | Random seçim yapılır |

### 9.2 Hata Senaryoları

| # | Hata | Error Code | Mesaj |
|---|------|------------|-------|
| H1 | Geçersiz seçim değeri | `INVALID_CHOICE` | "Geçersiz seçim" |
| H2 | Zaten seçim yapılmış | `ALREADY_CHOSEN` | "Zaten seçiminizi yaptınız" |
| H3 | Oyun bitmiş | `GAME_OVER` | "Oyun bitti" |

---

## 10. TEST SENARYOLARI

### 10.1 Given-When-Then Senaryoları

#### Senaryo 1: Karşılıklı İşbirliği

```
GIVEN: Yeni başlatılmış bir oyun
  AND: Bot stratejisi "always-cooperate"
WHEN: Oyuncu "İşbirliği" butonuna tıklar
  AND: Bot da "İşbirliği" seçer
THEN: Her iki oyuncu 3 puan alır
  AND: Sonuç "Mutual Cooperation" olarak gösterilir
```

#### Senaryo 2: Oyuncu İhanet Eder

```
GIVEN: Yeni başlatılmış bir oyun
  AND: Bot stratejisi "always-cooperate"
WHEN: Oyuncu "İhanet" butonuna tıklar
  AND: Bot "İşbirliği" seçer
THEN: Oyuncu 5 puan alır
  AND: Bot 0 puan alır
  AND: "Kazandın" mesajı görünür
```

#### Senaryo 3: Karşılıklı İhanet

```
GIVEN: Yeni başlatılmış bir oyun
  AND: Bot stratejisi "always-defect"
WHEN: Oyuncu "İhanet" butonuna tıklar
  AND: Bot da "İhanet" seçer
THEN: Her iki oyuncu 1 puan alır
```

---

## 11. METRİKLER

### 11.1 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:prisoners-dilemma:start` | Oyun başladığında | `{ botStrategy, difficulty }` |
| `game:prisoners-dilemma:choice` | Seçim yapıldığında | `{ choice, responseTime }` |
| `game:prisoners-dilemma:complete` | Oyun bittiğinde | `{ playerChoice, botChoice, outcome }` |

### 11.2 Başarı Kriterleri

| KPI | Hedef |
|-----|-------|
| Tamamlama oranı | > 95% |
| Ortalama süre | < 1 dakika |
| Tekrar oynama | > 60% |

---

## 12. ACHIEVEMENTS

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `pd-first-game` | İlk Adım | İlk oyunu tamamla | 1 oyun tamamla |
| `pd-cooperator` | Güvenilir Dost | 5 kez üst üste işbirliği yap | 5 ardışık cooperate |
| `pd-defector` | Soğukkanlı | 5 kez üst üste ihanet et | 5 ardışık defect |
| `pd-win-streak` | Kazanma Serisi | 3 oyun üst üste kazan | 3 ardışık win |

---

## 13. i18n KEYS

### 13.1 İngilizce

```json
"prisoners-dilemma": {
  "name": "Prisoner's Dilemma",
  "description": "The classic game of cooperation vs betrayal",
  "actions": {
    "cooperate": "Cooperate",
    "defect": "Defect"
  },
  "outcomes": {
    "mutual_cooperation": "You both cooperated!",
    "mutual_defection": "You both defected.",
    "you_exploited": "You exploited your opponent!",
    "you_were_exploited": "You were exploited!"
  },
  "results": {
    "won": "You Won!",
    "lost": "You Lost",
    "draw": "It's a Draw"
  }
}
```

### 13.2 Türkçe

```json
"prisoners-dilemma": {
  "name": "Mahkum İkilemi",
  "description": "İşbirliği ve ihanet arasındaki klasik oyun",
  "actions": {
    "cooperate": "İşbirliği",
    "defect": "İhanet"
  },
  "outcomes": {
    "mutual_cooperation": "İkiniz de işbirliği yaptınız!",
    "mutual_defection": "İkiniz de ihanet ettiniz.",
    "you_exploited": "Rakibini sömürdün!",
    "you_were_exploited": "Sömürüldün!"
  },
  "results": {
    "won": "Kazandın!",
    "lost": "Kaybettin",
    "draw": "Berabere"
  }
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Core Bağımlılıkları

| Modül | Kullanım |
|-------|----------|
| `/core/store.js` | State yönetimi |
| `/core/events.js` | Event emit |
| `/core/utils/i18n.js` | Çeviri |
| `/core/components/base-component.js` | Web Component base |

### 14.2 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/prisoners-dilemma.svg` |
| Thumbnail | `/assets/images/games/pd-thumb.svg` |

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| Tek turlu varsayılan | Basitlik, hızlı öğrenme |
| Eşzamanlı seçim | Orijinal oyun formatına sadık |
| Matrix her zaman görünür | Eğitici, şeffaf |

### 15.2 Gelecek İyileştirmeler

- [ ] Payoff değerlerini özelleştirme
- [ ] Multiplayer modu
- [ ] Turnuva modu

### 15.3 Referanslar

- Axelrod, R. (1984). The Evolution of Cooperation
- Nicky Case - The Evolution of Trust

---

## 16. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |
