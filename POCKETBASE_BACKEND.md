# POCKETBASE BACKEND
## SaaS Backend Specification

| Field | Value |
|-------|-------|
| Document ID | `POCKETBASE_BACKEND` |
| Version | 1.0.0 |
| Backend | PocketBase |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Why PocketBase?

| Feature | Benefit |
|---------|---------|
| Single binary | No dependencies, easy deploy |
| Built-in Auth | Email, OAuth, JWT out of box |
| Real-time | WebSocket subscriptions |
| Admin UI | No separate admin panel needed |
| SQLite | Simple backup, portable |
| Free | Open source, self-hosted |

### 1.2 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      CLIENT                              │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Cheory Web App                      │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │    │
│  │  │ Games   │  │ Auth UI │  │ PocketBase SDK  │  │    │
│  │  └─────────┘  └─────────┘  └────────┬────────┘  │    │
│  └─────────────────────────────────────┼───────────┘    │
└────────────────────────────────────────┼────────────────┘
                                         │ HTTPS
┌────────────────────────────────────────┼────────────────┐
│                     SERVER              │                │
│  ┌─────────────────────────────────────▼───────────┐    │
│  │                 PocketBase                       │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │    │
│  │  │   Auth   │  │   API    │  │  Admin UI    │   │    │
│  │  └──────────┘  └──────────┘  └──────────────┘   │    │
│  │  ┌──────────────────────────────────────────┐   │    │
│  │  │              SQLite Database              │   │    │
│  │  └──────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌───────────────────────▼─────────────────────────┐    │
│  │              Stripe Webhooks                     │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### 1.3 Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | PocketBase (Go) |
| Database | SQLite (embedded) |
| Auth | PocketBase Auth + OAuth |
| Payment | Stripe |
| Hosting | Railway / Fly.io / VPS |
| CDN | Cloudflare (optional) |

---

## 2. SETUP & DEPLOYMENT

### 2.1 Local Development

```bash
# Download PocketBase
wget https://github.com/pocketbase/pocketbase/releases/download/v0.22.0/pocketbase_0.22.0_linux_amd64.zip
unzip pocketbase_0.22.0_linux_amd64.zip

# Run
./pocketbase serve

# Admin UI: http://127.0.0.1:8090/_/
# API: http://127.0.0.1:8090/api/
```

### 2.2 Project Structure

```
backend/
├── pocketbase              # Binary
├── pb_data/                # Database & uploads
│   ├── data.db             # SQLite database
│   ├── logs.db             # Logs database
│   └── storage/            # File uploads
├── pb_migrations/          # Schema migrations
│   ├── 1_initial.js
│   ├── 2_subscriptions.js
│   └── 3_game_records.js
├── pb_hooks/               # Custom hooks (JS)
│   ├── auth.pb.js
│   ├── subscriptions.pb.js
│   └── webhooks.pb.js
└── Dockerfile
```

### 2.3 Railway Deployment

```dockerfile
# Dockerfile
FROM alpine:latest

ARG PB_VERSION=0.22.0

RUN apk add --no-cache \
    unzip \
    ca-certificates

ADD https://github.com/pocketbase/pocketbase/releases/download/v${PB_VERSION}/pocketbase_${PB_VERSION}_linux_amd64.zip /tmp/pb.zip
RUN unzip /tmp/pb.zip -d /pb/

# Copy migrations and hooks
COPY ./pb_migrations /pb/pb_migrations
COPY ./pb_hooks /pb/pb_hooks

EXPOSE 8080

CMD ["/pb/pocketbase", "serve", "--http=0.0.0.0:8080"]
```

```yaml
# railway.toml
[build]
  builder = "dockerfile"

[deploy]
  healthcheckPath = "/api/health"
  restartPolicyType = "on_failure"
```

### 2.4 Fly.io Deployment

```toml
# fly.toml
app = "cheory-backend"
primary_region = "fra"

[build]
  dockerfile = "Dockerfile"

[http_service]
  internal_port = 8080
  force_https = true

[mounts]
  source = "pb_data"
  destination = "/pb/pb_data"
```

```bash
# Deploy
fly launch
fly volumes create pb_data --size 1
fly deploy
```

### 2.5 Environment Variables

```bash
# .env
POCKETBASE_URL=https://api.cheory.app
STRIPE_SECRET_KEY=sk_live_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
ADMIN_EMAIL=admin@cheory.app
ADMIN_PASSWORD=secure_password_here

# OAuth (optional)
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
APPLE_CLIENT_ID=xxx
APPLE_CLIENT_SECRET=xxx
```

---

## 3. COLLECTIONS SCHEMA

### 3.1 Overview

```
Collections:
├── users (system)          # Extended auth collection
├── subscriptions           # Payment/tier data
├── game_records            # Game history
├── user_stats              # Aggregated statistics
├── achievements            # Unlocked achievements
└── user_preferences        # Settings sync
```

### 3.2 Users Collection (Extend System)

