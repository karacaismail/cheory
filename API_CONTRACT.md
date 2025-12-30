# API CONTRACT
## Technical Specification Document

| Field | Value |
|-------|-------|
| Document ID | `API_CONTRACT` |
| Version | 1.0.0 |
| Tier | 3 - Technical |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Purpose

This document defines internal JavaScript APIs, event contracts, and module interfaces for the Cheory platform. V1 is client-only; no external REST API.

### 1.2 API Categories

| Category | Description |
|----------|-------------|
| Game Engine API | Core game operations |
| Store API | State management |
| Event API | Application-wide events |
| Module API | Game module interface |
| Storage API | Data persistence |
| UI API | Component interactions |

---

## 2. GAME ENGINE API

### 2.1 Core Functions

```typescript
// /core/engine/index.js

interface GameEngine {
  // Lifecycle
  createGame(moduleId: string, options: GameOptions): GameState
  loadGame(gameId: string): GameState | null
  saveGame(state: GameState): void
  deleteGame(gameId: string): void
  
  // Actions
  dispatch(state: GameState, action: Action): GameState
  getAvailableActions(state: GameState, playerId: string): Action[]
  
  // Queries
  isGameOver(state: GameState): boolean
  getWinner(state: GameState): string | null
  getScores(state: GameState): Record<string, number>
  
  // Bots
  getBotAction(state: GameState, playerId: string): Action
}

interface GameOptions {
  players: PlayerConfig[]
  settings?: Record<string, any>
}

interface PlayerConfig {
  type: 'human' | 'bot'
  name?: string
  strategyId?: string
}
```

### 2.2 Usage Example

```javascript
import { GameEngine } from '/core/engine/index.js'
import { prisonersDilemma } from '/games/prisoners-dilemma/index.js'

// Register game module
GameEngine.register(prisonersDilemma)

// Create new game
const state = GameEngine.createGame('prisoners-dilemma', {
  players: [
    { type: 'human', name: 'Player 1' },
    { type: 'bot', name: 'Bot', strategyId: 'tit-for-tat' }
  ]
})

// Dispatch action
const newState = GameEngine.dispatch(state, {
  type: 'CHOOSE',
  playerId: 'player-1',
  payload: { choice: 'cooperate' }
})

// Check result
if (GameEngine.isGameOver(newState)) {
  const winner = GameEngine.getWinner(newState)
  console.log(`Winner: ${winner}`)
}
```

---

## 3. STORE API

### 3.1 State Management

```typescript
// /core/store/index.js

interface Store<T> {
  // Read
  getState(): T
  subscribe(listener: (state: T) => void): () => void
  
  // Write
  dispatch(action: Action): void
  
  // Selectors
  select<R>(selector: (state: T) => R): R
}

// Application Store
interface AppState {
  user: UserProfile
  currentGame: GameState | null
  ui: UIState
  settings: AppSettings
}

interface UIState {
  currentView: string
  isLoading: boolean
  error: string | null
  modalOpen: string | null
  toasts: Toast[]
}
```

### 3.2 Store Implementation

```javascript
// /core/store/createStore.js

export function createStore(initialState, reducer) {
  let state = initialState
  const listeners = new Set()
  
  return {
    getState() {
      return state
    },
    
    dispatch(action) {
      state = reducer(state, action)
      listeners.forEach(listener => listener(state))
      
      // Persist to storage
      this.persist()
    },
    
    subscribe(listener) {
      listeners.add(listener)
      return () => listeners.delete(listener)
    },
    
    select(selector) {
      return selector(state)
    },
    
    persist() {
      localStorage.setItem('cheory:state', JSON.stringify({
        user: state.user,
        settings: state.settings
      }))
    },
    
    hydrate() {
      const saved = localStorage.getItem('cheory:state')
      if (saved) {
        const parsed = JSON.parse(saved)
        state = { ...state, ...parsed }
      }
    }
  }
}
```

### 3.3 Action Types

