---
name: vueuse-composables
description: Use when writing or reviewing Vue 3 / Nuxt 3-4 code that imports from `@vueuse/core` / `@vueuse/integrations` / `@vueuse/router` / `@vueuse/math` / `@vueuse/nuxt`, or about to write custom debounce/throttle, manual `addEventListener` / `removeEventListener` cleanup, direct `window` / `document` / `navigator` access, `localStorage` / `sessionStorage`, `ResizeObserver` / `IntersectionObserver` / `MutationObserver` wiring, `WebSocket` / `EventSource` boilerplate, or you see `watchPausable` / `computedEager` / `templateRef` / `breakpointsVuetify` / `unref` on a getter / `require('@vueuse/...')` / root imports from `@vueuse/integrations`. Targets `@vueuse/core` 14.3.0, Vue 3.5+.
---

# VueUse Composables Reference

Reference for **`@vueuse/core` 14.3.x** (pinned 14.3.0). Use VueUse composables instead of rolling your own DOM / browser-API glue. Cover composables only — `@vueuse/components` (renderless components + directives) is intentionally out of scope.

- **Target**: `@vueuse/core` 14.3.0 (released 2026-05-01)
- **Vue peer**: `^3.5.0` (raised in v14.0.0; ESM-only since v13.0.0)
- **Nuxt peer**: 3.x / 4.x (`@vueuse/nuxt` v14.0.0+ uses the Nuxt v4 kit)
- **Last verified**: 2026-05-24
- **Last changelog check**: https://github.com/vueuse/vueuse/releases/tag/v14.3.0

## When to use

Apply whenever you are about to:

- Write or refactor a `<script setup>` / composable that touches DOM, browser APIs, timers, async state, or storage
- Replace hand-rolled `addEventListener` + `removeEventListener`, `setInterval` + `clearInterval`, debounce / throttle, `ResizeObserver` / `IntersectionObserver` / `MutationObserver` wiring
- Reach for `localStorage`, `sessionStorage`, IndexedDB, cookies, or URL search params
- Add `WebSocket`, `EventSource`, `fetch` / `axios` calls with loading + error state
- Detect dark mode, media queries, breakpoints, online status, network info, geolocation, battery, clipboard, etc.
- Review a PR that imports from any `@vueuse/*` package
- Migrate code off `watchPausable`, `computedEager`, `templateRef`, or `toValue` from `@vueuse/shared`

**Concrete symptoms — STOP and check here:**

