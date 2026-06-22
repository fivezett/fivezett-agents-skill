# Breaking Changes & Anti-pattern Catalog

Per-major breaking changes (v12 → v13 → v14), the full anti-pattern catalog (20 bad/good pairs), and the upgrade workflow.

---

## 1. v13 → v14 (released 2025-10-22)

| Category | Change | Action |
|---|---|---|
| Build | Migrated to tsdown (dist layout changed) | Rebuild — no source change |
| Vue peer | **Vue 3.5+ required** | Upgrade Vue first |
| Aliases | Old alias exports **deprecated** | Use the original function names |
| Components | Renderless components refactored to a consistent API (breaking) | Out of scope here |
| `computedAsync` | Default `flush` changed `'pre'` → **`'sync'`** | Audit `computedAsync` callsites — pass `{ flush: 'pre' }` if you relied on the old behavior |
| `createSharedComposable` | Returns **only the shared composable on client** (return shape changed) | Update destructuring |
| `useClipboard` | Return values wrapped in `readonly()` (type change) | Update consumer types |
| `useSwipe` | `isPassiveEventSupported` removed | Stop reading it |
| `useThrottleFn` | Switched to "traditional throttle" semantics (trailing/leading behavior changed) | Re-test throttled handlers |
| `firebase` | `firebase ^12` required | Upgrade firebase first |
| `nuxt` | Uses Nuxt v4 kit | Works on Nuxt 3.x and 4.x |
| `shared` | Some long-deprecated APIs fully removed | Replace explicitly |
| `watchPausable` | **Deprecated** | Use Vue 3.5's `watch()` return value `{ stop, pause, resume }` |
| `computedEager` | **Deprecated** | Vue 3.4+ `computed()` already skips invalidation when value unchanged |

### v14.x minor highlights

- **v14.0.0** (2025-10-22): `refManualReset` added; `useIdle` becomes a `Stoppable`; `useAsyncState` initial value can be a ref; `watchAtMost` gains `pause`/`resume`; `useTransition` supports custom interpolation
- **v14.1.0** (2025-11-27): `useElementVisibility.initialValue`; `useDropZone.checkValidity`; `useWebSocket.autoConnect.delay` callable; scheduler option introduced
- **v14.2.0** (2026-01-31): `useIntersectionObserver.rootMargin` reactive; `useSortable.watchElement`; `useDraggable` auto-scroll; `useFocusTrap` peer widened to `focus-trap ^7 || ^8`
- **v14.3.0** (2026-05-01): `useCssSupports.ssrValue`; common directive cleanup; `onClickOutside` Shadow-DOM-iframe detection; `useWakeLock` auto-release on unmount

---

## 2. v12 → v13 (released 2025-03-10)

| Category | Change | Action |
|---|---|---|
| Build | **CJS build removed — ESM only** | Use `import`, not `require` |
| Vue peer | Vue 3.3+ required | Upgrade Vue |
| v13.6.0 | `templateRef('key')` deprecated | Use Vue's `useTemplateRef('key')` |
| v13.6.0 | `useTemplateRefsList` deprecated | Use Vue 3.5 array refs |
| **v13.7.0** | `useAsyncState`: `onError` default changed to `globalThis.reportError` | Errors no longer silently swallowed — explicitly handle if you depended on silence |
| v13.x additions | `useTimeAgoIntl` (v13.7.0); `useStorageAsync.onReady` + Promise return (v13.6.0); `useRefHistory.shouldCommit` (v13.4.0); `useUrlSearchParams.stringify` (v13.4.0); `useEventSource.serializer` (v13.8.0); `useNow.immediate` (v13.3.0); `useClipboardItems.read()` (v13.7.0) | Opt-in |

---

## 3. v11 → v12 (released ~2024-12-02)

- **Vue 2 support fully ended** (no more `vue-demi`). For Vue 2, pin `@vueuse/core@11.x`.
- v12.3.0: `toValue` deprecated in `@vueuse/shared` — `import { toValue } from 'vue'` instead.
- v12.8.0: `MaybeRef` / `MaybeRefOrGetter` types deprecated in `@vueuse/shared` — import from `'vue'`.

---

## 4. Anti-pattern catalog (20 bad/good pairs)

