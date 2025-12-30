# CHEORY: Oyun Spesifikasyon Şablonu
## GAME_SPEC_TEMPLATE.md

> Bu şablon, her yeni oyun için kopyalanıp doldurulacaktır.
> Dosya adı formatı: `{sıra}_{OYUN_ADI}.md` (örn: `01_PRISONERS_DILEMMA.md`)

---

# {OYUN_ADI}
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `{kebab-case-id}` |
| Versiyon | 1.0.0 |
| Kategori | {classic/bargaining/coordination/social/zero-sum/voting} |
| Zorluk | {easy/medium/hard} |
| Tier | {free/premium} |
| Durum | {draft/review/approved/implemented} |

---

## 1. ÖZET

### 1.1 Kısa Açıklama

| Dil | Açıklama |
|-----|----------|
| EN | {One sentence description in English} |
| TR | {Türkçe tek cümlelik açıklama} |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| {Konsept 1} | {Bu oyun bu konsepti nasıl öğretiyor} |
| {Konsept 2} | {Açıklama} |
| {Konsept 3} | {Açıklama} |

### 1.3 Gerçek Hayat Bağlantısı

{Bu oyunun gerçek hayattaki karşılıkları. Çocukların anlayabileceği örnekler.}

Örnekler:
- {Örnek 1}
- {Örnek 2}
- {Örnek 3}

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. {Kural 1}
2. {Kural 2}
3. {Kural 3}
4. {Kural 4}
5. {Kural 5}
```

### 2.2 Oyuncu Bilgisi

| Alan | Değer |
|------|-------|
| Minimum Oyuncu | {sayı} |
| Maximum Oyuncu | {sayı} |
| Optimal Oyuncu | {sayı} |
| Bot Desteği | {Evet/Hayır} |
| Eşzamanlı Hamle | {Evet/Hayır} |

### 2.3 Tur Yapısı

| Alan | Değer |
|------|-------|
| Tur Sayısı | {Sabit sayı / Değişken / Sınırsız} |
| Varsayılan Tur | {sayı} |
| Tur Süresi | {saniye veya "Süresiz"} |

### 2.4 Kazanma Koşulu

```
{Oyunun nasıl bittiği ve kazananın nasıl belirlendiği}
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `{ACTION_TYPE_1}` | {Ne yapar} | {Ne zaman yapılabilir} |
| `{ACTION_TYPE_2}` | {Ne yapar} | {Ne zaman yapılabilir} |
| `{ACTION_TYPE_3}` | {Ne yapar} | {Ne zaman yapılabilir} |

### 3.2 Eylem Detayları

#### {ACTION_TYPE_1}

```
Action {
  type: '{ACTION_TYPE_1}'
  playerId: string
  payload: {
    {field1}: {type}
    {field2}: {type}
  }
}
```

**Validation Kuralları:**
- {Kural 1}
- {Kural 2}

**Sonuç:**
- {State'de ne değişir}

#### {ACTION_TYPE_2}

```
Action {
  type: '{ACTION_TYPE_2}'
  playerId: string
  payload: {
    {field1}: {type}
  }
}
```

**Validation Kuralları:**
- {Kural 1}

**Sonuç:**
- {State'de ne değişir}

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi

{Eğer 2x2 matris oyunu ise:}

```
                    Oyuncu 2
                 {Seçenek A}  {Seçenek B}
Oyuncu 1  {A}      ({a}, {a})    ({b}, {c})
          {B}      ({c}, {b})    ({d}, {d})
```

{Değerler:}
| Sembol | Değer | Açıklama |
|--------|-------|----------|
| {a} | {sayı} | {açıklama} |
| {b} | {sayı} | {açıklama} |
| {c} | {sayı} | {açıklama} |
| {d} | {sayı} | {açıklama} |

### 4.2 Skor Hesaplama

```
Pseudo-kod:

function calculatePayoff(player1Choice, player2Choice):
    {hesaplama mantığı}
    return { player1Score, player2Score }
```

### 4.3 Final Skor

```
{Toplam skorun nasıl hesaplandığı}

finalScore = {formül}
```

### 4.4 Kazanan Belirleme

```
{Kazananın nasıl belirlendiği}

if (player1.score > player2.score):
    winner = player1
else if (player2.score > player1.score):
    winner = player2
else:
    winner = 'draw'
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  // Oyuna özel alanlar
  {field1}: {type}           // {açıklama}
  {field2}: {type}           // {açıklama}
  {field3}: {type}           // {açıklama}
  
  // Tur verileri
  currentRound: {
    {roundField1}: {type}
    {roundField2}: {type}
  }
  
  // Geçmiş
  rounds: Round[]
}

Round {
  number: number
  {player1Action}: {type}
  {player2Action}: {type}
  {result}: {type}
}
```

### 5.2 Örnek Initial State

```
{
  gameId: "game-123",
  moduleId: "{kebab-case-id}",
  
  players: [
    { id: "p1", type: "human", name: "Player 1", score: 0 },
    { id: "p2", type: "bot", name: "Bot", score: 0, strategyId: "default" }
  ],
  
  currentPlayerId: "p1",
  phase: "playing",
  round: 1,
  maxRounds: {sayı},
  
  data: {
    {field1}: {initialValue},
    {field2}: {initialValue},
    currentRound: {
      {roundField1}: null,
      {roundField2}: null
    },
    rounds: []
  },
  
  history: [],
  result: null
}
```

### 5.3 Örnek Mid-Game State

```
{
  gameId: "game-123",
  moduleId: "{kebab-case-id}",
  
  players: [
    { id: "p1", type: "human", name: "Player 1", score: {örnek} },
    { id: "p2", type: "bot", name: "Bot", score: {örnek} }
  ],
  
  currentPlayerId: "p2",
  phase: "playing",
  round: 3,
  maxRounds: 5,
  
  data: {
    {field1}: {midGameValue},
    currentRound: {
      {roundField1}: {value},
      {roundField2}: null
    },
    rounds: [
      { number: 1, ... },
      { number: 2, ... }
    ]
  },
  
  history: [ ... ],
  result: null
}
```

### 5.4 Örnek Final State

```
{
  ...
  phase: "finished",
  round: 5,
  
  result: {
    winnerId: "p1",
    finalScores: { "p1": {skor}, "p2": {skor} },
    summary: { 
      en: "{English summary}", 
      tr: "{Türkçe özet}" 
    }
  }
}
```

---

## 6. UI TASARIMI

### 6.1 Ekran Listesi

| Ekran | Açıklama | Önem |
|-------|----------|------|
| Setup | {Oyun başlamadan önce ayarlar} | Zorunlu |
| Main Game | {Ana oyun ekranı} | Zorunlu |
| Round Result | {Tur sonucu} | Opsiyonel |
| Final Result | {Oyun sonu} | Zorunlu |

### 6.2 Main Game Layout

```
┌─────────────────────────────────────────┐
│            HEADER                        │
│  {Oyun adı} | Tur: {x}/{y} | {Timer}    │
├─────────────────────────────────────────┤
│                                         │
│            GAME BOARD                   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │                                 │   │
│  │    {Ana oyun alanı taslağı}    │   │
│  │                                 │   │
│  └─────────────────────────────────┘   │
│                                         │
├─────────────────────────────────────────┤
│            SCORE PANEL                  │
│  Player 1: {score}  |  Player 2: {score}│
├─────────────────────────────────────────┤
│            ACTIONS                      │
│  [{Buton 1}]  [{Buton 2}]  [{Buton 3}] │
└─────────────────────────────────────────┘
```

### 6.3 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| {Component1} | `<{prefix}-{name}>` | {Ne yapar} |
| {Component2} | `<{prefix}-{name}>` | {Ne yapar} |
| {Component3} | `<{prefix}-{name}>` | {Ne yapar} |

### 6.4 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| {Anim1} | {Ne zaman} | {ms} |
| {Anim2} | {Ne zaman} | {ms} |

### 6.5 Sesler (Opsiyonel)

| Ses | Tetikleyici |
|-----|-------------|
| {Ses1} | {Ne zaman} |
| {Ses2} | {Ne zaman} |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `{strategy-1}` | {İsim} | Easy | {Kısa açıklama} |
| `{strategy-2}` | {İsim} | Medium | {Kısa açıklama} |
| `{strategy-3}` | {İsim} | Hard | {Kısa açıklama} |

### 7.2 Strateji Detayları

#### {strategy-1}: {İsim}

**Algoritma (Pseudo-kod):**
```
function decide(state, playerId):
    {karar mantığı}
    return action
```

**Davranış Özellikleri:**
- {Özellik 1}
- {Özellik 2}

**Ne Zaman Kullanılır:**
- {Senaryo}

#### {strategy-2}: {İsim}

**Algoritma (Pseudo-kod):**
```
function decide(state, playerId):
    {karar mantığı}
    return action
```

#### {strategy-3}: {İsim}

**Algoritma (Pseudo-kod):**
```
function decide(state, playerId):
    {karar mantığı}
    return action
```

### 7.3 Varsayılan Strateji

```
defaultStrategyId: '{strategy-1}'

Zorluk bazlı öneri:
- Easy: {strategy-1}
- Medium: {strategy-2}
- Hard: {strategy-3}
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef | Etkileşim |
|------|--------|-------|-----------|
| 1 | {Başlık} | {Neyi öğretiyor} | {click/wait/input} |
| 2 | {Başlık} | {Neyi öğretiyor} | {click/wait/input} |
| 3 | {Başlık} | {Neyi öğretiyor} | {click/wait/input} |
| 4 | {Başlık} | {Neyi öğretiyor} | {click/wait/input} |
| 5 | {Başlık} | {Neyi öğretiyor} | {click/wait/input} |

### 8.2 Tutorial Detayları

#### Adım 1: {Başlık}

```
{
  id: 'step-1',
  title: { en: '{English}', tr: '{Türkçe}' },
  content: { 
    en: '{Detailed explanation}', 
    tr: '{Detaylı açıklama}' 
  },
  target: '{CSS selector}',
  position: 'bottom',
  action: { type: 'highlight', target: '{selector}' },
  waitFor: { type: 'click', target: '{selector}' }
}
```

#### Adım 2: {Başlık}

```
{
  id: 'step-2',
  ...
}
```

{Diğer adımlar için aynı format}

### 8.3 Tutorial Ayarları

```
{
  isSkippable: true,
  showOnFirstPlay: true,
  estimatedTime: '{dakika} min'
}
```

---

## 9. EDGE CASES

### 9.1 Olası Durumlar

| # | Durum | Beklenen Davranış |
|---|-------|-------------------|
| E1 | {Edge case 1} | {Ne yapılmalı} |
| E2 | {Edge case 2} | {Ne yapılmalı} |
| E3 | {Edge case 3} | {Ne yapılmalı} |
| E4 | {Edge case 4} | {Ne yapılmalı} |
| E5 | {Edge case 5} | {Ne yapılmalı} |

### 9.2 Hata Senaryoları

| # | Hata | Error Code | Mesaj |
|---|------|------------|-------|
| H1 | {Hata durumu} | `{ERROR_CODE}` | {Kullanıcıya gösterilecek mesaj} |
| H2 | {Hata durumu} | `{ERROR_CODE}` | {Mesaj} |
| H3 | {Hata durumu} | `{ERROR_CODE}` | {Mesaj} |

---

## 10. TEST SENARYOLARI

### 10.1 Unit Tests

#### Engine Tests

```
describe('{GameName} Engine', () => {

  describe('createInitialState', () => {
    it('should {test case 1}')
    it('should {test case 2}')
  })
  
  describe('getAvailableActions', () => {
    it('should {test case 1}')
    it('should {test case 2}')
  })
  
  describe('executeAction', () => {
    it('should {test case 1}')
    it('should {test case 2}')
    it('should {test case 3}')
  })
  
  describe('calculatePayoff', () => {
    it('should return {x} when {condition 1}')
    it('should return {y} when {condition 2}')
    it('should return {z} when {condition 3}')
  })
  
  describe('isGameOver', () => {
    it('should return false when {condition}')
    it('should return true when {condition}')
  })

})
```

### 10.2 Integration Tests

```
describe('{GameName} Game Flow', () => {

  it('should complete a full game')
  it('should handle player vs bot')
  it('should persist state correctly')
  it('should calculate final scores')

})
```

### 10.3 Given-When-Then Senaryoları

#### Senaryo 1: {Temel oyun akışı}

```
GIVEN: {Başlangıç durumu}
WHEN: {Kullanıcı aksiyonu}
THEN: {Beklenen sonuç}
```

#### Senaryo 2: {Kazanma senaryosu}

```
GIVEN: {Başlangıç durumu}
WHEN: {Kullanıcı aksiyonu}
THEN: {Beklenen sonuç}
```

#### Senaryo 3: {Edge case senaryosu}

```
GIVEN: {Başlangıç durumu}
WHEN: {Kullanıcı aksiyonu}
THEN: {Beklenen sonuç}
```

---

## 11. METRİKLER

### 11.1 Oyun Metrikleri

| Metrik | Açıklama | Hesaplama |
|--------|----------|-----------|
| {Metrik 1} | {Ne ölçer} | {Nasıl hesaplanır} |
| {Metrik 2} | {Ne ölçer} | {Nasıl hesaplanır} |
| {Metrik 3} | {Ne ölçer} | {Nasıl hesaplanır} |

### 11.2 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:{id}:start` | Oyun başladığında | `{ difficulty, botStrategy }` |
| `game:{id}:action` | Her hamlede | `{ actionType, round }` |
| `game:{id}:complete` | Oyun bittiğinde | `{ winner, totalRounds, duration }` |
| `game:{id}:abandon` | Oyun terk edildiğinde | `{ round, reason }` |

### 11.3 Başarı Kriterleri

| KPI | Hedef | Ölçüm Yöntemi |
|-----|-------|---------------|
| Tamamlama oranı | > {x}% | Başlayan / Bitiren |
| Ortalama süre | {x} dakika | Toplam süre / Oyun sayısı |
| Tekrar oynama | > {x}% | 2+ kez oynayan / Toplam |

---

## 12. ACHIEVEMENTS (Opsiyonel)

### 12.1 Başarı Listesi

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `{ach-1}` | {İsim} | {Açıklama} | {Unlock koşulu} |
| `{ach-2}` | {İsim} | {Açıklama} | {Unlock koşulu} |
| `{ach-3}` | {İsim} | {Açıklama} | {Unlock koşulu} |

### 12.2 Başarı Detayları

#### {ach-1}: {İsim}

```
{
  id: '{ach-1}',
  name: { en: '{English}', tr: '{Türkçe}' },
  description: { en: '{...}', tr: '{...}' },
  icon: '{icon path}',
  condition: {
    type: '{condition type}',
    value: {threshold}
  },
  reward: {
    type: '{reward type}',
    value: {amount}
  }
}
```

---

## 13. i18n KEYS

### 13.1 Oyun-Özel Çeviri Anahtarları

```
// locales/en/games.json içine eklenecek

"{game-id}": {
  "name": "{English name}",
  "description": "{English description}",
  "rules": {
    "title": "Rules",
    "items": [
      "{Rule 1}",
      "{Rule 2}"
    ]
  },
  "actions": {
    "{action1}": "{Action 1 label}",
    "{action2}": "{Action 2 label}"
  },
  "messages": {
    "yourTurn": "{Your turn message}",
    "waiting": "{Waiting message}",
    "won": "{Win message}",
    "lost": "{Lose message}",
    "draw": "{Draw message}"
  },
  "tutorial": {
    "step1": { "title": "...", "content": "..." },
    "step2": { "title": "...", "content": "..." }
  }
}
```

### 13.2 Türkçe Çeviriler

```
// locales/tr/games.json içine eklenecek

"{game-id}": {
  "name": "{Türkçe isim}",
  "description": "{Türkçe açıklama}",
  ...
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Core Bağımlılıkları

| Modül | Kullanım |
|-------|----------|
| `/core/store.js` | State yönetimi |
| `/core/events.js` | Event dispatch |
| `/core/utils/i18n.js` | Çeviri |
| `/core/components/base-component.js` | Component base class |

### 14.2 Paylaşılan Component'lar

| Component | Kullanım |
|-----------|----------|
| `<modal-dialog>` | {Nerede kullanılıyor} |
| `<toast-message>` | {Nerede kullanılıyor} |
| `<loading-spinner>` | {Nerede kullanılıyor} |

### 14.3 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/{id}.svg` |
| Thumbnail | `/assets/images/games/{id}-thumb.svg` |
| Sesler | `/assets/sounds/games/{id}/` |

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| {Karar 1} | {Neden böyle yapıldı} |
| {Karar 2} | {Neden böyle yapıldı} |

### 15.2 Bilinen Limitasyonlar

- {Limitasyon 1}
- {Limitasyon 2}

### 15.3 Gelecek İyileştirmeler

- [ ] {İyileştirme 1}
- [ ] {İyileştirme 2}
- [ ] {İyileştirme 3}

### 15.4 Referanslar

- {Akademik kaynak veya link 1}
- {Akademik kaynak veya link 2}
- {İlham alınan uygulama}

---

## 16. ONAY

| Rol | İsim | Tarih | Durum |
|-----|------|-------|-------|
| Tasarım | {isim} | {tarih} | {Onaylandı/Bekliyor} |
| Geliştirme | {isim} | {tarih} | {Onaylandı/Bekliyor} |
| Test | {isim} | {tarih} | {Onaylandı/Bekliyor} |

---

## 17. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik | Yazar |
|----------|-------|------------|-------|
| 0.1 | {tarih} | İlk taslak | {isim} |
| 1.0 | {tarih} | Onaylanan versiyon | {isim} |

---

> **Kullanım:** Bu şablonu `/docs/games/` altına kopyalayın ve tüm `{placeholder}` alanlarını doldurun.
