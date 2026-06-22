# Use-case → Composable Catalog

Reference of `@vueuse/core` 14.3.x composables organized by what you want to do. Components and directives from `@vueuse/components` are intentionally excluded.

Tags: `(introduced in vX.Y)` for recent additions; `(deprecated in vX.Y)` for sunset APIs. No tag = stable since v12 or earlier.

---

## 1. State persistence

### Persist a value to localStorage and restore on reload — `useLocalStorage`

```ts
import { useLocalStorage } from '@vueuse/core'

const settings = useLocalStorage('app-settings', { theme: 'light', lang: 'ja' })
settings.value.theme = 'dark'   // auto-writes
```

SSR safe (returns the default server-side). Auto-cleanup. Serializer picked from the initial value's type.

### sessionStorage version — `useSessionStorage`

```ts
const draft = useSessionStorage('draft', '')
```

### Arbitrary storage backend / async storage — `useStorage` / `useStorageAsync`

`useStorageAsync` gained `onReady` callback + Promise return in v13.6.0.

```ts
import { useStorageAsync } from '@vueuse/core'

const value = useStorageAsync('k', 'default', myAsyncStorage, {
  onReady: v => console.log('hydrated', v),
})
```

### IndexedDB key-value — `useIDBKeyval`

```ts
import { useIDBKeyval } from '@vueuse/integrations/useIDBKeyval'

const count = useIDBKeyval('count', 0)
```

### Cookies — `useCookies` (integrations)

```ts
import { useCookies } from '@vueuse/integrations/useCookies'

const cookies = useCookies(['locale'])
cookies.set('locale', 'ja')
```

In Nuxt prefer Nuxt's `useCookie` (VueUse's `useCookie` is excluded from Nuxt auto-import).

### Undo / redo history — `useRefHistory` family

```ts
const text = ref('')
const { history, undo, redo } = useRefHistory(text)
```

Variants: `useManualRefHistory`, `useDebouncedRefHistory`, `useThrottledRefHistory`. `shouldCommit` option (introduced in v13.4.0) lets you skip noise.

### Track last modified time — `useLastChanged`

### Global / shared state — `createGlobalState` / `createSharedComposable` / `createInjectionState`

```ts
import { createGlobalState, useStorage } from '@vueuse/core'

export const useGlobalAuth = createGlobalState(() =>
  useStorage('auth', { token: '' })
)
```

v14.0.0: `createSharedComposable` now returns only the shared composable on the client (return shape changed).

### Extended provide/inject — `provideLocal` / `injectLocal`

### Promise state (`{ state, isLoading, error, execute }`) — `useAsyncState`

```ts
const { state, isReady, isLoading, error, execute } = useAsyncState(
  () => fetch('/api').then(r => r.json()),
  null,
)
```

v14.0.0: initial value can now be a ref. v13.7.0: `onError` now defaults to `globalThis.reportError` — no more silent swallow.

---

## 2. DOM events

### Auto-cleanup event listener — `useEventListener`

```ts
useEventListener(window, 'scroll', () => {})
useEventListener(el, 'click', e => {}, { passive: true })
```

### Click outside an element — `onClickOutside`

```ts
const modal = useTemplateRef('modal')
onClickOutside(modal, close, { ignore: ['.dropdown'] })
```

v14.0.0: target accepts a getter. v14.3.0: Shadow-DOM-internal iframe detection option.

### Element removed from DOM — `onElementRemoval`

### Single-key keystroke — `onKeyStroke`

```ts
onKeyStroke('Enter', e => submit())
onKeyStroke(['ArrowUp', 'ArrowDown'], handleArrow)
```

### Chord / combo keys — `useMagicKeys`

```ts
const keys = useMagicKeys()
whenever(keys['Meta+K'], openCommandPalette)
```

### Long press — `onLongPress`

v14.0.0: `delay` accepts a function.

### Detect user starts typing into a form — `onStartTyping`

---

## 3. Element size / position / visibility

| Composable | What it does |
|---|---|
| `useElementSize` | `ResizeObserver`-based size |
| `useElementBounding` | reactive `getBoundingClientRect()` |
| `useElementVisibility` | viewport intersection (v14.1.0 `initialValue`, v14.2.0 `rootMargin` reactive) |
| `useIntersectionObserver` | low-level IntersectionObserver wrapper (v14.2.0 `rootMargin` reactive) |
| `useMutationObserver` | DOM mutation observer wrapper |
| `useResizeObserver` | low-level ResizeObserver wrapper |
| `useParentElement` / `useCurrentElement` | parent / current `$el` |
| `useScroll` / `useWindowScroll` | scroll position + direction |
| `useInfiniteScroll` | infinite scroll (v14.1.0 added `flush: 'post'`) |
| `useVirtualList` | virtual scrolling |
| `useScrollLock` | lock scroll (modals) |
| `useDraggable` | draggable element (v14.2.0 auto-scroll + container-restricted) |
| `useDropZone` | drop zone (v14.1.0 `checkValidity`) |
| `useDocumentVisibility` | document visible/hidden |
| `useWindowSize` / `useWindowFocus` | window dims + focus |
| `useActiveElement` | currently focused element |
| `useElementHover` | hover state ref |

