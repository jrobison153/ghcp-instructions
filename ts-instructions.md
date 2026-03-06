# TypeScript Copilot Instructions

These instructions are TypeScript-specific and **additive** to `software-craft-instructions.md`.
Do not restate or weaken general engineering rules from the base file.

## Scope and Interaction with General Rules
- Apply `software-craft-instructions.md` for architecture, testing strategy, documentation, naming, and error-handling fundamentals.
- Use this file for TypeScript language, typing, compiler, and TS-tooling decisions.
- If both files mention the same topic, treat this file as a TypeScript refinement of the base rule.

## Compiler Configuration (Required Defaults)
- Keep `"strict": true`.
- Prefer enabling these safety options:
  - `"noUncheckedIndexedAccess": true`
  - `"exactOptionalPropertyTypes": true`
  - `"noImplicitOverride": true`
  - `"useUnknownInCatchVariables": true`
  - `"noFallthroughCasesInSwitch": true`
- Run type checks in CI with `tsc --noEmit`.
- Keep `skipLibCheck` off by default; enable only with documented rationale.

## Type Modeling and Contracts
- Model domain concepts with explicit types before implementation details.
- Prefer `interface` for public/extensible object contracts.
- Use `type` for unions, intersections, mapped/conditional types, and aliases.
- Prefer literal unions over broad primitives for constrained values.
- Use `readonly` and `ReadonlyArray<T>` by default for immutable data.
- Prefer `unknown` over `any`; narrow at usage sites.

## Boundary Safety (Runtime + Types)
- Treat external input (HTTP, storage, env, user input) as `unknown`.
- Validate/parse inputs at boundaries before mapping to domain types.
- Avoid direct assertions from untrusted data to domain models.
- Centralize boundary schemas/parsers to reduce drift.

## API and Function Typing
- Add explicit return types for exported/public functions.
- Allow local inference for short internal functions when clear.
- Use discriminated unions for variant/state modeling.
- Enforce exhaustiveness with `never` checks in `switch` handling.
- Keep generics constrained and readable; avoid over-engineered type logic.

## Nullability and Optional Semantics
- Make nullability explicit (`T | null`, `T | undefined`) based on meaning.
- Use optional properties only when omission is semantically distinct.
- Avoid non-null assertions (`!`) except in narrow, justified cases.

## Modules and Public Surface
- Use `import type` / `export type` for type-only symbols where supported.
- Keep public exports intentional; avoid accidental API growth.
- Prefer curated barrels over broad `export *` on large surfaces.
- Separate internal implementation types from public API types.

## Enums, Constants, and Derived Types
- Prefer `as const` objects plus derived unions for app-level constants.
- Use `enum` primarily for interop needs; use explicit values when used.
- Derive type unions from runtime constants to prevent divergence.

## Error and Result Typing
- Use structured error shapes or typed error classes.
- Preserve root cause when rethrowing (e.g., with `cause`).
- For expected domain failures, prefer typed result objects/unions.

## TypeScript in Tests (Type-Specific Additions)
- Keep test fixtures/factories strongly typed.
- Avoid `as any` in tests; if unavoidable, isolate and document intent.
- Prefer `@ts-expect-error` to `@ts-ignore`, with a short reason comment.

## Tooling Hygiene
- Keep `typescript`, `@types/*`, and TS-aware lint rules up to date.
- Track and periodically remove suppression comments and temporary type workarounds.
- Add compile-time type tests for critical utility types when complexity is non-trivial.

## Practical Defaults
- Be explicit and narrow at system boundaries.
- Prefer simpler, maintainable type designs over maximal type cleverness.
- Optimize for long-term readability and refactor safety.

