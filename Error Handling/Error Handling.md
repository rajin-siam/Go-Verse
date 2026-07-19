## What is Error Handling?

Error handling is the process of detecting, reporting, and responding to unexpected situations in a program.

Examples:

- File not found
- Database connection failed
- Invalid user input
- Network timeout
- JSON parsing failed
- Authentication failed

Without proper error handling, programs crash, behave unpredictably, or return incorrect results.

---

# Go's Philosophy

Many languages use **exceptions**.

For example:

```java
try {
    readFile();
} catch (Exception e) {
    // Handle error
}
```

Go takes a different approach.

Functions **return errors as values**.

```go
data, err := os.ReadFile("notes.txt")
if err != nil {
    return err
}
```

Errors are ordinary values that you can:

- Return
- Compare
- Wrap
- Log
- Store
- Pass to other functions

This simplicity is one of Go's core design principles.

---

# Errors Are Expected

One of the biggest mindset changes in Go is:

> Errors are not exceptional.

They are part of normal program flow.

Example:

```go
user, err := repo.FindByID(id)
if err != nil {
    return err
}
```

A missing database record is not a programming mistake.

It is simply something that can happen.

---

# Why Doesn't Go Use Exceptions?

Imagine this code.

```go
user := GetUser()

orders := GetOrders(user)

invoice := CreateInvoice(orders)
```

In an exception-based language, any of these functions might suddenly throw an exception.

The caller cannot easily see where failures may occur.

In Go:

```go
user, err := GetUser()
if err != nil {
    return err
}

orders, err := GetOrders(user)
if err != nil {
    return err
}

invoice, err := CreateInvoice(orders)
if err != nil {
    return err
}
```

Every possible failure is explicit.

---

# The Standard Error Pattern

Nearly every Go function that can fail follows this pattern.

```go
func Something() (Result, error)
```

Example

```go
func ReadFile(name string) ([]byte, error)
```

Usage

```go
data, err := os.ReadFile("notes.txt")

if err != nil {
    return err
}
```

The second return value tells you whether the operation succeeded.

---

# Understanding `if err != nil`

This is probably the most common line in Go.

```go
if err != nil {
    return err
}
```

Think of it like a checkpoint.

```
Call Function
      │
      ▼
Did it fail?
      │
 ┌────┴────┐
 │         │
 ▼         ▼
Yes        No
 │          │
Return     Continue
```

---

# Real World Example

Imagine withdrawing money from an ATM.

```
Insert Card
      │
      ▼
Read Card
      │
      ▼
PIN Correct?
      │
 ┌────┴────┐
 │         │
 ▼         ▼
No        Yes
 │          │
Stop      Continue
```

Go programs work the same way.

Each operation is checked before moving on.

---

# Returning Errors

Example

```go
func Login(username string) error {

    if username == "" {
        return errors.New("username is required")
    }

    return nil
}
```

Calling

```go
err := Login("")

if err != nil {
    fmt.Println(err)
}
```

Output

```
username is required
```

---

# What Does `nil` Mean?

When no error occurs,

the error value is

```go
nil
```

Example

```go
err := Save()

if err == nil {
    fmt.Println("Success")
}
```

Think of

```
nil

↓

No Error
```

---

# Error Propagation

Suppose your application has three layers.

```
HTTP Handler

↓

Service

↓

Repository

↓

Database
```

The database fails.

```
Database

↓

Error
```

Repository returns it.

```
Repository

↓

Service

↓

Handler
```

The error travels upward until someone handles it.

This is called **error propagation**.

---

# Who Should Handle Errors?

A good rule:

The layer that **can make a meaningful decision** should handle the error.

Example

Repository

```go
return err
```

Service

```go
return fmt.Errorf("create user: %w", err)
```

Handler

```go
if errors.Is(err, ErrUserExists) {
    return 409
}

return 500
```

Each layer has a different responsibility.

---

# Add Context While Propagating

Bad

```go
return err
```

Good

```go
return fmt.Errorf("create user: %w", err)
```

Now the caller knows

- what failed
- where it failed

without losing the original error.

We'll study wrapping in detail later.

