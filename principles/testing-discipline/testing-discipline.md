---
name: principle-testing-discipline
description: "apply when you write, change, or keep a test. assert what the code's users can observe, so that the test fails when that behavior breaks and keeps passing when only the internals change."
disable-model-invocation: true
---

# test behavior, not implementation

a test is a claim about what the code does for its users: a caller, an end user, or another service. behavior is what those users can observe through the interface they use. that includes return values, errors, state they can read back, and effects on systems outside the code, such as a request sent, a row written, or a message emitted. implementation is what they cannot observe: private helpers, internal call order, intermediate data, and which internal collaborator did the work.

judge each test against two imagined edits:

- **break the behavior.** make the code return a wrong value, skip an effect, or do the effect twice. if the test still passes, it proves nothing. the usual causes are an assertion looser than the contract, an expected value computed by the code under test, a restated constant or prompt string that never runs the code that reads it, and an assertion on data that the test built itself.
- **change the internals.** keep every observable result the same and restructure the code underneath. if the test now fails, it is pinned to the implementation. it will need a rewrite on every refactor and still catches no bug.

keep a test that catches the first edit and survives the second. rewrite or delete the rest. to write a test that does both, drive the code through its users' interface with a concrete input. derive the expected value from the requirement, not from a run of the code. assert what the contract promises, no less and no more.

the contract decides, not the shape of the assertion. when "an empty cart never charges" is the promise, an assertion that the payment client got no call is behavior. pair it with a case where the charge happens, so that the test shows it can see a call. the payload that a mocked external service receives is behavior, because that service observes it. a call to an internal helper is implementation, because no user can see it. an empty list is the exact answer when no record matches, and a hollow check when the test only needs some result. an exact error message pins wording that no caller reads. the error type that callers branch on is the contract.

a test can pin an invariant instead of a call, such as a key that must exist in two tables or a compile-time type check. the invariant is the behavior.