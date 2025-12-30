# ITERATED PRISONER'S DILEMMA
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `iterated-pd` |
| Versiyon | 1.0.0 |
| Kategori | classic |
| Zorluk | medium |
| Tier | premium |
| Durum | approved |

---

## 1. ÖZET

### 1.1 Kısa Açıklama

| Dil | Açıklama |
|-----|----------|
| EN | Play multiple rounds against the same opponent - will cooperation emerge? |
| TR | Aynı rakibe karşı birçok tur oyna - işbirliği ortaya çıkacak mı? |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| Tekrarlı Etkileşim | Uzun vadeli ilişkilerin dinamiğini gösterir |
| İtibar | Geçmiş davranışların gelecekteki kararları etkilediğini öğretir |
| Strateji Evrimi | Farklı stratejilerin zaman içinde performansını karşılaştırır |
| Af ve Öç | Affetmenin ve misillemenin etkilerini gösterir |
| Shadow of the Future | Gelecek beklentisinin bugünkü kararları nasıl etkilediğini anlatır |

### 1.3 Gerçek Hayat Bağlantısı

Bu oyun gerçek hayatta şu durumlara benzer:

- **Arkadaşlık:** Aynı arkadaşla yıllar boyu ilişki - güven nasıl inşa edilir?
- **İş İlişkileri:** Aynı tedarikçiyle tekrarlı anlaşmalar
- **Komşuluk:** Her gün gördüğün komşularla ilişki
- **Uluslararası İlişkiler:** Ülkeler arası tekrarlı müzakereler
- **Online Oyunlar:** Aynı oyuncularla tekrar tekrar eşleşme

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. Tek turlu Prisoner's Dilemma'nın tekrarlı versiyonu
2. Oyuncular önceden belirlenen sayıda tur boyunca aynı rakiple oynar
3. Her turda standart PD kuralları geçerli
4. Oyuncular rakibin geçmiş hamlelerini görebilir
5. Toplam skor tüm turların toplamıdır
6. En yüksek toplam skoru alan kazanır
```

### 2.2 Oyuncu Bilgisi

| Alan | Değer |
|------|-------|
| Minimum Oyuncu | 2 |
| Maximum Oyuncu | 2 |
| Optimal Oyuncu | 2 |
| Bot Desteği | Evet |
| Eşzamanlı Hamle | Evet (her turda) |

### 2.3 Tur Yapısı

| Alan | Değer |
|------|-------|
| Varsayılan Tur | 10 |
| Minimum Tur | 5 |
| Maximum Tur | 50 |
| Tur Süresi | Süresiz (veya opsiyonel 15 saniye) |

### 2.4 Kazanma Koşulu

```
Tüm turlar bittiğinde:
- En yüksek toplam skora sahip oyuncu kazanır
- Eşitlik durumunda berabere

Opsiyonel Bitiş Koşulları:
- Sabit tur sayısı (varsayılan)
- Rastgele bitiş (%5 şansla her turdan sonra)
- Hedef skora ulaşma
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `CHOOSE` | Oyuncu bu tur için seçimini yapar | Oyuncu bu turda henüz seçim yapmamışsa |

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
- Oyuncu bu turda henüz seçim yapmamış olmalı
- Oyun 'playing' fazında olmalı
- Mevcut tur maxRounds'u geçmemiş olmalı

**Sonuç:**
- Oyuncunun seçimi mevcut tur verisine kaydedilir
- İki oyuncu da seçim yaptıysa tur sonuçlanır
- Tur sonuçlandığında yeni tur başlar veya oyun biter

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi (Her Tur)

```
                      Oyuncu 2
                 İşbirliği    İhanet
Oyuncu 1  İşbirliği  (3, 3)     (0, 5)
          İhanet     (5, 0)     (1, 1)
```

Standart PD değerleri kullanılır (T=5, R=3, P=1, S=0).

### 4.2 Skor Hesaplama

```
Pseudo-kod:

function calculateRoundPayoff(player1Choice, player2Choice):
    // Tek turlu PD ile aynı
    matrix = {
        'cooperate-cooperate': { p1: 3, p2: 3 },
        'cooperate-defect':    { p1: 0, p2: 5 },
        'defect-cooperate':    { p1: 5, p2: 0 },
        'defect-defect':       { p1: 1, p2: 1 }
    }
    return matrix[player1Choice + '-' + player2Choice]

function calculateTotalScore(history):
    total = { p1: 0, p2: 0 }
    for each round in history:
        total.p1 += round.payoff.p1
        total.p2 += round.payoff.p2
    return total
```

### 4.3 Final Skor

