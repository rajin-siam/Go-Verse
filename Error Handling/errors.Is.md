

## Why Do We Need `errors.Is`?

Suppose you have:

```go
err := sql.ErrNoRows
```

Checking it is easy:

```go
if err == sql.ErrNoRows {

}
```

Now imagine the error was wrapped.

```go
err = fmt.Errorf("find user: %w", sql.ErrNoRows)
```

Now:

```go
err == sql.ErrNoRows
```

returns

```
false
```

because they are different values.

This is why `errors.Is` exists.

---

# What Does `errors.Is` Do?

It searches through the entire wrapped error chain.

Example

```go
err := fmt.Errorf("find user: %w", sql.ErrNoRows)

if errors.Is(err, sql.ErrNoRows) {
	fmt.Println("User not found")
}
```

Output

```
User not found
```

---

# Error Chain

Imagine this:

```
Handler

↓

Service

↓

Repository

↓

sql.ErrNoRows
```

After wrapping:

```
login failed

↓

find user failed

↓

sql.ErrNoRows
```

`errors.Is`

starts at the top

```
login failed
```

then asks

```
Is this sql.ErrNoRows?

↓

No
```

Move deeper.

```
find user failed

↓

No
```

Move deeper.

```
sql.ErrNoRows

↓

Yes
```

Return true.

---

# Visual Representation

```
Wrapped Error

↓

Wrapped Error

↓

Wrapped Error

↓

Original Error
```

`errors.Is` walks through every level.

---

# Example

Repository

```go
return fmt.Errorf("find user: %w", sql.ErrNoRows)
```

Service

```go
return fmt.Errorf("login: %w", err)
```

Handler

```go
if errors.Is(err, sql.ErrNoRows) {

	http.Error(w, "User not found", 404)

	return
}
```

The handler doesn't need to know how many layers wrapped the error.

---

# Without `errors.Is`

You would have to unwrap manually.

```
Layer

↓

Layer

↓

Layer

↓

Layer
```

Very inconvenient.

`errors.Is` does this automatically.

---

# How It Works Internally

Conceptually

```go
current := err

for current != nil {

	if current == target {
		return true
	}

	current = unwrap(current)
}
```

The actual implementation also supports custom `Is` methods, but this mental model is enough.

---

# When Should You Use It?

Use `errors.Is` for **known error values**.

Examples

```go
sql.ErrNoRows

os.ErrNotExist

context.Canceled

context.DeadlineExceeded
```

---

# Common Mistakes

Bad

```go
if err == sql.ErrNoRows
```

Fails if wrapped.

Good

```go
errors.Is(err, sql.ErrNoRows)
```

---

# Best Practices

✅ Prefer `errors.Is` over direct equality for wrapped errors.

---

✅ Wrap errors with `%w` so `errors.Is` can find the original error.

---

# Summary

- `errors.Is` compares **error values**.
- It searches through wrapped errors automatically.
- It should replace `==` for errors that may have been wrapped.