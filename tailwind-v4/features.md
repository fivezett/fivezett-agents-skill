# Tailwind v4 Feature Catalog

Catalog of v4.0 baseline + v4.1.x feature additions. Many are missed in v3-era code.

## v4.1.0 — text-shadow utilities

```html
<p class="text-shadow-2xs">…</p>
<p class="text-shadow-xs">…</p>
<p class="text-shadow-sm">…</p>
<p class="text-shadow-md">…</p>
<p class="text-shadow-lg">…</p>

<!-- Colored text shadow with opacity modifier -->
<p class="text-shadow-sky-300/40">…</p>

<!-- Configurable via theme -->
<!-- @theme { --text-shadow-glow: 0 0 8px var(--color-cyan-400); } -->
<p class="text-shadow-glow">…</p>
```

## v4.1.0 — mask utilities (compositional)

```html
<!-- Linear gradient masks (directional) -->
<div class="mask-t-from-50%">…</div>
<div class="mask-b-from-20% mask-b-to-80%">…</div>
<div class="mask-l-from-0% mask-r-from-100%">…</div>
<div class="mask-x-from-90%">…</div>

<!-- Radial masks -->
<div class="mask-radial-from-80%">…</div>
<div class="mask-radial-at-right">…</div>

<!-- Conic masks -->
<div class="mask-conic-from-50%">…</div>
```

Masks compose: stack `mask-t-*` with `mask-radial-*` etc.

## v4.1.0 — colored drop-shadow

```html
<img class="drop-shadow-xl drop-shadow-cyan-500/50">
<img class="drop-shadow-lg drop-shadow-indigo-500/50">
```

## v4.1.0 — wrap utilities

```html
<p class="wrap-anywhere">long-unbreakable-strings-go-here</p>
<p class="wrap-break-word">…</p>
<p class="wrap-normal">…</p>
```

## v4.1.0 — baseline-last alignment

```html
<div class="flex items-baseline-last">…</div>
<div class="grid grid-cols-3"><span class="self-baseline-last">…</span></div>
```

Aligns to the baseline of the **last** line — useful for "label + multi-line content" layouts.

## v4.1.0 — safe alignment

```html
<!-- Falls back to start-alignment when content overflows -->
<div class="flex justify-center-safe">…</div>
<div class="grid items-center-safe">…</div>
<div class="content-center-safe place-content-center-safe">…</div>
<div class="self-center-safe">…</div>
```

Prevents content from being cut off at both ends in overflow situations.

## v4.1.0 — pointer / any-pointer variants

```html
<!-- Primary pointing device precision -->
<button class="pointer-fine:px-2 pointer-coarse:px-4">…</button>
<button class="pointer-none:hidden">…</button>

<!-- Any connected pointing device -->
<div class="any-pointer-coarse:text-lg">…</div>
```

`pointer-*` reflects the primary input, `any-pointer-*` reflects all connected inputs.

## v4.1.0 — accessibility / form variants

```html
<!-- OS-level inverted color scheme -->
<div class="inverted-colors:bg-white inverted-colors:shadow-none">…</div>

<!-- JavaScript disabled -->
<div class="noscript:block hidden">JS required</div>

<!-- Form validity AFTER user interaction (no scary red on first paint) -->
<input class="user-invalid:border-red-500 user-valid:border-green-500">
```

Prefer `user-valid:` / `user-invalid:` over `valid:` / `invalid:` to avoid showing errors before the user touches the field.

## v4.1.0 — `<details>` content targeting

```html
<details>
  <summary>Header</summary>
  <!-- Targets the slotted content container, not the <details> root -->
  <p class="details-content:p-4 details-content:bg-zinc-50">Content here</p>
</details>
```

## v4.1.0 — `@source not` and `@source inline()`

```css
@source not "./legacy";
@source inline("bg-red-500 hover:bg-red-700");
@source inline("{hover:,focus:,}text-{red,blue,green}-{500,600,700}");
@source not inline("debug-grid debug-outline");
```

## v4.1.0 — bg-position / bg-size arbitrary shorthand

```html
<div class="bg-position-[center_top] bg-size-[200%_auto]">…</div>
```

## v4.1.0 — shadow opacity modifier

```html
<div class="shadow-lg/30">…</div>
<div class="inset-shadow-md/50">…</div>
<div class="drop-shadow-xl/40">…</div>
<div class="text-shadow-md/60">…</div>
```

## v4.1.5 — line-height height units

```html
<div class="h-lh">…</div>      <!-- height: 1lh -->
<div class="min-h-lh">…</div>
<div class="max-h-lh">…</div>
```

## v4.1.5 — transition includes display/visibility/etc

