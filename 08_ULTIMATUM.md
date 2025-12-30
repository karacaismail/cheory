# ULTIMATUM GAME
## Game Specification Document

| Field | Value |
|-------|-------|
| Module ID | `ultimatum` |
| Version | 1.0.0 |
| Category | bargaining |
| Difficulty | easy |
| Tier | free |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Short Description

| Language | Description |
|----------|-------------|
| EN | Split the money - but if rejected, nobody gets anything! |
| TR | Parayı paylaş - reddedilirse kimse bir şey alamaz! |

### 1.2 Concepts Taught

| Concept | Description |
|---------|-------------|
| Fairness | People reject unfair offers at personal cost |
| Bargaining Power | Proposer has power but risks rejection |
| Bounded Rationality | "Rational" tiny offers get rejected |
| Social Norms | Cultural expectations of fair division |

### 1.3 Real-Life Connections

- **Splitting a Gift:** Dividing birthday money with sibling
- **Job Negotiations:** Take it or leave it offers
- **Business Deals:** Final offer negotiations

---

## 2. GAME RULES

### 2.1 Basic Rules

```
1. A pot of $10 is to be divided
2. Proposer makes an offer: "I take X, you get Y"
3. Responder either ACCEPTS or REJECTS
4. If ACCEPTED: Both get their shares
5. If REJECTED: Both get NOTHING
```

### 2.2 Player Information

| Field | Value |
|-------|-------|
| Minimum Players | 2 |
| Maximum Players | 2 |
| Bot Support | Yes |
| Turn-Based | Yes (Proposer → Responder) |

### 2.3 Game Theory vs Reality

```
Theory: Proposer offers $0.01, Responder accepts
Reality: Average offer ~42%, offers <20% often rejected
```

---

## 3. ACTIONS

| Action | Description |
|--------|-------------|
| `PROPOSE` | Proposer makes offer (0 to pot) |
| `RESPOND` | Responder accepts or rejects |

```
PROPOSE: { offer: number }
RESPOND: { accept: boolean }
```

---

## 4. PAYOFF

```
ACCEPTED: Proposer gets (Pot - Offer), Responder gets Offer
REJECTED: Both get $0
```

### Examples ($10 pot)

| Offer | Response | Proposer | Responder |
|-------|----------|----------|-----------|
| $5 | Accept | $5 | $5 |
| $2 | Accept | $8 | $2 |
| $1 | Reject | $0 | $0 |

---

## 5. STATE STRUCTURE

```
GameData {
  pot: number
  proposerId: string
  responderId: string
  offer: number | null
  response: 'accept' | 'reject' | null
}
```

---

## 6. UI DESIGN

### Proposer Screen
```
THE POT: $10
MAKE YOUR OFFER: [$0]====[●]====[$10] → $4
You get: $6 | They get: $4
⚠️ If rejected, BOTH get $0!
[MAKE OFFER]
```

### Responder Screen
```
OFFER: $4 out of $10 (40%)
[✓ ACCEPT: Get $4] [✗ REJECT: Both get $0]
```

---

## 7. BOT STRATEGIES

### Proposer
| ID | Offers |
|----|--------|
| `generous` | 50% |
| `fair` | 40% |
| `greedy` | 20% |

### Responder
| ID | Accepts if |
|----|------------|
| `accepting` | > 0% |
| `fair` | ≥ 30% |
| `strict` | ≥ 40% |

---

## 8. ACHIEVEMENTS

| ID | Name | Condition |
|----|------|-----------|
| `ug-fair` | Fair Player | 5 offers of 40%+ |
| `ug-punisher` | Punisher | Reject unfair offer |
| `ug-generous` | Generous | Offer 50%+ |

---

## 9. i18n KEYS

### English
```json
"ultimatum": {
  "name": "Ultimatum Game",
  "description": "Split the money - take it or leave it!",
  "roles": { "proposer": "Proposer", "responder": "Responder" },
  "actions": { "accept": "Accept", "reject": "Reject" },
  "outcomes": {
    "accepted": "Deal! Money divided.",
    "rejected": "Rejected! Both get nothing."
  }
}
```

---

## 10. REFERENCES

- Güth, Schmittberger, Schwarze (1982)
- Camerer, C. - "Behavioral Game Theory"
