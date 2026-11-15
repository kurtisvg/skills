---
name: code-style
description: Use when editing or reviewing code, tests, or embedded scripts
  and SQL. Guides formatting and readability, including line wrapping, naming,
  comments, helpers, and test layout.
---

# Code Style

Apply these preferences to the code you change. Preserve behavior. Do not
reformat unrelated code. When the user corrects one line, find the
preference behind the correction. Apply that preference to the rest of the
current change.

Language syntax and required formatters take precedence over these
preferences. For Go, run gofmt. Then check what gofmt does not change, such
as long lines and the contents of string literals.

## Wrapping and Indentation

Aim for lines of about 88 characters. This is a target, not a hard limit.
Keep short signatures and calls on one line when they remain easy to read.

The prime directive of line-wrapping is: prefer to break at a higher
syntactic level. A break inside a nested call hides the structure of the
statement.

**Before:**

```go
total := basePrice(order) + shippingCost(order.Address,
	order.Weight)
```

**After:**

```go
total := basePrice(order) +
	shippingCost(order.Address, order.Weight)
```

When a line needs a break, choose the position by these rules:

- **Assignments:** Break after an assignment operator.
- **Calls:** Keep a function name attached to its opening parenthesis.
- **Commas:** Keep a comma attached to the token before it.
- **Arrows:** Do not break next to the arrow (`=>`) of an arrow function. A
  break after the arrow is acceptable when a single expression without
  braces follows.

When a call, literal, or list needs multiple lines, put parallel arguments or
entries on separate lines and indent them consistently. Do not split short,
related arguments onto separate lines only because there are several of them.

```go
project, err := manager.Create(ctx, id, displayName)

server, err := NewServer(
	WithAddress(address),
	WithWorkspace(workspace),
	WithLogger(logger),
)
```

The first call is short, so its three arguments share one line. The second
call needs multiple lines, so each option gets its own line.

## Embedded Scripts

For embedded scripts and SQL, prefer well-formatted code inside the string
literal. Expand shell `case` branches and SQL clauses into readable blocks.
Indent the embedded code one level deeper than the statement that contains
it.

**Before:**

```go
	script := `#!/bin/sh
while IFS= read -r line; do case "$line" in query) stty raw -echo; head -c 8 > response; stty sane;; *) printf 'received: %s\n' "$line";; esac; done
`
```

**After:**

```go
	script := `#!/bin/sh
		while IFS= read -r line; do
			case "$line" in
				query)
					stty raw -echo
					head -c 8 > response
					stty sane
					;;
				*)
					printf 'received: %s\n' "$line"
					;;
			esac
		done
	`
```

Do not allow whitespace to change the meaning of the embedded code. A raw
string keeps every character, including the indentation you add. If the
embedded code needs exact whitespace, wrap the literal in a function that
removes the added whitespace.

```go
	query := dedent(`
		SELECT id, name
		FROM projects
		WHERE state = 'ready'
		ORDER BY name
	`)
```

Here `dedent` removes the first line break and the shared indentation. The
source stays formatted, and the resulting string has no added whitespace.

## Structure and Readability

### Control Flow

Keep the happy path on the left. Return early on errors when that clarifies
control flow. Use blank lines between logical phases, not between every
statement.

**Before:**

```go
func load(path string) (*Config, error) {
	data, err := os.ReadFile(path)
	if err == nil {
		var config Config
		if err := json.Unmarshal(data, &config); err == nil {
			return &config, nil
		} else {
			return nil, fmt.Errorf("parse %s: %w", path, err)
		}
	}
	return nil, fmt.Errorf("read %s: %w", path, err)
}
```

**After:**

```go
func load(path string) (*Config, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, fmt.Errorf("read %s: %w", path, err)
	}

	var config Config
	if err := json.Unmarshal(data, &config); err != nil {
		return nil, fmt.Errorf("parse %s: %w", path, err)
	}

	return &config, nil
}
```

The function returns each error where it occurs, so the happy path stays on
the left. Blank lines separate reading the file, parsing it, and returning
the result.

### Names and Comments

Prefer names that explain an operation's effect or a value's role. Follow
these rules:

- **Length:** Make the length of a name proportional to the size of its
  scope. A short name suits a few lines. A name used across a file needs
  more words.
- **Receivers:** Use one or two letters that abbreviate the type. Use the
  same receiver name in every method of that type. Do not use `this` or
  `self`.
- **Single letters:** Use a single letter only where its meaning is obvious,
  such as `i` for a loop index, `r` for an `io.Reader`, or `w` for an
  `http.ResponseWriter`.
- **Types:** Omit the type from a name. Keep it only when two forms of one
  value are in scope, such as `age` and `ageString`.
- **Context:** Omit words that the package, type, or method already supplies.

| Before | After |
| --- | --- |
| `func (this *ReportWriter) Flush()` | `func (w *ReportWriter) Flush()` |
| `userSlice`, `numUsers` | `users`, `userCount` |
| `widget.NewWidget()` | `widget.New()` |
| `func (p *Project) ProjectName()` | `func (p *Project) Name()` |

Comments should explain intent, ownership, cleanup, or a constraint that is
not obvious.

**Before:**

```go
// Read a line, then print output.
```

**After:**

```go
// Wait for input before overflowing history, then stay alive for reconnect.
```

The revised comment explains intent that the code does not show.

### Helpers

A little copying is better than a little dependency. Prefer to repeat a few
lines over adding a helper or a package that several callers then depend
on. Extract a helper when its name clarifies a meaningful operation. Do not
split straightforward code only to shorten a function.

### Tests

Keep the tests for a source file in the matching test file. For example,
`server_test.go` generally tests the code in `server.go`.

In table tests, give each case an explicit name. Keep each case's setup near
its expected result.

```go
tests := []struct {
	name, body string
	status     int
}{
	{
		name:   "created",
		body:   `{"name":"My Project","id":"alpha"}`,
		status: 201,
	},
	{
		name:   "missing display name",
		body:   `{"id":"alpha"}`,
		status: 400,
	},
}
```

Each case names the behavior it checks. The request body sits beside the
status that the body must produce.
