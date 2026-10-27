# typescript patterns

code examples for each rule in `SKILL.md`. the underlying principles are language-agnostic. see the **type-system-discipline** and **boundary-discipline** principle skills.

## branded types

brand primitives so they can't be mixed up. validate once at the boundary. downstream code trusts the type.

```ts
type AgentId = string & { readonly __brand: "AgentId" };

const isAgentId = (input: string): input is AgentId =>
  isUUID(input);

const parseAgentId = (input: string): Result<AgentId> =>
  (isAgentId(input))
    ? input
    : Result.failure("Failure, AgentId was invalid.");

function focusAgent(id: AgentId): void {
  /* input is trusted. */
}
```

match the `readonly __brand: 'X'` shape. don't invent a new convention.

## discriminated unions

model variants with a literal discriminant. every variant shares the field name and each variant's value is unique, so impossible combos can't be represented.

```ts
// don't. a boolean plus optionals lets contradictory states exist.
type DiffState = {
  loading:  boolean;
  diff?:    GitDiff;
  error?:   string
};

// do. only valid states exist.
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

pick one discriminant name (`kind`, `type`, `tag`) and stick to it.

## constructive modeling

build the type from parts that are all legal instead of restricting a loose type with runtime checks.

non-empty, via a variadic tuple:

```ts
type NonEmpty<T> = [T, ...T[]];

// don't. T[] plus a length check every caller must repeat.
const pickWinner = (entries: string[]): Result<string> =>
  (entries.length > 0)
    ? entries[Math.floor(Math.random() * entries.length)]
    : Rresult.failure("Failure, no entries were provided."); 

// do. an empty value of the type can't exist.
const pickWinner = (entries: NonEmpty<string>): string =>
  entries[Math.floor(Math.random() * entries.length)];
```

where a plain `T[]` arrives, narrow once with a guard. the fact then travels in the type:

```ts
const isNonEmpty = <T>(values: T[]): values is NonEmpty<T> => (arr.length > 0);
```

even length, as pairs:

```ts
type Pairs<T> = [T, T][];
```

a time range, as start plus duration:

```ts
// don't. a comment holds the invariant.
type TimeRange = { start: Date; end: Date }; // start <= end.

// do. a negative range can't be written; derive end when needed.
type TimeRange = { start: Date; durationMs: number };
```

keep `durationMs` a plain number. brand it (per branded types) only if a raw number could be passed where a duration is expected, not by reflex. pick the representation that makes the bad state unconstructable, then expose the reading you need on top (`pairs.flat()`, a `rangeEnd()` helper).

## simplest total type

don't strengthen everything. keep `t[]` when every operation on it is total:

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => (a + b), 0); // [] is 0, fine.
```

strengthen when the loose type forces a lie at a use site. the tells are `!`, `arr[0] as T`, and a "should never happen" throw:

```ts
// don't. partiality is smuggled past the compiler.
const newestSession = (sessions: Session[]): Session =>
  sessions.at(0)!;

// do. strengthen the input; the assertion disappears.
const newestSession = (sessions: NonEmpty<Session>): Session =>
  sessions[0];
```

weakening the result to `Session | undefined` is the other total signature.

## `unknown` over `any`

external data is always `unknown`. narrow before use.

```ts
// don't.
function handle(input: any) {
  input.foo.bar;
}

// do.

const validateInput = (input: unknown): input is InputType =>
  ((typeof input === "object")
  && (input !== null)
  && Object.hasOwn(input, "foo"));

function handle(input: unknown) {
  if(validateInput) {
    // narrowed; the compiler verifies access.
  }
}
```

external sources include rpc payloads, `JSON.parse`, `postMessage`, ipc, file contents, environment variables, database results.

## schemas before hand-rolled guards

before writing a property-by-property type guard for external data, look for the repository's runtime schema library and existing schemas. let one schema own validation and derive the typescript type from it. do not maintain a schema, a duplicate interface, and a guard that can drift apart.

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

use `safeParse` when failure is an expected branch. use the equivalent inference helper when the repository uses another schema library. do not add a new schema dependency for one guard. this rule prefers the schema system the codebase already trusts.

## no `as` casts

