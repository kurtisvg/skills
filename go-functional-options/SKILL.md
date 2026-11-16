---
name: go-functional-options
description: Use when designing or reviewing a Go constructor that has
  optional settings, or when adding functional options such as
  `WithTimeout`. Guides the option type, defaults, validation, and when to
  use ordinary parameters instead.
---

# Go Functional Options

Use this pattern to avoid long lists of positional arguments. Without it,
every caller passes a value for each optional setting. With it, callers
pass only the settings that they change, and a new option does not break
existing callers.

**Before:**

```go
server, err := NewServer("localhost", 8080, 0, nil, 0)
```

**After:**

```go
server, err := NewServer("localhost", 8080, WithTimeout(5*time.Second))
```

In the first call, the reader cannot tell what each value means, or whether
`0` and `nil` select a default. In the second call, the changed setting has
a name, and the other settings use their defaults.

## When to Use the Pattern

Use the pattern when all of these are true:

- The settings are optional, and each one has a sensible default.
- The function is part of an API that other packages call.
- There are three or more optional settings, or a current requirement
  says that settings will be added.

Do not use the pattern in these cases:

- **Required values:** Pass a required value as an ordinary parameter
  before the options, such as `host` and `port`. An option that every
  caller must pass hides a requirement from the compiler.
- **One or two optional settings:** Use ordinary parameters. The pattern
  adds a type, and a function for each setting.

Prefer the pattern over an exported options struct, such as
`ServerOptions{Timeout: 5 * time.Second}`. A struct cannot distinguish an
unset field from a zero value. Suppose that a timeout of `0` means "no
timeout", and the default is 30 seconds. The constructor cannot tell
`Timeout: 0` from a field that the caller did not set.

## Define the Options

```go
// config holds the optional settings of NewServer.
type config struct {
	timeout  time.Duration
	logger   *slog.Logger
	maxConns int
}

// Option changes one optional setting of NewServer.
type Option func(*config) error

// WithTimeout sets the request timeout. A timeout of 0 disables the
// limit. The default is 30 seconds.
func WithTimeout(timeout time.Duration) Option {
	return func(c *config) error {
		if timeout < 0 {
			return fmt.Errorf("timeout must not be negative: %v", timeout)
		}
		c.timeout = timeout
		return nil
	}
}

// WithLogger sets the logger. The default is slog.Default().
func WithLogger(logger *slog.Logger) Option {
	return func(c *config) error {
		if logger == nil {
			return errors.New("logger must not be nil")
		}
		c.logger = logger
		return nil
	}
}

// WithMaxConns sets the maximum number of open connections. The default
// is 100.
func WithMaxConns(maxConns int) Option {
	return func(c *config) error {
		if maxConns <= 0 {
			return fmt.Errorf("maxConns must be positive: %d", maxConns)
		}
		c.maxConns = maxConns
		return nil
	}
}

// NewServer returns a server that listens on host and port.
func NewServer(host string, port int, opts ...Option) (*Server, error) {
	c := config{
		timeout:  30 * time.Second,
		logger:   slog.Default(),
		maxConns: 100,
	}
	for _, opt := range opts {
		if err := opt(&c); err != nil {
			return nil, err
		}
	}

	return &Server{
		host:     host,
		port:     port,
		timeout:  c.timeout,
		logger:   c.logger,
		maxConns: c.maxConns,
	}, nil
}
```

- **Settings struct:** Apply options to an unexported settings struct, such
  as `config`, not to the `Server`. Callers then cannot apply an option to
  a `Server` after construction.
- **Option type:** Declare `type Option func(*config) error` when an option
  can reject its value. When no option validates, omit the error result
  and declare `type Option func(*config)`. When a package has options for
  several constructors, name each type for its constructor, such as
  `ServerOption`.
- **Option functions:** Name each one `With` followed by the setting. State
  the default in its comment.
- **Defaults:** Set every default in the constructor before it applies the
  options. A call without options then returns a usable object.
- **Order:** Apply the options in the order given. A later option replaces
  the value from an earlier one.
- **Validation:** When an option can reject its value, validate the value
  in the function that the option returns. The constructor returns the
  first error. Validate in the constructor after the loop only when a
  check compares several settings.

## Test Cases

Test through the constructor. Observe each setting through the behavior of
the `Server`, or through its fields in a test in the same package.

- A call without options uses every default.
- Each option changes its own setting and no other.
- When an option is given twice, the later value is used.
- The constructor returns an error for each invalid value.
