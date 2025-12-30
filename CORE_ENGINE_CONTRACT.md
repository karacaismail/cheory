# CHEORY: Oyun Motoru Sözleşmesi
## CORE_ENGINE_CONTRACT.md

> Bu doküman, tüm oyun modüllerinin uyması gereken interface ve kuralları tanımlar.
> Yeni oyun geliştiren herkes bu sözleşmeye uymalıdır.

---

## 1. GENEL BAKIŞ

### 1.1 Sözleşme Amacı

| Amaç | Açıklama |
|------|----------|
| Tutarlılık | Tüm oyunlar aynı yapıda |
| Değiştirilebilirlik | UI değişebilir, mantık kalır |
| Test Edilebilirlik | Pure functions, kolay test |
| Plug-and-Play | Yeni oyun eklemek kolay |

### 1.2 Temel Prensipler

| Prensip | Açıklama |
|---------|----------|
| Pure Functions | Engine fonksiyonları side-effect'siz |
| Immutable State | State asla mutate edilmez |
| Deterministic | Aynı input = aynı output |
| Separation | Logic ≠ View |

---

## 2. MODÜL YAPISI

### 2.1 Zorunlu Export'lar

Her oyun modülü (`/games/{game-id}/index.js`) şu yapıyı export etmeli:

```
GameModule {
  // METADATA (Zorunlu)
  id: string
  metadata: GameMetadata
  
  // ENGINE (Zorunlu)
  engine: GameEngine
  
  // VIEW (Zorunlu)
  view: GameView
  
  // STRATEGIES (Opsiyonel)
  strategies?: BotStrategies
  
  // TUTORIAL (Opsiyonel)
  tutorial?: TutorialConfig
}
```

### 2.2 Örnek Export

```
// /games/prisoners-dilemma/index.js

export default {
  id: 'prisoners-dilemma',
  metadata: { ... },
  engine: { ... },
  view: { ... },
  strategies: { ... },
  tutorial: { ... }
}
```

---

## 3. METADATA SÖZLEŞMESI

### 3.1 GameMetadata Interface

```
GameMetadata {
  // Kimlik
  id: string                    // Unique, kebab-case
  version: string               // Semantic versioning (1.0.0)
  
  // Görüntüleme
  name: LocalizedString         // { en: "...", tr: "..." }
  description: LocalizedString  // Kısa açıklama
  icon: string                  // SVG path veya asset path
  thumbnail: string             // Kart görseli path
  
  // Kategorizasyon
  category: GameCategory
  tags: string[]                // Arama için
  difficulty: Difficulty
  
  // Oyuncu Bilgisi
  playerCount: PlayerCount
  estimatedTime: string         // "5-10 min"
  
  // Erişim
  tier: Tier                    // Hangi tier'da açık
  isNew: boolean                // "Yeni" rozeti
  isFeatured: boolean           // Öne çıkan
  
  // Teknik
  supportsBot: boolean
  supportsTutorial: boolean
  supportsMultiplayer: boolean  // V2 için
}
```

### 3.2 Alt Tipler

```
LocalizedString {
  en: string
  tr: string
}

GameCategory = 
  | 'classic'           // Klasik oyun teorisi
  | 'bargaining'        // Pazarlık oyunları
  | 'coordination'      // Koordinasyon oyunları
  | 'social'            // Sosyal ikilemler
  | 'zero-sum'          // Sıfır toplamlı
  | 'voting'            // Oylama sistemleri

Difficulty = 'easy' | 'medium' | 'hard'

PlayerCount {
  min: number           // Minimum oyuncu (genelde 2)
  max: number           // Maximum oyuncu
  optimal: number       // Önerilen
}

Tier = 'free' | 'premium'
```

### 3.3 Örnek Metadata

