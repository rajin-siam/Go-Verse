# Iota

## What is `iota`?

`iota` is a special identifier in Go that is used **only inside constant declarations**.

It automatically generates successive integer values.

Think of it as the compiler keeping a counter while processing a `const` block.

Example:

```go
const (
	A = iota
	B
	C
)
```

The compiler replaces it with:

```go
const (
	A = 0
	B = 1
	C = 2
)
```

Output:

```
A = 0
B = 1
C = 2
```

---

# Why Does `iota` Exist?

Imagine writing:

```go
const (
	Pending = 0
	Preparing = 1
	Ready = 2
	Delivered = 3
	Cancelled = 4
)
```

This works.

But what happens when you insert a new status?

```go
const (
	Pending = 0
	Confirmed = 1
	Preparing = 2
	Ready = 3
	Delivered = 4
	Cancelled = 5
)
```

You must manually update every value after the insertion.

`iota` solves this.

```go
const (
	Pending = iota
	Confirmed
	Preparing
	Ready
	Delivered
	Cancelled
)
```

The compiler assigns the numbers automatically.

---


---

# Basic Example

```go
const (
	A = iota
	B
	C
	D
)
```

Compiler sees:

```
Line 1 → 0

Line 2 → 1

Line 3 → 2

Line 4 → 3
```

Result

```go
A == 0

B == 1

C == 2

D == 3
```

---

# How Does the Compiler Think?

Imagine the compiler processing the block.

```
Counter = 0

A = iota

↓

Counter++

↓

Counter = 1

B

↓

Counter++

↓

Counter = 2

C
```

Every **new line** inside the `const` block increments `iota`.

---

# `iota` Resets

Every new `const` block starts over.

```go
const (
	A = iota
	B
)
```

Result

```
A = 0

B = 1
```

Another block

```go
const (
	X = iota
	Y
)
```

Result

```
X = 0

Y = 1
```

Each block has its own counter.

---

# Expression Reuse

One of Go's nicest features:

```go
const (
	A = iota * 10
	B
	C
	D
)
```

The compiler repeats the previous expression.

Equivalent to:

```go
const (
	A = 0 * 10
	B = 1 * 10
	C = 2 * 10
	D = 3 * 10
)
```

Output

```
0

10

20

30
```

---

# Multiple Constants Per Line

```go
const (
	A, B = iota, iota
	C, D
)
```

Compiler sees

```
A = 0

B = 0

C = 1

D = 1
```

Notice:

`iota` is evaluated **once per line**, not once per constant.

---

# Starting From 1

Many APIs prefer starting at 1.

```go
const (
	Pending = iota + 1
	Preparing
	Ready
)
```

Compiler

```
1

2

3
```

---

# Using Arithmetic

```go
const (
	A = iota * 5
	B
	C
	D
)
```

Compiler

```
0

5

10

15
```

---

# Bit Flags

This is one of the most powerful uses of `iota`.

```go
const (
	Read = 1 << iota
	Write
	Execute
)
```

Compiler

```
Read    = 1

Write   = 2

Execute = 4
```

Binary

```
001

010

100
```

These values can be combined using bitwise OR.

```go
permission := Read | Write
```

Result

```
011
```

Meaning:

- Read ✓
- Write ✓
- Execute ✗

Bit flags are used throughout operating systems, networking, and file permissions.

---

# Enum Pattern

Most Go enums use `iota`.

```go
type OrderStatus int

const (
	StatusPending OrderStatus = iota
	StatusPreparing
	StatusReady
	StatusDelivered
)
```

Output

```
0

1

2

3
```

This is the standard enum pattern in Go.

---

# String Enums?

`iota` generates **integers**, not strings.

This is **wrong**:

```go
const (
	Pending = iota
)
```

because it produces

```
0
```

If you want

```
pending
```

use string constants instead.

```go
type OrderStatus string

const (
	StatusPending OrderStatus = "pending"
	StatusPreparing OrderStatus = "preparing"
	StatusReady OrderStatus = "ready"
)
```

---

# Compiler Internals

`iota` does **not** exist at runtime.

The compiler replaces every occurrence during compilation.

Example

```go
const (
	A = iota
	B
	C
)
```

Generated values

```
0

1

2
```

After compilation, there is no `iota` variable anywhere.

It is purely a compile-time construct.

---

# Common Mistakes

## Thinking `iota` Is a Variable

Wrong.

```
iota++
```

Impossible.

It is not a runtime variable.

---

## Expecting It to Continue Across Blocks

```go
const (
	A = iota
	B
)

const (
	C = iota
)
```

Output

```
A = 0

B = 1

C = 0
```

---

## Using `iota` for Database Values

Imagine

```
Pending

Preparing

Ready
```

Today they are

```
0

1

2
```

Tomorrow you insert

```
Confirmed
```

Now

```
Pending = 0

Confirmed = 1

Preparing = 2
```

Your database rows now refer to different meanings.

For values stored externally (databases, APIs, files), changing `iota`-based enums can break compatibility.

---

# Best Practices

✅ Use `iota` for internal numeric enums.

```go
type Direction int

const (
	North Direction = iota
	South
	East
	West
)
```

---

✅ Use string constants for APIs and JSON.

```go
type Status string

const (
	StatusPending Status = "pending"
	StatusReady Status = "ready"
)
```

---

✅ Use bit flags with `1 << iota` when representing combinations of options.

---

❌ Don't rely on `iota` values remaining stable if they're persisted outside your application.

---

# Interview Questions

### What is `iota`?

A compile-time identifier that produces successive integer values within a `const` block.

---

### Does `iota` exist at runtime?

No. The compiler replaces it with constant values during compilation.

---

### When does `iota` reset?

At the beginning of each new `const` block.

---

### Why is `1 << iota` commonly used?

It generates powers of two (`1, 2, 4, 8, ...`), allowing values to be combined efficiently as bit flags.

---

### Is `iota` suitable for JSON or database values?

Usually no. Numeric `iota` values are convenient internally, but string constants are generally more stable and readable for external representations.

---

# Related Notes

- [[Constants]]

---

# Summary

- `iota` is a compile-time identifier used only in `const` declarations.
- It starts at `0` and increments once per line.
- Each `const` block gets its own `iota` sequence.
- Expressions involving `iota` are repeated on subsequent lines.
- `1 << iota` is the standard pattern for creating bit flags.
- `iota` is excellent for internal numeric enums but should be used carefully for values persisted outside the program.