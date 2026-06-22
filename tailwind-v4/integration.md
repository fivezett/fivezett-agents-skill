# Tailwind v4 Setup & Framework Integration

Setup snippets for Vite, Nuxt 4/3, PostCSS, plus Vue/Nuxt-specific patterns and shadcn/ui on v4.

## Setup snippets

### Plain Vite project

```bash
npm install tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import { defineConfig } from "vite"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss()],
})
```

```css
/* src/main.css */
@import "tailwindcss";
```

### Nuxt 4 (app/ directory structure)

```ts
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  compatibilityDate: "2025-07-15",
  css: ["./app/assets/css/main.css"],
  vite: {
    plugins: [tailwindcss()],
  },
})
```

```css
/* app/assets/css/main.css */
@import "tailwindcss";
```

### Nuxt 3 (legacy ~/ alias)

```ts
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  css: ["~/assets/css/main.css"],
  vite: {
    plugins: [tailwindcss()],
  },
})
```

No separate Nuxt module is needed for v4. The Vite plugin is the supported path. The v3-era `@nuxtjs/tailwindcss` module is **not v4-compatible**.

### PostCSS (only if you can't use the Vite plugin)

```bash
npm install tailwindcss @tailwindcss/postcss
```

```js
// postcss.config.js
export default {
  plugins: { "@tailwindcss/postcss": {} },
}
```

Prefer `@tailwindcss/vite` whenever the bundler is Vite — it's faster and integrates more tightly than the PostCSS path.

## Vue / Nuxt integration patterns

### When to use `<style scoped>`

Default: don't. Use Tailwind classes in the template. Reach for `<style scoped>` only when:

- You need keyframes that aren't trivial to express via `@theme`'s `--animate-*` block.
- You need a complex selector Tailwind can't model cleanly (deep child traversal, `:has()` patterns with multiple conditions).
- You're overriding third-party widget internals.

When you do use `<style scoped>` with `@apply`, `@variant`, or `theme(...)`, **always** add `@reference` at the top:

```vue
<style scoped>
@reference "@/assets/css/main.css";

.card {
  @apply bg-brand text-brand-fg rounded-card;
}
</style>
```

Without `@reference`, custom theme tokens won't resolve and `@apply` will throw `Cannot apply unknown utility class`.

### Reactive dynamic values

For values that change per-instance at runtime, use `v-bind()` to write to a CSS custom property and read it with a Tailwind utility:

```vue
<template>
  <div class="bg-(--card-bg) text-(--card-fg) p-4 rounded-card">
    ...
  </div>
</template>

<script setup>
const props = defineProps<{ bg: string; fg: string }>()
</script>

<style scoped>
div {
  --card-bg: v-bind(bg);
  --card-fg: v-bind(fg);
}
</style>
```

This keeps the design system inside Tailwind utilities while threading reactive values through CSS variables. The Tailwind utility's modifiers (opacity, etc.) still work because the value flows through standard `var()` resolution.

### Class composition

For multi-variant components, prefer `tailwind-variants` or `class-variance-authority` over hand-rolled conditionals — they emit whole class strings (scanner-friendly). When merging conditional class lists, use `clsx` + `tailwind-merge` (`tailwind-merge` v3+ for v4 rename support).

### shadcn/ui on v4

shadcn/ui has migrated to v4. Key points:

- Use the v4 templates, not v3 ones.
- The animation plugin moved from `tailwindcss-animate` to `tw-animate-css`.
- If you symlink or vendor shadcn into a path outside default scanning, add `@source "../node_modules/@shadcn/ui";` or similar.
