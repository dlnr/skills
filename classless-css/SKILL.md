---
name: classless-css
description: Write semantic, classless-first, DRY CSS using native HTML element selectors, modern CSS nesting for scoping, and CSS custom properties. Trigger whenever writing, refactoring, or reviewing HTML and CSS stylesheets, components, or layouts.
---

# Classless & Semantic CSS Guidelines

## Core Philosophy
Prioritize **semantic HTML** and **element-level styling** over utility classes or BEM-heavy architectures. HTML should remain clean, readable, and structural; CSS should do the heavy lifting through bare element selectors, modern native CSS nesting, relational pseudo-classes, and custom properties (variables).

---

## 1. HTML-First & Classless Selectors
- **Default to Bare HTML Elements:** Style elements directly (`body`, `main`, `header`, `footer`, `section`, `article`, `nav`, `aside`, `h1`–`h6`, `p`, `ul`, `ol`, `dl`, `figure`, `blockquote`, `dialog`, `details`, `summary`, `table`, `form`, `fieldset`, `label`, `input`, `select`, `textarea`, `button`).
- **Use Semantic Landmarks Instead of Divs:** Replace `<div class="header">`, `<div class="card">`, or `<div class="modal">` with `<header>`, `<article>`, or `<dialog>`.
- **Leverage HTML Attributes for Variants & States:** Before inventing a class, use native HTML attributes and ARIA states as styling hooks:
  - Input types: `input[type="search"]`, `input[type="checkbox"]`, `button[type="submit"]`
  - States: `[disabled]`, `[required]`, `[readonly]`, `[open]` (for `<details>` and `<dialog>`), `[aria-current="page"]`, `[aria-expanded="true"]`
  - Custom data attributes (when semantics aren't enough): `[data-variant="danger"]`, `[data-theme="dark"]`

---

## 2. Strategic Scoping with Native CSS Nesting
When classes are necessary (e.g., to distinguish specific layout regions or component contexts), apply **a single scope class on the root container** and style its children via native CSS nesting (`&`) and direct element selectors.

### Do This (Scoped Root + Element Nesting):
```html
<article class="pricing-tier">
  <header>
    <h3>Pro Plan</h3>
    <p>For growing teams</p>
  </header>
  <ul>
    <li>Unlimited projects</li>
    <li>Priority support</li>
  </ul>
  <footer>
    <button type="button">Select Plan</button>
  </footer>
</article>
```

```css
.pricing-tier {
  display: grid;
  gap: var(--space-md);
  padding: var(--space-lg);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);

  & > header {
    border-bottom: 1px solid var(--border-color);
    padding-bottom: var(--space-sm);

    & h3 {
      margin: 0;
      color: var(--color-primary);
    }

    & p {
      margin: 0;
      color: var(--text-muted);
    }
  }

  & > ul {
    list-style: "✓ ";
    padding-inline-start: var(--space-md);
  }

  & > footer button {
    width: 100%;
  }
}
```

### Avoid This (BEM / Utility Bloat):
```html
<!-- AVOID -->
<div class="pricing-tier">
  <div class="pricing-tier__header">
    <h3 class="pricing-tier__title">Pro Plan</h3>
    <p class="pricing-tier__subtitle">For growing teams</p>
  </div>
  <ul class="pricing-tier__list">
    <li class="pricing-tier__item">Unlimited projects</li>
  </ul>
  <div class="pricing-tier__footer">
    <button class="btn btn--primary btn--full">Select Plan</button>
  </div>
</div>
```

### Nesting Rules:
- **Max Nesting Depth:** Keep nesting to **2–3 levels maximum** to prevent overly brittle specificity and mirror only essential structural relationships.
- **Prefer Direct Child Combinators (`& > element`):** When scoping layout structure inside a component, use `>` so styles don't accidentally bleed into deeply nested sub-elements.
- **Nest Pseudo-Classes & Media Queries:** Keep element states (`&:hover`, `&:focus-visible`, `&:disabled`) and responsive tweaks (`@media`, `@container`) nested directly inside the element selector they modify.

---

## 3. DRY & Reusable Architecture
Write styles once at the highest sensible level and let inheritance and custom properties handle variations.

### A. Design Tokens via CSS Custom Properties
Define all colors, spacing, typography, borders, and shadows in `:root`. Never hardcode magic values across rules.
```css
:root {
  --font-sans: system-ui, -apple-system, sans-serif;
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;

  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;

  --surface-bg: #ffffff;
  --surface-muted: #f4f4f5;
  --text-main: #18181b;
  --text-muted: #71717a;
  --color-primary: #2563eb;
  --border-color: #e4e4e7;
}
```

### B. Contextual Token Overrides (Zero-Duplication Variants)
Instead of rewriting properties for variants or themes, override the custom properties inside the scoped selector or attribute:
```css
button,
[role="button"] {
  --_bg: var(--color-primary);
  --_text: #ffffff;

  background: var(--_bg);
  color: var(--_text);
  padding: var(--space-sm) var(--space-md);
  border: none;
  border-radius: var(--radius-sm);
  cursor: pointer;

  &[type="reset"],
  &[data-variant="secondary"] {
    --_bg: var(--surface-muted);
    --_text: var(--text-main);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
```

### C. Modern Selector Grouping (`:is()` and `:where()`)
- Use `:where()` for base resets and shared defaults to keep **zero specificity** (`0,0,0`), making future overrides effortless without `!important`.
- Use `:is()` to group shared typography, layout, or interactive patterns concisely:
```css
:where(article, section, dialog) {
  background: var(--surface-bg);
  border-radius: var(--radius-md);
}

:is(h1, h2, h3, h4) {
  line-height: 1.2;
  text-wrap: balance;
}

:is(input, select, textarea):not([type="checkbox"], [type="radio"]) {
  width: 100%;
  padding: var(--space-sm);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-sm);
  font: inherit;
}
```

### D. Relational & Structural Styling (`:has()` and Flow Rhythm)
- **Lobotomized Owl / Flow Spacing:** Manage vertical rhythm globally on content containers instead of adding margin classes to individual elements:
```css
:where(main, article, section, form) > * + * {
  margin-block-start: var(--flow-space, var(--space-md));
}
```
- **Parent-State Adaptation with `:has()`:** Adjust layout automatically based on child elements rather than toggling modifier classes in JS/HTML:
```css
/* Automatically style a label when its nested input is checked or invalid */
label:has(input:checked) {
  border-color: var(--color-primary);
  background: color-mix(in srgb, var(--color-primary) 10%, transparent);
}

/* Adjust header layout only when it contains a nav */
header:has(> nav) {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

---

## 4. Implementation Checklist for Every Task
When generating or refactoring HTML & CSS:
1. **Audit HTML first:** Strip unnecessary `<div>` wrappers and redundant classes. Replace them with semantic tags (`<article>`, `<section>`, `<nav>`, `<aside>`, `<header>`, `<footer>`, `<figure>`, `<dl>`, `<fieldset>`).
2. **Push styles to base elements:** Ensure forms, tables, buttons, typography, and lists look complete and polished using bare tag selectors alone.
3. **Scope only at the boundary:** Add a class only when multiple instances of the same semantic tag in the same context behave differently. Put the class on the parent wrapper and nest child element selectors inside it.
4. **Eliminate repetition:** Extract repeated values into `:root` custom properties, consolidate identical blocks with `:is()`/`:where()`, and handle variations by reassigning local custom properties (`--_prop`).

