## Why Learn This?

Most developers know that constants "cannot change."

But the more interesting questions are:

- Why are constants sometimes faster than variables?
- Why can constants hold enormous numbers?
- Why do untyped constants behave differently?
- Where are constants stored?
- Does memory get allocated for constants?

To answer these, we need to understand what the **Go compiler** does.

---

# The Journey of a Go Program

When you run:

```bash
go build
```

Your code goes through several stages.

```
Go Source Code
       │
       ▼
Lexical Analysis
       │
       ▼
Parsing
       │
       ▼
Type Checking
       │
       ▼
Constant Evaluation
       │
       ▼
Optimization
       │
       ▼
Machine Code
```

Notice that **constant evaluation happens before the executable is produced**.

This is why constants are called **compile-time values**.

---

# Variables vs Constants

Consider this:

```go
const A = 10

var B = 10
```

Although they look similar, the compiler treats them very differently.

---

## Variable

```
Memory

+-------+
|  10   |
+-------+
```

The program stores `10` in memory.

Whenever the CPU needs `B`, it loads the value from memory.

Conceptually:

```
CPU

↓

Memory

↓

10
```

---

## Constant

```
const A = 10
```

The compiler often replaces every use of `A` with `10`.

Example:

```go
fmt.Println(A)
```

becomes conceptually:

```go
fmt.Println(10)
```

No lookup is required.

---

# Constant Substitution

This optimization is called **constant substitution**.

Source code

```go
const MaxRetries = 5

if retries > MaxRetries {

}
```

Compiler sees

```go
if retries > 5 {

}
```

The name disappears.

Only the value remains.

---

# Constant Folding

The compiler also evaluates expressions.

Example

```go
const A = 10

const B = 20

const C = A + B
```

Compiler performs

```
10 + 20

↓

30
```

Generated program

```go
const C = 30
```

The addition never happens while your program is running.

---

# More Examples

```go
const KB = 1024

const MB = KB * 1024

const GB = MB * 1024
```

Compiler calculates

```
KB = 1024

MB = 1048576

GB = 1073741824
```

The executable already contains the final values.

---

# Runtime vs Compile Time

Compile time

```
const A = 5 + 10
```

Compiler computes

```
15
```

Runtime

```go
x := readFromDatabase()

y := x + 10
```

The compiler cannot know `x`.

The CPU performs the addition while the program runs.

---

# Ideal (Untyped) Constants

One of Go's most unique features is the concept of **ideal constants**.

Example

```go
const X = 100
```

Notice:

No type.

Internally, the compiler does **not** immediately treat this as an `int`.

Instead it stores something like

```
Ideal Integer

↓

100
```

Later, when needed

```
100

↓

int

↓

int64

↓

float64
```

This is why untyped constants are so flexible.

---

# Arbitrary Precision

Variables must fit inside memory.

```
int64

64 bits
```

Constants are different.

Example

```go
const Huge = 1000000000000000000000000000000000000000000
```

This compiles.

Why?

Because the compiler stores the value using **arbitrary precision** while evaluating constant expressions.

It does not immediately choose a machine-sized integer.

---

# When Errors Happen

This compiles

```go
const Huge = 100000000000000000000000000
```

But this does not

```go
var x int64 = Huge
```

Why?

Because now the compiler must fit the value into an `int64`.

It checks:

```
Can Huge fit inside int64?

↓

No

↓

Compile Error
```

---

# Compile-Time Arithmetic

Go evaluates constant expressions before execution.

```go
const A = 5

const B = A * 20

const C = B + 100

const D = C / 4
```

Compiler computes

```
5

↓

100

↓

200

↓

50
```

The executable contains only the final results.

---

# Why Can't Slices Be Constants?

Consider

```go
[]int{1,2,3}
```

A slice contains:

```
Pointer

Length

Capacity
```

These depend on runtime memory allocation.

The compiler cannot represent them as compile-time constants.

Similarly,

- maps
- channels
- functions
- structs
- slices

cannot be constants.

---

# String Constants

Strings are constants.

```go
const Name = "Siam"
```