---

# Don't Ignore Errors

Bad

```go
data, _ := os.ReadFile("file.txt")
```

If the file doesn't exist,

you'll never know why.

Good

```go
data, err := os.ReadFile("file.txt")

if err != nil {
    return err
}
```

---

# Avoid Comparing Error Strings

Bad

```go
if err.Error() == "file not found" {

}
```

Strings change.

Messages change.

Use

```go
errors.Is()
```

instead.

We'll cover this later.

---

# Return Errors, Don't Panic

Expected failures

```
File Missing

Invalid Password

Database Timeout
```

↓

Return an error.

Unexpected programmer bugs

```
Impossible State

Broken Invariant

Corrupted Internal Data
```

↓

Maybe panic.

Most applications should return errors far more often than they panic.

---

# Error Handling Flow

```
Call Function
      │
      ▼
Receive Error
      │
      ▼
Is Error nil?
      │
 ┌────┴────┐
 │         │
 ▼         ▼
Yes        No
 │          │
Continue   Return / Wrap / Handle
```

---

# Common Error Sources

## Files

```go
os.Open()
```

---

## Database

```go
db.Query()
```

---

## HTTP

```go
http.Get()
```

---

## JSON

```go
json.Unmarshal()
```

---

## Network

```go
net.Dial()
```

---

## User Input

```go
strconv.Atoi()
```

---

# Common Mistakes

## Ignoring Errors

```go
_, _ = os.ReadFile("a.txt")
```

Never do this unless you intentionally want to ignore the error.

---

## Logging and Returning the Same Error

Bad

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

Now the same error appears three times.

Usually, **log an error once**, at the application boundary (for example, the HTTP handler or worker).

---

## Panicking for Normal Errors

Bad

```go
panic(err)
```

if the user entered an invalid password.

That's a normal situation.

Return an error instead.

---

## Hiding Useful Context

Bad

```go
return errors.New("failed")
```

Better

```go
return fmt.Errorf("save user: %w", err)
```

---

# Best Practices

✅ Check errors immediately.

---

✅ Return errors instead of panicking for expected failures.

---

✅ Add context while returning errors.

---

✅ Let higher layers decide how to respond.

---

✅ Handle errors once.

---

✅ Use `errors.Is()` and `errors.As()` instead of comparing strings.

---

✅ Never ignore errors unless you're absolutely sure it's safe.

---

# Error Handling in a Real Codebase

A typical backend application looks like this:

```
HTTP Request
      │
      ▼
Handler
      │
      ▼
Service
      │
      ▼
Repository
      │
      ▼
Database
```

Example flow:

```
Database
    │
    ▼
duplicate key
    │
    ▼
Repository
"create user: duplicate key"
    │
    ▼
Service
"register user: create user: duplicate key"
    │
    ▼
Handler
409 Conflict
```

Each layer adds context but doesn't lose the original error.

---

# Interview Questions

### Why does Go return errors instead of using exceptions?

Because errors are explicit values. This makes control flow easier to understand and avoids hidden exception paths.

---

### Why should errors be wrapped?

To add context while preserving the original error.

---

### Should every error be logged?

No.

Log errors once at the application's boundary.

---

### Should libraries panic?

Generally no.

Libraries should return errors and let callers decide how to handle them.

---

### Should you compare error strings?

No.

Use `errors.Is()` or `errors.As()` when appropriate.

---

# Related Notes

- [[The error Interface]]
- [[Creating Errors]]
- [[Returning Errors]]
- [[Wrapping Errors]]
- [[errors.Is]]
- [[errors.As]]
- [[Sentinel Errors]]
- [[Custom Error Types]]
- [[Panic]]
- [[Recover]]
- [[Error Logging]]

---

# Summary

- Errors are ordinary values in Go.
- Functions usually return `(value, error)`.
- Check errors immediately after calling a function.
- Return errors for expected failures instead of panicking.
- Propagate errors upward until a layer can make a meaningful decision.
- Add context when returning errors to make debugging easier.
- Log errors once, typically at the application's boundary.
- Prefer `errors.Is()` and `errors.As()` over comparing error messages.