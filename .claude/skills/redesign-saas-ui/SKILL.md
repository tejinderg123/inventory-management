---
name: redesign-saas-ui
description: Transform a Vue 3 app from a horizontal top-nav bar to a modern SaaS layout with a vertical sidebar, polished typography, and consistent spacing. Delegates all .vue file changes to vue-expert.
---

# SaaS UI Redesign Skill

Transforms the app shell from a horizontal top-nav to a left-sidebar SaaS layout. All `.vue` file modifications **must** be delegated to `vue-expert` (CLAUDE.md rule).

---

## Design System

Apply these tokens consistently across the entire redesign. Do not deviate — visual consistency depends on them.

**Color palette:**
- `--sidebar-bg`: `#0f172a` — sidebar background (deep navy)
- `--sidebar-bg-hover`: `#1e293b` — nav item hover
- `--sidebar-bg-active`: `#1d4ed8` — active nav item pill
- `--sidebar-text`: `#94a3b8` — default nav text (slate-400)
- `--sidebar-text-active`: `#ffffff` — active nav text
- `--sidebar-text-hover`: `#e2e8f0` — hover nav text
- `--sidebar-border`: `#1e293b` — sidebar internal dividers
- `--content-bg`: `#f8fafc` — main content area background
- `--topbar-bg`: `#ffffff` — filter bar strip background
- `--topbar-border`: `#e2e8f0` — filter bar bottom border

**Spacing rhythm:** `4px` base unit. Use multiples: `8`, `12`, `16`, `20`, `24`, `32`, `40`.

**Sidebar dimensions:**
- Width: `240px`
- Brand area height: `64px`
- Nav item height: `40px`, with `8px` vertical padding and `16px` horizontal padding
- Nav item border-radius: `8px`
- Active indicator: full pill (bg color), not just a left border

**Typography (no changes from existing):** Inherit the existing `Inter` font stack. Nav labels are `0.875rem`, weight `500`.

---

## Target Layout Structure

The finished `App.vue` template shell should look like this (exact implementation delegated to vue-expert):

```
.app-shell                     ← flex row, height: 100vh, overflow: hidden
  aside.sidebar                ← 240px wide, flex column, overflow-y: auto
    .sidebar-brand             ← 64px tall, logo + app name + subtitle
    nav.sidebar-nav            ← flex column, gap: 4px, padding: 16px 12px
      router-link (× 7)        ← each: flex row, icon + label, active class
    .sidebar-footer            ← margin-top: auto, LanguageSwitcher + ProfileMenu
  .app-body                    ← flex 1, flex column, overflow: hidden
    .topbar                    ← FilterBar lives here, bg white, border-bottom
    main.main-content          ← flex 1, overflow-y: auto, padding: 24px 32px
      router-view
  ProfileDetailsModal          ← unchanged
  TasksModal                   ← unchanged
```

---

## Nav Link Icons

Use these inline SVG paths for each route. All icons use `viewBox="0 0 20 20"`, `fill="none"`, `stroke="currentColor"`, `stroke-width="1.5"`, size `18×18`.

| Route | Label | SVG path(s) |
|---|---|---|
| `/` | Overview | `M3 10.5L10 3l7 7.5M5 9v8h3.5v-4h3V17H15V9` |
| `/inventory` | Inventory | `M3 8l7-5 7 5v9a1 1 0 01-1 1H4a1 1 0 01-1-1V8z M8 17V12h4v5` |
| `/orders` | Orders | `M4 4h12v2H4zm0 5h12v2H4zm0 5h8v2H4z` |
| `/spending` | Finance | `M10 2v16M5 6h8.5a2 2 0 010 4H6.5a2 2 0 000 4H15` |
| `/demand` | Demand | `M3 14l4-5 4 3 4-7 4 4` |
| `/reports` | Reports | `M3 17V9m4 8V4m4 13V7m4 10V2` |
| `/restocking` | Restocking | `M4 10a6 6 0 1111.9-1M4 10l-2 2m2-2l2 2` |

---

## Phase 1: Audit the current App.vue

Read `client/src/App.vue` and identify:
1. The existing `.top-nav` block (header element with nav-container, logo, nav-tabs, LanguageSwitcher, ProfileMenu)
2. The `<FilterBar />` placement (currently between header and main)
3. The `.main-content` element (currently `max-width: 1600px; margin: 0 auto; padding: 1.5rem 2rem`)
4. All global styles in the `<style>` block that reference `.top-nav`, `.nav-container`, `.nav-tabs`, `.logo`, `.subtitle`

These are the only sections that change. All view files, composables, and other components remain untouched.

---

