# Error Logging

## Why Log Errors?

Errors returned to users are often simple:

```
Internal Server Error
```

Logs contain the detailed information developers need.

---

# Where Should Errors Be Logged?

One of the biggest mistakes is logging the same error multiple times.

Bad:

Repository

```go
log.Println(err)
return err
```

Service

```go
log.Println(err)
return err
```

Handler

```go
log.Println(err)
```

Output:

```
duplicate key
duplicate key
duplicate key
```

---

# Better

Repository

```go
return fmt.Errorf("create user: %w", err)
```

Service

```go
return fmt.Errorf("register user: %w", err)
```

Handler

```go
log.Println(err)
```

Output:

```
register user:
create user:
duplicate key
```

Log once.

---

# Structured Logging

Instead of:

```go
log.Println(err)
```

Modern applications often log structured data.

Example:

```go
logger.Error(
	"register user",
	"userID", id,
	"error", err,
)
```

Benefits:

- Easier searching
- Better filtering
- Machine-readable logs

---

# Don't Log Sensitive Data

Bad:

```
Password

JWT Token

Credit Card

API Keys
```

Logs may be stored for months.

Treat them as sensitive.

---

# Error Levels

Typical log levels:

```
DEBUG

INFO

WARN

ERROR

FATAL
```

Most returned errors are logged as `ERROR`.

---

# Best Practices

✅ Log errors once.

✅ Add useful context.

✅ Avoid logging secrets.

✅ Prefer structured logging over plain text.

---

# Summary

- Logs help developers diagnose failures.
- Wrap errors throughout the application.
- Log once near the application's boundary.
- Use structured logging when possible.