# HAWK-DOVE
## Game Specification Document

| Alan | Değer |
|------|-------|
| Modül ID | `hawk-dove` |
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
| EN | Will you be aggressive like a Hawk or peaceful like a Dove? |
| TR | Şahin gibi saldırgan mı olacaksın, yoksa güvercin gibi barışçıl mı? |

### 1.2 Öğrettiği Konseptler

| Konsept | Açıklama |
|---------|----------|
| Saldırganlık Maliyeti | İki saldırgan çarpışırsa ikisi de kaybeder |
| Anti-koordinasyon | Farklı davranmak bazen en iyisi |
| Blöf | Karşı tarafı geri adım atmaya zorlama |
| Evrimsel Denge | Popülasyonda Şahin/Güvercin karışımı |
| Kaynak Paylaşımı | Sınırlı kaynaklar için rekabet |

### 1.3 Gerçek Hayat Bağlantısı

Bu oyun gerçek hayatta şu durumlara benzer:

- **Trafik:** İki araba dar yolda karşılaşınca kim geri çekilir?
- **Pazarlık:** Kim ilk teklifi verir, kim taviz verir?
- **Kardeş Kavgası:** Son kurabiye için kim savaşır, kim paylaşır?
- **Uluslararası Kriz:** Gerginlikte kim geri adım atar?
- **Okul:** Zorbalık ve karşı koyma dinamiği

### 1.4 Alternatif İsimler

- **Chicken Game** (Amerikan versiyonu)
- **Game of Chicken** (Araba yarışı senaryosu)
- **Brinkmanship** (Uluslararası ilişkilerde)

---

## 2. OYUN KURALLARI

### 2.1 Temel Kurallar

```
1. İki oyuncu bir kaynak için rekabet ediyor
2. Her oyuncu "Şahin" (saldırgan) veya "Güvercin" (barışçıl) seçer
3. Şahin savaşır, Güvercin geri çekilir
4. İki Şahin çarpışırsa ikisi de zarar görür
5. Şahin vs Güvercin → Şahin kazanır
6. İki Güvercin → Kaynağı paylaşırlar
```

### 2.2 Hikaye Bağlamı

```
İki hayvan aynı yiyecek kaynağını buldu. 
Her biri Şahin (savaş) veya Güvercin (geri çekil) gibi davranabilir.

- İki Şahin: Kavga! İkisi de yaralanır, kaynak hasar görür.
- Şahin vs Güvercin: Şahin kaynağı alır, Güvercin eli boş.
- İki Güvercin: Kaynağı barışçıl şekilde paylaşırlar.
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
En yüksek puan alan kazanır.
Eşitlik durumunda berabere.

ÖNEMLİ: Bu oyunda "en iyi" sonuç olmak
zorunda değil - bazen Güvercin olmak daha iyi!
```

---

## 3. EYLEMLER (Actions)

### 3.1 Eylem Listesi

| Eylem | Açıklama | Koşul |
|-------|----------|-------|
| `CHOOSE` | Oyuncu stratejisini seçer | Henüz seçim yapılmamışsa |

### 3.2 Eylem Detayları

#### CHOOSE

```
Action {
  type: 'CHOOSE'
  playerId: string
  payload: {
    choice: 'hawk' | 'dove'
  }
}
```

---

## 4. PAYOFF / SKORLAMA

### 4.1 Payoff Matrisi

```
                      Oyuncu 2
                   Şahin      Güvercin
Oyuncu 1  Şahin    (-2, -2)   (4, 0)
          Güvercin (0, 4)     (2, 2)
```

| Sonuç | Puan | Açıklama |
|-------|------|----------|
| Şahin-Şahin | -2, -2 | Kavga! İkisi de yaralandı |
| Şahin-Güvercin | 4, 0 | Şahin kaynağı aldı |
| Güvercin-Şahin | 0, 4 | Güvercin geri çekildi |
| Güvercin-Güvercin | 2, 2 | Barışçıl paylaşım |

### 4.2 Değer Açıklaması

```
V = Kaynak değeri = 4 puan
C = Kavga maliyeti = 6 puan

Şahin-Şahin: (V - C) / 2 = (4 - 6) / 2 = -1 → -2 (yuvarlanmış)
Şahin-Güvercin: V = 4, 0
Güvercin-Güvercin: V / 2 = 2, 2
```

### 4.3 Oyun Teorisi Analizi

