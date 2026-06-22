# Nuxt Integration & SSR

Specifics for using VueUse inside Nuxt 3.x / 4.x projects, plus general SSR notes for any Vue 3 SSR setup.

---

## 1. Setup

```bash
# Recommended
npx nuxt@latest module add vueuse

# Manual
npm i @vueuse/nuxt @vueuse/core
```

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@vueuse/nuxt'],
})
```

`@vueuse/nuxt` 14.0.0+ uses the **Nuxt v4 kit** and supports both Nuxt 3.x and 4.x natively.

---

## 2. Auto-import behavior

Every `@vueuse/core` function becomes available without `import` inside:

- `.vue` components (`<script setup>`)
- `composables/*.ts`
- `app.vue`
- Nuxt server routes (`server/api/*.ts`)
- TypeScript completion is wired up by the module

You can still write explicit imports — they just become optional.

```vue
<script setup lang="ts">
// no import needed
const { x, y } = useMouse()
const settings = useLocalStorage('app', { theme: 'light' })
</script>
```

---

## 3. Auto-import exclusions (collisions with Nuxt built-ins)

These VueUse exports are **NOT** auto-imported in a Nuxt project — Nuxt's own versions take precedence. If you actually want VueUse's, import explicitly.

| VueUse export | Nuxt built-in | Recommendation |
|---|---|---|
| `toRef` | Vue native | use the native one (already auto-imported by Nuxt) |
| `toRefs` | Vue native | same |
| `toValue` | Vue native | `import { toValue } from 'vue'` |
| `useFetch` | Nuxt's `useFetch` (SSR payload, streaming, dedupe) | use Nuxt's — almost always what you want |
| `useCookie` | Nuxt's `useCookie` (cookie SSR/CSR sync) | use Nuxt's |
| `useHead` | `@unhead/vue`'s `useHead` | use `@unhead/vue`'s |
| `useTitle` | covered by `useHead({ title })` | prefer `useHead` |
| `useStorage` | Nitro's `useStorage` (server-side KV) | use Nuxt's on server; for client localStorage, explicitly import VueUse's |
| `useImage` | `@nuxt/image`'s `useImage` | use `@nuxt/image`'s |

### When you DO want VueUse's `useStorage` on the client

```ts
// composables/useClientSettings.ts
import { useStorage } from '@vueuse/core'

export function useClientSettings() {
  return useStorage('client-settings', { theme: 'light' })
}
```

### Common mistake — VueUse `useFetch` in Nuxt

```ts
// ❌ (in Nuxt) — skips SSR payload + streaming + dedupe
import { useFetch } from '@vueuse/core'
const { data } = useFetch('/api/posts')

// ✅ — Nuxt's auto-imported useFetch
const { data } = await useFetch('/api/posts')
```

---

## 4. SSR behavior

### Defaults on the server

VueUse composables return sensible defaults on the server, so calling them in `<script setup>` outside any guard is fine **for the most part**.

```ts
const { width, height } = useWindowSize()
// Server: width = Infinity (configurable), height = Infinity
// Client: real values after mount
```

### Hydration mismatches to watch for

These composables can produce client values that differ from what the server rendered:

- `useWindowSize` / `useWindowFocus` / `useDocumentVisibility`
- `useMediaQuery`
- `useBreakpoints` (any preset)
- `usePreferredDark` / `usePreferredColorScheme`
- `useDark` (depends on `usePreferredDark` unless storage-backed)

**Mitigations:**

1. **`useSSRWidth(width)`** — set an assumed server viewport, so breakpoint composables render the same on server and client:
   ```ts
   // app.vue or a plugin
   import { provideSSRWidth } from '@vueuse/core'
   provideSSRWidth(1024)  // mobile-first sites: 375 / desktop: 1280
   ```
2. **Render only on client** — wrap in `<ClientOnly>` for components whose UI genuinely depends on viewport state:
   ```vue
   <ClientOnly>
     <DesktopOnlyWidget />
   </ClientOnly>
   ```
3. **Persist in cookie** — for `useDark` / `useColorMode`, use a cookie-backed storage so the server can read the user's preference and render correctly the first time.

### When to write `if (isClient)`

When you genuinely need a browser-only side effect (writing to `localStorage` outside `useStorage`, calling a third-party `window.foo()`):

```ts
import { isClient } from '@vueuse/core'

if (isClient) {
  myThirdPartyLib.init()
}
```

For most cases, `tryOnMounted` is cleaner because it also handles the "called outside a component" edge case.

---

## 5. Server routes

VueUse functions that don't touch browser APIs (e.g. `useEventBus`, `useArrayFilter`, math/utility composables) work in `server/api/*.ts` too. Browser-API composables are no-ops or return defaults on the server — usually not what you want server-side, so reach for native APIs there.

---

## 6. Composables vs Nuxt's composables — quick rule

> If Nuxt provides a same-named composable, use **Nuxt's**.

`useFetch`, `useCookie`, `useStorage`, `useHead`, `useTitle`, `useImage` — always Nuxt's in a Nuxt project. VueUse's versions exist for non-Nuxt apps.

For everything else (`useEventListener`, `useLocalStorage`, `useMouse`, `useScroll`, `useWebSocket`, `useDebounceFn`, breakpoints, …) — VueUse is the answer.

---

## 7. Pinia + VueUse

Composables called from a Pinia store run outside a component `setup()`. Two rules:

1. Use `tryOnScopeDispose` for cleanup (not `onUnmounted`).
2. Wrap multiple composable calls in an `effectScope` if you want to dispose them together.

```ts
// stores/mouse.ts
import { defineStore } from 'pinia'
import { effectScope } from 'vue'
import { useEventListener, useMouse, tryOnScopeDispose } from '@vueuse/core'

export const useMouseStore = defineStore('mouse', () => {
  const scope = effectScope(true)
  const state = scope.run(() => useMouse())!

  tryOnScopeDispose(() => scope.stop())

  return state
})
```