```
metadata: {
  id: 'prisoners-dilemma',
  version: '1.0.0',
  
  name: {
    en: "Prisoner's Dilemma",
    tr: "Mahkum İkilemi"
  },
  description: {
    en: "The classic game of cooperation vs betrayal",
    tr: "İşbirliği ve ihanet arasındaki klasik oyun"
  },
  icon: '/assets/icons/games/pd.svg',
  thumbnail: '/assets/images/games/pd-thumb.svg',
  
  category: 'classic',
  tags: ['cooperation', 'defection', 'trust', 'strategy'],
  difficulty: 'easy',
  
  playerCount: { min: 2, max: 2, optimal: 2 },
  estimatedTime: '5-10 min',
  
  tier: 'free',
  isNew: false,
  isFeatured: true,
  
  supportsBot: true,
  supportsTutorial: true,
  supportsMultiplayer: false
}
```

---

## 4. ENGINE SÖZLEŞMESI

### 4.1 GameEngine Interface

```
GameEngine {
  // STATE OLUŞTURMA
  createInitialState(config: GameConfig): GameState
  
  // EYLEM YÖNETİMİ
  getAvailableActions(state: GameState, playerId: string): Action[]
  validateAction(state: GameState, action: Action): ValidationResult
  executeAction(state: GameState, action: Action): GameState
  
  // OYUN DURUMU
  isGameOver(state: GameState): boolean
  getWinner(state: GameState): string | null | 'draw'
  
  // SKORLAMA
  calculatePayoff(state: GameState): PayoffMatrix
  calculateScore(state: GameState): ScoreBoard
  
  // YARDIMCI
  getGamePhase(state: GameState): GamePhase
  getRoundInfo(state: GameState): RoundInfo
}
```

### 4.2 GameState Interface

Tüm oyunların ortak state yapısı:

```
GameState {
  // Kimlik
  gameId: string              // Unique game instance ID
  moduleId: string            // Oyun modülü ID (prisoners-dilemma)
  
  // Oyuncular
  players: Player[]
  currentPlayerId: string | null
  
  // Zaman
  createdAt: timestamp
  updatedAt: timestamp
  
  // Durum
  phase: GamePhase
  round: number
  maxRounds: number | null    // null = sınırsız
  
  // Oyun-spesifik veri
  data: GameSpecificData      // Her oyun kendi yapısını tanımlar
  
  // Geçmiş
  history: HistoryEntry[]
  
  // Sonuç (oyun bittiyse)
  result: GameResult | null
}
```

### 4.3 Alt Tipler (State)

```
Player {
  id: string
  type: 'human' | 'bot'
  name: string
  strategyId?: string         // Bot ise strateji
  score: number
  stats: PlayerStats
}

PlayerStats {
  wins: number
  losses: number
  draws: number
  totalScore: number
}

GamePhase = 
  | 'setup'         // Oyun ayarları
  | 'playing'       // Oyun devam ediyor
  | 'waiting'       // Diğer oyuncu bekleniyor
  | 'reveal'        // Sonuç gösterimi
  | 'finished'      // Oyun bitti

HistoryEntry {
  round: number
  playerId: string
  action: Action
  timestamp: timestamp
  result?: ActionResult
}

GameResult {
  winnerId: string | null     // null = berabere
  finalScores: { [playerId: string]: number }
  summary: LocalizedString
  achievements?: Achievement[]
}
```

### 4.4 Action Interface

```
Action {
  type: string                // Oyun-spesifik action tipi
  playerId: string
  payload: ActionPayload      // Oyun-spesifik veri
  timestamp: timestamp
}

// Örnek Action'lar:

// Prisoner's Dilemma
PDAction {
  type: 'CHOOSE'
  playerId: 'player-1'
  payload: { choice: 'cooperate' | 'defect' }
  timestamp: 1234567890
}

// Ultimatum Game
UltimatumAction {
  type: 'PROPOSE' | 'RESPOND'
  playerId: 'player-1'
  payload: { 
    amount?: number           // PROPOSE için
    accept?: boolean          // RESPOND için
  }
  timestamp: 1234567890
}
```

### 4.5 Validation Interface

```
ValidationResult {
  isValid: boolean
  errors: ValidationError[]
}

ValidationError {
  code: string                // 'INVALID_TURN', 'INVALID_ACTION', etc.
  message: LocalizedString
  field?: string
}
```

### 4.6 Payoff ve Score Interface