```
10 turlu oyunda maksimum/minimum skorlar:

Maksimum (hep sömürme): 50 puan (5 x 10)
Karşılıklı işbirliği: 30 puan (3 x 10)
Karşılıklı ihanet: 10 puan (1 x 10)
Minimum (hep sömürülme): 0 puan (0 x 10)

Ortalama beklenti (karşılıklı işbirliği): 30 puan
```

### 4.4 Strateji Performans Metrikleri

```
İstatistikler:
- Cooperation Rate: cooperate_count / total_rounds
- Average Round Score: total_score / total_rounds
- Exploitation Count: Kaç kez 5-0 veya 0-5 oldu
- Mutual Cooperation Count: Kaç kez 3-3 oldu
- Mutual Defection Count: Kaç kez 1-1 oldu
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  // Ayarlar
  totalRounds: number
  randomEndProbability: number    // 0-1, varsayılan 0
  showOpponentHistory: boolean    // Rakibin geçmişi görünür mü
  
  // Mevcut tur
  currentRound: {
    player1Choice: 'cooperate' | 'defect' | null
    player2Choice: 'cooperate' | 'defect' | null
  }
  
  // İstatistikler
  stats: {
    player1: PlayerStats
    player2: PlayerStats
  }
}

PlayerStats {
  cooperateCount: number
  defectCount: number
  timesExploited: number      // 0 puan aldığı tur sayısı
  timesExploiting: number     // 5 puan aldığı tur sayısı
  mutualCoopCount: number
  mutualDefectCount: number
}
```

### 5.2 Örnek Initial State

```
{
  gameId: "game-ipd-001",
  moduleId: "iterated-pd",
  
  players: [
    { id: "player-1", type: "human", name: "Player 1", score: 0 },
    { id: "player-2", type: "bot", name: "Tit-for-Tat Bot", score: 0, strategyId: "tit-for-tat" }
  ],
  
  currentPlayerId: null,
  phase: "playing",
  round: 1,
  maxRounds: 10,
  
  data: {
    totalRounds: 10,
    randomEndProbability: 0,
    showOpponentHistory: true,
    currentRound: {
      player1Choice: null,
      player2Choice: null
    },
    stats: {
      player1: { cooperateCount: 0, defectCount: 0, timesExploited: 0, timesExploiting: 0, mutualCoopCount: 0, mutualDefectCount: 0 },
      player2: { cooperateCount: 0, defectCount: 0, timesExploited: 0, timesExploiting: 0, mutualCoopCount: 0, mutualDefectCount: 0 }
    }
  },
  
  history: [],
  result: null
}
```

### 5.3 Örnek Mid-Game State (Tur 5)

```
{
  gameId: "game-ipd-001",
  moduleId: "iterated-pd",
  
  players: [
    { id: "player-1", type: "human", name: "Player 1", score: 14 },
    { id: "player-2", type: "bot", name: "Tit-for-Tat Bot", score: 14 }
  ],
  
  phase: "playing",
  round: 5,
  maxRounds: 10,
  
  data: {
    totalRounds: 10,
    currentRound: {
      player1Choice: null,
      player2Choice: null
    },
    stats: {
      player1: { cooperateCount: 3, defectCount: 1, timesExploited: 0, timesExploiting: 1, mutualCoopCount: 3, mutualDefectCount: 0 },
      player2: { cooperateCount: 4, defectCount: 0, timesExploited: 1, timesExploiting: 0, mutualCoopCount: 3, mutualDefectCount: 0 }
    }
  },
  
  history: [
    { round: 1, p1: "cooperate", p2: "cooperate", payoff: { p1: 3, p2: 3 } },
    { round: 2, p1: "cooperate", p2: "cooperate", payoff: { p1: 3, p2: 3 } },
    { round: 3, p1: "defect", p2: "cooperate", payoff: { p1: 5, p2: 0 } },
    { round: 4, p1: "cooperate", p2: "defect", payoff: { p1: 0, p2: 5 } },
    // Tit-for-Tat öç aldı
  ],
  
  result: null
}
```

### 5.4 Örnek Final State

```
{
  gameId: "game-ipd-001",
  moduleId: "iterated-pd",
  
  players: [
    { id: "player-1", type: "human", name: "Player 1", score: 26 },
    { id: "player-2", type: "bot", name: "Tit-for-Tat Bot", score: 26 }
  ],
  
  phase: "finished",
  round: 10,
  maxRounds: 10,
  
  history: [
    // 10 tur verisi
  ],
  
  result: {
    winnerId: null,  // Berabere
    finalScores: { "player-1": 26, "player-2": 26 },
    summary: {
      en: "It's a draw! You both scored 26 points over 10 rounds.",
      tr: "Berabere! 10 turda ikiniz de 26 puan aldınız."
    },
    analysis: {
      cooperationRate: { p1: 0.7, p2: 0.7 },
      mutualCoopRate: 0.6,
      mutualDefectRate: 0.1
    }
  }
}
```