The compiler knows the value at compile time.

The string data is stored in the program's read-only data section, and uses of the constant refer to that immutable data.

---

# Boolean Constants

```go
const Enabled = true
```

The compiler substitutes

```
true
```

wherever it is used.

---

# Floating-Point Constants

```go
const Pi = 3.141592653589793
```

The compiler keeps high precision during constant evaluation.

Only when assigning to

```go
float32
```

does rounding occur.

---

# Constant Propagation

Suppose

```go
const Timeout = 30

func Retry() {

	wait := Timeout

}
```

Compiler sees

```go
wait := 30
```

The constant value propagates through the code.

This is another optimization.

---

# Does a Constant Occupy Memory?

This question has a subtle answer.

### Example 1

```go
const A = 10

fmt.Println(A)
```

Usually, **no separate storage is allocated** for `A`.

The compiler substitutes `10` directly.

---

### Example 2

```go
const Message = "Hello"
```

The string data itself exists in the compiled program's read-only data section.

The constant name `Message` is a compile-time concept—it doesn't become a runtime variable.

So it's better to think of it this way:

- **Simple numeric constants** are often substituted directly into instructions.
- **String constants** refer to immutable data embedded in the executable.

The exact implementation depends on the compiler and architecture.

---

# Compiler Optimizations

The Go compiler performs several optimizations involving constants.

### Constant Folding

```
10 + 20

↓

30
```

---

### Constant Propagation

```
Timeout

↓

30
```

---

### Dead Code Elimination

```go
const Debug = false

if Debug {

	fmt.Println("debug")
}
```

The compiler knows the condition is always false.

It may remove the entire `if` block from the generated code.

---

# Why Compile-Time Evaluation Matters

Imagine

```go
const KB = 1024

const MB = KB * 1024

const GB = MB * 1024
```

Without compile-time evaluation

Every program execution would perform:

```
1024 × 1024

↓

1048576

↓

×1024

↓

1073741824
```

Instead, the compiler performs these calculations once during compilation.

---

# Common Misconceptions

## "Constants are stored like variables."

Not necessarily.

Many numeric constants are substituted directly into the generated instructions.

---

## "Untyped constants are int."

Wrong.

Initially they are **ideal constants**.

Only later does the compiler choose a concrete type.

---

## "Constants are faster because they're immutable."

Not exactly.

They are often faster because the compiler can evaluate expressions and substitute values before the program runs.

---

## "Everything known at compile time can be a constant."

Not true.

A value must also be one of Go's supported constant kinds (numbers, strings, runes, booleans, and constant expressions). Structs, slices, maps, and channels are not allowed as constants, even if their contents are known.

---

# Best Practices

✅ Use constants for values known at compile time.

---

✅ Let the compiler evaluate arithmetic.

```go
const MB = 1024 * 1024
```

---

✅ Prefer meaningful constant names over magic numbers.

---

✅ Understand that `const` is primarily a **compile-time feature**, not a runtime storage mechanism.

---

# Interview Questions

### Why can constants hold huge numbers?

Because the compiler represents untyped constants with arbitrary precision during constant evaluation, instead of immediately choosing a fixed machine type.

---

### Does a constant always occupy memory?

No. Numeric constants are often substituted directly into the generated code. String constants refer to immutable data embedded in the executable.

---

### What is constant folding?

The compiler evaluates constant expressions during compilation.

Example:

```
10 + 20

↓

30
```

---

### What is constant propagation?

Replacing references to constants with their actual values during compilation.

---

### Why can't slices be constants?

Because slices contain runtime state (pointer, length, and capacity), which requires memory allocation and cannot be represented as a compile-time constant.

---

# Related Notes

- [[Constants]]
---

# Summary

- Constants are evaluated during compilation.
- The compiler performs **constant substitution**, **constant folding**, and **constant propagation**.
- Untyped constants are represented internally as ideal values with arbitrary precision.
- Numeric constants often do not require separate runtime storage.
- String constants refer to immutable data embedded in the executable.
- Compile-time evaluation reduces runtime work and enables additional compiler optimizations.