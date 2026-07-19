
## Why Wrap Errors?

Suppose the database returns:

```text
duplicate key value
```

If you simply return it:

```go
return err
```

The caller knows **what** happened, but not **where**.

Wrapping adds context while preserving the original error.

---

# Basic Wrapping

```go
return fmt.Errorf("create user: %w", err)
```

Notice the `%w`.

It means:

> Wrap this error.

Now the error becomes:

```
create user: duplicate key value
```

---

# Error Propagation

Repository

```go
return fmt.Errorf("insert user: %w", err)
```

↓

Service

```go
return fmt.Errorf("register user: %w", err)
```

↓

Handler

```
register user:
insert user:
duplicate key value
```

Every layer adds useful context.

---

# Why `%w`?

`%w` preserves the original error.

Later,

```go
errors.Is(err, sql.ErrNoRows)
```

still works.

If you use `%v`

```go
fmt.Errorf("failed: %v", err)
```

the error becomes plain text.

The original error is lost.

---

# `%w` vs `%v`

### `%v`

```go
fmt.Errorf("failed: %v", err)
```

Result

```
failed: file not found
```

But the original error cannot be extracted.

---

### `%w`

```go
fmt.Errorf("failed: %w", err)
```

Result

```
failed: file not found
```

Looks the same to humans.

Internally, the original error is preserved.

---

# Real World Example

Repository

```go
return fmt.Errorf("find user %d: %w", id, err)
```

Service

```go
return fmt.Errorf("login: %w", err)
```

Handler

```go
log.Println(err)
```

Output

```
login:
find user 10:
sql: no rows in result set
```

You immediately know the path the error took.

---

# Wrapping Multiple Layers

```
Database

↓

sql.ErrNoRows

↓

Repository

↓

find user

↓

Service

↓

login

↓

Handler
```

The original cause is never lost.

---

# Common Mistakes

❌ Using `%v`

```go
fmt.Errorf("save user: %v", err)
```

You cannot use `errors.Is()` on the original error anymore.

---

❌ Wrapping when there is no additional context.

Bad

```go
return fmt.Errorf("%w", err)
```

Just return `err`.

---

# Best Practices

✅ Add context at each application layer.

---

✅ Use `%w` whenever you want callers to inspect the original error.

---

✅ Keep context short and meaningful.

Good

```
create user
```

Bad

```
an unexpected problem occurred while attempting to create a user record in the database
```

---

# Summary

- Wrapping adds context without losing the original error.
- Use `fmt.Errorf("...: %w", err)` for wrapping.
- `%w` enables `errors.Is()` and `errors.As()`.
- Use `%v` only when you intentionally do **not** want to wrap the error.