# PUBLIC GOODS GAME
## Game Specification Document

| Field | Value |
|-------|-------|
| Module ID | `public-goods` |
| Version | 1.0.0 |
| Category | social |
| Difficulty | medium |
| Tier | premium |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Short Description

| Language | Description |
|----------|-------------|
| EN | Contribute to the common pool - but will others do their share? |
| TR | Ortak havuza katkıda bulun - ama diğerleri paylarını yapacak mı? |

### 1.2 Concepts Taught

| Concept | Description |
|---------|-------------|
| Free Riding | Why individuals might not contribute to public goods |
| Social Dilemma | Individual vs collective rationality |
| Multiplier Effect | How contributions grow when pooled |
| Tragedy of Commons | When everyone takes, no one benefits |
| Civic Duty | Why contributing matters for society |

### 1.3 Real-Life Connections

- **Class Project:** Everyone contributes effort, grade is shared
- **Neighborhood Cleanup:** Will everyone pitch in?
- **Public Radio:** Donations fund what everyone enjoys
- **Climate Action:** Reducing emissions benefits all, costs individuals
- **Team Fundraiser:** Everyone sells, team shares the prize

---

## 2. GAME RULES

### 2.1 Basic Rules

```
1. Each player starts with an endowment (e.g., 10 tokens)
2. Players secretly choose how much to contribute to the pool
3. The pool is multiplied by a factor (e.g., 2x)
4. The multiplied pool is divided equally among ALL players
5. Final payoff = (endowment - contribution) + (share of pool)
```

### 2.2 Player Information

| Field | Value |
|-------|-------|
| Minimum Players | 2 |
| Maximum Players | 4 |
| Optimal Players | 4 |
| Bot Support | Yes |
| Simultaneous Moves | Yes |

### 2.3 Round Structure

| Field | Value |
|-------|-------|
| Default Rounds | 1 |
| Min Rounds | 1 |
| Max Rounds | 10 |

### 2.4 Win Condition

```
Highest final payoff wins.
Note: The "winner" may be the free-rider who contributed nothing!
```

---

## 3. ACTIONS

### 3.1 Action List

| Action | Description | Condition |
|--------|-------------|-----------|
| `CONTRIBUTE` | Player sets contribution amount | Player hasn't contributed this round |

### 3.2 Action Details

```
Action {
  type: 'CONTRIBUTE'
  playerId: string
  payload: {
    amount: number  // 0 to endowment
  }
}
```

---

## 4. PAYOFF / SCORING

### 4.1 Payoff Formula

```
Pool = sum of all contributions × multiplier
Share = Pool / number of players
Payoff = (endowment - contribution) + Share
```

### 4.2 Example (4 players, 10 tokens each, 2x multiplier)

```
All contribute 10: Pool=80, Share=20, Payoff=20 each ✓
All contribute 0: Pool=0, Share=0, Payoff=10 each
One free-rider: Pool=60, Share=15
  - Contributors: 15 each
  - Free-rider: 25 ← Highest!
```

### 4.3 The Dilemma

```
Individual Rationality: Contribute 0
Collective Rationality: Contribute all
Nash Equilibrium: Everyone contributes 0
Pareto Optimum: Everyone contributes all
```

---

## 5. STATE STRUCTURE

### 5.1 Game-Specific Data

```
GameData {
  endowment: number
  multiplier: number
  contributions: { [playerId]: number | null }
  totalContributed: number | null
  pool: number | null
  sharePerPlayer: number | null
}
```

---

## 6. UI DESIGN

### 6.1 Main Game Layout

```
+-----------------------------------------------+
|  PUBLIC GOODS  |  Round 1/1  |  4 Players     |
+-----------------------------------------------+
|  YOUR TOKENS: 10                              |
|  POOL: $0  |  MULTIPLIER: 2x  |  SPLIT: 4    |
+-----------------------------------------------+
|  HOW MUCH WILL YOU CONTRIBUTE?                |
|  [0]==========[●]============[10]             |
|              5 tokens                         |
|  You keep: 5  |  Your share: ???              |
+-----------------------------------------------+
|  [CONFIRM CONTRIBUTION]                       |
+-----------------------------------------------+
```

### 6.2 Results Display

```
| Player  | Gave | Kept | Share | Total |
|---------|------|------|-------|-------|
| You     |  10  |   0  |  14   |  14   |
| Alex    |  10  |   0  |  14   |  14   |
| Sam     |   8  |   2  |  14   |  16   |
| Jordan  |   0  |  10  |  14   |  24   | 🏆
```

### 6.3 Components

| Component | Tag | Description |
|-----------|-----|-------------|
| Endowment Display | `<pg-endowment>` | Starting tokens |
| Pool Display | `<pg-pool>` | Current pool |
| Contribution Slider | `<pg-slider>` | Amount selector |
| Results Table | `<pg-results>` | Final breakdown |

---

## 7. BOT STRATEGIES

| ID | Name | Difficulty | Behavior |
|----|------|------------|----------|
| `generous` | Generous | Easy | Always 100% |
| `fair` | Fair | Easy | Always 50% |
| `random` | Random | Easy | Random 0-100% |
| `free-rider` | Free Rider | Medium | Always 0% |
| `conditional` | Conditional | Hard | Matches group average |

---

## 8. TUTORIAL

| Step | Title | Goal |
|------|-------|------|
| 1 | The Setup | Explain endowment and pool |
| 2 | The Multiplier | Show how pool grows |
| 3 | The Split | Equal division among all |
| 4 | The Dilemma | Why not to contribute |
| 5 | The Lesson | Free-rider effect |

---

## 9. TEST SCENARIOS

#### Full Cooperation
```
GIVEN: 4 players, 10 tokens each, 2x multiplier
WHEN: All contribute 10
THEN: Each gets 20 tokens
```

#### One Free Rider
```
GIVEN: 4 players, 10 tokens each, 2x multiplier
WHEN: 3 contribute 10, 1 contributes 0
THEN: Contributors get 15, Free rider gets 25
```

---

## 10. ACHIEVEMENTS

| ID | Name | Description |
|----|------|-------------|
| `pg-generous` | Big Giver | Contribute 100% for 5 games |
| `pg-free-rider` | Tempted | Win by contributing nothing |
| `pg-full-coop` | Dream Team | All players contribute 100% |

---

## 11. i18n KEYS

### English (Primary)
```json
"public-goods": {
  "name": "Public Goods Game",
  "description": "Contribute to the common pool",
  "labels": {
    "endowment": "Your Tokens",
    "pool": "Common Pool",
    "contribution": "Your Contribution"
  },
  "insights": {
    "free_rider": "Contributing nothing is individually rational, but collectively harmful."
  }
}
```

### Turkish
```json
"public-goods": {
  "name": "Kamu Malı Oyunu",
  "description": "Ortak havuza katkıda bulun"
}
```

---

## 12. REFERENCES

- Olson, M. - "The Logic of Collective Action"
- Hardin, G. - "Tragedy of the Commons"
- Ostrom, E. - "Governing the Commons"
