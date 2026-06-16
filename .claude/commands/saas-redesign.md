---
description: Redesign the Vue 3 app UI into a modern SaaS-style interface with a vertical sidebar navigation
---

Redesign this Vue 3 application's UI from a horizontal top-nav layout to a modern SaaS-style layout with a vertical sidebar on the left. The result should look like a polished B2B dashboard product (think Linear, Vercel, or Retool).

---

## Phase 1 — Audit the current layout

Read these files before making any changes:
- `client/src/App.vue` — identify all nav links, their routes, labels, and any slots (filter bar, profile menu, language switcher, modals)
- `client/src/composables/useI18n.js` and `client/src/locales/en.js` — find translated nav labels
- `client/src/components/FilterBar.vue` — understand what it renders so you can reposition it

Build a mental map of: nav items (route + label), utility controls (profile, language), and the filter bar placement.

---

## Phase 2 — Design system to apply

Use this exact design system. Do not invent different values.

### Color tokens (CSS variables on `:root`)
```css
--color-sidebar-bg:      #0f172a;   /* deep navy */
--color-sidebar-border:  #1e293b;
--color-sidebar-text:    #94a3b8;
--color-sidebar-text-active: #f1f5f9;
--color-sidebar-hover:   #1e293b;
--color-sidebar-active:  #1e3a5f;   /* active item bg */
--color-accent:          #3b82f6;   /* blue accent for active indicator */

--color-content-bg:      #f8fafc;
--color-content-border:  #e2e8f0;

--color-surface:         #ffffff;
--color-surface-raised:  #f1f5f9;

--color-text-primary:    #0f172a;
--color-text-secondary:  #64748b;
--color-text-muted:      #94a3b8;

--color-success:   #10b981;
--color-warning:   #f59e0b;
--color-danger:    #ef4444;
--color-info:      #3b82f6;
```

### Spacing scale
All spacing should use multiples of 4px: `4 8 12 16 20 24 32 40 48`.

### Typography
```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
font-size base: 14px;
line-height: 1.5;
```

### Sidebar dimensions
- Width (open): `220px`
- Icon column (if icons used): `20px`
- Nav item height: `36px`
- Sidebar padding: `12px`

### Cards and surfaces
- `border-radius: 8px` on cards
- `border: 1px solid var(--color-content-border)` on cards
- `box-shadow: 0 1px 3px rgba(0,0,0,0.06)` subtle lift on cards
- Content area padding: `24px`

---

## Phase 3 — Rewrite App.vue (MANDATORY: delegate to vue-expert)

**YOU MUST use the vue-expert subagent** for this step per the MANDATORY RULE in CLAUDE.md.

Give vue-expert the full brief:

### Layout structure to build

Replace the `<header class="top-nav">` with a two-column layout:

```
┌──────────────────────────────────────────────────────┐
│ sidebar (220px fixed) │ main area (flex: 1)           │
│                       │                               │
│  [Logo + App name]    │  [FilterBar]                  │
│                       │  [<router-view />]            │
│  [Nav items list]     │                               │
│                       │                               │
│  ─────────────────    │                               │
│  [User profile row]   │                               │
│  [Language switcher]  │                               │
└──────────────────────────────────────────────────────┘
```

### Sidebar HTML structure

```html
<aside class="sidebar">
  <div class="sidebar-header">
    <div class="sidebar-logo">
      <h1 class="sidebar-brand">{{ t('nav.companyName') }}</h1>
      <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
    </div>
  </div>

  <nav class="sidebar-nav">
    <router-link class="sidebar-item" to="/"          :class="{ active: $route.path === '/' }">{{ t('nav.overview') }}</router-link>
    <router-link class="sidebar-item" to="/inventory" :class="{ active: $route.path === '/inventory' }">{{ t('nav.inventory') }}</router-link>
    <!-- ...all other routes... -->
  </nav>

  <div class="sidebar-footer">
    <LanguageSwitcher />
    <ProfileMenu @show-profile-details="..." @show-tasks="..." />
  </div>
</aside>

<div class="content-wrapper">
  <FilterBar />
  <main class="main-content">
    <router-view />
  </main>
</div>
```

