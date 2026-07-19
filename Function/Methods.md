
## What is a Method?

A **method** is a function that is associated with a specific type.

Unlike a normal function, a method has a **receiver**.

Normal function:

```go
func Add(a, b int) int {
	return a + b
}
```

Method:

```go
type User struct {
	Name string
}

func (u User) Greet() {
	fmt.Println("Hello", u.Name)
}
```

The receiver `(u User)` connects the function to the `User` type.

---

# Why Do We Need Methods?

Suppose you have a `User`.

```go
type User struct {
	Name string
}
```

Without methods:

```go
func Greet(u User) {
	fmt.Println("Hello", u.Name)
}
```

With methods:

```go
func (u User) Greet() {
	fmt.Println("Hello", u.Name)
}
```

Calling:

```go
user.Greet()
```

is much more natural than

```go
Greet(user)
```

The behavior belongs to the type itself.

---

# Real World Analogy

Think of a car.

A car has:

```
Data

↓

Color

Brand

Speed
```

And behaviors:

```
Start()

Stop()

Accelerate()
```

The behaviors belong to the car.

Similarly,

```
User

↓

Login()

Logout()

ChangePassword()
```

Methods represent what a type **can do**.

---

# Method Syntax

```go
func (receiver Type) MethodName(parameters) returnType {

}
```

Example:

```go
type Rectangle struct {
	Width  float64
	Height float64
}

func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}
```

Call:

```go
rect := Rectangle{10, 5}

fmt.Println(rect.Area())
```

Output

```
50
```

---

# Understanding the Receiver

The receiver appears before the method name.

```go
func (r Rectangle) Area() float64
```

Here:

```
Receiver Variable

↓

r

Receiver Type

↓

Rectangle
```

Think of the receiver as a special parameter.

---

# Receiver vs Parameter

Method

```go
func (u User) Greet()
```

Normal function

```go
func Greet(u User)
```

Conceptually, they're similar.

The difference is how they're called.

Function:

```go
Greet(user)
```

Method:

```go
user.Greet()
```

Methods improve readability and organization.

---

# Value Receiver

```go
type Counter struct {
	Value int
}

func (c Counter) Increment() {
	c.Value++
}
```

Usage:

```go
counter := Counter{10}

counter.Increment()

fmt.Println(counter.Value)
```

Output

```
10
```

Why?

Because the receiver is **copied**.

Memory:

```
counter

↓

Copy

↓

c
```

Only the copy changes.

---

# Pointer Receiver

```go
func (c *Counter) Increment() {
	c.Value++
}
```

Now:

```go
counter := Counter{10}

counter.Increment()

fmt.Println(counter.Value)
```

Output

```
11
```

The receiver points to the original object.

Memory:

```
counter

↓

Pointer

↓

Original Value
```

---

# Value vs Pointer Receiver

### Value Receiver

```
Original

↓

Copy

↓

Method
```

Changes do **not** affect the original value.

---

### Pointer Receiver

```
Original

↓

Pointer

↓

Method
```

Changes affect the original value.

---

# Automatic Addressing

Go automatically handles many method calls.

```go
counter := Counter{}

counter.Increment()
```

Even though `Increment` expects a pointer receiver.

The compiler effectively treats it as:

```go
(&counter).Increment()
```

Likewise:

```go
ptr := &counter

ptr.Value()
```

can call a value receiver because the compiler dereferences the pointer when needed.

This convenience only works for **addressable values**.

---

# Methods on Non-Struct Types

Methods are **not limited to structs**.

Example:

```go
type Celsius float64

func (c Celsius) Fahrenheit() float64 {
	return float64(c)*9/5 + 32
}
```

Usage:

```go
temp := Celsius(30)

fmt.Println(temp.Fahrenheit())
```

Methods can be attached to any **named type**.

---

# Method Sets

Every type has a **method set**.

Given:

```go
type User struct{}
```

```go
func (u User) A() {}
```