### A1: Casing typos

```ts
// ❌
import { useLocalstorage } from '@vueuse/core'  // doesn't exist
// ✅
import { useLocalStorage } from '@vueuse/core'
```

### A2: Passing a ref to a child as `:prop`

```vue
<!-- ❌ — title is a Ref; child receives the Ref object, not the string -->
<MyChild :title="title" />
<!-- ✅ — pass the value -->
<MyChild :title="title.value" />
<!-- ✅ — or design the child to accept a ref/getter explicitly -->
```

### A3: Destructure-and-snapshot loses reactivity

```ts
// ❌
const { x, y } = useMouse()
const pos = { x: x.value, y: y.value }   // frozen snapshot
// ✅
const { x, y } = useMouse()
watch([x, y], ([nx, ny]) => { /* ... */ })
```

### A4: Composable called outside `setup`/composable/scope

```ts
// ❌ — cleanup never registers
function onClick() {
  useEventListener('mousemove', handler)
}
// ✅
const scope = effectScope()
function onClick() {
  scope.run(() => useEventListener('mousemove', handler))
}
// scope.stop() when done
```

### A5: SSR-unsafe direct browser access

```ts
// ❌ — crashes on the server
const width = window.innerWidth
// ✅
const { width } = useWindowSize()
```

### A6: Hand-rolled when core has it

```ts
// ❌
function debounce(fn, ms) { let t; return (...a) => { clearTimeout(t); t = setTimeout(() => fn(...a), ms) } }
// ✅
import { useDebounceFn } from '@vueuse/core'
const debounced = useDebounceFn(fn, 300)
```

### A7: Wrong package for a composable

```ts
// ❌
import { useAxios } from '@vueuse/core'        // not here
import { useRouteQuery } from '@vueuse/core'   // not here
// ✅
import { useAxios } from '@vueuse/integrations/useAxios'
import { useRouteQuery } from '@vueuse/router'
```

### A8: `.value` everywhere instead of `toValue`

```ts
// ❌ — crashes when `x` is a primitive
function double(x) { return x.value * 2 }
// ✅ — handles value / ref / getter
function double(x) { return toValue(x) * 2 }
```

### A9: `unref()` on a getter

```ts
// ❌ — when input is a getter function, `v` IS the function
function useFeat(input: MaybeRefOrGetter<number>) {
  const v = unref(input)
}
// ✅
function useFeat(input: MaybeRefOrGetter<number>) {
  const v = toValue(input)  // invokes the getter
}
```

### A10: `toValue` from `@vueuse/shared`

```ts
// ❌ (deprecated v12.3.0)
import { toValue } from '@vueuse/shared'
// ✅
import { toValue } from 'vue'
```

### A11: VueUse `useFetch` / `useCookie` in Nuxt

```ts
// ❌ — skips Nuxt's SSR payload / streaming / dedupe
import { useFetch } from '@vueuse/core'
const { data } = useFetch('/api')
// ✅ — Nuxt's auto-imported useFetch
const { data } = await useFetch('/api')
```

### A12: `templateRef` in new code

```ts
// ❌ (deprecated v13.6.0)
const el = templateRef('my-el')
// ✅ — Vue native
const el = useTemplateRef('my-el')
```

### A13: `watchPausable`

```ts
// ❌ (deprecated v14.0.0)
const { pause, resume } = watchPausable(src, cb)
// ✅ — Vue 3.5 native
const { pause, resume, stop } = watch(src, cb)
```

### A14: `computedEager`

```ts
// ❌ (deprecated v14.0.0)
const v = computedEager(() => isFoo.value)
// ✅ — Vue 3.4+ computed suffices
const v = computed(() => isFoo.value)
```

### A15: `breakpointsVuetify` alias

```ts
// ❌ — ambiguous, deprecated
import { breakpointsVuetify } from '@vueuse/core'
// ✅ — explicit
import { breakpointsVuetifyV3 } from '@vueuse/core'
```

### A16: `JSON.parse(useLocalStorage(...).value)`

```ts
// ❌ — useStorage already serializes
const raw = useLocalStorage('k', '{}')
const obj = JSON.parse(raw.value)
// ✅ — pass the default object directly; serializer chosen by type
const obj = useLocalStorage('k', { foo: 'bar' })
```

