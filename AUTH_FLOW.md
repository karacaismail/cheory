# AUTH FLOW
## Technical Specification Document

| Field | Value |
|-------|-------|
| Document ID | `AUTH_FLOW` |
| Version | 1.0.0 |
| Tier | 3 - Technical |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Purpose

This document defines authentication, authorization, and user management flows for the Cheory platform. V1 uses a local-first approach with optional cloud sync.

### 1.2 Auth Strategy

```
V1 (MVP): Anonymous local user
V2 (Future): Optional account creation for sync

NO mandatory authentication required to play.
```

### 1.3 User Types

| Type | Description | Features |
|------|-------------|----------|
| Anonymous | Auto-created on first visit | Full free tier access |
| Registered | Email + password (V2) | Cloud sync, cross-device |
| Premium | Paid subscription | All games unlocked |

---

## 2. ANONYMOUS USER FLOW

### 2.1 First Visit

```
┌─────────────────────────────────────────────┐
│                FIRST VISIT                   │
├─────────────────────────────────────────────┤
│                                             │
│  1. User visits site                        │
│  2. Check localStorage for existing user    │
│  3. If none: Generate UUID                  │
│  4. Create default UserProfile              │
│  5. Store in localStorage                   │
│  6. Show onboarding/welcome                 │
│                                             │
└─────────────────────────────────────────────┘
```

### 2.2 Implementation

```javascript
// /core/auth/anonymous.js

const STORAGE_KEY = 'cheory:user'

function generateUserId() {
  return 'user_' + crypto.randomUUID()
}

function createDefaultProfile() {
  return {
    id: generateUserId(),
    createdAt: Date.now(),
    updatedAt: Date.now(),
    displayName: 'Player',
    avatar: 'default',
    language: detectLanguage(),
    theme: 'system',
    level: 1,
    xp: 0,
    tier: 'free',
    tutorialCompleted: false,
    onboardingCompleted: false,
    analyticsConsent: null  // Will ask on first session
  }
}

function getOrCreateUser() {
  let user = localStorage.getItem(STORAGE_KEY)
  
  if (user) {
    return JSON.parse(user)
  }
  
  const newUser = createDefaultProfile()
  localStorage.setItem(STORAGE_KEY, JSON.stringify(newUser))
  return newUser
}

function detectLanguage() {
  const browserLang = navigator.language.slice(0, 2)
  return ['en', 'tr'].includes(browserLang) ? browserLang : 'en'
}

export { getOrCreateUser, createDefaultProfile }
```

### 2.3 Returning Visit

```
┌─────────────────────────────────────────────┐
│              RETURNING VISIT                 │
├─────────────────────────────────────────────┤
│                                             │
│  1. User visits site                        │
│  2. Load user from localStorage             │
│  3. Validate user object structure          │
│  4. Migrate if schema changed               │
│  5. Update lastVisitAt                      │
│  6. Resume where they left off              │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 3. USER PROFILE MANAGEMENT

### 3.1 Profile Update

```javascript
function updateProfile(updates) {
  const user = getOrCreateUser()
  
  const updatedUser = {
    ...user,
    ...updates,
    updatedAt: Date.now()
  }
  
  // Validate updates
  if (!validateProfile(updatedUser)) {
    throw new Error('Invalid profile update')
  }
  
  localStorage.setItem(STORAGE_KEY, JSON.stringify(updatedUser))
  
  // Emit event for UI updates
  window.dispatchEvent(new CustomEvent('user:updated', {
    detail: updatedUser
  }))
  
  return updatedUser
}

function validateProfile(profile) {
  const required = ['id', 'displayName', 'language', 'theme', 'tier']
  return required.every(field => profile[field] !== undefined)
}
```

### 3.2 Profile Fields

| Field | Editable | Validation |
|-------|----------|------------|
| displayName | Yes | 1-30 chars, no special chars |
| avatar | Yes | Valid avatar ID |
| language | Yes | 'en' or 'tr' |
| theme | Yes | 'dark', 'light', 'system' |
| tier | No | Set by payment system |
| level | No | Calculated from XP |
| xp | No | Earned through gameplay |

### 3.3 Display Name Validation

```javascript
function validateDisplayName(name) {
  if (!name || typeof name !== 'string') return false
  if (name.length < 1 || name.length > 30) return false
  if (!/^[a-zA-Z0-9_ ]+$/.test(name)) return false
  return true
}
```

---

## 4. TIER SYSTEM

### 4.1 Tier Definitions

```javascript
const TIERS = {
  free: {
    id: 'free',
    name: 'Free',
    price: 0,
    features: {
      games: ['prisoners-dilemma', 'stag-hunt', 'matching-pennies', 
              'ultimatum', 'dictator', 'rps'],
      maxSavedGames: 10,
      adsEnabled: true,
      exportData: true,
      cloudSync: false
    }
  },
  
  premium: {
    id: 'premium',
    name: 'Premium',
    price: 4.99,  // Monthly
    features: {
      games: 'all',  // All games unlocked
      maxSavedGames: 100,
      adsEnabled: false,
      exportData: true,
      cloudSync: true,
      prioritySupport: true
    }
  }
}
```

### 4.2 Access Control

```javascript
function canAccessGame(user, gameId) {
  const tier = TIERS[user.tier]
  
  if (tier.features.games === 'all') return true
  return tier.features.games.includes(gameId)
}

