---
name: go-style-guide
description: Go programming best practices and style guide for writing, reviewing, refactoring, and modernizing Go code. Use for Go code reviews, concurrency patterns, error handling, API design, performance-sensitive code, testing, linting, Go-version compatibility, and idiomatic Go style decisions.
---

# Go Best Practices

Styles are the conventions that govern our code. The term style is a bit of
misnomer, since these conventions cover far more than just source file
formatting—gofmt handles that for us.

The goal of this guide is to manage this complexity by describing in detail the
Dos and Don'ts of writing Go code at Direkt. These rules exist to keep the code
base manageable while still allowing developers to use Go language features
productively. Created based on the Uber Go Style Guide.

## When to Apply

Reference these guidelines when:
- Writing new Go code
- Reviewing code for performance issues
- Refactoring existing Go code
- Optimizing performance, latency, or allocations, while avoiding micro-optimizations outside hot paths
- Reviewing code for Go-version-sensitive behavior
- Modernizing older Go code to current idioms
- Editing code for readability, maintainability, and consistency

## Version-sensitive guidance

Before applying advice, consider the project’s target Go version.

Important version-sensitive areas include:

- Go 1.20+: `errors.Join` and multiple `%w` wrapping
- Go 1.22+: loop variables are scoped per iteration
- Go 1.23+: timer and ticker garbage collection and channel behavior changed
- Go 1.25+: `sync.WaitGroup.Go`
- Go 1.26+: `fmt.Errorf("x")` allocation behavior and `errors.AsType`

If the target Go version is unknown, prefer guidance compatible with the two most recent stable Go releases, and call out compatibility notes when relevant.

Performance guidance should be applied primarily to hot paths or allocation-sensitive code. Avoid recommending broad rewrites based only on old benchmark numbers. Prefer explaining the likely trade-off and recommending measurement with benchmarks where the impact matters.

The guidance is version-aware. When reviewing or generating code, account for the target Go version, especially for behavior introduced in Go 1.20 through Go 1.26 such as errors.Join, loop variable semantics, timer behavior, WaitGroup.Go, and fmt.Errorf allocation changes.

## Rule Categories by Priority

| Priority | Category | Impact | Examples |
|----------|----------|--------|----------|
| 1 | Correctness & Error Handling | CRITICAL | `panic.md`, `error-once.md`, `error-wrap.md`, `exit-once.md` |
| 2 | Concurrency | CRITICAL | `goroutine-forget.md`, `goroutine-exit.md`, `channel-size.md`, `atomic.md` |
| 3 | APIs & Types | HIGH | `interface-pointer.md`, `interface-compliance.md`, `functional-option.md`, `struct-tag.md` |
| 4 | Performance | HIGH | `performance.md`, `container-capacity.md`, `strconv.md`, `string-byte-slice.md` |
| 5 | Readability & Style | MEDIUM | `var-scope.md`, `nest-less.md`, `decl-group.md`, `function-order.md` |
| 6 | Testing | MEDIUM | `table-driven-tests.md`, `test-table.md` |
| 7 | Tooling | LOW | `lint.md`, `printf-name.md`, `struct-field-key.md` |

## Quick Reference

### 1. Correctness & Error Handling (CRITICAL)

- `panic.md` - Avoid panics in production; return errors.
- `error-type.md`, `error-wrap.md`, `error-once.md`, `error-name.md` - Use errors consistently; wrap with context; handle once.
- `exit-main.md`, `exit-once.md` - Exit in `main()`; keep exit logic in one place.

### 2. Concurrency (CRITICAL)

- `goroutine-forget.md` - Don’t fire-and-forget goroutines.
- `goroutine-exit.md` - Always provide a way to wait for goroutines to exit.
- `goroutine-init.md` - Don’t start goroutines in `init()`.
- `channel-size.md` - Channel size should be one or unbuffered.
- `mutex-zero-value.md` - Zero-value mutexes are valid (avoid pointers).
- `atomic.md` - Prefer typed atomics (`sync/atomic`, Go 1.19+).

### 3. APIs & Types (HIGH)

- `interface-pointer.md`, `interface-compliance.md`, `interface-receiver.md` - Interface and receiver best practices.
- `functional-option.md` - Prefer functional options for extensible APIs.
- `struct-tag.md` - Add tags for marshaled struct fields.
- `embed-public.md`, `struct-embed.md` - Be careful with embedding (especially in public structs).

### 4. Performance (HIGH)

- `performance.md` - Apply performance guidance on hot paths.
- `container-capacity.md` - Pre-size slices/maps where possible.
- `container-copy.md` - Copy slices/maps at boundaries to avoid aliasing bugs.
- `strconv.md` - Prefer `strconv` over `fmt` for primitives.
- `string-byte-slice.md` - Avoid repeated string-to-`[]byte` conversions.

### 5. Readability & Style (MEDIUM)

- `consistency.md`, `line-length.md` - Optimize for readability and consistency.
- `decl-group.md`, `import-group.md` - Group similar declarations and imports.
- `function-name.md`, `package-name.md`, `builtin-name.md` - Naming conventions.
- `function-order.md`, `nest-less.md`, `else-unnecessary.md`, `var-scope.md` - Reduce nesting and keep scope tight.
- `var-decl.md`, `slice-nil.md`, `map-init.md` - Prefer idiomatic declarations/initialization.
- `struct-field-key.md`, `struct-field-zero.md`, `struct-pointer.md`, `struct-zero.md` - Struct initialization conventions.

### 6. Testing (MEDIUM)

- `test-table.md`, `table-driven-tests.md` - Table tests, subtests, and when to avoid over-complicated tables.

### 7. Tooling (LOW)

- `lint.md` - Lint consistently across the codebase.
- `printf-name.md`, `printf-const.md` - Enable `go vet` format-string checking.

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/SUMMARY.md
AGENTS.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and references

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
