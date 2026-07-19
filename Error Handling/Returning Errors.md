# Returning Errors

## Why Do Functions Return Errors?

Go doesn't throw exceptions.

Instead, functions return errors as values.

Most functions follow this pattern:

```go
func Something() (Result, error)
```

---

# Standard Pattern

```go
func ReadConfig(path string) ([]byte, error)
```

Usage

```go
data, err := ReadConfig("config.json")
if err != nil {
	return err
}
```

---

# Returning Success

When everything works:

```go
return value, nil
```

Example

```go
func Square(n int) (int, error) {
	return n * n, nil
}
```

---

# Returning Failure

```go
func Login(name string) error {

	if name == "" {
		return errors.New("username is required")
	}

	return nil
}
```

---

# Error Propagation

Suppose:

```
Handler

↓

Service

↓

Repository
```

Repository

```go
return err
```

Service

```go
return err
```

Handler

```go
return err
```

The error travels upward until someone can make a decision.

---

# Don't Swallow Errors

Bad

```go
if err != nil {
	return nil
}
```

The caller loses all information.

Instead

```go
if err != nil {
	return err
}
```

---

# Return Early

Good

```go
if err != nil {
	return err
}

// Continue
```

Avoid deeply nested code.

---

# Best Practices

✅ Return `nil` on success.

---

✅ Return the original error unless you have useful context to add.

---

✅ Handle errors as close as possible to where meaningful action can be taken.

---

# Summary

- Successful operations return `nil`.
- Failed operations return an error.
- Errors usually propagate upward through the call stack.
- Prefer early returns over nested `if` statements.