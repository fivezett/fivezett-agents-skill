# Tailwind v3 → v4 Migration & Version Coverage

Migration cheatsheet, default behavior changes, version coverage matrix, and post-v4.1.x feature notes.

## Automated upgrade

`npx @tailwindcss/upgrade` (requires Node 20+) handles ~90% of mechanical changes. As of **v4.1.5**, the same tool also handles v4.* → v4.* upgrades (named-value migration, etc.). Run it in a fresh branch and diff carefully — it catches mechanical changes but not default-value regressions.

## Renamed utilities

| v3 | v4 |
|---|---|
| `shadow` | `shadow-sm` |
| `shadow-sm` | `shadow-xs` |
| `drop-shadow` | `drop-shadow-sm` |
| `drop-shadow-sm` | `drop-shadow-xs` |
| `blur` | `blur-sm` |
| `blur-sm` | `blur-xs` |
| `backdrop-blur` | `backdrop-blur-sm` |
| `backdrop-blur-sm` | `backdrop-blur-xs` |
| `rounded` | `rounded-sm` |
| `rounded-sm` | `rounded-xs` |
| `outline-none` (transparent outline for forced-colors a11y) | `outline-hidden` |
| `ring` (3px) | `ring-3` (default ring is now 1px) |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `break-words` | `wrap-break-word` *(v4.1.15+ upgrade-tool target)* |

> **`outline-none` behavior change**: in v4, `outline-none` now *actually* sets `outline-style: none`. The old v3 behavior (transparent outline that stays visible in forced-colors mode) moved to `outline-hidden`. If you're disabling focus rings for design but still want a11y, use `outline-hidden`.

## Removed utilities → replacements

| v3 | v4 |
|---|---|
| `bg-opacity-50` | `bg-black/50` (or `bg-(--token)/50`) |
| `text-opacity-50` | `text-black/50` |
| `border-opacity-50` | `border-black/50` |
| `divide-opacity-50` | `divide-black/50` |
| `ring-opacity-50` | `ring-black/50` |
| `placeholder-opacity-50` | `placeholder-black/50` |
| `flex-shrink-*` | `shrink-*` |
| `flex-grow-*` | `grow-*` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` |
| `decoration-clone` | `box-decoration-clone` |

## Deprecated in v4.1.0 (still work, but migrate)

| Deprecated | Replacement |
|---|---|
| `bg-left-top` | `bg-top-left` |
| `bg-left-bottom` | `bg-bottom-left` |
| `bg-right-top` | `bg-top-right` |
| `bg-right-bottom` | `bg-bottom-right` |
| `object-left-top` | `object-top-left` |
| `object-left-bottom` | `object-bottom-left` |
| `object-right-top` | `object-top-right` |
| `object-right-bottom` | `object-bottom-right` |

## Setup migration

```css
/* v3 */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* v4 */
@import "tailwindcss";
```

## Config migration

```js
// v3 — tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: { brand: "#3b82f6" },
      fontFamily: { display: ["Inter", "sans-serif"] },
    },
  },
};
```

```css
/* v4 — main.css */
@import "tailwindcss";

@theme {
  --color-brand: #3b82f6;
  --font-display: "Inter", sans-serif;
}
```

## Dark mode

```js
// v3
darkMode: "class"
```

```css
/* v4 */
@custom-variant dark (&:where(.dark, .dark *));
```

## Container customization

```js
// v3
theme: { container: { center: true, padding: "2rem" } }
```

```css
/* v4 */
@utility container {
  margin-inline: auto;
  padding-inline: 2rem;
}
```

## Important marker position

```html
<!-- v3 -->
<div class="!bg-red-500 hover:!bg-red-600">