## Phase 2: Delegate App.vue transformation to vue-expert

Pass vue-expert the following precise instructions. Do not rephrase them — include the exact CSS tokens and structure from this skill.

**Instruction payload for vue-expert:**

> Transform `client/src/App.vue` from a top-nav layout to a SaaS sidebar layout.
>
> **Template changes:**
> Replace the entire `<header class="top-nav">...</header>` block and `<FilterBar />` and `<main class="main-content">` with the following structure:
>
> ```html
> <div class="app-shell">
>   <aside class="sidebar">
>     <div class="sidebar-brand">
>       <span class="sidebar-brand-name">{{ t('nav.companyName') }}</span>
>       <span class="sidebar-brand-sub">{{ t('nav.subtitle') }}</span>
>     </div>
>     <nav class="sidebar-nav">
>       <router-link to="/" :class="{ active: $route.path === '/' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M3 10.5L10 3l7 7.5M5 9v8h3.5v-4h3V17H15V9"/></svg>
>         <span>{{ t('nav.overview') }}</span>
>       </router-link>
>       <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M3 8l7-5 7 5v9a1 1 0 01-1 1H4a1 1 0 01-1-1V8z M8 17V12h4v5"/></svg>
>         <span>{{ t('nav.inventory') }}</span>
>       </router-link>
>       <router-link to="/orders" :class="{ active: $route.path === '/orders' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M4 4h12v2H4zm0 5h12v2H4zm0 5h8v2H4z"/></svg>
>         <span>{{ t('nav.orders') }}</span>
>       </router-link>
>       <router-link to="/spending" :class="{ active: $route.path === '/spending' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M10 2v16M5 6h8.5a2 2 0 010 4H6.5a2 2 0 000 4H15"/></svg>
>         <span>{{ t('nav.finance') }}</span>
>       </router-link>
>       <router-link to="/demand" :class="{ active: $route.path === '/demand' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M3 14l4-5 4 3 4-7 4 4"/></svg>
>         <span>{{ t('nav.demandForecast') }}</span>
>       </router-link>
>       <router-link to="/reports" :class="{ active: $route.path === '/reports' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M3 17V9m4 8V4m4 13V7m4 10V2"/></svg>
>         <span>Reports</span>
>       </router-link>
>       <router-link to="/restocking" :class="{ active: $route.path === '/restocking' }" class="nav-item">
>         <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M4 10a6 6 0 1111.9-1M4 10l-2 2m2-2l2 2"/></svg>
>         <span>Restocking</span>
>       </router-link>
>     </nav>
>     <div class="sidebar-footer">
>       <LanguageSwitcher />
>       <ProfileMenu
>         @show-profile-details="showProfileDetails = true"
>         @show-tasks="showTasks = true"
>       />
>     </div>
>   </aside>
>   <div class="app-body">
>     <div class="topbar">
>       <FilterBar />
>     </div>
>     <main class="main-content">
>       <router-view />
>     </main>
>   </div>
> </div>
> ```
>
> Wrap the above plus the existing modals inside a single `<div class="app">` (keep outer wrapper, just change internals).
>
> **CSS changes — replace the old nav/layout styles with these:**
>
> Remove these CSS blocks entirely:
> - `.top-nav`, `.nav-container`, `.nav-tabs`, `.nav-tabs a`, `.nav-tabs a:hover`, `.nav-tabs a.active`, `.nav-tabs a.active::after`, `.logo`, `.subtitle`, `.nav-container > .nav-tabs`, `.nav-container > .language-switcher`
>
> Add these new rules (append to the `<style>` block):
>
> ```css
> .app {
>   display: flex;
>   flex-direction: row;
>   height: 100vh;
>   overflow: hidden;
> }
>
> .sidebar {
>   width: 240px;
>   min-width: 240px;
>   background: #0f172a;
>   display: flex;
>   flex-direction: column;
>   overflow-y: auto;
>   overflow-x: hidden;
> }
>
> .sidebar-brand {
>   height: 64px;
>   min-height: 64px;
>   display: flex;
>   flex-direction: column;
>   justify-content: center;
>   padding: 0 20px;
>   border-bottom: 1px solid #1e293b;
> }
>
> .sidebar-brand-name {
>   font-size: 0.9375rem;
>   font-weight: 700;
>   color: #ffffff;
>   letter-spacing: -0.02em;
>   line-height: 1.2;
> }
>
> .sidebar-brand-sub {
>   font-size: 0.6875rem;
>   color: #64748b;
>   font-weight: 400;
>   margin-top: 2px;
> }
>
> .sidebar-nav {
>   display: flex;
>   flex-direction: column;
>   gap: 2px;
>   padding: 16px 12px;
>   flex: 1;
> }
>
> .nav-item {
>   display: flex;
>   align-items: center;
>   gap: 10px;
>   padding: 9px 12px;
>   border-radius: 8px;
>   color: #94a3b8;
>   text-decoration: none;
>   font-size: 0.875rem;
>   font-weight: 500;
>   transition: background 0.15s ease, color 0.15s ease;
>   white-space: nowrap;
> }
>
> .nav-item:hover {
>   background: #1e293b;
>   color: #e2e8f0;
> }
>
> .nav-item.active {
>   background: #1d4ed8;
>   color: #ffffff;
> }
>
> .nav-icon {
>   width: 18px;
>   height: 18px;
>   flex-shrink: 0;
> }
>
> .sidebar-footer {
>   margin-top: auto;
>   padding: 12px;
>   border-top: 1px solid #1e293b;
>   display: flex;
>   align-items: center;
>   gap: 8px;
> }
>
> .app-body {
>   flex: 1;
>   display: flex;
>   flex-direction: column;
>   overflow: hidden;
>   min-width: 0;
> }
>
> .topbar {
>   background: #ffffff;
>   border-bottom: 1px solid #e2e8f0;
>   flex-shrink: 0;
> }
>
> .main-content {
>   flex: 1;
>   overflow-y: auto;
>   padding: 24px 32px;
>   background: #f8fafc;
> }
> ```
>
> Keep all other global styles (`.card`, `.badge`, `.table-container`, `.stats-grid`, `.stat-card`, `.page-header`, `.loading`, `.error`, etc.) exactly as they are.

