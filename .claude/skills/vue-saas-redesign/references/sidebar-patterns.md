# Sidebar Patterns

Complete markup and CSS for the sidebar shell, plus collapse and responsive behavior.

## Shell markup

The shell keeps whatever component style the app already uses — Options API with `setup()`, `<script setup>`, or plain Options API. Do not convert component style during a redesign.

```vue
<template>
  <div class="app">
    <aside class="sidebar" :class="{ collapsed: isCollapsed }">
      <div class="sidebar-brand">
        <span class="brand-mark">{{ brandInitial }}</span>
        <div class="brand-text">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          :class="{ active: $route.path === item.path }"
        >
          <span class="nav-icon" v-html="item.icon"></span>
          <span class="nav-label">{{ t(item.labelKey) }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu />
        <button class="collapse-toggle" @click="isCollapsed = !isCollapsed">
          <!-- chevron icon -->
        </button>
      </div>
    </aside>

    <div class="app-main">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>
  </div>
</template>
```

Driving the nav from a `navItems` array rather than repeated hand-written links removes the duplication the old top nav usually carries, and makes the icon slot uniform. Keep the label keys pointing at the app's existing i18n paths.

If the app has a link with a hard-coded label (common — one tab someone forgot to translate), add the missing key rather than special-casing it in the array.

## Shell CSS

```css
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
  background: var(--surface-sunken);
}

.app.sidebar-collapsed {
  grid-template-columns: var(--sidebar-width-collapsed) 1fr;
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: var(--sidebar-width);
  display: flex;
  flex-direction: column;
  background: var(--surface-sidebar);
  border-right: 1px solid var(--border);
  z-index: 100;
  transition: width 0.2s ease;
}

.sidebar.collapsed { width: var(--sidebar-width-collapsed); }

.app-main {
  grid-column: 2;
  display: flex;
  flex-direction: column;
  min-width: 0;   /* prevents wide tables forcing horizontal overflow */
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max);
  margin: 0 auto;
  padding: var(--space-6) var(--space-8);
}
```

`min-width: 0` on the content column is not optional. Grid items default to `min-width: auto`, so a wide table inside will push the column past its track and produce a page-level horizontal scrollbar.

## Regions

```css
.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--border);
  min-height: var(--header-height);
}

.brand-mark {
  display: grid;
  place-items: center;
  width: 32px; height: 32px;
  flex-shrink: 0;
  border-radius: var(--radius-md);
  background: var(--accent);
  color: var(--text-inverse);
  font-weight: 700;
}

.brand-text h1 {
  font-size: var(--text-md);
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.01em;
}

.brand-subtitle {
  font-size: var(--text-xs);
  color: var(--text-muted);
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-4) var(--space-3);
  overflow-y: auto;
}

.sidebar-footer {
  margin-top: auto;               /* pins utilities to the bottom */
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3);
  border-top: 1px solid var(--border);
}
```

## Nav items

```css
.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-3);
  border-radius: var(--radius-md);
  color: var(--text-muted);
  font-size: var(--text-base);
  font-weight: 500;
  text-decoration: none;
  white-space: nowrap;
  transition: var(--transition);
  position: relative;
}

.sidebar-nav a:hover {
  background: var(--surface-hover);
  color: var(--text-primary);
}

.sidebar-nav a.active {
  background: var(--accent-soft);
  color: var(--accent);
  font-weight: 600;
}

/* Left accent bar — the vertical analogue of a tab underline */
.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: 0; top: 50%;
  transform: translateY(-50%);
  width: 3px; height: 60%;
  border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
  background: var(--accent);
}

.nav-icon {
  display: grid;
  place-items: center;
  width: 20px; height: 20px;
  flex-shrink: 0;
}

.nav-icon svg { width: 18px; height: 18px; }
```

### Active-state matching

Preserve whatever the app already does.

```vue
<!-- Manual path comparison — exact match, no nested-route matching -->
:class="{ active: $route.path === item.path }"

<!-- Router's own class — matches nested routes as active too -->
<!-- Switching to this changes behavior; only do it deliberately -->
```

The manual form is common in apps whose root route is `/`. Switching to `router-link-active` would mark `/` active on every page, since it prefix-matches. If the app uses `router-link-exact-active`, that is equivalent to the manual form.

## Collapsed variant

At the collapsed width only icons show. Labels are hidden but must stay in the DOM for screen readers.

```css
.sidebar.collapsed .nav-label,
.sidebar.collapsed .brand-text {
  opacity: 0;
  width: 0;
  overflow: hidden;
  pointer-events: none;
}

.sidebar.collapsed .sidebar-nav a { justify-content: center; padding: var(--space-3); }
.sidebar.collapsed .sidebar-brand { justify-content: center; padding: var(--space-5) 0; }
```

Add a native tooltip so collapsed icons remain identifiable:

