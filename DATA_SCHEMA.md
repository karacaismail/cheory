# DATA SCHEMA
## Technical Specification Document

| Field | Value |
|-------|-------|
| Document ID | `DATA_SCHEMA` |
| Version | 1.0.0 |
| Tier | 3 - Technical |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Purpose

This document defines all data structures, storage mechanisms, and data flow patterns for the Cheory platform. All data is stored client-side using browser APIs (localStorage, IndexedDB).

### 1.2 Storage Strategy

```
PRIMARY: localStorage (simple key-value)
SECONDARY: IndexedDB (complex queries, large data)
BACKUP: Export/Import JSON files

No server-side storage in V1.
```

### 1.3 Data Categories

| Category | Storage | Description |
|----------|---------|-------------|
| User Profile | localStorage | Preferences, settings |
| Game State | localStorage | Current game in progress |
| Game History | IndexedDB | Past game records |
| Statistics | IndexedDB | Aggregated metrics |
| Achievements | localStorage | Unlocked achievements |
| Settings | localStorage | App configuration |

---

## 2. CORE DATA MODELS

### 2.1 User Profile

```typescript
interface UserProfile {
  id: string                    // UUID, generated on first visit
  createdAt: number             // Unix timestamp
  updatedAt: number             // Unix timestamp
  
  // Display
  displayName: string           // "Player" default
  avatar: string                // Avatar ID or URL
  
  // Preferences
  language: 'en' | 'tr'         // Default: 'en'
  theme: 'dark' | 'light' | 'system'
  
  // Progression
  level: number                 // 1-100
  xp: number                    // Experience points
  tier: 'free' | 'premium'      // Subscription status
  
  // Flags
  tutorialCompleted: boolean
  onboardingCompleted: boolean
  analyticsConsent: boolean
}
```

**Storage Key:** `cheory:user`

### 2.2 Game State

```typescript
interface GameState {
  gameId: string                // Unique game instance ID
  moduleId: string              // e.g., "prisoners-dilemma"
  
  // Players
  players: Player[]
  currentPlayerId: string | null
  
  // Game Progress
  phase: GamePhase
  round: number
  maxRounds: number
  
  // Module-Specific Data
  data: Record<string, any>     // Defined per game module
  
  // History
  history: RoundHistory[]
  
  // Result (when finished)
  result: GameResult | null
  
  // Timestamps
  createdAt: number
  updatedAt: number
  pausedAt: number | null
}

type GamePhase = 
  | 'setup'
  | 'playing'
  | 'waiting'
  | 'reveal'
  | 'finished'
  | 'paused'

interface Player {
  id: string
  type: 'human' | 'bot'
  name: string
  score: number
  
  // Bot-specific
  strategyId?: string
  difficulty?: 'easy' | 'medium' | 'hard'
  
  // Stats for this game
  stats?: PlayerGameStats
}

interface PlayerGameStats {
  wins: number
  losses: number
  draws: number
  totalScore: number
}
```

**Storage Key:** `cheory:game:current`

### 2.3 Game History Record

```typescript
interface GameRecord {
  id: string                    // Same as gameId
  moduleId: string
  
  // Summary
  playedAt: number              // When game ended
  duration: number              // Seconds
  roundsPlayed: number
  
  // Players
  players: PlayerSummary[]
  
  // Outcome
  winnerId: string | null
  finalScores: Record<string, number>
  
  // Analytics
  metrics: GameMetrics
}

interface PlayerSummary {
  id: string
  type: 'human' | 'bot'
  name: string
  finalScore: number
  strategyId?: string
}

interface GameMetrics {
  // Game-specific metrics
  cooperationRate?: number      // For PD-type games
  acceptanceRate?: number       // For bargaining games
  averageOffer?: number         // For ultimatum/dictator
  // ... other game-specific metrics
}
```

**Storage:** IndexedDB `games` object store

### 2.4 Round History

