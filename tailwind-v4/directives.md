# Tailwind v4 Directive Reference

Full reference for `@import`, `@theme`, `@utility`, `@variant`, `@custom-variant`, `@source`, `@apply`, `@reference`, `@config`, `@plugin`.

## `@import "tailwindcss"`

The single entry point. Replaces v3's three `@tailwind` directives. Pulls in Preflight, theme defaults, and utilities.

Sub-imports for fine control (rare, but useful for stripping Preflight):

```css
@layer theme, base, components, utilities;
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);
```

Omit `preflight.css` to disable Preflight.

Class prefix (replaces v3's `prefix:` config):

```css
@import "tailwindcss" prefix(tw);
```

Generates `tw:flex tw:items-center` etc.

## `@theme`

Defines design tokens. Each variable simultaneously becomes a CSS custom property at runtime and generates utility classes at build time. Must be top-level (cannot live inside `@media`).

```css
@theme {
  --color-brand: oklch(0.7 0.15 250);
  --color-brand-fg: #fff;
  --font-display: "Inter", sans-serif;
  --text-hero: 4.5rem;
  --spacing-18: 4.5rem;
  --radius-card: 0.875rem;
  --shadow-card: 0 8px 24px -8px rgb(0 0 0 / 0.2);
  --breakpoint-3xl: 120rem;
  --animate-fade-in: fade-in 0.5s ease-out;

  @keyframes fade-in {
    from { opacity: 0; }
    to { opacity: 1; }
  }
}
```

Generates: `bg-brand`, `text-brand-fg`, `font-display`, `text-hero`, `p-18`, `rounded-card`, `shadow-card`, `3xl:flex`, `animate-fade-in`.

### `@theme` variants

| Variant | Behavior |
|---|---|
| `@theme { ... }` | Default. Emits CSS variables at runtime AND generates utilities. |
| `@theme inline { ... }` | Generates utilities, but **inlines** the value into each class instead of referencing a `var()`. Use when consumers (e.g. shadcn/ui) define their own `var()` indirection. |
| `@theme reference { ... }` | Does **not** emit CSS variables or utilities. Used as a placeholder/lookup only — values exist for `theme(...)` and `@apply` resolution but never reach the output CSS. |
| `@theme static { ... }` | Always emits the CSS variable even if no utility using it appears in source. Use for runtime JS access. |

### Namespaces (left side controls which utilities are generated)

| Prefix | Generates utilities for |
|---|---|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*`, `stroke-*`, etc. |
| `--font-*` | `font-*` (family) |
| `--text-*` | `text-*` (size) |
| `--font-weight-*` | `font-*` (weight) |
| `--tracking-*` | `tracking-*` |
| `--leading-*` | `leading-*` |
| `--spacing-*` | `p-*`, `m-*`, `w-*`, `h-*`, `gap-*`, etc. (also drives `--spacing` arithmetic) |
| `--breakpoint-*` | responsive prefixes (`sm:`, `md:`, custom) |
| `--container-*` | `@container` query breakpoints (`@sm:`, etc.) |
| `--radius-*` | `rounded-*` |
| `--shadow-*` | `shadow-*` |
| `--inset-shadow-*` | `inset-shadow-*` |
| `--drop-shadow-*` | `drop-shadow-*` |
| `--text-shadow-*` | `text-shadow-*` *(v4.1.0+)* |
| `--blur-*` | `blur-*` |
| `--perspective-*` | `perspective-*` |
| `--aspect-*` | `aspect-*` |
| `--ease-*` | `ease-*` |
| `--animate-*` | `animate-*` |

### Override / reset

```css
@theme {
  --color-*: initial;             /* clear all default colors */
  --color-primary: #3b82f6;       /* then redefine */
}
```

To wipe the entire default theme: `--*: initial;`.

## `@utility`

Define a custom utility that integrates with variants (`hover:`, `md:`, etc.).

### Static utility (name without `*`)

```css
@utility content-auto {
  content-visibility: auto;
}
```

### Functional utility (name ending in `-*`)

```css
@utility tab-* {
  tab-size: --value(integer);
}
```

The functional form receives the class suffix via `--value(...)` (for the part before any `/`) and `--modifier(...)` (for the part after `/`).

### `--value()` / `--modifier()` argument modes

| Mode | Accepts | Example |
|---|---|---|
| `--<namespace>-*` | Values registered in a `@theme` namespace | `--value(--color-*)`, `--value(--spacing-*)`, `--value(--text-*)` |
| `number` | Bare decimal numbers | `--value(number)` |
| `integer` | Bare integers | `--value(integer)` |
| `ratio` | Fractions, treated as a value+modifier unit | `--value(ratio)` |
| `percentage` | Bare `%` values | `--value(percentage)` |
| `[color]` | Arbitrary value `[...]` validated as a color | `--value([color])` |
| `[length]` | Arbitrary value validated as a length | `--value([length])` |
| `[*]` | Arbitrary value, no type check | `--value([*])` |

**Important**: `length`, `color`, `image`, `url`, `position`, `family-name`, `generic-name` are **only valid inside square brackets** as arbitrary-value types. They are **not** valid as bare-value types — `--value(length)` will throw a warning (added in v4.1.3). Use `--value([length])` or `--value(--spacing-*)` instead.

### Combining modes (OR)

```css
@utility tab-* {
  /* Resolve from theme key OR accept arbitrary integer */
  tab-size: --value(--tab-size-*, integer, [integer]);
}

@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], [*]);
}

