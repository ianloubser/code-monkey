# Code patterns

## Naming

- **Files:** `kebab-case.ts`, `snake_case.py` (language convention wins).
- **Functions:** verbs. `parseUser`, not `userParser`.
- **Booleans:** predicates. `isReady`, `hasPermission`, `canEdit`.
- **Constants:** `SCREAMING_SNAKE_CASE` only for true constants (env
  names, magic numbers with semantic meaning). Use `const` for
  module-level values.

## Functions

- **One job.** If your function name has "and" in it, split it.
- **Short.** 20 lines is a soft limit. 50 is a hard one — split it.
- **Few params.** 0–3 is fine. 4+ needs a params object.
- **Return early.** Guard clauses beat nested ifs.

## Types

- **Strict by default.** No `any` in TS, no untyped dicts in Python.
- **Prefer narrow types.** `UserId` over `string` if it adds meaning.
- **Parse, don't validate.** At the boundary, return a typed value or
  reject — don't pass a string deep into the call stack.

## Errors

- **Throw for exceptional, return for expected.** Network down: throw.
  Validation failed: return result type.
- **Wrap with context.** `throw new Error(\`fetchUser(${id}): ${e}\`)`
  not just `throw e`.
- **Don't swallow.** `catch {}` is almost always wrong.

## Comments

- **Why, not what.** The code shows what; the comment shows why.
- **No narrative.** `// increment i` is noise.
- **TODOs are tickets.** `// TODO: fix this` should be `// TODO(#123):
  fix this` with a tracked issue.

## Git

- **One logical change per commit.** Not one file, not one line — one
  change a reviewer can reason about.
- **Subject line under 72 chars.** Imperative mood. `Add rate limiter`
  not `Added`.
- **Body explains why.** What is in the diff; why is in the message.
