# Migration Recipes

Before/after pairs for the components that recur in nearly every Vue app. Apply during Phase 4, one component at a time.

Class names in the "after" column are unchanged — only declarations differ. Renaming a class means editing every template that uses it, which is out of scope for a styling pass.

## Card

The most common surface. Typical drift: a 10px radius, a heavy shadow, and inconsistent header padding.

```css
/* Before */
.card {
  background: white;
  border-radius: 10px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  margin-bottom: 1.5rem;
}
.card-header {
  padding: 1.25rem 1.5rem;
  border-bottom: 1px solid #e2e8f0;
}
.card-title { font-size: 1.125rem; font-weight: 600; color: #0f172a; }

/* After */
.card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--space-6);
  overflow: hidden;            /* keeps child corners inside the radius */
}
.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
  padding: var(--space-4) var(--space-6);
  border-bottom: 1px solid var(--border);
}
.card-title {
  font-size: var(--text-lg);
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: -0.01em;
}
```

Adding a hairline border alongside a lighter shadow is what reads as "modern SaaS" — heavy shadow with no border reads as 2016 Material.

Making `.card-header` a flex row with `space-between` gives header-right actions a home without extra markup.

## Stat tile

```css
/* Before */
.stat-card {
  background: white;
  padding: 1.5rem;
  border-radius: 10px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  border-left: 4px solid #3b82f6;
}
.stat-label { font-size: 0.875rem; color: #64748b; }
.stat-value { font-size: 1.875rem; font-weight: 700; color: #0f172a; }

/* After */
.stat-card {
  position: relative;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  box-shadow: var(--shadow-sm);
  transition: var(--transition);
}
.stat-card:hover { box-shadow: var(--shadow-md); border-color: var(--border-strong); }

.stat-label {
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--text-muted);
}
.stat-value {
  margin-top: var(--space-1);
  font-size: var(--text-2xl);
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.02em;
}

/* Status variants — a top accent rule instead of a thick left border */
.stat-card.success::before,
.stat-card.warning::before,
.stat-card.danger::before,
.stat-card.info::before {
  content: '';
  position: absolute;
  inset: 0 0 auto 0;
  height: 3px;
  border-radius: var(--radius-lg) var(--radius-lg) 0 0;
}
.stat-card.success::before { background: var(--success); }
.stat-card.warning::before { background: var(--warning); }
.stat-card.danger::before  { background: var(--danger); }
.stat-card.info::before    { background: var(--info); }
```

Uppercase micro-label over a large tight-tracked number is the SaaS KPI idiom. The 4px left border is the dated part.

## Table

```css
/* Before */
table { width: 100%; border-collapse: collapse; }
th {
  text-align: left;
  padding: 0.75rem 1rem;
  background: #f8fafc;
  font-size: 0.875rem;
  color: #64748b;
  border-bottom: 1px solid #e2e8f0;
}
td { padding: 0.75rem 1rem; border-bottom: 1px solid #f1f5f9; }

/* After */
.table-container { overflow-x: auto; }

table { width: 100%; border-collapse: collapse; }

th {
  text-align: left;
  padding: var(--space-3) var(--space-4);
  background: var(--surface-sunken);
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--text-muted);
  border-bottom: 1px solid var(--border);
  white-space: nowrap;
}

td {
  padding: var(--space-3) var(--space-4);
  font-size: var(--text-base);
  color: var(--text-body);
  border-bottom: 1px solid var(--surface-hover);
}

tbody tr:last-child td { border-bottom: none; }
tbody tr { transition: background 0.1s ease; }
tbody tr:hover { background: var(--surface-sunken); }
```

`overflow-x: auto` on the wrapper keeps wide tables inside the content column instead of widening the page. Dropping the last row's border avoids a double line against the card edge.

## Badge