```javascript
// pb_migrations/1_users_extend.js

migrate((db) => {
  const users = db.collection('users')
  
  // Add custom fields to built-in users
  users.schema.addField({
    name: 'display_name',
    type: 'text',
    required: true,
    min: 2,
    max: 20,
    pattern: '^[a-zA-Z0-9_ ]+$'
  })
  
  users.schema.addField({
    name: 'avatar',
    type: 'text',
    options: { maxSelect: 1 }
  })
  
  users.schema.addField({
    name: 'tier',
    type: 'select',
    options: { values: ['free', 'premium'] },
    default: 'free'
  })
  
  users.schema.addField({
    name: 'xp',
    type: 'number',
    default: 0
  })
  
  users.schema.addField({
    name: 'level',
    type: 'number',
    default: 1
  })
  
  users.schema.addField({
    name: 'language',
    type: 'select',
    options: { values: ['en', 'tr'] },
    default: 'en'
  })
  
  users.schema.addField({
    name: 'theme',
    type: 'select',
    options: { values: ['dark', 'light', 'system'] },
    default: 'system'
  })
  
  users.schema.addField({
    name: 'onboarding_completed',
    type: 'bool',
    default: false
  })
  
  users.schema.addField({
    name: 'stripe_customer_id',
    type: 'text'
  })
  
  return users.save()
})
```

**Users Schema:**

| Field | Type | Required | Default |
|-------|------|----------|---------|
| id | string | auto | - |
| email | string | yes | - |
| password | string | yes | - |
| display_name | string | yes | "Player" |
| avatar | string | no | "default" |
| tier | select | yes | "free" |
| xp | number | no | 0 |
| level | number | no | 1 |
| language | select | no | "en" |
| theme | select | no | "system" |
| onboarding_completed | bool | no | false |
| stripe_customer_id | string | no | null |
| created | datetime | auto | - |
| updated | datetime | auto | - |

### 3.3 Subscriptions Collection

```javascript
// pb_migrations/2_subscriptions.js

migrate((db) => {
  const collection = new Collection({
    name: 'subscriptions',
    type: 'base',
    schema: [
      {
        name: 'user',
        type: 'relation',
        required: true,
        options: {
          collectionId: '_pb_users_auth_',
          cascadeDelete: true,
          maxSelect: 1
        }
      },
      {
        name: 'stripe_subscription_id',
        type: 'text',
        required: true
      },
      {
        name: 'stripe_price_id',
        type: 'text',
        required: true
      },
      {
        name: 'status',
        type: 'select',
        required: true,
        options: {
          values: ['active', 'canceled', 'past_due', 'trialing', 'incomplete']
        }
      },
      {
        name: 'current_period_start',
        type: 'date',
        required: true
      },
      {
        name: 'current_period_end',
        type: 'date',
        required: true
      },
      {
        name: 'cancel_at_period_end',
        type: 'bool',
        default: false
      }
    ]
  })
  
  return db.saveCollection(collection)
})
```

**Subscriptions Schema:**

| Field | Type | Required |
|-------|------|----------|
| id | string | auto |
| user | relation→users | yes |
| stripe_subscription_id | string | yes |
| stripe_price_id | string | yes |
| status | select | yes |
| current_period_start | date | yes |
| current_period_end | date | yes |
| cancel_at_period_end | bool | no |
| created | datetime | auto |
| updated | datetime | auto |

### 3.4 Game Records Collection

```javascript
// pb_migrations/3_game_records.js

migrate((db) => {
  const collection = new Collection({
    name: 'game_records',
    type: 'base',
    schema: [
      {
        name: 'user',
        type: 'relation',
        required: true,
        options: {
          collectionId: '_pb_users_auth_',
          cascadeDelete: true,
          maxSelect: 1
        }
      },
      {
        name: 'module_id',
        type: 'text',
        required: true
      },
      {
        name: 'opponent_type',
        type: 'select',
        required: true,
        options: { values: ['bot', 'human'] }
      },
      {
        name: 'opponent_strategy',
        type: 'text'
      },
      {
        name: 'difficulty',
        type: 'select',
        options: { values: ['easy', 'medium', 'hard'] }
      },
      {
        name: 'rounds_played',
        type: 'number',
        required: true
      },
      {
        name: 'player_score',
        type: 'number',
        required: true
      },
      {
        name: 'opponent_score',
        type: 'number',
        required: true
      },
      {
        name: 'result',
        type: 'select',
        required: true,
        options: { values: ['win', 'lose', 'draw'] }
      },
      {
        name: 'duration_seconds',
        type: 'number',
        required: true
      },
      {
        name: 'xp_earned',
        type: 'number',
        default: 0
      },
      {
        name: 'actions_log',
        type: 'json'
      },
      {
        name: 'played_at',
        type: 'date',
        required: true
      }
    ],
    indexes: [
      'CREATE INDEX idx_game_records_user ON game_records (user)',
      'CREATE INDEX idx_game_records_module ON game_records (module_id)',
      'CREATE INDEX idx_game_records_played ON game_records (played_at)'
    ]
  })
  
  return db.saveCollection(collection)
})
```

**Game Records Schema:**