```
PayoffMatrix {
  [playerId: string]: {
    [otherPlayerId: string]: number
  }
}

ScoreBoard {
  players: {
    [playerId: string]: {
      score: number
      rank: number
      delta: number           // Bu turda değişim
    }
  }
  leader: string | null
}

RoundInfo {
  current: number
  total: number | null        // null = sınırsız
  isLastRound: boolean
  turnsRemaining: number
}
```

---

## 5. VIEW SÖZLEŞMESI

### 5.1 GameView Interface

```
GameView {
  // COMPONENT KAYDI
  components: WebComponentDefinition[]
  
  // ANA RENDER
  render(container: HTMLElement, context: RenderContext): void
  
  // GÜNCELLEME
  update(context: RenderContext): void
  
  // TEMİZLİK
  destroy(): void
}
```

### 5.2 RenderContext Interface

```
RenderContext {
  // State
  state: GameState
  previousState: GameState | null
  
  // Actions
  dispatch: (action: Action) => void
  
  // Navigation
  navigate: (path: string) => void
  
  // i18n
  t: (key: string, params?: object) => string
  locale: string
  
  // Theme
  theme: 'dark' | 'light'
  
  // User
  currentUserId: string
  
  // Entitlement
  tier: Tier
  
  // Utilities
  formatNumber: (n: number) => string
  formatDate: (d: Date) => string
}
```

### 5.3 WebComponentDefinition

```
WebComponentDefinition {
  tagName: string             // 'pd-game-board'
  componentClass: class       // PDGameBoard extends HTMLElement
}
```

### 5.4 Component İsimlendirme Kuralı

```
Pattern: {game-prefix}-{component-name}

Örnekler:
- pd-game-board          // Prisoner's Dilemma board
- pd-choice-button       // Seçim butonu
- pd-result-display      // Sonuç gösterimi
- ug-proposal-slider     // Ultimatum Game slider
- vl-ballot-form         // Voting Lab oy pusulası
```

### 5.5 Zorunlu UI Bölgeleri

Her oyun view'ı şu bölgeleri içermeli:

```
┌─────────────────────────────────────────┐
│              GAME HEADER                │
│  (Oyun adı, tur bilgisi, çıkış butonu) │
├─────────────────────────────────────────┤
│                                         │
│             GAME BOARD                  │
│  (Ana oyun alanı, interaktif bölge)    │
│                                         │
├─────────────────────────────────────────┤
│            SCORE PANEL                  │
│  (Oyuncu skorları, sıralama)           │
├─────────────────────────────────────────┤
│           ACTION PANEL                  │
│  (Butonlar, kontroller)                │
└─────────────────────────────────────────┘
```

---

## 6. STRATEGIES SÖZLEŞMESI (Opsiyonel)

### 6.1 BotStrategies Interface

```
BotStrategies {
  // Strateji listesi
  strategies: Strategy[]
  
  // Varsayılan strateji
  defaultStrategyId: string
  
  // Strateji seçimi
  getStrategy(id: string): Strategy
  
  // Zorluk bazlı öneri
  getRecommendedStrategy(difficulty: Difficulty): Strategy
}
```

### 6.2 Strategy Interface

```
Strategy {
  id: string
  name: LocalizedString
  description: LocalizedString
  difficulty: Difficulty
  
  // Karar fonksiyonu (PURE)
  decide(state: GameState, playerId: string): Action
  
  // Açıklama (Tutorial için)
  explainDecision?(state: GameState, action: Action): LocalizedString
}
```

### 6.3 Örnek Stratejiler (Prisoner's Dilemma)

