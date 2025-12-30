# BATTLE OF THE SEXES
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `battle-of-sexes` |
| Versiyon | 1.0.0 |
| Kategori | coordination |
| Zorluk | easy |
| Tier | premium |
| Durum | approved |

---

## 1. ÖZET

### 1.1 Kısa Açıklama

| Dil | Açıklama |
|-----|----------|
| EN | Two friends want to meet but prefer different places. Can you coordinate? |
| TR | İki arkadaş buluşmak istiyor ama farklı yerleri tercih ediyor. Koordine olabilir misiniz? |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| Koordinasyon | Birlikte olmak tek başına olmaktan iyidir |
| Fedakarlık | Bazen kendi tercihinden vazgeçmek gerekir |
| Pazarlık | Kim kimin tercihine uyacak? |
| Çoklu Denge | Birden fazla "doğru" çözüm olabilir |
| Adalet | Dönüşümlü fedakarlık |

### 1.3 Gerçek Hayat Bağlantısı

Bu oyun gerçek hayatta şu durumlara benzer:

- **Akşam Planı:** Sinemaya mı, restorana mı gidelim?
- **Tatil:** Denize mi, dağa mı gidelim?
- **Yemek Siparişi:** Pizza mı, sushi mi?
- **Hafta Sonu:** Evde mi kalalım, dışarı mı çıkalım?
- **Film Seçimi:** Aksiyon mu, romantik komedi mi?

### 1.4 Oyun Adı Notu

```
Klasik isim "Battle of the Sexes" cinsiyet stereotipleri içerir.
Biz daha nötr "Buluşma Noktası" veya "Tercih Koordinasyonu" 
ismini de kullanabiliriz.

Alternatif senaryolar:
- Opera vs Futbol → Sinema vs Lunapark
- Karı-koca → İki arkadaş
```

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. İki arkadaş akşam buluşmak istiyor
2. İletişim yok - aynı anda yer seçmeliler
3. Her birinin farklı bir tercihi var
4. Aynı yere giderlerse ikisi de mutlu (biri daha mutlu)
5. Farklı yerlere giderlerse ikisi de mutsuz
```

### 2.2 Hikaye Bağlamı

```
Ali ve Ayşe bu akşam buluşmak istiyor.
Ali sinemayı tercih ediyor, Ayşe lunaparkı.
Ama en önemlisi birlikte olmak!

- İkisi de sinemaya: Ali çok mutlu, Ayşe mutlu
- İkisi de lunaparka: Ayşe çok mutlu, Ali mutlu  
- Farklı yerlere: İkisi de yalnız ve mutsuz
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
Koordinasyon başarılıysa:
- Favori yere giden daha yüksek puan alır
- Ama ikisi de pozitif puan alır

Koordinasyon başarısızsa:
- İkisi de 0 puan alır
- Kimse kazanmaz

Hedef: Koordine olmak, mümkünse kendi yerinde!
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `CHOOSE` | Buluşma yeri seçimi | Henüz seçim yapılmamışsa |

### 3.2 Eylem Detayları

#### CHOOSE

```
Action {
  type: 'CHOOSE'
  playerId: string
  payload: {
    choice: 'cinema' | 'park'  // veya tema bazlı
  }
}
```

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi

```
                         Oyuncu 2 (Lunapark sever)
                      Sinema        Lunapark
Oyuncu 1    Sinema    (3, 2)         (0, 0)
(Sinema     Lunapark  (0, 0)         (2, 3)
 sever)
```

| Sonuç | P1 | P2 | Açıklama |
|-------|----|----|----------|
| Sinema-Sinema | 3 | 2 | Birlikte sinema! P1 favorisi |
| Sinema-Lunapark | 0 | 0 | Kaçırdılar! Yalnız kaldılar |
| Lunapark-Sinema | 0 | 0 | Kaçırdılar! Yalnız kaldılar |
| Lunapark-Lunapark | 2 | 3 | Birlikte lunapark! P2 favorisi |

### 4.2 Oyun Teorisi Analizi

```
İki Nash Dengesi (saf strateji):
1. (Sinema, Sinema) - P1'in favorisi
2. (Lunapark, Lunapark) - P2'nin favorisi

Bir de karışık strateji dengesi var.

Problem: Hangi dengeye koordine olunacak?
- Önceki deneyim
- Sosyal norm
- Focal point (belirgin nokta)
```