| Field | Type | Required |
|-------|------|----------|
| id | string | auto |
| user | relation→users | yes |
| module_id | string | yes |
| opponent_type | select | yes |
| opponent_strategy | string | no |
| difficulty | select | no |
| rounds_played | number | yes |
| player_score | number | yes |
| opponent_score | number | yes |
| result | select | yes |
| duration_seconds | number | yes |
| xp_earned | number | no |
| actions_log | json | no |
| played_at | date | yes |

### 3.5 User Stats Collection

```javascript
// pb_migrations/4_user_stats.js

migrate((db) => {
  const collection = new Collection({
    name: 'user_stats',
    type: 'base',
    schema: [
      {
        name: 'user',
        type: 'relation',
        required: true,
        options: {
          collectionId: '_pb_users_auth_',
          cascadeDelete: true,
          maxSelect: 1
        }
      },
      {
        name: 'module_id',
        type: 'text',
        required: true
      },
      {
        name: 'games_played',
        type: 'number',
        default: 0
      },
      {
        name: 'games_won',
        type: 'number',
        default: 0
      },
      {
        name: 'games_lost',
        type: 'number',
        default: 0
      },
      {
        name: 'games_draw',
        type: 'number',
        default: 0
      },
      {
        name: 'total_score',
        type: 'number',
        default: 0
      },
      {
        name: 'high_score',
        type: 'number',
        default: 0
      },
      {
        name: 'total_time_seconds',
        type: 'number',
        default: 0
      },
      {
        name: 'current_win_streak',
        type: 'number',
        default: 0
      },
      {
        name: 'best_win_streak',
        type: 'number',
        default: 0
      },
      {
        name: 'last_played_at',
        type: 'date'
      }
    ],
    indexes: [
      'CREATE UNIQUE INDEX idx_user_stats_unique ON user_stats (user, module_id)'
    ]
  })
  
  return db.saveCollection(collection)
})
```

### 3.6 Achievements Collection

```javascript
// pb_migrations/5_achievements.js

migrate((db) => {
  const collection = new Collection({
    name: 'achievements',
    type: 'base',
    schema: [
      {
        name: 'user',
        type: 'relation',
        required: true,
        options: {
          collectionId: '_pb_users_auth_',
          cascadeDelete: true,
          maxSelect: 1
        }
      },
      {
        name: 'achievement_id',
        type: 'text',
        required: true
      },
      {
        name: 'unlocked_at',
        type: 'date',
        required: true
      },
      {
        name: 'progress',
        type: 'number',
        default: 100
      }
    ],
    indexes: [
      'CREATE UNIQUE INDEX idx_achievements_unique ON achievements (user, achievement_id)'
    ]
  })
  
  return db.saveCollection(collection)
})
```

### 3.7 User Preferences Collection

```javascript
// pb_migrations/6_user_preferences.js

migrate((db) => {
  const collection = new Collection({
    name: 'user_preferences',
    type: 'base',
    schema: [
      {
        name: 'user',
        type: 'relation',
        required: true,
        options: {
          collectionId: '_pb_users_auth_',
          cascadeDelete: true,
          maxSelect: 1
        }
      },
      {
        name: 'sound_enabled',
        type: 'bool',
        default: true
      },
      {
        name: 'sound_volume',
        type: 'number',
        default: 80
      },
      {
        name: 'music_enabled',
        type: 'bool',
        default: true
      },
      {
        name: 'music_volume',
        type: 'number',
        default: 50
      },
      {
        name: 'animations_enabled',
        type: 'bool',
        default: true
      },
      {
        name: 'show_hints',
        type: 'bool',
        default: true
      },
      {
        name: 'confirm_actions',
        type: 'bool',
        default: false
      }
    ],
    indexes: [
      'CREATE UNIQUE INDEX idx_user_prefs_unique ON user_preferences (user)'
    ]
  })
  
  return db.saveCollection(collection)
})
```

---

## 4. API RULES (PERMISSIONS)

### 4.1 Users Collection Rules

```javascript
// Admin UI → Collections → users → API Rules

// List/Search: Only admins
listRule: '@request.auth.id != ""'  // Logged in users can list
// or null for admins only

// View: Own record or admin
viewRule: '@request.auth.id = id'

// Create: Public (registration)
createRule: ''

// Update: Own record only
updateRule: '@request.auth.id = id'

// Delete: Own record only (or disable)
deleteRule: '@request.auth.id = id'
```

### 4.2 Game Records Rules

```javascript
// List: Own records only
listRule: '@request.auth.id = user.id'

// View: Own records only
viewRule: '@request.auth.id = user.id'

// Create: Logged in, own user field
createRule: '@request.auth.id != "" && @request.auth.id = @request.data.user'

// Update: Never (immutable)
updateRule: null

// Delete: Own records only
deleteRule: '@request.auth.id = user.id'
```

### 4.3 Subscriptions Rules

```javascript
// List: Own records only
listRule: '@request.auth.id = user.id'

// View: Own records only
viewRule: '@request.auth.id = user.id'

// Create: Only via webhook (admin)
createRule: null

// Update: Only via webhook (admin)
updateRule: null

// Delete: Only admin
deleteRule: null
```

### 4.4 Achievements Rules