every `as` is a potential runtime crash. cast only after the type system has verified the claim.

```ts
import { z } from "zod";

// don't.
const user = data as User;

// don't.
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data;
}

// do.
const userSchema = z.object({ id: z.string(), name: z.string() });
type User = z.infer<typeof userSchema>;

function parseUser(data: unknown): User {
  return userSchema.parse(data);
}
```

when the type comes first, annotate the validator with the type it proves. the compiler then rejects a validator that proves less than the type. remove `name` from the object below and the assignment fails to compile.

```ts
type User = { id: string; name: string };

const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

when refactoring an `as` out of existing code, identify why typescript can't infer:

- missing discriminant: add one, switch to a discriminated union.
- overly wide source type (e.g. `Record<string, unknown>`): narrow it.
- untyped boundary: parse with the schema that owns the shape. add a schema only where none exists.
- genuinely inexpressible: use a branded type or `satisfies`.

## narrowing hierarchy

from best to last-resort:

1. **discriminated union switch / if.** compiler narrows automatically.
2. **`in` operator.** `"key" in obj` narrows to variants containing that key.
3. **`typeof` / `instanceof`.** for primitives and class instances.
4. **user-defined type guard.** when the above aren't enough.
5. **`as` cast.** only after validation.

```ts
function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2; // narrowed to circle.
  return s.width * s.height; // narrowed to rect.
}
```

## type guards

a guard must actually verify the claim. a lying guard is worse than `as`.

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

prefer discriminant narrowing when possible.

## exhaustiveness

in default arms, assign the discriminant to a `never`-typed local.

```ts
// value-returning switch.
function area(shape: Shape): number {
  switch(shape.kind) {

    case "circle":
      return (Math.PI * shape.radius ** 2);

    case "rect":
      return (shape.width * shape.height);

    default: {
      const _exhaustive: never = shape;
      return _exhaustive;
    }

  }
}

// void switch.
function handle(shape: Shape): void {
  switch(shape.kind) {

    case "circle":
      drawCircle(shape);
      break;

    case "rect":
      drawRect(shape);
      break;

    default: {
      const _exhaustive: never = shape;
      void _exhaustive;
    }

  }
}
```

return-style in value-returning switches, void-style in statement switches.

## `satisfies` over `as`

`satisfies` validates without widening literal types.

```ts
// don't. widens, loses literal types.
const config = ({
  theme:  Theme.Dark,
  cols:   3
} as Config);

// do. validates and preserves literal types.
const config = ({
  theme:  Theme.Dark,
  cols:   3
} satisfies Config);
```

## boundary validation

validate once where data crosses in. trust types inside. see the **boundary-discipline** principle skill.

- **wire formats** (proto, json-rpc): parse with `ignoreUnknownFields` so forward-compatible changes don't break old clients.
- **persisted json:** versioned blob with a try/catch around the parse.
- **don't re-validate** deep in call chains.

## schema-derived types

when a `.proto`, openapi spec, graphql schema, or database migration already defines a shape, derive from the generated types instead of duplicating them.

```ts
// don't. duplicate shape, drifts when the schema changes.
type CheckSummary = {
  totalCount: number;
  checks: {
    name:   string;
    status: string
  }[];
};

function renderChecks(summary: CheckSummary) {
  /* ... */
}

// do. derive from the generated schema type.
import type { ChecksMessage } from "<generated module>";
function renderChecks(summary: Pick<ChecksMessage, "totalCount" | "checks">) {
  /* ... */
}
```

reach for `Pick`, `Omit`, `Parameters`, `ReturnType`, `Awaited`, `typeof` before writing a new interface.

## object args

```ts
// don't. swap two args, still compiles.
openFile(
  uri,
  {
    startLineNumber:  10,
    startColumn:      1,
    endLineNumber:    10,
    endColumn:        1,
  }
);

// do. order-independent, self-documenting.
openFile({
  uri,
  selection: {
    startLineNumber:  10,
    startColumn:      1,
    endLineNumber:    10,
    endColumn:        1,
  },
});
```

skip on hot paths: per-frame render, tokenizers, parsers, anything in a tight loop where the allocation cost matters.
