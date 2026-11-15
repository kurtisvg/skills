# Text

Implement `MarshalText` and `UnmarshalText` when the enum is encoded as text.
Encoders for flags, YAML, and XML call these two methods. `encoding/json`
calls them for map keys. It also calls them for a JSON string when the type
has no JSON methods, but not for `null`. To reject `null`, read
[JSON](json.md).

```go
func (s State) MarshalText() ([]byte, error) {
	if _, err := ParseState(s.value); err != nil {
		return nil, err
	}
	return []byte(s.value), nil
}

func (s *State) UnmarshalText(text []byte) error {
	parsed, err := ParseState(string(text))
	if err != nil {
		return err
	}
	*s = parsed
	return nil
}
```

- **Output:** `MarshalText` calls the parser, so `State{}` returns an error.
  The unexported field prevents other packages from setting an arbitrary
  string. It does not prevent `State{}`. Validation on output closes that
  gap.
- **Input:** `UnmarshalText` assigns to the receiver only after the parser
  succeeds. A failed call leaves the receiver unchanged.