```css
/* Before */
.badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 500;
}
.badge.success { background: #d1fae5; color: #065f46; }

/* After */
.badge {
  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  padding: var(--space-1) var(--space-2);
  border-radius: var(--radius-sm);
  font-size: var(--text-xs);
  font-weight: 600;
  line-height: 1.4;
  white-space: nowrap;
}
.badge.success { background: var(--success-soft); color: var(--success); }
.badge.warning { background: var(--warning-soft); color: var(--warning); }
.badge.danger  { background: var(--danger-soft);  color: var(--danger); }
.badge.info    { background: var(--info-soft);    color: var(--info); }
```

`inline-flex` with a gap lets a status dot or icon sit inside the badge later without markup churn.

## Buttons

Many apps have a secondary button but no primary. Define both.

```css
.btn-primary,
.btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  padding: var(--space-3) var(--space-5);
  border-radius: var(--radius-md);
  font-family: inherit;
  font-size: var(--text-base);
  font-weight: 600;
  cursor: pointer;
  transition: var(--transition);
  border: 1px solid transparent;
}

.btn-primary {
  background: var(--accent);
  color: var(--text-inverse);
}
.btn-primary:hover:not(:disabled) { background: var(--accent-hover); }

.btn-secondary {
  background: var(--surface);
  border-color: var(--border);
  color: var(--text-body);
}
.btn-secondary:hover:not(:disabled) {
  background: var(--surface-hover);
  border-color: var(--border-strong);
}

.btn-primary:disabled,
.btn-secondary:disabled { opacity: 0.5; cursor: not-allowed; }

.btn-primary:focus-visible,
.btn-secondary:focus-visible { outline: none; box-shadow: var(--ring); }
```

`font-family: inherit` matters — buttons default to the system UI font and will otherwise not match surrounding text.

## Form controls

```css
/* Before */
.filter-select {
  padding: 0.4rem 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.813rem;
}
.filter-select:focus { border-color: #3b82f6; box-shadow: 0 0 0 3px rgba(59,130,246,0.1); }

/* After */
.filter-select,
.text-input {
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-sm);
  background: var(--surface);
  font-family: inherit;
  font-size: var(--text-sm);
  color: var(--text-body);
  transition: var(--transition);
}
.filter-select:hover, .text-input:hover { border-color: var(--text-subtle); }
.filter-select:focus, .text-input:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: var(--ring);
}
```

If the app already had a good focus ring, this is a rename, not a restyle. Apply the same ring everywhere — that consistency is most of the "polish".

## Page header

```css
/* Before */
.page-header { margin-bottom: 1.5rem; }
.page-header h2 { font-size: 1.875rem; font-weight: 700; }
.page-header p { color: #64748b; font-size: 0.938rem; }

/* After */
.page-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-4);
  margin-bottom: var(--space-6);
}
.page-header h2 {
  font-size: var(--text-2xl);
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.02em;
  margin-bottom: var(--space-1);
}
.page-header p { color: var(--text-muted); font-size: var(--text-md); }
```

Wrap the heading and description in a div so `space-between` can push page-level actions right.

## Grid layouts

```css
/* Before */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

/* After */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-4);
  margin-bottom: var(--space-6);
}
```

Lower the `minmax` floor when the sidebar takes horizontal space — a 280px floor that fit four tiles across full width will drop to three once 248px goes to the sidebar.

## Modal

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.5);
  display: grid;
  place-items: center;
  padding: var(--space-4);
  z-index: 1000;                 /* must exceed the sidebar */
}
.modal-container {
  background: var(--surface);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  width: 100%;
  max-width: 560px;
  max-height: 90vh;
  overflow-y: auto;
}
.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-5) var(--space-6);
  border-bottom: 1px solid var(--border);
}
.modal-body { padding: var(--space-6); }
.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: var(--space-3);
  padding: var(--space-4) var(--space-6);
  border-top: 1px solid var(--border);
}
```

Check every modal's `z-index` against the sidebar's. Teleported modals escape the shell's stacking context, so the raw number is what decides.

## Checklist per file

- [ ] Every hard-coded hex replaced with a semantic token
- [ ] Every padding, gap, and radius on the scale
- [ ] Focus states on all interactive elements
- [ ] Class names unchanged
- [ ] Template markup unchanged except where a wrapper was needed for flex
- [ ] Verified in the browser before moving on