---

## 6. UI TASARIMI

### 6.1 Ekran Listesi

| Ekran | Açıklama | Önem |
|-------|----------|------|
| Setup | Tur sayısı, bot stratejisi seçimi | Zorunlu |
| Main Game | Seçim + geçmiş görünümü | Zorunlu |
| Round Result | Tur sonucu (kısa) | Zorunlu |
| Final Result | Oyun sonu analiz | Zorunlu |
| Strategy Analysis | Strateji performans grafiği | Opsiyonel |

### 6.2 Main Game Layout

```
+-----------------------------------------------+
|                HEADER                          |
|  Iterated PD  |  Tur: 5/10  |  [Skor: 14-14] |
+-----------------------------------------------+
|                                               |
|  HISTORY PANEL (Son 5 tur)                    |
|  +-------------------------------------------+|
|  | Tur | Sen      | Rakip    | Skor         ||
|  |-----|----------|----------|--------------|
|  |  1  | İşbirliği| İşbirliği| 3-3          ||
|  |  2  | İşbirliği| İşbirliği| 3-3          ||
|  |  3  | İhanet   | İşbirliği| 5-0          ||
|  |  4  | İşbirliği| İhanet   | 0-5          ||
|  +-------------------------------------------+|
|                                               |
|  CURRENT ROUND                                |
|  "Tur 5 - Seçimini yap"                      |
|                                               |
+-----------------------------------------------+
|  CHOICE BUTTONS                               |
|  +------------------+ +------------------+    |
|  |   İŞBİRLİĞİ     | |     İHANET      |    |
|  +------------------+ +------------------+    |
+-----------------------------------------------+
|  STATS BAR                                    |
|  Sen: 14 puan | Rakip: 14 puan | Kalan: 6 tur|
+-----------------------------------------------+
```

### 6.3 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| Game Container | `<ipd-game>` | Ana wrapper |
| History Table | `<ipd-history>` | Geçmiş turlar tablosu |
| History Row | `<ipd-history-row>` | Tek tur satırı |
| Score Tracker | `<ipd-score-tracker>` | Toplam skor gösterimi |
| Round Indicator | `<ipd-round-indicator>` | Mevcut tur / toplam tur |
| Choice Button | `<ipd-choice-button>` | İşbirliği/İhanet butonu |
| Round Result | `<ipd-round-result>` | Tur sonucu popup |
| Final Analysis | `<ipd-final-analysis>` | Son analiz ekranı |
| Cooperation Chart | `<ipd-coop-chart>` | İşbirliği oranı grafiği |

### 6.4 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| History row slide in | Yeni tur eklenince | 300ms |
| Score update pulse | Skor değişince | 400ms |
| Round transition | Tur bitince | 500ms |
| Chart animate | Final ekranında | 1000ms |
| Streak indicator | 3+ aynı seçim | 600ms |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `always-cooperate` | Her Zaman İşbirliği | Easy | Hep işbirliği |
| `always-defect` | Her Zaman İhanet | Easy | Hep ihanet |
| `random` | Rastgele | Easy | %50-%50 |
| `tit-for-tat` | Kısasa Kısas | Medium | Klasik TFT |
| `generous-tft` | Cömert TFT | Medium | Bazen affeder |
| `suspicious-tft` | Şüpheci TFT | Medium | İlk ihanet |
| `grudger` | Kinci | Hard | Bir ihanet = sonsuza kadar |
| `pavlov` | Pavlov | Hard | Win-stay, lose-shift |
| `random-tft` | Gürültülü TFT | Hard | %10 hata yapan TFT |

### 7.2 Strateji Detayları

#### tit-for-tat: Kısasa Kısas (Axelrod Turnuvası Kazananı)

**Algoritma:**
```
function decide(state, playerId):
    history = state.history
    
    // İlk turda işbirliği
    if (history.length === 0):
        return cooperate
    
    // Rakibin son hamlesini taklit et
    lastRound = history[history.length - 1]
    opponentLastChoice = getOpponentChoice(lastRound, playerId)
    return opponentLastChoice
```

**Özellikler:**
- Nice (ilk işbirliği)
- Retaliatory (ihanete karşılık verir)
- Forgiving (hemen affeder)
- Clear (anlaşılması kolay)