```
strategies: [
  {
    id: 'always-cooperate',
    name: { en: 'Always Cooperate', tr: 'Her Zaman İşbirliği' },
    description: { en: 'Always chooses to cooperate', tr: 'Her zaman işbirliği yapar' },
    difficulty: 'easy',
    decide: (state, playerId) => ({
      type: 'CHOOSE',
      playerId,
      payload: { choice: 'cooperate' },
      timestamp: Date.now()
    })
  },
  {
    id: 'always-defect',
    name: { en: 'Always Defect', tr: 'Her Zaman İhanet' },
    difficulty: 'easy',
    decide: (state, playerId) => ({
      type: 'CHOOSE',
      playerId,
      payload: { choice: 'defect' },
      timestamp: Date.now()
    })
  },
  {
    id: 'tit-for-tat',
    name: { en: 'Tit for Tat', tr: 'Kısasa Kısas' },
    description: { 
      en: 'Cooperates first, then copies opponent', 
      tr: 'Önce işbirliği, sonra rakibi taklit' 
    },
    difficulty: 'medium',
    decide: (state, playerId) => {
      // İlk tur: işbirliği
      // Sonraki turlar: rakibin son hamlesini taklit
      ...
    }
  },
  {
    id: 'random',
    name: { en: 'Random', tr: 'Rastgele' },
    difficulty: 'easy',
    decide: (state, playerId) => ({
      type: 'CHOOSE',
      playerId,
      payload: { choice: Math.random() > 0.5 ? 'cooperate' : 'defect' },
      timestamp: Date.now()
    })
  },
  {
    id: 'grudger',
    name: { en: 'Grudger', tr: 'Kinci' },
    description: {
      en: 'Cooperates until betrayed, then always defects',
      tr: 'İhanete uğrayana kadar işbirliği, sonra hep ihanet'
    },
    difficulty: 'hard',
    decide: (state, playerId) => { ... }
  }
]
```

---

## 7. TUTORIAL SÖZLEŞMESI (Opsiyonel)

### 7.1 TutorialConfig Interface

```
TutorialConfig {
  // Adımlar
  steps: TutorialStep[]
  
  // Ayarlar
  isSkippable: boolean
  showOnFirstPlay: boolean
  
  // İlerleme
  getProgress(completedSteps: string[]): number  // 0-100
}
```

### 7.2 TutorialStep Interface

```
TutorialStep {
  id: string
  order: number
  
  // İçerik
  title: LocalizedString
  content: LocalizedString
  
  // Görsel
  image?: string
  animation?: string
  
  // Hedefleme
  target?: string             // CSS selector
  position?: 'top' | 'bottom' | 'left' | 'right'
  
  // Etkileşim
  action?: TutorialAction
  waitFor?: WaitCondition
  
  // Navigasyon
  nextStepId?: string
  previousStepId?: string
}

TutorialAction = 
  | { type: 'click', target: string }
  | { type: 'input', target: string, value: string }
  | { type: 'wait', duration: number }
  | { type: 'highlight', target: string }

WaitCondition =
  | { type: 'click', target: string }
  | { type: 'stateChange', path: string, value: any }
  | { type: 'timeout', duration: number }
```

---

## 8. YAŞAM DÖNGÜSÜ

### 8.1 Oyun Yaşam Döngüsü

```
┌──────────────────────────────────────────────────────────────┐
│                    GAME LIFECYCLE                             │
└──────────────────────────────────────────────────────────────┘

1. LOAD
   │
   ├── Game Loader modülü yükler
   ├── Metadata okunur
   ├── Entitlement kontrol edilir
   │
   ▼
2. INITIALIZE
   │
   ├── engine.createInitialState(config) çağrılır
   ├── View render edilir
   ├── Event listener'lar bağlanır
   │
   ▼
3. SETUP (phase: 'setup')
   │
   ├── Oyuncu/bot seçimi
   ├── Zorluk ayarı
   ├── Tur sayısı ayarı
   │
   ▼
4. PLAY LOOP (phase: 'playing')
   │
   ├─── getAvailableActions() ile seçenekler göster
   │
   ├─── Kullanıcı action seçer VEYA bot.decide() çağrılır
   │
   ├─── validateAction() ile kontrol
   │
   ├─── executeAction() ile state güncelle
   │
   ├─── View güncelle
   │
   ├─── isGameOver() kontrolü
   │     │
   │     ├── false → PLAY LOOP devam
   │     │
   │     └── true → FINISH'e git
   │
   ▼
5. FINISH (phase: 'finished')
   │
   ├── calculateScore() ile final skor
   ├── getWinner() ile kazanan
   ├── Sonuç ekranı göster
   ├── Achievement kontrol
   ├── Skor kaydet (persist)
   │
   ▼
6. CLEANUP
   │
   ├── Event listener'lar kaldır
   ├── view.destroy() çağır
   ├── Bellek temizle
   │
   ▼
7. RESTART veya EXIT
   │
   ├── RESTART → INITIALIZE'a dön
   │
   └── EXIT → Ana menüye git
```