```
Nash Dengeleri:
1. (Şahin, Güvercin) - Asimetrik
2. (Güvercin, Şahin) - Asimetrik
3. Karışık strateji: Her biri %66 Güvercin, %33 Şahin

Prisoner's Dilemma ve Stag Hunt'tan farkı:
- PD: İşbirliği ≠ Nash dengesi
- Stag Hunt: İşbirliği = Nash dengesi (biri)
- Hawk-Dove: "İşbirliği" = zıt davranış!
```

### 4.4 Skor Hesaplama

```
function calculatePayoff(player1Choice, player2Choice):
    matrix = {
        'hawk-hawk':     { p1: -2, p2: -2 },
        'hawk-dove':     { p1: 4, p2: 0 },
        'dove-hawk':     { p1: 0, p2: 4 },
        'dove-dove':     { p1: 2, p2: 2 }
    }
    return matrix[player1Choice + '-' + player2Choice]
```

---

## 5. STATE YAPISI

### 5.1 Game-Specific Data

```
GameData {
  player1Choice: 'hawk' | 'dove' | null
  player2Choice: 'hawk' | 'dove' | null
  
  outcome: {
    type: 'collision' | 'hawk_wins' | 'dove_wins' | 'sharing' | null
    winner: string | null
  } | null
}
```

### 5.2 Örnek Final State (Çarpışma)

```
{
  gameId: "game-hd-001",
  moduleId: "hawk-dove",
  
  players: [
    { id: "player-1", type: "human", name: "Player 1", score: -2 },
    { id: "player-2", type: "bot", name: "Hawk Bot", score: -2 }
  ],
  
  phase: "finished",
  
  data: {
    player1Choice: "hawk",
    player2Choice: "hawk",
    outcome: {
      type: "collision",
      winner: null
    }
  },
  
  result: {
    winnerId: null,  // İkisi de kaybetti
    finalScores: { "player-1": -2, "player-2": -2 },
    summary: {
      en: "Collision! Both hawks fought and got injured.",
      tr: "Çarpışma! İki şahin savaştı ve ikisi de yaralandı."
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
|  Şahin-Güvercin  |  🦅 vs 🕊️                 |
+-----------------------------------------------+
|                                               |
|              CONFRONTATION SCENE              |
|  +-------------------------------------------+|
|  |                                           ||
|  |        🦅              🕊️                ||
|  |       (Şahin)       (Güvercin)            ||
|  |                                           ||
|  |            💎 KAYNAK 💎                   ||
|  |                                           ||
|  +-------------------------------------------+|
|                                               |
|  "Kaynağı almak için saldıracak mısın?"      |
|                                               |
+-----------------------------------------------+
|  CHOICE BUTTONS                               |
|                                               |
|  +--------------------+  +------------------+ |
|  |  🦅 ŞAHİN          |  |  🕊️ GÜVERCİN   | |
|  |  Saldır!           |  |  Geri çekil     | |
|  |  Riskli: -2 veya 4 |  |  Güvenli: 0/2   | |
|  +--------------------+  +------------------+ |
|                                               |
+-----------------------------------------------+
```

### 6.2 Reveal Screens

#### Çarpışma (Hawk-Hawk)

```
+-----------------------------------------------+
|               💥 ÇARPIŞMA! 💥                  |
+-----------------------------------------------+
|                                               |
|           🦅  ⚔️  🦅                          |
|          -2      -2                           |
|                                               |
|    İki şahin savaştı, ikisi de yaralandı!    |
|                                               |
+-----------------------------------------------+
```

#### Şahin Kazandı

```
+-----------------------------------------------+
|             🦅 ŞAHİN KAZANDI!                 |
+-----------------------------------------------+
|                                               |
|        🦅                🕊️                   |
|        +4               0                     |
|       Kaynak            Geri çekildi          |
|                                               |
+-----------------------------------------------+
```

### 6.3 Component Listesi

| Component | Tag | Açıklama |
|-----------|-----|----------|
| Scene | `<hd-scene>` | Karşılaşma sahnesi |
| Animal | `<hd-animal>` | Şahin/Güvercin sprite |
| Resource | `<hd-resource>` | Kaynak (elmas/yiyecek) |
| Choice Card | `<hd-choice-card>` | Seçim kartı |
| Collision Effect | `<hd-collision>` | Çarpışma efekti |

