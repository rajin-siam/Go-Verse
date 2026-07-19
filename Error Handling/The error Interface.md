# The `error` Interface

## What is `error`?

In Go, `error` is **not a keyword**.

It is **not a class**.

It is **not an exception**.

It is simply an **interface** defined by the standard library.

Definition:

```go
type error interface {
	Error() string
}
```

That's it.

Any type that implements this single method automatically satisfies the `error` interface.

---

# Breaking It Down

The interface contains one method:

```go
Error() string
```

This method returns a human-readable description of the error.

Example:

```go
err := errors.New("file not found")

fmt.Println(err.Error())
```

Output

```
file not found
```

Usually, you don't call `Error()` directly because functions like `fmt.Println` call it automatically.

---

# Why an Interface?

Go prefers **behavior over inheritance**.

Instead of asking:

> "Is this object an Error class?"

Go asks:

> "Does this type provide an `Error() string` method?"

If yes, it is an error.

---

# Real World Analogy

Imagine a security checkpoint.

To enter, everyone must show an ID card.

```
Person A

↓

Has ID
```

Allowed.

```
Person B

↓

Has ID
```

Allowed.

The checkpoint doesn't care **who** the person is.

It only cares whether they satisfy the requirement.

The `error` interface works the same way.

```
Type

↓

Has Error() string ?

↓

Yes

↓

It is an error.
```

---

# Creating a Custom Error

Suppose we create our own type.

```go
type ValidationError struct {
	Message string
}
```

Implement the interface:

```go
func (e ValidationError) Error() string {
	return e.Message
}
```

Now:

```go
err := ValidationError{
	Message: "email is required",
}

fmt.Println(err)
```

Output

```
email is required
```

`ValidationError` is now an `error`.

---

# Interface Satisfaction

Notice:

```go
type error interface {
	Error() string
}
```

Our type:

```go
type ValidationError struct{}
```

Method:

```go
func (e ValidationError) Error() string
```

The compiler automatically connects them.

There is **no** keyword like

```go
implements
```

used in Java or C#.

Go uses **implicit interface implementation**.

---

# Why This Is Powerful

Many completely different types can all behave as errors.

Examples:

```
ValidationError

DatabaseError

NetworkError

FileError

PermissionError
```

All implement:

```go
Error() string
```

Therefore,

they can all be stored in:

```go
var err error
```

---

# Interface Values

Consider:

```go
var err error
```

At first:

```
Type

↓

nil

Value

↓

nil
```

After:

```go
err = errors.New("boom")
```

Conceptually:

```
Interface

↓

Concrete Type

↓

errorString

↓

Value

↓

"boom"
```

An interface stores **two pieces of information**:

1. The concrete type.
2. The concrete value.

This is very important for understanding `nil` interfaces later.

---

# How `errors.New()` Works

When you write:

```go
err := errors.New("file not found")
```

You might imagine a special language feature.

It isn't.

Conceptually, the standard library does something like:

```go
type errorString struct {
	text string
}

func (e *errorString) Error() string {
	return e.text
}

func New(text string) error {
	return &errorString{text: text}
}
```

The actual implementation differs slightly, but the idea is the same.

`errors.New` creates a concrete type that implements the `error` interface.

---

# Why `fmt.Println(err)` Works

When you write:

```go
fmt.Println(err)
```

The `fmt` package checks:

```
Does this value implement error?
```

If yes,

it calls:

```go
err.Error()
```

automatically.

So these are effectively equivalent:

```go
fmt.Println(err)
```

and

```go
fmt.Println(err.Error())
```

---

# Methods and Interfaces

Since `error` is an interface,

any method receiver works.

Value receiver:

```go
func (e ValidationError) Error() string
```

Pointer receiver:

```go
func (e *ValidationError) Error() string
```

Choose the receiver based on your type's needs, just like with any other method.

---

# The Nil Interface Trap

This is one of the most famous Go interview questions.

Consider:

```go
type MyError struct{}

func (e *MyError) Error() string {
	return "oops"
}

func Test() error {
	var e *MyError = nil
	return e
}
```

Now:

```go
err := Test()

fmt.Println(err == nil)
```

Output

```
false
```

Why?

Because the interface contains:

```
Type

↓

*MyError

Value

↓

nil
```

The interface itself is **not nil** because it still has a concrete type.

A truly nil interface has:

```
Type

↓

nil

Value

↓

nil
```

This distinction is crucial when returning errors.

---

# Compiler Internals

When you assign:

```go
var err error

err = ValidationError{
	Message: "invalid input",
}
```

The compiler creates an interface value.

Conceptually:

```
+----------------------+
| Interface Value      |
+----------------------+
| Type: ValidationError|
| Value: {Message: ...}|
+----------------------+
```

Calling:

```go
err.Error()
```

uses the stored type information to dispatch to the correct `Error` method.

---

# Memory Model

An interface value conceptually stores:

```
+----------------------+
| Concrete Type        |
+----------------------+
| Concrete Value       |
+----------------------+
```

The concrete value may itself live on the stack or heap depending on escape analysis.

---

# Common Mistakes

## Thinking `error` Is Special

It isn't.

It's just an interface.

---

## Comparing Error Strings

Bad:

```go
if err.Error() == "file not found" {

}
```

Prefer:

```go
errors.Is(err, ErrFileNotFound)
```

We'll cover this later.

---

## Returning Typed Nil Errors

Avoid:

```go
var e *MyError = nil

return e
```

The caller receives a non-nil `error` interface.

Instead, return:

```go
return nil
```

when there is no error.

---

# Best Practices

✅ Treat `error` as an interface, not a special language feature.

---

✅ Return `nil` when there is no error.

---

✅ Implement `Error() string` for custom error types.

---

✅ Use descriptive error messages.

---

✅ Understand the nil interface trap.

---

# Interview Questions

### What is the `error` type?

An interface with one method:

```go
Error() string
```

---

### How does a type become an error?

By implementing the `Error() string` method.

---

### Does Go require an `implements` keyword?

No.

Interfaces are implemented implicitly.

---

### Why can many different error types be stored in an `error` variable?

Because they all satisfy the `error` interface.

---

### Why can `err != nil` even if the underlying pointer is nil?

Because an interface is only nil when **both** its concrete type and concrete value are nil.

---

# Related Notes

- [[Error Handling]]
- [[Creating Errors]]
- [[Custom Error Types]]
- [[Wrapping Errors]]
- [[errors.Is]]
- [[errors.As]]
- [[Interfaces]]
- [[Method Sets]]
- [[Compiler Internals of Interfaces]]

---

# Summary

- `error` is an interface, not a built-in exception type.
- The `error` interface has a single method: `Error() string`.
- Any type that implements this method automatically satisfies the interface.
- `errors.New()` returns a concrete type that implements `error`.
- Interface values store both a concrete type and a concrete value.
- A typed nil inside an interface is not the same as a nil interface.
- Understanding the `error` interface is the foundation for custom errors, wrapping, and advanced error handling.