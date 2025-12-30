# Changelog

All notable changes to Cheory will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Planned
- Sound effects and music
- Additional bot strategies
- Game variants (different payoff matrices)
- Keyboard shortcut system
- Performance optimizations

---

## [1.0.0] - 2025-XX-XX

### 🎉 Initial Release

First public release of Cheory — the game theory education platform.

### Added

#### Core Platform
- Vanilla JavaScript architecture (zero dependencies)
- Web Components UI system
- Event-driven state management
- localStorage + IndexedDB storage
- Responsive design (mobile, tablet, desktop)
- Dark/Light/System theme support
- English and Turkish language support

#### Games (12 Total)

**Free Tier:**
- Prisoner's Dilemma — Classic cooperation game
- Stag Hunt — Coordination and trust
- Matching Pennies — Zero-sum randomization
- Ultimatum Game — Fairness in bargaining
- Dictator Game — Altruism without consequences
- Rock Paper Scissors — Cyclic dominance

**Premium Tier:**
- Iterated Prisoner's Dilemma — Repeated interactions
- Hawk-Dove — Aggression and conflict
- Battle of Sexes — Coordination with preferences
- Public Goods Game — Free-rider problem
- Trust Game — Investment and reciprocity
- Voting Lab — Electoral systems comparison

#### Bot Strategies
- Easy: Random, Always-Cooperate, Always-Defect
- Medium: Tit-for-Tat, Fair, Conditional
- Hard: Grudger, Pavlov, Pattern-Detection

#### Features
- Interactive tutorials for each game
- Achievement system (20+ achievements)
- XP and leveling system (100 levels)
- Game history and statistics
- Data export/import (JSON)
- Offline support

#### Documentation
- Complete architecture documentation
- Game specification for all 12 games
- Technical specifications
- API contracts
- Contributing guidelines

### Technical Details
- No build step required
- No npm dependencies
- Pure ES Modules
- Web Components (Custom Elements)
- CSS Custom Properties
- WCAG 2.1 AA accessibility

---

## [0.9.0] - 2025-XX-XX (Beta)

### Added
- Beta testing release
- Core game engine
- 6 initial games
- Basic UI components

### Known Issues
- Mobile layout needs refinement
- Some animations not smooth on older devices
- Turkish translations incomplete

---

## [0.5.0] - 2025-XX-XX (Alpha)

### Added
- Alpha prototype
- Prisoner's Dilemma proof of concept
- Basic bot implementation
- Initial design system

---

## Version History Summary

| Version | Date | Highlights |
|---------|------|------------|
| 1.0.0 | 2025-XX-XX | Initial public release, 12 games |
| 0.9.0 | 2025-XX-XX | Beta, 6 games |
| 0.5.0 | 2025-XX-XX | Alpha prototype |

---

## Upgrade Guide

### From 0.9.x to 1.0.0

**Breaking Changes:**
- None expected

**Data Migration:**
- Automatic migration of localStorage data
- No user action required

**New Features to Try:**
- 6 new premium games
- Achievement system
- Enhanced tutorials

---

## Deprecation Notices

*No deprecations in current version.*

---

## Security Updates

### 1.0.0
- Input sanitization for display names
- XSS prevention in user-generated content
- Secure localStorage handling

---

## Links

- [Documentation](docs/)
- [Migration Guide](docs/MIGRATION.md)
- [GitHub Releases](https://github.com/cheory/cheory/releases)

---

*For detailed commit history, see [GitHub Commits](https://github.com/cheory/cheory/commits/main).*
