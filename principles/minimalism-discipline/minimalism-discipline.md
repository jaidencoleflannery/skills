---
name: principle-minimalism-discipline
description: "apply when sequencing an addition, refactor, or rewrite. remove dead code, redundant validators, and stub references first, then build on the simpler base."
disable-model-invocation: true
---

# target minimalism, avoid complexity

when evolving a system, remove complexity first, then build. default to subtraction.

make simplification a continual investment. leave the design slightly simpler and more capable behind the same or smaller surface than you found it.

- sequence removal before construction
- cut before you polish (get to the minimum before investing in quality)
- design for observed usage, not speculative edge cases
- no speculative validators, parsers, or guards beyond what the spec demands
- simplify prompts (remove redundant instructions, excessive templates)
- when a reference has no novel content, delete it rather than leaving a stub
- do not repeat the same choice in several places. put it behind one source of truth and pass the result as a simple flag.
- make the smallest change that solves the problem. fewer lines beat "elegant" boilerplate.