#### generous-tft: Cömert Kısasa Kısas

**Algoritma:**
```
function decide(state, playerId):
    history = state.history
    
    if (history.length === 0):
        return cooperate
    
    opponentLastChoice = getOpponentChoice(lastRound, playerId)
    
    // Rakip ihanet ettiyse %10 ihtimalle yine de işbirliği yap
    if (opponentLastChoice === 'defect'):
        if (Math.random() < 0.1):
            return cooperate
        return defect
    
    return cooperate
```

**Özellikler:**
- Gürültülü ortamlarda TFT'den daha iyi
- İhanet sarmalını kırabilir

#### grudger: Kinci (Grim Trigger)

**Algoritma:**
```
function decide(state, playerId):
    history = state.history
    
    // Rakip hiç ihanet etti mi?
    for round in history:
        if (getOpponentChoice(round, playerId) === 'defect'):
            // Bir kez ihanet = sonsuza kadar ihanet
            return defect
    
    return cooperate
```

**Özellikler:**
- Asla affetmez
- Caydırıcı ama kırılgan

#### pavlov: Win-Stay, Lose-Shift

**Algoritma:**
```
function decide(state, playerId):
    history = state.history
    
    if (history.length === 0):
        return cooperate
    
    lastRound = history[history.length - 1]
    myLastChoice = getMyChoice(lastRound, playerId)
    myLastPayoff = getMyPayoff(lastRound, playerId)
    
    // İyi sonuç aldıysan (3 veya 5) aynı şeyi yap
    if (myLastPayoff >= 3):
        return myLastChoice
    
    // Kötü sonuç aldıysan (0 veya 1) değiştir
    return opposite(myLastChoice)
```

**Özellikler:**
- Karşılıklı işbirliğinde kalır
- Sömürülünce ihanet'e geçer
- Karşılıklı ihanetten çıkabilir

### 7.3 Varsayılan Strateji

```
defaultStrategyId: 'tit-for-tat'

Zorluk bazlı öneri:
- Easy: 'always-cooperate', 'random'
- Medium: 'tit-for-tat', 'generous-tft'
- Hard: 'grudger', 'pavlov', 'suspicious-tft'
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef |
|------|--------|-------|
| 1 | Giriş | Tek turlu PD'den farkı |
| 2 | Geçmiş | History panelini tanıt |
| 3 | Strateji | Uzun vadeli düşünme |
| 4 | Tit-for-Tat | En ünlü stratejiyi anlat |
| 5 | Deneme | 3 tur oyna |
| 6 | Analiz | Sonuçları yorumla |

### 8.2 Tutorial Ayarları

```
{
  isSkippable: true,
  showOnFirstPlay: true,
  estimatedTime: '4 min',
  practiceRounds: 3
}
```

---

## 9. EDGE CASES

### 9.1 Olası Durumlar

| # | Durum | Beklenen Davranış |
|---|-------|-------------------|
| E1 | Oyuncu son turda sürekli ihanet | Geçerli, strateji kararı |
| E2 | Tüm turlar karşılıklı ihanet | Oyun normal biter, analiz gösterilir |
| E3 | Oyun ortasında çıkış | State kaydedilir, devam edilebilir |
| E4 | Random end tetiklenirse | Oyun erken biter, skor hesaplanır |
| E5 | 50 turlu uzun oyun | Performans optimize, lazy load history |

### 9.2 Hata Senaryoları

| # | Hata | Error Code | Mesaj |
|---|------|------------|-------|
| H1 | Tur limiti aşıldı | `ROUND_EXCEEDED` | "Tüm turlar tamamlandı" |
| H2 | Geçersiz tur numarası | `INVALID_ROUND` | "Geçersiz tur" |

---

## 10. TEST SENARYOLARI

### 10.1 Given-When-Then Senaryoları

#### Senaryo 1: Tit-for-Tat vs Cooperator

```
GIVEN: 5 turlu oyun
  AND: Bot stratejisi "always-cooperate"
  AND: Oyuncu Tit-for-Tat oynuyor
WHEN: Tüm turlar oynanır
THEN: Her turda 3-3
  AND: Final skor 15-15
  AND: Berabere
```

#### Senaryo 2: Tit-for-Tat vs Defector

```
GIVEN: 5 turlu oyun
  AND: Bot stratejisi "always-defect"
  AND: Oyuncu TFT oynuyor
WHEN: Tüm turlar oynanır
THEN: Tur 1: 0-5 (TFT işbirliği, bot ihanet)
  AND: Tur 2-5: 1-1 (ikisi de ihanet)
  AND: Final skor: 4-9
  AND: Bot kazanır