@utility aspect-* {
  /* ratio treats both sides of `/` as one value */
  aspect-ratio: --value(--aspect-ratio-*, ratio, [ratio]);
}
```

## `@variant`

Apply a Tailwind variant to a rule inside your CSS:

```css
.btn {
  background: white;

  @variant dark {
    background: black;
  }

  @variant hover {
    opacity: 0.9;
  }
}
```

## `@custom-variant`

Define your own variant. Equivalent to a v3 JS plugin variant but expressed in CSS.

```css
/* Shorthand form */
@custom-variant theme-midnight (&:where([data-theme="midnight"] *));

/* Multi-rule form */
@custom-variant any-hover {
  @media (any-hover: hover) {
    &:hover { @slot; }
  }
}
```

The most common one — class-based dark mode override:

```css
@custom-variant dark (&:where(.dark, .dark *));
```

This replaces v3's `darkMode: 'class'` config.

## `@source`

Adjust source detection. v4 auto-scans the project (respecting `.gitignore` and excluding `node_modules` by default), so `@source` is for edge cases.

```css
@source "../../packages/ui/src/**/*.{vue,ts}";   /* Add a path */
@source not "./legacy";                          /* Exclude a path (v4.1.0+) */
@source inline("bg-red-500 text-white p-4");     /* Safelist literal class names (v4.1.0+) */
@source not inline("debug-grid");                /* Force-exclude a class (v4.1.0+) */

/* Brace expansion (v4.1.0+) */
@source inline("{hover:,}bg-red-{50,{100..900..100},950}");
```

For dynamic class names you cannot refactor, `@source inline(...)` is the supported escape hatch (replaces v3's `safelist`).

When pulling classes from `node_modules` (e.g. shadcn/ui, Headless UI), explicitly add the source:

```css
@source "../node_modules/@shadcn/ui";
```

## `@apply`

Still works, same semantics as v3, but with two critical caveats:

**Caveat 1**: In scoped style blocks (Vue SFC `<style scoped>`, CSS Modules, Astro scoped, etc.) the scoped block doesn't see your `@theme` tokens unless you tell it where to find them.

```vue
<style scoped>
@reference "@/assets/css/main.css";

.card {
  @apply bg-brand text-brand-fg rounded-card;
}
</style>
```

`@reference` imports the theme **without** re-emitting Tailwind's CSS. **`@reference "tailwindcss"` only gives you the default theme — for custom tokens, point it at your own CSS entry file.**

**Caveat 2**: `@apply` only resolves utilities and `@utility` definitions. **Custom classes wrapped in `@layer components` are no longer applyable**. Migrate them to `@utility`:

```css
/* ❌ Doesn't work in v4 */
@layer components {
  .btn-primary { background: blue; }
}
.toolbar { @apply btn-primary; }

/* ✅ Works in v4 */
@utility btn-primary {
  background: blue;
}
.toolbar { @apply btn-primary; }
```

## `@reference`

See above. Required in any CSS context that's compiled separately from your main entry (Vue `<style scoped>`, CSS Modules, Astro scoped, etc.) when you want `@apply`, `@variant`, or `theme(...)` to resolve custom tokens.

## `@config` (legacy)

Loads a v3-style `tailwind.config.js`. Only use this when migrating an existing project incrementally.

```css
@config "../tailwind.config.js";
```

Not all v3 config options carry over — `corePlugins`, `safelist`, and `separator` are unsupported. For safelisting, use `@source inline()`.

## `@plugin` (legacy)

Loads a v3-style JS plugin.

```css
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
```
