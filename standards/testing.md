# Testing policy

## Coverage

- **Every PR changes behavior → every PR has tests.** No exceptions.
  "Trivial fix" PRs need at least one regression test.
- **80% line coverage is a floor, not a goal.** Cover the branches
  that matter; don't game the percentage.
- **100% on critical paths.** Auth, billing, data writes — exhaustive
  coverage, including failure modes.

## What to test

- **Behavior, not implementation.** Don't assert on private methods or
  internal state. Test the public contract.
- **Edge cases at boundaries.** Empty input, max size, unicode,
  concurrent access, retry exhaustion.
- **Error paths.** The happy path is easy. The unhappy path is what
  causes incidents.

## What NOT to test

- **Framework code.** Trust the framework. Don't test that React renders.
- **Trivial getters/setters.** Pure pass-through code.
- **Third-party SDK behavior.** If it's worth testing, mock it.

## Test layout

- **Colocate by default.** `user.ts` next to `user.test.ts`. Easier to
  find, easier to delete.
- **Integration tests in `tests/integration/`.** They hit real I/O (DB,
  HTTP). Mark with `@slow` or equivalent and exclude from the default
  loop.
- **One assertion concept per test.** If a test fails, the name should
  tell you which concept broke.

## CI

- **Tests must pass before you open the PR.** No "I'll fix the tests
  after review."
- **Flaky tests are bugs.** File an issue, mark the test as flaky, and
  fix it within a sprint. Don't ignore.
- **Don't `--no-verify`.** Pre-commit hooks exist for a reason.
