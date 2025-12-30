# UI COMPONENTS
## Technical Specification Document

| Field | Value |
|-------|-------|
| Document ID | `UI_COMPONENTS` |
| Version | 1.0.0 |
| Tier | 3 - Technical |
| Status | approved |

---

## 1. OVERVIEW

### 1.1 Purpose

This document defines all reusable UI components for the Cheory platform, built as native Web Components (Custom Elements) with no framework dependencies.

### 1.2 Component Architecture

```
BaseComponent (abstract)
├── Core Components (buttons, inputs, cards)
├── Layout Components (shell, grid, panels)
├── Game Components (board, matrix, choices)
└── Feedback Components (toast, modal, loader)
```

### 1.3 Naming Convention

```
Prefix: ch-
Format: ch-[category]-[name]

Examples:
- ch-button
- ch-game-board
- ch-modal-dialog
```

---

## 2. BASE COMPONENT

### 2.1 Abstract Base Class

```javascript
// /core/components/base-component.js

export class BaseComponent extends HTMLElement {
  constructor() {
    super()
    this.attachShadow({ mode: 'open' })
  }
  
  // Lifecycle
  connectedCallback() {
    this.render()
    this.setupEventListeners()
  }
  
  disconnectedCallback() {
    this.cleanup()
  }
  
  // To be overridden
  render() {}
  setupEventListeners() {}
  cleanup() {}
  
  // Utilities
  $(selector) {
    return this.shadowRoot.querySelector(selector)
  }
  
  $$(selector) {
    return this.shadowRoot.querySelectorAll(selector)
  }
  
  emit(eventName, detail = {}) {
    this.dispatchEvent(new CustomEvent(eventName, {
      bubbles: true,
      composed: true,
      detail
    }))
  }
  
  // Template helper
  html(strings, ...values) {
    return strings.reduce((acc, str, i) => 
      acc + str + (values[i] ?? ''), '')
  }
  
  // Style injection
  injectStyles(css) {
    const style = document.createElement('style')
    style.textContent = css
    this.shadowRoot.appendChild(style)
  }
}
```

---

## 3. CORE COMPONENTS

### 3.1 Button `<ch-button>`

```javascript
// Attributes: variant, size, disabled, loading, icon
// Events: click

class ChButton extends BaseComponent {
  static get observedAttributes() {
    return ['variant', 'size', 'disabled', 'loading', 'icon']
  }
  
  get variant() { 
    return this.getAttribute('variant') || 'primary' 
  }
  
  get size() { 
    return this.getAttribute('size') || 'medium' 
  }
  
  render() {
    this.injectStyles(`
      :host {
        display: inline-block;
      }
      
      button {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        gap: var(--space-2);
        padding: var(--space-2) var(--space-4);
        border: none;
        border-radius: var(--radius-md);
        font-family: inherit;
        font-weight: 500;
        cursor: pointer;
        transition: var(--transition-fast);
      }
      
      /* Variants */
      .primary {
        background: var(--brand-gradient);
        color: white;
      }
      
      .primary:hover {
        box-shadow: var(--glow-brand);
      }
      
      .secondary {
        background: var(--glass-bg);
        color: var(--text-primary);
        border: 1px solid var(--glass-border);
      }
      
      .ghost {
        background: transparent;
        color: var(--text-secondary);
      }
      
      .danger {
        background: var(--color-error);
        color: white;
      }
      
      /* Sizes */
      .small { 
        padding: var(--space-1) var(--space-2);
        font-size: var(--text-sm);
      }
      
      .large { 
        padding: var(--space-3) var(--space-6);
        font-size: var(--text-lg);
      }
      
      /* States */
      :host([disabled]) button {
        opacity: 0.5;
        cursor: not-allowed;
      }
      
      :host([loading]) button {
        pointer-events: none;
      }
      
      .spinner {
        width: 1em;
        height: 1em;
        border: 2px solid currentColor;
        border-top-color: transparent;
        border-radius: 50%;
        animation: spin 0.6s linear infinite;
      }
      
      @keyframes spin {
        to { transform: rotate(360deg); }
      }
    `)
    
    const loading = this.hasAttribute('loading')
    
    this.shadowRoot.innerHTML += `
      <button class="${this.variant} ${this.size}">
        ${loading ? '<span class="spinner"></span>' : ''}
        <slot></slot>
      </button>
    `
  }
}

customElements.define('ch-button', ChButton)
```