### Sidebar CSS rules to write

```css
.app {
  display: flex;
  min-height: 100vh;
  background: var(--color-content-bg);
}

.sidebar {
  width: 220px;
  flex-shrink: 0;
  background: var(--color-sidebar-bg);
  border-right: 1px solid var(--color-sidebar-border);
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  z-index: 100;
  overflow-y: auto;
}

.sidebar-header {
  padding: 20px 16px 12px;
  border-bottom: 1px solid var(--color-sidebar-border);
}

.sidebar-brand {
  font-size: 14px;
  font-weight: 700;
  color: var(--color-sidebar-text-active);
  letter-spacing: -0.2px;
  margin: 0;
}

.sidebar-subtitle {
  font-size: 11px;
  color: var(--color-sidebar-text);
  display: block;
  margin-top: 2px;
}

.sidebar-nav {
  flex: 1;
  padding: 8px 12px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-item {
  display: flex;
  align-items: center;
  height: 36px;
  padding: 0 12px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  color: var(--color-sidebar-text);
  text-decoration: none;
  transition: background 0.12s, color 0.12s;
}

.sidebar-item:hover {
  background: var(--color-sidebar-hover);
  color: var(--color-sidebar-text-active);
}

.sidebar-item.active {
  background: var(--color-sidebar-active);
  color: var(--color-sidebar-text-active);
  font-weight: 600;
  border-left: 2px solid var(--color-accent);
  padding-left: 10px; /* compensate for 2px border */
}

.sidebar-footer {
  padding: 12px;
  border-top: 1px solid var(--color-sidebar-border);
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.content-wrapper {
  margin-left: 220px;   /* offset for fixed sidebar */
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main-content {
  flex: 1;
  padding: 24px;
}
```

### Other rules for vue-expert
- Keep ALL existing modal components (`ProfileDetailsModal`, `TasksModal`) in the template — just at the bottom of `.app`
- Keep ALL existing script/composable logic untouched — only restructure template HTML and replace the top-nav CSS with sidebar CSS
- Move global CSS variables (color tokens from Phase 2) into the `:root` block at the top of the `<style>` section
- Remove the old `.top-nav`, `.nav-container`, `.nav-tabs` rules
- The `FilterBar` stays at the top of `.content-wrapper`, above `<main>`
- Do not add emojis or icons to nav items (text-only labels are correct per the design system)

---

## Phase 4 — Update card and table styles globally

After App.vue is done, still using vue-expert, scan `client/src/App.vue`'s existing global styles for `.card`, `.table`, `.stat-card`, and badge styles. Update them to match the new design system:

- Cards: use `var(--color-surface)`, add `border-radius: 8px`, `border: 1px solid var(--color-content-border)`, `box-shadow: 0 1px 3px rgba(0,0,0,0.06)`
- Table headers: `background: var(--color-surface-raised)`, `color: var(--color-text-secondary)`
- Stat card values: `color: var(--color-text-primary)`, larger `font-size` (24–28px), `font-weight: 700`
- Preserve all existing status badge colors (success/warning/danger/info)

---

## Phase 5 — Verify the result

1. Start both servers using the `/start` skill (or manually: backend on 8001, frontend on 3000)
2. Use Playwright (`mcp__playwright__*`) to screenshot `http://localhost:3000`
3. Check all 6 routes render without layout breakage
4. Confirm the sidebar is visible and the active route is highlighted
5. Confirm modals still open from the sidebar footer profile menu
6. If anything is broken, fix it before reporting done

---

## Definition of done

- [ ] Sidebar is visible on all routes with the correct active state
- [ ] No horizontal top nav bar exists anymore
- [ ] Content area has correct left margin / is not hidden behind sidebar
- [ ] FilterBar sits above the main content area, inside the content wrapper
- [ ] Profile and language controls are in the sidebar footer
- [ ] Modals still function
- [ ] CSS uses the design token variables (no raw hex colors in component logic)
- [ ] No console errors in the browser
