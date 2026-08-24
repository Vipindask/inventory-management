# Design Tokens

The full token scale to install in the global style block, plus the mapping from common ad-hoc values.

## Complete `:root` block

```css
:root {
  /* ---- Spacing: 4px base, no odd steps ---- */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;

  /* ---- Radius: three steps plus pill ---- */
  --radius-sm: 6px;    /* inputs, badges, small controls */
  --radius-md: 8px;    /* buttons, dropdowns */
  --radius-lg: 12px;   /* cards, modals, panels */
  --radius-full: 9999px;

  /* ---- Elevation: low and diffuse, tinted with the neutral ---- */
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-md: 0 2px 8px rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 8px 24px rgba(15, 23, 42, 0.10);

  /* ---- Surfaces ---- */
  --surface: #ffffff;
  --surface-sunken: #f8fafc;
  --surface-hover: #f1f5f9;
  --surface-sidebar: #ffffff;

  /* ---- Borders ---- */
  --border: #e2e8f0;
  --border-strong: #cbd5e1;

  /* ---- Text ---- */
  --text-primary: #0f172a;
  --text-body: #334155;
  --text-muted: #64748b;
  --text-subtle: #94a3b8;
  --text-inverse: #ffffff;

  /* ---- Accent ---- */
  --accent: #2563eb;
  --accent-hover: #1d4ed8;
  --accent-soft: #eff6ff;

  /* ---- Status ---- */
  --success: #059669;  --success-soft: #ecfdf5;
  --warning: #d97706;  --warning-soft: #fffbeb;
  --danger:  #dc2626;  --danger-soft:  #fef2f2;
  --info:    #2563eb;  --info-soft:    #eff6ff;

  /* Add a -border variant only where a status surface needs a visible outline
     (error banners, warning callouts). Don't add all four pre-emptively. */
  --danger-border: #fecaca;

  /* ---- Layout ---- */
  --sidebar-width: 248px;
  --sidebar-width-collapsed: 68px;
  --content-max: 1440px;
  --header-height: 64px;

  /* ---- Motion ---- */
  --transition: all 0.15s ease;
  --transition-slow: all 0.25s ease;
}
```

## Deriving the palette from an existing app

The hex values above are a slate/blue default. **Replace them with what the audit found.** The grep in Phase 1 ranks colors by frequency; the top neutrals and the top accent are almost always the real palette.

Typical mapping from a slate-based Vue app:

| Audited value | Role | Token |
|---|---|---|
| `#f8fafc` | page background | `--surface-sunken` |
| `#f1f5f9` | hover fill | `--surface-hover` |
| `#e2e8f0` | default border | `--border` |
| `#cbd5e1` | hover border | `--border-strong` |
| `#94a3b8` | subtle text, icons | `--text-subtle` |
| `#64748b` | secondary text | `--text-muted` |
| `#334155` | body text | `--text-body` |
| `#0f172a` | headings | `--text-primary` |
| `#2563eb` / `#3b82f6` | accent | `--accent` |
| `#eff6ff` | accent tint | `--accent-soft` |

If the app uses two blues (a darker one for text-on-white and a lighter one for fills), keep both — map the darker to `--accent` and the lighter to a `--accent-bright` for chart series and focus rings.

## Collapsing ad-hoc values

The audit typically finds six to eight distinct radii and a dozen padding values. Map them onto the scale:

### Radius

| Found | Token | Note |
|---|---|---|
| `2px`, `3px`, `4px` | `--radius-sm` | round up; sub-6px reads as unintentional |
| `6px` | `--radius-sm` | |
| `8px` | `--radius-md` | |
| `10px`, `12px` | `--radius-lg` | 10px is almost always meant as 12px |
| `50%`, `9999px` | `--radius-full` | keep `50%` for square avatars |

### Padding and gap

| Found | Token |
|---|---|
| `0.25rem` / `4px` | `--space-1` |
| `0.5rem` / `8px` | `--space-2` |
| `0.75rem` / `12px` | `--space-3` |
| `1rem` / `16px` | `--space-4` |
| `1.25rem` / `20px` | `--space-5` |
| `1.5rem` / `24px` | `--space-6` |
| `2rem` / `32px` | `--space-8` |
| `3rem` / `48px` | `--space-12` |

Two-value padding maps componentwise: `0.625rem 1.25rem` → `var(--space-3) var(--space-5)`, accepting the 10px→12px rounding.

**Values that resist mapping are worth questioning.** A lone `0.875rem` or `1.75rem` is usually drift, not intent. Round to the nearest step unless it visibly breaks alignment.

## Focus rings

One ring definition, applied to every interactive element. Most apps have inconsistent or missing focus states; this is where accessibility is usually won or lost.

```css
--ring: 0 0 0 3px rgba(37, 99, 235, 0.12);
```

```css
.some-input:focus,
.some-button:focus-visible {
  outline: none;
  border-color: var(--accent);
  box-shadow: var(--ring);
}
```

Prefer `:focus-visible` on buttons so mouse clicks don't show the ring, and plain `:focus` on text inputs where it should always show.

## Typography

Tokenize only if the audit shows drift. A scale that works:

```css
--text-xs: 0.75rem;    --text-sm: 0.813rem;   --text-base: 0.875rem;
--text-md: 0.938rem;   --text-lg: 1.125rem;   --text-xl: 1.375rem;
--text-2xl: 1.875rem;
```

Keep the app's existing font stack. Changing typeface is a brand decision, not a layout one.
