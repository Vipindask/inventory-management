---
name: vue-saas-redesign
description: Redesign a Vue 3 application's UI into a modern SaaS-style interface with a left vertical navigation sidebar, a consistent spacing and radius scale, and polished cards, tables, and controls. Use this skill when asked to modernize, restyle, or redesign a Vue app's look and feel, convert a top nav bar or horizontal tabs to a sidebar layout, introduce design tokens or a design system, or fix inconsistent spacing across .vue components.
---

# Vue 3 SaaS Redesign

Convert a Vue 3 application from a top navigation bar to a modern SaaS layout: a fixed left sidebar, a token-driven spacing scale, and a consistent component surface. This skill discovers the target app's structure rather than assuming it — file names, class names, and palettes vary between projects.

## When to Use

- Converting a top nav bar or horizontal tab strip to a vertical left sidebar
- Modernizing a Vue app that looks dated or inconsistent
- Introducing design tokens where components use hard-coded spacing and colors
- Normalizing spacing, radii, and shadows that drifted across many `.vue` files

## Procedure

Work in four phases, in order. Do not skip Phase 1 — every later decision depends on what it finds.

1. **Audit** — discover the shell, the global styles, the nav, and the existing scale
2. **Tokens** — install a `:root` scale derived from what the app already uses
3. **Shell** — replace the top nav with a sidebar and rework the layout grid
4. **Views** — migrate components onto the tokens

## Phase 1: Audit

Never assume file names. `App.vue` is a convention, not a guarantee, and the shell may live in a layout component. Run these before editing anything.

```bash
# Which files define the app shell and routing?
grep -rln "router-view" --include='*.vue' src/
grep -rln "router-link" --include='*.vue' src/

# Which file holds global (unscoped) CSS? Tokens go here.
grep -rln "<style>" --include='*.vue' src/

# What spacing values are actually in use? This is the inconsistency evidence.
grep -rhoE "border-radius: [^;]+;" --include='*.vue' src/ | sort | uniq -c | sort -rn
grep -rhoE "padding: [^;]+;" --include='*.vue' src/ | sort | uniq -c | sort -rn
grep -rhoE "gap: [^;]+;" --include='*.vue' src/ | sort | uniq -c | sort -rn

# What colors exist? Derive the new palette from these, don't invent one.
grep -rhoE "#[0-9a-fA-F]{3,8}\b" --include='*.vue' src/ | sort | uniq -c | sort -rn | head -30

# Anything sticky? Its offset is tied to the top nav height and WILL break.
grep -rn "position: sticky" -A 3 --include='*.vue' src/

# Any existing responsive behavior to preserve?
grep -rc "@media" --include='*.vue' src/ | grep -v ":0"
```

Record before proceeding:

- **Shell file** and the class name of its root element
- **Global style file** — the one component with an unscoped `<style>` block
- **Nav markup** — how links are rendered and how the active class is computed
- **Sticky elements** and their `top` offsets
- **Top nav height** — the number those offsets are derived from
- **Radius/padding/gap value counts** — the concrete case for tokenizing
- **Whether any `@media` queries exist** — if zero, the app is not responsive and the sidebar must introduce breakpoints, not preserve them

Report the audit to the user before making changes. A count like "7 distinct border-radius values across 15 scoped style blocks" is the justification for the work.

## Phase 2: Install Design Tokens

Add a `:root` block at the top of the global style block. Full scale and the ad-hoc-value mapping table are in [design-tokens.md](./references/design-tokens.md).

Derive colors from the audit. If the app already uses a slate neutral ramp and a blue accent, keep them — a redesign that silently changes brand hue is a bug, not a polish pass. Tokens rename existing values; they do not replace the palette.

```css
:root {
  /* Spacing — 4px base */
  --space-1: 4px;   --space-2: 8px;   --space-3: 12px;
  --space-4: 16px;  --space-6: 24px;  --space-8: 32px;  --space-12: 48px;

  /* Radius — collapse the audited values onto three steps */
  --radius-sm: 6px; --radius-md: 8px; --radius-lg: 12px; --radius-full: 9999px;

  /* Elevation */
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-md: 0 2px 8px rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 8px 24px rgba(15, 23, 42, 0.10);

  /* Layout */
  --sidebar-width: 248px;
  --sidebar-width-collapsed: 68px;
  --content-max: 1440px;
}
```

Add semantic color roles alongside — surface, border, text-primary, text-muted, accent, and the status set. Map them to the app's existing hex values rather than new ones.

## Phase 3: Build the Sidebar Shell

The core transformation. Complete markup, collapse behavior, and both responsive variants are in [sidebar-patterns.md](./references/sidebar-patterns.md).

### Shell layout

The root element changes from a vertical flex stack to a two-column grid.

```css
/* Before: header on top, content below */
.app { display: flex; flex-direction: column; min-height: 100vh; }

/* After: sidebar beside content */
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
}
```