```vue
<router-link :title="isCollapsed ? t(item.labelKey) : null" ...>
```

Persist the collapsed state if the app already persists other UI preferences — check for an existing `localStorage` convention and match its key naming rather than inventing one.

## Responsive behavior

If the audit found zero `@media` queries, the app was never responsive and the sidebar makes that everyone's problem: a 248px fixed panel on a 375px viewport leaves nothing.

```css
@media (max-width: 1024px) {
  .app { grid-template-columns: var(--sidebar-width-collapsed) 1fr; }
  .sidebar { width: var(--sidebar-width-collapsed); }
  .sidebar .nav-label, .sidebar .brand-text { opacity: 0; width: 0; overflow: hidden; }
}

@media (max-width: 768px) {
  .app { grid-template-columns: 1fr; }

  .sidebar {
    width: var(--sidebar-width);
    transform: translateX(-100%);
    transition: transform 0.2s ease;
    box-shadow: var(--shadow-lg);
  }
  .sidebar.open { transform: translateX(0); }

  .app-main { grid-column: 1; }
  .main-content { padding: var(--space-4); }
}

.sidebar-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.4);
  z-index: 99;                     /* below sidebar, above content */
}
```

Below 768px the sidebar becomes off-canvas with a tap-to-dismiss overlay, and the shell needs a hamburger trigger in the content area — the sidebar's own toggle is off-screen when closed.

## Sticky offset recomputation

Every element positioned relative to the old top nav needs its offset zeroed.

```css
/* Before — pinned below a 70px top nav */
.filters-bar {
  position: sticky;
  top: 70px;
  z-index: 90;
}

/* After — the nav is vertical; the content column starts at the top */
.filters-bar {
  position: sticky;
  top: 0;
  z-index: 90;
}
```

Also remove the centering the old nav container imposed:

```css
/* Before */
.filters-container { max-width: 1600px; margin: 0 auto; padding: 0 2rem; }

/* After — the sidebar defines the left edge; match the content padding */
.filters-container { max-width: var(--content-max); margin: 0 auto; padding: 0 var(--space-8); }
```

## Relocated header controls

Locale switchers, profile menus, and notification bells were built for a top nav. Moving them into the sidebar footer breaks three things, all easy to miss:

**1. Dropdowns open the wrong way.** A menu anchored `top: calc(100% + 8px)` opens downward off the bottom of the viewport once its trigger sits at the bottom of the sidebar. Flip it:

```css
/* Before — hangs below a header control */
.dropdown-menu { top: calc(100% + var(--space-2)); right: 0; }

/* After — rises from a footer control */
.dropdown-menu { bottom: calc(100% + var(--space-2)); left: 0; }
```

When the sidebar is collapsed, flip again to open sideways so the menu isn't trapped in a 68px column:

```css
:global(.sidebar.collapsed) .dropdown-menu {
  left: calc(100% + var(--space-2));
  bottom: 0;
}
```

**2. Bordered pill triggers look wrong.** A control styled as a bordered button reads as a form field when stacked under nav rows. Strip the border and background, and match the nav item's padding and radius so the footer reads as one list.

**3. Scoped styles can't see the sidebar's state class.** The `collapsed` class lives on the shell, not the child component. From inside a child's `<style scoped>`, reach it with `:global()` — and **wrap the entire selector**, not just the prefix:

```css
/* WRONG — compiles to `.sidebar.collapsed { display: none }`, which hides
   the whole sidebar. Vue drops the descendant part with no warning. */
:global(.sidebar.collapsed) .profile-name { display: none; }

/* RIGHT — the full selector goes inside :global() */
:global(.sidebar.collapsed .profile-name) { display: none; }
```

This failure is silent: the build succeeds, no warning is logged, and the symptom (an entire region vanishing) looks nothing like a selector bug. If a layout region disappears after restyling a child component, read the compiled CSS before anything else:

```bash
curl -s "http://localhost:3000/src/components/Foo.vue?vue&type=style&index=0&scoped=true&lang.css"
```

Then confirm no rule collapsed onto a container:

```bash
grep -oE "\.sidebar\{[^}]*display: *none"   # should match nothing
```

Repeat the same rules inside the `max-width: 1024px` breakpoint (auto-collapse) and reverse them inside `max-width: 768px`, where the drawer is full width and labels should return.

## Z-index ordering

Establish the order explicitly; sidebars and modals collide otherwise.

| Layer | z-index |
|---|---|
| Sticky sub-header | 90 |
| Mobile overlay | 99 |
| Sidebar | 100 |
| Dropdowns | 200 |
| Modals | 1000 |

Check existing modal z-indexes against the sidebar's. A modal at `z-index: 50` renders behind a sidebar at 100 — and if the modal uses `<Teleport to="body">` it escapes the shell's stacking context, so only the raw numbers matter.