### 4.3 Skor Hesaplama

```
function calculatePayoff(p1Choice, p2Choice, p1Prefers, p2Prefers):
    // Koordinasyon başarısız
    if (p1Choice !== p2Choice):
        return { p1: 0, p2: 0 }
    
    // Koordinasyon başarılı
    if (p1Choice === p1Prefers):
        return { p1: 3, p2: 2 }  // P1'in favorisi
    else:
        return { p1: 2, p2: 3 }  // P2'nin favorisi
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  // Tercihler
  player1Preference: 'cinema' | 'park'
  player2Preference: 'cinema' | 'park'
  
  // Seçimler
  player1Choice: 'cinema' | 'park' | null
  player2Choice: 'cinema' | 'park' | null
  
  // Sonuç
  coordinated: boolean | null
  favoredPlayer: string | null  // Kimin tercihi gerçekleşti
}
```

### 5.2 Örnek Initial State

```
{
  gameId: "game-bos-001",
  moduleId: "battle-of-sexes",
  
  players: [
    { id: "player-1", type: "human", name: "Ali", score: 0 },
    { id: "player-2", type: "bot", name: "Ayşe Bot", score: 0 }
  ],
  
  phase: "playing",
  round: 1,
  maxRounds: 1,
  
  data: {
    player1Preference: "cinema",
    player2Preference: "park",
    player1Choice: null,
    player2Choice: null,
    coordinated: null,
    favoredPlayer: null
  },
  
  history: [],
  result: null
}
```

### 5.3 Örnek Final State (Başarılı Koordinasyon)

```
{
  gameId: "game-bos-001",
  moduleId: "battle-of-sexes",
  
  players: [
    { id: "player-1", type: "human", name: "Ali", score: 3 },
    { id: "player-2", type: "bot", name: "Ayşe Bot", score: 2 }
  ],
  
  phase: "finished",
  
  data: {
    player1Preference: "cinema",
    player2Preference: "park",
    player1Choice: "cinema",
    player2Choice: "cinema",
    coordinated: true,
    favoredPlayer: "player-1"
  },
  
  result: {
    winnerId: "player-1",
    finalScores: { "player-1": 3, "player-2": 2 },
    summary: {
      en: "You both went to the cinema! Ali's favorite place.",
      tr: "İkiniz de sinemaya gittiniz! Ali'nin favori yeri."
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
|  Buluşma Noktası  |  🎬 vs 🎡                 |
+-----------------------------------------------+
|                                               |
|  SCENARIO                                     |
|  +-------------------------------------------+|
|  | Bu akşam buluşmak istiyorsunuz!           ||
|  |                                           ||
|  | 👤 Sen: Sinema tercih ediyorsun 🎬        ||
|  | 🤖 Rakip: Lunapark tercih ediyor 🎡       ||
|  |                                           ||
|  | Ama en önemlisi BİRLİKTE olmak!          ||
|  +-------------------------------------------+|
|                                               |
|  IMPORTANT NOTE                               |
|  ⚠️ Farklı yerlere giderseniz buluşamazsınız!|
|                                               |
+-----------------------------------------------+
|  CHOICE BUTTONS                               |
|                                               |
|  +--------------------+  +------------------+ |
|  |  🎬 SİNEMA         |  |  🎡 LUNAPARK    | |
|  |  (Senin favorin)   |  |  (Onun favorisi)|  |
|  |  Birlikte: 3-2     |  |  Birlikte: 2-3  | |
|  +--------------------+  +------------------+ |
|                                               |
+-----------------------------------------------+
```

### 6.2 Reveal Screen - Koordinasyon Başarılı

```
+-----------------------------------------------+
|            🎉 BULUŞTUNUZ! 🎉                  |
+-----------------------------------------------+
|                                               |
|              🎬 SİNEMA 🎬                     |
|                                               |
|     👤 +3 puan      🤖 +2 puan               |
|       (Favori!)       (Fedakarlık)           |
|                                               |
|    "Birlikte harika bir akşam geçirdiniz!"   |
|                                               |
+-----------------------------------------------+
```