```go
func (u *User) B() {}
```

Method set of `User`

```
A()
```

Method set of `*User`

```
A()

B()
```

This becomes very important when working with interfaces.

---

# Methods and Interfaces

Consider:

```go
type Speaker interface {
	Speak()
}
```

```go
type Dog struct{}

func (d Dog) Speak() {
	fmt.Println("Woof")
}
```

Now:

```go
var s Speaker

s = Dog{}
```

Works because `Dog` implements the interface by providing the required method.

Methods are how types satisfy interfaces in Go.

---

# Methods Are Not Inside Structs

Coming from languages like Java or C++, you might think methods are stored inside the struct.

Not in Go.

The struct:

```go
type User struct {
	Name string
}
```

contains only data.

The method:

```go
func (u User) Greet() {}
```

is a separate function that the compiler associates with the `User` type.

---

# Compiler Internals

This method call:

```go
user.Greet()
```

is conceptually transformed into something like:

```go
Greet(user)
```

The receiver becomes the first argument.

For pointer receivers:

```go
user.Increment()
```

is conceptually:

```go
Increment(&user)
```

The exact implementation differs, but this mental model explains method calls well.

---

# Memory Model

Value receiver:

```
Original Object

↓

Copy

↓

Method
```

Pointer receiver:

```
Original Object

↓

Pointer

↓

Method
```

Choose the receiver type based on whether you need to modify the original object or avoid copying.

---

# When to Use Value Receivers

Use value receivers when:

- The method doesn't modify the receiver.
- The type is small (for example, a couple of fields).
- The type behaves like an immutable value.

Example:

```go
type Point struct {
	X, Y int
}
```

---

# When to Use Pointer Receivers

Use pointer receivers when:

- The method modifies the receiver.
- The struct is large and expensive to copy.
- You want a consistent method set, especially if some methods already require pointers.

Example:

```go
type Config struct {
	Data [100000]int
}
```

Copying this struct repeatedly is inefficient.

---

# Common Mistakes

## Expecting Value Receivers to Modify Data

```go
func (u User) Rename() {
	u.Name = "Bob"
}
```

The original value remains unchanged.

---

## Mixing Receiver Types Without Reason

Avoid randomly using both value and pointer receivers on the same type.

Be consistent.

A common guideline:

- If one method needs a pointer receiver, consider making all methods pointer receivers unless you have a good reason not to.

---

## Confusing Methods with Functions

Method:

```go
user.Login()
```

Function:

```go
Login(user)
```

Methods belong to a type.

Functions stand alone.

---

# Best Practices

✅ Use pointer receivers when modifying the receiver.

---

✅ Keep receiver names short.

Common choices:

```go
(u User)

(c Config)

(r Rectangle)
```

Avoid names like:

```go
(receiver User)
```

---

✅ Be consistent with receiver types.

---

✅ Use methods to express behavior that naturally belongs to the type.

---

# Interview Questions

### What is a method?

A function associated with a named type through a receiver.

---

### What is a receiver?

The special parameter that specifies which type the method belongs to.

---

### What's the difference between value and pointer receivers?

- Value receivers operate on a copy.
- Pointer receivers operate on the original value.

---

### Can methods exist on non-struct types?

Yes.

Methods can be attached to any named type.

---

### Why are methods important for interfaces?

Interfaces are implemented implicitly by providing the required methods.

---

# Related Notes

- [[Functions]]
- [[Pointers]]
- [[Interfaces]]
- [[Structs]]
- [[Method Sets]]
- [[Receiver]]
- [[Compiler Internals of Functions]]
- [[Escape Analysis]]

---

# Summary

- A method is a function associated with a named type.
- Methods use receivers to access the value they operate on.
- Value receivers receive a copy of the value.
- Pointer receivers receive a pointer to the original value.
- Methods can be defined on any named type, not just structs.
- Methods determine whether a type satisfies an interface.
- Choose receiver types consistently based on whether mutation or efficient access is required.