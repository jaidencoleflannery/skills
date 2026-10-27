---
name: principle-boundary-discipline
description: "apply when wiring validation, error handling, or framework adapters. concentrate guards at system boundaries (cli, config, network, external apis); trust internal types and keep business logic in pure functions."
disable-model-invocation: true
---

# boundary discipline

place validation, type narrowing, and error handling at system boundaries. trust internal code unconditionally. business logic lives in pure functions. the shell is thin and mechanical.

**why:** scattered validation is noisy, redundant, and gives a false sense of safety. keep logic out of framework wiring so it can be tested without the framework.

**the pattern:**
- **at boundaries** (cli args, config files, external apis, network protocols): validate, return errors, handle defensively.
- **inside the system:** typed data, error propagation, no re-validation. trust the types.
- **across the boundary.** expose domain concepts, not the boundary's private representation. keep general-purpose mechanism inside and special-purpose policy at the edge.

**applications:**

validation and error handling:
- validate config at parse time (the boundary), not inside business logic
- parse raw data into domain types at the boundary
- do not re-export transport, storage, framework, or wire types through the public surface
- no redundant nil checks deep in call chains if the boundary already validated

code organization:
- business logic in pure functions with no framework dependencies
- parse functions: pure transforms from raw bytes to typed state
- prompt construction: structured state in, string out
- scoring and assessment: pure transforms from state to results

**the tests:**
- "is this data crossing a system boundary right now?" if not, validation is redundant.
- "can this be a pure function that the shell just calls?" if yes, extract it.
