---
name: tailwind-v4
description: Use when writing or reviewing CSS classes, picking utility names, configuring @theme / @utility / @variant / @source / @reference / @apply directives, migrating v3→v4, setting up Tailwind in a new project, or about to emit inline `style=""` / `<style>` blocks in a Tailwind v4.1.x project. Triggers include `shadow-sm`, `bg-gradient-to-r`, `bg-opacity-*`, `flex-shrink-*`, `overflow-ellipsis`, `tailwind.config.js`, `@tailwind base/components/utilities`, `[16px]`/`[24px]`/`[100px]` arbitrary px values on `p-*`/`m-*`/`w-*`/`h-*`/`gap-*`, `grid-cols-[13]`/`z-[99]` style arbitrary integers, `mt-[-16px]` negative arbitraries, or any uncertainty about whether a v4 class name exists.
---

# Tailwind CSS v4.1.x Reference

Reference for **Tailwind CSS v4.1.x** (pinned v4.1.18). Make every styling decision produce idiomatic v4 output — no inline-style or v3 fallbacks.

- **Target**: v4.1.18 (final v4.1 release, 2025-12-11)
- **Range**: ^4.1.0 <4.2.0
- **Last verified**: 2026-05-24
- **Browser floor**: Safari 16.4+, Chrome 111+, Firefox 128+, Node 20+
- **Related packages**: `@tailwindcss/vite`, `@tailwindcss/postcss`, `@tailwindcss/cli`, `@tailwindcss/upgrade` (all `^4.1.0`)
- **Last changelog check**: https://github.com/tailwindlabs/tailwindcss/releases/tag/v4.1.18

## When to Use

Apply whenever you are about to:

- Write CSS classes or pick utility names in a Tailwind project
- Set up a new Tailwind project or migrate from v3
- Write/edit `@theme`, `@utility`, `@variant`, `@custom-variant`, `@source`, `@reference`, or `@apply` directives
- Emit `style=""` or `<style>` blocks (consult before falling back to inline CSS)
- Review or refactor existing Tailwind code

**Concrete symptoms — STOP and check here:**

- About to write `shadow-sm`, `rounded-sm`, `rounded` (no size), `blur`, `drop-shadow`
- About to write `bg-gradient-to-r`, `bg-opacity-*`, `text-opacity-*`, `border-opacity-*`, `ring-opacity-*`
- About to write `flex-shrink-*`, `flex-grow-*`, `overflow-ellipsis`, `decoration-slice`, `break-words`
- About to write `!bg-red-500` (v3 prefix `!` syntax)
- About to write `grid-cols-[1fr,2fr]` (commas in arbitrary values)
- About to write `p-[16px]`, `mt-[24px]`, `w-[100px]`, `gap-[12px]` (px arbitrary on spacing utilities — v4 accepts `p-4`, `mt-6`, `w-25`, `gap-3` directly)
- About to write `grid-cols-[13]`, `z-[99]`, `col-span-[16]`, `border-[3px]`, `rotate-[37deg]` (v4 dynamic utilities accept these as `grid-cols-13`, `z-99`, `col-span-16`, `border-3`, `rotate-37`)
- About to write `w-[16px] h-[16px]` (use `size-4`)
- About to write `bg-[var(--brand)]` (v4 short form is `bg-(--brand)`)
- About to write `mt-[-16px]` (negative goes on the class: `-mt-4`)
- About to create `tailwind.config.js`
- About to write `@tailwind base; @tailwind components; @tailwind utilities;`
- About to fall back to inline `style=""` because you're unsure if a v4 class exists

**When NOT to use:** Project is on v3 (look for `tailwind.config.js` with `content: [...]` array or `@tailwind base;`). Confirm with the user before mixing v4 patterns into a v3 project.

## Why this matters

Most LLM training data predates Tailwind v4's release. Three failure modes you must actively prevent:

1. **Outdated class names** — emitting `shadow-sm` when v4 means `shadow-xs`, `bg-gradient-to-r` when v4 means `bg-linear-to-r`, or `bg-opacity-50` when v4 wants `bg-black/50`.
2. **Outdated config patterns** — generating `tailwind.config.js` or `@tailwind base; @tailwind components; @tailwind utilities;` when v4 uses a single `@import "tailwindcss"` plus `@theme`.
3. **Inline-style fallbacks** — when uncertain about a class name, hedging with `style="..."` or a `<style>` block. This defeats the purpose of using Tailwind.