`transition` and `transition-all` now automatically transition `display`, `visibility`, `content-visibility`, `overlay`, and `pointer-events`. This dramatically simplifies `@starting-style` usage:

```html
<dialog
  class="opacity-0 starting:open:opacity-0 open:opacity-100
         transition duration-300"
>…</dialog>
```

No more need to manually list every property — the discrete transition just works.

> Note: v4.1.13 corrected this — `transition` no longer transitions `visibility`.

## v4.0 baseline features (often missed in v3-era code)

```html
<!-- Container queries (built-in) -->
<div class="@container">
  <div class="@sm:flex @md:grid @md:grid-cols-2">…</div>
</div>

<!-- not-* variant -->
<div class="not-hover:opacity-50 not-first:mt-4 not-supports-grid:flex">

<!-- @starting-style for enter/exit transitions -->
<dialog class="opacity-0 starting:open:opacity-0 open:opacity-100 transition">

<!-- 3D transforms -->
<div class="transform-3d perspective-distant rotate-x-12 rotate-y-45 translate-z-12">

<!-- size-* shorthand -->
<div class="size-12">  <!-- equivalent to w-12 h-12 -->

<!-- field-sizing -->
<textarea class="field-sizing-content"></textarea>

<!-- Logical properties (RTL-aware) -->
<div class="ps-4 pe-2 ms-auto start-0 end-4 border-s">

<!-- Conic & radial gradients with interpolation -->
<div class="bg-conic from-red-500 via-purple-500 to-blue-500 bg-conic-180">
<div class="bg-radial from-amber-200 to-amber-600 bg-radial-[at_top_left]">
<div class="bg-linear-to-r/oklch from-red-500 to-blue-500">

<!-- inert variant -->
<div class="inert:opacity-50 inert:pointer-events-none">

<!-- color-scheme -->
<div class="scheme-light dark:scheme-dark">

<!-- forced-colors a11y -->
<button class="forced-colors:border forced-color-adjust-none">

<!-- Direct children / all descendants -->
<ul class="*:py-2 **:text-sm">

<!-- font-stretch -->
<span class="font-stretch-condensed">

<!-- inset-ring (inside-only ring) -->
<button class="inset-ring inset-ring-zinc-200">

<!-- text-wrap -->
<h1 class="text-balance">…</h1>
<p class="text-pretty">…</p>
```

## Value syntax

### Slash opacity modifier

```html
<div class="bg-blue-500/50 text-black/80 border-red-500/25 shadow-lg/30">
```

Works on any utility that takes a color, and (v4.1.0+) on `shadow-*`, `inset-shadow-*`, `drop-shadow-*`, `text-shadow-*`.

### Arbitrary values

```html
<div class="w-[42rem] grid-cols-[1fr_auto_1fr] bg-[#ff0080]">
```

**Underscore = space**. Commas are not supported. CSS variable references inside arbitrary values keep their underscores literal (`var(--my_var)` stays as `my_var`).

### CSS custom property shorthand

```html
<!-- v4 shorthand -->
<div class="bg-(--brand) p-(--space-md)">

<!-- Equivalent verbose form (also works) -->
<div class="bg-[var(--brand)] p-[var(--space-md)]">
```

### Modifier on arbitrary values

```html
<div class="bg-[#ff0080]/50 text-(--brand)/75">
```

## Source detection

- v4 scans the project tree automatically. **There is no `content: [...]` array.**
- It respects `.gitignore`.
- `node_modules` is **excluded by default** (changed in v4.1.0). To pull classes from a package, add `@source "../node_modules/<pkg>"`.
- Binary files, lockfiles, and obvious non-source directories are skipped.
- Only **complete class name strings** that appear in scanned files are emitted. String concatenation, template literals, and computed names are invisible.

### Handling dynamic class names

```html
<!-- BAD — scanner won't see these -->
<div :class="`bg-${color}-500`">
<div :class="'text-' + size">

<!-- GOOD — full class names, branched -->
<div :class="color === 'red' ? 'bg-red-500' : 'bg-blue-500'">

<!-- GOOD — lookup map (the strings appear literally in source) -->
<script setup>
const classes = {
  red: 'bg-red-500 text-white',
  blue: 'bg-blue-500 text-white',
}
</script>
<div :class="classes[color]">

<!-- GOOD — last resort: safelist via @source inline() -->
<!-- in main.css: @source inline("bg-{red,blue,green}-{400,500,600}"); -->
```

### Debugging source detection

Set `DEBUG=*` to log which files were scanned and which directives ran:

```bash
DEBUG=* npm run dev
```

(Added in v4.1.6.)
