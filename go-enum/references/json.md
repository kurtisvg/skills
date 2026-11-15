# JSON

Implement `MarshalJSON` and `UnmarshalJSON` when the enum is encoded as JSON.

```go
func (s State) MarshalJSON() ([]byte, error) {
	if _, err := ParseState(s.value); err != nil {
		return nil, err
	}
	return json.Marshal(s.value)
}

func (s *State) UnmarshalJSON(data []byte) error {
	var value string
	if err := json.Unmarshal(data, &value); err != nil {
		return err
	}
	parsed, err := ParseState(value)
	if err != nil {
		return err
	}
	*s = parsed
	return nil
}
```

- **Output:** `MarshalJSON` calls the parser, so `State{}` returns an error.
- **Input:** `UnmarshalJSON` receives the raw JSON value. A value that is
  not a JSON string fails in `json.Unmarshal`. `null` decodes into an empty
  string, and the parser rejects it.
- **Failure:** `UnmarshalJSON` assigns to the receiver only after the parser
  succeeds. A failed call leaves the receiver unchanged.
- **Map keys:** JSON does not use these methods for map keys. A map with the
  enum as its key needs the methods in [Text](text.md).

## Required and Optional Fields

JSON calls no method for a missing field, so the enum cannot reject it.
Check a required field on the enclosing struct after decoding.

```go
type request struct {
	State State `json:"state"`
}

func decodeRequest(data []byte) (request, error) {
	var r request
	if err := json.Unmarshal(data, &r); err != nil {
		return request{}, err
	}
	if r.State == (State{}) {
		return request{}, errors.New("state is required")
	}
	return r, nil
}
```

Declare an optional field as a pointer. `{}` and `{"state": null}` both
leave the pointer as `nil`. With `omitempty`, a `nil` pointer is omitted on
output.

```go
type update struct {
	State *State `json:"state,omitempty"`
}
```

Do not use `omitempty` on a field of type `State`. It does not omit the zero
value, so encoding fails. Use a pointer or `omitzero`.
