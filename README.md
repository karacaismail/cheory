# Cheory

**Game Theory Education Platform**

Learn game theory through interactive games. No textbooks, no formulas — just play and discover.

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-web-lightgrey.svg)]()

---

## What is Cheory?

Cheory is an interactive platform that teaches game theory concepts through playable games. Instead of reading about the Prisoner's Dilemma, you *play* it. Instead of memorizing Nash Equilibrium formulas, you *discover* them.

### Target Audience

- **Students** (10+) learning strategic thinking
- **Educators** teaching economics, psychology, or philosophy
- **Curious minds** interested in decision-making and strategy

### Key Features

- 🎮 **12 Classic Games** — From Prisoner's Dilemma to Voting Systems
- 🤖 **Smart Bots** — Multiple difficulty levels and strategies
- 📚 **Interactive Tutorials** — Learn by doing
- 📊 **Progress Tracking** — Stats, achievements, and levels
- 🌐 **Bilingual** — English and Turkish support
- 📱 **Responsive** — Works on desktop, tablet, and mobile
- 🔒 **Privacy-First** — All data stored locally

---

## Games

### Free Tier

| Game | Category | Concept |
|------|----------|---------|
| Prisoner's Dilemma | Classic | Cooperation vs Defection |
| Stag Hunt | Coordination | Trust and Coordination |
| Matching Pennies | Zero-Sum | Randomization |
| Ultimatum Game | Bargaining | Fairness |
| Dictator Game | Bargaining | Altruism |
| Rock Paper Scissors | Zero-Sum | Cyclic Dominance |

### Premium Tier

| Game | Category | Concept |
|------|----------|---------|
| Iterated PD | Classic | Repeated Interactions |
| Hawk-Dove | Classic | Aggression and Conflict |
| Battle of Sexes | Coordination | Compromise |
| Public Goods | Social | Free-Riding |
| Trust Game | Social | Trust and Reciprocity |
| Voting Lab | Voting | Electoral Systems |

---

## Quick Start

### Play Online

Visit [cheory.app](https://cheory.app) to start playing immediately.

### Run Locally

```bash
# Clone the repository
git clone https://github.com/cheory/cheory.git
cd cheory

# No build step required! Just serve the files
npx serve .

# Or use Python
python -m http.server 8000

# Or use PHP
php -S localhost:8000
```

Open `http://localhost:8000` in your browser.

### Requirements

- Modern browser (Chrome, Firefox, Safari, Edge)
- JavaScript enabled
- No server or database required

---

## Architecture

Cheory is built with a **vanilla JavaScript** architecture — no frameworks, no build tools, no npm dependencies.

```
cheory/
├── index.html          # Entry point
├── core/               # Engine, store, events, i18n
├── games/              # Game modules
├── components/         # Web Components
├── assets/             # Icons, images, sounds
└── docs/               # Documentation
```

### Key Principles

1. **Zero Dependencies** — Vanilla JS, CSS, HTML only
2. **Modular Design** — Each game is a self-contained module
3. **Local-First** — All data stored in browser (localStorage/IndexedDB)
4. **Progressive Enhancement** — Works without JS for basic content
5. **Accessibility** — WCAG 2.1 AA compliance

### Tech Stack

| Layer | Technology |
|-------|------------|
| UI | Web Components (Custom Elements) |
| State | Custom Store (Redux-like pattern) |
| Storage | localStorage + IndexedDB |
| Styling | CSS Custom Properties + BEM |
| i18n | Custom lightweight solution |

---

## Documentation

### Architecture (Tier 1)

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | System overview and design |
| [FOLDER_STRUCTURE.md](docs/FOLDER_STRUCTURE.md) | Project organization |
| [CORE_ENGINE_CONTRACT.md](docs/CORE_ENGINE_CONTRACT.md) | Game engine API |
| [GAME_SPEC_TEMPLATE.md](docs/GAME_SPEC_TEMPLATE.md) | Template for game specs |
| [STYLE_GUIDE.md](docs/STYLE_GUIDE.md) | Visual design system |

### Game Specifications (Tier 2)

All 12 game specifications are in `docs/games/`:

- `01_PRISONERS_DILEMMA.md` through `12_VOTING_LAB.md`

### Technical Specs (Tier 3)

| Document | Description |
|----------|-------------|
| [DATA_SCHEMA.md](docs/DATA_SCHEMA.md) | Data models and storage |
| [UI_COMPONENTS.md](docs/UI_COMPONENTS.md) | Component library |
| [AUTH_FLOW.md](docs/AUTH_FLOW.md) | User management |
| [API_CONTRACT.md](docs/API_CONTRACT.md) | Internal APIs |
| [PLATFORM_RULES.md](docs/PLATFORM_RULES.md) | Policies and rules |

---

## Development

### Project Setup

```bash
git clone https://github.com/cheory/cheory.git
cd cheory
```

No `npm install` needed. The project runs directly in the browser.

### Code Style

- ES Modules (`import`/`export`)
- Web Components for UI
- Event-driven architecture
- Functional approach for game logic

### Adding a New Game

1. Copy `docs/GAME_SPEC_TEMPLATE.md`
2. Fill in all 16 sections
3. Create module in `games/your-game/`
4. Register in `games/index.js`
5. Add i18n keys

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

### Testing

```bash
# Run in browser console
import { runTests } from '/core/testing/index.js'
runTests()
```

---

## Roadmap

### v1.0 (Current)

- [x] 12 game modules
- [x] Bot strategies
- [x] Local storage
- [x] Achievements
- [x] English + Turkish

### v1.1 (Planned)

- [ ] More bot strategies
- [ ] Game variants
- [ ] Sound effects
- [ ] Keyboard shortcuts

### v2.0 (Future)

- [ ] User accounts
- [ ] Cloud sync
- [ ] Multiplayer
- [ ] Leaderboards
- [ ] Mobile apps

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for:

- Code style guide
- Pull request process
- Issue reporting
- Translation help

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Credits

### Game Theory References

- Axelrod, R. — *The Evolution of Cooperation*
- Camerer, C. — *Behavioral Game Theory*
- Nicky Case — *The Evolution of Trust* (inspiration)

### Team

Built with ❤️ by the Cheory team.

---

## Links

- 🌐 Website: [cheory.app](https://cheory.app)
- 📖 Documentation: [docs.cheory.app](https://docs.cheory.app)
- 🐛 Issues: [GitHub Issues](https://github.com/cheory/cheory/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/cheory/cheory/discussions)

---

*"The best way to learn game theory is to play games."*
