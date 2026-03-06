# GitHub Copilot Instructions

This repository contains a modular set of instruction files designed to guide GitHub Copilot in generating high-quality, maintainable code across different technology stacks and concerns.

## Philosophy

The instructions are organized by scope and specificity, allowing you to compose them based on your project needs. Each file focuses on a specific domain without duplicating guidance from other files.

## Instruction Files

### 📘 software-craft-instructions.md
**Purpose:** Language-agnostic software engineering best practices

**Scope:** Universal engineering principles applicable to any programming language or framework

**Key Topics:**
- **Modularity** - Single responsibility, composition over inheritance, file size guidelines
- **Testing** - TDD, test organization, explicit test doubles (dummies, stubs, spies, mocks, fakes)
- **Dependency Inversion** - Abstraction-based design, dependency injection patterns
- **Cyclomatic Complexity** - Guard clauses, complexity reduction, function extraction
- **Code Documentation** - Function/method documentation standards, inline comments
- **Expressive Code** - Naming conventions, boolean prefixes, verb/noun usage
- **Type Safety** - Contract modeling, immutability, compile-time validation
- **Error Handling** - Async/sync error patterns, logging, user-friendly messages

**Use When:** Starting any software project, regardless of language or stack

**Detailed Coverage:**
- Test doubles organization: `tests/doubles/{dummies,stubs,spies,mocks,fakes}/`
- Explicit test double creation over framework mock functions
- Comprehensive examples with ❌ Bad and ✅ Good patterns
- AAA (Arrange-Act-Assert) testing pattern
- Edge case and error path testing

---

### 📗 ts-instructions.md
**Purpose:** TypeScript-specific language features and compiler configuration

**Scope:** TypeScript language, type system, and tooling decisions

**Key Topics:**
- **Compiler Configuration** - Strict mode settings, safety options, CI integration
- **Type Modeling** - Interface vs type usage, discriminated unions, literal types
- **Boundary Safety** - Runtime validation at system edges, unknown over any
- **API Typing** - Explicit return types, exhaustiveness checking, generic constraints
- **Nullability** - Explicit null/undefined handling, optional semantics
- **Modules** - Import/export patterns, public API surface management
- **Enums & Constants** - `as const` preference, derived union types
- **Error Typing** - Structured errors, typed result objects
- **Testing** - Type-safe test fixtures, suppression comment hygiene
- **Tooling** - Type checking in CI, lint integration, upgrade cadence

**Use When:** Working on TypeScript projects (additive to software-craft-instructions.md)

**Does NOT Cover:** General testing strategy, modularity, DI patterns (see software-craft-instructions.md)

---

### 📕 react-instructions.md
**Purpose:** React-specific patterns, hooks, and component architecture

**Scope:** React framework, component design, and ecosystem best practices

**Key Topics:**
- **Component Architecture** - Function components, prop design, discriminated unions
- **Hooks Best Practices** - Custom hooks, dependency arrays, useCallback/useMemo usage
- **State Management** - Local vs global state, functional updates, derived state avoidance
- **Performance** - React.memo strategies, code splitting with lazy/Suspense
- **Event Handling** - Typed event handlers, React event types
- **Data Fetching** - Server state libraries (TanStack Query, SWR), loading/error states
- **Forms** - Controlled components, validation patterns, accessibility
- **Accessibility** - Semantic HTML, ARIA attributes, keyboard navigation, screen readers
- **TypeScript Integration** - Component typing, generics, avoiding React.FC
- **Testing** - React Testing Library, user behavior testing, accessible queries
- **Error Boundaries** - Error handling, fallback UI, monitoring integration
- **Project Structure** - Feature-based organization, file co-location
- **Styling** - CSS Modules, CSS-in-JS, theming with custom properties
- **Development Tools** - Strict Mode, React DevTools, ESLint configuration

**Use When:** Building React applications (complements software-craft and ts-instructions)

**Does NOT Cover:** General testing patterns, type safety principles, error handling fundamentals (see other files)

---

### 📄 the-one-instructions.md
**Purpose:** Original comprehensive instructions (legacy/reference)

**Scope:** Full-stack guidance including React, TypeScript, Tailwind, Vite, and general practices

**Status:** This file contains the original monolithic instructions before they were refactored into focused, modular files. It includes framework-specific guidance (React, Tailwind CSS, Vite) alongside general engineering practices.

**Use When:** You need the complete original instruction set or are working with a full React/TypeScript/Tailwind/Vite stack.

**Note:** The modular instruction files (software-craft, ts-instructions, react-instructions) were extracted from this file to enable better composability and reuse across different project types.

---

## How to Use These Instructions

### For a React + TypeScript Project:
Use all three modular files:
1. `software-craft-instructions.md` - Engineering fundamentals
2. `ts-instructions.md` - TypeScript specifics
3. `react-instructions.md` - React patterns

### For a TypeScript Backend Project:
Use two files:
1. `software-craft-instructions.md` - Engineering fundamentals
2. `ts-instructions.md` - TypeScript specifics

### For a Non-TypeScript React Project:
Use two files:
1. `software-craft-instructions.md` - Engineering fundamentals
2. `react-instructions.md` - React patterns

### For a Python/Go/Java Project:
Use one file:
1. `software-craft-instructions.md` - Engineering fundamentals (language-agnostic)

### For Any Project:
At minimum, use:
- `software-craft-instructions.md` - Core engineering practices apply universally

## Key Design Principles

### 1. **No Duplication**
Each file covers distinct concerns. For example:
- Testing strategy is in `software-craft-instructions.md`
- TypeScript type safety is in `ts-instructions.md`
- React-specific testing (RTL patterns) is in `react-instructions.md`

### 2. **Additive Composition**
Files build on each other without contradiction:
- `ts-instructions.md` refines general type safety from software-craft
- `react-instructions.md` adds framework-specific testing patterns to general testing strategy

### 3. **Independence**
Files don't explicitly reference each other, allowing flexible use across different project contexts.

### 4. **Examples Over Rules**
Each file includes extensive ❌ Bad and ✅ Good code examples to demonstrate principles in practice.

## Test Double Guidelines

The `software-craft-instructions.md` file emphasizes creating **explicit test doubles** over using framework mock functions. Test doubles should be organized by type:

```
tests/
  doubles/
    dummies/     # Objects passed but never used
    stubs/       # Provide canned responses
    spies/       # Record call information
    mocks/       # Pre-programmed with expectations
    fakes/       # Working implementations with shortcuts
```

**Why Explicit Doubles?**
- Framework independence
- Reusable across tests
- Type-safe
- Clear semantics
- Easier to understand test intent

## Contributing

When adding new instructions:
1. Determine the appropriate scope (language-agnostic, language-specific, framework-specific)
2. Avoid duplicating existing guidance
3. Include concrete examples with ❌ Bad and ✅ Good patterns
4. Ensure new instructions don't contradict existing files
5. Keep files focused on their specific domain

## Version Information

These instructions reflect current best practices as of March 2026, including:
- React 18+ patterns (hooks, Suspense, concurrent features)
- TypeScript 5.x strict configuration
- Modern testing approaches (React Testing Library, explicit test doubles)
- Contemporary state management (Zustand, Jotai, TanStack Query)
- Current accessibility standards (ARIA, semantic HTML, WCAG)

## License

These instructions are provided as-is for use with GitHub Copilot to improve code generation quality.