### 8.2 Tur Döngüsü (Round Loop)

```
┌──────────────────────────────────────────────────────────────┐
│                     ROUND LIFECYCLE                           │
└──────────────────────────────────────────────────────────────┘

ROUND START
    │
    ├── Tur bilgisi güncelle
    ├── UI'da tur göster
    │
    ▼
PLAYER TURNS
    │
    ├── Her oyuncu için (sırayla veya eşzamanlı):
    │   │
    │   ├── getAvailableActions()
    │   ├── Action bekle/al
    │   ├── validateAction()
    │   ├── executeAction()
    │   └── Sonraki oyuncuya geç
    │
    ▼
ROUND END
    │
    ├── calculatePayoff() bu tur için
    ├── Skorları güncelle
    ├── History'ye ekle
    ├── Reveal animasyonu (varsa)
    │
    ▼
CHECK GAME END
    │
    ├── isGameOver() ?
    │   │
    │   ├── true → FINISH phase
    │   └── false → Sonraki ROUND START
```

---

## 9. STATE YÖNETİM KURALLARI

### 9.1 Immutability Kuralı

```
YANLIŞ (Mutate):
function executeAction(state, action) {
  state.round++;                    // ✗ Mutate
  state.players[0].score += 10;     // ✗ Mutate
  return state;
}

DOĞRU (Immutable):
function executeAction(state, action) {
  return {
    ...state,
    round: state.round + 1,
    players: state.players.map((p, i) => 
      i === 0 
        ? { ...p, score: p.score + 10 }
        : p
    )
  };
}
```

### 9.2 Pure Function Kuralı

```
YANLIŞ (Side Effect):
function calculatePayoff(state) {
  const result = compute(state);
  saveToDatabase(result);           // ✗ Side effect
  console.log(result);              // ✗ Side effect
  return result;
}

DOĞRU (Pure):
function calculatePayoff(state) {
  return compute(state);            // ✓ Sadece hesaplama
}
```

### 9.3 Determinism Kuralı

```
YANLIŞ (Non-deterministic):
function createInitialState() {
  return {
    gameId: generateUUID(),         // ✗ Her seferinde farklı
    createdAt: Date.now(),          // ✗ Her seferinde farklı
    ...
  };
}

DOĞRU (Deterministic - config üzerinden):
function createInitialState(config) {
  return {
    gameId: config.gameId,          // ✓ Dışarıdan veriliyor
    createdAt: config.timestamp,    // ✓ Dışarıdan veriliyor
    ...
  };
}
```

---

## 10. PAYOFF HESAPLAMA

### 10.1 Payoff Matrix Yapısı

```
Klasik 2x2 Oyun Matrisi:

                    Player 2
                 Cooperate  Defect
Player 1  Coop    (R, R)    (S, T)
          Defect  (T, S)    (P, P)

Prisoner's Dilemma değerleri:
T > R > P > S
T = 5 (Temptation)
R = 3 (Reward)
P = 1 (Punishment)
S = 0 (Sucker)
```

### 10.2 Payoff Hesaplama Fonksiyonu

```
// Pseudo-kod
function calculatePayoff(state) {
  const { players, data } = state;
  const [p1, p2] = players;
  const { p1Choice, p2Choice } = data.currentRound;
  
  const matrix = {
    'cooperate-cooperate': { p1: 3, p2: 3 },
    'cooperate-defect':    { p1: 0, p2: 5 },
    'defect-cooperate':    { p1: 5, p2: 0 },
    'defect-defect':       { p1: 1, p2: 1 }
  };
  
  const key = `${p1Choice}-${p2Choice}`;
  return matrix[key];
}
```

### 10.3 Oyunlara Göre Payoff Yapıları

