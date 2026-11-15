---
name: go-enum
description: Use when designing or reviewing Go enums with validated parsing
  and serialization, or when replacing magic strings that have a fixed set
  of values. Guides a struct-based enum type that rejects unknown values in
  text, JSON, and SQL.
---

# Go Enums

Use this pattern to avoid magic strings. A magic string is a literal, such
as `"ready"`, that code compares or stores directly. The compiler cannot
check its spelling, so a mistake fails only at run time.

**Before:**

```go
if p.State == "raedy" { // Compiles, and is never true.
```

**After:**

```go
if p.State == StateReady {
```

With this pattern, code uses predefined values that the compiler checks.
Unknown strings are rejected where they enter the program.

Use the pattern when a value has a fixed set of spellings and needs validated
construction. Do not use it for an open-ended string.

## Define the Type

Use a small struct with an unexported string field. A named string type is
not enough, because callers can still create any value with a literal or a
conversion, such as `State("unknown")`. With an unexported field, code
outside the package can obtain a `State` only from a predefined value or
from the parser.

```go
// State describes a project's lifecycle. Use ParseState to create one.
type State struct{ value string }

// Predefined states are shared values. Callers must not reassign them.
var (
	StateReady    = State{value: "ready"}
	StateDeleting = State{value: "deleting"}
)

// ParseState accepts only recognized states, with exact spelling.
func ParseState(value string) (State, error) {
	switch value {
	case "ready", "deleting":
		return State{value: value}, nil
	default:
		return State{}, fmt.Errorf("invalid state %q", value)
	}
}

func (s State) String() string { return s.value }
```

- **Parser:** Provide one parser that accepts only recognized spellings.
  Compare the input with fixed spellings, not with the predefined variables,
  because a variable can be reassigned.
- **`String()`:** Use it for logs and messages. It returns the stored value
  without checking it, so it returns an empty string for the zero value. To
  serialize the enum, use one of the interfaces below.
- **Predefined values:** They are variables, not constants. Treat them as
  read-only. If callers need bindings that cannot be reassigned, provide
  functions that return the values.
- **Zero value:** Go allows `State{}` without the parser. Document whether
  the zero value is valid. Unless the domain permits it, reject it when you
  serialize or persist the enum.

## Validate at Each Boundary

A boundary is a place where the enum is read from another form or written to
one, such as JSON or a database column. Implement only the boundaries that
the type uses. These rules apply to every boundary:

- Call `ParseState` in each boundary method. Do not repeat the list of valid
  spellings.
- Validate on output as well as on input. Validation on output rejects the
  zero value.
- Change the receiver only after parsing succeeds.

Choose the interfaces from how the enum is used. Each example is independent:

| You want | Implement | Example |
| --- | --- | --- |
| Text encoding: flags, YAML, XML, map keys, or JSON strings | `encoding.TextMarshaler`, `encoding.TextUnmarshaler` | [Text](references/text.md) |
| JSON encoding that rejects `null` | `json.Marshaler`, `json.Unmarshaler` | [JSON](references/json.md) |
| Storage in a database | `sql.Scanner`, `driver.Valuer` | [SQL](references/sql.md) |

Read the example for each row that applies.

With only the text interfaces, `encoding/json` accepts `null` without an
error, because it does not call `UnmarshalText` for `null`.

## Test and Refactor

Test these cases for each boundary that you implement:

- Each valid spelling survives a round trip.
- Input rejects an unknown spelling, an empty string, and a spelling with
  the wrong case.
- The receiver stays unchanged after each rejected input.
- Output returns an error for the zero value.
- JSON input rejects `null`. Database input rejects NULL and a value of
  another type.

Preserve the existing JSON and database spellings during refactors. Keep the
implementation specific to the enum. A generic framework is unnecessary.
