# CLAUDE.md - Client

This file provides guidance to Claude Code (claude.ai/code) when working with the Vue 3 frontend.

## Running the Client

```bash
npm run dev          # http://localhost:3000
npm run build        # Production build
```

## Component Style

All components use Vue 3 Composition API with `setup()` — no Options API. Structure: `<template>` → `<script>` → `<style scoped>`.

## Singleton Filter State

`composables/useFilters.js` declares refs at **module level** (outside the function), so all components share the same filter state. This is intentional — don't move refs inside the function.

`getCurrentFilters()` maps UI names to API params: `selectedLocation` → `warehouse`, `selectedPeriod` → `month`.

## i18n

`composables/useI18n.js` supports English and Japanese. Locale stored in localStorage. Currency maps: `en` → USD, `ja` → JPY. Translation files in `src/locales/`. `translateProductName()`, `translateCustomerName()`, and `translateWarehouse()` handle domain-specific translations.

## API Client

`api.js` uses Axios with base URL `http://localhost:8001/api`. Filter params are built with `URLSearchParams`, skipping any value set to `'all'`.

## Key Conventions

- `ref()` for mutable state, `computed()` for derived data — never store derived data in refs
- Always use unique domain keys for `v-for` (`item.id`, `item.sku`) — never array index
- Validate dates before calling `.getMonth()` — check `!isNaN(date.getTime())`
- Data loading pattern: `loading` ref + `error` ref + try/catch/finally
- All styles scoped to components; global styles live in `App.vue`