```typescript
interface RoundHistory {
  round: number
  
  // Actions taken
  actions: PlayerAction[]
  
  // Outcomes
  payoff: Record<string, number>
  
  // Timing
  startedAt: number
  endedAt: number
}

interface PlayerAction {
  playerId: string
  type: string                  // e.g., "CHOOSE", "PROPOSE", "RESPOND"
  payload: Record<string, any>
  timestamp: number
}
```

### 2.5 Statistics

```typescript
interface UserStatistics {
  // Overall
  totalGamesPlayed: number
  totalTimePlayed: number       // Seconds
  
  // By Game
  byGame: Record<string, GameStatistics>
  
  // Streaks
  currentWinStreak: number
  longestWinStreak: number
  currentPlayStreak: number     // Days
  longestPlayStreak: number
  
  // Last played
  lastPlayedAt: number
  lastPlayedGameId: string
}

interface GameStatistics {
  moduleId: string
  
  // Counts
  gamesPlayed: number
  gamesWon: number
  gamesLost: number
  gamesDraw: number
  
  // Scores
  totalScore: number
  highScore: number
  averageScore: number
  
  // Game-specific
  metrics: Record<string, number>
  
  // Time
  totalTimePlayed: number
  averageGameDuration: number
  
  // Last played
  lastPlayedAt: number
}
```

**Storage:** IndexedDB `statistics` object store

### 2.6 Achievements

```typescript
interface Achievement {
  id: string
  unlockedAt: number
  progress?: number             // For progressive achievements
}

interface AchievementProgress {
  achievementId: string
  currentValue: number
  targetValue: number
  isComplete: boolean
}

// Storage format
interface AchievementsData {
  unlocked: Achievement[]
  progress: Record<string, AchievementProgress>
}
```

**Storage Key:** `cheory:achievements`

### 2.7 Settings

```typescript
interface AppSettings {
  // Audio
  soundEnabled: boolean
  soundVolume: number           // 0-100
  musicEnabled: boolean
  musicVolume: number           // 0-100
  
  // Display
  animationsEnabled: boolean
  reducedMotion: boolean
  fontSize: 'small' | 'medium' | 'large'
  
  // Gameplay
  confirmActions: boolean
  showHints: boolean
  autoAdvance: boolean
  turnTimer: number | null      // Seconds, null = disabled
  
  // Privacy
  analyticsEnabled: boolean
  crashReportsEnabled: boolean
}
```

**Storage Key:** `cheory:settings`

---

## 3. GAME MODULE DATA

### 3.1 Prisoner's Dilemma

```typescript
interface PDGameData {
  player1Choice: 'cooperate' | 'defect' | null
  player2Choice: 'cooperate' | 'defect' | null
  showPayoffMatrix: boolean
  timeLimit: number | null
  outcome: {
    type: 'mutual_cooperation' | 'mutual_defection' | 'exploitation'
    exploiter: string | null
  } | null
}
```

### 3.2 Iterated PD

```typescript
interface IPDGameData {
  totalRounds: number
  currentRound: {
    player1Choice: 'cooperate' | 'defect' | null
    player2Choice: 'cooperate' | 'defect' | null
  }
  stats: {
    player1: IPDPlayerStats
    player2: IPDPlayerStats
  }
}

interface IPDPlayerStats {
  cooperateCount: number
  defectCount: number
  timesExploited: number
  timesExploiting: number
}
```

### 3.3 Ultimatum / Dictator

```typescript
interface UltimatumGameData {
  pot: number
  proposerId: string
  responderId: string
  offer: number | null
  response: 'accept' | 'reject' | null
}

interface DictatorGameData {
  pot: number
  dictatorId: string
  receiverId: string
  gift: number | null
}
```

### 3.4 Trust Game

```typescript
interface TrustGameData {
  endowment: number
  multiplier: number
  investorId: string
  trusteeId: string
  amountSent: number | null
  amountReturned: number | null
}
```