When in doubt about a v4 class name: **grep the existing codebase or search the v4 docs at <https://tailwindcss.com/docs>. Do not fall back to inline CSS.**

## Hard rules

These apply to every code output in any Tailwind v4 project:

1. **Never** emit `@tailwind base;`, `@tailwind components;`, or `@tailwind utilities;`. Use `@import "tailwindcss";`.
2. **Never** create a new `tailwind.config.js`. Configure via `@theme` in CSS. The `@config` directive exists only as a legacy escape hatch for existing v3 configs being migrated.
3. **Never** use the removed opacity utilities (`bg-opacity-*`, `text-opacity-*`, `border-opacity-*`, `divide-opacity-*`, `ring-opacity-*`, `placeholder-opacity-*`). Use the slash modifier: `bg-black/50`.
4. **Never** emit inline `style="..."` attributes for properties Tailwind covers. Acceptable uses of `style=""` are limited to:
   - Genuinely dynamic per-instance values that change at runtime (e.g. a progress bar's `width` driven by JS state).
   - Values bound to framework reactivity (`v-bind` in Vue, `style={{}}` in React with dynamic props).
   - CSS custom property assignments (`style="--color: red"`) that Tailwind utilities then read via `bg-(--color)`.
5. **Never** emit `<style>` blocks (including Vue `<style scoped>`) for properties Tailwind covers. Acceptable uses of `<style>` are limited to: keyframes, complex selectors Tailwind doesn't model well, third-party widget overrides, and global resets beyond Preflight.
6. **Never** construct class names with string interpolation (`bg-${color}-500`). Tailwind's scanner only sees complete class strings. Use a conditional that emits whole names, a lookup map, or `@source inline()` to safelist.
7. **Never** mix `!` important position styles. v4 puts `!` at the **end**: `bg-red-500!`, not `!bg-red-500`. The v3 prefix form is deprecated.
8. **Never** use comma-separated values in arbitrary values (v3 allowed `grid-cols-[1fr,2fr]`). v4 only accepts underscores: `grid-cols-[1fr_2fr]`.
9. **Never** rely on `hover:` triggering on touch devices. v4 wraps `hover:` in `@media (hover: hover)` — touch devices skip it. Override with `@custom-variant hover (&:hover);` if you need the v3 behavior.
10. **Never** put `_` inside CSS variable names in arbitrary values expecting underscore→space conversion. `var(--foo_bar)` keeps the underscore in v4 (unlike v3).
11. **Never** reach for `[...]` arbitrary values when a standard class fits. v4's `--spacing` scale accepts any positive number on `p/m/w/h/size/gap/inset/top..left/translate/space/scroll-{m,p}/leading/indent/basis<n>`; dynamic utilities (`grid-cols-*`, `z-*`, `col-span-*`, `border-*`, `rotate-*`, `opacity-*`, `duration-*`, `delay-*`, `scale-*`, `skew-*`, `outline-*`, `ring-*`, `columns-*`, `stroke-*`, `line-clamp-*`, `aspect-<a>/<b>`, etc.) accept any integer. `p-[16px]` → `p-4`, `grid-cols-[13]` → `grid-cols-13`, `z-[99]` → `z-99`. See **arbitrary-values.md** for the full conversion tables and decision flowchart. Arbitrary values are reserved for off-scale numerics (`[3px]`, `[7px]`), CSS functions (`calc`, `clamp`, `min`, `max`), and theme-namespace-only utilities that don't match a named key.
12. **Never** put the negative sign inside the bracket. `mt-[-16px]` → `-mt-4`. The negative prefix applies to the class, not the value.

## Top v3 → v4 renames (most common)

| v3 | v4 |
|---|---|
| `shadow` | `shadow-sm` |
| `shadow-sm` | `shadow-xs` |
| `rounded` | `rounded-sm` |
| `rounded-sm` | `rounded-xs` |
| `blur` / `blur-sm` | `blur-sm` / `blur-xs` |
| `drop-shadow` / `drop-shadow-sm` | `drop-shadow-sm` / `drop-shadow-xs` |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `bg-opacity-50` | `bg-black/50` |
| `text-opacity-50` | `text-black/50` |
| `flex-shrink-*` | `shrink-*` |
| `flex-grow-*` | `grow-*` |
| `overflow-ellipsis` | `text-ellipsis` |
| `outline-none` (a11y-preserving) | `outline-hidden` |
| `break-words` | `wrap-break-word` |
| `ring` (3px default) | `ring-3` (default ring is now 1px) |

> **`outline-none` semantic change**: in v4, `outline-none` actually sets `outline-style: none`. The old v3 behavior (transparent outline kept for forced-colors a11y) is now `outline-hidden`. If you're disabling focus rings for design but still want a11y, use `outline-hidden`.

For the full migration tables (removed utilities, deprecated in v4.1.0, config migration, dark mode, container, variant ordering, default behavior changes) see **migration.md**.

## Most-used utilities quick reference

**Layout**: `flex`, `grid`, `block`, `inline-block`, `hidden`, `contents`
**Flex**: `flex-row`, `flex-col`, `flex-wrap`, `items-center`, `items-baseline-last`, `justify-between`, `justify-center-safe`, `gap-4`, `shrink-0`, `grow`
**Grid**: `grid-cols-3`, `grid-rows-2`, `col-span-2`, `auto-cols-fr`, `place-items-center`
**Container queries**: `@container`, `@sm:flex`, `@md:grid`, `@lg:gap-4`
**Spacing**: `p-4`, `px-2`, `mt-8`, `gap-4`, `space-y-2`, `ps-4`, `pe-2`
**Sizing**: `w-full`, `h-screen`, `h-lh`, `size-12`, `max-w-prose`, `min-h-dvh`
**Typography**: `text-base`, `font-medium`, `tracking-tight`, `leading-snug`, `text-balance`, `text-pretty`, `text-shadow-sm`, `wrap-anywhere`
**Colors**: `bg-slate-900`, `text-white`, `border-zinc-200`, `bg-blue-500/50`
**Borders**: `border`, `border-2`, `border-t`, `border-zinc-200`, `rounded-lg`, `rounded-xs`, `inset-ring`
**Effects**: `shadow-sm`, `shadow-xs`, `shadow-lg/30`, `ring-2`, `ring-blue-500/50`, `blur-sm`, `backdrop-blur-sm`, `mask-t-from-50%`
**Transitions**: `transition`, `transition-colors`, `duration-200`, `ease-out`, `starting:opacity-0`
**Variants**: `hover:`, `focus:`, `active:`, `disabled:`, `dark:`, `md:`, `lg:`, `@md:`, `not-hover:`, `has-[input]:`, `peer-checked:`, `group-hover:`, `aria-expanded:`, `data-[state=open]:`, `starting:`, `motion-safe:`, `motion-reduce:`, `pointer-fine:`, `pointer-coarse:`, `user-valid:`, `user-invalid:`, `details-content:`, `inverted-colors:`, `noscript:`, `forced-colors:`, `*:`, `**:`

## Output rules for code generation

When generating any Tailwind v4 code:

- Default to utility classes on the element. No `style=""` for properties expressible as utilities.
- Use v4 names. If unsure whether a name changed, prefer the v4 form (`shadow-xs`, `rounded-xs`, `shrink-0`, `text-ellipsis`, `bg-linear-to-r`).
- Prefer `size-*` over `w-* h-*` when width and height match.
- Use slash opacity (`bg-black/50`), never `bg-opacity-*`.
- Put `!` at the end (`bg-red-500!`).
- Stack variants left-to-right (`dark:hover:bg-black`).
- Use logical properties (`ps-*`, `pe-*`, `ms-*`, `me-*`) when the design supports RTL.
- Use `outline-hidden` to suppress focus outlines while preserving forced-colors accessibility; `outline-none` actually removes the outline.
- For dynamic values, use whole-name conditionals or CSS custom properties bound via `v-bind()` / inline `style=""` assignments — never string-interpolated class names.
- For forms with validation, use `user-valid:` / `user-invalid:` instead of `valid:` / `invalid:` to avoid showing errors before the user touches the field.
- For overflow-prone centering, prefer safe-alignment (`justify-center-safe`) where content could overflow.

When the user hasn't specified v3 vs v4, assume v4. If you see `tailwind.config.js` or `@tailwind base;` in the codebase, that's v3 — confirm before mixing v4 patterns.

## Common anti-patterns

**Inline style for static values** — use utilities:
```html
<!-- ❌ --> <div style="margin-top: 16px; background: #3b82f6;">
<!-- ✅ --> <div class="mt-4 bg-blue-500">
```

**v3 opacity utilities** — use slash modifier:
```html
<!-- ❌ --> <div class="bg-black bg-opacity-50 text-white text-opacity-80">
<!-- ✅ --> <div class="bg-black/50 text-white/80">
```

**v3 renamed utilities** — use v4 names:
```html
<!-- ❌ --> <div class="shadow-sm rounded-sm flex-shrink-0 overflow-ellipsis bg-gradient-to-r">
<!-- ✅ --> <div class="shadow-xs rounded-xs shrink-0 text-ellipsis bg-linear-to-r">
```

**Arbitrary values where standards exist** — use dynamic utilities / `--spacing` scale:
```html
<!-- ❌ --> <div class="p-[16px] mt-[24px] w-[100px] gap-[12px] grid-cols-[13] z-[99] border-[3px] rotate-[37deg] w-[16px] h-[16px] bg-[var(--brand)]">
<!-- ✅ --> <div class="p-4 mt-6 w-25 gap-3 grid-cols-13 z-99 border-3 rotate-37 size-4 bg-(--brand)">
```

**Negative as bracketed value** — prefix the class instead:
```html
<!-- ❌ --> <div class="mt-[-16px] translate-x-[-8px] rotate-[-15deg]">
<!-- ✅ --> <div class="-mt-4 -translate-x-2 -rotate-15">
```

**Dynamic class construction** — scanner can't see interpolated names:
```vue
<!-- ❌ --> <div :class="`bg-${color}-500`">
<!-- ✅ --> <div :class="color === 'red' ? 'bg-red-500' : 'bg-blue-500'">
```

**Important marker in wrong position**:
```html
<!-- ❌ --> <div class="!bg-red-500 hover:!opacity-100">
<!-- ✅ --> <div class="bg-red-500! hover:opacity-100!">
```

**`@layer components` for apply-able classes** — use `@utility`:
```css
/* ❌ */ @layer components { .btn { @apply px-4 py-2 rounded; } }
/* ✅ */ @utility btn { padding-inline: 1rem; padding-block: 0.5rem; border-radius: 0.5rem; }
```

For full anti-pattern catalog and Vue-specific `<style scoped>` patterns, see **migration.md** and **integration.md**.

## Where to find more detail

When the answer isn't in this main file, consult these support files:

- **directives.md** — Full reference for `@import`, `@theme` (variants, namespaces, override), `@utility` (static + functional, `--value()` / `--modifier()` modes), `@variant`, `@custom-variant`, `@source` (incl. `inline`, `not`), `@apply` (with caveats), `@reference`, `@config` / `@plugin` (legacy)
- **migration.md** — Full v3→v4 cheatsheet (renamed/removed/deprecated tables, setup/config/dark-mode migration, variant ordering, subtle changes, default behavior changes, version coverage matrix, post-v4.1 features in v4.2+/v4.3+, maintenance workflow)
- **arbitrary-values.md** — `[...]` arbitrary value → standard class conversion tables (spacing, sizing, positioning, typography, border, effects, transform, transition, grid/flex), dynamic-utility integer ranges, when arbitrary values are still required (theme-namespace, off-scale, CSS functions), negative/decimal handling, decision flowchart
- **features.md** — v4.0/v4.1.x feature catalog (text-shadow, mask, wrap, baseline-last, safe alignment, pointer variants, accessibility/form variants, details-content, h-lh, transition-includes-display, container queries, 3D transforms, size-*, field-sizing, gradients, value syntax, source detection)
- **integration.md** — Setup snippets (Vite, Nuxt 4/3, PostCSS, class prefix) and Vue/Nuxt patterns (`<style scoped>` with `@reference`, reactive values with `v-bind()`, class composition, shadcn/ui on v4)

When this skill doesn't cover the exact utility or directive, **search the v4 docs at <https://tailwindcss.com/docs>** before guessing or falling back to inline CSS.
