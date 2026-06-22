# Zod v4 API Reference

Full API reference for `zod` ^4.0.0. All examples use `import * as z from 'zod'`.

## Primitives

```ts
z.string()
z.number()
z.bigint()
z.boolean()
z.date()
z.symbol()
z.undefined()
z.null()
z.any()
z.unknown()
z.never()
z.void()
```

## String Validators (v4 top-level)

All string format validators are now top-level. Method chains (`.email()` etc.) are **deprecated**.

```ts
z.email()
z.uuid()
z.url()
z.emoji()
z.base64()
z.base64url()
z.nanoid()
z.cuid()
z.cuid2()
z.ulid()
z.ipv4()
z.ipv6()
z.cidrv4()
z.cidrv6()

// ISO formats live under z.iso
z.iso.date()        // "2024-01-01"
z.iso.time()        // "12:30:00"
z.iso.datetime()    // "2024-01-01T12:30:00Z"
z.iso.duration()    // "P1Y2M3D"
```

All string validators extend `ZodString` and support the same string methods:

```ts
z.email().min(5).max(100)
z.uuid().optional()
```

## String Methods

```ts
z.string()
  .min(n)
  .max(n)
  .length(n)
  .regex(/pattern/)
  .includes("str")
  .startsWith("str")
  .endsWith("str")
  .trim()
  .toLowerCase()
  .toUpperCase()
```

## Number Methods

```ts
z.number()
  .min(n)   // alias: .gte(n)
  .max(n)   // alias: .lte(n)
  .gt(n)
  .lt(n)
  .int()
  .positive()
  .nonnegative()
  .negative()
  .nonpositive()
  .multipleOf(n)
  .finite()
  .safe()
```

Note: `z.number()` in v4 does not accept `Infinity` / `-Infinity` by default. Use `.finite()` or `z.number().catch(0)` to handle.

## Coerce

Input type is `unknown` in v4 (was specific type in v3):

```ts
z.coerce.string()   // input: unknown, output: string
z.coerce.number()   // input: unknown, output: number
z.coerce.boolean()  // input: unknown, output: boolean
z.coerce.bigint()   // input: unknown, output: bigint
z.coerce.date()     // input: unknown, output: Date
```

## Literals, Enums

```ts
z.literal("admin")
z.literal(42)
z.literal(true)

z.enum(["red", "green", "blue"])
// type: "red" | "green" | "blue"
// access values: MyEnum.options → ["red", "green", "blue"]

z.nativeEnum(MyTSEnum)  // TypeScript enum
```

## Objects

```ts
const User = z.object({
  id:    z.uuid(),
  name:  z.string(),
  email: z.email().optional(),
})

type User = z.infer<typeof User>
// { id: string; name: string; email?: string }
```

Object modifiers:

```ts
User.partial()           // all fields optional
User.partial({ id: true })  // only id optional
User.required()          // all fields required
User.pick({ id: true, name: true })
User.omit({ email: true })
User.extend({ age: z.number() })
User.merge(OtherSchema)

// Key access
User.shape.id  // → ZodString
User.keyof()   // → ZodEnum<["id", "name", "email"]>
```

Unknown key handling:

```ts
z.object({...}).strip()      // (default) strip unknown keys
z.object({...}).strict()     // throw on unknown keys
z.object({...}).passthrough()// pass unknown keys through
```

Defaults work inside optional fields in v4:

```ts
z.object({
  role: z.string().default("user"),  // ✅ works in v4
})
```

## Arrays

```ts
z.array(z.string())
z.string().array()  // same

z.array(z.number()).min(1)
z.array(z.number()).max(10)
z.array(z.number()).length(5)
z.array(z.number()).nonempty()  // min 1 item, type: [number, ...number[]]
```

## Tuples

```ts
z.tuple([z.string(), z.number()])
// type: [string, number]

z.tuple([z.string()]).rest(z.number())
// type: [string, ...number[]]
```

## Records

```ts
z.record(z.string(), z.number())
// type: Record<string, number>

z.record(z.enum(["a", "b"]), z.string())
// type: Record<"a" | "b", string>
```

## Maps & Sets

```ts
z.map(z.string(), z.number())
// type: Map<string, number>

z.set(z.string())
// type: Set<string>
z.set(z.string()).min(1).max(10)
```

## Unions & Intersections

```ts
z.union([z.string(), z.number()])
// type: string | number

// Discriminated union (faster, required discriminant field)
const Shape = z.discriminatedUnion("type", [
  z.object({ type: z.literal("circle"),    radius: z.number() }),
  z.object({ type: z.literal("rectangle"), width: z.number(), height: z.number() }),
])

z.intersection(z.object({ a: z.string() }), z.object({ b: z.number() }))
// type: { a: string } & { b: number }
// Prefer z.merge() for objects
```

## Optional, Nullable, Nullish

```ts
schema.optional()       // T | undefined
schema.nullable()       // T | null
schema.nullish()        // T | null | undefined

z.optional(schema)      // same as above (functional form)
z.nullable(schema)
z.nullish(schema)
```

## Default & Catch

```ts
z.string().default("hello")           // undefined → "hello"
z.string().default(() => uuid())      // lazy default

z.number().catch(0)                   // invalid input → 0 (no throw)
z.number().catch((ctx) => ctx.input as number ?? 0)
```

## Transforms & Pipes

```ts
// Transform output
z.string().transform(s => s.toUpperCase())
// input: string, output: string (uppercased)

// Pipe into another schema
z.string()
  .transform(s => parseInt(s, 10))
  .pipe(z.number().int())
// input: string, output: number

// Preprocess input
z.preprocess(val => String(val), z.string())
```

## Refinements

```ts
// Simple refinement
z.string().refine(s => s.length > 5, "Must be longer than 5")

// With options
z.string().refine(
  s => s.startsWith("https"),
  { error: "Must start with https" }
)

// Async refinement
z.string().refine(async (s) => await isAvailable(s), "Already taken")

// SuperRefine — multiple issues, control flow
z.object({ password: z.string(), confirm: z.string() })
  .superRefine((data, ctx) => {
    if (data.password !== data.confirm) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Passwords don't match",
        path: ["confirm"],
      })
    }
  })
```

## Error Customization

```ts
// String shorthand
z.string().min(5, "Too short")
z.email("Invalid email")

// Object form
z.string().min(5, { error: "Too short" })

// Error function (full control)
z.string({
  error: (issue) => {
    if (issue.code === "too_small") return `Min length: ${issue.minimum}`
  }
})

// Global error map
z.setErrorMap((issue, ctx) => {
  if (issue.code === "invalid_type") return { message: "Wrong type" }
  return { message: ctx.defaultError }
})
```

## Parsing

```ts
// Throws ZodError on failure
const data = schema.parse(input)

// Returns result object — no throw
const result = schema.safeParse(input)
if (result.success) {
  result.data   // typed output
} else {
  result.error  // ZodError
}

// Async (for async refinements/transforms)
const data = await schema.parseAsync(input)
const result = await schema.safeParseAsync(input)
```

## Error Handling

```ts
import * as z from 'zod'

try {
  schema.parse(badInput)
} catch (e) {
  if (e instanceof z.ZodError) {
    e.issues        // ZodIssue[]
    e.flatten()     // { formErrors: string[], fieldErrors: Record<string, string[]> }
    e.format()      // nested error object
  }
}

// ZodIssue structure
// { code, message, path, ... }
// code: z.ZodIssueCode.invalid_type | too_small | too_big | custom | ...
```

## Type Extraction

```ts
const Schema = z.object({ name: z.string(), age: z.number() })

type Output = z.infer<typeof Schema>      // { name: string; age: number }
type Input  = z.input<typeof Schema>      // same when no transforms
type Out    = z.output<typeof Schema>     // same as z.infer

// With transform
const Coerced = z.string().transform(s => parseInt(s, 10))
type In  = z.input<typeof Coerced>    // string
type Out = z.output<typeof Coerced>   // number
```

## Recursive & Lazy Schemas

```ts
// Self-referencing
type Category = { name: string; subcategories: Category[] }
const Category: z.ZodType<Category> = z.object({
  name: z.string(),
  subcategories: z.lazy(() => Category.array()),
})
```

## Branded Types

```ts
const UserId = z.string().uuid().brand<"UserId">()
type UserId = z.infer<typeof UserId>  // string & { [BRAND]: "UserId" }
```

## Zod Mini

Functional, tree-shakable variant. Same types, different API:

```ts
import * as z from 'zod/mini'

// Method chains become function calls
z.optional(z.string())       // vs z.string().optional()
z.union([z.string(), z.number()])
z.array(z.string())

// Validation becomes .check()
z.string().check(z.minLength(5), z.maxLength(100))
z.string().check(z.email())
z.string().check(z.regex(/pattern/))
```

Use Zod Mini when bundle size is critical. API is one-to-one with main `zod`.
