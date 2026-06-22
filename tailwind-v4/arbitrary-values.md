# Arbitrary Values → Standard Classes (v4)

Reference for converting `[...]` arbitrary values to v4 standard classes, plus the dynamic-utility ranges that make `grid-cols-13`, `z-99`, `border-3`, etc. valid without brackets. Prefer standard classes whenever conversion is possible — arbitrary values defeat the dev-experience benefits of the scale and bloat the JIT output.

## TL;DR

- **`--spacing` (default `0.25rem` = 4px) drives every size utility** (`p-*`, `m-*`, `w-*`, `h-*`, `size-*`, `gap-*`, `inset-*`, `top/right/bottom/left-*`, `translate-*`, `space-*`, `scroll-{m,p}-*`, `leading-<number>`, `indent-*`, `basis-<number>`). They all accept any positive number (decimals included) directly. Formula: **`px ÷ 4 = class number`** at the default root font size.
  - `p-[16px]` → `p-4`, `mt-[24px]` → `mt-6`, `w-[100px]` → `w-25`, `gap-[10px]` → `gap-2.5`.
- **Dynamic utilities accept any integer without brackets**: `grid-cols-*`, `grid-rows-*`, `col-{span,start,end}-*`, `row-{span,start,end}-*`, `z-*`, `order-*`, `opacity-*`, `duration-*`, `delay-*`, `rotate-*` (incl. `-x/-y/-z`), `scale-*` (incl. `-z`), `skew-*`, `border-*` (width), `outline-*` (width, now also auto-applies `outline-style: solid`), `ring-*`, `inset-ring-*`, `columns-*`, `stroke-*`, `line-clamp-*`, `grow-*`, `shrink-*`, `flex-<n>`, `aspect-<a>/<b>`.
- **Still need `[...]` for**: theme-namespace-only utilities (`rounded-*`, `text-*` font-size, `shadow-*`, `blur-*`, `tracking-*`, `ease-*`, `aspect-<name>`, `perspective-*`, `animate-*`, `font-*`), off-scale spacing values (`[3px]`, `[5px]`, `[7px]`, ...), and CSS functions / expressions (`calc()`, `clamp()`, `min()`, `max()`, `repeat()`, color literals).

## The `--spacing` scale

```css
@theme { --spacing: 0.25rem; /* = 4px */ }
```

Each spacing utility emits `calc(var(--spacing) * <n>)`. **Caveat**: if a project overrides `--spacing` (e.g. `--spacing: 1px`), the `px ÷ 4` formula no longer holds — recompute against the project's value.

### px ↔ class number lookup (default `--spacing: 0.25rem`)

| px | class | px | class | px | class |
|---|---|---|---|---|---|
| 0 | `0` | 24 | `6` | 64 | `16` |
| 2 | `0.5` | 28 | `7` | 80 | `20` |
| 4 | `1` | 32 | `8` | 96 | `24` |
| 6 | `1.5` | 36 | `9` | 100 | `25` |
| 8 | `2` | 40 | `10` | 112 | `28` |
| 10 | `2.5` | 44 | `11` | 128 | `32` |
| 12 | `3` | 48 | `12` | 160 | `40` |
| 14 | `3.5` | 56 | `14` | 192 | `48` |
| 16 | `4` | | | 256 | `64` |
| 20 | `5` | | | | |

`.5` increments are valid (`p-0.5`, `p-1.5`, `p-2.5`, `p-3.5`, `mt-7.5`, ...). Off-scale values like `3px`, `5px`, `7px`, `9px`, `0.625rem`, `1.375rem` stay as arbitrary values.

### Unit conversion rules

- **px**: multiple of 4, or in 2-step increments (2, 6, 10, 14) for `.5` classes.
- **rem**: multiple of 0.25rem; 0.125rem maps to `0.5`.
- **em**: keep as arbitrary (context-dependent).
- **%**: convert to fractions where supported: `w-[50%]` → `w-1/2`, `w-[33.333%]` → `w-1/3`, `translate-x-[50%]` → `translate-x-1/2`. Fractions are accepted on `w-*`, `h-*`, `size-*`, `basis-*`, `translate-*`, `inset-*` (and `top/right/bottom/left-*` accept fractions too).
- **CSS functions** (`calc()`, `clamp()`, `min()`, `max()`, `repeat()`): always keep as arbitrary.