---

## Phase 3: Verify FilterBar compatibility

After the App.vue change, read `client/src/components/FilterBar.vue`. Check:
- It must not have any styles that assume full viewport width from a top position
- If it has `position: sticky; top: 0` or assumes it is the topmost element, those assumptions are still valid — it now sits in `.topbar` which flows normally

No changes to FilterBar are expected unless it has explicit height/positioning that breaks in the new context. If it does, delegate the minimal fix to vue-expert.

---

## Phase 4: Verify ProfileMenu and LanguageSwitcher in sidebar footer

The sidebar footer is dark (`#0f172a` background). Check if `ProfileMenu.vue` or `LanguageSwitcher.vue` have hardcoded light-background colors (white backgrounds, dark borders) that will clash with the dark sidebar footer.

If either component has white/light backgrounds on its trigger button, delegate this fix to vue-expert: change the trigger button background to `transparent`, text color to `#94a3b8`, and hover state to `#1e293b`. Preserve dropdown/modal styling unchanged since dropdowns escape the sidebar via z-index.

---

## Phase 5: Spot-check one view for spacing regressions

Read `client/src/views/Dashboard.vue`. Check whether `.main-content` removing the old `max-width: 1600px; margin: 0 auto` causes layout issues. With the sidebar consuming 240px, the content area is `calc(100vw - 240px)` — the `max-width` constraint is now less critical but can be re-added if needed.

If the content looks too wide, delegate to vue-expert: add `max-width: 1360px; margin: 0 auto` inside `.main-content` (not on a per-view wrapper).

---

## Phase 6: Browser verification

Open `http://localhost:3000` and confirm:
1. Sidebar is visible on the left at 240px wide with dark navy background
2. All 7 nav links appear with icons, correct labels, and correct active highlight
3. Active link switches correctly when navigating between routes
4. FilterBar appears horizontally below the brand area (inside the white topbar strip)
5. Main content scrolls vertically without the sidebar scrolling
6. ProfileMenu and LanguageSwitcher are visible in the sidebar footer
7. No horizontal scrollbar on the page body
8. Modals (ProfileDetailsModal, TasksModal) still open correctly

If any of these fail, read the relevant component and delegate the minimal fix to vue-expert.

---

## Common Pitfalls

- **`.app` flex direction**: must be `row`, not `column` — the sidebar and content sit side by side
- **Overflow**: `.app` gets `overflow: hidden`; `.main-content` gets `overflow-y: auto` — if you forget this, the whole page won't scroll
- **`min-width: 0` on `.app-body`**: required — flex children don't shrink below their content width by default, causing overflow
- **Active class on router-link**: Vue Router auto-applies `.router-link-active` and `.router-link-exact-active`, but this app uses an explicit `:class="{ active: $route.path === '...' }"` pattern — keep it exactly that way
- **Removing max-width from main-content**: the old layout centered content with `max-width: 1600px; margin: 0 auto` — with a sidebar, the available width is already constrained, so this is optional but harmless to keep
