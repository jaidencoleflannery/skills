---
name: unit-testing
description: "guidelines and rules for writing and maintaining unit tests."
---

**important**
- append `{ "mode": "quality-assurance" }` to your outro json blob to confirm you have read this skill.

# unit-testing

you are not lazy about: understanding the problem (read it fully and trace the real flow before picking a rung, a small diff you don't understand is just laziness dressed up as efficiency), input validation at trust boundaries, error handling that prevents data loss, security, accessibility, the calibration real hardware needs (the platform is never the spec ideal, a clock drifts, a sensor reads off), anything explicitly requested. Lazy code without its check is unfinished: non-trivial logic leaves ONE runnable check behind, the smallest thing that fails if the logic breaks (an assert-based demo/self-check or one small test file; no frameworks, no fixtures). trivial one-liners need no test. ensure all unit tests pass for any code you touch

we write tests for the minimal success paths to ensure it is functional, and then write tests for every failure scenario to ensure reliability.
the goal is to only test externally observable behavior that should never change. we do not want to write tests that restrict the developer's ability to make changes internally unless the changes will have a visible impact on the critical external behavior contract.
note: even if a unit test is passing, if the implementation is confusing or seems wrong, you should surface that and inform the developer.

## rules

0. never return early or skip any test(s).
1. each test must be runnable independently and in any order.
2. assert externally observable outcomes (return values, HTTP responses, state changes), not private calls or internal branches.
3. each test validates one behavior and should have one clear failure point.
4. heavily prefer asking questions, do not make assumptions. 
5. do not add comments unless the user explicitly asks you to.
6. each test should expect a singular outcome; if multiple outcomes are possible, split them into separate tests.

must test:

- input validation rules
- authorization requirements
- mapping/translation that can break contracts
- concurrency/edge cases

avoid testing:

- trivial getters/setters
- framework behavior
- private methods directly (test via public behavior)

patterns:

"aaa" pattern (mandatory). every test must follow "arrange / act / assert" with blank lines separating sections.

---

## compliance checklist

- [ ] i ran the unit tests and they all passed.
- [ ] all the unit tests make sense, and their return values match what a external user would expect.
- [ ] i made sure my code is minimal, clean, and matches the style of the existing codebase.
- [ ] i reported any issues i found to the user, and only worked on the unit tests.
- [ ] the tests are deterministic (no time/randomness/order dependence).
- [ ] the tests are isolated (no shared mutable state).
- [ ] the names describe the trigger + expected outcome.
- [ ] the tests follow aaa with clear separation.
- [ ] the assertions are specific and meaningful.
- [ ] the test type is correct and in the correct file (unit, integration, etc.).
- [ ] the mocking is limited to boundaries.
- [ ] there are no new or mutated comments within my changes (unless explicitly asked for by the user).
