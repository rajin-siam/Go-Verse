

## What are Function Parameters?

A **parameter** is a variable declared in a function's definition that receives a value from the caller.

Think of it as an **input slot**.

```
Caller
   │
   ▼
Add(10, 20)
   │
   ▼
+----------------+
| a = 10         |
| b = 20         |
+----------------+
   │
   ▼
return 30
```

Example:

```go
func Add(a int, b int) int {
	return a + b
}
```

Here:

- `a` and `b` are **parameters**
- `10` and `20` are **arguments**

---

# Parameters vs Arguments

Many developers confuse these terms.

### Parameters

Declared in the function.

```go
func Add(a int, b int)
```

```
a

b
```

These are **parameters**.

---

### Arguments

Passed when calling the function.

```go
Add(10, 20)
```

```
10

20
```

These are **arguments**.

---

# Real World Analogy

Imagine ordering food.

Restaurant menu

```
Burger(Size, Cheese)
```

```
Size

Cheese
```

These are parameters.

Customer order

```
Burger("Large", true)
```

```
Large

true
```

These are arguments.

---

# Single Parameter

```go
func Square(n int) int {
	return n * n
}
```

Call

```go
Square(5)
```

Inside the function

```
n = 5
```

---

# Multiple Parameters

```go
func Add(a int, b int) int {
	return a + b
}
```

Call

```go
Add(3,4)
```

Result

```
a = 3

b = 4
```

---

# Shorthand Syntax

Instead of

```go
func Add(a int, b int)
```

Go allows

```go
func Add(a, b int)
```

Both mean exactly the same thing.

---

# Different Types

```go
func Register(name string, age int, active bool)
```

Parameters may have different types.

```
name

↓

string

age

↓

int

active

↓

bool
```

---

# Zero Parameters

Functions don't need parameters.

```go
func PrintVersion() {

	fmt.Println("v1.0")
}
```

Called as

```go
PrintVersion()
```

---

# What Happens During a Function Call?

Consider

```go
func Add(a, b int) int {
	return a + b
}

Add(10,20)
```

Conceptually, Go performs:

```
Caller

↓

10

↓

Copy

↓

a
```

```
Caller

↓

20

↓

Copy

↓

b
```

The function receives **copies** of the arguments.

---

# Go Uses Pass-by-Value

This is one of the most important rules in Go.

Every argument is **copied** into the function parameter.

Example

```go
func Increment(x int) {

	x++

}

func main() {

	a := 10

	Increment(a)

	fmt.Println(a)

}
```

Output

```
10
```

Why?

Because

```
a

↓

Copy

↓

x
```

Only `x` changes.

The original variable remains unchanged.

---

# Visualizing the Copy

Before calling

```
Main

a

↓

10
```

Function call

```
Increment(a)
```

Memory

```
Main

a

↓

10
```

```
Function

x

↓

10
```

Two separate variables.

Changing one does not affect the other.

---

# Passing Structs

```go
type User struct {
	Name string
}

func ChangeName(u User) {
	u.Name = "Bob"
}
```

Call

```go
user := User{"Alice"}

ChangeName(user)
```

Output

```
Alice
```

Why?

The entire struct was copied.

---

# Passing Pointers

If you want the function to modify the original value:

```go
func ChangeName(u *User) {
	u.Name = "Bob"
}
```

Call

```go
ChangeName(&user)
```

Now

```
Alice

↓

Bob
```

We'll study this deeply in [[Pointers]].

---

# Passing Arrays

Arrays are copied.

```go
func Update(arr [3]int) {

	arr[0] = 100

}
```

Original array remains unchanged.

Large arrays can be expensive to copy.

---

# Passing Slices

Many beginners think slices are passed by reference.

That's **not exactly true**.

A slice is a small structure containing:

```
Pointer

Length

Capacity
```

When passed to a function, **the slice header is copied**, not the underlying array.

```
Caller

Slice Header

↓

Copy

↓

Function

Slice Header
```

Both headers still point to the same underlying array.

This is why modifying slice elements inside a function is usually visible to the caller.

We'll explore this in detail in [[Slices]].

---

# Passing Maps

Maps behave similarly.

The map value itself is copied, but it contains a reference to runtime-managed hash table data.

```go
func Update(m map[string]int) {

	m["age"] = 25

}
```

Caller sees the change.

---

# Passing Interfaces

Interfaces are also passed by value.

An interface value contains:

```
Type Information

+

Underlying Value
```

The interface value is copied into the function parameter.

If the underlying value is a pointer, both copies refer to the same object.

---

# Large Objects

Suppose

```go
type Big struct {

	Data [100000]int

}
```

Passing

```go
func Process(b Big)
```

copies the entire array.

This can be expensive.

Sometimes using a pointer is a better choice.

---

# Should Everything Be a Pointer?

No.

Small values

```
int

bool

float64

time.Time (depending on context)
```

are often passed by value.

Pointers introduce shared mutable state, making code harder to reason about.

Use pointers when you need to:

- modify the original value
- avoid copying large objects
- represent optional (`nil`) values

---

# Compiler Internals

Conceptually, a function call looks like this:

```
Caller

↓

Arguments

↓

Copy Into Parameters

↓

Execute Function

↓

Return
```

The compiler places parameters in the function's **stack frame** (or uses registers on many modern architectures).

Small functions may even be **inlined**, removing the call entirely.

We'll explore this in [[Compiler Internals of Functions]].

---

# Common Mistakes

## Confusing Parameters and Arguments

```go
func Add(a,b int)
```

`a` and `b` are parameters.

```go
Add(1,2)
```

`1` and `2` are arguments.

---

## Assuming Go Uses Pass-by-Reference

Go always uses **pass-by-value**.

Sometimes the value being copied is itself a pointer or a descriptor (like a slice or map), which is why changes can appear to affect the caller.

---

## Copying Large Structs Unnecessarily

Large structs can be expensive to copy repeatedly.

Consider pointers when appropriate.

---

## Assuming Slices Behave Like Arrays

Arrays are fully copied.

Slices copy only their header.

This difference is fundamental in Go.

---

# Best Practices

✅ Prefer passing small values by value.

---

✅ Use pointers when the function must modify the caller's data.

---

✅ Avoid copying very large structs unnecessarily.

---

✅ Understand how slices, maps, and interfaces are represented before optimizing.

---

# Interview Questions

### Does Go support pass-by-reference?

No.

Go is strictly **pass-by-value**.

---

### Why can a function modify a slice?

Because the slice header is copied, but both copies point to the same underlying array.

---

### Are maps passed by reference?

No.

The map value is copied, but it contains a reference to shared runtime data.

---

### Why use pointers as parameters?

To modify the original value, avoid copying large objects, or allow `nil` to represent the absence of a value.

---

### What's the difference between parameters and arguments?

- **Parameters** are declared in the function definition.
- **Arguments** are the actual values supplied by the caller.

---

# Related Notes

- [[Functions]]
- [[Pointers]]
- [[Slices]]
- [[Maps]]
- [[Interfaces]]
- [[Escape Analysis]]
- [[Compiler Internals of Functions]]
- [[Function Memory Model]]

---

# Summary

- Parameters are variables declared in a function that receive values from the caller.
- Arguments are the actual values passed to a function.
- Go always passes arguments **by value**.
- Passing by value means the parameter receives a copy of the argument.
- Structs and arrays are copied in full.
- Slices, maps, and interfaces are also passed by value, but the copied values contain references to shared underlying data.
- Choose pointers when you need to modify the original value or avoid copying large data structures.