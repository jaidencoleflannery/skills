---
name: principle-type-system-discipline
description: "apply when designing types, reviewing a function signature, or writing code in any statically-typed language. make illegal states unrepresentable, brand semantic primitives, parse external data at boundaries, exhaust variants, derive from authoritative schemas."
disable-model-invocation: true
---

# type system discipline

make the compiler reject impossible states, mismatched primitives, and unhandled variants. prefer defining errors and special cases out of existence over proliferating handlers. for typescript syntax, see the `typescript-best-practices` skill.

- **brand semantic primitives.** `UserId` and `OrderId` are strings underneath but should not be interchangeable. give each its own type, such as a branded intersection in typescript or a newtype elsewhere. validate once at creation, trust the type downstream. if two arguments share a primitive type but mean different things, brand them.
- **external data is untyped until parsed.** rpc payloads, json, ipc messages, cli args, config files, environment variables, database rows. have a parse function at every boundary that turns unstructured input into the typed model. the **boundary-discipline** principle skill says where to put validation.
- **derive types from authoritative schemas.** when a protocol buffer, openapi spec, graphql schema, database migration, design-token file, or another file owns a shape, derive from it instead of hand-rolling a parallel type. see the **encode-lessons-in-structure** principle skill.
