# PLATFORM RULES
## Technical Specification Document

| Field | Value |
|-------|-------|
| Document ID | `PLATFORM_RULES` |
| Version | 1.0.0 |
| Tier | 3 - Technical |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Purpose

This document defines platform policies, business rules, content guidelines, and operational constraints for the Cheory game theory education platform.

### 1.2 Scope

- Content policies
- User conduct rules
- Monetization rules
- Progression system
- Scoring and achievements
- Accessibility requirements
- Legal compliance

---

## 2. CONTENT POLICIES

### 2.1 Age Appropriateness

```
Target Audience: 10+ years old
Content Rating: Everyone (E)

ALLOWED:
✓ Educational game theory content
✓ Friendly competition
✓ Strategic thinking challenges
✓ Age-appropriate language
✓ Abstract representations of conflict

NOT ALLOWED:
✗ Violence or gore
✗ Sexual content
✗ Profanity
✗ Gambling mechanics (real money)
✗ Substance references
✗ Discrimination or hate speech
```

### 2.2 Language Guidelines

```
Tone: Friendly, educational, encouraging

PREFERRED:
- "You lost this round" instead of "You failed"
- "Try again!" instead of "Game over"
- "Your opponent" instead of "Enemy"
- "Challenge" instead of "Fight"

BOT NAMES:
- Friendly names (Alex, Sam, Jordan)
- No threatening or negative names
- No real celebrity/political names
```

### 2.3 Game Themes

```
Prisoner's Dilemma:
- "Two friends" not "prisoners"
- "Cooperation/Defection" not "Confess/Betray"
- Focus on trust, not crime

Hawk-Dove:
- Animal metaphor, not violence
- "Aggressive/Peaceful" strategies
- No blood or injury depiction

Ultimatum/Dictator:
- "Sharing" not "power dynamics"
- Fair play emphasis
- No exploitation glorification
```

---

## 3. USER CONDUCT

### 3.1 Display Name Rules

```javascript
const DisplayNameRules = {
  minLength: 2,
  maxLength: 20,
  
  // Allowed characters
  pattern: /^[a-zA-Z0-9_\- ]+$/,
  
  // Blocked patterns
  blocked: [
    /admin/i,
    /moderator/i,
    /cheory/i,
    /official/i,
    // Profanity filter list...
  ],
  
  // Reserved names
  reserved: ['System', 'Bot', 'AI', 'Computer', 'Admin']
}

function validateDisplayName(name) {
  if (name.length < DisplayNameRules.minLength) return false
  if (name.length > DisplayNameRules.maxLength) return false
  if (!DisplayNameRules.pattern.test(name)) return false
  if (DisplayNameRules.blocked.some(p => p.test(name))) return false
  if (DisplayNameRules.reserved.includes(name)) return false
  return true
}
```

### 3.2 Fair Play

```
EXPECTED BEHAVIOR:
- Complete games you start
- Don't exploit bugs
- Don't use automation/bots
- Respect other players (multiplayer V2)

CONSEQUENCES (V2 Multiplayer):
- Warning for first offense
- Temporary suspension for repeated offense
- Permanent ban for severe violations
```

---

## 4. MONETIZATION RULES

### 4.1 Tier Structure

```javascript
const TierConfig = {
  free: {
    games: [
      'prisoners-dilemma',
      'stag-hunt', 
      'matching-pennies',
      'ultimatum',
      'dictator',
      'rps'
    ],
    features: {
      tutorials: true,
      localStats: true,
      exportData: true,
      achievements: true,
      botDifficulties: ['easy', 'medium'],
      savedGames: 10
    },
    ads: true
  },
  
  premium: {
    games: 'all',
    features: {
      tutorials: true,
      localStats: true,
      exportData: true,
      achievements: true,
      cloudSync: true,
      botDifficulties: ['easy', 'medium', 'hard'],
      savedGames: 100,
      customStrategies: true,
      advancedAnalytics: true
    },
    ads: false,
    price: {
      monthly: 4.99,
      yearly: 39.99  // ~33% savings
    }
  }
}
```

### 4.2 Payment Rules

```
REFUND POLICY:
- 7-day money-back guarantee
- No questions asked
- Automatic downgrade to free tier

SUBSCRIPTION:
- Cancel anytime
- Access continues until period ends
- No pro-rating for partial periods

PROHIBITED:
- No loot boxes
- No gacha mechanics
- No pay-to-win advantages
- No predatory pricing
```

### 4.3 Ad Guidelines (Free Tier)

```
AD PLACEMENT:
✓ Banner at bottom (non-intrusive)
✓ Between games (interstitial)
✓ Before tutorial (skippable after 5s)

✗ No mid-game interruptions
✗ No ads during active gameplay
✗ No auto-playing video with sound
✗ No fake "X" buttons

FREQUENCY:
- Max 1 interstitial per 5 games
- Max 1 banner visible at a time
- Respect user's paid removal
```

---

## 5. PROGRESSION SYSTEM

### 5.1 Experience Points (XP)

```javascript
const XPRules = {
  // Base XP for completing games
  gameComplete: 10,
  
  // Bonuses
  bonuses: {
    win: 5,
    perfectScore: 10,
    firstTimeGame: 20,
    dailyFirst: 15,
    streak3: 25,
    streak7: 50
  },
  
  // Multipliers
  multipliers: {
    easy: 0.8,
    medium: 1.0,
    hard: 1.5
  },
  
  // Daily cap (prevent grinding)
  dailyCap: 500
}

function calculateXP(gameResult) {
  let xp = XPRules.gameComplete
  
  if (gameResult.won) xp += XPRules.bonuses.win
  if (gameResult.perfectScore) xp += XPRules.bonuses.perfectScore
  if (gameResult.isFirstTime) xp += XPRules.bonuses.firstTimeGame
  
  xp *= XPRules.multipliers[gameResult.difficulty]
  
  return Math.min(xp, getRemainingDailyCap())
}
```

### 5.2 Leveling

```javascript
const LevelRules = {
  maxLevel: 100,
  
  // XP required for each level
  xpForLevel: (level) => {
    // Polynomial scaling: 100, 250, 450, 700, 1000...
    return Math.floor(100 * level + 50 * Math.pow(level, 1.5))
  },
  
  // Total XP for level
  totalXPForLevel: (level) => {
    let total = 0
    for (let i = 1; i < level; i++) {
      total += LevelRules.xpForLevel(i)
    }
    return total
  }
}

// Level 1: 0 XP
// Level 2: 100 XP
// Level 5: 850 XP
// Level 10: 3,500 XP
// Level 50: 75,000 XP
// Level 100: 350,000 XP
```

### 5.3 Level Rewards

```javascript
const LevelRewards = {
  5: { type: 'avatar', id: 'avatar-scholar' },
  10: { type: 'badge', id: 'badge-strategist' },
  15: { type: 'avatar', id: 'avatar-thinker' },
  20: { type: 'title', id: 'Game Theorist' },
  25: { type: 'avatar', id: 'avatar-expert' },
  50: { type: 'title', id: 'Nash Master' },
  100: { type: 'badge', id: 'badge-legend' }
}
```

---

## 6. ACHIEVEMENT RULES

### 6.1 Achievement Categories

```javascript
const AchievementCategories = {
  gameplay: {
    name: 'Gameplay',
    icon: '🎮',
    achievements: [
      'first-game',
      'ten-games',
      'hundred-games',
      'win-streak-3',
      'win-streak-10'
    ]
  },
  
  mastery: {
    name: 'Mastery',
    icon: '🏆',
    achievements: [
      'all-games-played',
      'all-games-won',
      'beat-hard-bot',
      'perfect-game'
    ]
  },
  
  learning: {
    name: 'Learning',
    icon: '📚',
    achievements: [
      'complete-tutorial',
      'all-tutorials',
      'try-all-strategies',
      'analyze-game'
    ]
  },
  
  social: {
    name: 'Social',
    icon: '👥',
    achievements: [
      'share-result',
      'invite-friend',
      'play-with-friend'
    ]
  }
}
```

### 6.2 Achievement Rules

```
UNLOCK CONDITIONS:
- Must be earned through gameplay
- Cannot be purchased
- Cannot be transferred

DISPLAY:
- Show unlock notification
- Add to profile
- Track progress for progressive achievements

HIDDEN ACHIEVEMENTS:
- Some achievements are secret
- Revealed only when unlocked
- Add discovery element
```

---

## 7. SCORING RULES

### 7.1 Score Standardization

```javascript
const ScoreRules = {
  // All games normalize scores to 0-100 scale for comparison
  normalizeScore: (rawScore, gameId) => {
    const config = GameScoreConfigs[gameId]
    const range = config.maxScore - config.minScore
    return Math.round(((rawScore - config.minScore) / range) * 100)
  },
  
  // Game-specific configs
  GameScoreConfigs: {
    'prisoners-dilemma': { minScore: 0, maxScore: 5 },
    'iterated-pd': { minScore: 0, maxScore: 50 },  // 10 rounds
    'ultimatum': { minScore: 0, maxScore: 10 },
    'public-goods': { minScore: 0, maxScore: 30 }
  }
}
```

### 7.2 Leaderboard Rules (V2)

```
RANKING CRITERIA:
1. Win rate (primary)
2. Total games played (tiebreaker)
3. Average score (tiebreaker)

ELIGIBILITY:
- Minimum 10 games played
- No banned/cheating flags
- Active in last 30 days

RESET:
- Monthly leaderboards
- All-time separate
```

