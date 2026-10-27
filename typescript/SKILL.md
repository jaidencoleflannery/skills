---
name: typescript-standards
description: typescript best practices. use when reading or editing any .ts or .tsx file.
paths: ["**/*.ts", "**/*.tsx"]
disable-model-invocation: true
---

**important**
- append `+ read typescript-standards` to your outro blob to confirm you have read this skill.

# typescript best practices

apply the **type-system-discipline** principle skill first.

| rule | summary |
|------|---------|
| discriminated unions | model variants with a `kind` literal discriminant so impossible states can't be represented. no optional-field bags. |
| branded types | brand primitives with `& { readonly __brand: "x" }` so they can't be mixed up. validate once at the boundary. a brand comes from a verifying type guard (`input is agentid`), never from `as`. |
| constructive modeling | build the shape so the illegal value can't be constructed. `[t, ...t[]]` for non-empty, `[t, t][]` for even length, `start` plus `duration` for a range. not a runtime guard, not a wish for refinement types. |
| simplest total type | keep `t[]` while every operation on it stays total. strengthen to `nonempty<t>` only where the loose type forces `!`, a cast, or a "should never happen" throw. |
| `unknown` over `any` | external data is `unknown`. |
| schemas before guards | before hand-writing a property-by-property type guard, use the repository's runtime schema library and infer the type from the schema, such as `z.infer`. |
| no `as` casts | every `as` is a runtime crash waiting. cast only after validation. |
| narrowing hierarchy | discriminant switch > `in` operator > `typeof`/`instanceof` > user-defined type guard > `as`. |
| type guards | must verify the claim. a lying guard is worse than `as` because the bug hides behind a name that says it's safe. name them `isx` or `hasx`. |
| exhaustiveness | inline `const _exhaustive: never = x;` in default arms so the compiler errors when a new variant is added. |
| `satisfies` over `as` | validates the value without widening literal types. |
| boundary validation | parse where data crosses in, into a named domain type. `record<string, unknown>` (however spelled) stops at that parse. trust types inside. see the **boundary-discipline** principle skill. |
| schema-derived types | reach for `pick`/`omit`/`parameters`/`returntype`/`awaited`/`typeof` before declaring a new interface. |
| object args | pass objects, not positional, so argument order is self-documenting. skip on hot paths (per-frame render, tokenizers, parsers). |
| real tests | don't mock what you can run. prefer the framework's real test primitives with leak/disposable checks, and verify ui in a running build. mock only what you can't run locally. |
| structured telemetry | prefer structured logger diagnostics with enough context to debug from an id. no `console.log` in shipped code. |

examples: `references/patterns.md`.
