# Mutation Testing

Static review shows what a test claims. A mutation tool shows what it detects: it changes the production code (flips `<` to `<=`, deletes a call, returns a constant) and reruns the suite. A **surviving mutant** is a change no test noticed. A **NoCoverage** mutant sits on a line no test runs. Tools count a mutant that times out as detected.

## Preconditions

Run only when all are true:
- A mutation tool for the language is installed, or the user agrees to install one
- The suite passes and is not flaky; a flaky suite makes survivors meaningless
- The user agrees to the run time

Otherwise skip, and state in the summary that the audit is static only.

## Tools

| Language | Tool | Scope to the audit |
|----------|------|--------------------|
| JS / TS | StrykerJS | `--mutate src/app.js:5-7`; `--incremental` |
| C# | Stryker.NET | `--since[:committish]` |
| Scala | Stryker4s | `mutate` (file globs only) |
| Java / JVM | PIT (pitest) | `targetClasses`, `targetTests`; `-DwithHistory` |
| Python | mutmut | `mutmut run "my_module*"`; `# pragma: no mutate` |
| Rust | cargo-mutants | `--in-diff DIFF_FILE` |
| PHP | Infection | `--git-diff-lines`, `--git-diff-filter=AM` |
| Go | Gremlins | `--diff <branch-or-commit>` (needs full git history) |

## Procedure

1. Scope the run to the audit. For a diff audit, mutate only changed lines with the tool's diff option; for a file audit, mutate only the production file that test exercises.
2. Run on a clean, committed tree or a copy, with test credentials only.
3. Collect every surviving and NoCoverage mutant.
4. Classify each one by asking: would a user or caller notice this change?

   | Class | What it is | Verdict |
   |-------|------------|---------|
   | **Productive** | A real behavior change no test detects | Finding: name the missing assertion or case. **HIGH** in an error path, a boundary, or a security check; else **MEDIUM** |
   | **NoCoverage** | A changed line no test runs | Finding: name the missing test. **MEDIUM** |
   | **Unproductive** | A change in logging, messages, tuning constants, timeouts, or a mocked layer | Leave it. A test written to kill it would be a change-detector |
   | **Equivalent** | No observable behavior change | Leave it; optionally suggest a simpler code form |

5. Keep one mutant per line: the most severe.

Done when every surviving and NoCoverage mutant has a class.

## Reporting

Report each finding in the Step 5 format of `SKILL.md`. Show the mutated line under **Current**, and under **Fix** the assertion or test that would fail on it. Add each missing case to the **Suggested test list**.

Many survivors in a heavily mocked unit point to a Design finding ("Testability and Design Signals" in `audit-criteria.md`): the tests check calls, not results.

In the summary, list the tool, the scope, and the productive and NoCoverage counts. Give the mutation score as context only: the findings are the result, and chasing a 100% score produces change-detector tests.
