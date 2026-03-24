---
name: saas-redesign
description: Redesign the Vue 3 application UI from a top nav bar layout into a modern SaaS-style interface with a vertical sidebar, consistent spacing, and a polished professional look. Use this skill when redesigning the app layout or converting to a sidebar navigation pattern.
---

# SaaS UI Redesign Skill

Redesign this Vue 3 application from its current horizontal top-nav layout into a modern SaaS-style interface with a persistent vertical sidebar on the left.

## MANDATORY: Use the vue-expert subagent

All `.vue` file creation and significant modifications MUST be delegated to the `vue-expert` subagent. Pass it the full design specification for each file it needs to modify.

## Target Layout

```
+----------+--------------------------------------------------+
|          |  Top Bar (breadcrumb, search, profile, lang)      |
|  Sidebar +--------------------------------------------------+
|  (fixed) |                                                    |
|          |  Filter Bar (contextual, only when relevant)       |
|  Logo    +--------------------------------------------------+
|  Nav     |                                                    |
|  Links   |  Main Content Area                                 |
|          |  (page-level views with consistent padding)        |
|          |                                                    |
|          |                                                    |
|  Spacer  |                                                    |
|          |                                                    |
|  User    |                                                    |
|  Menu    |                                                    |
+----------+--------------------------------------------------+
```

## Design Tokens

Use these exact values for consistency across all components:

### Colors
```
--sidebar-bg:         #0f172a     (slate-900)
--sidebar-text:       #94a3b8     (slate-400)
--sidebar-text-hover: #e2e8f0     (slate-200)
--sidebar-active-bg:  #1e293b     (slate-800)
--sidebar-active-text:#ffffff
--sidebar-accent:     #3b82f6     (blue-500)

--page-bg:            #f1f5f9     (slate-100)
--content-bg:         #ffffff
--border:             #e2e8f0     (slate-200)
--border-light:       #f1f5f9     (slate-100)

--text-primary:       #0f172a     (slate-900)
--text-secondary:     #64748b     (slate-500)
--text-muted:         #94a3b8     (slate-400)

--success:            #10b981     (emerald-500)
--warning:            #f59e0b     (amber-500)
--danger:             #ef4444     (red-500)
--info:               #3b82f6     (blue-500)
```

### Spacing
```
--sidebar-width:      260px
--sidebar-collapsed:  72px       (icon-only mode, optional)
--topbar-height:      60px
--content-padding:    1.5rem
--card-padding:       1.25rem
--card-radius:        12px
--card-shadow:        0 1px 3px rgba(0,0,0,0.04), 0 1px 2px rgba(0,0,0,0.06)
```

### Typography
```
--font-family:        'Inter', -apple-system, BlinkMacSystemFont, sans-serif
--font-size-xs:       0.75rem
--font-size-sm:       0.813rem
--font-size-base:     0.875rem
--font-size-lg:       1rem
--font-size-xl:       1.25rem
--font-size-2xl:      1.5rem
--font-size-3xl:      1.875rem
```

## Files to Modify

### 1. `client/src/App.vue` (PRIMARY — layout restructure)

This is the most critical file. Transform the layout from:
- Current: `.app` > `.top-nav` + `FilterBar` + `.main-content`
- Target:  `.app-layout` > `.sidebar` + `.main-panel` > `.topbar` + `.page-content`

**Sidebar requirements:**
- Fixed position, full viewport height, `var(--sidebar-width)` wide
- Dark background (`--sidebar-bg`)
- Logo section at top (company name + subtitle, white text)
- Nav links as vertical list with SVG icons (24x24) to the left of each label
- Active link has `--sidebar-active-bg` background, `--sidebar-accent` left border (3px), white text
- Hover state: `--sidebar-text-hover` color, subtle background shift
- Bottom section: user avatar + name + role, separated by a border-top
- LanguageSwitcher placed in the sidebar bottom section or the topbar

**Icons for each nav item** (use inline SVG, no icon library):
- Overview: grid/dashboard icon (4 squares)
- Inventory: box/package icon
- Orders: clipboard/list icon
- Finance: dollar/chart icon
- Demand Forecast: trending-up icon
- Reports: bar-chart icon
- Restocking: refresh/rotate icon

**Topbar requirements:**
- White background, subtle bottom border
- Left side: page title (dynamic based on current route) or breadcrumb
- Right side: FilterBar component (inline), profile menu button
- Height: `var(--topbar-height)`

**Main content area:**
- `margin-left: var(--sidebar-width)` to offset for sidebar
- `padding: var(--content-padding)`
- Background: `var(--page-bg)`
- Remove the existing `max-width: 1600px` constraint — let content fill the available space

**Global style updates (unscoped `<style>` in App.vue):**
- Update `.card` to use `var(--card-radius)`, `var(--card-shadow)`, `var(--card-padding)`
- Update `.stats-grid` gap to `1rem`
- Update `.stat-card` to have slightly more vertical padding
- Update table styles: tighter row height, `--font-size-sm` for cells
- Ensure `.badge` styles remain unchanged (they already work well)
- Keep all existing badge variants (success, warning, danger, info, increasing, stable, decreasing, high, medium, low)

### 2. `client/src/components/FilterBar.vue`

Adapt to work inside the topbar:
- Horizontal layout with smaller select inputs
- Remove any outer padding/margin that assumed it was a standalone bar
- Compact styling: smaller font size, tighter gaps
- Reset button as an icon-only button (no text)

### 3. `client/src/components/ProfileMenu.vue`

Adapt for topbar placement:
- Smaller avatar in topbar (32px)
- Larger avatar in sidebar bottom section if used there
- Dropdown menu direction should work from the topbar (drop down-left)

### 4. View files (`client/src/views/*.vue`)

Each view should:
- Remove any redundant `.page-header` top margin (the topbar now provides the page title)
- Ensure cards use the global `.card` class for consistent radius and shadow
- Verify `.stats-grid` uses `grid-template-columns: repeat(auto-fit, minmax(240px, 1fr))` for better density
- No layout changes to internal content — only spacing adjustments

## Implementation Steps

1. **Start with App.vue** — restructure the template and global styles. This is the foundation.
2. **Update FilterBar.vue** — make it compact and topbar-friendly.
3. **Update ProfileMenu.vue** — adapt for new layout context.
4. **Sweep view files** — adjust spacing if any views had hardcoded margins that conflict.
5. **Test in browser** — use Playwright MCP tools to verify:
   - Sidebar renders correctly with all nav items
   - Active link highlighting works on navigation
   - FilterBar is usable in the topbar
   - Content area scrolls independently of sidebar
   - All existing functionality (filters, modals, tables) still works
   - Japanese locale still works (sidebar text should translate)

## Constraints

- Do NOT change any backend code
- Do NOT change any composables, API client, or router configuration
- Do NOT change any business logic in view components
- Do NOT remove any existing functionality (filters, modals, i18n, etc.)
- Do NOT add any new npm dependencies — use only inline SVG for icons
- Keep all existing i18n keys working — add new keys only if needed for new UI elements
- The sidebar must use `router-link` with the same paths as the current nav
- Preserve all existing CSS class names used by view components (`.card`, `.stats-grid`, `.stat-card`, `.badge`, `.table-container`, etc.) — views depend on these global styles
