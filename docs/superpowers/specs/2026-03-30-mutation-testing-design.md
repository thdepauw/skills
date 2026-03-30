# Mutation Testing Skill — Design Spec

## Overview

A Claude Code skill (`/mutant-test`) that performs mutation testing on JS/TS codebases. Claude acts as the mutation engine: it reads source code, generates mutants, applies them one at a time, runs the test suite, records whether each mutant was killed or survived, and produces a terminal report.

## Invocation & Modes

The skill is user-invocable. Three modes via arguments:

| Invocation | Behavior |
|---|---|
| `/mutant-test full` | Mutate all JS/TS source files in the project |
| `/mutant-test file src/utils/retry.ts` | Mutate only the specified file(s) |
| `/mutant-test diff` | Mutate only files changed vs `main` (or `master` if no `main`) |
| `/mutant-test diff develop` | Mutate only files changed vs the specified branch |

- Default mode (no arguments): `full`
- Target files: `.js`, `.ts`, `.jsx`, `.tsx`
- Excluded: `node_modules`, test files (`*.test.*`, `*.spec.*`, `__tests__/`), config files, type declarations (`.d.ts`), generated files

## Green Baseline & Monorepo Detection

Before any mutation work begins:

### 1. Detect monorepo structure

Check for:
- `workspaces` field in root `package.json`
- `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`

If monorepo:
- Identify which packages are affected (based on mode)
- Map internal package dependencies between them
- Understand that mutating package A may require running tests in packages B and C that depend on A

### 2. Auto-detect the test runner

Per package (monorepo) or project-wide (single repo). Priority order:
1. `test:ci` or `ci:test` script in `package.json` (preferred — matches CI)
2. `test` script in `package.json` (fallback)
3. Direct detection of Jest/Vitest/Mocha config files if no script found

### 3. Run the full test suite

- In a monorepo, run tests for all affected packages (the mutated package + its dependents)
- **If tests fail:** Stop immediately, report the failures, tell the user to fix them. No mutants introduced.
- **If tests pass:** Proceed to mutation phase.

## Mutation Operators

### Split: 70% classical / 30% semantic

### Classical operators (70%)

Deterministic, well-known transformations:

| Operator | Example |
|---|---|
| Conditional flip | `===` to `!==`, `>` to `<=`, `&&` to `\|\|` |
| Arithmetic swap | `+` to `-`, `*` to `/` |
| Remove return value | `return x` to `return undefined` |
| Delete function call | `logger.warn(msg)` to *(removed)* |
| Negate boolean | `true` to `false`, `!x` to `x` |
| Boundary shift | `x > 0` to `x >= 0`, `i < len` to `i <= len` |
| Empty collection | `return [items]` to `return []`, `return {...}` to `return {}` |
| Remove exception | `throw new Error(...)` to *(removed)* |

### Semantic/LLM-powered operators (30%)

Claude reasons about the code and introduces plausible-but-wrong changes:

| Operator | Example |
|---|---|
| Off-by-one | Loop bounds, array indices, slice arguments |
| Wrong variable | Swap a variable for a similarly-named one in scope |
| Incorrect default | Change a default parameter to a plausible but wrong value |
| Subtle logic error | Reorder conditions, swap early-return logic |
| Missing null check | Remove a guard clause that protects against null/undefined |

For semantic mutations, Claude reads the surrounding context to produce mutations that a real developer might accidentally introduce — not random noise.

### Per-function budget

The number of mutants per function is determined by function complexity:

- **Simple functions** (few branches, linear flow, short body): 1-2 mutants
- **Moderate functions** (some branching, loops, moderate length): 3-5 mutants
- **Complex functions** (deep nesting, multiple branches, long body, error handling): 6-10 mutants

Complexity is assessed by cyclomatic indicators: branch count, nesting depth, number of operators/expressions, and function length. Effort is concentrated where bugs are most likely to hide.

## Execution & Parallelism

### Workflow after green baseline passes

1. **File collection** — based on mode, collect all target source files
2. **Mutant generation** — for each file, Claude reads the source, identifies functions, assesses complexity, and generates a mutant plan (which mutations to apply, in what order). No code is modified yet.
3. **Test mapping** — for each source file, determine the scoped test command:
   - Map `src/utils/retry.ts` to `retry.test.ts` (or equivalent)
   - In monorepo: run the scoped test within the correct package, plus scoped tests in dependent packages that exercise the mutated code
   - Use test runner flags for scoping (e.g., Jest `--testPathPattern`, Vitest file path argument, Mocha `--grep`)
   - If no specific test file mapping can be determined, fall back to the package-level test suite
4. **Batching** — files are distributed across subagents. Each subagent gets a batch of files with their mutant plans and scoped test commands.
5. **Subagent fan-out** — subagents dispatched in parallel. Each subagent:
   - For each mutant in its batch:
     - Applies the mutation (edits the source file)
     - Runs the **scoped** test command (only tests relevant to that file)
     - Records result: **killed** (test failed) or **survived** (test passed)
     - Reverts the mutation via `git checkout -- <file>`
   - Returns results to the main agent
6. **Aggregation** — main agent collects results, renders report

### Why scoped tests matter for parallelism

- Running full test suites in parallel would cause side effects (shared state, port conflicts, resource contention)
- Scoped test commands ensure each subagent only touches the tests relevant to its mutated file
- This also makes each mutation cycle faster

### Concurrency limits

- Max 5 subagents at a time
- Each subagent handles ~5-10 files depending on total file count
- For small runs (< 5 files), skip subagents entirely and run sequentially in main context

### Safety

- Every mutation is reverted immediately after testing via `git checkout -- <file>`
- If a subagent fails or times out, its files are reported as "not tested" rather than silently dropped

## Report Output

Terminal output with three sections:

### 1. Summary header

```
Mutation Testing Report
═══════════════════════
Mode: diff (vs main)
Files tested: 12
Total mutants: 47
Killed: 38 (80.9%)
Survived: 9 (19.1%)
Not tested: 0
```

### 2. Surviving mutants

```
Surviving Mutants
─────────────────
src/utils/retry.ts:24
  Function: retryOperation
  Mutation: [BOUNDARY] changed `i < maxRetries` → `i <= maxRetries`

src/utils/retry.ts:31
  Function: retryOperation
  Mutation: [SEMANTIC] removed null check on `options.onRetry`

src/services/auth.ts:55
  Function: validateToken
  Mutation: [CONDITIONAL] flipped `===` → `!==`
```

Each entry shows file, line number, function name, mutation category (classical type or SEMANTIC), and a human-readable description of what was changed.

### 3. Per-file breakdown

```
File Breakdown
──────────────
src/utils/retry.ts        5 mutants   3 killed   2 survived
src/services/auth.ts      8 mutants   7 killed   1 survived
src/services/user.ts      6 mutants   6 killed   0 survived
```

### Mutation score interpretation

Included at the bottom of the report:
- **80%+**: Good coverage
- **60-80%**: Gaps worth investigating
- **<60%**: Significant test gaps

## Non-goals

- No persistent report files — terminal only (user can request a write-out separately)
- No automatic generation of missing tests — report only
- No integration with external mutation testing tools (Stryker, etc.)
- No support for non-JS/TS languages