```javascript
// List: Own records only
listRule: '@request.auth.id = user.id'

// View: Own records only
viewRule: '@request.auth.id = user.id'

// Create: Logged in, own user
createRule: '@request.auth.id != "" && @request.auth.id = @request.data.user'

// Update: Never
updateRule: null

// Delete: Never
deleteRule: null
```

---

## 5. AUTH CONFIGURATION

### 5.1 Email/Password Auth

```javascript
// Admin UI → Settings → Auth providers

{
  "emailAuth": {
    "enabled": true,
    "minPasswordLength": 8,
    "requireEmail": true
  }
}
```

### 5.2 OAuth Providers

```javascript
// Admin UI → Settings → Auth providers → OAuth2

// Google
{
  "google": {
    "enabled": true,
    "clientId": "xxx.apps.googleusercontent.com",
    "clientSecret": "xxx",
    "authUrl": "https://accounts.google.com/o/oauth2/v2/auth",
    "tokenUrl": "https://oauth2.googleapis.com/token"
  }
}

// Apple
{
  "apple": {
    "enabled": true,
    "clientId": "com.cheory.app",
    "clientSecret": "xxx",
    "authUrl": "https://appleid.apple.com/auth/authorize",
    "tokenUrl": "https://appleid.apple.com/auth/token"
  }
}
```

### 5.3 Email Templates

```javascript
// Admin UI → Settings → Mail settings

// Verification Email
{
  "subject": "Verify your Cheory account",
  "body": `
    <h2>Welcome to Cheory!</h2>
    <p>Click the link below to verify your email:</p>
    <p><a href="{ACTION_URL}">Verify Email</a></p>
    <p>This link expires in 24 hours.</p>
  `
}

// Password Reset
{
  "subject": "Reset your Cheory password",
  "body": `
    <h2>Password Reset</h2>
    <p>Click the link below to reset your password:</p>
    <p><a href="{ACTION_URL}">Reset Password</a></p>
    <p>This link expires in 1 hour.</p>
  `
}
```

### 5.4 SMTP Configuration

```javascript
// Admin UI → Settings → Mail settings

{
  "smtp": {
    "enabled": true,
    "host": "smtp.postmarkapp.com",
    "port": 587,
    "username": "xxx",
    "password": "xxx",
    "tls": true
  },
  "sender": {
    "name": "Cheory",
    "address": "noreply@cheory.app"
  }
}
```

---

## 6. FRONTEND INTEGRATION

### 6.1 SDK Installation

```html
<!-- Option 1: CDN -->
<script src="https://unpkg.com/pocketbase@0.22.0/dist/pocketbase.umd.js"></script>

<!-- Option 2: ES Module -->
<script type="module">
  import PocketBase from 'https://unpkg.com/pocketbase@0.22.0/dist/pocketbase.es.mjs'
</script>
```

### 6.2 Client Initialization

```javascript
// /core/backend/client.js

import PocketBase from 'pocketbase'

const pb = new PocketBase('https://api.cheory.app')

// Auto-refresh auth
pb.authStore.onChange((token, model) => {
  console.log('Auth changed:', model?.email)
})

export { pb }
```

### 6.3 Auth Functions

```javascript
// /core/backend/auth.js

import { pb } from './client.js'

// Register
export async function register(email, password, displayName) {
  const user = await pb.collection('users').create({
    email,
    password,
    passwordConfirm: password,
    display_name: displayName,
    tier: 'free',
    level: 1,
    xp: 0
  })
  
  // Auto login after register
  await pb.collection('users').authWithPassword(email, password)
  
  return user
}

// Login
export async function login(email, password) {
  const authData = await pb.collection('users').authWithPassword(email, password)
  return authData.record
}

// OAuth Login
export async function loginWithGoogle() {
  const authData = await pb.collection('users').authWithOAuth2({ provider: 'google' })
  return authData.record
}

// Logout
export function logout() {
  pb.authStore.clear()
}

// Get current user
export function getCurrentUser() {
  return pb.authStore.model
}

// Check if logged in
export function isLoggedIn() {
  return pb.authStore.isValid
}

// Update profile
export async function updateProfile(data) {
  const user = getCurrentUser()
  if (!user) throw new Error('Not logged in')
  
  return await pb.collection('users').update(user.id, data)
}

// Request password reset
export async function requestPasswordReset(email) {
  return await pb.collection('users').requestPasswordReset(email)
}

// Verify email
export async function requestVerification(email) {
  return await pb.collection('users').requestVerification(email)
}
```

### 6.4 Data Sync Functions