### 6.3 Reveal Screen - Koordinasyon Başarısız

```
+-----------------------------------------------+
|            😢 BULUŞAMDINIZ 😢                 |
+-----------------------------------------------+
|                                               |
|     🎬 Sinema          🎡 Lunapark           |
|        👤                 🤖                  |
|       (Sen)             (Rakip)              |
|                                               |
|     0 puan             0 puan                |
|                                               |
|    "Farklı yerlere gittiniz, yalnız kaldınız"|
|                                               |
+-----------------------------------------------+
```

### 6.4 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| Scene | `<bos-scene>` | Senaryo açıklaması |
| Preference Badge | `<bos-preference>` | Kimin neyi tercih ettiği |
| Location Card | `<bos-location>` | Yer kartı (sinema/park) |
| Payoff Preview | `<bos-payoff-preview>` | Olası sonuçlar |
| Result Scene | `<bos-result>` | Sonuç animasyonu |

### 6.5 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| Character walk | Seçim yapılınca | 800ms |
| Meet animation | Koordinasyon başarılı | 1000ms |
| Lonely animation | Koordinasyon başarısız | 800ms |
| Confetti | Başarılı koordinasyon | 1500ms |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `stubborn` | İnatçı | Easy | Hep kendi tercihini seçer |
| `yielding` | Uyumlu | Easy | Hep rakibin tercihini seçer |
| `random` | Rastgele | Easy | %50-%50 |
| `alternating` | Dönüşümlü | Medium | Sırayla fedakarlık |
| `tit-for-tat` | Karşılıklı | Hard | Rakip fedakarlık yaparsa yapar |

### 7.2 Strateji Detayları

#### stubborn: İnatçı

```
function decide(state, playerId):
    return myPreference  // Her zaman kendi tercihim
```

**Özellikler:**
- Asla taviz vermez
- Koordinasyon rakibe bağlı
- Bencil strateji

#### yielding: Uyumlu

```
function decide(state, playerId):
    return opponentPreference  // Her zaman rakibin tercihi
```

**Özellikler:**
- Her zaman fedakarlık yapar
- Koordinasyon garanti (eğer rakip inatçı değilse)
- Fedakar strateji

#### alternating: Dönüşümlü (Multi-round için)

```
function decide(state, playerId):
    roundNumber = state.round
    if (roundNumber % 2 === 1):
        return myPreference  // Tek turlarda benim
    return opponentPreference  // Çift turlarda onun
```

#### tit-for-tat: Karşılıklı

```
function decide(state, playerId):
    if (no previous game):
        return myPreference  // İlk turda kendi tercihim
    
    if (opponent yielded last time):
        return opponentPreference  // Ben de fedakarlık yapayım
    
    return myPreference  // Yoksa kendi tercihim
```

### 7.3 Varsayılan Strateji

```
defaultStrategyId: 'random'

Zorluk bazlı:
- Easy: 'yielding', 'stubborn'
- Medium: 'random', 'alternating'
- Hard: 'tit-for-tat'
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef |
|------|--------|-------|
| 1 | Hikaye | Buluşma senaryosu |
| 2 | Tercihler | Kimin neyi tercih ettiği |
| 3 | Koordinasyon | Birlikte olmanın önemi |
| 4 | Fedakarlık | Bazen taviz vermek gerekir |
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
| E1 | İki inatçı oyuncu | Her zaman 0-0 sonuç |
| E2 | İki uyumlu oyuncu | Rastgele yer, her ikisi +2 veya +3 |
| E3 | Tercihler aynıysa | Koordinasyon kolay, her zaman birlikte |

---

## 10. TEST SENARYOLARI

### 10.1 Given-When-Then Senaryoları

#### Senaryo 1: Oyuncu Favorisinde Buluşma

```
GIVEN: Oyuncu sinema sever, bot lunapark sever
WHEN: Her ikisi de "Sinema" seçer
THEN: Oyuncu 3 puan, Bot 2 puan alır
  AND: "Birlikte sinemaya gittiniz!" mesajı
```

#### Senaryo 2: Bot Favorisinde Buluşma

```
GIVEN: Oyuncu sinema sever, bot lunapark sever
WHEN: Her ikisi de "Lunapark" seçer
THEN: Oyuncu 2 puan, Bot 3 puan alır
  AND: "Birlikte lunaparka gittiniz!" mesajı
