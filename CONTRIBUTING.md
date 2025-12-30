# Contributing to Cheory

Thank you for your interest in contributing to Cheory! This document provides guidelines and instructions for contributing.

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Setup](#development-setup)
4. [Project Structure](#project-structure)
5. [Coding Standards](#coding-standards)
6. [Making Changes](#making-changes)
7. [Adding a New Game](#adding-a-new-game)
8. [Translations](#translations)
9. [Pull Request Process](#pull-request-process)
10. [Issue Guidelines](#issue-guidelines)

---

## Code of Conduct

### Our Standards

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment or discrimination
- Trolling or insulting comments
- Public or private harassment
- Publishing others' private information

---

## Getting Started

### Prerequisites

- Modern web browser (Chrome, Firefox, Safari, Edge)
- Text editor (VS Code recommended)
- Git
- Basic knowledge of HTML, CSS, JavaScript

### No Build Tools Required

Cheory uses **zero dependencies**. No npm, no webpack, no build step.

```bash
# Clone and run immediately
git clone https://github.com/cheory/cheory.git
cd cheory
npx serve .
```

---

## Development Setup

### 1. Fork and Clone

```bash
# Fork the repo on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/cheory.git
cd cheory
git remote add upstream https://github.com/cheory/cheory.git
```

### 2. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

### 3. Start Local Server

```bash
# Option 1: Node.js
npx serve .

# Option 2: Python
python -m http.server 8000

# Option 3: PHP
php -S localhost:8000
```

### 4. Open in Browser

Navigate to `http://localhost:8000`

### Recommended VS Code Extensions

```json
{
  "recommendations": [
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "ritwickdey.liveserver",
    "bradlc.vscode-tailwindcss"
  ]
}
```

---

## Project Structure

```
cheory/
├── index.html              # Entry point
├── core/                   # Core modules
│   ├── engine/             # Game engine
│   ├── store/              # State management
│   ├── events/             # Event bus
│   ├── i18n/               # Internationalization
│   ├── storage/            # Data persistence
│   └── components/         # Base component class
├── games/                  # Game modules
│   ├── prisoners-dilemma/
│   │   ├── index.js        # Module entry
│   │   ├── engine.js       # Game logic
│   │   ├── view.js         # UI components
│   │   ├── strategies.js   # Bot AI
│   │   └── tutorial.js     # Tutorial config
│   └── [other-games]/
├── components/             # Shared UI components
├── assets/                 # Static assets
│   ├── icons/
│   ├── images/
│   └── sounds/
├── styles/                 # Global styles
│   ├── tokens.css          # Design tokens
│   ├── reset.css           # CSS reset
│   └── global.css          # Global styles
├── locales/                # Translation files
│   ├── en.json
│   └── tr.json
└── docs/                   # Documentation
```

---

## Coding Standards

### JavaScript

```javascript
// ✅ DO: Use ES Modules
import { something } from './module.js'
export function myFunction() {}

// ✅ DO: Use const/let, never var
const immutable = 'value'
let mutable = 'value'

// ✅ DO: Use arrow functions for callbacks
array.map(item => item.value)

// ✅ DO: Use template literals
const message = `Hello, ${name}!`

// ✅ DO: Use destructuring
const { id, name } = player

// ❌ DON'T: Use var
var oldStyle = 'bad'

// ❌ DON'T: Use == (use === instead)
if (value == null) // bad
if (value === null) // good

// ❌ DON'T: Use external dependencies
import React from 'react' // NO!
```

### File Naming

```
✅ kebab-case for files: game-board.js
✅ PascalCase for classes: GameBoard
✅ camelCase for functions: calculateScore
✅ SCREAMING_CASE for constants: MAX_PLAYERS
```

### CSS

```css
/* ✅ DO: Use CSS Custom Properties */
.button {
  background: var(--brand-primary);
  padding: var(--space-4);
}

/* ✅ DO: Use BEM naming */
.game-board {}
.game-board__header {}
.game-board--highlighted {}

/* ❌ DON'T: Use IDs for styling */
#myButton {} /* bad */

/* ❌ DON'T: Use !important */
.button { color: red !important; } /* bad */
```

### Web Components

```javascript
// ✅ DO: Extend BaseComponent
class MyComponent extends BaseComponent {
  static get observedAttributes() {
    return ['value']
  }
  
  render() {
    this.injectStyles(`/* CSS here */`)
    this.shadowRoot.innerHTML += `<!-- HTML here -->`
  }
}

// ✅ DO: Use 'ch-' prefix
customElements.define('ch-my-component', MyComponent)
```

### Comments

```javascript
// ✅ DO: Explain WHY, not WHAT
// Calculate payoff using standard PD matrix
// because custom matrices aren't supported yet
const payoff = standardMatrix[p1][p2]

// ❌ DON'T: State the obvious
// Set x to 5
const x = 5
```

---

## Making Changes

### 1. Keep Changes Focused

- One feature/fix per PR
- Small, reviewable changes
- Clear commit messages

### 2. Commit Message Format

```
type(scope): description

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting (no code change)
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance

**Examples:**
```
feat(games): add Iterated PD game module
fix(engine): resolve score calculation bug
docs(readme): update installation instructions
style(components): format button styles
```

### 3. Test Your Changes

```javascript
// Run in browser console
import { runTests } from '/core/testing/index.js'
runTests()

// Or test specific module
import { testEngine } from '/games/prisoners-dilemma/tests.js'
testEngine()
```

---

## Adding a New Game

### Step 1: Create Specification

Copy `docs/GAME_SPEC_TEMPLATE.md` and fill all 16 sections:

1. Overview
2. Game Rules
3. Actions
4. Payoff/Scoring
5. State Structure
6. UI Design
7. Bot Strategies
8. Tutorial
9. Edge Cases
10. Test Scenarios
11. Metrics
12. Achievements
13. i18n Keys
14. Dependencies
15. Notes
16. Document History

### Step 2: Create Module

```
games/
└── your-game/
    ├── index.js        # Module exports
    ├── engine.js       # Game logic
    ├── view.js         # Components
    ├── strategies.js   # Bot AI
    ├── tutorial.js     # Tutorial config
    └── tests.js        # Unit tests
```

### Step 3: Implement Engine

```javascript
// games/your-game/engine.js

export const engine = {
  createInitialState(options) {
    return {
      // Initial game state
    }
  },
  
  validateAction(state, action) {
    // Return { valid: boolean, error?: string }
  },
  
  executeAction(state, action) {
    // Return new state
  },
  
  isGameOver(state) {
    // Return boolean
  },
  
  getWinner(state) {
    // Return playerId or null
  }
}
```

### Step 4: Register Module

```javascript
// games/index.js
export { yourGame } from './your-game/index.js'
```

### Step 5: Add Translations

```json
// locales/en.json
{
  "your-game": {
    "name": "Your Game",
    "description": "Game description"
  }
}
```

---

## Translations

### Adding a New Language

1. Copy `locales/en.json` to `locales/XX.json`
2. Translate all strings
3. Register in `core/i18n/index.js`

### Translation Guidelines

```json
{
  "key": "Translation",
  
  // ✅ DO: Keep placeholders
  "welcome": "Welcome, {{name}}!",
  
  // ✅ DO: Match tone (friendly, educational)
  "try_again": "Let's try again!",
  
  // ❌ DON'T: Translate technical terms inconsistently
  // Pick one term and stick with it
}
```

### Current Languages

| Code | Language | Status |
|------|----------|--------|
| en | English | ✅ Complete |
| tr | Turkish | ✅ Complete |
| es | Spanish | 🔜 Planned |
| de | German | 🔜 Planned |

---

## Pull Request Process

### 1. Before Submitting

- [ ] Code follows style guidelines
- [ ] Self-reviewed the changes
- [ ] Added/updated documentation
- [ ] Added/updated translations
- [ ] Tested in multiple browsers
- [ ] No console errors

### 2. PR Title Format

```
type(scope): brief description
```

### 3. PR Description Template

```markdown
## Description
Brief description of changes.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation
- [ ] Refactoring

## Testing
How was this tested?

## Screenshots
If applicable.

## Checklist
- [ ] Code follows style guide
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Tests added/updated
```

### 4. Review Process

1. Automated checks run
2. Maintainer reviews
3. Feedback addressed
4. Approval and merge

---

## Issue Guidelines

### Bug Reports

```markdown
**Description**
Clear description of the bug.

**Steps to Reproduce**
1. Go to...
2. Click on...
3. See error

**Expected Behavior**
What should happen.

**Actual Behavior**
What actually happens.

**Environment**
- Browser: Chrome 120
- OS: Windows 11
- Screen size: 1920x1080

**Screenshots**
If applicable.
```

### Feature Requests

```markdown
**Problem**
What problem does this solve?

**Proposed Solution**
How should it work?

**Alternatives Considered**
Other approaches you've thought about.

**Additional Context**
Any other information.
```

---

## Recognition

Contributors are recognized in:

- README.md Contributors section
- Release notes
- Special achievements in-app (future)

---

## Questions?

- 💬 [GitHub Discussions](https://github.com/cheory/cheory/discussions)
- 📧 Email: contribute@cheory.app

---

Thank you for contributing to Cheory! 🎮