- About to type `window.addEventListener(...)`, `document.addEventListener(...)`, `addEventListener('resize', ...)`
- About to type `new ResizeObserver(...)`, `new IntersectionObserver(...)`, `new MutationObserver(...)`, `new WebSocket(...)`, `new EventSource(...)`
- About to type `setInterval(`, `setTimeout(` that you'll need to clean up in `onUnmounted`
- About to write a `debounce` / `throttle` helper from scratch
- About to read `localStorage.getItem` / `localStorage.setItem` or `sessionStorage.*`
- About to read `navigator.clipboard`, `navigator.geolocation`, `navigator.onLine`, `navigator.battery`, `navigator.share`, `navigator.permissions`
- About to read `window.innerWidth` / `window.innerHeight`, `window.matchMedia('...')`
- About to wire up `useTemplateRef` (good!) — but seeing `templateRef('...')` from `@vueuse/core` (legacy, deprecated)
- About to type `unref(input)` where `input` could be a getter — use `toValue` instead
- About to import `{ toValue }` or `{ MaybeRef, MaybeRefOrGetter }` from `@vueuse/shared` (deprecated; import from `vue`)
- About to import `{ useAxios }` / `{ useIDBKeyval }` / `{ useFocusTrap }` from `@vueuse/integrations` root (must be subpath import)
- About to write `require('@vueuse/core')` (CJS removed in v13.0.0; ESM only)
- About to write `import { useFetch } from '@vueuse/core'` inside a Nuxt project (use Nuxt's `useFetch` instead — see [nuxt.md](nuxt.md))

**When NOT to use:** Project pins Vue 2 (use `@vueuse/core@11.x`, not 14), or uses `@vueuse/head` (sunset — migrate to `@unhead/vue`).

## Why this matters

Three failure modes you must actively prevent:

1. **Hand-rolled glue** — writing a debounce/throttle/event-listener helper that VueUse already ships, with worse cleanup behavior than `useEventListener` / `useDebounceFn` / `useIntervalFn`.
2. **Deprecated APIs that still "work"** — `watchPausable`, `computedEager`, `templateRef`, `toValue` from `@vueuse/shared`, `breakpointsVuetify` (no version suffix), `import from '@vueuse/integrations'` (root). These ship warnings or pull unwanted peers but won't error loudly.
3. **SSR / reactivity loss** — touching `window` outside a composable, calling `unref()` on a getter (returns the function!), destructuring composable return then snapshotting `.value`, passing `reactive(...).x` instead of `() => reactive.x` to a composable.

When in doubt about a composable name or import path: **search <https://vueuse.org/functions> or grep the existing codebase. Do not invent names.**

## Hard rules

These apply to every Vue 3 / Nuxt project using VueUse:

1. **Never** `import { toValue, type MaybeRef, type MaybeRefOrGetter } from '@vueuse/shared'`. Import them from `'vue'` (deprecated in v12.3.0 / v12.8.0; still exported but flagged).
2. **Never** `import { templateRef } from '@vueuse/core'` in new code. Use Vue's native `useTemplateRef('key')` (deprecated in v13.6.0).
3. **Never** use `watchPausable` (deprecated in v14.0.0). Vue 3.5's `watch()` returns `{ stop, pause, resume }` natively.
4. **Never** use `computedEager` (deprecated in v14.0.0). Vue 3.4+ `computed()` already invalidates only on value change.
5. **Never** use `breakpointsVuetify` (deprecated alias). Use `breakpointsVuetifyV2` or `breakpointsVuetifyV3` explicitly.
6. **Never** root-import from `@vueuse/integrations`. Always subpath: `import { useAxios } from '@vueuse/integrations/useAxios'` (root import disables tree-shaking and pulls every optional peer).
7. **Never** `require('@vueuse/core')` or `require('@vueuse/*')`. CJS was removed in v13.0.0 — ESM only.
8. **Never** call `unref(input)` when `input: MaybeRefOrGetter<T>`. `unref` does not invoke getters (function stays a function). Use `toValue(input)`.
9. **Never** call a composable outside `setup()` / another composable / an `effectScope` run callback (cleanup hook registration breaks). Use `tryOnMounted` / `tryOnScopeDispose` when you must.
10. **Never** touch `window`, `document`, `navigator` directly when a VueUse composable covers it. They crash under SSR; VueUse returns sensible server-side defaults.
11. **Never** destructure a composable result and snapshot `.value` — keep the refs and pass them around: `const { x, y } = useMouse()` then watch `x`, don't `const pos = { x: x.value }`.
12. **Never** pass `reactive(state).url` to a composable expecting `MaybeRefOrGetter<string>`. The lookup snapshots the string and loses reactivity. Pass `() => state.url` or `toRef(state, 'url')`.
13. **In Nuxt projects:** never `import { useFetch / useCookie / useStorage / useHead / useTitle / useImage } from '@vueuse/core'` — these collide with Nuxt built-ins. Use Nuxt's auto-imported versions. See [nuxt.md](nuxt.md).

## Top deprecated → current map

| Deprecated / removed | Use instead | Since |
|---|---|---|
| `toValue` from `@vueuse/shared` | `toValue` from `'vue'` | v12.3.0 |
| `MaybeRef`, `MaybeRefOrGetter` from `@vueuse/shared` | from `'vue'` | v12.8.0 |
| `templateRef('key')` | `useTemplateRef('key')` from `'vue'` | v13.6.0 |
| `useTemplateRefsList` | Vue 3.5 array refs | v13.6.0 |
| `watchPausable(src, cb)` | `const { pause, resume, stop } = watch(src, cb)` (Vue 3.5) | v14.0.0 |
| `computedEager(fn)` | `computed(fn)` (Vue 3.4+ skips invalidation on same value) | v14.0.0 |
| `breakpointsVuetify` (alias) | `breakpointsVuetifyV2` or `breakpointsVuetifyV3` | v14.0.0 |
| `import { useAxios } from '@vueuse/integrations'` | `import { useAxios } from '@vueuse/integrations/useAxios'` | always |
| `require('@vueuse/core')` | `import { ... } from '@vueuse/core'` | v13.0.0 (ESM only) |
| `unref(input)` on `MaybeRefOrGetter` | `toValue(input)` | v9.0.0+ |

## v14 breaking-change quick list

When upgrading from v13 → v14 (or reviewing 14.x code), watch for:

- `computedAsync` default `flush` changed from `'pre'` to **`'sync'`**
- `useClipboard` return values are now wrapped in `readonly()` (type change)
- `useThrottleFn` switched to "traditional throttle" semantics (different trailing/leading)
- `useSwipe` no longer exposes `isPassiveEventSupported`
- `createSharedComposable` returns only the shared composable on client (return shape changed)
- `useUserMedia` constraints are now deep-watched
- `@vueuse/firebase` requires `firebase ^12`

Full breaking-change history (v12 / v13 / v14) is in [deprecations.md](deprecations.md).

## Most-reached-for composables (cheat sheet)

| Need | Composable | Package |
|---|---|---|
| Add a DOM event listener with auto-cleanup | `useEventListener` | core |
| Detect click outside an element | `onClickOutside` | core |
| Debounce / throttle a function | `useDebounceFn` / `useThrottleFn` | core |
| Debounce / throttle a ref | `refDebounced` / `refThrottled` | core |
| Run on an interval | `useIntervalFn` (`pause`/`resume`/`isActive`) | core |
| Persist to localStorage / sessionStorage | `useLocalStorage` / `useSessionStorage` | core |
| Persist to IndexedDB | `useIDBKeyval` | `integrations/useIDBKeyval` |
| Promise → `{ state, isLoading, error, execute }` | `useAsyncState` | core |
| REST call | `useFetch` (Vue) or Nuxt's `useFetch` (Nuxt) | core |
| HTTP via axios | `useAxios` | `integrations/useAxios` |
| WebSocket with reconnect + heartbeat | `useWebSocket` | core |
| Window size / focus / scroll | `useWindowSize` / `useWindowFocus` / `useWindowScroll` | core |
| Element size / bounding / visibility | `useElementSize` / `useElementBounding` / `useElementVisibility` | core |
| Observe scroll on an element | `useScroll` | core |
| Infinite scroll | `useInfiniteScroll` | core |
| Mouse / pointer / swipe | `useMouse` / `usePointer` / `useSwipe` | core |
| Media query / breakpoint preset | `useMediaQuery` / `useBreakpoints` + `breakpointsTailwind` etc. | core |
| Dark mode | `useDark` + `useToggle` (or `useColorMode` for tri-state) | core |
| Clipboard | `useClipboard` | core |
| Global / shared state | `createGlobalState` / `createSharedComposable` | core |
| Reactive ref history (undo/redo) | `useRefHistory` | core |
| Conditional `watch` (run once true) | `whenever` | core |
| Wait for a condition (Promise) | `until` | core |
| Run watcher N times max | `watchAtMost` | core |
| Route query / params / hash | `useRouteQuery` / `useRouteParams` / `useRouteHash` | `router` |
| Reactive math (sum / average / clamp) | `useSum` / `useAverage` / `useClamp` | `math` |

Full use-case → composable catalog (state, DOM events, sizing, mouse, media queries, network, timers, async, browser APIs, animation, reactivity helpers, watch family, array/math, time format, component utilities) is in [composables.md](composables.md).

## Common anti-patterns

**`unref` on a getter** — returns the function unchanged:

```ts
// ❌
function useFeat(input: MaybeRefOrGetter<number>) { return unref(input) * 2 }
// ✅
function useFeat(input: MaybeRefOrGetter<number>) { return toValue(input) * 2 }
```

**Destructure + snapshot** — loses reactivity:

```ts
// ❌
const { x, y } = useMouse()
const pos = { x: x.value, y: y.value }
// ✅
const { x, y } = useMouse()
watch([x, y], ([nx, ny]) => { /* ... */ })
```

**Passing reactive property by value** — snapshots a primitive:

```ts
// ❌
const state = reactive({ url: '/api' })
useFetch(state.url)
// ✅
useFetch(() => state.url)              // or toRef(state, 'url')
```

**Root import from integrations** — pulls every optional peer, breaks tree-shaking:

```ts
// ❌
import { useAxios } from '@vueuse/integrations'
// ✅
import { useAxios } from '@vueuse/integrations/useAxios'
```

**SSR-unsafe direct browser access** — crashes on the server:

```ts
// ❌
const width = window.innerWidth
// ✅
const { width } = useWindowSize()
```

**Hand-rolled debounce when core has it**:

```ts
// ❌
function debounce(fn, ms) { let t; return (...a) => { clearTimeout(t); t = setTimeout(() => fn(...a), ms) } }
// ✅
import { useDebounceFn } from '@vueuse/core'
const onInput = useDebounceFn(handler, 300)
```

**Calling a composable from an event handler** — wrong lifecycle, cleanup leaks:

```ts
// ❌
function onClick() { useEventListener('mousemove', handler) }
// ✅
const scope = effectScope()
function onClick() { scope.run(() => useEventListener('mousemove', handler)) }
// later: scope.stop()
```

**Deprecated `watchPausable`**:

```ts
// ❌
const { pause, resume } = watchPausable(src, cb)
// ✅ (Vue 3.5)
const { pause, resume, stop } = watch(src, cb)
```

**Deprecated `templateRef`**:

```ts
// ❌
const el = templateRef('my-el')
// ✅
const el = useTemplateRef('my-el')   // from 'vue'
```

**Deprecated `toValue` import**:

```ts
// ❌
import { toValue } from '@vueuse/shared'
// ✅
import { toValue } from 'vue'
```

**Nuxt `useFetch` collision** — VueUse's version skips Nuxt's SSR payload/streaming:

```ts
// ❌ (in Nuxt project)
import { useFetch } from '@vueuse/core'
// ✅
const { data } = await useFetch('/api')   // Nuxt auto-import
```

**`JSON.parse(useLocalStorage(...).value)`** — `useStorage` already serializes:

```ts
// ❌
const raw = useLocalStorage('k', '{}')
const obj = JSON.parse(raw.value)
// ✅
const obj = useLocalStorage('k', { foo: 'bar' })   // serializer picked from initial value
```

Full anti-pattern catalog (20 bad/good pairs) is in [deprecations.md](deprecations.md).

## Where to find more detail

- **[composables.md](composables.md)** — Use-case → composable catalog organized by category: state persistence, DOM events, sizing / visibility, mouse / pointer, media queries / breakpoints, network, timers, async state, browser APIs, animation, reactivity helpers, watch family, array, math, time formatting, component utilities, full `@vueuse/integrations` table.
- **[patterns.md](patterns.md)** — Required patterns: `MaybeRefOrGetter` + `toValue()`, SSR safety (`useSupported`, `isClient`, `useSSRWidth`), automatic vs. manual cleanup (`effectScope`, `tryOn*`), configurable options (`ConfigurableWindow` / `Document` / `Navigator` / `EventFilter`).
- **[nuxt.md](nuxt.md)** — Nuxt module setup, auto-import behavior, the 9 names excluded from auto-import (`toRef`, `toRefs`, `toValue`, `useFetch`, `useCookie`, `useHead`, `useTitle`, `useStorage`, `useImage`), SSR hydration mismatch avoidance.
- **[deprecations.md](deprecations.md)** — Breaking changes per major (v12 → v13 → v14), full anti-pattern catalog (20 bad/good pairs), upgrade workflow + checklist.

When this skill doesn't cover the exact composable, **search <https://vueuse.org/functions> or the GitHub releases at <https://github.com/vueuse/vueuse/releases>** before inventing names or rolling your own.