### 3.5 Public Goods

```typescript
interface PublicGoodsData {
  endowment: number
  multiplier: number
  contributions: Record<string, number | null>
  totalContributed: number | null
  pool: number | null
  sharePerPlayer: number | null
}
```

### 3.6 Voting Lab

```typescript
interface VotingLabData {
  candidates: string[]
  votingSystem: VotingSystem
  votes: Record<string, Vote>
  results: VotingResult | null
}

type VotingSystem = 
  | 'plurality'
  | 'ranked'
  | 'borda'
  | 'approval'
  | 'condorcet'

type Vote = 
  | { type: 'plurality', choice: string }
  | { type: 'ranked', ranking: string[] }
  | { type: 'approval', approved: string[] }
```

---

## 4. STORAGE IMPLEMENTATION

### 4.1 localStorage Wrapper

```typescript
const Storage = {
  prefix: 'cheory:',
  
  get<T>(key: string): T | null {
    try {
      const data = localStorage.getItem(this.prefix + key)
      return data ? JSON.parse(data) : null
    } catch {
      return null
    }
  },
  
  set<T>(key: string, value: T): boolean {
    try {
      localStorage.setItem(this.prefix + key, JSON.stringify(value))
      return true
    } catch {
      return false
    }
  },
  
  remove(key: string): void {
    localStorage.removeItem(this.prefix + key)
  },
  
  clear(): void {
    Object.keys(localStorage)
      .filter(k => k.startsWith(this.prefix))
      .forEach(k => localStorage.removeItem(k))
  }
}
```

### 4.2 IndexedDB Schema

```typescript
const DB_NAME = 'cheory'
const DB_VERSION = 1

const STORES = {
  games: {
    keyPath: 'id',
    indexes: [
      { name: 'moduleId', keyPath: 'moduleId' },
      { name: 'playedAt', keyPath: 'playedAt' }
    ]
  },
  statistics: {
    keyPath: 'moduleId'
  }
}

// Initialization
function initDB(): Promise<IDBDatabase> {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open(DB_NAME, DB_VERSION)
    
    request.onerror = () => reject(request.error)
    request.onsuccess = () => resolve(request.result)
    
    request.onupgradeneeded = (event) => {
      const db = request.result
      
      for (const [name, config] of Object.entries(STORES)) {
        if (!db.objectStoreNames.contains(name)) {
          const store = db.createObjectStore(name, { 
            keyPath: config.keyPath 
          })
          
          config.indexes?.forEach(idx => {
            store.createIndex(idx.name, idx.keyPath)
          })
        }
      }
    }
  })
}
```

### 4.3 Data Access Layer

```typescript
const GameDB = {
  async saveGame(record: GameRecord): Promise<void> {
    const db = await initDB()
    const tx = db.transaction('games', 'readwrite')
    tx.objectStore('games').put(record)
    await tx.complete
  },
  
  async getGame(id: string): Promise<GameRecord | null> {
    const db = await initDB()
    return db.transaction('games').objectStore('games').get(id)
  },
  
  async getGamesByModule(moduleId: string): Promise<GameRecord[]> {
    const db = await initDB()
    const index = db.transaction('games')
      .objectStore('games')
      .index('moduleId')
    return index.getAll(moduleId)
  },
  
  async getRecentGames(limit: number = 10): Promise<GameRecord[]> {
    const db = await initDB()
    const all = await db.transaction('games')
      .objectStore('games')
      .index('playedAt')
      .getAll()
    return all.slice(-limit).reverse()
  }
}
```

---

## 5. DATA VALIDATION

### 5.1 Schema Validation

