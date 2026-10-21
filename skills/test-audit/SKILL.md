---
name: test-audit
description: Audit test suites (unit, integration, E2E) in any language for correctness and quality. Use when the user wants tests reviewed or improved ("audit my tests", "are my tests good"), test coverage or gaps checked, or test strength measured with mutation testing.
license: MIT
metadata:
  version: "1.1.0"
---

# Test Suite Audit

## Step 1: Clarify Scope

If the request already names the scope, skip to Step 2. Otherwise ask:

> Would you like me to audit:
> 1. **A specific file** — Provide the path to the test file
> 2. **Only changed files** — I'll audit tests in the diff against `main` (new or modified tests)
> 3. **The entire project** — I'll scan for all test files in the codebase
>
> Also: is there a spec, issue, or list of required test cases I should check the tests against?

Wait for the answer. Done when the scope is fixed and you know whether a specification exists.

## Step 2: Gather Context

Read the project's test config, package scripts, and CI files to find the framework, the test command, and the test files. Then collect, for every test file in scope:

- **Code under test** — read it; many test problems start in the production code
- **Specification** — the user's case list, PR description, linked issue, spec or plan file
- **History** (diff audits) — tests added in commits that only delete code

Done when you have the test command, the list of test files in scope, and the production file each one exercises.

## Step 3: Apply Audit Criteria

Apply every section of `references/audit-criteria.md` to every test in scope. Done when each section has been checked against each file, including the sections that produce **Design** findings (Testability and Design Signals) and specification gaps (Traceability to the Specification).

## Step 4: Measure Test Strength (Optional)

If a mutation testing tool exists for the language, offer to run it on the audit scope. Read `references/mutation-testing.md` before you run it: it has the preconditions, tool flags, and how to classify surviving mutants. If the user declines or no tool exists, the audit is static only.

## Step 5: Report Findings

Severity, unless a criteria section sets its own:
- **HIGH** — the test can pass while the behavior it claims is broken, or a required error or security case has no test
- **MEDIUM** — the test is weak, brittle, or misleading: it misses some bugs or breaks on a refactor
- **LOW** — naming, structure, style

For each issue:

```
### [HIGH | MEDIUM | LOW] [Test | Design] Issue Title

**File**: `path/to/test_file.ext`
**Test**: `test name`
**Lines**: X–Y

**Problem**: What's wrong

**Current**:
[code snippet]

**Fix**:
[corrected code]

**Why**: One-sentence rationale
```

For a **Design** finding, set **File** to the production file and give the production-code fix under **Fix**, with a short-term test fix after it.

## Step 6: Summarize

Conclude with:

1. **Critical issues** — the HIGH findings
2. **Quality issues** — the MEDIUM findings
3. **Style issues** — the LOW findings
4. **Design findings** — production-code changes that make the tests simpler
5. **Test strength** — if Step 4 ran: tool, scope, productive surviving mutants (score as context only); else "static audit only"
6. **Overall health** — one paragraph
7. **Top 3–5 priorities** — highest-impact improvements
8. **Suggested test list** — each missing case as one given/when/then line, ready to paste into the next prompt to an agent