```javascript
// /core/backend/sync.js

import { pb } from './client.js'

// Save game record
export async function saveGameRecord(gameData) {
  const user = pb.authStore.model
  if (!user) return null
  
  return await pb.collection('game_records').create({
    user: user.id,
    module_id: gameData.moduleId,
    opponent_type: gameData.opponentType,
    opponent_strategy: gameData.opponentStrategy,
    difficulty: gameData.difficulty,
    rounds_played: gameData.roundsPlayed,
    player_score: gameData.playerScore,
    opponent_score: gameData.opponentScore,
    result: gameData.result,
    duration_seconds: gameData.duration,
    xp_earned: gameData.xpEarned,
    actions_log: gameData.actionsLog,
    played_at: new Date().toISOString()
  })
}

// Get game history
export async function getGameHistory(moduleId = null, page = 1, perPage = 20) {
  const user = pb.authStore.model
  if (!user) return []
  
  let filter = `user = "${user.id}"`
  if (moduleId) {
    filter += ` && module_id = "${moduleId}"`
  }
  
  return await pb.collection('game_records').getList(page, perPage, {
    filter,
    sort: '-played_at'
  })
}

// Get user stats
export async function getUserStats(moduleId = null) {
  const user = pb.authStore.model
  if (!user) return null
  
  let filter = `user = "${user.id}"`
  if (moduleId) {
    filter += ` && module_id = "${moduleId}"`
  }
  
  return await pb.collection('user_stats').getFullList({
    filter
  })
}

// Update user stats (called after game)
export async function updateUserStats(moduleId, gameResult) {
  const user = pb.authStore.model
  if (!user) return null
  
  // Find existing stats
  let stats
  try {
    stats = await pb.collection('user_stats').getFirstListItem(
      `user = "${user.id}" && module_id = "${moduleId}"`
    )
  } catch {
    // Create new stats record
    stats = await pb.collection('user_stats').create({
      user: user.id,
      module_id: moduleId,
      games_played: 0,
      games_won: 0,
      games_lost: 0,
      games_draw: 0,
      total_score: 0,
      high_score: 0,
      total_time_seconds: 0,
      current_win_streak: 0,
      best_win_streak: 0
    })
  }
  
  // Calculate new stats
  const updates = {
    games_played: stats.games_played + 1,
    total_score: stats.total_score + gameResult.playerScore,
    total_time_seconds: stats.total_time_seconds + gameResult.duration,
    last_played_at: new Date().toISOString()
  }
  
  if (gameResult.playerScore > stats.high_score) {
    updates.high_score = gameResult.playerScore
  }
  
  if (gameResult.result === 'win') {
    updates.games_won = stats.games_won + 1
    updates.current_win_streak = stats.current_win_streak + 1
    if (updates.current_win_streak > stats.best_win_streak) {
      updates.best_win_streak = updates.current_win_streak
    }
  } else if (gameResult.result === 'lose') {
    updates.games_lost = stats.games_lost + 1
    updates.current_win_streak = 0
  } else {
    updates.games_draw = stats.games_draw + 1
  }
  
  return await pb.collection('user_stats').update(stats.id, updates)
}

// Unlock achievement
export async function unlockAchievement(achievementId) {
  const user = pb.authStore.model
  if (!user) return null
  
  // Check if already unlocked
  try {
    await pb.collection('achievements').getFirstListItem(
      `user = "${user.id}" && achievement_id = "${achievementId}"`
    )
    return null // Already unlocked
  } catch {
    // Not found, create new
    return await pb.collection('achievements').create({
      user: user.id,
      achievement_id: achievementId,
      unlocked_at: new Date().toISOString(),
      progress: 100
    })
  }
}

// Get achievements
export async function getAchievements() {
  const user = pb.authStore.model
  if (!user) return []
  
  return await pb.collection('achievements').getFullList({
    filter: `user = "${user.id}"`
  })
}
```

### 6.5 Real-time Subscriptions

```javascript
// /core/backend/realtime.js

import { pb } from './client.js'

// Subscribe to user changes
export function subscribeToUser(callback) {
  const user = pb.authStore.model
  if (!user) return null
  
  return pb.collection('users').subscribe(user.id, (e) => {
    console.log('User updated:', e.action)
    callback(e.record)
  })
}

// Subscribe to achievements
export function subscribeToAchievements(callback) {
  const user = pb.authStore.model
  if (!user) return null
  
  return pb.collection('achievements').subscribe('*', (e) => {
    if (e.record.user === user.id) {
      callback(e.action, e.record)
    }
  })
}

// Unsubscribe
export function unsubscribeAll() {
  pb.collection('users').unsubscribe()
  pb.collection('achievements').unsubscribe()
}
```

### 6.6 Offline → Online Merge

```javascript
// /core/backend/merge.js

import { pb } from './client.js'
import { local } from '/core/storage/local.js'
import { GameDB } from '/core/storage/indexeddb.js'

// Merge local data to server
export async function mergeLocalToServer() {
  const user = pb.authStore.model
  if (!user) return
  
  // Get local game records
  const localGames = await GameDB.getAllGames()
  
  // Filter games not yet synced
  const unsynced = localGames.filter(g => !g.syncedAt)
  
  for (const game of unsynced) {
    try {
      await pb.collection('game_records').create({
        user: user.id,
        ...game
      })
      
      // Mark as synced locally
      await GameDB.update(game.id, { syncedAt: Date.now() })
    } catch (error) {
      console.error('Sync failed for game:', game.id, error)
    }
  }
  
  // Sync achievements
  const localAchievements = local.get('achievements')?.unlocked || []
  for (const ach of localAchievements) {
    if (!ach.syncedAt) {
      try {
        await pb.collection('achievements').create({
          user: user.id,
          achievement_id: ach.id,
          unlocked_at: new Date(ach.unlockedAt).toISOString()
        })
        ach.syncedAt = Date.now()
      } catch {
        // Might already exist, ignore
      }
    }
  }
  local.set('achievements', { unlocked: localAchievements })
  
  console.log('Local data merged to server')
}

// Pull server data to local
export async function pullServerToLocal() {
  const user = pb.authStore.model
  if (!user) return
  
  // Update local user data
  local.set('user', {
    id: user.id,
    email: user.email,
    displayName: user.display_name,
    tier: user.tier,
    xp: user.xp,
    level: user.level,
    language: user.language,
    theme: user.theme
  })
  
  // Pull achievements
  const serverAchievements = await pb.collection('achievements').getFullList({
    filter: `user = "${user.id}"`
  })
  
  const localAchievements = serverAchievements.map(a => ({
    id: a.achievement_id,
    unlockedAt: new Date(a.unlocked_at).getTime(),
    syncedAt: Date.now()
  }))
  
  local.set('achievements', { unlocked: localAchievements })
  
  console.log('Server data pulled to local')
}
```

---

## 7. PAYMENT INTEGRATION (STRIPE)

### 7.1 Stripe Products Setup

```
Products in Stripe Dashboard:

1. Cheory Premium Monthly
   - Price ID: price_monthly_xxx
   - Amount: $4.99/month

2. Cheory Premium Yearly
   - Price ID: price_yearly_xxx
   - Amount: $39.99/year
```

### 7.2 Checkout Session (Frontend)

```javascript
// /core/backend/payments.js

import { pb } from './client.js'

const STRIPE_PRICES = {
  monthly: 'price_monthly_xxx',
  yearly: 'price_yearly_xxx'
}

// Create checkout session
export async function createCheckoutSession(plan) {
  const user = pb.authStore.model
  if (!user) throw new Error('Not logged in')
  
  const response = await fetch('https://api.cheory.app/api/create-checkout', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': pb.authStore.token
    },
    body: JSON.stringify({
      priceId: STRIPE_PRICES[plan],
      userId: user.id,
      customerEmail: user.email
    })
  })
  
  const { url } = await response.json()
  window.location.href = url
}

// Create customer portal session
export async function createPortalSession() {
  const user = pb.authStore.model
  if (!user) throw new Error('Not logged in')
  
  const response = await fetch('https://api.cheory.app/api/create-portal', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': pb.authStore.token
    },
    body: JSON.stringify({
      userId: user.id
    })
  })
  
  const { url } = await response.json()
  window.location.href = url
}
```

### 7.3 Webhook Handler (PocketBase Hook)

```javascript
// pb_hooks/webhooks.pb.js

/// <reference path="../pb_data/types.d.ts" />

routerAdd("POST", "/api/stripe-webhook", (c) => {
  const stripe = require('stripe')($os.getenv('STRIPE_SECRET_KEY'))
  const endpointSecret = $os.getenv('STRIPE_WEBHOOK_SECRET')
  
  const sig = c.request().header.get('stripe-signature')
  const body = readerToString(c.request().body)
  
  let event
  try {
    event = stripe.webhooks.constructEvent(body, sig, endpointSecret)
  } catch (err) {
    return c.json(400, { error: 'Webhook signature verification failed' })
  }
  
  switch (event.type) {
    case 'checkout.session.completed': {
      const session = event.data.object
      handleCheckoutComplete(session)
      break
    }
    
    case 'customer.subscription.updated': {
      const subscription = event.data.object
      handleSubscriptionUpdate(subscription)
      break
    }
    
    case 'customer.subscription.deleted': {
      const subscription = event.data.object
      handleSubscriptionDeleted(subscription)
      break
    }
    
    case 'invoice.payment_failed': {
      const invoice = event.data.object
      handlePaymentFailed(invoice)
      break
    }
  }
  
  return c.json(200, { received: true })
})

function handleCheckoutComplete(session) {
  const userId = session.metadata.userId
  const subscriptionId = session.subscription
  const customerId = session.customer
  
  // Update user
  const users = $app.dao().findCollectionByNameOrId('users')
  const user = $app.dao().findRecordById(users, userId)
  user.set('tier', 'premium')
  user.set('stripe_customer_id', customerId)
  $app.dao().saveRecord(user)
  
  // Create subscription record
  const subs = $app.dao().findCollectionByNameOrId('subscriptions')
  const record = new Record(subs)
  record.set('user', userId)
  record.set('stripe_subscription_id', subscriptionId)
  record.set('stripe_price_id', session.metadata.priceId)
  record.set('status', 'active')
  record.set('current_period_start', new Date())
  record.set('current_period_end', new Date(Date.now() + 30 * 24 * 60 * 60 * 1000))
  $app.dao().saveRecord(record)
}

function handleSubscriptionUpdate(subscription) {
  const subs = $app.dao().findCollectionByNameOrId('subscriptions')
  const record = $app.dao().findFirstRecordByData(subs, 'stripe_subscription_id', subscription.id)
  
  if (record) {
    record.set('status', subscription.status)
    record.set('current_period_start', new Date(subscription.current_period_start * 1000))
    record.set('current_period_end', new Date(subscription.current_period_end * 1000))
    record.set('cancel_at_period_end', subscription.cancel_at_period_end)
    $app.dao().saveRecord(record)
    
    // Update user tier based on status
    const users = $app.dao().findCollectionByNameOrId('users')
    const user = $app.dao().findRecordById(users, record.get('user'))
    user.set('tier', subscription.status === 'active' ? 'premium' : 'free')
    $app.dao().saveRecord(user)
  }
}

function handleSubscriptionDeleted(subscription) {
  const subs = $app.dao().findCollectionByNameOrId('subscriptions')
  const record = $app.dao().findFirstRecordByData(subs, 'stripe_subscription_id', subscription.id)
  
  if (record) {
    // Update user to free tier
    const users = $app.dao().findCollectionByNameOrId('users')
    const user = $app.dao().findRecordById(users, record.get('user'))
    user.set('tier', 'free')
    $app.dao().saveRecord(user)
    
    // Delete subscription record
    $app.dao().deleteRecord(record)
  }
}

function handlePaymentFailed(invoice) {
  // Log and potentially notify user
  console.log('Payment failed for customer:', invoice.customer)
}
```

