# Recover

## What is Recover?

`recover()` stops a panic and allows the program to continue.

However,

it only works inside a deferred function.

---

# Basic Example

```go
func main() {

	defer func() {

		if r := recover(); r != nil {

			fmt.Println("Recovered:", r)
		}

	}()

	panic("boom")
}
```

Output:

```
Recovered: boom
```

---

# Why Must It Be Deferred?

When a panic happens,

Go begins unwinding the stack.

Only deferred functions execute during this process.

Without `defer`,

`recover()` has no panic to intercept.

---

# Typical Usage

HTTP servers often recover from panics.

```
Request

↓

Handler

↓

panic

↓

recover

↓

500 Internal Server Error
```

The server continues serving other requests.

---

# Recover Does NOT Fix the Problem

It prevents the process from crashing.

You should still:

- Log the panic.
- Investigate the root cause.
- Fix the underlying bug.

---

# Common Mistakes

❌ Using `recover()` as a replacement for error handling.

Panics are not a normal control-flow mechanism.

---

# Best Practices

✅ Recover only at application boundaries.

Examples:

- HTTP middleware
- Worker goroutines
- CLI entry point

Don't scatter `recover()` throughout your business logic.

---

# Summary

- `recover()` catches a panic.
- It only works inside deferred functions.
- It should be used at application boundaries.