## Dynamic utilities — accept any integer without brackets

| Utility | Form | Negative? | Notes |
|---|---|---|---|
| `grid-cols-<n>` / `grid-rows-<n>` | integer | ✗ | `repeat(n, minmax(0, 1fr))` |
| `col-span-<n>` / `row-span-<n>` | integer | ✗ | `col-span-full` / `row-span-full` also valid |
| `col-{start,end}-<n>` / `row-{start,end}-<n>` / `row-<n>` | integer | ✓ | |
| `order-<n>` | integer | ✓ | |
| `z-<n>` | integer | ✓ | |
| `opacity-<n>` | 0–100 | ✗ | Emits `opacity: n%` |
| `duration-<n>` / `delay-<n>` | integer ms | ✗ | |
| `rotate-<n>` / `rotate-{x,y,z}-<n>` | integer deg | ✓ | `-x/-y/-z` axes are v4-new (3D transforms) |
| `scale-<n>` / `scale-{x,y,z}-<n>` | integer % | ✓ (mirror) | `scale-z` is v4-new |
| `skew-<n>` / `skew-{x,y}-<n>` | integer deg | ✓ | |
| `border-<n>` / `border-{t,r,b,l,x,y,s,e}-<n>` | integer px | ✗ | |
| `divide-{x,y}-<n>` | integer px | ✗ | |
| `outline-<n>` | integer px | ✗ | **v4 auto-applies `outline-style: solid`** |
| `outline-offset-<n>` | integer px | ✓ | |
| `ring-<n>` / `inset-ring-<n>` | integer px | ✗ | Default ring is 1px in v4 (was 3px). Use `ring-3` to match v3 `ring`. |
| `grow-<n>` / `shrink-<n>` | number | ✗ | |
| `columns-<n>` | integer | ✗ | |
| `stroke-<n>` | integer | ✗ | |
| `line-clamp-<n>` | integer | ✗ | |
| `aspect-<a>/<b>` | fraction | ✗ | v4-new fraction syntax; also `aspect-square`, `aspect-video` |
| `flex-<n>` | integer | ✗ | |

**Range**: no explicit upper bound in v4. `grid-cols-9999` and `z-1000000` will generate, but keep numbers in normal ranges to avoid bloating CSS.

## Conversion tables (arbitrary → standard)

### Spacing — padding / margin / gap / space / inset

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `p-[16px]`, `p-[1rem]` | `p-4` | |
| `px-[8px]` | `px-2` | |
| `mt-[24px]` | `mt-6` | |
| `-ml-[16px]` | `-ml-4` | Negative via `-` prefix on the class |
| `gap-[12px]` | `gap-3` | |
| `gap-x-[16px]` | `gap-x-4` | |
| `space-y-[8px]` | `space-y-2` | Selector changed in v4 — audit inline-child layouts |
| `p-[2px]` | `p-0.5` | |
| `p-[10px]` | `p-2.5` | |
| `p-[14px]` | `p-3.5` | |
| `p-[3px]` | `p-[3px]` (keep) | Off-scale |
| `top-[16px]` | `top-4` | |
| `top-[-8px]` | `-top-2` | |
| `inset-[0px]` | `inset-0` | |
| `left-[50%]` | `left-1/2` | Fractions OK on inset |
| `z-[99]` | `z-99` | |
| `z-[-1]` | `-z-1` | |

### Sizing — width / height / size / min / max

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `w-[16px] h-[16px]` | `size-4` | `size-*` covers width + height |
| `w-[100px]` | `w-25` | spacing × 25 |
| `w-[50%]` | `w-1/2` | |
| `w-[33.333%]` | `w-1/3` | |
| `min-w-[200px]` | `min-w-50` | |
| `max-w-[768px]` | `max-w-3xl` | Or keep as arbitrary if not on container scale |
| `h-[100dvh]` | `h-dvh` | `dvh`, `svh`, `lvh` have dedicated utilities |

