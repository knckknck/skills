# Error Types

There are few options for declaring errors.
Consider the following before picking the option best suited for your use case.

- Does the caller need to match the error so that they can handle it?
  If yes, we must support the [`errors.Is`], [`errors.As`], or [`errors.AsType`]
  functions by declaring a top-level error variable or a custom type.
- Is the error message a static string,
  or is it a dynamic string that requires contextual information?
  For static messages, prefer [`errors.New`] because it communicates a plain
  static error. For dynamic messages or wrapping, typically use [`fmt.Errorf`]
  or a custom error type.
- Are we propagating a new error returned by a downstream function?
  If so, see the [section on error wrapping](error-wrap.md).

[`errors.Is`]: https://pkg.go.dev/errors#Is
[`errors.As`]: https://pkg.go.dev/errors#As
[`errors.AsType`]: https://pkg.go.dev/errors#AsType

| Error matching? | Error Message | Guidance                            |
|-----------------|---------------|-------------------------------------|
| No              | static        | [`errors.New`]                      |
| No              | dynamic       | [`fmt.Errorf`]                      |
| Yes             | static        | top-level `var` with [`errors.New`] |
| Yes             | dynamic       | custom `error` type                 |

[`errors.New`]: https://pkg.go.dev/errors#New
[`fmt.Errorf`]: https://pkg.go.dev/fmt#Errorf

## Static vs dynamic errors

Use [`errors.New`] for an error with a static string.
Export this error as a variable to support matching it with `errors.Is`
if the caller needs to match and handle this error.

Starting in Go 1.26, `fmt.Errorf("x")` allocates less and generally matches the
allocation behavior of `errors.New("x")` for unformatted strings. This removes a
performance reason to avoid `fmt.Errorf` in that case, but it does not change the
style recommendation: prefer `errors.New` for static errors because it clearly
communicates that no formatting or wrapping is intended.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

For an error with a dynamic string,
use [`fmt.Errorf`] if the caller does not need to match it,
and a custom `error` if the caller does need to match it.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  notFound, ok := errors.AsType[*NotFoundError](err)
  if ok {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

For Go versions before 1.26, use `errors.As` instead of `errors.AsType`:

```go
var notFound *NotFoundError
if errors.As(err, &notFound) {
  // handle the error
}
```

## Error contracts and API stability

Exported error variables and types become part of the public API of the package.
Callers may depend on them with `errors.Is`, `errors.As`, or `errors.AsType`, so
changing or removing them can be a breaking change.

Be intentional about what errors you expose. Prefer unexported errors unless
callers need to handle the condition differently.