### 6.4 Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| Hawk aggressive | Hawk seçildiğinde | 500ms |
| Dove retreat | Dove seçildiğinde | 500ms |
| Collision | Hawk-Hawk | 1000ms |
| Victory | Hawk-Dove | 800ms |
| Sharing | Dove-Dove | 600ms |

---

## 7. BOT STRATEJİLERİ

### 7.1 Strateji Listesi

| ID | İsim | Zorluk | Açıklama |
|----|------|--------|----------|
| `aggressive` | Saldırgan | Easy | %80 şahin |
| `peaceful` | Barışçıl | Easy | %80 güvercin |
| `random` | Rastgele | Easy | %50-%50 |
| `mixed-optimal` | Optimal Karışık | Medium | %33 şahin, %66 güvercin |
| `retaliator` | Misillemeci | Hard | Önceki sonuca göre |
| `bourgeois` | Sahipler | Hard | Kaynak "sahibi" ise şahin |

### 7.2 Strateji Detayları

#### mixed-optimal: Optimal Karışık Strateji

```
function decide(state, playerId):
    // Evrimsel olarak stabil strateji
    // V=4, C=6 için optimal: p(Hawk) = V/C = 4/6 ≈ 0.67 Dove
    if (Math.random() < 0.33):
        return 'hawk'
    return 'dove'
```

**Özellikler:**
- Uzun vadede ortalama skoru maksimize eder
- Tahmin edilemez
- Evrimsel Kararlı Strateji (ESS)

#### retaliator: Misillemeci

```
function decide(state, playerId):
    if (no previous game):
        return 'dove'  // Barışçıl başla
    
    // Rakip geçen sefer şahin mi oynadı?
    if (opponent played hawk last time):
        return 'hawk'  // Karşılık ver
    
    return 'dove'
```

#### bourgeois: Sahipler Stratejisi

```
function decide(state, playerId):
    // Kaynak "sahibi" belirle (ilk gelen veya random)
    if (isOwner):
        return 'hawk'  // Savun
    return 'dove'  // Saygı göster
```

**Özellikler:**
- Mülkiyet haklarını simüle eder
- Çarpışmayı önler
- Hayvan davranışlarına benzer

### 7.3 Varsayılan Strateji

```
defaultStrategyId: 'mixed-optimal'

Zorluk bazlı:
- Easy: 'aggressive', 'peaceful'
- Medium: 'random', 'mixed-optimal'
- Hard: 'retaliator', 'bourgeois'
```

---

## 8. TUTORIAL

### 8.1 Tutorial Akışı

| Adım | Başlık | Hedef |
|------|--------|-------|
| 1 | Hikaye | Kaynak rekabeti senaryosu |
| 2 | Şahin | Saldırgan stratejinin riski |
| 3 | Güvercin | Barışçıl stratejinin güvenliği |
| 4 | Çarpışma | İki şahin = felaket |
| 5 | Denge | Optimal strateji açıklaması |
| 6 | Dene | İlk seçimi yap |

### 8.2 Tutorial Ayarları

```
{
  isSkippable: true,
  showOnFirstPlay: true,
  estimatedTime: '3 min'
}
```

---

## 9. EDGE CASES

### 9.1 Olası Durumlar

| # | Durum | Beklenen Davranış |
|---|-------|-------------------|
| E1 | İki şahin | Negatif skor göster, uyarı mesajı |
| E2 | Sürekli şahin seçimi | Uyarı: "Dikkat, çarpışma riski!" |
| E3 | Negatif toplam skor | Skor 0'ın altına inebilir |

---

## 10. TEST SENARYOLARI

### 10.1 Given-When-Then Senaryoları

#### Senaryo 1: Çarpışma

```
GIVEN: Yeni oyun
  AND: Bot stratejisi "aggressive"
WHEN: Oyuncu "Şahin" seçer
  AND: Bot "Şahin" seçer (%80 ihtimal)
THEN: Her ikisi -2 puan alır
  AND: "Çarpışma!" animasyonu gösterilir
```

#### Senaryo 2: Şahin Zaferi

```
GIVEN: Yeni oyun
  AND: Bot stratejisi "peaceful"
WHEN: Oyuncu "Şahin" seçer
  AND: Bot "Güvercin" seçer
THEN: Oyuncu 4 puan alır
  AND: Bot 0 puan alır
  AND: "Şahin kazandı!" mesajı
```