### 7.4 Checkout Endpoint (PocketBase Hook)

```javascript
// pb_hooks/checkout.pb.js

routerAdd("POST", "/api/create-checkout", (c) => {
  const stripe = require('stripe')($os.getenv('STRIPE_SECRET_KEY'))
  
  const data = $apis.requestInfo(c).data
  const { priceId, userId, customerEmail } = data
  
  // Verify user is authenticated
  const authRecord = c.get('authRecord')
  if (!authRecord || authRecord.id !== userId) {
    return c.json(401, { error: 'Unauthorized' })
  }
  
  // Create checkout session
  const session = stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    customer_email: customerEmail,
    line_items: [{
      price: priceId,
      quantity: 1
    }],
    success_url: 'https://cheory.app/settings?success=true',
    cancel_url: 'https://cheory.app/settings?canceled=true',
    metadata: {
      userId: userId,
      priceId: priceId
    }
  })
  
  return c.json(200, { url: session.url })
}, $apis.requireRecordAuth())

routerAdd("POST", "/api/create-portal", (c) => {
  const stripe = require('stripe')($os.getenv('STRIPE_SECRET_KEY'))
  
  const data = $apis.requestInfo(c).data
  const { userId } = data
  
  // Get user's stripe customer ID
  const users = $app.dao().findCollectionByNameOrId('users')
  const user = $app.dao().findRecordById(users, userId)
  const customerId = user.get('stripe_customer_id')
  
  if (!customerId) {
    return c.json(400, { error: 'No subscription found' })
  }
  
  const session = stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: 'https://cheory.app/settings'
  })
  
  return c.json(200, { url: session.url })
}, $apis.requireRecordAuth())
```

---

## 8. ADMIN & SECURITY

### 8.1 Admin Account Setup

```bash
# First run - create admin
./pocketbase serve

# Visit http://127.0.0.1:8090/_/
# Create admin account with secure password
```

### 8.2 Security Settings

```javascript
// Admin UI → Settings → Application

{
  "appName": "Cheory",
  "appUrl": "https://cheory.app",
  "hideControls": false,
  "rateLimits": {
    "rules": [
      {
        "label": "Auth endpoints",
        "audience": "*",
        "duration": 60,
        "maxRequests": 10,
        "paths": [
          "POST /api/collections/users/auth-with-password",
          "POST /api/collections/users/auth-with-oauth2"
        ]
      },
      {
        "label": "Create records",
        "audience": "*",
        "duration": 60,
        "maxRequests": 30,
        "paths": [
          "POST /api/collections/game_records/records"
        ]
      }
    ]
  }
}
```

### 8.3 CORS Configuration

```javascript
// pb_hooks/cors.pb.js

routerUse((next) => {
  return (c) => {
    c.response().header().set('Access-Control-Allow-Origin', 'https://cheory.app')
    c.response().header().set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS')
    c.response().header().set('Access-Control-Allow-Headers', 'Content-Type, Authorization')
    
    if (c.request().method === 'OPTIONS') {
      return c.noContent(204)
    }
    
    return next(c)
  }
})
```

### 8.4 Input Validation Hook

```javascript
// pb_hooks/validation.pb.js

onRecordBeforeCreateRequest((e) => {
  const collection = e.collection.name
  
  if (collection === 'users') {
    // Validate display name
    const displayName = e.record.get('display_name')
    if (!/^[a-zA-Z0-9_ ]{2,20}$/.test(displayName)) {
      throw new BadRequestError('Invalid display name')
    }
    
    // Force default values
    e.record.set('tier', 'free')
    e.record.set('xp', 0)
    e.record.set('level', 1)
  }
  
  if (collection === 'game_records') {
    // Validate module_id
    const validModules = [
      'prisoners-dilemma', 'iterated-pd', 'stag-hunt',
      'hawk-dove', 'matching-pennies', 'battle-of-sexes',
      'public-goods', 'ultimatum', 'dictator',
      'trust-game', 'rps', 'voting-lab'
    ]
    
    if (!validModules.includes(e.record.get('module_id'))) {
      throw new BadRequestError('Invalid module_id')
    }
  }
}, 'users', 'game_records')
```