```typescript
// Action type constants
const ActionTypes = {
  // User
  USER_UPDATE: 'USER_UPDATE',
  USER_LEVEL_UP: 'USER_LEVEL_UP',
  
  // Game
  GAME_CREATE: 'GAME_CREATE',
  GAME_ACTION: 'GAME_ACTION',
  GAME_END: 'GAME_END',
  GAME_RESET: 'GAME_RESET',
  
  // UI
  UI_SET_VIEW: 'UI_SET_VIEW',
  UI_SET_LOADING: 'UI_SET_LOADING',
  UI_SET_ERROR: 'UI_SET_ERROR',
  UI_OPEN_MODAL: 'UI_OPEN_MODAL',
  UI_CLOSE_MODAL: 'UI_CLOSE_MODAL',
  UI_SHOW_TOAST: 'UI_SHOW_TOAST',
  
  // Settings
  SETTINGS_UPDATE: 'SETTINGS_UPDATE'
}
```

---

## 4. EVENT API

### 4.1 Event Bus

```javascript
// /core/events/index.js

class EventBus {
  constructor() {
    this.listeners = new Map()
  }
  
  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set())
    }
    this.listeners.get(event).add(callback)
    
    // Return unsubscribe function
    return () => this.off(event, callback)
  }
  
  off(event, callback) {
    this.listeners.get(event)?.delete(callback)
  }
  
  emit(event, data = {}) {
    const eventData = {
      type: event,
      timestamp: Date.now(),
      ...data
    }
    
    this.listeners.get(event)?.forEach(cb => {
      try {
        cb(eventData)
      } catch (error) {
        console.error(`Event handler error for ${event}:`, error)
      }
    })
    
    // Also emit to wildcard listeners
    this.listeners.get('*')?.forEach(cb => cb(eventData))
  }
  
  once(event, callback) {
    const unsubscribe = this.on(event, (data) => {
      callback(data)
      unsubscribe()
    })
    return unsubscribe
  }
}

export const events = new EventBus()
```

### 4.2 Event Catalog

```typescript
// Game Events
'game:created'      // { gameId, moduleId }
'game:started'      // { gameId }
'game:action'       // { gameId, action, newState }
'game:round:end'    // { gameId, round, payoffs }
'game:finished'     // { gameId, winner, scores }
'game:paused'       // { gameId }
'game:resumed'      // { gameId }

// User Events
'user:updated'      // { user }
'user:level:up'     // { oldLevel, newLevel }
'user:xp:gained'    // { amount, total }

// Achievement Events
'achievement:unlocked'  // { achievementId }
'achievement:progress'  // { achievementId, progress }

// UI Events
'ui:view:changed'   // { from, to }
'ui:modal:opened'   // { modalId }
'ui:modal:closed'   // { modalId }
'ui:toast:shown'    // { message, type }

// Analytics Events
'analytics:page:view'   // { path }
'analytics:game:start'  // { gameId, moduleId }
'analytics:game:end'    // { gameId, duration, outcome }
```

### 4.3 Usage Example

```javascript
import { events } from '/core/events/index.js'

// Subscribe to game events
events.on('game:finished', ({ gameId, winner, scores }) => {
  console.log(`Game ${gameId} finished. Winner: ${winner}`)
  
  // Update statistics
  updateStats(gameId, scores)
  
  // Check achievements
  checkAchievements(gameId, scores)
})

// Emit event
events.emit('game:action', {
  gameId: 'game-123',
  action: { type: 'CHOOSE', payload: { choice: 'cooperate' } },
  newState: state
})

// Listen to all events (for debugging)
events.on('*', (event) => {
  console.log('[Event]', event.type, event)
})
```

---

## 5. MODULE API

### 5.1 Game Module Interface

```typescript
// Required exports from each game module
interface GameModule {
  // Metadata
  id: string
  metadata: GameMetadata
  
  // Engine
  engine: {
    createInitialState(options: GameOptions): GameData
    getAvailableActions(state: GameState, playerId: string): Action[]
    validateAction(state: GameState, action: Action): ValidationResult
    executeAction(state: GameState, action: Action): GameState
    isGameOver(state: GameState): boolean
    getWinner(state: GameState): string | null
    calculatePayoff(state: GameState): Record<string, number>
  }
  
  // View
  view: {
    components: WebComponentDefinition[]
    renderGame(state: GameState): void
  }
  
  // Bot Strategies
  strategies: BotStrategies
  
  // Tutorial
  tutorial: TutorialConfig
}
```

### 5.2 Module Registration