| Oyun | Payoff Tipi | Açıklama |
|------|-------------|----------|
| Prisoner's Dilemma | 2x2 Matrix | Klasik matris |
| Stag Hunt | 2x2 Matrix | Koordinasyon matrisi |
| Hawk-Dove | 2x2 Matrix | Anti-koordinasyon |
| Matching Pennies | 2x2 Matrix | Sıfır-toplam |
| Battle of Sexes | 2x2 Matrix | Koordinasyon |
| Ultimatum | Split | Teklif/kabul paylaşımı |
| Dictator | Allocation | Tek taraflı paylaşım |
| Trust | Investment | Yatırım çarpanı |
| Public Goods | Contribution | Havuz çarpanı |
| Voting | Election | Oy sonucu |

---

## 11. EVENT KONTRATI

### 11.1 Game Events

Oyunların emit etmesi gereken event'ler:

```
Event Formatı:
{
  type: string,
  gameId: string,
  timestamp: number,
  payload: object
}
```

### 11.2 Zorunlu Event'ler

| Event | Ne Zaman | Payload |
|-------|----------|---------|
| `game:initialized` | State oluşturulduğunda | `{ gameId, moduleId }` |
| `game:started` | Oyun başladığında | `{ gameId, players }` |
| `game:action` | Her action'da | `{ gameId, action, newState }` |
| `game:roundEnd` | Tur bittiğinde | `{ gameId, round, scores }` |
| `game:finished` | Oyun bittiğinde | `{ gameId, result }` |

### 11.3 Opsiyonel Event'ler

| Event | Ne Zaman | Payload |
|-------|----------|---------|
| `game:tutorialStart` | Tutorial başladığında | `{ gameId, stepId }` |
| `game:tutorialStep` | Tutorial adımında | `{ gameId, stepId }` |
| `game:tutorialEnd` | Tutorial bittiğinde | `{ gameId, completed }` |
| `game:achievementUnlock` | Başarı açıldığında | `{ gameId, achievementId }` |

---

## 12. HATA YÖNETİMİ

### 12.1 Error Types

```
GameError {
  code: GameErrorCode
  message: LocalizedString
  details?: object
  recoverable: boolean
}

GameErrorCode =
  | 'INVALID_STATE'         // Geçersiz state
  | 'INVALID_ACTION'        // Geçersiz action
  | 'NOT_YOUR_TURN'         // Sıra değil
  | 'GAME_OVER'             // Oyun bitti
  | 'PLAYER_NOT_FOUND'      // Oyuncu bulunamadı
  | 'STRATEGY_ERROR'        // Bot hatası
  | 'RENDER_ERROR'          // UI hatası
```

### 12.2 Error Handling Yaklaşımı

```
Validation sırasında:
- validateAction() → ValidationResult döner
- Hata varsa action execute edilmez
- UI'a hata mesajı gösterilir

Execution sırasında:
- Try-catch ile sar
- Hata olursa önceki state'e dön
- Event log'a yaz
- Kullanıcıya bildir

View sırasında:
- Component error boundary
- Fallback UI göster
- Console'a detay yaz
```

---

## 13. TEST KONTRATI

### 13.1 Zorunlu Test Senaryoları

Her oyun engine'i için şu testler yazılmalı:

```
describe('GameEngine', () => {
  
  describe('createInitialState', () => {
    it('should create valid initial state')
    it('should respect config options')
    it('should be deterministic with same config')
  })
  
  describe('getAvailableActions', () => {
    it('should return valid actions for current player')
    it('should return empty for non-current player')
    it('should return empty when game is over')
  })
  
  describe('validateAction', () => {
    it('should accept valid actions')
    it('should reject invalid action type')
    it('should reject wrong player turn')
    it('should reject after game over')
  })
  
  describe('executeAction', () => {
    it('should return new state (not mutate)')
    it('should update current player')
    it('should update scores')
    it('should add to history')
  })
  
  describe('isGameOver', () => {
    it('should return false during game')
    it('should return true when max rounds reached')
    it('should return true when win condition met')
  })
  
  describe('calculatePayoff', () => {
    it('should return correct payoff for each combination')
    it('should handle all player combinations')
  })
  
})
```