### Typography

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `text-[16px]` | `text-[16px]` or `text-base` | **font-size is theme-namespace only — no integer dynamic form** |
| `text-[14px]/[20px]` | `text-sm/5` | font-size / line-height slash syntax |
| `leading-[24px]` | `leading-6` | **`leading-<number>` is `--spacing`-driven in v4** |
| `leading-[1.5]` | keep, or `leading-normal` | Unitless line-height stays arbitrary |
| `tracking-[0.1em]` | keep | `tracking-*` is theme-namespace only |
| `indent-[16px]` | `indent-4` | Negative OK: `-indent-4` |
| `line-clamp-[7]` | `line-clamp-7` | |

### Border / radius / outline / ring

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `border-[1px]` | `border` or `border-1` | |
| `border-[2px]` | `border-2` | |
| `border-[3px]` | `border-3` | |
| `border-t-[4px]` | `border-t-4` | |
| `rounded-[8px]` | `rounded-lg` | When matches `--radius-*` |
| `rounded-[4px]` | `rounded-sm` | (v3 `rounded-sm` is v4 `rounded-xs`) |
| `rounded-[10px]` | keep | Off-scale |
| `divide-x-[2px]` | `divide-x-2` | |
| `outline-[2px]` | `outline-2` | Auto-applies `outline-style: solid` |
| `outline-offset-[2px]` | `outline-offset-2` | |
| `ring-[2px]` | `ring-2` | Use `ring-3` to match v3 default |

### Effects — opacity / shadow / blur / filter

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `opacity-[0.5]` | `opacity-50` | |
| `opacity-[0.67]` | `opacity-67` or keep | Non-standard integers work but are not surfaced by IntelliSense |
| `shadow-[0_1px_2px_rgb(0_0_0/0.05)]` | `shadow-xs` | If matches `--shadow-*` |
| `blur-[4px]` | `blur-xs` | Theme namespace `--blur-*` |
| `brightness-[1.25]` | `brightness-125` | |
| `contrast-[150]` | `contrast-150` | |
| `saturate-[150]` | `saturate-150` | |
| `hue-rotate-[90deg]` | `hue-rotate-90` | Negative OK: `-hue-rotate-90` |
| `sepia-[100]` | `sepia` | |
| `grayscale-[100]` | `grayscale` | |
| `invert-[100]` | `invert` | |
| `backdrop-blur-[8px]` | `backdrop-blur-sm` | |

### Transform

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `rotate-[45deg]` | `rotate-45` | |
| `rotate-[-15deg]` | `-rotate-15` | |
| `rotate-x-[30deg]` | `rotate-x-30` | **v4-new (3D)** |
| `scale-[1.25]` | `scale-125` | Percentage-of-100 |
| `scale-x-[1.5]` | `scale-x-150` | |
| `scale-z-[2]` | `scale-z-200` | **v4-new** |
| `translate-x-[16px]` | `translate-x-4` | |
| `translate-y-[-8px]` | `-translate-y-2` | |
| `translate-x-[50%]` | `translate-x-1/2` | |
| `translate-z-[8px]` | `translate-z-2` | **v4-new (needs `transform-3d`)** |
| `skew-x-[6deg]` | `skew-x-6` | |

### Transition

| v3 / arbitrary | v4 standard |
|---|---|
| `duration-[200ms]` | `duration-200` |
| `delay-[100ms]` | `delay-100` |
| `duration-[1s]` | `duration-1000` |

### Grid / Flex

| v3 / arbitrary | v4 standard |
|---|---|
| `grid-cols-[13]` | `grid-cols-13` |
| `grid-cols-[repeat(15,minmax(0,1fr))]` | `grid-cols-15` |
| `grid-rows-[8]` | `grid-rows-8` |
| `col-span-[7]` | `col-span-7` |
| `col-start-[3]` | `col-start-3` |
| `row-span-[4]` | `row-span-4` |
| `order-[15]` | `order-15` |
| `basis-[200px]` | `basis-50` |
| `basis-[50%]` | `basis-1/2` |

### Other