```typescript
import { z } from 'zod'  // Or custom validation

const PlayerSchema = z.object({
  id: z.string(),
  type: z.enum(['human', 'bot']),
  name: z.string(),
  score: z.number()
})

const GameStateSchema = z.object({
  gameId: z.string(),
  moduleId: z.string(),
  players: z.array(PlayerSchema),
  phase: z.enum(['setup', 'playing', 'waiting', 'reveal', 'finished', 'paused']),
  round: z.number().min(1),
  maxRounds: z.number().min(1),
  data: z.record(z.any()),
  history: z.array(z.any()),
  result: z.any().nullable(),
  createdAt: z.number(),
  updatedAt: z.number()
})

function validateGameState(data: unknown): GameState {
  return GameStateSchema.parse(data)
}
```

### 5.2 Migration Support

```typescript
interface Migration {
  version: number
  up: (data: any) => any
}

const migrations: Migration[] = [
  {
    version: 2,
    up: (data) => ({
      ...data,
      newField: 'default'
    })
  }
]

function migrateData(data: any, fromVersion: number): any {
  let current = data
  for (const m of migrations) {
    if (m.version > fromVersion) {
      current = m.up(current)
    }
  }
  return current
}
```

---

## 6. DATA EXPORT / IMPORT

### 6.1 Export Format

```typescript
interface ExportData {
  version: string
  exportedAt: number
  
  user: UserProfile
  settings: AppSettings
  achievements: AchievementsData
  statistics: UserStatistics
  games: GameRecord[]
}

function exportAllData(): ExportData {
  return {
    version: '1.0.0',
    exportedAt: Date.now(),
    user: Storage.get('user'),
    settings: Storage.get('settings'),
    achievements: Storage.get('achievements'),
    statistics: /* from IndexedDB */,
    games: /* from IndexedDB */
  }
}
```

### 6.2 Import Validation

```typescript
async function importData(json: string): Promise<boolean> {
  try {
    const data = JSON.parse(json) as ExportData
    
    // Validate version compatibility
    if (!isCompatibleVersion(data.version)) {
      throw new Error('Incompatible version')
    }
    
    // Validate structure
    validateExportData(data)
    
    // Import to storage
    Storage.set('user', data.user)
    Storage.set('settings', data.settings)
    Storage.set('achievements', data.achievements)
    
    // Import to IndexedDB
    for (const game of data.games) {
      await GameDB.saveGame(game)
    }
    
    return true
  } catch (error) {
    console.error('Import failed:', error)
    return false
  }
}
```

---

## 7. DATA LIFECYCLE

### 7.1 Creation

```
User visits → Generate UUID → Create UserProfile
Start game → Create GameState → Save to localStorage
Finish game → Create GameRecord → Save to IndexedDB
```

### 7.2 Updates

```
Action taken → Update GameState → Save immediately
Settings change → Update Settings → Save immediately
Achievement unlock → Update Achievements → Save immediately
```

### 7.3 Cleanup

```typescript
// Remove old game records (keep last 100)
async function cleanupOldGames(): Promise<void> {
  const games = await GameDB.getAllGames()
  if (games.length > 100) {
    const toDelete = games.slice(0, games.length - 100)
    for (const game of toDelete) {
      await GameDB.deleteGame(game.id)
    }
  }
}

// Clear all data (factory reset)
function factoryReset(): void {
  Storage.clear()
  indexedDB.deleteDatabase('cheory')
}
```

---

## 8. STORAGE KEYS REFERENCE

| Key | Type | Description |
|-----|------|-------------|
| `cheory:user` | localStorage | User profile |
| `cheory:settings` | localStorage | App settings |
| `cheory:achievements` | localStorage | Achievement data |
| `cheory:game:current` | localStorage | Active game state |
| `cheory:tutorial:{id}` | localStorage | Tutorial completion flags |
| IndexedDB `games` | IndexedDB | Game history records |
| IndexedDB `statistics` | IndexedDB | Aggregated statistics |

---

## 9. REFERENCES

- localStorage API: MDN Web Docs
- IndexedDB API: MDN Web Docs
- Zod validation library (optional)