---

## 9. BACKUP & MIGRATION

### 9.1 Automated Backup

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# Create backup
cp /pb/pb_data/data.db "$BACKUP_DIR/data_$DATE.db"

# Keep only last 7 days
find $BACKUP_DIR -name "data_*.db" -mtime +7 -delete

# Upload to S3 (optional)
aws s3 cp "$BACKUP_DIR/data_$DATE.db" s3://cheory-backups/
```

### 9.2 Railway Backup Cron

```yaml
# railway.toml
[build]
  builder = "dockerfile"

[deploy]
  cronSchedule = "0 3 * * *"  # Daily at 3 AM
```

### 9.3 Data Export Endpoint

```javascript
// pb_hooks/export.pb.js

routerAdd("GET", "/api/export-my-data", (c) => {
  const authRecord = c.get('authRecord')
  if (!authRecord) {
    return c.json(401, { error: 'Unauthorized' })
  }
  
  const userId = authRecord.id
  
  // Gather all user data
  const data = {
    user: authRecord,
    gameRecords: $app.dao().findRecordsByExpr('game_records', 
      $dbx.exp('user = {:id}', { id: userId })),
    achievements: $app.dao().findRecordsByExpr('achievements',
      $dbx.exp('user = {:id}', { id: userId })),
    stats: $app.dao().findRecordsByExpr('user_stats',
      $dbx.exp('user = {:id}', { id: userId })),
    preferences: $app.dao().findRecordsByExpr('user_preferences',
      $dbx.exp('user = {:id}', { id: userId })),
    exportedAt: new Date().toISOString()
  }
  
  c.response().header().set('Content-Disposition', 
    'attachment; filename="cheory-data.json"')
  return c.json(200, data)
}, $apis.requireRecordAuth())
```

### 9.4 Account Deletion

```javascript
// pb_hooks/delete_account.pb.js

routerAdd("DELETE", "/api/delete-my-account", (c) => {
  const authRecord = c.get('authRecord')
  if (!authRecord) {
    return c.json(401, { error: 'Unauthorized' })
  }
  
  const userId = authRecord.id
  
  // Cancel Stripe subscription if exists
  const customerId = authRecord.get('stripe_customer_id')
  if (customerId) {
    const stripe = require('stripe')($os.getenv('STRIPE_SECRET_KEY'))
    const subscriptions = stripe.subscriptions.list({ customer: customerId })
    for (const sub of subscriptions.data) {
      stripe.subscriptions.cancel(sub.id)
    }
  }
  
  // Delete related records (cascade should handle most)
  // But explicitly delete just in case
  $app.dao().deleteRecordsByExpr('game_records', $dbx.exp('user = {:id}', { id: userId }))
  $app.dao().deleteRecordsByExpr('achievements', $dbx.exp('user = {:id}', { id: userId }))
  $app.dao().deleteRecordsByExpr('user_stats', $dbx.exp('user = {:id}', { id: userId }))
  $app.dao().deleteRecordsByExpr('user_preferences', $dbx.exp('user = {:id}', { id: userId }))
  $app.dao().deleteRecordsByExpr('subscriptions', $dbx.exp('user = {:id}', { id: userId }))
  
  // Delete user
  $app.dao().deleteRecord(authRecord)
  
  return c.json(200, { message: 'Account deleted' })
}, $apis.requireRecordAuth())
```

---

## 10. MONITORING

### 10.1 Health Check

```javascript
// pb_hooks/health.pb.js

routerAdd("GET", "/api/health", (c) => {
  return c.json(200, {
    status: 'ok',
    timestamp: new Date().toISOString(),
    version: '1.0.0'
  })
})
```

### 10.2 Basic Analytics Hook

```javascript
// pb_hooks/analytics.pb.js

onRecordAfterCreateRequest((e) => {
  if (e.collection.name === 'game_records') {
    // Log game completion
    console.log(JSON.stringify({
      event: 'game_completed',
      module: e.record.get('module_id'),
      result: e.record.get('result'),
      timestamp: new Date().toISOString()
    }))
  }
}, 'game_records')

onRecordAfterCreateRequest((e) => {
  if (e.collection.name === 'users') {
    // Log new user
    console.log(JSON.stringify({
      event: 'user_registered',
      timestamp: new Date().toISOString()
    }))
  }
}, 'users')
```

---

## 11. QUICK START CHECKLIST

```
□ Download PocketBase binary
□ Create admin account
□ Run migrations (create collections)
□ Configure auth providers
□ Set up SMTP for emails
□ Configure CORS
□ Set environment variables
□ Deploy to Railway/Fly.io
□ Set up Stripe products
□ Configure webhook endpoint
□ Test auth flow
□ Test payment flow
□ Set up backup cron
□ Monitor health endpoint
```

---

## 12. REFERENCES

- PocketBase Docs: https://pocketbase.io/docs
- PocketBase JS SDK: https://github.com/pocketbase/js-sdk
- Stripe Docs: https://stripe.com/docs
- Railway Docs: https://docs.railway.app
- Fly.io Docs: https://fly.io/docs