```javascript
// /core/modules/registry.js

class ModuleRegistry {
  constructor() {
    this.modules = new Map()
  }
  
  register(module) {
    // Validate module structure
    this.validate(module)
    
    // Register components
    module.view.components.forEach(comp => {
      if (!customElements.get(comp.tag)) {
        customElements.define(comp.tag, comp.class)
      }
    })
    
    // Store module
    this.modules.set(module.id, module)
    
    console.log(`Module registered: ${module.id}`)
  }
  
  get(moduleId) {
    return this.modules.get(moduleId)
  }
  
  getAll() {
    return Array.from(this.modules.values())
  }
  
  validate(module) {
    const required = ['id', 'metadata', 'engine', 'view', 'strategies']
    for (const field of required) {
      if (!module[field]) {
        throw new Error(`Module missing required field: ${field}`)
      }
    }
  }
}

export const registry = new ModuleRegistry()
```

### 5.3 Module Example

```javascript
// /games/prisoners-dilemma/index.js

import { engine } from './engine.js'
import { view } from './view.js'
import { strategies } from './strategies.js'
import { tutorial } from './tutorial.js'

export const prisonersDilemma = {
  id: 'prisoners-dilemma',
  
  metadata: {
    name: { en: "Prisoner's Dilemma", tr: "Mahkum İkilemi" },
    description: { en: "Classic cooperation vs betrayal", tr: "Klasik işbirliği vs ihanet" },
    category: 'classic',
    difficulty: 'easy',
    playerCount: { min: 2, max: 2 },
    tier: 'free',
    tags: ['classic', 'cooperation', 'trust'],
    estimatedTime: '2 min'
  },
  
  engine,
  view,
  strategies,
  tutorial
}
```

---

## 6. STORAGE API

### 6.1 Storage Interface

```typescript
// /core/storage/index.js

interface StorageAPI {
  // LocalStorage (simple data)
  local: {
    get<T>(key: string): T | null
    set<T>(key: string, value: T): void
    remove(key: string): void
    clear(): void
  }
  
  // IndexedDB (complex queries)
  db: {
    games: GameStore
    statistics: StatisticsStore
  }
}

interface GameStore {
  add(game: GameRecord): Promise<void>
  get(id: string): Promise<GameRecord | null>
  getByModule(moduleId: string): Promise<GameRecord[]>
  getRecent(limit: number): Promise<GameRecord[]>
  delete(id: string): Promise<void>
  clear(): Promise<void>
}
```

### 6.2 Implementation

```javascript
// /core/storage/local.js

const PREFIX = 'cheory:'

export const local = {
  get(key) {
    try {
      const data = localStorage.getItem(PREFIX + key)
      return data ? JSON.parse(data) : null
    } catch {
      return null
    }
  },
  
  set(key, value) {
    try {
      localStorage.setItem(PREFIX + key, JSON.stringify(value))
    } catch (e) {
      console.error('Storage set error:', e)
    }
  },
  
  remove(key) {
    localStorage.removeItem(PREFIX + key)
  },
  
  clear() {
    Object.keys(localStorage)
      .filter(k => k.startsWith(PREFIX))
      .forEach(k => localStorage.removeItem(k))
  }
}
```

---

## 7. UI API

### 7.1 Router

```javascript
// /core/router/index.js

class Router {
  constructor() {
    this.routes = new Map()
    this.currentRoute = null
    
    window.addEventListener('popstate', () => this.handleRoute())
  }
  
  register(path, handler) {
    this.routes.set(path, handler)
  }
  
  navigate(path, state = {}) {
    history.pushState(state, '', path)
    this.handleRoute()
  }
  
  handleRoute() {
    const path = window.location.pathname
    const handler = this.routes.get(path) || this.routes.get('*')
    
    if (handler) {
      this.currentRoute = path
      handler({ path, params: this.getParams() })
      
      events.emit('ui:view:changed', { path })
    }
  }
  
  getParams() {
    return Object.fromEntries(new URLSearchParams(window.location.search))
  }
}

export const router = new Router()
```

### 7.2 Route Definitions

```javascript
// /app/routes.js

import { router } from '/core/router/index.js'

router.register('/', () => renderHome())
router.register('/games', () => renderGameList())
router.register('/games/:id', ({ params }) => renderGame(params.id))
router.register('/stats', () => renderStats())
router.register('/settings', () => renderSettings())
router.register('/achievements', () => renderAchievements())
router.register('*', () => render404())
```

### 7.3 Toast API

```javascript
// /core/ui/toast.js

export function showToast(message, options = {}) {
  const {
    type = 'info',
    duration = 3000,
    action = null
  } = options
  
  const toast = document.createElement('ch-toast')
  toast.setAttribute('message', message)
  toast.setAttribute('type', type)
  toast.setAttribute('duration', duration)
  
  getToastContainer().appendChild(toast)
  
  events.emit('ui:toast:shown', { message, type })
  
  return toast
}

function getToastContainer() {
  let container = document.querySelector('ch-toast-container')
  if (!container) {
    container = document.createElement('ch-toast-container')
    document.body.appendChild(container)
  }
  return container
}

// Convenience methods
export const toast = {
  success: (msg) => showToast(msg, { type: 'success' }),
  error: (msg) => showToast(msg, { type: 'error' }),
  warning: (msg) => showToast(msg, { type: 'warning' }),
  info: (msg) => showToast(msg, { type: 'info' })
}
```

### 7.4 Modal API

```javascript
// /core/ui/modal.js

let activeModal = null

export function openModal(id, props = {}) {
  closeModal() // Close any open modal
  
  const modal = document.createElement('ch-modal')
  modal.setAttribute('open', '')
  modal.id = id
  
  Object.entries(props).forEach(([key, value]) => {
    if (key === 'content') {
      modal.innerHTML = value
    } else {
      modal.setAttribute(key, value)
    }
  })
  
  modal.addEventListener('close', () => closeModal())
  
  document.body.appendChild(modal)
  activeModal = modal
  
  events.emit('ui:modal:opened', { modalId: id })
  
  return modal
}

export function closeModal() {
  if (activeModal) {
    const id = activeModal.id
    activeModal.remove()
    activeModal = null
    events.emit('ui:modal:closed', { modalId: id })
  }
}
```

---

## 8. INTEGRATION EXAMPLE

```javascript
// Full integration example
import { store } from '/core/store/index.js'
import { events } from '/core/events/index.js'
import { registry } from '/core/modules/registry.js'
import { router } from '/core/router/index.js'
import { toast } from '/core/ui/toast.js'

// Initialize app
async function initApp() {
  // Hydrate state
  store.hydrate()
  
  // Register game modules
  const modules = await import('/games/index.js')
  Object.values(modules).forEach(m => registry.register(m))
  
  // Setup routes
  await import('/app/routes.js')
  
  // Setup event listeners
  events.on('game:finished', handleGameFinished)
  events.on('achievement:unlocked', handleAchievement)
  
  // Start router
  router.handleRoute()
  
  console.log('Cheory initialized')
}

function handleGameFinished({ winner, scores }) {
  if (winner === 'player-1') {
    toast.success('You won!')
  } else {
    toast.info('Game over!')
  }
}

function handleAchievement({ achievementId }) {
  toast.success(`Achievement unlocked: ${achievementId}`)
}

// Start
initApp()
```

---

## 9. ERROR HANDLING

```javascript
// /core/errors/index.js

class GameError extends Error {
  constructor(code, message, details = {}) {
    super(message)
    this.name = 'GameError'
    this.code = code
    this.details = details
  }
}

const ErrorCodes = {
  INVALID_ACTION: 'INVALID_ACTION',
  INVALID_STATE: 'INVALID_STATE',
  MODULE_NOT_FOUND: 'MODULE_NOT_FOUND',
  STORAGE_ERROR: 'STORAGE_ERROR',
  VALIDATION_ERROR: 'VALIDATION_ERROR'
}

function handleError(error) {
  console.error('[Error]', error)
  
  if (error instanceof GameError) {
    toast.error(error.message)
    events.emit('error', { code: error.code, message: error.message })
  } else {
    toast.error('An unexpected error occurred')
    events.emit('error', { code: 'UNKNOWN', message: error.message })
  }
}

export { GameError, ErrorCodes, handleError }
```

---

## 10. REFERENCES

- Custom Events: MDN Web Docs
- IndexedDB API: MDN Web Docs
- History API: MDN Web Docs