| v3 / arbitrary | v4 standard | Notes |
|---|---|---|
| `aspect-[16/9]` | `aspect-video` or `aspect-16/9` | |
| `aspect-[3/2]` | `aspect-3/2` | **v4-new fraction syntax** |
| `aspect-[1/1]` | `aspect-square` | |
| `columns-[5]` | `columns-5` | |
| `stroke-[3]` | `stroke-3` | |
| `fill-[#ff0000]` | keep, or `fill-red-500` | Color literals stay arbitrary unless on palette |

### CSS variable references — syntax shift

v3 wrote `bg-[var(--brand)]`. v4 prefers the shorter `bg-(--brand)`:

```html
<!-- v3 -->
<div class="bg-[var(--brand)] w-[var(--sidebar)]">

<!-- v4 -->
<div class="bg-(--brand) w-(--sidebar)">
```

## When arbitrary values are still required

### Theme-namespace-only utilities

These read only from their theme namespace — no dynamic integer form:

| Namespace | Utilities | Scale keys |
|---|---|---|
| `--radius-*` | `rounded-*` | `xs`, `sm`, `md`, `lg`, `xl`, `2xl`, `3xl`, `4xl`, `full`, `none` |
| `--text-*` | `text-*` (font-size) | `xs`, `sm`, `base`, `lg`, `xl`, `2xl`, ..., `9xl` |
| `--shadow-*` | `shadow-*` | xs / sm / md / lg / xl / 2xl |
| `--inset-shadow-*` | `inset-shadow-*` | |
| `--drop-shadow-*` | `drop-shadow-*` | |
| `--blur-*` | `blur-*`, `backdrop-blur-*` | xs / sm / md / lg / xl / 2xl / 3xl |
| `--ease-*` | `ease-*` | |
| `--tracking-*` | `tracking-*` | tighter / tight / normal / wide / wider / widest |
| `--font-*` | `font-*` (family) | |
| `--aspect-*` | `aspect-<name>` (square / video / auto) — fractions like `aspect-3/2` are a separate dynamic form |
| `--perspective-*` | `perspective-*` | dramatic / near / normal / midrange / distant / none |
| `--animate-*` | `animate-*` | |
| `--container-*` | `max-w-md`, `columns-md`, `basis-md`, ... | t-shirt keys only |

### Off-scale numeric values

- Off-scale px: `3px`, `5px`, `7px`, `9px`, `11px`, `13px`, `15px`, ...
- Non-typical rem: `0.625rem`, `1.375rem`, ...
- The `.5` increments Tailwind allows (`p-0.5`, `p-3.5`, ...) are the *only* fractional steps without arbitrary values.

### CSS functions / expressions

| Pattern | Example |
|---|---|
| `clamp(...)` | `text-[clamp(1rem,2vw,2rem)]` |
| `calc(...)` | `w-[calc(100%-2rem)]` |
| `min(...)`, `max(...)` | `h-[max(50vh,400px)]` |
| `var(...)` | `bg-(--my-color)` (v4 short form) |
| Complex grid tracks | `grid-cols-[200px_minmax(900px,_1fr)_100px]` |
| Color literals | `text-[#ff5722]` |
| Compound transforms | `transform-[matrix(1,2,3,4,5,6)]` |

## Negative values

Always prefix the **class**, not the value:

```html
<!-- ✅ --> <div class="-mt-4 -translate-x-2 -z-10 -rotate-45">
<!-- ❌ (works but verbose) --> <div class="mt-[-16px]">
```

**Supports negatives**: all margin axes, all inset/positioning (`-top-*`, `-inset-*`, `-start-*`, `-end-*`), all translate axes, all rotate axes, all skew axes, all scale axes (mirrors), `-z-*`, `-order-*`, `-col-{start,end}-*`, `-row-{start,end}-*`, `-row-*`, `-indent-*`, `-outline-offset-*`, `-hue-rotate-*`, `-tracking-<key>` (custom scale only).

