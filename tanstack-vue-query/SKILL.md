---
name: tanstack-vue-query
description: Use when writing or reviewing Vue 3 code that imports from `@tanstack/vue-query`, about to use `useQuery` / `useMutation` / `useInfiniteQuery` / `useQueryClient` / `queryOptions` / `useSuspenseQuery`, or seeing v4 patterns like `onSuccess`/`onError`/`onSettled` in query options, `cacheTime`, `keepPreviousData`, `isPreviousData`, positional `useQuery(key, fn, opts)` signature, spread `{...queryOptions(), enabled}` pattern (TS2769), or missing `initialPageParam` in `useInfiniteQuery`. Targets `@tanstack/vue-query` v5.x, Vue 3.x.
---

# TanStack Vue Query v5 Reference

Reference for **`@tanstack/vue-query` v5.x** (pinned ^5.0.0). Vue 3 向けサーバー状態管理ライブラリ。v4 から多くの破壊的変更があり、v4 パターンのまま書くとランタイムエラーまたは型エラーになる。

- **Target**: `@tanstack/vue-query` ^5.0.0
- **Vue peer**: `^3.x`
- **Devtools**: `@tanstack/vue-query-devtools` (別パッケージ)
- **Last verified**: 2026-06-22

## When to Use

Apply whenever you are about to:

- `useQuery` / `useMutation` / `useInfiniteQuery` / `useQueryClient` を使う
- `queryOptions()` / `infiniteQueryOptions()` ファクトリを書く
- QueryClient メソッド (`invalidateQueries`, `setQueryData` 等) を呼ぶ
- Suspense 統合 (`useSuspenseQuery`) を使う
- 楽観的更新 (optimistic updates) を実装する
- VueQueryPlugin をセットアップする

**Concrete symptoms — STOP and check here:**

- `useQuery` のオプションに `onSuccess` / `onError` / `onSettled` を書こうとしている (v4 → 削除済み)
- `cacheTime` を書こうとしている (v4 → `gcTime` に改名)
- `keepPreviousData: true` を書こうとしている (v4 → `placeholderData: keepPreviousData`)
- `isPreviousData` を参照しようとしている (v4 → `isPlaceholderData`)
- `useQuery(queryKey, queryFn, options)` の位置引数形式で書こうとしている (v4 → 単一オブジェクト形式のみ)
- `useInfiniteQuery` で `initialPageParam` を省略しようとしている (v5 で必須)
- `{...queryOptions(), enabled: ...}` の spread + 追加プロパティパターンを書こうとしている (TS2769 バグ)
- `useSuspenseQuery` に `enabled` を渡そうとしている (v5 で未サポート)

## セットアップ

```ts
// main.ts
import { VueQueryPlugin } from '@tanstack/vue-query'
app.use(VueQueryPlugin)

// オプションカスタマイズ
app.use(VueQueryPlugin, {
  queryClientConfig: {
    defaultOptions: {
      queries: { staleTime: 1000 * 60 * 5, gcTime: 1000 * 60 * 10 },
    },
  },
})

// devtools (開発環境のみ)
// <VueQueryDevtools /> を App.vue に追加
import { VueQueryDevtools } from '@tanstack/vue-query-devtools'
```

## Hard Rules

