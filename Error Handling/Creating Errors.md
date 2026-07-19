# Creating Errors

## What Does "Creating an Error" Mean?

An error is simply a value that implements the `error` interface.

Sometimes your function needs to create and return a new error.

Example:

```go
func Divide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}

	return a / b, nil
}
```

---

# Ways to Create Errors

Go provides several ways.

```
errors.New()

↓

fmt.Errorf()

↓

Custom Error Types
```

Each has different use cases.

---

# 1. errors.New()

The simplest way.

```go
err := errors.New("file not found")
```

Internally it creates an object implementing the `error` interface.

Use when:

- The message is fixed.
- No dynamic values are needed.

Example:

```go
if username == "" {
	return errors.New("username is required")
}
```

---

# 2. fmt.Errorf()

Use when the message needs formatting.

```go
name := "users.json"

return fmt.Errorf("cannot open %s", name)
```

Output

```
cannot open users.json
```

This is like `fmt.Printf`, but it returns an error.

---

# 3. Custom Error Types

Sometimes a string isn't enough.

```go
type ValidationError struct {
	Field string
}

func (e ValidationError) Error() string {
	return fmt.Sprintf("%s is invalid", e.Field)
}
```

Usage

```go
return ValidationError{
	Field: "email",
}
```

Useful when callers need structured information.

---

# Which One Should You Use?

| Situation | Recommendation |
|-----------|----------------|
| Fixed message | `errors.New()` |
| Dynamic message | `fmt.Errorf()` |
| Additional data or behavior | Custom error type |

---

# Best Practices

✅ Keep messages lowercase.

Good

```go
errors.New("file not found")
```

Bad

```go
errors.New("File Not Found")
```

---

✅ Don't end error messages with periods.

Good

```
user not found
```

Bad

```
user not found.
```

This follows the Go style guide because errors are often combined into larger messages.

---

# Common Mistakes

❌ Creating vague errors.

```go
errors.New("failed")
```

Better

```go
errors.New("failed to save user")
```

---

# Related Notes

- [[The error Interface]]
- [[Returning Errors]]
- [[Wrapping Errors]]
- [[Custom Error Types]]

---

# Summary

- Use `errors.New()` for fixed messages.
- Use `fmt.Errorf()` for formatted messages.
- Use custom types when callers need more than a string.