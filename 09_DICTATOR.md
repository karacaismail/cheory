# DICTATOR GAME
## Game Specification Document

| Field | Value |
|-------|-------|
| Module ID | `dictator` |
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
| EN | You decide how to split the money - the other player must accept! |
| TR | Parayı nasıl böleceğine sen karar ver - diğer oyuncu kabul etmek zorunda! |

### 1.2 Concepts Taught

| Concept | Description |
|---------|-------------|
| Altruism | Do people give when they don't have to? |
| Fairness Norms | Internalized sense of fair division |
| Power Asymmetry | What happens with unchecked power? |
| Social Preferences | Caring about others' outcomes |

### 1.3 Real-Life Connections

- **Found Money:** You find $10, friend is with you
- **Charity:** Giving when there's no obligation
- **Inheritance:** Deciding how to share with siblings
- **Tipping:** No requirement but social expectation

### 1.4 Comparison with Ultimatum

```
Ultimatum: Responder can reject → Proposer offers ~40%
Dictator: Responder cannot reject → What will Dictator give?

This comparison isolates the effect of rejection power.
```

---

## 2. GAME RULES

### 2.1 Basic Rules

```
1. A pot of $10 is to be divided
2. Dictator decides how much to give Receiver
3. Receiver MUST accept whatever is given
4. No rejection possible - Dictator has full control
```

### 2.2 Player Information

| Field | Value |
|-------|-------|
| Minimum Players | 2 |
| Maximum Players | 2 |
| Bot Support | Yes |
| One-Sided | Yes (only Dictator acts) |

### 2.3 Game Theory vs Reality

```
Theory: Dictator gives $0 (rational self-interest)
Reality: Average gift ~28% (people are not purely selfish)
```

---

## 3. ACTIONS

| Action | Description |
|--------|-------------|
| `ALLOCATE` | Dictator decides split |

```
ALLOCATE: { gift: number }  // Amount given to Receiver
```

---

## 4. PAYOFF

```
Dictator gets: Pot - Gift
Receiver gets: Gift

No rejection possible - split is final.
```

### Examples ($10 pot)

| Gift | Dictator | Receiver |
|------|----------|----------|
| $0 | $10 | $0 |
| $3 | $7 | $3 |
| $5 | $5 | $5 |

---

## 5. STATE STRUCTURE

```
GameData {
  pot: number
  dictatorId: string
  receiverId: string
  gift: number | null
}
```

---

## 6. UI DESIGN

### Dictator Screen
```
YOU ARE THE DICTATOR
THE POT: $10

How much will you GIVE?
[$0]========[●]========[$10] → $3

You keep: $7 | They get: $3

💡 They cannot reject. This is your choice alone.

[CONFIRM ALLOCATION]
```

### Receiver Screen (Waiting)
```
WAITING FOR THE DICTATOR...
The Dictator is deciding how to split $10.
You must accept whatever they give.
```

### Receiver Screen (Result)
```
THE DICTATOR HAS DECIDED
You received: $3 out of $10 (30%)
The Dictator kept: $7

💡 Unlike Ultimatum, you had no power to reject.
```

---

## 7. BOT STRATEGIES

### Dictator Strategies
| ID | Gift % | Description |
|----|--------|-------------|
| `selfish` | 0% | Keeps everything |
| `minimal` | 10% | Token gesture |
| `fair` | 30% | Average real behavior |
| `generous` | 50% | Equal split |
| `altruist` | 70%+ | Gives more than keeps |

### Receiver
No strategy needed - must accept.

---

## 8. ACHIEVEMENTS

| ID | Name | Condition |
|----|------|-----------|
| `dg-first` | First Allocation | Complete first game |
| `dg-generous` | Generous Soul | Give 50%+ five times |
| `dg-altruist` | True Altruist | Give more than you keep |
| `dg-fair` | Fair Minded | Give exactly 50% |

---

## 9. i18n KEYS

### English
```json
"dictator": {
  "name": "Dictator Game",
  "description": "You decide - they must accept",
  "roles": { "dictator": "Dictator", "receiver": "Receiver" },
  "labels": {
    "pot": "The Pot",
    "gift": "Your Gift",
    "keep": "You Keep"
  },
  "insights": {
    "no_rejection": "Unlike Ultimatum, they cannot reject.",
    "altruism": "Do you give when you don't have to?",
    "comparison": "People give less in Dictator than Ultimatum. Why?"
  }
}
```

---

## 10. EXPERIMENTAL VALUE

```
Dictator Game is crucial for behavioral economics because:

1. Isolates "pure" fairness preferences
2. No strategic concerns (rejection impossible)
3. Compares with Ultimatum to show rejection power effect
4. Tests altruism vs self-interest

Typical findings:
- Mean allocation: ~28% (vs ~42% in Ultimatum)
- ~36% give nothing
- ~17% split 50-50
- Social distance matters (anonymous = give less)
```

---

## 11. REFERENCES

- Kahneman, Knetsch, Thaler (1986)
- Forsythe et al. (1994)
- Engel (2011) - Meta-analysis