---

## 8. ACCESSIBILITY REQUIREMENTS

### 8.1 WCAG Compliance

```
Target: WCAG 2.1 Level AA

REQUIRED:
✓ Keyboard navigation for all interactions
✓ Screen reader compatibility
✓ Minimum contrast ratio 4.5:1
✓ Focus indicators visible
✓ Text resizable to 200%
✓ No content that flashes > 3 times/second

RECOMMENDED:
- Color not sole indicator
- Captions for audio (if any)
- Skip navigation links
- Consistent navigation
```

### 8.2 Implementation

```css
/* Focus visible */
:focus-visible {
  outline: 2px solid var(--brand-primary);
  outline-offset: 2px;
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

/* High contrast */
@media (prefers-contrast: high) {
  :root {
    --text-primary: #000000;
    --bg-primary: #ffffff;
    --brand-primary: #0000ff;
  }
}
```

### 8.3 Keyboard Shortcuts

```javascript
const KeyboardShortcuts = {
  global: {
    '?': 'Show help',
    'Escape': 'Close modal / Cancel',
    '/': 'Focus search',
    'm': 'Toggle menu'
  },
  
  game: {
    '1-9': 'Quick select option',
    'Enter': 'Confirm selection',
    'Space': 'Pause/Resume',
    'r': 'Restart game',
    'q': 'Quit to menu'
  }
}
```

---

## 9. LEGAL COMPLIANCE

### 9.1 Data Privacy (GDPR/CCPA)

```
USER RIGHTS:
- Access their data (export feature)
- Delete their data (factory reset)
- Opt-out of analytics
- Withdraw consent

DATA COLLECTED:
- Game history (local only)
- User preferences (local only)
- Analytics (with consent, anonymized)

DATA NOT COLLECTED:
- Personal information (V1)
- Payment info (handled by provider)
- Location data
- Device identifiers
```

### 9.2 Terms of Service Summary

```
KEY POINTS:
1. Service provided "as is"
2. User owns their data
3. No guaranteed uptime
4. Content may change
5. Pricing may change with notice
6. Account may be terminated for violations

AGE REQUIREMENT:
- 13+ to create account (V2)
- 18+ for purchases (or parent consent)
```

### 9.3 Cookie Policy

```
NECESSARY COOKIES:
- Session management
- User preferences
- Authentication (V2)

OPTIONAL COOKIES:
- Analytics (with consent)
- Marketing (with consent)

COOKIE-FREE OPERATION:
- V1 uses localStorage only
- No third-party cookies
```

---

## 10. OPERATIONAL LIMITS

### 10.1 Rate Limits

```javascript
const RateLimits = {
  // Actions per minute
  gameActions: 60,
  
  // API calls (V2)
  apiCalls: 100,
  
  // Storage
  maxLocalStorage: 5 * 1024 * 1024,  // 5MB
  maxGameHistory: 100,               // games
  
  // Session
  sessionTimeout: 24 * 60 * 60 * 1000  // 24 hours
}
```

### 10.2 Content Limits

```javascript
const ContentLimits = {
  displayName: {
    min: 2,
    max: 20
  },
  
  savedGames: {
    free: 10,
    premium: 100
  },
  
  exportSize: {
    max: 10 * 1024 * 1024  // 10MB
  }
}
```

---

## 11. VERSIONING & UPDATES

### 11.1 Version Policy

```
SEMANTIC VERSIONING: Major.Minor.Patch

Major: Breaking changes, data migration needed
Minor: New features, backward compatible
Patch: Bug fixes, no data changes

NOTIFICATION:
- Major: 30-day advance notice
- Minor: Changelog on update
- Patch: Silent update
```

### 11.2 Data Migration

```javascript
const MigrationRules = {
  // Always preserve user data
  preserveUserData: true,
  
  // Backup before migration
  backupFirst: true,
  
  // Rollback on failure
  rollbackOnError: true,
  
  // Version compatibility
  supportedVersions: ['0.9.x', '1.0.x', '1.1.x']
}
```

---

## 12. SUPPORT POLICIES

### 12.1 Support Channels

```
FREE TIER:
- FAQ / Help documentation
- Community forums (V2)
- Email (72-hour response)

PREMIUM TIER:
- All free tier options
- Priority email (24-hour response)
- Bug reports priority
```

### 12.2 Bug Reporting

```
REQUIRED INFO:
- Browser/OS version
- Steps to reproduce
- Expected vs actual behavior
- Screenshots if applicable

PRIORITY LEVELS:
- Critical: App unusable
- High: Major feature broken
- Medium: Minor feature issue
- Low: Cosmetic/enhancement
```

---

## 13. REFERENCES

- WCAG 2.1 Guidelines
- GDPR Compliance Checklist
- COPPA Requirements
- App Store Guidelines (for future mobile)
