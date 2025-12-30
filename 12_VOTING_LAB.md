# VOTING LAB
## Game Specification Document

| Field | Value |
|-------|-------|
| Module ID | `voting-lab` |
| Version | 1.0.0 |
| Category | voting |
| Difficulty | hard |
| Tier | premium |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Short Description

| Language | Description |
|----------|-------------|
| EN | Explore different voting systems - see how the same votes can produce different winners! |
| TR | Farklı oylama sistemlerini keşfet - aynı oyların nasıl farklı kazananlar çıkardığını gör! |

### 1.2 Concepts Taught

| Concept | Description |
|---------|-------------|
| Voting Systems | Plurality, Ranked, Approval, etc. |
| Arrow's Impossibility | No perfect voting system exists |
| Strategic Voting | Sometimes voting honestly isn't optimal |
| Condorcet Paradox | Cyclic preferences in groups |
| Majority vs Plurality | Difference between 50%+ and most votes |

### 1.3 Real-Life Connections

- **School Elections:** Class president voting
- **Family Decisions:** Where to go for dinner
- **Political Elections:** Different countries use different systems
- **Award Shows:** How winners are selected

---

## 2. GAME MODES

### 2.1 Available Voting Systems

| System | Description |
|--------|-------------|
| Plurality | Most votes wins (First Past the Post) |
| Runoff | Top 2 face off if no majority |
| Ranked Choice (IRV) | Eliminate last, redistribute |
| Borda Count | Points by ranking position |
| Approval | Vote for all you approve |
| Condorcet | Head-to-head winner |

### 2.2 Player Information

| Field | Value |
|-------|-------|
| Minimum Players | 3 |
| Maximum Players | 10 |
| Optimal Players | 5 |
| Bot Support | Yes |

---

## 3. GAME STRUCTURE

### 3.1 Setup Phase
```
1. Choose number of candidates (3-5)
2. Choose voting system(s) to compare
3. Each player gets preferences (or sets them)
```

### 3.2 Voting Phase
```
1. All players submit their votes/rankings
2. System calculates winner
3. If comparing systems: show different outcomes
```

### 3.3 Analysis Phase
```
1. Show vote distribution
2. Reveal winner under each system
3. Highlight paradoxes/differences
```

---

## 4. VOTING SYSTEMS DETAIL

### 4.1 Plurality (First Past the Post)

```
Each voter picks ONE candidate
Most votes wins (no majority needed)

Example (15 voters, 3 candidates):
A: 6 votes | B: 5 votes | C: 4 votes
Winner: A (with only 40%)

Problem: Vote splitting, spoiler effect
```

### 4.2 Ranked Choice (Instant Runoff)

```
Voters rank all candidates
Count first-choice votes
If no majority: eliminate last place
Redistribute eliminated candidate's votes
Repeat until majority

Example:
Round 1: A:6, B:5, C:4 → C eliminated
C's voters had B as 2nd choice
Round 2: A:6, B:9 → B wins!

Different winner than plurality!
```

### 4.3 Borda Count

```
Points based on ranking position
N candidates → 1st gets N-1, 2nd gets N-2, etc.

Example (3 candidates, 15 voters):
Each 1st place = 2 pts, 2nd = 1 pt, 3rd = 0 pts

If A is polarizing (6 first, 9 last):
A: 6×2 + 0×1 + 9×0 = 12 pts

If B is compromise (5 first, 10 second):
B: 5×2 + 10×1 + 0×0 = 20 pts

B wins despite fewer first-place votes!
```

### 4.4 Approval Voting

```
Vote for ALL candidates you "approve"
Most approvals wins

Example:
Voter 1: Approves A, B
Voter 2: Approves B, C
Voter 3: Approves A, C
...

Favors consensus candidates over polarizing ones
```

### 4.5 Condorcet Method

```
Compare each pair head-to-head
Condorcet winner: beats ALL others 1-on-1

Example:
A vs B: A wins (8-7)
A vs C: A wins (9-6)
B vs C: B wins (10-5)
Condorcet winner: A

Condorcet Paradox: Sometimes no Condorcet winner exists!
A beats B, B beats C, C beats A → Cycle!
```

---

## 5. STATE STRUCTURE

```
GameData {
  candidates: string[]
  votingSystems: VotingSystem[]
  votes: {
    [playerId]: Vote  // Format depends on system
  }
  results: {
    [system]: {
      winner: string
      breakdown: object
    }
  }
}

Vote (Plurality): { choice: string }
Vote (Ranked): { ranking: string[] }
Vote (Approval): { approved: string[] }
```

---

## 6. UI DESIGN

### Plurality Voting
```
VOTE FOR ONE CANDIDATE

  [🅰️ ALICE]  [🅱️ BOB]  [🅲️ CAROL]

Your vote: [ALICE selected]

[SUBMIT VOTE]
```

### Ranked Choice
```
RANK ALL CANDIDATES (drag to order)

  1. 🅰️ ALICE    ↕️
  2. 🅲️ CAROL    ↕️
  3. 🅱️ BOB      ↕️

[SUBMIT RANKING]
```

### Results Comparison
```
SAME VOTES, DIFFERENT WINNERS!

Votes cast:
- 6 voters: A > B > C
- 5 voters: B > C > A
- 4 voters: C > B > A

RESULTS BY SYSTEM:
+-----------------+--------+
| System          | Winner |
+-----------------+--------+
| Plurality       | A 🏆   |
| Ranked Choice   | B 🏆   |
| Borda Count     | B 🏆   |
| Condorcet       | B 🏆   |
+-----------------+--------+

💡 The "winner" depends on the rules!
```

---

## 7. BOT VOTER PROFILES

| ID | Name | Behavior |
|----|------|----------|
| `honest` | Honest | Votes true preferences |
| `strategic` | Strategic | Votes to maximize outcome |
| `random` | Random | Random preferences |
| `polarized` | Polarized | Extreme preferences |
| `moderate` | Moderate | Centrist preferences |

---

## 8. EDUCATIONAL SCENARIOS

### 8.1 Spoiler Effect
```
Scenario: A vs B close race, C enters
Without C: A wins
With C: C takes votes from A, B wins
Demonstrates: Third-party spoiler effect
```

### 8.2 Condorcet Paradox
```
Scenario: Rock-paper-scissors preferences
1/3 prefer A>B>C
1/3 prefer B>C>A
1/3 prefer C>A>B
No Condorcet winner - cycle!
```

### 8.3 Strategic Voting
```
Scenario: Your favorite can't win
Honest: Vote for favorite anyway
Strategic: Vote for lesser evil to prevent worst
```

---

## 9. ACHIEVEMENTS

| ID | Name | Condition |
|----|------|-----------|
| `vl-first` | First Vote | Complete first voting game |
| `vl-explorer` | System Explorer | Try all voting systems |
| `vl-paradox` | Paradox Found | Witness different winners |
| `vl-condorcet` | Cycle Spotter | Find a Condorcet paradox |
| `vl-strategist` | Strategist | Win by voting strategically |

---

## 10. i18n KEYS

### English
```json
"voting-lab": {
  "name": "Voting Lab",
  "description": "Explore how different voting systems work",
  "systems": {
    "plurality": "Plurality (First Past the Post)",
    "ranked": "Ranked Choice",
    "borda": "Borda Count",
    "approval": "Approval Voting",
    "condorcet": "Condorcet Method"
  },
  "actions": {
    "vote": "Vote",
    "rank": "Rank",
    "approve": "Approve"
  },
  "insights": {
    "no_perfect": "Arrow's Theorem: No voting system is perfect!",
    "same_votes": "Same votes can produce different winners.",
    "strategic": "Sometimes voting honestly isn't optimal."
  }
}
```

---

## 11. PEDAGOGICAL VALUE

```
Voting Lab teaches:

1. NO PERFECT SYSTEM
   - Arrow's Impossibility Theorem
   - Every system has trade-offs

2. CONTEXT MATTERS
   - Same preferences, different outcomes
   - Rules shape results

3. STRATEGIC BEHAVIOR
   - Honest vs strategic voting
   - Game theory in elections

4. REAL-WORLD APPLICATION
   - Different countries use different systems
   - Debates about electoral reform
```

---

## 12. REFERENCES

- Arrow, K. - Social Choice and Individual Values
- Gibbard-Satterthwaite Theorem
- Condorcet, Marquis de - Voting paradoxes
- Borda, Jean-Charles de - Borda count