function canAccessFeature(user, featureId) {
  const tier = TIERS[user.tier]
  return tier.features[featureId] === true
}

// Usage
if (!canAccessGame(user, 'iterated-pd')) {
  showUpgradePrompt()
}
```

### 4.3 Upgrade Flow (V2)

```
┌─────────────────────────────────────────────┐
│              UPGRADE FLOW                    │
├─────────────────────────────────────────────┤
│                                             │
│  1. User clicks "Upgrade" or locked game    │
│  2. Show pricing modal                      │
│  3. Redirect to payment provider            │
│  4. Handle payment callback                 │
│  5. Update user tier                        │
│  6. Unlock features immediately             │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 5. SESSION MANAGEMENT

### 5.1 Session Tracking

```javascript
const SESSION_KEY = 'cheory:session'

function startSession() {
  const session = {
    id: crypto.randomUUID(),
    startedAt: Date.now(),
    lastActivityAt: Date.now(),
    pageViews: 1,
    gamesPlayed: 0
  }
  
  sessionStorage.setItem(SESSION_KEY, JSON.stringify(session))
  return session
}

function updateSessionActivity() {
  const session = JSON.parse(sessionStorage.getItem(SESSION_KEY))
  if (session) {
    session.lastActivityAt = Date.now()
    session.pageViews++
    sessionStorage.setItem(SESSION_KEY, JSON.stringify(session))
  }
}

function getSessionDuration() {
  const session = JSON.parse(sessionStorage.getItem(SESSION_KEY))
  if (!session) return 0
  return Date.now() - session.startedAt
}
```

### 5.2 Activity Tracking

```javascript
// Track important user actions
function trackActivity(action, data = {}) {
  const user = getOrCreateUser()
  const session = JSON.parse(sessionStorage.getItem(SESSION_KEY))
  
  const event = {
    action,
    data,
    userId: user.id,
    sessionId: session?.id,
    timestamp: Date.now()
  }
  
  // Store locally (or send to analytics if consent given)
  if (user.analyticsConsent) {
    sendToAnalytics(event)
  }
  
  // Always update local stats
  updateLocalStats(action, data)
}

// Usage
trackActivity('game:start', { gameId: 'prisoners-dilemma' })
trackActivity('game:complete', { gameId: 'prisoners-dilemma', score: 15 })
```

---

## 6. DATA PRIVACY

### 6.1 Consent Flow

```
┌─────────────────────────────────────────────┐
│           FIRST SESSION CONSENT             │
├─────────────────────────────────────────────┤
│                                             │
│  "We use cookies and analytics to improve   │
│   your experience."                         │
│                                             │
│  [Accept All]  [Customize]  [Reject All]   │
│                                             │
└─────────────────────────────────────────────┘
```

### 6.2 Consent Storage

```javascript
const CONSENT_KEY = 'cheory:consent'

const ConsentTypes = {
  NECESSARY: 'necessary',      // Always on
  ANALYTICS: 'analytics',      // Usage data
  PREFERENCES: 'preferences',  // Settings sync
  MARKETING: 'marketing'       // If applicable
}

function saveConsent(consents) {
  const consentRecord = {
    ...consents,
    timestamp: Date.now(),
    version: '1.0'
  }
  
  localStorage.setItem(CONSENT_KEY, JSON.stringify(consentRecord))
  
  // Update user profile
  updateProfile({
    analyticsConsent: consents.analytics
  })
}

function getConsent(type) {
  const consent = JSON.parse(localStorage.getItem(CONSENT_KEY))
  if (!consent) return null
  return consent[type]
}
```

### 6.3 Data Deletion

