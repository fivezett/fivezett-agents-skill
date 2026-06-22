# Required Patterns

The four patterns every VueUse composable user must internalize: `MaybeRefOrGetter` + `toValue()`, SSR safety, cleanup, configurable options. Plus `tryOn*` for lifecycle-outside calls.

---

## 1. `MaybeRefOrGetter<T>` + `toValue()`

Since VueUse 9.0, the recommended argument shape for any composable is **"accept a value, a ref, or a getter"**. Vue 3.3 made `toValue` official, and VueUse v12.3.0 deprecated its own re-export in favor of the `vue` native.

```ts
import { computed, type MaybeRefOrGetter, ref, toValue } from 'vue'

function useDouble(input: MaybeRefOrGetter<number>) {
  return computed(() => toValue(input) * 2)
}

useDouble(2)              // value
useDouble(ref(2))         // ref
useDouble(() => x.value)  // getter — tracks a reactive object's property
```

### `toValue` vs `unref`

| Input | `unref(x)` | `toValue(x)` |
|---|---|---|
| primitive | returns as-is | returns as-is |
| `Ref<T>` | unwraps `.value` | unwraps `.value` |
| getter `() => T` | **returns the function unchanged** | **calls and returns the result** |

Using `unref` where `toValue` is needed is the most common bug: the function gets passed downstream as a function, causing comparisons / multiplications / template renderings to fail silently.

### Import locations (current)

```ts
// ✅ current
import { toValue, type MaybeRef, type MaybeRefOrGetter } from 'vue'

// ❌ deprecated (still exported but flagged)
import { toValue } from '@vueuse/shared'                          // v12.3.0
import type { MaybeRef, MaybeRefOrGetter } from '@vueuse/shared'  // v12.8.0
```

### When to prefer a getter over a ref

When the source is a property of a reactive object, the ref form snapshots:

```ts
const state = reactive({ url: '/api' })

useFetch(state.url)         // ❌ string snapshot — loses reactivity
useFetch(() => state.url)   // ✅ tracks state.url
useFetch(toRef(state, 'url')) // ✅ alternative
```

---

## 2. SSR safety

VueUse composables are written with SSR in mind — they return sensible defaults on the server. Direct browser-API access outside a composable is what crashes.

### Patterns (in priority order)

1. **Use the composable.** Don't read `window.innerWidth` — use `useWindowSize()`. Don't read `navigator.onLine` — use `useOnline()`.
2. **`useSupported(() => /* check */)`** for capability detection:
   ```ts
   import { useSupported } from '@vueuse/core'
   const supported = useSupported(() => navigator && 'getBattery' in navigator)
   ```
3. **`isClient` guard** when you need a literal check:
   ```ts
   import { isClient } from '@vueuse/core'
   if (isClient) { /* browser-only code */ }
   ```
4. **`onMounted`** when you genuinely need a value the composable can't expose:
   ```ts
   onMounted(() => { /* now safe to touch window */ })
   ```

### Hydration mismatch composables

`useWindowSize`, `useMediaQuery`, `useBreakpoints` can hydrate with values that differ from what SSR rendered. Mitigate with **`useSSRWidth`** (introduced v12.1.0):

```ts
// In your app root, before any breakpoint composable runs
import { provideSSRWidth } from '@vueuse/core'
provideSSRWidth(1024, app)  // sets the server-assumed viewport width

// or per-component:
import { useSSRWidth } from '@vueuse/core'
useSSRWidth(1024)
```

### `ConfigurableWindow` / `ConfigurableDocument` / `ConfigurableNavigator`

Most browser-API composables accept `{ window? / document? / navigator? }` so you can substitute a different global (iframe parent, JSDOM mock, etc.):

```ts
const { x, y } = useMouse({ window: window.parent })
```

Use these when working with multi-windows, testing mocks, or non-default SSR shims. (Direct quote from the VueUse guidelines.)

---

## 3. Cleanup

> Similar to Vue's `watch` and `computed` that will be disposed when the component is unmounted, VueUse's functions also clean up the side-effects automatically. — VueUse best-practice guide

### Automatic (don't write `onUnmounted` for these)