---

## 4. Mouse / touch / pointer

| Composable | What it does |
|---|---|
| `useMouse` | mouse coords (page / client / screen) |
| `useMouseInElement` | element-relative coords |
| `useMousePressed` | pressed state |
| `usePointer` / `usePointerLock` / `usePointerSwipe` | generic pointer |
| `useSwipe` | swipe direction + delta (v14.0.0 dropped `isPassiveEventSupported`) |
| `usePageLeave` | mouse left page detection |
| `useParallax` | parallax effect |
| `useElementByPoint` | element at (x, y) |

---

## 5. Media queries / breakpoints / preferences

### Single media query — `useMediaQuery`

```ts
const isMobile = useMediaQuery('(max-width: 768px)')
```

### Preset breakpoints — `useBreakpoints` + preset

```ts
import { breakpointsTailwind, useBreakpoints } from '@vueuse/core'

const bp = useBreakpoints(breakpointsTailwind)
const isLg = bp.greaterOrEqual('lg')
```

Available presets: `breakpointsTailwind`, `breakpointsBootstrapV5`, `breakpointsVuetifyV2`, `breakpointsVuetifyV3`, `breakpointsElement`, `breakpointsAntDesign`, `breakpointsQuasar`, `breakpointsSematic`, `breakpointsMasterCss`, `breakpointsPrimeFlex`.

- Presets are not auto-imported (explicit import required for tree-shaking).
- `breakpointsVuetify` is a deprecated alias — pick V2 or V3 explicitly.
- For SSR, use `useSSRWidth` (v12.1.0+) to set the server-side width.

### User preferences

- `usePreferredDark`, `usePreferredColorScheme`, `usePreferredContrast`
- `usePreferredReducedMotion`, `usePreferredReducedTransparency`
- `usePreferredLanguages`

### Dark mode toggle — `useDark` + `useToggle`

```ts
const isDark = useDark()
const toggleDark = useToggle(isDark)
```

### Tri-state color mode (light/dark/auto/custom) — `useColorMode`

---

## 6. Network

### REST (`fetch` wrapper) — `useFetch`

```ts
const { data, error, isFetching, execute } = useFetch('/api/posts').get().json()
```

In Nuxt prefer Nuxt's `useFetch` (VueUse's is excluded from auto-import).

### axios — `useAxios` (integrations)

```ts
import { useAxios } from '@vueuse/integrations/useAxios'

const { data, isFinished } = await useAxios('/api/posts')
```

### WebSocket — `useWebSocket`

```ts
const { status, data, send, open, close } = useWebSocket('wss://...', {
  autoReconnect: true,
  heartbeat: { interval: 30_000, message: 'ping' },
})
```

v14.1.0: `autoConnect.delay` accepts a function. v14.3.0: open/close race condition fixed.

### Server-Sent Events — `useEventSource` (v13.8.0 `serializer` option)

### Online / network info — `useOnline` / `useNetwork`

---

## 7. Timers & scheduling

| Composable | What it does |
|---|---|
| `useDebounceFn` | debounce a function |
| `useThrottleFn` | throttle a function (**v14.0.0** changed to "traditional throttle" semantics) |
| `refDebounced` / `refThrottled` | debounced / throttled ref |
| `useIntervalFn` | recurring callback with `pause`/`resume`/`isActive` |
| `useTimeoutFn` / `useTimeout` | one-shot timeout |
| `useCountdown` | countdown timer |
| `useRafFn` | `requestAnimationFrame` loop |
| `useNow` / `useTimestamp` / `useInterval` | reactive clock (v14.1.0 scheduler option) |
| `useTimeoutPoll` | poll, waiting for previous run to complete |

---

## 8. Async state helpers

| Composable | What it does |
|---|---|
| `useAsyncQueue` | parallel queue of Promises |
| `useConfirmDialog` | reveal → wait → confirm/cancel flow |
| `useEventBus` | typed inter-component event bus |
| `useMemoize` | memoize a function's results |
| `useOffsetPagination` | pagination state |
| `usePrevious` | hold previous value |
| `useStepper` | wizard / stepper state |
| `useCycleList` | cyclable list |
| `useCounter` | counter with `inc` / `dec` / `set` / `reset` |

