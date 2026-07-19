## Why Does Go Have Two Kinds of Constants?

When learning Go, you'll often see code like this:

```go
const A = 10

var x int = A
var y int64 = A
var z float64 = A
```

How can the same constant become an `int`, `int64`, or `float64` without an explicit conversion?

The answer is that **Go has untyped constants.**

Unlike variables, constants don't always have a concrete type.

---

# Real World Analogy

Imagine water.

Water has no shape of its own.

It takes the shape of its container.

```
Water
 │
 ├── Cup
 ├── Bottle
 ├── Glass
 └── Bucket
```

Untyped constants work similarly.

```
10

↓

int

↓

int64

↓

float64

↓

complex128
```

The constant adapts to the context.

---

# Typed Constants

A typed constant has a fixed type.

Example:

```go
const Age int = 25
```

Now the compiler knows:

```
Value : 25

Type : int
```

This constant behaves much like an immutable variable.

---

# Untyped Constants

An untyped constant has a value but **no concrete Go type yet.**

Example

```go
const Age = 25
```

The compiler stores

```
Value

25
```

Notice:

No type.

The type is chosen later when needed.

---

# Why Does Go Do This?

Suppose Go required every constant to have a type.

Then this would fail:

```go
const A int = 10

var b int64 = A
```

because

```
int

↓

int64
```

would require a conversion.

Instead, Go says:

> "Since this is just the number 10, I'll let it become whatever numeric type is appropriate."

This makes code much cleaner.

---

# Example 1

```go
const A = 10

var i int = A
var j int32 = A
var k int64 = A
```

All compile successfully.

The compiler silently gives `A` the required type.

---

# Example 2

```go
const Pi = 3.14

var x float32 = Pi
var y float64 = Pi
```

Again,

```
Pi

↓

float32

↓

float64
```

No explicit conversion is needed.

---

# Example 3

```go
const Flag = true

var ok bool = Flag
```

Same idea.

---

# Default Types

Eventually an untyped constant needs a real type.

Go assigns a **default type**.

| Untyped Constant | Default Type |
| ---------------- | ------------ |
| Integer          | int          |
| Floating-point   | float64      |
| Rune             | rune         |
| String           | string       |
| Boolean          | bool         |
| Complex          | complex128   |

Example

```go
const A = 100

fmt.Printf("%T\n", A)
```

Output

```
int
```

Although `A` starts untyped, when its type is required in this context, the default type (`int`) is chosen.

---

# When Does an Untyped Constant Get a Type?

It becomes typed when the context requires one.

```go
const A = 100

x := A
```

Here,

```
A

↓

int

↓

x
```

because `:=` uses the constant's default type.

---

Another example

```go
const A = 100

var b int64 = A
```

Now

```
A

↓

int64
```

because the variable requires an `int64`.

---

# Untyped Constants Are More Flexible Than Variables

Variable

```go
var x int = 10

var y int64 = x
```

Compile error.

Need conversion.

```go
var y int64 = int64(x)
```

Constant

```go
const x = 10

var y int64 = x
```

Works.

Why?

Because `x` is untyped.

---

# Large Numbers

One amazing feature of untyped constants is that they can hold values much larger than ordinary integer types.

```go
const Huge = 100000000000000000000000000000000
```

This compiles.

Why?

Because no actual machine type has been chosen yet.

The compiler stores it with arbitrary precision for constant evaluation.

---

But this fails:

```go
var x int64 = Huge
```

Error.

The value doesn't fit into an `int64`.

---

# Compile-Time Precision

Go evaluates constant expressions with high precision.

Example

```go
const A = 1e1000
```

The compiler can represent enormous constant values during compilation, even though they may not fit into runtime numeric types.

The error only appears when assigning the value to a type that cannot represent it.

---

# Arithmetic with Untyped Constants

```go
const A = 5
const B = 2

const C = A / B
```

Result

```
2
```

Both operands are integer constants, so integer division is used.

---

Floating-point example

```go
const A = 5.0
const B = 2

const C = A / B
```

Result

```
2.5
```

The presence of a floating-point constant changes the result.

---

# Mixed Constant Expressions

```go
const A = 5

const B = A + 3.5
```

The compiler evaluates this as

```
8.5
```

No casts are needed.

---

# Typed Constants Are Less Flexible

```go
const A int = 10

var x int64 = A
```

Compile error.

Need conversion.

```go
var x int64 = int64(A)
```

Unlike untyped constants, typed constants obey the normal assignment rules.

---

# Why Use Typed Constants?

Sometimes you want to restrict values.

Example

```go
type Status int

const Running Status = 1
```

Now

```
Running

↓

Status
```

instead of

```
Running

↓

int
```

This improves type safety and is the foundation of enum-like patterns in Go.

---

# Compiler Internals

Internally, the compiler treats untyped constants as **ideal values**.

Think of them as mathematical values that haven't been assigned a machine type yet.

```
10

↓

Ideal Integer

↓

int

↓

int32

↓

int64

↓

float64
```

Only when a context requires a concrete type does the compiler convert the value.

This is why constants can participate in expressions more flexibly than variables.

---

# Common Mistakes

## Thinking Every Constant Has a Type

Wrong.

```go
const A = 10
```

Initially, `A` is untyped.

---

## Assuming Variables Behave the Same Way

Variables always have a concrete type.

```go
var x = 10
```

`x` is an `int` immediately.

---

## Forgetting That Typed Constants Need Conversions

```go
const A int = 10

var x int64 = A
```

Requires:

```go
var x int64 = int64(A)
```

---

# Best Practices

✅ Use **untyped constants** for general numeric values.

```go
const MaxRetries = 3
```

They are flexible and work naturally with different numeric types.

---

✅ Use **typed constants** when creating custom types, especially enum-like values.

```go
type OrderStatus string

const Pending OrderStatus = "pending"
```

This provides better type safety and clearer APIs.

---

# Interview Questions

### Why can an untyped constant be assigned to different numeric types?

Because it doesn't have a concrete type until the surrounding context requires one.

---

### Why can't variables behave like untyped constants?

Variables occupy memory at runtime and therefore must always have a specific machine type.

---

### What is an "ideal constant"?

It's the compiler's internal representation of an untyped constant—a value with arbitrary precision and no fixed machine type until needed.

---

### Why are typed constants useful?

They improve type safety and are commonly used with custom types to build enum-like APIs.

---

# Related Notes

- [[Constants]]
---

# Summary

- Go has **typed** and **untyped** constants.
- Untyped constants have a value but no fixed Go type initially.
- The compiler assigns a type only when required by the surrounding context.
- Untyped constants are more flexible than variables and support arbitrary-precision constant evaluation.
- Typed constants behave like immutable values of a specific type and are ideal for building enum-like APIs.
- Understanding this distinction is essential for mastering `iota`, enums, and Go's type system.