### A17: Root import from integrations

```ts
// ❌ — disables tree-shaking, pulls every optional peer
import { useAxios } from '@vueuse/integrations'
// ✅
import { useAxios } from '@vueuse/integrations/useAxios'
```

### A18: Reactive property snapshotted by value

```ts
// ❌ — string snapshot, no reactivity
const state = reactive({ url: '/api' })
useFetch(state.url)
// ✅
useFetch(() => state.url)
// or
useFetch(toRef(state, 'url'))
```

### A19: `ref('.foo')` in an ignore list

```ts
// ❌ — wrapping a static selector string in a ref is meaningless
onClickOutside(target, cb, { ignore: [ref('.foo')] })
// ✅
onClickOutside(target, cb, { ignore: ['.foo', otherRef] })
```

### A20: CJS `require`

```ts
// ❌ (ESM only since v13.0.0)
const { useMouse } = require('@vueuse/core')
// ✅
import { useMouse } from '@vueuse/core'
```

---

## 5. Upgrade workflow

When a new VueUse major (or minor with breaking notes) lands:

```bash
# 1. Current latest
npm view @vueuse/core version

# 2. Releases since your pinned version
gh release list -R vueuse/vueuse --limit 20

# 3. Extract breaking changes (🚨 marker)
gh release view vX.Y.Z -R vueuse/vueuse | grep -A 3 '🚨'

# 4. Per-composable history — Source link in the docs
#    https://vueuse.org/core/useXxx/

# 5. Peer dep range changes
npm view @vueuse/core peerDependencies
npm view @vueuse/integrations peerDependencies peerDependenciesMeta --json
```

### Checklist

- [ ] Build format change? (e.g. CJS dropped in v13.0.0)
- [ ] Vue peer raised? (e.g. Vue 3.5+ in v14.0.0)
- [ ] All 🚨 sections reviewed and reflected in the use-case catalog
- [ ] New deprecations added to anti-pattern catalog
- [ ] New composables slotted into the catalog
- [ ] Nuxt auto-import exclusion list checked for changes
- [ ] `@vueuse/integrations` peer ranges still match what we install

### Skill maintenance

When a new VueUse major ships:

1. Update `last_verified` and `last_changelog_check` in the skill body
2. Append breaking changes to §1 / §2 / §3 of this file
3. Remove sunset composables from `composables.md`
4. Add new anti-patterns to §4 here
5. Update the **Most-reached-for composables** cheat sheet in `SKILL.md` if needed

Do **not** pin VueUse to a range in code — the release cadence is too fast and the API surface is too large to maintain compatibility shims. Just stay current.

---

## 6. Source links

**Official docs (primary):**

- VueUse docs: https://vueuse.org/
- All functions: https://vueuse.org/functions
- Best practice: https://vueuse.org/guide/best-practice
- Config: https://vueuse.org/guide/config
- Integrations README: https://vueuse.org/integrations/readme
- Nuxt README: https://vueuse.org/nuxt/readme.html
- Router README: https://vueuse.org/router/readme
- Guidelines: https://vueuse.org/guidelines

**GitHub Releases (primary for breaking changes):**

- v14.0.0: https://github.com/vueuse/vueuse/releases/tag/v14.0.0
- v14.1.0: https://github.com/vueuse/vueuse/releases/tag/v14.1.0
- v14.3.0: https://github.com/vueuse/vueuse/releases/tag/v14.3.0
- v13.0.0: https://github.com/vueuse/vueuse/releases/tag/v13.0.0
- v12.0.0: https://github.com/vueuse/vueuse/releases/tag/v12.0.0
- All: https://github.com/vueuse/vueuse/releases

**npm:**

- `@vueuse/core`: https://www.npmjs.com/package/@vueuse/core
- `@vueuse/nuxt`: https://www.npmjs.com/package/@vueuse/nuxt
- `@vueuse/integrations`: https://www.npmjs.com/package/@vueuse/integrations
- `@vueuse/router`: https://www.npmjs.com/package/@vueuse/router

**Vue native:**

- Composables guide (`toValue` / `MaybeRefOrGetter`): https://vuejs.org/guide/reusability/composables
- Vue release policy: https://vuejs.org/about/releases