---

## 9. Browser APIs

### Clipboard — `useClipboard` / `useClipboardItems`

```ts
const { copy, copied, text, isSupported } = useClipboard()
```

HTTPS required. v14.0.0: return values wrapped in `readonly()` (type change). v13.7.0: `useClipboardItems` exposes `read()`.

### Misc browser APIs

- `useShare` (Web Share)
- `useWakeLock` (screen wake; v14.3.0 auto-release on unmount)
- `usePermission`
- `useBattery`
- `useGeolocation`
- `useWebNotification`
- `useDisplayMedia` (screen share)
- `useUserMedia` (camera/mic; v14.0.0 deep-watches constraints)
- `useDevicesList`
- `useScreenOrientation`
- `useScreenSafeArea`
- `useFullscreen`
- `useTitle` / `useFavicon` / `useTextDirection`
- `useScriptTag` (dynamic `<script>` injection)
- `useStyleTag` (dynamic `<style>` injection)
- `useTextareaAutosize`
- `useFileDialog` (v13.6.0 attributes accept MaybeRef)
- `useFileSystemAccess`
- `useBroadcastChannel`
- `useBluetooth`
- `useGamepad`
- `useEyeDropper`
- `useImage` (collides with Nuxt's `useImage`)
- `useObjectUrl`
- `usePerformanceObserver`
- `useMemory`
- `useDevicePixelRatio`
- `useCssVar` / `useCssSupports` (v14.3.0 SSR `ssrValue`)
- `useVibrate`
- `useSpeechRecognition` / `useSpeechSynthesis`
- `useMediaControls` (video/audio)
- `useUrlSearchParams` (v14.0.0 history/navigation fix; v13.4.0 `stringify` option)
- `useBrowserLocation`
- `useTextSelection`
- `useFps`
- `useDeviceMotion` / `useDeviceOrientation`
- `useIdle` (v14.0.0 becomes a `Stoppable`)
- `useWebWorker` / `useWebWorkerFn`

---

## 10. Animation / transitions

### Value tween — `useTransition`

```ts
const baseValue = ref(0)
const tweened = useTransition(baseValue, {
  duration: 500,
  transition: TransitionPresets.easeInOutCubic,
})
```

v14.0.0: custom interpolation function supported. `TransitionPresets` ships many easings.

### Web Animations API — `useAnimate`

### Spring-style motion — out of scope (use `@vueuse/motion` package separately)

---

## 11. Reactivity helpers (ref extensions)

| Composable | What it does |
|---|---|
| `computedAsync` | async computed (**v14.0.0** default `flush` changed `'pre'` → `'sync'`) |
| `computedEager` | **deprecated v14.0.0** — Vue 3.4+ `computed()` suffices |
| `computedWithControl` | manually-triggered computed |
| `computedInject` | computed + inject |
| `extendRef` | attach methods/props to a ref |
| `reactify` / `reactifyObject` | turn value functions into ref-aware ones |
| `reactiveComputed` / `reactivePick` / `reactiveOmit` | reactive-object ops |
| `refAutoReset` | ref that returns to a default after N ms |
| `refManualReset` | **introduced v14.0.0** — ref with manual reset |
| `refDefault` | ref with default fallback |
| `refWithControl` | ref with get/set hooks |
| `syncRef` / `syncRefs` | one/two-way ref syncing |
| `toReactive` | ref → reactive object |
| `createRef` | unified ref factory |
| `get` / `set` / `toRef` / `toRefs` | thin getter/setter wrappers |

---

## 12. Watch family

| Composable | What it does |
|---|---|
| `until` | Promise that resolves when condition true |
| `whenever` | run only when source is truthy |
| `watchOnce` | fire once then stop |
| `watchImmediate` | `immediate: true` watch |
| `watchDeep` | `deep: true` watch |
| `watchArray` | distinguish add / delete |
| `watchAtMost` | cap firings to N (v14.0.0 added `pause` / `resume`) |
| `watchDebounced` | debounce a watch |
| `watchThrottled` | throttle a watch |
| `watchIgnorable` | ignore programmatic mutations |
| `watchPausable` | **deprecated v14.0.0** — use Vue 3.5's `watch()` return value |
| `watchTriggerable` | manually re-evaluate |
| `watchWithFilter` | apply an `EventFilter` |

---

## 13. Array helpers

`useArrayMap`, `useArrayFilter`, `useArrayReduce`, `useArrayFind`, `useArrayFindIndex`, `useArrayFindLast`, `useArrayEvery`, `useArraySome`, `useArrayIncludes`, `useArrayUnique`, `useArrayDifference`, `useArrayJoin`, `useSorted`

---

## 14. Math (`@vueuse/math`)

`useSum`, `useAverage`, `useMin`, `useMax`, `useAbs`, `useCeil`, `useFloor`, `useRound`, `useTrunc`, `useClamp`, `usePrecision`, `useMath`, `useProjection`, `createProjection`, `createGenericProjection`

Logic gates: `logicAnd`, `logicOr`, `logicNot`

---

## 15. Time formatting

- `useDateFormat` (custom format strings)
- `useTimeAgo` ("5 minutes ago" style)
- `useTimeAgoIntl` (**introduced v13.7.0**, `Intl.RelativeTimeFormat`-based)
- `useCountdown`

---

## 16. Component utilities

| Composable | What it does |
|---|---|
| `createReusableTemplate` / `createTemplatePromise` | reuse template within one file |
| `templateRef` | **deprecated v13.6.0** — use Vue's `useTemplateRef()` |
| `useTemplateRefsList` | **deprecated v13.6.0** |
| `tryOnMounted` / `tryOnBeforeMount` / `tryOnUnmounted` / `tryOnBeforeUnmount` / `tryOnScopeDispose` | safe lifecycle hooks outside `setup()` |
| `unrefElement` | unwrap ref → DOM node (handles Component instances by returning `$el`) |
| `useCurrentElement` | current component's `$el` |
| `useMounted` | boolean ref of mounted state |
| `useVirtualList` | virtual scrolling |
| `useVModel` / `useVModels` | treat `v-model` as a ref |

---

## 17. Utilities

- `useBase64` — File / Blob / string / ref → Base64
- `useCached` — emit only on equality change (v14.3.0 comparator type improved)
- `useCloned` — reactive deep clone
- `useToggle` — boolean toggle
- `useToNumber` / `useToString` — reactive type conversion
- `useSupported` — generic feature-support detection
- `createDisposableDirective` — directive with dispose
- `createEventHook` — typed event hook
- `createUnrefFn` — auto-unref function arguments
- `isDefined` — null/undefined type-guard ref
- `makeDestructurable` — object + array destructure dual API

---

## 18. `@vueuse/integrations` — full table

**Always subpath import.** Peer deps are optional except `vue` — install only what you use.

| Composable | Purpose | Peer | Subpath |
|---|---|---|---|
| `useAsyncValidator` | form validation | `async-validator` | `/useAsyncValidator` |
| `useAxios` | HTTP | `axios` | `/useAxios` |
| `useChangeCase` | string case conversion | `change-case` | `/useChangeCase` |
| `useCookies` | cookies | `universal-cookie` | `/useCookies` |
| `useDrauu` | whiteboard drawing | `drauu` | `/useDrauu` |
| `useFocusTrap` | focus trap | `focus-trap ^7 \|\| ^8` (v14.2.0 added `^8`) | `/useFocusTrap` |
| `useFuse` | fuzzy search | `fuse.js` | `/useFuse` |
| `useIDBKeyval` | IndexedDB | `idb-keyval` | `/useIDBKeyval` |
| `useJwt` | JWT decode | `jwt-decode` | `/useJwt` |
| `useNProgress` | progress bar | `nprogress` | `/useNProgress` |
| `useQRCode` | QR code generation | `qrcode` | `/useQRCode` |
| `useSortable` | DnD sorting | `sortablejs` | `/useSortable` (v14.2.0 `watchElement`) |

```ts
// ✅
import { useAxios } from '@vueuse/integrations/useAxios'

// ❌ — disables tree-shaking, pulls every peer
import { useAxios } from '@vueuse/integrations'
```

To verify exact peer ranges for the pinned version:

```bash
npm view @vueuse/integrations@14.3.0 peerDependencies peerDependenciesMeta --json
```

---

## 19. Other packages (brief)

| Package | Composables | Peer |
|---|---|---|
| `@vueuse/router` | `useRouteQuery`, `useRouteParams`, `useRouteHash` | `vue-router@4 \|\| @5` |
| `@vueuse/rxjs` | `from`, `useObservable`, `useSubject`, `useSubscription`, `useExtractedObservable`, `watchExtractedObservable`, `toObserver` | `rxjs` |
| `@vueuse/firebase` | `useAuth`, `useFirestore`, `useRTDB` | `firebase ^12` (v14.0.0+) |
| `@vueuse/electron` | `useIpcRenderer*`, `useZoomFactor`, `useZoomLevel` | `electron` |
| `@vueuse/math` | reactive math (see §14) | — |
| `@vueuse/nuxt` | auto-import module | `nuxt 3.x \|\| 4.x` |
| `@vueuse/components` | renderless components + directives — **out of scope** | — |
| `@vueuse/head` | sunset — use `@unhead/vue` instead | — |