<!-- v4 -->
<div class="bg-red-500! hover:bg-red-600!">
```

## Variant ordering

In v3, stacked variants applied right-to-left. In v4, they apply **left-to-right**, matching CSS pseudo-selector intuition.

```html
<!-- v3: hover:dark:bg-black meant "in dark mode, on hover" -->
<!-- v4: dark:hover:bg-black means "in dark mode, on hover" -->
```

The codemod rewrites these, but verify hover states inside dark mode after upgrading.

## Other subtle changes

- `hover:` is now wrapped in `@media (hover: hover)` — touch devices skip hover styles. Override with `@custom-variant hover (&:hover);` to restore v3 behavior.
- `space-x-*` / `space-y-*` selector changed from `> :not([hidden]) ~ :not([hidden])` to `> :not(:last-child)`. Inline children render differently — audit list/inline layouts.
- `divide-x-*` / `divide-y-*` use the same new selector pattern.
- Arbitrary values no longer accept commas — use underscores: `grid-cols-[1fr_2fr_3fr]`.
- `var(--foo_bar)` keeps the underscore in v4. v3 converted it to space.
- Gradient stops persist across variants. To clear a `via-*` in dark mode, explicitly add `dark:via-none`.

## Default behavior changes (silent breakages)

These don't error but will look different after upgrade:

- **Border / divide default color**: `gray-200` → `currentColor`. Add explicit `border-gray-200` if you relied on the default.
- **Ring default**: 3px `blue-500` → 1px `currentColor`. Add `ring-3 ring-blue-500` to restore.
- **Placeholder color**: theme gray → current text color at 50% opacity.
- **Button cursor**: `pointer` → `default`. Add `cursor-pointer` explicitly on `<button>` if you want the old behavior.
- **`<dialog>` margins**: default centering margins reset by Preflight. Position manually.
- **Default color space**: palette switched to **OKLCH**. Visual output is similar but not identical to v3's sRGB defaults.
- **`hover:` on touch**: no longer triggers (see above).

---

## Version Coverage Matrix

Track here which features are available in the pinned version. When upgrading, compare against the latest CHANGELOG entries and update this table.

| Feature | Introduced | Status in v4.1.18 | Notes |
|---|---|---|---|
| `@import "tailwindcss"` entry point | v4.0.0 | ✅ stable | Replaces `@tailwind base/components/utilities` |
| `@theme` directive | v4.0.0 | ✅ stable | CSS-first config |
| `@theme inline` / `reference` / `static` | v4.0.0 | ✅ stable | |
| `@utility` (static + functional) | v4.0.0 | ✅ stable | Replaces `@layer components` for apply-able classes |
| `--value()` / `--modifier()` | v4.0.0 | ✅ stable | Bare types: number, integer, ratio, percentage only |
| Warning on unsupported `--value()` bare type | v4.1.3 | ✅ stable | |
| `@variant` / `@custom-variant` | v4.0.0 | ✅ stable | |
| `@source "..."` | v4.0.0 | ✅ stable | |
| `@source not "..."` | v4.1.0 | ✅ stable | Path exclusion |
| `@source inline(...)` | v4.1.0 | ✅ stable | Safelist replacement |
| `@source not inline(...)` | v4.1.0 | ✅ stable | Blocklist |
| Brace expansion in `@source inline()` | v4.1.0 | ✅ stable | `{hover:,}bg-red-{50,{100..900..100},950}` |
| `@reference` | v4.0.0 | ✅ stable | Required for scoped style blocks |
| `@config` / `@plugin` (legacy) | v4.0.0 | ✅ stable | For v3 migration only |
| `@apply` from `@layer components` | — | ❌ removed | Use `@utility` instead |
| Container queries (built-in) | v4.0.0 | ✅ stable | `@container`, `@sm:`, `@md:` |
| 3D transforms | v4.0.0 | ✅ stable | `rotate-x-*`, `translate-z-*`, etc. |
| `size-*` shorthand | v4.0.0 | ✅ stable | |
| `field-sizing-*` | v4.0.0 | ✅ stable | |
| Logical properties (`ps-*`, `pe-*`, `ms-*`, `me-*`, `start-*`, `end-*`) | v4.0.0 | ✅ stable | `start-*`/`end-*` deprecated in v4.2.0 (out of range) |
| Conic/radial gradients + interpolation | v4.0.0 | ✅ stable | `bg-conic`, `bg-radial`, `/oklch`, `/hsl` |
| `inert:` variant | v4.0.0 | ✅ stable | |
| `forced-colors:` variant + `forced-color-adjust-*` | v4.0.0 | ✅ stable | |
| `not-*` variant | v4.0.0 | ✅ stable | |
| `@starting-style` (`starting:`) | v4.0.0 | ✅ stable | |
| `*:` / `**:` (children / descendants) | v4.0.0 | ✅ stable | |
| `font-stretch-*` | v4.0.0 | ✅ stable | |
| `inset-ring*` | v4.0.0 | ✅ stable | |
| OKLCH default palette | v4.0.0 | ✅ stable | |
| `text-shadow-*` | v4.1.0 | ✅ stable | |
| `mask-*` compositional | v4.1.0 | ✅ stable | |
| Colored `drop-shadow-<color>/<alpha>` | v4.1.0 | ✅ stable | |
| `wrap-anywhere` / `wrap-break-word` / `wrap-normal` | v4.1.0 | ✅ stable | |
| `items-baseline-last` / `self-baseline-last` | v4.1.0 | ✅ stable | |
| `*-safe` alignment (`justify-center-safe`, etc.) | v4.1.0 | ✅ stable | |
| `pointer-*` / `any-pointer-*` variants | v4.1.0 | ✅ stable | |
| `noscript:` / `inverted-colors:` | v4.1.0 | ✅ stable | |
| `user-valid:` / `user-invalid:` | v4.1.0 | ✅ stable | |
| `details-content:` | v4.1.0 | ✅ stable | |
| `bg-position-(...)` / `bg-size-(...)` arbitrary | v4.1.0 | ✅ stable | |
| Shadow opacity modifier (`shadow-lg/30`) | v4.1.0 | ✅ stable | Applies to shadow/inset-shadow/drop-shadow/text-shadow |
| `node_modules` excluded by default | v4.1.0 | ✅ stable | Use `@source` to include |
| Improved Safari/Firefox legacy fallbacks | v4.1.0 | ✅ stable | `oklab`, `@property`, opacity fallbacks |
| `bg-{left,right}-{top,bottom}` deprecation | v4.1.0 | ⚠️ deprecated | Use `bg-{top,bottom}-{left,right}` |
| `object-{left,right}-{top,bottom}` deprecation | v4.1.0 | ⚠️ deprecated | Use `object-{top,bottom}-{left,right}` |
| `@tailwindcss/oxide-wasm32-wasi` | v4.1.4 | 🧪 experimental | StackBlitz/WASM runtimes |
| `@tailwindcss/upgrade` for v4→v4 | v4.1.5 | ✅ stable | Named-value migration, bare-value migration |
| `h-lh` / `min-h-lh` / `max-h-lh` | v4.1.5 | ✅ stable | line-height unit |
| `transition` includes `display`/`visibility`/`content-visibility`/`overlay`/`pointer-events` | v4.1.5 | ✅ stable | Simplifies `@starting-style` |
| Source maps in dev | v4.1.6 | ✅ stable | |
| `DEBUG=*` logs for `@source` | v4.1.6 | ✅ stable | |
| Upgrade-tool: bare-value migration | v4.1.7 | ✅ stable | |
| Upgrade-tool: arbitrary modifier → bare | v4.1.9 | ✅ stable | `/[0.16]` → `/16` |
| Upgrade-tool: negative arbitrary → negative bare | v4.1.9 | ✅ stable | |
| `--watch=always` CLI option | v4.1.11 | ✅ stable | |
| `@tailwindcss/vite` Vite 7 support | v4.1.11 | ✅ stable | |
| `@tailwindcss/postcss` `transformAssetUrls: false` | v4.1.12 | ✅ stable | URL rebase opt-out |
| `transition` no longer transitions `visibility` | v4.1.13 | ✅ stable | Correction |
| Upgrade-tool: `break-words` → `wrap-break-word` | v4.1.15 | ✅ stable | |
| `@tailwindcss/vite` Vite environment API | v4.1.18 | ✅ stable | |

---

## Out-of-range notes (post-v4.1.x)

Features available in v4.2.0+ but **not in v4.1.18**. Mention to the user if they ask about these but the project is pinned to v4.1.x:

- **v4.2.0 (2026-02-18)**: Block/inline logical properties (`pbs-*`, `pbe-*`, `mbs-*`, `mbe-*`, `inset-bs-*`, `inset-be-*`, `border-bs-*`, `border-be-*`, `inline-*`, `block-*`, `scroll-pbs-*`, `scroll-pbe-*`, `scroll-mbs-*`, `scroll-mbe-*`); `font-features-*`; new color palette additions (Mauve, Olive, Mist, Taupe); **deprecation of `start-*`/`end-*` in favor of `inset-s-*`/`inset-e-*`**; official Webpack plugin.
- **v4.3.0 (2026-05-08)**: `zoom-*`, `tab-*`, scrollbar utilities, `@container-size`, additional gradient interpolation modes.

If the user is on v4.1.x and wants any of the above, advise upgrading to the relevant minor version with `npx @tailwindcss/upgrade`.

---

## Maintenance workflow

To keep this skill in sync with upstream releases:

1. **Trigger**: A new `tailwindcss` version is published.
2. **Diff**: Compare the version metadata in SKILL.md with the new release. Fetch the CHANGELOG entries between them (https://github.com/tailwindlabs/tailwindcss/releases).
3. **Audit**: For each new release entry, classify:
   - **New utility / variant / directive** → MUST add to the relevant section (features.md or directives.md) AND to the Version Coverage Matrix above with `Introduced: vX.Y.Z`.
   - **Behavior change** → SHOULD add to "Default behavior changes" or the relevant section with a version tag.
   - **Bug fix** → MAY skip unless it changed observable behavior.
   - **Deprecation** → MUST add to migration tables with a "deprecated in vX.Y.Z" note.
4. **Update**: Bump version metadata in SKILL.md (`Target`, `Range`, `Last verified`, `Last changelog check`).
5. **Verify**: Test the new code patterns against a real v4 project before publishing.
