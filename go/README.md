# Go Engineering Standards & Code Style (AI-Ready)

**Version:** 1.0 • **Date:** 2025-10-07 • **Owner:** Internal Go Team
**Purpose:** Single source of truth for Go style and standards. Optimized for LLM-based Code Review (no ambiguity, stable anchors, explicit rules).

> This document is a normalized export of our internal Confluence page “GoLang — Guidelines” with light editorial fixes for clarity and consistency. Use the canonical headings and anchors below when you reference rules in code review comments (_e.g._, “see §ERR-003”).

---

## Table of Contents

- [1. General Guidelines](#1-general-guidelines)
  - [GL-001 Pointers to interfaces](#gl-001-pointers-to-interfaces)
  - [GL-002 Verify interface compliance](#gl-002-verify-interface-compliance)
  - [GL-003 Zero-value mutexes are valid](#gl-003-zero-value-mutexes-are-valid)
  - [GL-004 Copy slices/maps at boundaries](#gl-004-copy-slicesmaps-at-boundaries)
  - [GL-005 Returning slices/maps](#gl-005-returning-slicesmaps)
  - [GL-006 Defer for cleanup](#gl-006-defer-for-cleanup)
  - [GL-007 Channel size is one or none](#gl-007-channel-size-is-one-or-none)
  - [GL-008 Start enums at 1; reserve 0 as Unknown](#gl-008-start-enums-at-1-reserve-0-as-unknown)
  - [GL-009 Always use `time` package](#gl-009-always-use-time-package)
  - [GL-010 `time.Time` for instants; `time.Duration` for periods](#gl-010-timetime-for-instants-timeduration-for-periods)
  - [GL-011 Use `time.Time`/`time.Duration` with external systems](#gl-011-use-timetimetimeduration-with-external-systems)
- [2. Errors](#2-errors)
  - [ERR-001 Error types & matching](#err-001-error-types--matching)
  - [ERR-002 Error wrapping (`%w` vs `%v`)](#err-002-error-wrapping-w-vs-v)
  - [ERR-003 Error naming](#err-003-error-naming)
  - [ERR-004 Handle errors once](#err-004-handle-errors-once)
  - [ERR-005 Handle type assertion failures](#err-005-handle-type-assertion-failures)
  - [ERR-006 Don’t panic in production](#err-006-dont-panic-in-production)
- [3. Concurrency & Lifecycle](#3-concurrency--lifecycle)
  - [CON-001 Prefer `go.uber.org/atomic` wrappers](#con-001-prefer-gouberorgatomic-wrappers)
  - [CON-002 Avoid mutable globals; use DI](#con-002-avoid-mutable-globals-use-di)
  - [CON-003 Avoid embedding types in public structs](#con-003-avoid-embedding-types-in-public-structs)
  - [CON-004 Avoid goroutine leaks](#con-004-avoid-goroutine-leaks)
  - [CON-005 Wait for goroutines to exit](#con-005-wait-for-goroutines-to-exit)
  - [CON-006 No goroutines in `init()`](#con-006-no-goroutines-in-init)
  - [CON-007 No “magic” env vars (wire config)](#con-007-no-magic-env-vars-wire-config)
- [4. Performance (hot paths only)](#4-performance-hot-paths-only)
  - [PERF-001 Prefer `strconv` over `fmt` for conversions](#perf-001-prefer-strconv-over-fmt-for-conversions)
  - [PERF-002 Avoid repeated string→[]byte conversions](#perf-002-avoid-repeated-stringbyte-conversions)
  - [PERF-003 Specify container capacity](#perf-003-specify-container-capacity)
    - [PERF-003a Map capacity hints](#perf-003a-map-capacity-hints)
    - [PERF-003b Slice capacity](#perf-003b-slice-capacity)
- [5. Process & Program Structure](#5-process--program-structure)
  - [PROC-001 Avoid `init()` (with narrow exceptions)](#proc-001-avoid-init-with-narrow-exceptions)
  - [PROC-002 Exit only in `main()`](#proc-002-exit-only-in-main)
  - [PROC-003 Exit once](#proc-003-exit-once)
- [6. Style](#6-style)
  - [STY-001 Line length soft limit 120](#sty-001-line-length-soft-limit-120)
  - [STY-002 Be consistent](#sty-002-be-consistent)
  - [STY-003 Group similar declarations](#sty-003-group-similar-declarations)
  - [STY-004 Package names](#sty-004-package-names)
  - [STY-005 Function names](#sty-005-function-names)
  - [STY-006 Import aliasing](#sty-006-import-aliasing)
  - [STY-007 Function grouping & order](#sty-007-function-grouping--order)
  - [STY-008 Reduce nesting & unnecessary `else`](#sty-008-reduce-nesting--unnecessary-else)
  - [STY-009 Top-level var declarations](#sty-009-top-level-var-declarations)
  - [STY-010 Embedding order & rationale](#sty-010-embedding-order--rationale)
  - [STY-011 Local var declarations (`:=` vs `var`)](#sty-011-local-var-declarations--vs-var)
  - [STY-012 Nil slices are valid](#sty-012-nil-slices-are-valid)
  - [STY-013 Reduce scope](#sty-013-reduce-scope)
  - [STY-014 Avoid naked parameters](#sty-014-avoid-naked-parameters)
  - [STY-015 Use raw string literals](#sty-015-use-raw-string-literals)
  - [STY-016 Initialize structs by field name; omit zeroes](#sty-016-initialize-structs-by-field-name-omit-zeroes)
  - [STY-017 `var T` for zero-value structs; `&T{}` over `new(T)`](#sty-017-var-t-for-zero-value-structs-t-over-newt)
  - [STY-018 Format strings outside Printf](#sty-018-format-strings-outside-printf)
  - [STY-019 Name Printf-style functions (`...f`)](#sty-019-name-printf-style-functions-f)
- [7. Patterns](#7-patterns)
  - [PAT-001 Table-driven tests](#pat-001-table-driven-tests)
  - [PAT-002 Avoid complexity in table tests](#pat-002-avoid-complexity-in-table-tests)
  - [PAT-003 Functional options](#pat-003-functional-options)
- [Appendix A. Lint/Vet expectations](#appendix-a-lintvet-expectations)
- [Appendix B. Example snippets](#appendix-b-example-snippets)

---

## 1. General Guidelines

### GL-001 Pointers to interfaces

**Rule:** Pass interfaces by value; you almost never need `*Interface`. If methods must mutate receiver state, make the underlying concrete type a pointer, not the interface.  
**Why:** Interface value = (type, data). Passing `*Interface` adds confusion without benefit.

### GL-002 Verify interface compliance

**Rule:** For exported types (or sets of implementations), assert interface compliance at compile time:  

```go
var _ http.Handler = (*Handler)(nil)
```

Use the zero value on the right-hand side (nil for pointers/slices/maps, empty struct for concrete structs).

### GL-003 Zero-value mutexes are valid

**Rule:** Use `sync.Mutex`/`sync.RWMutex` as value fields, not pointers; don’t embed mutexes (even on unexported structs).  

```go
type S struct{ mu sync.Mutex } // good
```

### GL-004 Copy slices/maps at boundaries

**Rule:** Slices and maps carry internal pointers; copy when exposing across package/API boundaries to protect invariants and avoid aliasing.

### GL-005 Returning slices/maps

**Rule:** Return defensive copies if the caller must not observe internal state.

```go
func (s *Stats) Snapshot() map[string]int {
    s.mu.Lock(); defer s.mu.Unlock()
    out := make(map[string]int, len(s.counters))
    for k, v := range s.counters { out[k] = v }
    return out
}
```

### GL-006 Defer for cleanup

**Rule:** Prefer `defer` for resource release (`Close`, `Unlock`, `Stop`); the readability win outweighs tiny overhead. Use only measured data to justify avoiding `defer`.

### GL-007 Channel size is one or none

**Rule:** Channels are unbuffered by default; prefer size `0` or `1`. Any larger buffer requires clear backpressure reasoning and load behavior analysis.

### GL-008 Start enums at 1; reserve 0 as Unknown

**Rule:** Define `Unknown=0`, real values start at 1 to align with zero-values.  

```go
type Op int
const (
    Unknown Op = iota
    Add
    Sub
    Mul
)
```

### GL-009 Always use `time` package

**Rule:** Calendar/time is non-trivial. Always use `time` helpers instead of hand-rolled logic.

### GL-010 `time.Time` for instants; `time.Duration` for periods

**Rule:** Use `time.Time` for points-in-time and compare with methods; use `time.Duration` for delays/intervals. Prefer `AddDate` to move calendar days; prefer `Add(24*time.Hour)` to move exact 24h.

### GL-011 Use `time.Time`/`time.Duration` with external systems

**Rule:** Prefer these types in flags, JSON/YAML, DB drivers, etc. When unsupported (e.g., JSON durations), encode unit in field name (_e.g._, `timeoutMillis`). Use RFC3339 for timestamps when using strings.

---

## 2. Errors

### ERR-001 Error types & matching

**Rule:** If callers must branch on error, expose a top-level `var` error (for `errors.Is`) or a custom type (for `errors.As`). Use `errors.New` for static messages; `fmt.Errorf` or custom types for dynamic context.

### ERR-002 Error wrapping (`%w` vs `%v`)

**Rule:** Wrap with `%w` to preserve the cause for matching; use `%v` to obfuscate the cause. Keep messages succinct; avoid “failed to …” piling.

```go
if err != nil { return fmt.Errorf("new store: %w", err) }
```

### ERR-003 Error naming

**Rule:** Global errors use `ErrXxx`; unexported in-package errors use `errXxx`. Custom types end with `Error`.

### ERR-004 Handle errors once

**Rule:** Don’t log and return the same error. Either handle it locally (log & degrade) **or** propagate (wrapped).

### ERR-005 Handle type assertion failures

**Rule:** Always use the “comma ok” form.

```go
s, ok := v.(string); if !ok { /* handle */ }
```

### ERR-006 Don’t panic in production

**Rule:** Panic only for programmer bugs (nil deref) or fatal init failures. Prefer returning `error`. In tests, prefer `t.Fatal`/`t.FailNow`.

---

## 3. Concurrency & Lifecycle

### CON-001 Prefer `go.uber.org/atomic` wrappers

**Rule:** Use atomic wrappers (e.g., `atomic.Bool`) to avoid raw-typed `sync/atomic` hazards.

### CON-002 Avoid mutable globals; use DI

**Rule:** Wire dependencies explicitly via constructors/config structs; don’t read env deep inside packages.

### CON-003 Avoid embedding types in public structs

**Rule:** Don’t embed concrete types in exported structs—leaks API and limits evolution. Prefer fields and explicit delegation.

### CON-004 Avoid goroutine leaks

**Rule:** Every goroutine must have a shutdown signal and a way to wait for completion (`done` chan or `sync.WaitGroup`).

### CON-005 Wait for goroutines to exit

**Rule:** Provide `Shutdown/Close/Stop` that signals and blocks until goroutine exits.

### CON-006 No goroutines in `init()`

**Rule:** If background work is needed, expose a type that manages lifetime; start on demand and stop explicitly.

### CON-007 No “magic” env vars (wire config)

**Rule:** Avoid implicit `os.Getenv` at package scope. Pass config values via `Config` structs/params.

---

## 4. Performance (hot paths only)

### PERF-001 Prefer `strconv` over `fmt` for conversions

```go
_ = strconv.Itoa(n) // faster than fmt.Sprint(n)
```

### PERF-002 Avoid repeated string→[]byte conversions

```go
b := []byte("const payload")
w.Write(b) // avoid converting the same literal in a loop
```

### PERF-003 Specify container capacity

#### PERF-003a Map capacity hints

```go
m := make(map[string]T, nHint)
```

#### PERF-003b Slice capacity

```go
s := make([]T, 0, n) // future appends avoid realloc until cap is hit
```

---

## 5. Process & Program Structure

### PROC-001 Avoid `init()` (with narrow exceptions)

**Rule:** Prefer pure declarations or helpers called from `main()`. Avoid I/O, env, or ordering dependencies in `init()`.

### PROC-002 Exit only in `main()`

**Rule:** Library and helper funcs return errors. Only `main()` calls `log.Fatal`/`os.Exit`.

### PROC-003 Exit once

**Rule:** Centralize exit in one place (commonly `main()` delegating to `run()` that returns an error/exit code).

---

## 6. Style

### STY-001 Line length soft limit 120

**Rule:** Keep lines ≤120 chars where practical.

### STY-002 Be consistent

**Rule:** Consistency within a package > personal preference.

### STY-003 Group similar declarations

**Rule:** Group related `import`, `const`, `var`, `type` blocks. Don’t mix unrelated items.

### STY-004 Package names

**Rules:**

- Lowercase, short, descriptive; avoid plurals and “common/util/lib/shared”.
- Use abbreviations judiciously; underscores only if needed for clarity.
- Names should rarely require import renaming at call sites.

### STY-005 Function names

**Rule:** Use MixedCaps. Tests may use underscores to group cases (_e.g._, `TestFoo_Bar`).

### STY-006 Import aliasing

**Rule:** Use alias only when package name mismatches last path element or there’s a conflict.

### STY-007 Function grouping & order

**Rule:** Group by receiver; exported first; constructors near type; helpers last; rough call order.

### STY-008 Reduce nesting & unnecessary `else`

**Rule:** Return early on error and invalid input; avoid deep nesting; collapse trivial if/else into single assignment.

### STY-009 Top-level var declarations

**Rule:** Use `var` without explicit type unless you need a different interface type on the left.

### STY-010 Embedding order & rationale

**Rule:** If embedding (rare), put embedded fields first, separated by a blank line. Do not embed mutexes.

### STY-011 Local var declarations (`:=` vs `var`)

**Rule:** Prefer `:=` for explicit initializations; use `var` when emphasizing zero-value intent.

### STY-012 Nil slices are valid

**Rules:**

- Return `nil` for empty slices from APIs (unless a non-nil empty slice is a contract).
- Test emptiness with `len(s)==0`, not `s==nil`.
- `var s []T` is ready for `append` without `make`.

### STY-013 Reduce scope

**Rule:** Minimize variable/const scope consistent with readability and usage.

### STY-014 Avoid naked parameters

**Rule:** Replace positional literals with named consts or custom types (especially booleans).

### STY-015 Use raw string literals

**Rule:** Prefer backticks for multi-line and strings with quotes to avoid escaping.

### STY-016 Initialize structs by field name; omit zeroes

**Rule:** Use field names and omit zero-valued fields (unless name conveys meaning in tests).

### STY-017 `var T` for zero-value structs; `&T{}` over `new(T)`

**Rule:** Prefer `var v T` and `&T{}` to keep style uniform with keyed literals.

### STY-018 Format strings outside Printf

**Rule:** Prefer `const` format strings for vet static analysis.

### STY-019 Name Printf-style functions (`...f`)

**Rule:** End custom printf-like APIs with `f` (e.g., `Wrapf`) so `go vet` can validate.

---

## 7. Patterns

### PAT-001 Table-driven tests

**Rule:** Prefer table tests with `tests` slice and `tt` cases; use subtests with `t.Run(tt.name, ...)`. Prefer explicit `give`/`want` fields.

### PAT-002 Avoid complexity in table tests

**Rule:** If cases require conditional branches, heavy mocks, or different flows, split into multiple tables or test funcs.

### PAT-003 Functional options

**Rule:** Use functional options for extensible constructors and public APIs (≥3 params or optional args). Provide sensible defaults.

```go
type Options struct {{
    Log   *zap.Logger
    Cache bool
    Name  string
}}
type Option func(*Options)

func WithCache(v bool) Option         {{ return func(o *Options){{ o.Cache = v }} }}
func WithLogger(l *zap.Logger) Option {{ return func(o *Options){{ o.Log = l }} }}
func WithName(n string) Option        {{ return func(o *Options){{ o.Name = n }} }}

func Open(addr string, opts ...Option) (*Conn, error) {{
    o := Options{{Cache: true}}
    for _, fn := range opts {{ fn(&o) }}
    // ...
    return &Conn{{}}, nil
}}
```

---

## Appendix A. Lint/Vet expectations

- `go vet` should not flag: printf names (`...f`), raw strings, nil-slice checks (`len==0`), type assertions (comma-ok), RFC3339 timestamps, missing struct tags in marshaled types.
- `golangci-lint`/`staticcheck` alignment with the above rules is expected.

## Appendix B. Example snippets

### Error Wrapping Example (ERR-002)

```go
package main

import (
  "errors"
  "fmt"
)

var ErrCouldNotOpen = errors.New("could not open")

func openThing() error {
  return ErrCouldNotOpen
}

func run() error {
  if err := openThing(); err != nil {
    return fmt.Errorf("new store: %w", err)
  }
  return nil
}

func main() {
  if err := run(); err != nil {
    fmt.Println(err)
  }
}
```

### Goroutine Lifecycle Example (CON-004, CON-005)

```go
package worker

import (
  "time"
)

type Worker struct {
  stop chan struct{}
  done chan struct{}
}

func New() *Worker {
  return &Worker{stop: make(chan struct{}), done: make(chan struct{})}
}

func (w *Worker) Start() {
  go func() {
    defer close(w.done)
    t := time.NewTicker(100 * time.Millisecond)
    defer t.Stop()
    for {
      select {
      case <-w.stop:
        return
      case <-t.C:
        // work
      }
    }
  }()
}

func (w *Worker) Shutdown() {
  close(w.stop)
  <-w.done
}
```
