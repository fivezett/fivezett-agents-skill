---
name: zod
description: Use when writing or reviewing TypeScript code that imports from `zod`, about to write `import z from 'zod'` or `import { z } from 'zod'` (broken in v4 — activates Zod v3 classic), `z.string().email()` / `.uuid()` / `.url()` / `.datetime()` (deprecated in v4), `{ message: "..." }` on validators (deprecated: use `error`), `errorMap` (deprecated), or any schema definition / parse call using Zod. Targets `zod` ^4.0.0.
---

# Zod v4 Reference

Reference for **Zod v4.x** (pinned 4.0.1). Make every schema definition use the correct import pattern and v4 APIs — no v3-era fallbacks.

- **Target**: zod ^4.0.0
- **Last verified**: 2026-06-22
- **Docs**: https://zod.dev
- **Changelog**: https://zod.dev/v4/changelog

## Critical: Import Pattern

**This is the #1 failure mode.** Wrong imports silently activate Zod v3-era APIs.

```ts
// ❌ Default import — activates Zod v3 classic (broken in v4)
import z from 'zod'

// ❌ Named import — activates Zod v3 classic (broken in v4)
import { z } from 'zod'

// ❌ Named import of any symbol — forbidden
import { ZodSchema, ZodError } from 'zod'

// ✅ Namespace import — the only correct form
import * as z from 'zod'

// ✅ Type-only namespace import
import type * as z from 'zod'
```

**Why**: Zod v4 restructured its exports. Default/named imports resolve to the v3 compat shim, pulling in deprecated APIs even when `zod@4` is installed. The namespace import is the canonical v4 entry point.

Sub-package variants:
```ts
import * as z from 'zod'         // full OOP API (recommended)
import * as z from 'zod/mini'    // functional, tree-shakable API
import * as z from 'zod/v4/core' // bare core types (library authors only)
```

## When to Use

Apply whenever you are about to:

- Define a Zod schema (`z.object`, `z.string`, `z.number`, etc.)
- Validate data with `.parse()` / `.safeParse()`
- Extract TypeScript types from schemas (`z.infer<typeof schema>`)
- Configure error messages on validators
- Review or refactor code that imports from `zod`

**Concrete symptoms — STOP and check here:**