```

#### Senaryo 3: Kaçırma

```
GIVEN: Oyuncu sinema sever, bot lunapark sever
WHEN: Oyuncu "Sinema", Bot "Lunapark" seçer
THEN: Her ikisi 0 puan alır
  AND: "Farklı yerlere gittiniz!" mesajı
```

---

## 11. METRİKLER

### 11.1 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:battle-of-sexes:start` | Oyun başı | `{ botStrategy }` |
| `game:battle-of-sexes:choice` | Seçim | `{ choice, isFavorite }` |
| `game:battle-of-sexes:complete` | Oyun sonu | `{ coordinated, favoredPlayer }` |

### 11.2 Oyun Metrikleri

| Metrik | Açıklama |
|--------|----------|
| Coordination Rate | Başarılı koordinasyon oranı |
| Yielding Rate | Kendi tercihinden vazgeçme oranı |
| Mutual Miss Rate | Kaçırma oranı |

---

## 12. ACHIEVEMENTS

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `bos-first-meet` | İlk Buluşma | İlk başarılı koordinasyon | coordinated = true |
| `bos-compromise` | Uzlaşmacı | 3 kez fedakarlık yap | 3x yield |
| `bos-my-way` | Kendi Yolum | 3 kez kendi favorinde buluş | 3x favored |
| `bos-perfect-pair` | Mükemmel Çift | 5 oyun üst üste koordine ol | 5 consecutive coordination |

---

## 13. i18n KEYS

### 13.1 İngilizce

```json
"battle-of-sexes": {
  "name": "Meeting Point",
  "description": "Coordinate with different preferences",
  "locations": {
    "cinema": "Cinema",
    "park": "Theme Park"
  },
  "outcomes": {
    "coordinated_your_place": "You met at your favorite place!",
    "coordinated_their_place": "You met at their favorite place!",
    "missed": "You went to different places and missed each other!"
  },
  "preferences": {
    "you_prefer": "You prefer",
    "they_prefer": "They prefer"
  }
}
```

### 13.2 Türkçe

```json
"battle-of-sexes": {
  "name": "Buluşma Noktası",
  "description": "Farklı tercihlerle koordine ol",
  "locations": {
    "cinema": "Sinema",
    "park": "Lunapark"
  },
  "outcomes": {
    "coordinated_your_place": "Senin favori yerinde buluştunuz!",
    "coordinated_their_place": "Onun favori yerinde buluştunuz!",
    "missed": "Farklı yerlere gittiniz, buluşamadınız!"
  },
  "preferences": {
    "you_prefer": "Sen tercih ediyorsun",
    "they_prefer": "O tercih ediyor"
  }
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/battle-of-sexes.svg` |
| Sinema | `/assets/images/games/cinema.svg` |
| Lunapark | `/assets/images/games/park.svg` |
| Karakter 1 | `/assets/images/games/character-1.svg` |
| Karakter 2 | `/assets/images/games/character-2.svg` |

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| İsim değişikliği | Cinsiyet stereotiplerinden kaçınma |
| Sinema/Lunapark | Çocuk dostu, evrensel |
| Premium tier | Diğer koordinasyon oyunlarından farklı (asimetrik tercihler) |

### 15.2 Stag Hunt ile Karşılaştırma

| Özellik | Stag Hunt | Battle of Sexes |
|---------|-----------|-----------------|
| Tercihler | Aynı | Farklı |
| En iyi sonuç | Tek (Stag-Stag) | İki (her birinin favorisi) |
| Koordinasyon zorluğu | Daha kolay | Daha zor |
| Adalet sorunu | Yok | Var (kim fedakarlık yapacak?) |

### 15.3 Gelecek İyileştirmeler

- [ ] Çok turlu versiyon (adalet için dönüşüm)
- [ ] Üçüncü seçenek ekleme (compromise location)
- [ ] Custom tercih tanımlama
- [ ] Gerçek multiplayer

### 15.4 Referanslar

- Luce & Raiffa - "Games and Decisions"
- Coordination Game literature
- Focal Point theory (Schelling)

---

## 16. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |
