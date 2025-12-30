# ROCK PAPER SCISSORS
## Game Specification Document

| Field | Value |
|-------|-------|
| Module ID | `rps` |
| Version | 1.0.0 |
| Category | zero-sum |
| Difficulty | easy |
| Tier | free |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Short Description

| Language | Description |
|----------|-------------|
| EN | The classic game - Rock, Paper, Scissors. Can you predict your opponent? |
| TR | Klasik oyun - Taş, Kağıt, Makas. Rakibini tahmin edebilir misin? |

### 1.2 Concepts Taught

| Concept | Description |
|---------|-------------|
| Zero-Sum | One player's win = another's loss |
| Cyclic Dominance | Rock > Scissors > Paper > Rock |
| Mixed Strategy | Why random is optimal |
| Pattern Recognition | Exploiting human predictability |

---

## 2. GAME RULES

### 2.1 Basic Rules

```
Rock beats Scissors (crushes)
Scissors beats Paper (cuts)
Paper beats Rock (covers)
Same = Tie
```

### 2.2 Player Information

| Field | Value |
|-------|-------|
| Players | 2 |
| Bot Support | Yes |
| Simultaneous | Yes |

---

## 3. PAYOFF

```
Winner: +1 | Loser: -1 | Tie: 0
```

---

## 4. UI DESIGN

```
  [✊ ROCK]  [📄 PAPER]  [✌️ SCISSORS]
```

---

## 5. BOT STRATEGIES

| ID | Description |
|----|-------------|
| `random` | True 1/3 each (optimal) |
| `rock-bias` | 50% rock |
| `copycat` | Plays your last move |
| `counter` | Beats your last move |
| `pattern` | Detects patterns |

---

## 6. GAME MODES

| Mode | Rounds |
|------|--------|
| Single | 1 |
| Best of 3 | First to 2 |
| Best of 5 | First to 3 |

---

## 7. ACHIEVEMENTS

| ID | Name | Condition |
|----|------|-----------|
| `rps-streak-3` | Triple Threat | Win 3 in a row |
| `rps-streak-5` | Unstoppable | Win 5 in a row |

---

## 8. i18n KEYS

### English
```json
"rps": {
  "name": "Rock Paper Scissors",
  "choices": { "rock": "Rock", "paper": "Paper", "scissors": "Scissors" },
  "results": { "win": "You win!", "lose": "You lose!", "tie": "Tie!" }
}
```

---

## 9. PEDAGOGICAL VALUE

```
- Zero-sum game example
- Mixed strategy Nash equilibrium: 1/3 each
- Humans show patterns (rock is common first choice)
- Comparison with Matching Pennies
```