```javascript
function deleteAllUserData() {
  // Confirm with user
  if (!confirm('This will delete ALL your data. Continue?')) {
    return false
  }
  
  // Clear localStorage
  Object.keys(localStorage)
    .filter(k => k.startsWith('cheory:'))
    .forEach(k => localStorage.removeItem(k))
  
  // Clear IndexedDB
  indexedDB.deleteDatabase('cheory')
  
  // Clear sessionStorage
  sessionStorage.clear()
  
  // Reload to create fresh user
  window.location.reload()
  
  return true
}
```

---

## 7. REGISTERED USER FLOW (V2)

### 7.1 Registration

```
┌─────────────────────────────────────────────┐
│             REGISTRATION (V2)                │
├─────────────────────────────────────────────┤
│                                             │
│  Email: [________________]                  │
│  Password: [________________]               │
│  Confirm: [________________]                │
│                                             │
│  [x] I agree to Terms of Service           │
│  [x] I want to receive updates             │
│                                             │
│  [Create Account]                           │
│                                             │
│  Or continue with:                          │
│  [Google] [Apple]                           │
│                                             │
└─────────────────────────────────────────────┘
```

### 7.2 Data Migration

```javascript
// When anonymous user registers
async function migrateToRegisteredUser(credentials, existingUser) {
  // 1. Create server account
  const account = await createAccount(credentials)
  
  // 2. Upload local data
  await uploadUserData({
    userId: account.id,
    localData: {
      profile: existingUser,
      games: await GameDB.getAllGames(),
      achievements: Storage.get('achievements'),
      statistics: await getStatistics()
    }
  })
  
  // 3. Update local storage with server user
  const serverUser = {
    ...existingUser,
    id: account.id,
    email: credentials.email,
    isRegistered: true
  }
  
  Storage.set('user', serverUser)
  
  return serverUser
}
```

### 7.3 Login Flow

```
┌─────────────────────────────────────────────┐
│                 LOGIN (V2)                   │
├─────────────────────────────────────────────┤
│                                             │
│  Email: [________________]                  │
│  Password: [________________]               │
│                                             │
│  [x] Remember me                            │
│                                             │
│  [Login]                                    │
│                                             │
│  [Forgot password?]                         │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 8. SECURITY CONSIDERATIONS

### 8.1 Data Security

```
Local Storage:
- User ID is UUID (not guessable)
- No sensitive data stored (no passwords)
- All data is on user's device

V2 (Server):
- Passwords hashed with bcrypt
- HTTPS only
- JWT tokens for auth
- Rate limiting on auth endpoints
```

### 8.2 XSS Prevention

```javascript
// Always sanitize display names
function sanitizeDisplayName(name) {
  const div = document.createElement('div')
  div.textContent = name
  return div.innerHTML
}

// Use textContent, not innerHTML for user data
element.textContent = user.displayName  // Safe
element.innerHTML = user.displayName    // Unsafe!
```

### 8.3 CSRF Protection (V2)

```javascript
// For API requests
async function apiRequest(endpoint, options = {}) {
  const csrfToken = getCsrfToken()
  
  return fetch(endpoint, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken
    }
  })
}
```

---

## 9. STATE DIAGRAM

```
                    ┌─────────────┐
                    │ First Visit │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  Anonymous  │ ◄────────────────┐
                    │    User     │                  │
                    └──────┬──────┘                  │
                           │                         │
            ┌──────────────┼──────────────┐         │
            ▼              ▼              ▼         │
     ┌──────────┐   ┌──────────┐   ┌──────────┐    │
     │ Play Game│   │ Settings │   │ Register │    │
     └──────────┘   └──────────┘   │   (V2)   │    │
                                   └─────┬────┘    │
                                         │         │
                                         ▼         │
                                   ┌──────────┐    │
                                   │Registered│    │
                                   │   User   │────┘
                                   └─────┬────┘    Logout
                                         │
                                         ▼
                                   ┌──────────┐
                                   │ Premium  │
                                   │  (Paid)  │
                                   └──────────┘
```

---

## 10. API ENDPOINTS (V2)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/register` | POST | Create account |
| `/auth/login` | POST | Login |
| `/auth/logout` | POST | Logout |
| `/auth/refresh` | POST | Refresh token |
| `/auth/forgot-password` | POST | Send reset email |
| `/auth/reset-password` | POST | Reset password |
| `/user/profile` | GET/PUT | Get/Update profile |
| `/user/sync` | POST | Sync local data |
| `/user/delete` | DELETE | Delete account |

---

## 11. REFERENCES

- OWASP Authentication Guidelines
- Web Storage API: MDN
- UUID Generation: crypto.randomUUID()