### 13.2 Test Helper'lar

```
// Test utilities sağlanacak

createMockState(overrides)     // Test state oluştur
createMockAction(type, payload) // Test action oluştur
createMockPlayer(type)          // Test oyuncu
simulateGame(engine, actions)   // Oyun simüle et
```

---

## 14. PERFORMANS KURALLARI

### 14.1 Engine Performans Hedefleri

| Metrik | Hedef | Maksimum |
|--------|-------|----------|
| createInitialState | < 1ms | 5ms |
| validateAction | < 0.5ms | 2ms |
| executeAction | < 2ms | 10ms |
| calculatePayoff | < 1ms | 5ms |
| isGameOver | < 0.5ms | 2ms |

### 14.2 View Performans Hedefleri

| Metrik | Hedef | Maksimum |
|--------|-------|----------|
| Initial render | < 50ms | 100ms |
| Update (action sonrası) | < 16ms | 32ms |
| Animation | 60fps | 30fps |

### 14.3 Optimizasyon Kuralları

```
DO:
- Memoization kullan (aynı input = cache'den al)
- Early return (gereksiz hesaplamadan kaçın)
- Minimal DOM update (diff algoritması)
- RequestAnimationFrame (animasyonlar için)

DON'T:
- Deep clone yapmaktan kaçın (shallow copy yeterli)
- Loop içinde DOM query yapmaktan kaçın
- Gereksiz re-render'dan kaçın
- Senkron hesaplama bloklamalarından kaçın
```

---

## 15. CHECKLIST

### 15.1 Yeni Oyun Ekleme Checklist

```
METADATA
[ ] id unique ve kebab-case
[ ] name ve description çift dilde
[ ] category doğru seçilmiş
[ ] difficulty uygun
[ ] tier belirlendi (free/premium)

ENGINE
[ ] createInitialState pure ve deterministic
[ ] getAvailableActions tüm durumları kapsar
[ ] validateAction tüm hataları yakalar
[ ] executeAction immutable
[ ] isGameOver tüm bitiş koşullarını kontrol
[ ] calculatePayoff matematiksel olarak doğru

VIEW
[ ] Tüm component'lar registered
[ ] render() ve update() implement
[ ] destroy() cleanup yapar
[ ] Responsive (mobile uyumlu)
[ ] Dark/light mode destekler
[ ] i18n kullanır

STRATEGIES (varsa)
[ ] En az 3 strateji (easy, medium, hard)
[ ] decide() pure function
[ ] Tüm stratejiler test edilmiş

TUTORIAL (varsa)
[ ] En az 5 adım
[ ] Görsel destekli
[ ] Skip edilebilir

TESTS
[ ] Engine unit testleri
[ ] Payoff hesaplama testleri
[ ] Edge case'ler
```

---

## 16. ÖRNEK IMPLEMENTASYON ŞABLONU

```
// /games/example-game/index.js

import engine from './engine.js';
import view from './view.js';
import strategies from './strategies.js';

export default {
  id: 'example-game',
  
  metadata: {
    id: 'example-game',
    version: '1.0.0',
    name: { en: 'Example Game', tr: 'Örnek Oyun' },
    description: { en: '...', tr: '...' },
    icon: '/assets/icons/games/example.svg',
    thumbnail: '/assets/images/games/example-thumb.svg',
    category: 'classic',
    tags: ['example', 'demo'],
    difficulty: 'easy',
    playerCount: { min: 2, max: 2, optimal: 2 },
    estimatedTime: '5 min',
    tier: 'free',
    isNew: true,
    isFeatured: false,
    supportsBot: true,
    supportsTutorial: true,
    supportsMultiplayer: false
  },
  
  engine,
  view,
  strategies
};
```

---

## 17. DOKÜMAN GEÇMİŞİ

| Versiyon | Tarih | Değişiklik |
|----------|-------|------------|
| 1.0 | 2025-01-XX | İlk versiyon |

---

> **Sonraki Adım:** `GAME_SPEC_TEMPLATE.md` - Her oyun için doldurulacak boş şablon
