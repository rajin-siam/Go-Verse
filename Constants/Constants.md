
## What are Constants?

A **constant** is a value that **cannot be changed after it is declared**.

Once the compiler assigns a value to a constant, that value remains the same throughout the lifetime of the program.

```go
const Pi = 3.14159
```

You cannot do this:

```go
Pi = 3.14 // Compile-time error
```

Unlike variables, constants are **immutable**.

---

# Why Do Constants Exist?

Imagine you're building an e-commerce application.

You have a VAT rate:

```go
vat := 0.15
```

Later, somewhere else in your code:

```go
vat = 0.20
```

Now every tax calculation becomes incorrect.

Some values should **never change** because they represent fixed business rules or system values.

Examples:

- Days in a week
- Pi
- HTTP status codes
- Order statuses
- Maximum retries
- API versions

These should be constants.

---

# Basic Syntax

Single constant

```go
const Age = 24
```

Typed constant

```go
const Age int = 24
```

Multiple constants

```go
const (
	MinAge = 18
	MaxAge = 60
)
```

---

# Constant vs Variable

| Constant | Variable |
|----------|----------|
| Cannot change | Can change |
| Declared using `const` | Declared using `var` or `:=` |
| Must be known at compile time | Can be assigned at runtime |
| Immutable | Mutable |

Example:

```go
const Country = "Bangladesh"

var city = "Dhaka"

city = "Khulna"      // OK
Country = "India"    // Error
```

---

# What Can Be a Constant?

Only values that the compiler knows during compilation.

Examples:

```go
const Name = "Siam"

const Age = 24

const Active = true

const Pi = 3.14159
```

Constant expressions are also allowed.

```go
const A = 10
const B = 20

const Sum = A + B
```

The compiler calculates this before the program runs.

---

# What Cannot Be a Constant?

Anything that requires runtime evaluation.

Wrong:

```go
const Now = time.Now()
```

Why?

Because the current time is unknown until the program starts.

Wrong:

```go
const UUID = uuid.New()
```

A UUID is generated while the program is running.

Wrong:

```go
const User = User{}
```

Structs cannot be constants.

Wrong:

```go
const Numbers = []int{1,2,3}
```

Slices cannot be constants.

Only simple compile-time values are allowed.

---

# Constant Expressions

Go allows complex compile-time calculations.

```go
const (
	A = 5
	B = 8

	C = A * B
	D = C + 100
	E = D / 3
)
```

Everything is evaluated during compilation.

---

# Memory Model

One common misconception is:

> "Does Go allocate memory for constants?"

Not always.

Consider:

```go
const Age = 24

fmt.Println(Age)
```

The compiler often replaces `Age` directly with `24`.

Conceptually:

```
Source Code

Age

↓

Compiler

↓

24
```

No variable lookup is required at runtime.

This is called **constant substitution** (or constant folding when expressions are simplified).

---

# Constant Folding

The compiler simplifies expressions involving constants.

Example

```go
const A = 5
const B = 10

const C = A * B
```

The compiler does this:

```
5 × 10

↓

50
```

Your program effectively becomes:

```go
const C = 50
```

This is one reason constants can improve performance for simple values.

---

# Why Not Make Everything a Constant?

Because many values are only known while the program is running.

Example

```go
name := os.Getenv("USER")
```

The environment variable depends on the machine running the program.

The compiler cannot know its value ahead of time.

Similarly:

- Database results
- API responses
- User input
- Current time
- Random numbers

All require variables.

---

# Common Mistakes

## Trying to Change a Constant

```go
const Age = 20

Age = 30
```

Compile error.

---

## Using Runtime Values

Wrong

```go
const Today = time.Now()
```

---

## Using Structs

Wrong

```go
const User = User{}
```

Structs are not constant values.

---

## Confusing Constants with Read-only Variables

Go does not have read-only variables.

If a value must never change, use a constant—but only if it is a valid compile-time value.

---

# Best Practices

✅ Use constants for business rules that never change.

```go
const MaxUploadSize = 10 * 1024 * 1024
```

---

✅ Group related constants.

```go
const (
	DefaultPort = 8080
	DefaultHost = "localhost"
)
```

---

✅ Give constants meaningful names.

---

✅ Prefer constants over magic numbers.

Instead of:

```go
if retries > 5
```

Use:

```go
const MaxRetries = 5

if retries > MaxRetries
```

The code is easier to understand and maintain.

---

# Interview Questions

### Why are constants faster than variables?

Because the compiler can often replace them directly with their values and simplify constant expressions during compilation.

---

### Why can't slices be constants?

Because slices contain runtime state (a pointer, length, and capacity), which isn't a compile-time constant.

---

### Can a struct be a constant?

No. Go constants are limited to compile-time values such as numbers, strings, booleans, and certain constant expressions.

---

### Is `const` always stored in memory?

Not necessarily. The compiler frequently substitutes constant values directly into the generated code instead of allocating storage for them.

---

# Related Notes

- [[Typed vs Untyped Constants]]
- [[Iota]]
- [[Enums in Go]]
- [[String Enums]]
- [[Enum Validation]]
- [[JSON with Enums]]
- [[Database Enums]]
- [[Compiler Internals of Constants]]

---

# Summary

- Constants are immutable compile-time values.
- They are declared using the `const` keyword.
- Only compile-time values can be constants.
- Constant expressions are evaluated during compilation.
- Runtime values (time, UUIDs, slices, maps, structs, function calls) cannot be constants.
- Constants improve readability by replacing magic numbers with meaningful names.
- Go's compiler can optimize constants through substitution and constant folding.