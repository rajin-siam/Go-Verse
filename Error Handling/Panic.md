# Panic

## What is Panic?

A panic is an unrecoverable runtime event that immediately stops the normal flow of the current goroutine.

Example:

```go
panic("something went terribly wrong")
```

Output:

```
panic: something went terribly wrong
```

---

# Panic vs Error

Expected failure:

```go
return errors.New("file not found")
```

Unexpected bug:

```go
panic("nil pointer")
```

Errors represent expected problems.

Panics represent programming mistakes or impossible states.

---

# Built-in Panics

Examples:

```go
var s []int

fmt.Println(s[100])
```

```
panic: index out of range
```

---

```go
var p *User

fmt.Println(p.Name)
```

```
panic: nil pointer dereference
```

---

# Stack Unwinding

Suppose:

```
main

↓

A

↓

B

↓

panic
```

Go unwinds the stack.

```
B

↓

A

↓

main
```

Running deferred functions along the way.

---

# When Should You Panic?

Rarely.

Good uses:

- Impossible states
- Corrupted internal state
- Programmer bugs

Bad uses:

- Invalid user input
- Database timeout
- Missing file
- HTTP 404

These should return errors.

---

# Best Practices

✅ Panic only for truly exceptional conditions.

✅ Libraries should almost always return errors instead of panicking.

---

# Summary

- Panics stop normal execution.
- Deferred functions still run.
- Most application code should return errors instead.