- About to write `import z from 'zod'` or `import { z } from 'zod'`
- About to write `z.string().email()`, `.uuid()`, `.url()`, `.datetime()`, `.date()`, `.time()`, `.duration()`, `.ip()`, `.emoji()`, `.base64()`, `.nanoid()`, `.cuid()`, `.cuid2()`, `.ulid()`
- About to write `{ message: "..." }` as the error param on any validator
- About to use `errorMap` on a schema definition
- About to assume `z.coerce.X()` has a typed input (it's `unknown` in v4)
- About to import types like `ZodSchema`, `ZodError`, `ZodIssue` with a named import

**When NOT to use:** Project is pinned to `"zod": "^3.x"`. Import rules and APIs differ in v3.

## Why This Matters

Three failure modes to actively prevent:

1. **Wrong import activates v3 shim** — `import z from 'zod'` or `import { z } from 'zod'` silently loads Zod v3 behavior. No runtime error — just deprecated APIs and potentially wrong TypeScript types.
2. **v3 string method validators still work but are deprecated** — `z.string().email()` / `.uuid()` pass silently but should be `z.email()` / `z.uuid()` for tree-shaking and v4 idiom.
3. **Wrong error param** — `{ message: "Too short" }` is deprecated. Use `{ error: "Too short" }` or the string shorthand `"Too short"` directly.

When in doubt about a v4 API: **check https://zod.dev or grep the codebase. Do not fall back to v3 patterns.**

## Hard Rules

1. **Never** `import z from 'zod'`. Use `import * as z from 'zod'`.
2. **Never** `import { z } from 'zod'`. Use `import * as z from 'zod'`.
3. **Never** named-import any Zod symbol: `import { ZodSchema, ZodError } from 'zod'`. All types live under the namespace: `z.ZodSchema`, `z.ZodError`, `z.ZodIssue`.
4. **Never** use deprecated string method validators in new code: `.email()`, `.uuid()`, `.url()`, `.emoji()`, `.datetime()`, `.date()`, `.time()`, `.duration()`, `.ip()`, `.base64()`, `.nanoid()`, `.cuid()`, `.cuid2()`, `.ulid()`. Use top-level validators.
5. **Never** use `{ message: "..." }` on validators. Use `{ error: "..." }` or a string directly: `.min(5, "Too short")`.
6. **Never** use `errorMap`. Use the `error` function: `z.string({ error: (issue) => "..." })`.
7. **Never** assume `z.coerce.X()` input is typed. In v4 all coerce schemas have input type `unknown`.

## v3 → v4 Quick Conversion

| v3 (deprecated/broken) | v4 (correct) |
|---|---|
| `import z from 'zod'` | `import * as z from 'zod'` |
| `import { z } from 'zod'` | `import * as z from 'zod'` |
| `z.string().email()` | `z.email()` |
| `z.string().uuid()` | `z.uuid()` |
| `z.string().url()` | `z.url()` |
| `z.string().datetime()` | `z.iso.datetime()` |
| `z.string().date()` | `z.iso.date()` |
| `z.string().time()` | `z.iso.time()` |
| `z.string().duration()` | `z.iso.duration()` |
| `z.string().ip()` | `z.ipv4()` or `z.ipv6()` |
| `z.string().emoji()` | `z.emoji()` |
| `z.string().base64()` | `z.base64()` |
| `z.string().nanoid()` | `z.nanoid()` |
| `z.string().cuid()` | `z.cuid()` |
| `z.string().cuid2()` | `z.cuid2()` |
| `z.string().ulid()` | `z.ulid()` |
| `.min(5, { message: "Too short" })` | `.min(5, { error: "Too short" })` or `.min(5, "Too short")` |
| `z.string({ errorMap: (iss, ctx) => ... })` | `z.string({ error: (iss) => "..." })` |

## Common Anti-Patterns

**Wrong import — silently loads v3 compat:**
```ts
// ❌
import z from 'zod'
import { z } from 'zod'

// ✅
import * as z from 'zod'
```

**String format validators (deprecated method chain):**
```ts
// ❌
const Email = z.string().email()
const Uuid  = z.string().uuid()
const Dt    = z.string().datetime()

// ✅
const Email = z.email()
const Uuid  = z.uuid()
const Dt    = z.iso.datetime()
```

**Error message param:**
```ts
// ❌
z.string().min(5, { message: "Too short" })
z.string({ errorMap: (iss, ctx) => ({ message: ctx.defaultError }) })

// ✅
z.string().min(5, "Too short")                                   // shorthand
z.string().min(5, { error: "Too short" })                        // object form
z.string({ error: (iss) => iss.code === "too_small" ? "Too short" : undefined })
```

**Named imports of types:**
```ts
// ❌
import { ZodError, ZodIssue, ZodSchema } from 'zod'

// ✅
import * as z from 'zod'
type Issue = z.ZodIssue
function handle(e: z.ZodError) { ... }
```

**Coerce schema input type:**
```ts
// ❌ — assuming coerce input is typed (was the case in v3)
const schema = z.coerce.number()
type In = z.input<typeof schema>  // ❌ don't assume number — it's unknown

// ✅
const schema = z.coerce.number()
type In = z.input<typeof schema>  // unknown — accept any, coerce to number
```

## Quick Reference: Most-used APIs

| Need | API |
|---|---|
| String | `z.string()` |
| Number | `z.number()` |
| Boolean | `z.boolean()` |
| Email | `z.email()` |
| UUID | `z.uuid()` |
| URL | `z.url()` |
| ISO datetime | `z.iso.datetime()` |
| ISO date | `z.iso.date()` |
| Literal | `z.literal("admin")` |
| Enum | `z.enum(["a", "b", "c"])` |
| Object | `z.object({ key: schema })` |
| Array | `z.array(schema)` |
| Tuple | `z.tuple([z.string(), z.number()])` |
| Record | `z.record(z.string(), schema)` |
| Union | `z.union([a, b])` |
| Discriminated union | `z.discriminatedUnion("type", [a, b])` |
| Optional | `schema.optional()` |
| Nullable | `schema.nullable()` |
| Default | `schema.default(value)` |
| Transform | `schema.transform(val => ...)` |
| Parse (throws) | `schema.parse(data)` |
| Parse (safe) | `schema.safeParse(data)` |
| Extract type | `z.infer<typeof schema>` |
| Input type | `z.input<typeof schema>` |
| Output type | `z.output<typeof schema>` |

Full API reference (primitives, string validators, objects, arrays, unions, transforms, error handling, async) is in [api.md](api.md).

## Where to Find More Detail

- **[api.md](api.md)** — Full API reference: primitives, string validators, objects, arrays, unions, transforms, refinements, error handling, async, Zod Mini
- **https://zod.dev** — Official docs
- **https://zod.dev/v4/changelog** — Full v3→v4 changelog