#### Senaryo 3: Barışçıl Paylaşım

```
GIVEN: Yeni oyun
WHEN: Her iki oyuncu "Güvercin" seçer
THEN: Her ikisi 2 puan alır
  AND: "Kaynağı paylaştınız" mesajı
```

---

## 11. METRİKLER

### 11.1 Analytics Events

| Event | Tetikleyici | Payload |
|-------|-------------|---------|
| `game:hawk-dove:start` | Oyun başı | `{ botStrategy }` |
| `game:hawk-dove:choice` | Seçim | `{ choice }` |
| `game:hawk-dove:complete` | Oyun sonu | `{ outcome, scores }` |
| `game:hawk-dove:collision` | Çarpışma olunca | `{}` |

---

## 12. ACHIEVEMENTS

| ID | İsim | Açıklama | Koşul |
|----|------|----------|-------|
| `hd-first-win` | İlk Zafer | Şahin olarak kazan | Hawk vs Dove sonucu |
| `hd-pacifist` | Barış Sever | 5 kez üst üste güvercin ol | 5 ardışık dove |
| `hd-survivor` | Hayatta Kalan | Hiç çarpışmadan 10 oyun | 10 oyun, 0 collision |
| `hd-brawler` | Kavgacı | 3 kez üst üste çarpışma | 3 hawk-hawk |
| `hd-strategist` | Stratejist | Optimal karışık stratejiyle 10 oyun | %30-40 hawk oranı |

---

## 13. i18n KEYS

### 13.1 İngilizce

```json
"hawk-dove": {
  "name": "Hawk-Dove",
  "description": "Aggressive or peaceful?",
  "choices": {
    "hawk": "Hawk",
    "dove": "Dove"
  },
  "outcomes": {
    "collision": "Collision! Both hawks got injured.",
    "hawk_wins": "Hawk wins! Dove retreated.",
    "sharing": "Peace! Doves shared the resource."
  },
  "warnings": {
    "collision_risk": "Warning: If both choose Hawk, you both lose!"
  }
}
```

### 13.2 Türkçe

```json
"hawk-dove": {
  "name": "Şahin-Güvercin",
  "description": "Saldırgan mı, barışçıl mı?",
  "choices": {
    "hawk": "Şahin",
    "dove": "Güvercin"
  },
  "outcomes": {
    "collision": "Çarpışma! İki şahin de yaralandı.",
    "hawk_wins": "Şahin kazandı! Güvercin geri çekildi.",
    "sharing": "Barış! Güvercinler kaynağı paylaştı."
  },
  "warnings": {
    "collision_risk": "Uyarı: İkisi de Şahin seçerse, ikisi de kaybeder!"
  }
}
```

---

## 14. BAĞIMLILIKLAR

### 14.1 Oyun-Özel Kaynaklar

| Kaynak | Path |
|--------|------|
| İkon | `/assets/icons/games/hawk-dove.svg` |
| Şahin Sprite | `/assets/images/games/hawk.svg` |
| Güvercin Sprite | `/assets/images/games/dove.svg` |
| Çarpışma Efekti | `/assets/images/games/collision.svg` |

---

## 15. NOTLAR

### 15.1 Tasarım Kararları

| Karar | Gerekçe |
|-------|---------|
| Premium tier | Negatif skor var, biraz karmaşık |
| Negatif puan | Gerçekçi, çarpışmanın maliyetini gösterir |
| Hayvan teması | "Chicken" yerine daha evrensel |

### 15.2 Diğer Oyunlarla Karşılaştırma

| Oyun | En İyi Sonuç | Nash Dengesi |
|------|--------------|--------------|
| PD | Mutual Coop | Mutual Defect |
| Stag Hunt | Stag-Stag | Stag-Stag veya Rabbit-Rabbit |
| Hawk-Dove | Dove-Dove? | Hawk-Dove veya Dove-Hawk |

### 15.3 Gelecek İyileştirmeler

- [ ] "Chicken" alternatif tema (araba yarışı)
- [ ] Kaynak değeri ayarlanabilir (V parametresi)
- [ ] Iterated versiyonu
- [ ] Population simulation

### 15.4 Referanslar

- Maynard Smith, J. - "Evolution and the Theory of Games"
- Chicken Game - Cuban Missile Crisis analizi
- Evolutionarily Stable Strategy (ESS)

---

## 16. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |
