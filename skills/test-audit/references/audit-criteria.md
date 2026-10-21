# Test Audit Criteria

Two leading words recur below:
- **False positive**: a test that passes while the behavior it claims is broken. Default severity HIGH.
- **Change-detector**: a test that fails when the code's structure changes but its behavior does not. It checks how the code is written, not what it does. Default severity MEDIUM.

## Assertions

Every test verifies what its name claims, with assertions that fail when the behavior breaks.

- **Exact values.** `expect(user.name).toBe("Alice")`, not `toBeTruthy()` or `toBeDefined()`. Check every property the behavior sets.
- **Every test asserts.** `await someAsyncFn()` with no assertion, or an assertion that always passes, is a false positive.
- **Structure for HTML/DOM.** Assert tags, roles, and attributes (`getByRole` + `toHaveTextContent`), not substrings of markup.
- **State and side effects.** Confirm the state change, not only the return value. `expect(response.status).toBe(200)` alone ignores the body.
- **Outcomes over calls.** Assert the result first. Verify calls only for state-changing calls to external systems, with only the arguments that matter.
- **Small, stable snapshots.** Use snapshots only for small output that stays the same between runs. For an API response, assert the key fields.
- **Derived expected values.** An expected value must be one a reader can check: a long opaque literal (hash, large number, full serialized object) with no derivation, or a snapshot created in the same change as the code, was likely copied from the actual output. It asserts whatever the code does today, bugs included. **MEDIUM**. Fix: derive the value, or assert a checkable property.

## Coverage

Hunt for gaps in:

- **Boundaries** — min/max values, empty inputs, null/undefined
- **Errors** — invalid inputs, network failures, permission denied
- **State transitions** — every significant change
- **Async behavior** — every promise awaited, timing controlled

A suite written from a bare "write tests" prompt usually mirrors the code's structure (one test per method, in source order). Audit it against behaviors, not methods.

## Traceability to the Specification

Find the specification: the user's case list, a PR description, a linked issue, `SPEC.md`, `PLAN.md`, an ADR, or acceptance criteria.

If one exists:
1. List each required behavior and edge case.
2. Find the test for each one.
3. Report each required case with no test: **HIGH** for an error path or security rule, else **MEDIUM**.
4. Report each test that maps to no requirement and no clear behavior: **LOW**. Tests for cases that cannot happen (inputs the type system or a validated boundary already excludes) belong here.

If none exists, write down the behaviors the tests imply and ask the user to confirm them.

Every missing case from this section and from Coverage goes into the **Suggested test list** in the summary.

## Isolation

Each test passes alone, in any order, at any time.

- Each test sets up the state it needs and cleans up what it creates
- Replace slow or non-deterministic dependencies: network, databases, filesystems, clock
- Use real objects for fast, deterministic collaborators; mock the project's own classes only with a stated reason

## Naming and Structure

- **Behavior names.** Name each test as given a state, when an action happens, then an outcome: `it("returns user for valid ID")`. Flag names like "works", "handles input", "test 1". **LOW**.
- **One behavior per test.** A name that needs "and" usually holds two.
- **Grouping.** Related tests share a describe/context block.
- **Arrange → Act → Assert.** Phases are distinct; exactly one Act; no assertions before the Act; no setup after it. **LOW**, unless the mixing hides what is under test.

## Maintainability

Tests change when behavior changes, and only then.

- Test through the public API; a test that reads internals is a change-detector
- Shared setup lives in helpers, factories, or fixtures, while each test still reads on its own
- Values carry names or come from factories, so a reader sees why each one matters
- Commented-out tests are deleted or carry a reason

### Removal and Absence Assertions

Agents often add a test after a "remove X" request that checks X is gone. It verified the removal once; in the suite it is a change-detector.

Sort every negative assertion into one class:

| Class | What it checks | Verdict |
|-------|----------------|---------|
| **Structural absence** | A symbol, export, method, file, package, CSS class, config key, or source text is gone | **MEDIUM** (LOW if cheap and isolated). Delete it. If the ban must stay permanent, move it to a lint rule (ESLint `no-restricted-imports`), an architecture test (ArchUnit), or the compiler |
| **Behavioral absence** | Through the public API or UI: a removed route returns 404, a response omits a secret field, a disabled feature has no effect | Keep it. Not a finding |

Signals of structural absence:
- Reads production source or manifest files (`readFileSync`, `open()`, `File.read` on `src/`, `package.json`, `requirements.txt`) and asserts on their text
- Asserts a symbol is missing: `expect(mod.oldFn).toBeUndefined()`, `not hasattr(...)`, `expect(() => require('./old')).toThrow()`, `assertFalse(method_exists(...))`
- Asserts rendered output omits a removed element, with no requirement behind it
- Name describes history, not behavior: "no longer", "removed", "was deleted", "does not exist anymore"
- Only negative assertions, with no positive assertion of the remaining behavior
- In a diff audit: the test arrives in a commit that only deletes code

To classify, ask: does a requirement, issue, or security rule say this must stay absent for users? Does the test go through the public API or UI as a user would? Two yes answers mean behavioral.

## Testability and Design Signals

A bad test is often a symptom of hard-to-test code. When a test shows a signal below, read the production code it exercises and find the cause there.

| Signal in the test | Probable cause in the code | Fix in the code |
|--------------------|----------------------------|-----------------|
| Many mocks, stubs, or patches in one test | Hard-coded dependencies (`new` of collaborators, module-level singletons) | Pass dependencies in through the constructor or a parameter |
| Patches of the project's own modules (`jest.mock('../services/x')`, `patch('myapp.module.func')`) | No seam at the dependency boundary | Add a seam: an injected interface or parameter |
| Patches of clock, randomness, or environment (`useFakeTimers`, `freezegun`, `os.environ` writes) | Direct reads of global state | Inject a clock, random source, or config object |
| Unit test starts a database, HTTP server, or filesystem to test pure logic | Logic mixed with I/O in one function | Split pure logic from I/O (Humble Object) |
| Arrange phase much longer than Act plus Assert | Unit needs too many collaborators, or constructor does I/O | Narrow the unit's responsibilities |
| Reads private members (reflection, `_private`, `@VisibleForTesting`, `as any`) | Behavior hidden behind internals | Test through the public API, or extract the hidden logic into its own unit |
| Production code has `if (NODE_ENV === 'test')` or test-only methods | Test logic in production | Remove the hook; add a seam |
| Sleeps, polling, or retry loops | Logic coupled to an async mechanism | Separate the logic from the scheduling |
| Asserts only `toHaveBeenCalledWith` / `verify(...)`, no state or output | Interaction-only change-detector | Assert on the outcome; keep call checks for external side effects |

Report these as **Design** findings: the production-code fix first, a test-only fix as a short-term step. Prefer real objects when they are fast and deterministic. Recommend a new seam only where it also simplifies the production code; an abstraction layer that exists only to enable a test is a cost too.

## Framework Idioms

Check each test against its framework's idioms. Flag hand-rolled versions of what the framework already provides: loops of cases instead of parametrized or table-driven tests (`@pytest.mark.parametrize`, `t.Run`, `@ParameterizedTest`, data providers); inline setup repeated across tests instead of hooks, fixtures, or factories; manual cleanup instead of the framework's teardown or transaction rollback (`RefreshDatabase`, `DatabaseCleaner`); mocks left active between tests instead of reset in a hook. **LOW**.