**Does NOT support negatives** (negative CSS value would be meaningless): `p-*`, `w-*`, `h-*`, `size-*`, `gap-*`, `border-*` (width), `outline-*` (width), `ring-*`, `opacity-*`, `duration-*`, `delay-*`, `shadow-*`, `blur-*`, `rounded-*`, `basis-*`, `grow-*`, `shrink-*`, `col-span-*`, `row-span-*`, `grid-cols-*`, `grid-rows-*`, `columns-*`, `stroke-*`.

## Decimal values

`--spacing`-driven utilities accept `.5`-step decimals natively:

```html
<div class="p-0.5 p-1.5 p-2.5 p-3.5 mt-7.5">
```

IntelliSense surfaces `0.5, 1.5, 2.5, 3.5` only, but the runtime accepts any decimal (`p-4.5`, `mt-12.75`). Integer-only dynamic utilities (`scale-*`, `rotate-*`) need arbitrary values for decimals:

```html
<div class="scale-[1.05] rotate-[2.5deg]">
```

## Conversion flowchart

```
class with `[...]`?
├── no  → check migration.md rename/removed tables
└── yes → which category?
         ├── (a) dynamic integer utility
         │       (grid-cols / grid-rows / col-span / row-span / order / z /
         │        opacity / duration / delay / rotate / scale / skew /
         │        border / outline / ring / columns / stroke / line-clamp / aspect)
         │     └── integer inside? yes → strip brackets (`z-[99]` → `z-99`)
         │                         no  → keep arbitrary
         │
         ├── (b) --spacing-driven utility
         │       (p / m / w / h / size / gap / inset / top..left /
         │        translate / space / scroll-{m,p} / leading / indent / basis<n>)
         │     └── value is:
         │         ├── px         → ÷4 → class number (`[16px]` → `4`)
         │         ├── rem        → ÷0.25 → class number (`[1rem]` → `4`)
         │         ├── %          → fraction (`[50%]` → `1/2`) — w / h / size / basis / translate / inset only
         │         ├── off-scale  → keep arbitrary
         │         └── calc/clamp/var → keep arbitrary
         │
         ├── (c) theme-namespace-only utility
         │       (rounded / text<size> / shadow / blur / tracking / ease /
         │        aspect<name> / perspective / animate / font<family>)
         │     └── matches a scale key (xs/sm/md/lg/xl/...)?
         │         yes → use the named key (`rounded-[8px]` → `rounded-lg`)
         │         no  → keep arbitrary
         │
         └── (d) fraction-capable utility (w / h / size / basis / translate / inset)
                 └── % matches a clean fraction? convert; else keep
```

Side rules:
- `[var(--foo)]` → `(--foo)` (v4 short form).
- Negatives: `mt-[-16px]` → `-mt-4`. Only valid for utilities in the negative-supporting list above.

## Migration tactics

1. **Phase 1**: `npx @tailwindcss/upgrade` (Node 20+) on a fresh branch. Handles renames, removed utilities, comma-in-arbitrary → underscore, `!`-position migration, and (since v4.1.7) bare-value / negative-arbitrary migration.
2. **Phase 2**: Grep for residual `[...]` arbitrary values the codemod left alone. Apply the conversion tables above.
3. **Phase 3**: Visual regression — default-color/ring/cursor changes (see migration.md §"Default behavior changes") aren't caught by class-name rewrites.

When ambiguous (e.g. `rounded-[7px]` between `rounded-md` (6px) and `rounded-lg` (8px)), keep the arbitrary value — that's a design call, not a mechanical one.

## Caveats

- **`opacity-<n>` outside the default ladder** (`opacity-67`) emits valid CSS but isn't suggested by IntelliSense. Prefer the standard ladder (0/5/10/.../95/100) for consistency.
- **`leading-<number>` being `--spacing`-driven is v4-new**; v3 read `leading-*` from the `--leading-*` namespace exclusively. In v4 both coexist: `leading-none/tight/snug/normal/relaxed/loose` (named) ↔ `leading-6`, `leading-12` (spacing-driven).
- **Don't read `text-<n>` as dynamic** — `text-14`, `text-16` do *not* produce font-size utilities. Use `text-[14px]` or extend `--text-*` in `@theme`.
- **Project-customized `--spacing`** invalidates the `px ÷ 4` formula — recompute against the actual value.
