# SQL

Implement `sql.Scanner` and `driver.Valuer` when the enum is stored in a
database. SQL struct scanners can then map enum columns directly.

```go
// Scan implements sql.Scanner.
func (s *State) Scan(value any) error {
	var text string
	switch value := value.(type) {
	case string:
		text = value
	case []byte:
		text = string(value)
	default:
		return fmt.Errorf("cannot scan state from %T", value)
	}
	parsed, err := ParseState(text)
	if err != nil {
		return err
	}
	*s = parsed
	return nil
}

// Value implements driver.Valuer.
func (s State) Value() (driver.Value, error) {
	if _, err := ParseState(s.value); err != nil {
		return nil, err
	}
	return s.value, nil
}
```

- **Reads:** Accept strings and byte slices. Reject every other type. A
  database NULL reaches `Scan` as the Go value `nil`, so the default case
  rejects it. Accept NULL only when the domain allows it.
- **Failure:** `Scan` assigns to the receiver only after the parser
  succeeds. A failed read leaves the receiver unchanged.
- **Writes:** `Value` calls the parser, so the zero value cannot be stored.
- **Constraints:** Keep the database constraints on the column, such as a
  `CHECK` constraint. The Go type does not replace them.