**Usage:**
```html
<ch-button variant="primary">Click Me</ch-button>
<ch-button variant="secondary" size="small">Cancel</ch-button>
<ch-button loading>Saving...</ch-button>
```

### 3.2 Card `<ch-card>`

```javascript
// Attributes: variant, clickable, selected
// Slots: default, header, footer

class ChCard extends BaseComponent {
  static get observedAttributes() {
    return ['variant', 'clickable', 'selected']
  }
  
  render() {
    this.injectStyles(`
      :host {
        display: block;
      }
      
      .card {
        background: var(--glass-bg);
        border: 1px solid var(--glass-border);
        border-radius: var(--radius-lg);
        backdrop-filter: blur(var(--glass-blur));
        overflow: hidden;
      }
      
      :host([clickable]) .card {
        cursor: pointer;
        transition: var(--transition-normal);
      }
      
      :host([clickable]) .card:hover {
        transform: translateY(-2px);
        box-shadow: var(--shadow-lg);
      }
      
      :host([selected]) .card {
        border-color: var(--brand-primary);
        box-shadow: var(--glow-brand);
      }
      
      .header {
        padding: var(--space-4);
        border-bottom: 1px solid var(--glass-border);
      }
      
      .content {
        padding: var(--space-4);
      }
      
      .footer {
        padding: var(--space-4);
        border-top: 1px solid var(--glass-border);
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="card">
        <div class="header"><slot name="header"></slot></div>
        <div class="content"><slot></slot></div>
        <div class="footer"><slot name="footer"></slot></div>
      </div>
    `
  }
}

customElements.define('ch-card', ChCard)
```

### 3.3 Input `<ch-input>`

```javascript
// Attributes: type, placeholder, value, disabled, error
// Events: input, change

class ChInput extends BaseComponent {
  static get observedAttributes() {
    return ['type', 'placeholder', 'value', 'disabled', 'error']
  }
  
  render() {
    this.injectStyles(`
      :host {
        display: block;
      }
      
      .wrapper {
        position: relative;
      }
      
      input {
        width: 100%;
        padding: var(--space-3);
        background: var(--bg-secondary);
        border: 1px solid var(--glass-border);
        border-radius: var(--radius-md);
        color: var(--text-primary);
        font-size: var(--text-base);
        transition: var(--transition-fast);
      }
      
      input:focus {
        outline: none;
        border-color: var(--brand-primary);
        box-shadow: 0 0 0 3px var(--brand-primary-alpha);
      }
      
      :host([error]) input {
        border-color: var(--color-error);
      }
      
      .error-text {
        color: var(--color-error);
        font-size: var(--text-sm);
        margin-top: var(--space-1);
      }
    `)
    
    const type = this.getAttribute('type') || 'text'
    const placeholder = this.getAttribute('placeholder') || ''
    const value = this.getAttribute('value') || ''
    const error = this.getAttribute('error')
    
    this.shadowRoot.innerHTML += `
      <div class="wrapper">
        <input 
          type="${type}" 
          placeholder="${placeholder}"
          value="${value}"
        />
        ${error ? `<div class="error-text">${error}</div>` : ''}
      </div>
    `
  }
  
  setupEventListeners() {
    const input = this.$('input')
    input?.addEventListener('input', (e) => {
      this.emit('input', { value: e.target.value })
    })
  }
}

customElements.define('ch-input', ChInput)
```

### 3.4 Slider `<ch-slider>`

```javascript
// Attributes: min, max, value, step, show-value
// Events: change

class ChSlider extends BaseComponent {
  static get observedAttributes() {
    return ['min', 'max', 'value', 'step', 'show-value']
  }
  
  render() {
    const min = this.getAttribute('min') || 0
    const max = this.getAttribute('max') || 100
    const value = this.getAttribute('value') || 50
    const step = this.getAttribute('step') || 1
    
    this.injectStyles(`
      :host {
        display: block;
      }
      
      .slider-wrapper {
        display: flex;
        align-items: center;
        gap: var(--space-3);
      }
      
      input[type="range"] {
        flex: 1;
        height: 6px;
        border-radius: var(--radius-full);
        background: var(--bg-tertiary);
        appearance: none;
      }
      
      input[type="range"]::-webkit-slider-thumb {
        appearance: none;
        width: 20px;
        height: 20px;
        border-radius: 50%;
        background: var(--brand-primary);
        cursor: pointer;
        transition: var(--transition-fast);
      }
      
      input[type="range"]::-webkit-slider-thumb:hover {
        transform: scale(1.1);
        box-shadow: var(--glow-brand);
      }
      
      .value-display {
        min-width: 3ch;
        text-align: center;
        font-weight: 600;
        color: var(--text-primary);
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="slider-wrapper">
        <span class="min-label">${min}</span>
        <input type="range" 
          min="${min}" 
          max="${max}" 
          value="${value}"
          step="${step}"
        />
        <span class="max-label">${max}</span>
        ${this.hasAttribute('show-value') ? 
          `<span class="value-display">${value}</span>` : ''}
      </div>
    `
  }
  
  setupEventListeners() {
    const input = this.$('input')
    const display = this.$('.value-display')
    
    input?.addEventListener('input', (e) => {
      if (display) display.textContent = e.target.value
      this.emit('change', { value: Number(e.target.value) })
    })
  }
}

customElements.define('ch-slider', ChSlider)
```

---

## 4. LAYOUT COMPONENTS

### 4.1 App Shell `<ch-shell>`

```javascript
// Slots: sidebar, main

class ChShell extends BaseComponent {
  render() {
    this.injectStyles(`
      :host {
        display: block;
        height: 100vh;
        background: var(--bg-primary);
      }
      
      .shell {
        display: grid;
        grid-template-columns: var(--sidebar-width) 1fr;
        height: 100%;
      }
      
      .sidebar {
        background: var(--bg-secondary);
        border-right: 1px solid var(--glass-border);
        overflow-y: auto;
      }
      
      .main {
        overflow-y: auto;
        padding: var(--space-6);
      }
      
      @media (max-width: 768px) {
        .shell {
          grid-template-columns: 1fr;
        }
        
        .sidebar {
          display: none;
        }
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="shell">
        <aside class="sidebar">
          <slot name="sidebar"></slot>
        </aside>
        <main class="main">
          <slot name="main"></slot>
        </main>
      </div>
    `
  }
}

customElements.define('ch-shell', ChShell)
```

### 4.2 Bento Grid `<ch-bento-grid>`

```javascript
// Attributes: columns, gap

class ChBentoGrid extends BaseComponent {
  static get observedAttributes() {
    return ['columns', 'gap']
  }
  
  render() {
    const columns = this.getAttribute('columns') || '3'
    const gap = this.getAttribute('gap') || '4'
    
    this.injectStyles(`
      :host {
        display: block;
      }
      
      .grid {
        display: grid;
        grid-template-columns: repeat(${columns}, 1fr);
        gap: var(--space-${gap});
      }
      
      @media (max-width: 768px) {
        .grid {
          grid-template-columns: 1fr;
        }
      }
      
      ::slotted([span="2"]) {
        grid-column: span 2;
      }
      
      ::slotted([span="3"]) {
        grid-column: span 3;
      }
      
      ::slotted([span-row="2"]) {
        grid-row: span 2;
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="grid">
        <slot></slot>
      </div>
    `
  }
}

customElements.define('ch-bento-grid', ChBentoGrid)
```

---

## 5. GAME COMPONENTS

### 5.1 Game Board `<ch-game-board>`

```javascript
// Attributes: game-id
// Slots: header, content, actions

class ChGameBoard extends BaseComponent {
  render() {
    this.injectStyles(`
      :host {
        display: block;
      }
      
      .board {
        background: var(--glass-bg);
        border: 1px solid var(--glass-border);
        border-radius: var(--radius-xl);
        backdrop-filter: blur(var(--glass-blur));
        overflow: hidden;
      }
      
      .header {
        padding: var(--space-4);
        border-bottom: 1px solid var(--glass-border);
        display: flex;
        justify-content: space-between;
        align-items: center;
      }
      
      .content {
        padding: var(--space-6);
        min-height: 300px;
      }
      
      .actions {
        padding: var(--space-4);
        border-top: 1px solid var(--glass-border);
        display: flex;
        justify-content: center;
        gap: var(--space-4);
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="board">
        <div class="header">
          <slot name="header"></slot>
        </div>
        <div class="content">
          <slot></slot>
        </div>
        <div class="actions">
          <slot name="actions"></slot>
        </div>
      </div>
    `
  }
}

customElements.define('ch-game-board', ChGameBoard)
```

### 5.2 Payoff Matrix `<ch-payoff-matrix>`

```javascript
// Attributes: player1-label, player2-label
// Properties: matrix (2D array)

class ChPayoffMatrix extends BaseComponent {
  static get observedAttributes() {
    return ['player1-label', 'player2-label']
  }
  
  set matrix(value) {
    this._matrix = value
    this.render()
  }
  
  get matrix() {
    return this._matrix || [
      [{ p1: 3, p2: 3 }, { p1: 0, p2: 5 }],
      [{ p1: 5, p2: 0 }, { p1: 1, p2: 1 }]
    ]
  }
  
  render() {
    const p1Label = this.getAttribute('player1-label') || 'You'
    const p2Label = this.getAttribute('player2-label') || 'Opponent'
    const actions = ['Cooperate', 'Defect']
    
    this.injectStyles(`
      :host {
        display: block;
      }
      
      table {
        width: 100%;
        border-collapse: collapse;
        text-align: center;
      }
      
      th, td {
        padding: var(--space-3);
        border: 1px solid var(--glass-border);
      }
      
      th {
        background: var(--bg-tertiary);
        font-weight: 600;
      }
      
      .payoff {
        font-weight: 500;
      }
      
      .p1 { color: var(--color-cooperate); }
      .p2 { color: var(--color-defect); }
    `)
    
    const m = this.matrix
    
    this.shadowRoot.innerHTML += `
      <table>
        <thead>
          <tr>
            <th></th>
            <th colspan="2">${p2Label}</th>
          </tr>
          <tr>
            <th>${p1Label}</th>
            <th>${actions[0]}</th>
            <th>${actions[1]}</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <th>${actions[0]}</th>
            <td class="payoff">
              <span class="p1">${m[0][0].p1}</span>, 
              <span class="p2">${m[0][0].p2}</span>
            </td>
            <td class="payoff">
              <span class="p1">${m[0][1].p1}</span>, 
              <span class="p2">${m[0][1].p2}</span>
            </td>
          </tr>
          <tr>
            <th>${actions[1]}</th>
            <td class="payoff">
              <span class="p1">${m[1][0].p1}</span>, 
              <span class="p2">${m[1][0].p2}</span>
            </td>
            <td class="payoff">
              <span class="p1">${m[1][1].p1}</span>, 
              <span class="p2">${m[1][1].p2}</span>
            </td>
          </tr>
        </tbody>
      </table>
    `
  }
}

customElements.define('ch-payoff-matrix', ChPayoffMatrix)
```

### 5.3 Choice Button `<ch-choice-button>`

```javascript
// Attributes: choice, selected, disabled
// Events: select

class ChChoiceButton extends BaseComponent {
  static get observedAttributes() {
    return ['choice', 'selected', 'disabled']
  }
  
  render() {
    const choice = this.getAttribute('choice') || 'cooperate'
    const isCooperate = choice === 'cooperate'
    
    this.injectStyles(`
      :host {
        display: block;
      }
      
      .choice-btn {
        width: 100%;
        padding: var(--space-6);
        border: 2px solid var(--glass-border);
        border-radius: var(--radius-lg);
        background: var(--glass-bg);
        cursor: pointer;
        transition: var(--transition-normal);
        text-align: center;
      }
      
      .choice-btn:hover {
        transform: translateY(-4px);
      }
      
      .choice-btn.cooperate:hover {
        border-color: var(--color-cooperate);
        box-shadow: 0 0 20px var(--color-cooperate-alpha);
      }
      
      .choice-btn.defect:hover {
        border-color: var(--color-defect);
        box-shadow: 0 0 20px var(--color-defect-alpha);
      }
      
      :host([selected]) .choice-btn.cooperate {
        background: var(--color-cooperate);
        border-color: var(--color-cooperate);
        color: white;
      }
      
      :host([selected]) .choice-btn.defect {
        background: var(--color-defect);
        border-color: var(--color-defect);
        color: white;
      }
      
      .icon {
        font-size: 2rem;
        margin-bottom: var(--space-2);
      }
      
      .label {
        font-size: var(--text-lg);
        font-weight: 600;
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <button class="choice-btn ${choice}">
        <div class="icon">${isCooperate ? '🤝' : '🗡️'}</div>
        <div class="label">${isCooperate ? 'Cooperate' : 'Defect'}</div>
      </button>
    `
  }
  
  setupEventListeners() {
    this.$('.choice-btn')?.addEventListener('click', () => {
      if (!this.hasAttribute('disabled')) {
        this.emit('select', { choice: this.getAttribute('choice') })
      }
    })
  }
}

customElements.define('ch-choice-button', ChChoiceButton)
```

### 5.4 Score Display `<ch-score>`

```javascript
// Attributes: value, label, animate

class ChScore extends BaseComponent {
  static get observedAttributes() {
    return ['value', 'label', 'animate']
  }
  
  render() {
    const value = this.getAttribute('value') || '0'
    const label = this.getAttribute('label') || 'Score'
    
    this.injectStyles(`
      :host {
        display: inline-block;
      }
      
      .score {
        text-align: center;
        padding: var(--space-4);
        background: var(--glass-bg);
        border-radius: var(--radius-lg);
        border: 1px solid var(--glass-border);
      }
      
      .value {
        font-size: var(--text-3xl);
        font-weight: 700;
        color: var(--brand-primary);
      }
      
      .label {
        font-size: var(--text-sm);
        color: var(--text-secondary);
        margin-top: var(--space-1);
      }
      
      :host([animate]) .value {
        animation: scoreUp 0.5s ease-out;
      }
      
      @keyframes scoreUp {
        from { transform: scale(1.5); opacity: 0; }
        to { transform: scale(1); opacity: 1; }
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="score">
        <div class="value">${value}</div>
        <div class="label">${label}</div>
      </div>
    `
  }
}

customElements.define('ch-score', ChScore)
```

---

## 6. FEEDBACK COMPONENTS

### 6.1 Modal `<ch-modal>`

```javascript
// Attributes: open, title
// Events: close
// Slots: default, footer

class ChModal extends BaseComponent {
  static get observedAttributes() {
    return ['open', 'title']
  }
  
  render() {
    const title = this.getAttribute('title') || ''
    
    this.injectStyles(`
      :host {
        display: none;
      }
      
      :host([open]) {
        display: block;
      }
      
      .overlay {
        position: fixed;
        inset: 0;
        background: rgba(0, 0, 0, 0.7);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: var(--z-modal);
        animation: fadeIn 0.2s ease-out;
      }
      
      .modal {
        background: var(--bg-secondary);
        border-radius: var(--radius-xl);
        max-width: 500px;
        width: 90%;
        max-height: 80vh;
        overflow: hidden;
        animation: slideUp 0.3s ease-out;
      }
      
      .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: var(--space-4);
        border-bottom: 1px solid var(--glass-border);
      }
      
      .title {
        font-size: var(--text-xl);
        font-weight: 600;
      }
      
      .close-btn {
        background: none;
        border: none;
        font-size: 1.5rem;
        cursor: pointer;
        color: var(--text-secondary);
      }
      
      .content {
        padding: var(--space-6);
        overflow-y: auto;
      }
      
      .footer {
        padding: var(--space-4);
        border-top: 1px solid var(--glass-border);
        display: flex;
        justify-content: flex-end;
        gap: var(--space-3);
      }
      
      @keyframes fadeIn {
        from { opacity: 0; }
        to { opacity: 1; }
      }
      
      @keyframes slideUp {
        from { transform: translateY(20px); opacity: 0; }
        to { transform: translateY(0); opacity: 1; }
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="overlay">
        <div class="modal">
          <div class="header">
            <span class="title">${title}</span>
            <button class="close-btn">&times;</button>
          </div>
          <div class="content">
            <slot></slot>
          </div>
          <div class="footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    `
  }
  
  setupEventListeners() {
    this.$('.close-btn')?.addEventListener('click', () => {
      this.removeAttribute('open')
      this.emit('close')
    })
    
    this.$('.overlay')?.addEventListener('click', (e) => {
      if (e.target.classList.contains('overlay')) {
        this.removeAttribute('open')
        this.emit('close')
      }
    })
  }
}

customElements.define('ch-modal', ChModal)
```

### 6.2 Toast `<ch-toast>`

```javascript
// Attributes: type, duration, message
// Types: success, error, warning, info

class ChToast extends BaseComponent {
  static get observedAttributes() {
    return ['type', 'duration', 'message']
  }
  
  connectedCallback() {
    super.connectedCallback()
    const duration = parseInt(this.getAttribute('duration') || '3000')
    setTimeout(() => this.remove(), duration)
  }
  
  render() {
    const type = this.getAttribute('type') || 'info'
    const message = this.getAttribute('message') || ''
    
    const icons = {
      success: '✓',
      error: '✕',
      warning: '⚠',
      info: 'ℹ'
    }
    
    this.injectStyles(`
      :host {
        display: block;
        animation: slideIn 0.3s ease-out;
      }
      
      .toast {
        display: flex;
        align-items: center;
        gap: var(--space-3);
        padding: var(--space-3) var(--space-4);
        border-radius: var(--radius-md);
        background: var(--bg-secondary);
        border-left: 4px solid;
        box-shadow: var(--shadow-lg);
      }
      
      .toast.success { border-color: var(--color-success); }
      .toast.error { border-color: var(--color-error); }
      .toast.warning { border-color: var(--color-warning); }
      .toast.info { border-color: var(--color-info); }
      
      .icon {
        font-size: 1.25rem;
      }
      
      @keyframes slideIn {
        from { transform: translateX(100%); opacity: 0; }
        to { transform: translateX(0); opacity: 1; }
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="toast ${type}">
        <span class="icon">${icons[type]}</span>
        <span class="message">${message}</span>
      </div>
    `
  }
}

customElements.define('ch-toast', ChToast)

// Toast container & helper
class ChToastContainer extends BaseComponent {
  render() {
    this.injectStyles(`
      :host {
        position: fixed;
        top: var(--space-4);
        right: var(--space-4);
        display: flex;
        flex-direction: column;
        gap: var(--space-2);
        z-index: var(--z-toast);
      }
    `)
    this.shadowRoot.innerHTML += `<slot></slot>`
  }
}

customElements.define('ch-toast-container', ChToastContainer)

// Global toast function
window.showToast = (message, type = 'info', duration = 3000) => {
  let container = document.querySelector('ch-toast-container')
  if (!container) {
    container = document.createElement('ch-toast-container')
    document.body.appendChild(container)
  }
  
  const toast = document.createElement('ch-toast')
  toast.setAttribute('message', message)
  toast.setAttribute('type', type)
  toast.setAttribute('duration', duration)
  container.appendChild(toast)
}
```

### 6.3 Loader `<ch-loader>`

```javascript
// Attributes: size, label

class ChLoader extends BaseComponent {
  static get observedAttributes() {
    return ['size', 'label']
  }
  
  render() {
    const size = this.getAttribute('size') || 'medium'
    const label = this.getAttribute('label') || ''
    
    const sizes = { small: '24px', medium: '40px', large: '60px' }
    
    this.injectStyles(`
      :host {
        display: inline-flex;
        flex-direction: column;
        align-items: center;
        gap: var(--space-2);
      }
      
      .spinner {
        width: ${sizes[size]};
        height: ${sizes[size]};
        border: 3px solid var(--bg-tertiary);
        border-top-color: var(--brand-primary);
        border-radius: 50%;
        animation: spin 0.8s linear infinite;
      }
      
      .label {
        color: var(--text-secondary);
        font-size: var(--text-sm);
      }
      
      @keyframes spin {
        to { transform: rotate(360deg); }
      }
    `)
    
    this.shadowRoot.innerHTML += `
      <div class="spinner"></div>
      ${label ? `<span class="label">${label}</span>` : ''}
    `
  }
}

customElements.define('ch-loader', ChLoader)
```

---

## 7. COMPONENT REGISTRY

```javascript
// /core/components/index.js

// Core
import './ch-button.js'
import './ch-card.js'
import './ch-input.js'
import './ch-slider.js'

// Layout
import './ch-shell.js'
import './ch-bento-grid.js'

// Game
import './ch-game-board.js'
import './ch-payoff-matrix.js'
import './ch-choice-button.js'
import './ch-score.js'

// Feedback
import './ch-modal.js'
import './ch-toast.js'
import './ch-loader.js'

console.log('Cheory components registered')
```

---

## 8. USAGE EXAMPLES

```html
<!-- Game setup -->
<ch-shell>
  <nav slot="sidebar">
    <ch-button variant="ghost">Home</ch-button>
    <ch-button variant="ghost">Games</ch-button>
  </nav>
  
  <section slot="main">
    <ch-game-board>
      <h2 slot="header">Prisoner's Dilemma</h2>
      
      <ch-payoff-matrix></ch-payoff-matrix>
      
      <div class="choices">
        <ch-choice-button choice="cooperate"></ch-choice-button>
        <ch-choice-button choice="defect"></ch-choice-button>
      </div>
      
      <div slot="actions">
        <ch-button variant="secondary">Quit</ch-button>
      </div>
    </ch-game-board>
  </section>
</ch-shell>

<!-- Modal -->
<ch-modal open title="Game Over">
  <p>You scored 15 points!</p>
  <div slot="footer">
    <ch-button variant="secondary">Close</ch-button>
    <ch-button>Play Again</ch-button>
  </div>
</ch-modal>
```

---

## 9. REFERENCES

- Web Components: MDN Web Docs
- Custom Elements: W3C Specification
- Shadow DOM: Encapsulation patterns