The sidebar is `position: fixed` so it does not scroll with content; the content column takes a matching `margin-left`. Grid alone is not enough — without `fixed`, a long page scrolls the nav out of view.

### Sidebar structure

Three stacked regions: brand at the top, nav in the middle, utility controls pinned to the bottom with `margin-top: auto`. Anything that sat to the right of the old top nav — locale switcher, profile menu, notifications — moves into that bottom region.

```
┌──────────────────┐
│ Brand / logo     │
├──────────────────┤
│ ▸ Nav item       │
│ ▸ Nav item       │  ← flex: 1
│ ▸ Nav item       │
├──────────────────┤
│ Locale · Profile │  ← margin-top: auto
└──────────────────┘
```

### Nav items

Horizontal tabs become vertical rows. The active treatment must change with the orientation: a bottom-border underline reads as "selected tab" horizontally but as an arbitrary rule vertically. Use a filled tint plus a left accent bar.

```css
.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-4);
  border-radius: var(--radius-md);
  color: var(--text-muted);
  text-decoration: none;
  transition: all 0.15s ease;
}
.sidebar-nav a:hover { background: var(--surface-hover); color: var(--text-primary); }
.sidebar-nav a.active { background: var(--accent-soft); color: var(--accent); font-weight: 600; }
```

Preserve however the app computes the active class. If it uses `:class="{ active: $route.path === '/x' }"`, keep that — switching to `router-link-active` changes matching semantics for nested routes.

### Relocated header controls

Locale switchers and profile menus moved into the footer need three fixes, detailed in [sidebar-patterns.md](./references/sidebar-patterns.md): their dropdowns must open **upward** (and sideways when collapsed), their bordered-pill styling must be stripped to match nav rows, and their scoped styles need `:global(.sidebar.collapsed)` to react to the shell's state class.

### Recompute sticky offsets

**This is the step most often missed.** Any element with `top: <nav-height>px` was positioned under a top nav that no longer exists. Once the nav is vertical, those offsets must become `top: 0` — otherwise the element floats down the page leaving a gap.

Check every hit from the Phase 1 sticky grep. Also drop the `max-width` + `margin: 0 auto` centering from the old nav container; the sidebar now defines the left edge.

## Phase 4: Migrate Views

Mechanical pass over each `.vue` file, replacing audited literals with tokens. Before/after pairs for each recurring component are in [migration-recipes.md](./references/migration-recipes.md).

Rules:

- **Preserve every existing class name.** Renaming `.stat-card` to `.metric-tile` means editing every template that uses it. A styling pass changes styles, not markup contracts.
- **Never change template logic.** No `v-if` edits, no computed property changes, no data flow changes in a redesign.
- **Only the shell's global block gets tokens.** Scoped blocks consume them via `var()`; they do not redeclare them.
- **One component at a time**, verifying in the browser between each. A token typo silently falls back to the property's initial value, which is easy to miss in a bulk edit.

## Verification

```bash
npm run dev
```

Walk every route and confirm:

- No horizontal scrollbar at any viewport width
- Active nav state correct on each route, including the default `/`
- No gap where a sticky element used to sit under the top nav
- Sidebar collapses at the breakpoint and the toggle restores it
- Every view kept its layout — no cards spanning full width that used to be in a grid
- Modals and overlays still sit above the sidebar (check `z-index` against the sidebar's)

Screenshot before and after for the diff.

## Best Practices

1. **Audit before editing.** The grep output is both the plan and the justification.
2. **Derive, don't impose.** Pull the palette from the app's existing colors.
3. **Tokens rename, they don't restyle.** Phase 2 should be visually near-invisible; the change lands in Phase 3.
4. **Fixed sidebar, matching content offset.** Grid alone lets the nav scroll away.
5. **Change the active-state idiom with the orientation.** Underline for tabs, filled tint for rows.
6. **Recompute every sticky offset.** They were all relative to a nav height that is now zero.
7. **Preserve class names and template logic.** Redesign touches CSS.
8. **Introduce breakpoints if none exist.** A fixed sidebar on a narrow viewport is unusable.
9. **Verify per component.** Bulk token edits fail silently.
10. **Keep the sidebar under 280px.** Wider steals content width without adding legibility.

## Key Reminders

- **Do not assume `App.vue`** — find the shell by grepping for `router-view`.
- **`top: 70px` becomes `top: 0`** — the single most common breakage.
- **Utility controls move to the sidebar bottom**, not into the content area.
- **A missing `var()` fallback is invisible** — the property just resets.
- **`:global()` must wrap the whole selector** — `:global(.a) .b` silently compiles to `.a`, so a rule meant to hide a label ends up hiding its container. If a region vanishes, read the compiled CSS first.
- **Zero `@media` queries means the app was never responsive** — adding the sidebar makes that a problem you now own.
- **Modal `z-index` must exceed the sidebar's**, or dialogs render behind it.
