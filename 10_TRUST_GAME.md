# TRUST GAME
## Game Specification Document

| Field | Value |
|-------|-------|
| Module ID | `trust-game` |
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
| EN | Send money to grow it - but will they share the profits? |
| TR | Para gönder, büyüsün - ama karı paylaşacaklar mı? |

### 1.2 Concepts Taught

| Concept | Description |
|---------|-------------|
| Trust | Willingness to be vulnerable |
| Reciprocity | Returning favors |
| Investment | Risk for potential greater return |
| Trustworthiness | Honoring trust placed in you |
| Social Capital | Value of trust relationships |

### 1.3 Real-Life Connections

- **Lending Money:** Will your friend pay you back?
- **Business Partnership:** Investing with someone
- **Online Transactions:** Paying before receiving goods
- **Team Projects:** Trusting teammates to do their part

---

## 2. GAME RULES

### 2.1 Basic Rules

```
1. Investor has $10
2. Investor sends any amount (0-10) to Trustee
3. Amount sent is TRIPLED by the bank
4. Trustee decides how much to return to Investor
5. Game ends - payoffs finalized
```

### 2.2 Example Flow

```
Investor sends: $5
Tripled amount: $15 (Trustee receives)
Trustee returns: $7 (generous) or $0 (betrayal)

If returns $7:
  Investor: (10-5) + 7 = $12 ✓ Trust paid off
  Trustee: 15 - 7 = $8

If returns $0:
  Investor: (10-5) + 0 = $5 ✗ Trust betrayed
  Trustee: 15 - 0 = $15
```

### 2.3 Player Information

| Field | Value |
|-------|-------|
| Minimum Players | 2 |
| Maximum Players | 2 |
| Bot Support | Yes |
| Turn-Based | Yes (Investor → Trustee) |

### 2.4 Multiplier

| Default | Range |
|---------|-------|
| 3x | 2x - 4x (configurable) |

---

## 3. ACTIONS

| Action | Description |
|--------|-------------|
| `SEND` | Investor sends amount to Trustee |
| `RETURN` | Trustee returns amount to Investor |

```
SEND: { amount: number }   // 0 to endowment
RETURN: { amount: number } // 0 to tripled amount
```

---

## 4. PAYOFF

```
Investor sends S, Trustee receives 3S
Trustee returns R

Investor payoff: (Endowment - S) + R
Trustee payoff: 3S - R
```

### Examples ($10 endowment, 3x multiplier)

| Sent | Tripled | Returned | Investor | Trustee |
|------|---------|----------|----------|---------|
| $0 | $0 | - | $10 | $0 |
| $5 | $15 | $0 | $5 | $15 |
| $5 | $15 | $8 | $13 | $7 |
| $10 | $30 | $15 | $15 | $15 |

### Game Theory vs Reality

```
Theory (backward induction):
- Trustee returns $0 (self-interest)
- Investor sends $0 (anticipating betrayal)
- Both get starting amounts - no growth

Reality:
- Investors send ~50% on average
- Trustees return ~30-40% of tripled amount
- Trust and reciprocity exist!
```

---

## 5. STATE STRUCTURE

```
GameData {
  endowment: number
  multiplier: number
  investorId: string
  trusteeId: string
  amountSent: number | null
  tripledAmount: number | null
  amountReturned: number | null
}
```

---

## 6. UI DESIGN

### Investor Screen
```
YOU ARE THE INVESTOR
Starting amount: $10
Multiplier: 3x

HOW MUCH WILL YOU SEND?
[$0]========[●]========[$10] → $6

If you send $6:
  → Trustee receives: $18 (6 × 3)
  → You keep: $4
  → You hope they return some of the $18

💡 Can you trust them to share the profits?

[SEND $6]
```

### Trustee Screen
```
YOU ARE THE TRUSTEE
Investor sent: $6
After 3x multiplier: $18

HOW MUCH WILL YOU RETURN?
[$0]========[●]========[$18] → $9

If you return $9:
  → Investor gets: $4 + $9 = $13
  → You keep: $18 - $9 = $9

💡 They trusted you. Will you be trustworthy?

[RETURN $9]
```

---

## 7. BOT STRATEGIES

### Investor Strategies
| ID | Sends | Description |
|----|-------|-------------|
| `cautious` | 20% | Low trust |
| `moderate` | 50% | Moderate trust |
| `trusting` | 80% | High trust |
| `all-in` | 100% | Full trust |

### Trustee Strategies
| ID | Returns | Description |
|----|---------|-------------|
| `betrayer` | 0% | Keeps everything |
| `minimal` | 20% | Token return |
| `fair` | 33% | Returns original sent |
| `reciprocal` | 50% | Equal profit share |
| `generous` | 66% | Gives investor profit |

---

## 8. ACHIEVEMENTS

| ID | Name | Condition |
|----|------|-----------|
| `tg-trust` | First Trust | Send 50%+ as Investor |
| `tg-trustworthy` | Trustworthy | Return 50%+ as Trustee |
| `tg-betrayed` | Betrayed | Get $0 back after sending |
| `tg-win-win` | Win-Win | Both players profit |
| `tg-all-in` | All In | Send 100% as Investor |

---

## 9. i18n KEYS

### English
```json
"trust-game": {
  "name": "Trust Game",
  "description": "Send to grow, hope for return",
  "roles": { "investor": "Investor", "trustee": "Trustee" },
  "labels": {
    "send": "Send",
    "tripled": "Tripled Amount",
    "return": "Return"
  },
  "insights": {
    "trust": "Trust creates value - the pie grows!",
    "risk": "But trust can be exploited...",
    "reciprocity": "Trustworthy behavior builds social capital."
  }
}
```

---

## 10. PEDAGOGICAL VALUE

```
Trust Game teaches:

1. TRUST CREATES VALUE
   - Sending money makes the pie bigger (3x)
   - No trust = no growth = both lose

2. VULNERABILITY
   - Investor takes real risk
   - Could lose what they sent

3. RECIPROCITY
   - Most trustees return something
   - Social norms create trustworthiness

4. REPEATED GAMES
   - Trust builds over time
   - Betrayal has long-term costs
```

---

## 11. REFERENCES

- Berg, Dickhaut, McCabe (1995) - Original Trust Game
- Camerer & Weigelt (1988)
- Johnson & Mislin (2011) - Meta-analysis