`useEventListener`, `useIntervalFn`, `useTimeoutFn`, `useResizeObserver`, `useIntersectionObserver`, `useMutationObserver`, `useScroll`, `onClickOutside`, `useWebSocket`, `useEventSource`, `useMediaQuery`, `useFocus`, `useRafFn`, and nearly every other I/O-touching composable.

### Manual stop available

Many composables also return a stop handler if you want to dispose earlier than unmount:

```ts
const stop = useEventListener(window, 'scroll', handler)
// later
stop()
```

### Calling composables outside a component setup

If the call site isn't inside `setup()` / another composable (e.g. a Pinia store, a deep composable chain, a conditional branch), there's no component lifecycle to attach cleanup to. Two options:

**Option A: `effectScope`** — group multiple composables into one disposable unit:

```ts
import { effectScope } from 'vue'

const scope = effectScope()
scope.run(() => {
  useEventListener('mousemove', handler)
  onClickOutside(el, close)
})

// later — disposes everything at once
scope.stop()
```

**Option B: `tryOn*`** — safe lifecycle-hook registration:

```ts
import {
  tryOnMounted,
  tryOnBeforeMount,
  tryOnUnmounted,
  tryOnBeforeUnmount,
  tryOnScopeDispose,
} from '@vueuse/core'

tryOnMounted(() => {
  // runs onMounted if inside a component; otherwise no-op
})

tryOnScopeDispose(() => {
  // attaches to current effectScope if one exists; otherwise no-op
})
```

v14.0.0: `tryOnScopeDispose` accepts a `failSilently` option.

### Anti-pattern: composable in event handler

```ts
// ❌ — every click registers a new listener, cleanup never fires
function onClick() {
  useEventListener('mousemove', handler)
}

// ✅
const scope = effectScope()
function onClick() {
  scope.run(() => useEventListener('mousemove', handler))
}
// when done
scope.stop()
```

---

## 4. Configurable options (common interfaces)

| Interface | What it lets you swap | Typical use |
|---|---|---|
| `ConfigurableWindow` | `{ window?: Window }` — global window | iframe / parent window / JSDOM mock |
| `ConfigurableDocument` | `{ document?: Document }` | same idea for document |
| `ConfigurableNavigator` | `{ navigator?: Navigator }` | testing / shim |
| `ConfigurableEventFilter` | `{ eventFilter?: EventFilter }` | throttle / debounce a watcher or storage write |
| `ConfigurableFlush` | `{ flush?: 'pre' \| 'post' \| 'sync' }` | scheduling — important for `computedAsync` (v14 default `'sync'`) |

```ts
import { throttleFilter, useLocalStorage } from '@vueuse/core'

// Throttle localStorage writes to once per second
const store = useLocalStorage('k', {}, { eventFilter: throttleFilter(1000) })
```

---

## 5. `tryOn*` quick reference

| Helper | When to use |
|---|---|
| `tryOnMounted(fn)` | run on mount if inside a component, else no-op |
| `tryOnBeforeMount(fn)` | before mount, or no-op |
| `tryOnUnmounted(fn)` | on unmount, or no-op |
| `tryOnBeforeUnmount(fn)` | before unmount, or no-op |
| `tryOnScopeDispose(fn)` | when current `effectScope` disposes, or no-op (v14.0.0 adds `failSilently`) |

Use them from inside Pinia stores, composables conditionally called, library helpers — anywhere the call site might not be a component `setup()`.

---

## 6. Pattern checklist (before saving a composable)

- [ ] Inputs typed as `MaybeRefOrGetter<T>` where reactivity is useful
- [ ] `toValue()` (not `unref()`) used to read those inputs
- [ ] `toValue` / `MaybeRefOrGetter` imported from `'vue'` (not `@vueuse/shared`)
- [ ] No direct `window` / `document` / `navigator` access — `useSupported` / `isClient` / `onMounted` instead
- [ ] If using `ConfigurableWindow` etc., the option is honored everywhere the global is read
- [ ] Cleanup is either auto (VueUse covers it) or registered with `tryOnScopeDispose`
- [ ] No `useXxx` call inside an event handler / async callback — wrap in `effectScope` if conditionally needed