```

#### Senaryo 3: İhanet Sonrası Af

```
GIVEN: 10 turlu oyun
  AND: Bot stratejisi "tit-for-tat"
WHEN: Oyuncu tur 5'te ihanet eder
  AND: Sonra tekrar işbirliğine döner
THEN: Bot tur 6'da ihanet eder (misilleme)
  AND: Bot tur 7'de işbirliğine döner (af)
  AND: İşbirliği devam eder
```

---

## 11. METRİKLER

### 11.1 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:iterated-pd:start` | Oyun başı | `{ totalRounds, botStrategy }` |
| `game:iterated-pd:round` | Her tur sonu | `{ round, choices, payoffs }` |
| `game:iterated-pd:complete` | Oyun sonu | `{ finalScores, cooperationRates, winner }` |
| `game:iterated-pd:strategy-change` | Oyuncu stratejisini değiştirirse | `{ fromPattern, toPattern, atRound }` |

### 11.2 Başarı Kriterleri

| KPI | Hedef |
|-----|-------|
| Tamamlama oranı | > 80% |
| Ortalama oyun süresi | 5-10 dakika |
| Tekrar oynama | > 70% |

---

## 12. ACHIEVEMENTS

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `ipd-marathon` | Maratoncu | 50 turlu oyun tamamla | 50 tur bitir |
| `ipd-cooperator` | Barış Elçisi | %90+ işbirliği oranıyla kazan | win + coop_rate > 0.9 |
| `ipd-comeback` | Geri Dönüş | 5+ puan geriden gel ve kazan | come from behind |
| `ipd-mutual-coop` | Mükemmel Uyum | 10 tur üst üste karşılıklı işbirliği | 10 consecutive (3,3) |
| `ipd-beat-tft` | Usta | Tit-for-Tat'ı yen | beat TFT bot |
| `ipd-analyst` | Analist | Final analiz ekranını incele | view analysis |

---

## 13. i18n KEYS

### 13.1 İngilizce

```json
"iterated-pd": {
  "name": "Iterated Prisoner's Dilemma",
  "description": "Multiple rounds against the same opponent",
  "setup": {
    "rounds": "Number of Rounds",
    "strategy": "Opponent Strategy"
  },
  "game": {
    "round": "Round {{current}} of {{total}}",
    "history": "History",
    "yourChoice": "Your Choice",
    "opponentChoice": "Opponent",
    "score": "Score"
  },
  "analysis": {
    "title": "Game Analysis",
    "cooperationRate": "Cooperation Rate",
    "mutualCoop": "Mutual Cooperation",
    "mutualDefect": "Mutual Defection",
    "exploitation": "Exploitation"
  }
}
```

### 13.2 Türkçe

```json
"iterated-pd": {
  "name": "Tekrarlı Mahkum İkilemi",
  "description": "Aynı rakibe karşı birçok tur",
  "setup": {
    "rounds": "Tur Sayısı",
    "strategy": "Rakip Stratejisi"
  },
  "game": {
    "round": "Tur {{current}} / {{total}}",
    "history": "Geçmiş",
    "yourChoice": "Senin Seçimin",
    "opponentChoice": "Rakip",
    "score": "Skor"
  },
  "analysis": {
    "title": "Oyun Analizi",
    "cooperationRate": "İşbirliği Oranı",
    "mutualCoop": "Karşılıklı İşbirliği",
    "mutualDefect": "Karşılıklı İhanet",
    "exploitation": "Sömürü"
  }
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/iterated-pd.svg` |
| Thumbnail | `/assets/images/games/ipd-thumb.svg` |

### 14.2 Tek Turlu PD ile İlişki

- Payoff hesaplama mantığı paylaşılabilir
- UI component'ları extend edilebilir
- Stratejiler her iki oyunda da kullanılır

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| Premium tier | Daha karmaşık, tek turlu versiyonu free |
| History görünür | Strateji geliştirmeye yardımcı |
| 10 tur varsayılan | Denge: çok kısa değil, sıkıcı değil |
| Final analiz | Eğitici değer katıyor |

### 15.2 Gelecek İyileştirmeler

- [ ] Turnuva modu (birden fazla strateji)
- [ ] Strateji builder (kendi stratejini yaz)
- [ ] Evolutionary simulation
- [ ] Noise/hata parametresi

### 15.3 Referanslar

- Axelrod, R. (1984). The Evolution of Cooperation
- Axelrod's Tournament Results
- Nowak & Sigmund - Tit for Tat in heterogeneous populations

---

## 16. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |
