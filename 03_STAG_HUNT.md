# STAG HUNT
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `stag-hunt` |
| Versiyon | 1.0.0 |
| Kategori | coordination |
| Zorluk | easy |
| Tier | free |
| Durum | approved |

---

## 1. ÖZET

### 1.1 Kısa Açıklama

| Dil | Açıklama |
|-----|----------|
| EN | Hunt the stag together for a big reward, or play it safe with a rabbit? |
| TR | Büyük ödül için birlikte geyik mi avlayacaksın, yoksa güvenli tavşanı mı seçeceksin? |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| Koordinasyon | Birlikte hareket etmenin değerini gösterir |
| Risk vs Ödül | Güvenli seçenek vs riskli ama yüksek ödül |
| Güven | Başkalarının da işbirliği yapacağına güvenme |
| Pareto Optimum | En iyi sonucun koordinasyon gerektirdiğini gösterir |
| Sosyal Sözleşme | Toplumsal işbirliğinin temellerini anlatır |

### 1.3 Gerçek Hayat Bağlantısı

Bu oyun gerçek hayatta şu durumlara benzer:

- **Takım Sporu:** Herkes pas oynarsa gol olur, ama birisi bencil oynarsa?
- **Grup Projesi:** Herkes çalışırsa A alırız, ama ya biri çalışmazsa?
- **Çevre Koruma:** Herkes az su kullanırsa kuraklık olmaz
- **Trafik:** Herkes kurallara uyarsa trafik akar
- **Aşı:** Herkes aşı olursa salgın biter

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. İki avcı birlikte ava çıkar
2. Her avcı "Geyik Avla" veya "Tavşan Avla" seçer
3. Geyik avlamak için İKİSİ DE geyik seçmeli
4. Tavşan tek başına avlanabilir
5. Geyik > 2 Tavşan (işbirliği daha değerli)
```

### 2.2 Hikaye Bağlamı

```
İki avcı ormanda. Uzakta bir geyik var - büyük ve değerli ama
yakalamak için iki kişi gerekiyor. Yolda tavşanlar da var -
küçük ama tek başına yakalanabilir.

Geyiği kovalarsın ama ortağın tavşana giderse, eli boş kalırsın.
Tavşanı seçersen en azından bir şeyin olur, ama geyik kaçar.

Ne yaparsın?
```

### 2.3 Oyuncu Bilgisi

| Alan | Değer |
|------|-------|
| Minimum Oyuncu | 2 |
| Maximum Oyuncu | 2 |
| Optimal Oyuncu | 2 |
| Bot Desteği | Evet |
| Eşzamanlı Hamle | Evet |

### 2.4 Tur Yapısı

| Alan | Değer |
|------|-------|
| Tur Sayısı | 1 (tek tur) |
| Varsayılan Tur | 1 |
| Tur Süresi | Süresiz |

### 2.5 Kazanma Koşulu

```
Tek turda oyun biter.
En yüksek puan alan kazanır.
Eşitlik durumunda berabere.
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `CHOOSE` | Avcı av tercihini yapar | Henüz seçim yapılmamışsa |

### 3.2 Eylem Detayları

#### CHOOSE

```
Action {
  type: 'CHOOSE'
  playerId: string
  payload: {
    choice: 'stag' | 'rabbit'
  }
}
```

**Validation Kuralları:**
- Geçerli oyuncu ID'si
- Henüz seçim yapılmamış olmalı
- choice 'stag' veya 'rabbit' olmalı

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi

```
                      Oyuncu 2
                   Geyik      Tavşan
Oyuncu 1  Geyik    (4, 4)     (0, 3)
          Tavşan   (3, 0)     (3, 3)
```

| Sonuç | Puan | Açıklama |
|-------|------|----------|
| Geyik-Geyik | 4, 4 | Birlikte geyik avladınız! En iyi sonuç |
| Geyik-Tavşan | 0, 3 | Sen bekledin, o tavşan yakaladı |
| Tavşan-Geyik | 3, 0 | Sen tavşan yakaladın, o eli boş |
| Tavşan-Tavşan | 3, 3 | İkiniz de tavşan yakaladınız |

### 4.2 Oyun Teorisi Analizi

```
İki Nash Dengesi var:
1. (Geyik, Geyik) - Pareto optimal, risk dominant DEĞİL
2. (Tavşan, Tavşan) - Risk dominant, Pareto optimal DEĞİL

Prisoner's Dilemma'dan farkı:
- PD'de tek Nash dengesi var (Defect, Defect)
- Stag Hunt'ta işbirliği de bir denge
- Sorun: hangi dengeye koordine olunacak?
```

### 4.3 Skor Hesaplama

```
function calculatePayoff(player1Choice, player2Choice):
    matrix = {
        'stag-stag':     { p1: 4, p2: 4 },
        'stag-rabbit':   { p1: 0, p2: 3 },
        'rabbit-stag':   { p1: 3, p2: 0 },
        'rabbit-rabbit': { p1: 3, p2: 3 }
    }
    return matrix[player1Choice + '-' + player2Choice]
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  player1Choice: 'stag' | 'rabbit' | null
  player2Choice: 'stag' | 'rabbit' | null
  
  outcome: {
    type: 'stag_success' | 'stag_fail' | 'both_rabbit' | null
    stagHunter: string | null    // Geyik seçip eli boş kalan
  } | null
}
```

### 5.2 Örnek Initial State

```
{
  gameId: "game-sh-001",
  moduleId: "stag-hunt",
  
  players: [
    { id: "player-1", type: "human", name: "Avcı 1", score: 0 },
    { id: "player-2", type: "bot", name: "Avcı Bot", score: 0, strategyId: "stag-prefer" }
  ],
  
  phase: "playing",
  round: 1,
  maxRounds: 1,
  
  data: {
    player1Choice: null,
    player2Choice: null,
    outcome: null
  },
  
  history: [],
  result: null
}
```

### 5.3 Örnek Final State (Başarılı Geyik Avı)

```
{
  gameId: "game-sh-001",
  moduleId: "stag-hunt",
  
  players: [
    { id: "player-1", type: "human", name: "Avcı 1", score: 4 },
    { id: "player-2", type: "bot", name: "Avcı Bot", score: 4 }
  ],
  
  phase: "finished",
  
  data: {
    player1Choice: "stag",
    player2Choice: "stag",
    outcome: {
      type: "stag_success",
      stagHunter: null
    }
  },
  
  result: {
    winnerId: null,  // Berabere
    finalScores: { "player-1": 4, "player-2": 4 },
    summary: {
      en: "You both hunted the stag! Maximum reward achieved!",
      tr: "Birlikte geyiği avladınız! Maksimum ödül kazandınız!"
    }
  }
}
```

---

## 6. UI TASARIMI

### 6.1 Ekran Listesi

| Ekran | Açıklama | Önem |
|-------|----------|------|
| Setup | Bot/zorluk seçimi | Zorunlu |
| Story | Hikaye anlatımı | Opsiyonel |
| Main Game | Av seçimi | Zorunlu |
| Reveal | Sonuç animasyonu | Zorunlu |
| Result | Final ve açıklama | Zorunlu |

### 6.2 Main Game Layout

```
+-----------------------------------------------+
|                HEADER                          |
|  Geyik Avı  |  🦌 vs 🐰                       |
+-----------------------------------------------+
|                                               |
|              SCENE ILLUSTRATION               |
|  +-------------------------------------------+|
|  |                                           ||
|  |     🦌                        🐰 🐰       ||
|  |    (Geyik)                   (Tavşanlar) ||
|  |                                           ||
|  |        👤          👤                     ||
|  |      (Sen)      (Rakip)                   ||
|  |                                           ||
|  +-------------------------------------------+|
|                                               |
|  "Birlikte geyik mi, yoksa güvenli tavşan mı?"|
|                                               |
+-----------------------------------------------+
|  CHOICE BUTTONS                               |
|                                               |
|  +--------------------+  +------------------+ |
|  |  🦌 GEYİK         |  |  🐰 TAVŞAN      | |
|  |  Riskli ama büyük |  |  Güvenli        | |
|  |  (4 puan birlikte)|  |  (3 puan)       | |
|  +--------------------+  +------------------+ |
|                                               |
+-----------------------------------------------+
```

### 6.3 Reveal Screen - Başarılı Geyik

```
+-----------------------------------------------+
|                 BAŞARDIN!                      |
+-----------------------------------------------+
|                                               |
|              🦌 GEYİK AVLANDI! 🦌             |
|                                               |
|      👤 Geyik    ←  →    👤 Geyik            |
|      (Sen)              (Rakip)              |
|                                               |
|              Birlikte başardınız!             |
|                                               |
|         +4 puan         +4 puan               |
|                                               |
+-----------------------------------------------+
|  [Tekrar Oyna]              [Farklı Oyun]    |
+-----------------------------------------------+
```

### 6.4 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| Scene View | `<sh-scene>` | Orman sahnesi illüstrasyonu |
| Animal Icon | `<sh-animal>` | Geyik/tavşan animasyonlu ikon |
| Choice Card | `<sh-choice-card>` | Seçim kartı (bilgi dahil) |
| Hunter Avatar | `<sh-hunter>` | Avcı avatarı |
| Result Scene | `<sh-result-scene>` | Sonuç animasyonu |

### 6.5 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| Stag walk | Sayfa yüklenince | Loop |
| Rabbits hop | Sayfa yüklenince | Loop |
| Choice highlight | Hover | 200ms |
| Hunt success | Stag-Stag sonucu | 1500ms |
| Stag escape | Stag-Rabbit sonucu | 1000ms |
| Rabbit catch | Tavşan seçilince | 800ms |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `stag-prefer` | Geyik Tercihli | Easy | %80 geyik, %20 tavşan |
| `rabbit-prefer` | Tavşan Tercihli | Easy | %80 tavşan, %20 geyik |
| `random` | Rastgele | Easy | %50-%50 |
| `copycat` | Taklitçi | Medium | Önceki oyuna göre |
| `risk-averse` | Riskten Kaçınan | Hard | Hep tavşan |

### 7.2 Strateji Detayları

#### stag-prefer: Geyik Tercihli

```
function decide(state, playerId):
    if (Math.random() < 0.8):
        return 'stag'
    return 'rabbit'
```

**Özellikler:**
- İşbirliğine açık
- Bazen hayal kırıklığına uğrar
- Çocuklar için iyi başlangıç

#### copycat: Taklitçi

```
function decide(state, playerId):
    // İlk oyunsa geyik dene
    if (no previous game):
        return 'stag'
    
    // Önceki oyunda ne olduysa onu yap
    if (previous was stag-stag):
        return 'stag'  // İşe yaradı, devam
    else if (previous was rabbit-*):
        return 'rabbit'  // Güvenli oyna
    else:
        return 'rabbit'  // Geyikte yanıldım, güvenliğe geç
```

#### risk-averse: Riskten Kaçınan

```
function decide(state, playerId):
    return 'rabbit'  // Her zaman güvenli seçenek
```

**Özellikler:**
- Asla 0 puan almaz
- Asla 4 puan alamaz
- Garantici strateji

### 7.3 Varsayılan Strateji

```
defaultStrategyId: 'stag-prefer'

Zorluk bazlı:
- Easy: 'stag-prefer'
- Medium: 'random', 'copycat'
- Hard: 'risk-averse'
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef |
|------|--------|-------|
| 1 | Hikaye | Avcı senaryosunu anlat |
| 2 | Geyik | Geyik avının zorluğu |
| 3 | Tavşan | Güvenli seçeneğin avantajı |
| 4 | İkilem | Koordinasyon problemi |
| 5 | Dene | İlk seçimi yap |
| 6 | Fark | PD'den farkını açıkla |

### 8.2 PD ile Fark Açıklaması

```
Tutorial'da gösterilecek karşılaştırma:

Prisoner's Dilemma:
- İhanet etmek her zaman bireysel olarak "mantıklı"
- İşbirliği yapmak risk

Stag Hunt:
- İşbirliği yapmak EN İYİ sonuç
- Ama koordinasyon gerekiyor
- Güven meselesi
```

---

## 9. EDGE CASES

### 9.1 Olası Durumlar

| # | Durum | Beklenen Davranış |
|---|-------|-------------------|
| E1 | Her iki oyuncu geyik | Başarı animasyonu, 4-4 |
| E2 | Her iki oyuncu tavşan | Her ikisi de 3 puan |
| E3 | Biri geyik, biri tavşan | Geyikçi 0, tavşancı 3 |
| E4 | Zaman aşımı | Otomatik tavşan (güvenli) |

---

## 10. TEST SENARYOLARI

### 10.1 Given-When-Then Senaryoları

#### Senaryo 1: Başarılı Koordinasyon

```
GIVEN: Yeni oyun başlatıldı
WHEN: Her iki oyuncu "Geyik" seçer
THEN: Her ikisi 4 puan alır
  AND: "Birlikte geyik avladınız!" mesajı gösterilir
  AND: Berabere sonucu
```

#### Senaryo 2: Başarısız Risk

```
GIVEN: Yeni oyun başlatıldı
  AND: Bot stratejisi "risk-averse"
WHEN: Oyuncu "Geyik" seçer
  AND: Bot "Tavşan" seçer
THEN: Oyuncu 0 puan alır
  AND: Bot 3 puan alır
  AND: "Geyik kaçtı!" mesajı gösterilir
```

#### Senaryo 3: Güvenli Oyun

```
GIVEN: Yeni oyun
WHEN: Her iki oyuncu "Tavşan" seçer
THEN: Her ikisi 3 puan alır
  AND: "İkiniz de tavşan yakaladınız" mesajı
```

---

## 11. METRİKLER

### 11.1 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:stag-hunt:start` | Oyun başı | `{ botStrategy }` |
| `game:stag-hunt:choice` | Seçim | `{ choice }` |
| `game:stag-hunt:complete` | Oyun sonu | `{ outcome, scores }` |

### 11.2 Oyun Metrikleri

| Metrik | Açıklama |
|--------|----------|
| Stag Success Rate | Başarılı geyik avı oranı |
| Risk Taking Rate | Geyik seçme oranı |
| Coordination Rate | Aynı seçimi yapma oranı |

---

## 12. ACHIEVEMENTS

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `sh-first-stag` | İlk Geyik | İlk başarılı geyik avı | Stag-Stag sonucu |
| `sh-trusting` | Güvenen | 5 kez üst üste geyik seç | 5 ardışık stag |
| `sh-safe-player` | Garantici | 5 kez üst üste tavşan seç | 5 ardışık rabbit |
| `sh-coordinator` | Koordinatör | 10 oyunda 8+ kez aynı seçim | %80+ coordination |

---

## 13. i18n KEYS

### 13.1 İngilizce

```json
"stag-hunt": {
  "name": "Stag Hunt",
  "description": "Hunt together or play it safe?",
  "choices": {
    "stag": "Hunt Stag",
    "rabbit": "Hunt Rabbit"
  },
  "outcomes": {
    "stag_success": "You both hunted the stag!",
    "stag_fail": "The stag escaped! One hunter went for rabbit.",
    "both_rabbit": "You both caught rabbits."
  },
  "hints": {
    "stag": "High reward, but needs coordination",
    "rabbit": "Safe choice, guaranteed reward"
  }
}
```

### 13.2 Türkçe

```json
"stag-hunt": {
  "name": "Geyik Avı",
  "description": "Birlikte avlan mı, güvenli oyna mı?",
  "choices": {
    "stag": "Geyik Avla",
    "rabbit": "Tavşan Avla"
  },
  "outcomes": {
    "stag_success": "Birlikte geyiği avladınız!",
    "stag_fail": "Geyik kaçtı! Bir avcı tavşana gitti.",
    "both_rabbit": "İkiniz de tavşan yakaladınız."
  },
  "hints": {
    "stag": "Yüksek ödül, ama koordinasyon gerek",
    "rabbit": "Güvenli seçim, garantili ödül"
  }
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/stag-hunt.svg` |
| Geyik Sprite | `/assets/images/games/stag.svg` |
| Tavşan Sprite | `/assets/images/games/rabbit.svg` |
| Orman Arka Plan | `/assets/images/games/forest-bg.svg` |

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| Free tier | Temel oyun, anlaşılması kolay |
| Görsel hikaye | Çocuklar için somutlaştırma |
| 4-3-0 değerleri | Standart Stag Hunt payoff'ları |
| Tek tur | Basitlik, PD ile tutarlılık |

### 15.2 PD ile Karşılaştırma

| Özellik | Prisoner's Dilemma | Stag Hunt |
|---------|-------------------|-----------|
| Nash Dengesi | 1 (Defect-Defect) | 2 (Stag-Stag, Rabbit-Rabbit) |
| Pareto Optimal | Cooperate-Cooperate | Stag-Stag |
| Risk | İşbirliği riskli | Geyik riskli |
| Mesaj | Bencillik vs işbirliği | Güven vs güvenlik |

### 15.3 Gelecek İyileştirmeler

- [ ] Multi-player versiyonu (3+ avcı)
- [ ] Iterated Stag Hunt
- [ ] Değişken ödül değerleri

### 15.4 Referanslar

- Jean-Jacques Rousseau - "Discourse on Inequality"
- Brian Skyrms - "The Stag Hunt and the Evolution of Social Structure"

---

## 16. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |
