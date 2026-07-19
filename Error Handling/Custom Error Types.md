# Custom Error Types

## What is a Custom Error Type?

Sometimes an error message alone is not enough.

Instead of returning only text,

you can create a type that stores additional information.

Example:

```go
type ValidationError struct {
	Field string
	Value string
}
```

Implement the `error` interface:

```go
func (e ValidationError) Error() string {
	return fmt.Sprintf("%s is invalid: %s", e.Field, e.Value)
}
```

Now it behaves like any other error.

---

# Why Use Custom Errors?

Suppose a user enters:

```
Email = abc
```

A plain error:

```go
errors.New("invalid email")
```

tells you very little.

A custom error:

```go
ValidationError{
	Field: "email",
	Value: "abc",
}
```

contains structured data.

---

# Real Example

```go
type HTTPError struct {
	Status int
	Message string
}
```

```go
func (e HTTPError) Error() string {
	return e.Message
}
```

Usage:

```go
return HTTPError{
	Status: 404,
	Message: "user not found",
}
```

Later:

```go
var httpErr HTTPError

if errors.As(err, &httpErr) {
	fmt.Println(httpErr.Status)
}
```

---

# When Should You Create One?

Create custom errors when callers need more than a message.

Examples:

- HTTP status
- Validation field
- Database error code
- Retry information
- File path

---

# Best Practices

✅ Store useful information.

✅ Keep the `Error()` message human-readable.

✅ Support wrapping if appropriate.

---

# Common Mistakes

❌ Creating a custom type that only stores a string.

If you don't need extra data,

`errors.New()` is usually enough.

---

# Summary

- Custom error types carry structured information.
- They implement `Error() string`.
- Use `errors.As()` to extract them.