1. **Never** `useQuery` / `useInfiniteQuery` のオプションに `onSuccess` / `onError` / `onSettled` を渡す。v5 で削除済み。副作用は `watch(query.data, ...)` / `watch(query.error, ...)` で代替。`useMutation` にはこれらが残存。
2. **Never** `cacheTime` を使う。v5 では `gcTime` に全面改名（`defaultOptions.queries.gcTime` も同様）。
3. **Never** `keepPreviousData: true` を使う。`placeholderData: keepPreviousData`（`@tanstack/vue-query` からインポート）に置き換え。`isPreviousData` → `isPlaceholderData`。
4. **Never** `useQuery(key, fn, options)` の位置引数形式。v5 は単一オブジェクト `useQuery({ queryKey, queryFn, ...options })` のみ。`useMutation` / `useInfiniteQuery` も同様。
5. **Never** `useInfiniteQuery` で `initialPageParam` を省略。v5 で必須（省略すると型エラー + ランタイム不整合）。
6. **Never** `{...queryOptions(), enabled: ..., staleTime: ...}` のように queryOptions() の結果を spread して追加プロパティをマージする。`MaybeRefDeep` 再帰展開でブランド型 (`dataTagSymbol`/`dataTagErrorSymbol`) が剥がれ **TS2769** が発生する既知バグ (issue #9920、未修正)。→ 明示的プロパティ転送で回避。
7. **Never** `useSuspenseQuery` に `enabled` を渡す。v5 では受け付けない（常に有効、data が `T | undefined` でなく `T` として推論される）。
8. **Never** `queryClient.getQueryData()` / `setQueryData()` の呼び出しで型パラメータを手書きする。`queryOptions()` ファクトリの `queryKey` 経由で自動推論させる。

## v4 → v5 破壊的変更クイックリファレンス

| v4 (削除/変更) | v5 (正しい書き方) |
|---|---|
| `useQuery(key, fn, opts)` 位置引数 | `useQuery({ queryKey, queryFn, ...opts })` 単一オブジェクト |
| `useQuery({ ..., onSuccess, onError, onSettled })` | 削除。`watch(query.data/error, ...)` で代替 |
| `cacheTime` | `gcTime` |
| `keepPreviousData: true` | `placeholderData: keepPreviousData` |
| `isPreviousData` | `isPlaceholderData` |
| `useInfiniteQuery({ getNextPageParam })` (initialPageParam 省略) | `useInfiniteQuery({ initialPageParam: 0, getNextPageParam })` |
| `getNextPageParam: (last, all) => ...` | `getNextPageParam: (last, all, lastParam) => ...` (第3引数追加) |
| `{...queryOptions(), enabled: ...}` spread | 明示的プロパティ転送（下記参照） |
| `isLoading` (fetch + no data) | `isPending && isFetching` で同等判定 |

## queryOptions() ファクトリ (TypeScript 推奨パターン)

型安全なキャッシュアクセスのため `queryOptions()` を使う。queryKey に `dataTagSymbol` ブランドを付与し、`queryClient.getQueryData(opts.queryKey)` で明示的型パラメータなしに型推論が効く。

```ts
// ✅ queryOptions ファクトリ定義
import { queryOptions } from '@tanstack/vue-query'

const userOptions = (userId: string) =>
  queryOptions({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 1000 * 60 * 5,
  })

// ✅ 使用側
const query = useQuery(userOptions(userId.value))

// ✅ QueryClient から型安全に取得
const cached = queryClient.getQueryData(userOptions('123').queryKey)
// ^? User | undefined  (型パラメータ不要)

// ✅ prefetch
await queryClient.prefetchQuery(userOptions('123'))
```

## TS2769 バグ回避 (spread + 追加プロパティ禁止)

```ts
// ❌ TS2769 — dataTagSymbol brand が剥がれる
const q = useQuery({ ...userOptions(id.value), enabled: !!id.value })

// ✅ 明示的プロパティ転送
const opts = userOptions(id.value)
const q = useQuery({
  queryKey: opts.queryKey,
  queryFn: opts.queryFn,
  staleTime: opts.staleTime,
  enabled: !!id.value,
})
```

## useQuery — 主要オプションと戻り値

```ts
const query = useQuery({
  queryKey: ['todos', todoId],   // 必須: キャッシュキー (Ref/computed 可)
  queryFn: ({ signal }) => fetchTodo(todoId.value, signal),  // 必須
  enabled: computed(() => !!todoId.value),  // false で自動フェッチ停止
  staleTime: 1000 * 60,         // ms: この時間はキャッシュを新鮮とみなす
  gcTime: 1000 * 60 * 5,        // ms: 非アクティブキャッシュの保持時間
  select: (data) => data.name,  // 変換後の値が data に入る
  placeholderData: keepPreviousData,  // ページネーション等でデータを維持
  initialData: () => getCachedTodo(), // 初期データ (stale 扱い)
  retry: 3,                     // リトライ回数
  refetchOnWindowFocus: true,   // ウィンドウフォーカス時にリフェッチ
})

// 戻り値 (すべて Ref)
query.data      // TData | undefined
query.error     // TError | null
query.isPending // キャッシュなし && フェッチ中
query.isFetching // フェッチ中 (バックグラウンド含む)
query.isSuccess
query.isError
query.isPlaceholderData  // placeholderData 表示中
query.refetch()
```

## useMutation — パターン

```ts
const mutation = useMutation({
  mutationFn: (newTodo: NewTodo) => createTodo(newTodo),
  onSuccess: (data, variables, context) => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
  onError: (error, variables, context) => { /* ロールバック */ },
  onSettled: () => { /* 成功/失敗問わず実行 */ },
})

mutation.mutate({ title: 'New Todo' })
mutation.mutateAsync({ title: 'New Todo' }).then(...)
mutation.isPending
mutation.isError
mutation.error
```

## useInfiniteQuery — v5 パターン

```ts
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['projects'],
  queryFn: ({ pageParam }) => fetchProjects(pageParam),  // pageParam: TPageParam
  initialPageParam: 0,  // v5 必須
  getNextPageParam: (lastPage, allPages, lastPageParam) =>
    lastPage.hasMore ? lastPageParam + 1 : undefined,
  getPreviousPageParam: (firstPage, allPages, firstPageParam) =>
    firstPageParam > 0 ? firstPageParam - 1 : undefined,
})

// data は InfiniteData<TData, TPageParam>
// data.value.pages: TData[]
// data.value.pageParams: TPageParam[]
```

## QueryClient メソッド

```ts
const queryClient = useQueryClient()

// キャッシュ無効化 (refetchType: 'active' がデフォルト)
queryClient.invalidateQueries({ queryKey: ['todos'] })
queryClient.invalidateQueries({ queryKey: ['todos'], refetchType: 'all' })

// 同期的キャッシュ更新 (存在しない場合は新規作成、デフォルト gcTime 5分)
queryClient.setQueryData(['todos', id], updatedTodo)
queryClient.setQueryData(['todos', id], (old) => ({ ...old, ...patch }))

// キャッシュ読み取り
const data = queryClient.getQueryData(['todos', id])

// バックグラウンドプリフェッチ (data も error も返さない)
await queryClient.prefetchQuery({ queryKey: ['todos', id], queryFn: fetchTodo })

// 進行中フェッチをキャンセル (楽観的更新前に呼ぶ)
await queryClient.cancelQueries({ queryKey: ['todos'] })

// キャッシュ削除
queryClient.removeQueries({ queryKey: ['todos'] })
```

## 楽観的更新 (Optimistic Updates)

### アプローチ 1: onMutate でキャッシュ操作

```ts
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // バックグラウンドリフェッチによる上書き防止 (cancelQueries 必須)
    await queryClient.cancelQueries({ queryKey: ['todos', newTodo.id] })
    const previousTodo = queryClient.getQueryData(['todos', newTodo.id])
    queryClient.setQueryData(['todos', newTodo.id], newTodo)  // 即時反映
    return { previousTodo }  // context としてロールバック用に渡す
  },
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(['todos', newTodo.id], context?.previousTodo)
  },
  onSettled: (data, error, variables) => {
    queryClient.invalidateQueries({ queryKey: ['todos', variables.id] })
  },
})
```

### アプローチ 2: variables + isPending (簡易・推奨)

```ts
// キャッシュを触らず UI 状態で楽観的表示
const mutation = useMutation({ mutationFn: addTodo })

// template 側で mutation.variables と mutation.isPending を使って仮表示
// mutation.isPending && mutation.variables で送信中アイテムを追加表示
```

## useSuspenseQuery

```ts
// enabled オプション非対応 (常に有効)
// data の型が T になる (T | undefined ではない)
// Suspense 境界 + ErrorBoundary が必要
const { data } = useSuspenseQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
})
// data: User (undefined なし)
```

## v4 コールバック削除の代替パターン

```ts
// ❌ v4 パターン (v5 では型エラー + 無視される)
useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
  onSuccess: (data) => toast.success(`${data.name} でログイン`),
  onError: (error) => toast.error(error.message),
})

// ✅ v5 パターン: watch で代替
const query = useQuery({ queryKey: ['user'], queryFn: fetchUser })
watch(query.data, (data) => {
  if (data) toast.success(`${data.name} でログイン`)
})
watch(query.error, (error) => {
  if (error) toast.error(error.message)
})
```

## queryKey 設計パターン (Query Key Factory)

```ts
// クエリキーを一元管理するファクトリ
export const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: string) => [...todoKeys.details(), id] as const,
}

// 使用例
useQuery({ queryKey: todoKeys.detail(id), queryFn: () => fetchTodo(id) })
queryClient.invalidateQueries({ queryKey: todoKeys.lists() })  // リスト系を全無効化
```

## Best Practices

### 依存クエリ (Dependent Queries)

```ts
const user = useQuery({ queryKey: ['user', email], queryFn: () => fetchUserByEmail(email.value) })
const userId = computed(() => user.data.value?.id ?? null)

const projects = useQuery({
  queryKey: computed(() => ['projects', userId.value]),
  queryFn: () => fetchProjectsByUserId(userId.value!),
  enabled: computed(() => userId.value !== null),
})
```

- `enabled` と `queryKey` を `computed` で明示し、前段が揃うまで後段を止める
- 注意: `enabled: false` 中は `isPending: true` + `fetchStatus: 'idle'` になる。ローディング判定に `fetchStatus !== 'idle'` も必要な場合がある
- ❌ `queryFn` の中で前クエリの `data` を直接参照して未取得時の分岐を散らす
- ✅ 依存値は `computed` で明示し `enabled` で実行条件を切る

### useQueries — 動的並列取得

```ts
const queries = computed(() =>
  userIds.value.map((id) => ({
    queryKey: ['messages', id],
    queryFn: () => fetchMessagesByUserId(id),
    staleTime: 30_000,
  })),
)
const messageQueries = useQueries({ queries })
```

- 動的な件数は `useQueries` に寄せる。`computed` で query 配列を作るのが Vue 3 での自然なパターン
- ❌ `v-for` の中で `useQuery()` を呼ぶ
- ✅ ID 配列から `queries` 配列を生成する

### composable として切り出す設計

```ts
// queries/user.ts — queryOptions をファクトリとして外出し
import { type MaybeRef, unref } from 'vue'
import { queryOptions, useQuery, useQueryClient } from '@tanstack/vue-query'

export const userQueryOptions = (userId: MaybeRef<number | null>) =>
  queryOptions({
    queryKey: ['user', userId],
    queryFn: () => fetchUserById(unref(userId)!),
    enabled: unref(userId) != null,
    staleTime: 60_000,
  })

export function useUserQuery(userId: MaybeRef<number | null>) {
  return useQuery(userQueryOptions(userId))
}

// 他の場所でも同じ定義を再利用できる
await queryClient.prefetchQuery(userQueryOptions(1))
queryClient.setQueryData(userQueryOptions(1).queryKey, updatedUser)
```

- `queryOptions()` を返すファクトリに切り出すと `useQuery` / `prefetchQuery` / `setQueryData` で同じ定義を再利用できる
- `queryKey` と `queryFn` を一箇所に寄せることでキー不整合を防ぐ
- ❌ spread で追加プロパティを足す (`{...userQueryOptions(id), enabled: !!id}` → TS2769)
- ✅ 追加条件が必要なら factory 側で `enabled` まで含めて返す

### staleTime 戦略

| データの性質 | staleTime |
|---|---|
| リアルタイム系 (チャット、株価) | `0` |
| ユーザープロフィール、マスタ | `30_000〜300_000` (30秒〜5分) |
| 重いレポート、読み取り中心 | `600_000+` (10分以上) |
| 手動 invalidate まで再取得不要 | `Infinity` |
| invalidation でも自動再取得しない | `'static'` |

```ts
// ❌ 全 query に同じ staleTime を機械的に入れる
// ✅ データの更新頻度と UX コストで判断する
// 補足: initialData 使用時は staleTime を併用しないと即再取得になる
```

`'static'` は `Infinity` より強く、`invalidateQueries` でも自動再取得されない。設定値や権限など実行中に変化しないデータ向け。

### グローバルエラーハンドリング

```ts
import { QueryCache, MutationCache, QueryClient } from '@tanstack/vue-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        if (error instanceof AuthError) return false  // 認証エラーはリトライしない
        return failureCount < 2
      },
      staleTime: 30_000,
    },
    mutations: { retry: 0 },
  },
  queryCache: new QueryCache({
    onError: (error, query) => {
      if (query.meta?.suppressGlobalError) return
      toast.error(getErrorMessage(error))
    },
  }),
  mutationCache: new MutationCache({
    onError: (error) => toast.error(getErrorMessage(error)),
  }),
})
```

- グローバル通知は各コンポーネントに散らさず `QueryCache` / `MutationCache` で一元化する
- ❌ `onError` を各画面で重複実装する
- ✅ `query.meta.suppressGlobalError` で個別抑制できるようにしておく

### select によるデータ変換

```ts
// 同じクエリを別々の select で再利用 — fetchTodos は1回しか呼ばれない
const activeTodos = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  select: (todos) => todos.filter((t) => !t.done),
})
const todoCount = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  select: (todos) => todos.length,
})
```

- `select` は純粋で軽い変換だけに使う。取得データ以外の reactive state を混ぜない
- 重い集計や画面ローカルの条件に依存する変換は `computed` 側に逃がす
- ❌ `select` の中で副作用 (store 更新、toast 等) を入れる

### enabled で条件付きフェッチ (ユーザーアクション後)

```ts
const draft = ref('')
const submitted = ref('')

const search = useQuery({
  queryKey: computed(() => ['search', submitted.value]),
  queryFn: () => searchArticles(submitted.value),
  enabled: computed(() => submitted.value.length > 0),
  staleTime: 15_000,
})

function onSearch() {
  submitted.value = draft.value.trim()
}
```

- 入力中の値 (`draft`) と実際にフェッチに使う値 (`submitted`) を分けることで queryKey が安定する
- ❌ 入力のたびに queryKey を変えて無駄にフェッチする
- ✅ `draft` と `submitted` を分ける。`enabled` はタイミングを制御するだけ

### mutation 後の invalidate vs setQueryData の使い分け

```ts
const updateTodo = useMutation({
  mutationFn: api.updateTodo,
  onSuccess: (updatedTodo) => {
    // 1件の詳細は即時更新
    queryClient.setQueryData(todoQueryOptions(updatedTodo.id).queryKey, updatedTodo)
    // 一覧・集計・フィルタなど広い影響範囲は invalidate
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

| 状況 | 推奨 |
|---|---|
| レスポンスが完全な更新後データ + 1件 | `setQueryData` |
| ソート条件・フィルタ・集計が絡む | `invalidateQueries` |
| 複数の関連クエリに影響する | `invalidateQueries` |
| サーバー側の副作用が読めない | `invalidateQueries` |

- ❌ サーバー側の副作用が読めないのに複数キャッシュを手動で書き換える
- ✅ 1件詳細は `setQueryData`、広い影響は `invalidateQueries`

## Common Anti-Patterns

**v4 コールバック (削除済み)**:
```ts
// ❌ useQuery の onSuccess/onError/onSettled は v5 で削除
useQuery({ queryKey: ['x'], queryFn: fn, onSuccess: cb })
// ✅ watch で代替 (上記参照)
```

**cacheTime (改名済み)**:
```ts
// ❌
useQuery({ queryKey: ['x'], queryFn: fn, cacheTime: 60000 })
// ✅
useQuery({ queryKey: ['x'], queryFn: fn, gcTime: 60000 })
```

**keepPreviousData (削除済み)**:
```ts
// ❌
const { data, isPreviousData } = useQuery({ ..., keepPreviousData: true })
// ✅
import { keepPreviousData } from '@tanstack/vue-query'
const { data, isPlaceholderData } = useQuery({ ..., placeholderData: keepPreviousData })
```

**initialPageParam 省略 (v5 必須)**:
```ts
// ❌
useInfiniteQuery({ queryKey: [...], queryFn: ({ pageParam = 0 }) => fetch(pageParam), getNextPageParam })
// ✅
useInfiniteQuery({ queryKey: [...], queryFn: ({ pageParam }) => fetch(pageParam), initialPageParam: 0, getNextPageParam })
```

**spread + 追加プロパティ (TS2769)**:
```ts
// ❌ dataTagSymbol が剥がれる
useQuery({ ...userOptions(id), enabled: !!id })
// ✅ 明示的プロパティ転送
const opts = userOptions(id); useQuery({ queryKey: opts.queryKey, queryFn: opts.queryFn, enabled: !!id })
```

**Reactive な queryKey に関数を渡さない**:
```ts
// ❌ id が変わっても queryKey が更新されない
const id = ref('1')
useQuery({ queryKey: ['user', id.value], queryFn: () => fetchUser(id.value) })
// ✅ ref/computed をそのまま渡す or computed で包む
useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id.value) })
// または
useQuery(computed(() => userOptions(id.value)))
```
