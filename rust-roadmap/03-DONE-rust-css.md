# CSS & Styling (Rustacean / Systems-Oriented)

## Purpose
This document defines how AI agents must design, write, and maintain CSS for this project.

This is a **Rust-first, systems-minded web application**.
Styling must reflect:
- **Determinism**
- **Clarity**
- **Maintainability**
- **Terminal / Developer Culture**

This is **NOT** a marketing website.
This is **NOT** a JS-heavy frontend.

---

## Core Principles

### 1. Rust Toolchain First
- CSS must be compiled and bundled via **Trunk**.
- Use **Lightning CSS** (default in Trunk).
- **No** runtime CSS generation.
- **No** CDN-based CSS frameworks.

> **Rule:** If it cannot be built deterministically, it does not belong here.

### 2. No Tailwind CDN (Ever)
**Forbidden:**
- `<script src="https://cdn.tailwindcss.com">`
- Runtime utility generation.
- Opaque class magic.

**Allowed:**
- Vanilla CSS.
- Light utility classes written manually.
- Explicit class names with intent.

### 3. CSS Is a First-Class Asset
CSS should be treated like Rust code:
- Readable
- Modular
- Versioned
- Diff-friendly

**Avoid:**
- Inline styles.
- Massive single-file CSS.
- Component styles embedded in Rust unless truly dynamic.

---

## File Structure

The project uses a modular CSS architecture. Each file must have a **clear responsibility**.

```text
styles/
├── base.css        # Reset, variables, typography
├── theme.css       # Colors, spacing, shadows
├── terminal.css    # Terminal window, prompt, panels
├── roadmap.css     # Roadmap diagram & layout
└── components/
    ├── buttons.css
    ├── cards.css
    └── overlays.css
```

---

## Import Strategy (index.html)

Use Trunk-managed CSS only. Define these in the `index.html` head:

```html
<link data-trunk rel="css" href="styles/base.css" />
<link data-trunk rel="css" href="styles/theme.css" />
<link data-trunk rel="css" href="styles/terminal.css" />
<link data-trunk rel="css" href="styles/roadmap.css" />
```

**Do NOT:**
- Import CSS dynamically.
- Conditionally load CSS via JS.

---

## Naming Conventions

### Class Naming
Use semantic, intention-revealing names.

| Context | Preferred (Semantic) | Avoid (Functional/Generic) |
| :--- | :--- | :--- |
| **Panels** | `.terminal-panel` | `.box1` |
| **Headers** | `.terminal-header` | `.flex-center-2` |
| **Input** | `.cmd-prompt` | `.mt-4-xl` |
| **Cards** | `.resource-card` | `.shadow-lg` |

**Note:** This is not Tailwind. This is not Bootstrap.

### Utility Classes (Allowed, Minimal)
Allowed only when behavior is universal and meaning is obvious.

- `.mono-font`
- `.text-dim`
- `.hidden`
- `.flex-grow`

**Do NOT recreate Tailwind.**

---

## Typography Rules

### Fonts
1.  **JetBrains Mono** → Code, headers, terminal UI.
2.  **Inter** → Long-form readable text.

### Rules
- Do not mix fonts arbitrarily.
- Code-like UI must feel code-like.

---

## Color Philosophy

**Style:** Dark-first UI. High contrast, low saturation.
**Inspiration:** Terminals, Editors (VS Code, Neovim), System Tools.

### CSS Variables
Use global variables defined in `theme.css`:

```css
:root {
  --rust-orange: #FF4400;
  --terminal-bg: #1e1e1e;
  --terminal-header: #2d2d2d;
  --text-main: #e5e5e5;
  --text-dim: #a3a3a3;
}
```

**Rule:** Hardcoded colors inside individual component files are discouraged.

---

## Animations & Motion

**Rules:**
- Subtle.
- Purposeful.
- Never flashy.

**Allowed:** `slide`, `fade`, `ease-out`.
**Forbidden:** `bounce`, `elastic`, `infinite animation loops`.

> If animation draws attention to itself, it is wrong.

---

## CSS-in-Rust (Leptos / Stylist)

Allowed **only** if:
1.  Style is dynamic.
2.  Driven by state.
3.  Cannot be expressed statically.

**Avoid:**
- Embedding large CSS blocks in Rust files.
- Styling entire layouts from Rust.

**Principle:** CSS belongs in CSS files.

---

## Accessibility & UX

- Respect "prefers-reduced-motion".
- Click targets must be obvious.
- Terminal metaphors must not block usability.

This is a **tool**, not a toy.

---

## Mental Model for Agents

When writing CSS, ask:
> **"Would a Rust developer respect this?"**

- "It looks cool but I don’t know how it works" → ❌ **Reject**
- "This feels like a system tool UI" → ✅ **Approve**

---

## Final Rule

1.  **Clarity** > Cleverness
2.  **Explicit** > Implicit
3.  **Deterministic** > Convenient

**Write CSS like you write Rust.**
