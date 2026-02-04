# Test Audit Criteria

## 1. Test Validity

Every test must verify what it claims.

| Area | Requirement |
|------|-------------|
| **HTML/DOM** | Assert on structure (tags, attributes, nesting)—not substrings |
| **Data** | Validate exact values, not mere existence |
| **Side effects** | Confirm state changes, not just return values |
| **Error paths** | Cover failures and edge cases, not only happy paths |
| **Mocks** | Verify call counts and arguments |

## 2. Assertion Quality

Strong assertions catch real bugs. Weak ones mask them.

- **Be specific.** Write `expect(user.name).toBe("Alice")`, not `toBeTruthy()`
- **Be complete.** Check all relevant properties, not just one
- **Be meaningful.** Avoid assertions that always pass
- **Limit snapshots.** Never use them for dynamic content
- **Require assertions.** Flag tests that assert nothing

## 3. Test Isolation

Tests must run independently—in any order, at any time.

- Avoid shared mutable state without proper setup/teardown
- Mock external dependencies: APIs, databases, filesystems
- Clean up resources after each test

## 4. Naming and Structure

Good tests document themselves.

- **Name precisely.** Describe the scenario and expected outcome
- **Test one thing.** One behavior per test
- **Group logically.** Use describe/context blocks for related tests

### AAA Paradigm (Required)

Every test must follow **Arrange → Act → Assert**:

| Phase | Purpose | Guidelines |
|-------|---------|------------|
| **Arrange** | Set up preconditions | Initialize data, create mocks, configure state. Keep it minimal—only what's needed for this test. |
| **Act** | Execute the code under test | One action only. If you need multiple actions, split into separate tests. |
| **Assert** | Verify the outcome | Check return values, state changes, and side effects. Be specific. |

**Flag these violations:**

```
// ❌ BAD: Mixed phases, no clear structure
it("processes order", () => {
  const order = createOrder();
  expect(order.id).toBeDefined();      // Assert before Act
  order.process();                      // Act
  const updated = fetchOrder(order.id); // More Arrange?
  expect(updated.status).toBe("done");  // Assert
});

// ✅ GOOD: Clear AAA separation
it("sets status to done when order is processed", () => {
  // Arrange
  const order = createOrder({ status: "pending" });
  
  // Act
  order.process();
  
  // Assert
  expect(order.status).toBe("done");
});
```

**AAA checklist:**
- [ ] Arrange section is clearly separated (comments optional but phases distinct)
- [ ] Only one Act per test
- [ ] No assertions before the Act phase
- [ ] No additional setup after the Act phase
- [ ] Assert phase checks the specific behavior under test

## 5. Coverage Quality

Hunt for gaps in:

- **Boundaries** — min/max values, empty inputs, null/undefined
- **Errors** — invalid inputs, network failures, permission denied
- **State transitions** — every significant change
- **Async behavior** — proper awaiting, no flaky timing

## 6. Maintainability

Tests should evolve with the codebase, not against it.

- Extract shared setup—but keep tests readable on their own
- Test behavior, not implementation
- Use factories and fixtures; avoid magic values
- Delete or justify commented-out tests

## 7. Framework Conventions

Apply idiomatic practices:

| Framework | Best Practices |
|-----------|----------------|
| **Jest** | Clear mocks, handle async correctly, use `beforeEach` |
| **pytest** | Leverage fixtures and `@parametrize` |
| **RSpec** | Use `let`/`let!`, `subject`, shared examples |
| **JUnit** | Pair `@BeforeEach`/`@AfterEach` properly |
| **PHPUnit** | Use data providers, `setUp`/`tearDown` |

## Common Antipatterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| `expect(html).toContain("Hello")` | Ignores HTML structure | Use `getByRole` + `toHaveTextContent` |
| `expect(result).toBeDefined()` | Passes for any value | Assert the exact expected value |
| `expect(response.status).toBe(200)` alone | Ignores the body | Add body assertions |
| `await someAsyncFn()` with no assertion | Verifies nothing | Assert the result or side effect |
| `it("works")` | Says nothing | `it("returns user for valid ID")` |
| Large snapshot of API response | Brittle, hard to review | Assert on key fields explicitly |
