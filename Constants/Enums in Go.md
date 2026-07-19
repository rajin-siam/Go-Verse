## What is an Enum?

An **enum** (short for *enumeration*) is a type that can have **only one value from a predefined set**.

For example, an order can only be in one of these states:

- Pending
- Preparing
- Ready
- Delivered
- Cancelled

Not:

- Banana
- 999
- "Hello"

Enums make your code **more readable**, **type-safe**, and **less error-prone**.

---

# Does Go Have Enums?

No.

Go has **no `enum` keyword**.

Instead, Go creates enum-like behavior using:

- A custom type
- Constants

Example:

```go
type OrderStatus int

const (
	StatusPending OrderStatus = iota
	StatusPreparing
	StatusReady
	StatusDelivered
)
```

This is the standard Go enum pattern.

---

# Why Doesn't Go Have an enum Keyword?

Go's designers wanted to keep the language small and simple.

Instead of introducing a special language feature, they realized that:

- Custom types
- Constants
- Methods
- Interfaces

already provide everything needed to model enums.

---

# Building Your First Enum

```go
type OrderStatus int

const (
	StatusPending OrderStatus = iota
	StatusPreparing
	StatusReady
	StatusDelivered
)
```

Here:

```
Custom Type

↓

OrderStatus
```

Possible values

```
0

1

2

3
```

---

# Type Safety

Consider two enums:

```go
type OrderStatus int

type PaymentStatus int
```

Constants

```go
const (
	OrderPending OrderStatus = iota
	OrderReady
)

const (
	PaymentPending PaymentStatus = iota
	PaymentPaid
)
```

Now imagine:

```go
func Ship(status OrderStatus) {}
```

Calling:

```go
Ship(PaymentPaid)
```

Produces a compile-time error because `PaymentStatus` and `OrderStatus` are different types.

This is one of the biggest advantages of custom types.

---

# Can Invalid Values Still Exist?

Yes.

This surprises many Go developers.

```go
var status OrderStatus = 100
```

This compiles.

Why?

Because `OrderStatus` is still based on `int`.

The compiler only checks the type, not whether the value is one of your declared constants.

This means Go enums are **conventions**, not strict language-enforced enums.

---

# Solving This with Validation

A common pattern is to validate values.

```go
func (s OrderStatus) IsValid() bool {
	switch s {
	case StatusPending,
		StatusPreparing,
		StatusReady,
		StatusDelivered:
		return true
	}

	return false
}
```

Usage:

```go
status := OrderStatus(100)

if !status.IsValid() {
	fmt.Println("invalid status")
}
```

---

# Using Enums in switch

Enums work naturally with `switch`.

```go
switch status {

case StatusPending:
	fmt.Println("Waiting")

case StatusPreparing:
	fmt.Println("Cooking")

case StatusReady:
	fmt.Println("Ready")

case StatusDelivered:
	fmt.Println("Delivered")
}
```

Much better than:

```go
switch status {

case 0:

case 1:

case 2:
}
```

Named constants make the code self-documenting.

---

# Adding Methods to Enums

Because enums are custom types, you can attach methods.

Example:

```go
func (s OrderStatus) String() string {
	switch s {

	case StatusPending:
		return "Pending"

	case StatusPreparing:
		return "Preparing"

	case StatusReady:
		return "Ready"

	case StatusDelivered:
		return "Delivered"

	default:
		return "Unknown"
	}
}
```

Now:

```go
fmt.Println(StatusReady)
```

Output (with `%v` or `Println`):

```
2
```

But:

```go
fmt.Println(StatusReady.String())
```

Output

```
Ready
```

> **Advanced note:** If your type implements `String() string`, packages like `fmt` automatically use it when formatting with verbs such as `%v` or when using `fmt.Println`.

---

# Numeric Enums

Most internal Go enums use integers.

```go
type Direction int

const (
	North Direction = iota
	South
	East
	West
)
```

Advantages

- Fast comparisons
- Small memory footprint
- Good for internal logic

Disadvantages

- Not human-readable
- Poor for APIs and JSON without conversion

---

# String Enums

Another common approach:

```go
type OrderStatus string

const (
	StatusPending OrderStatus = "pending"
	StatusPreparing OrderStatus = "preparing"
	StatusReady OrderStatus = "ready"
)
```

Advantages

- Human-readable
- Great for JSON
- Great for databases
- Stable across versions

Disadvantages

- Slightly larger than integers
- Comparisons are generally a bit slower than integer comparisons (though usually not a practical concern)

---

# Which Should You Choose?

## Internal Logic

```go
type Direction int
```

Great choice.

---

## APIs

```go
type Status string
```

Better choice.

Clients understand

```
pending
```

more easily than

```
2
```

---

## Databases

String enums are often preferred because they remain meaningful when viewed directly in the database and are less likely to break if enum values are reordered.

---

# Compiler Internals

Remember:

```go
type OrderStatus int
```

does **not** create a brand-new primitive type.

It creates a **new named type** whose **underlying type** is `int`.

Conceptually:

```
OrderStatus

↓

Underlying Type

↓

int
```

The compiler treats it as a distinct type for assignments and function parameters, which provides type safety.

---

# Common Mistakes

## Using Plain Integers

```go
const (
	Pending = 0
	Ready = 1
)
```

These are just integers.

A custom type provides much better clarity.

---

## Comparing with Magic Numbers

Bad

```go
if status == 2
```

Good

```go
if status == StatusReady
```

---

## Forgetting Validation

This compiles:

```go
status := OrderStatus(999)
```

Always validate enum values when they come from user input, APIs, or databases.

---

## Using Numeric Enums in Public APIs

Clients shouldn't have to remember:

```
0 = Pending

1 = Ready

2 = Delivered
```

Prefer string values for external communication.

---

# Best Practices

✅ Create a custom type for every enum.

```go
type OrderStatus string
```

---

✅ Use descriptive constant names.

```go
StatusPending
```

instead of

```go
Pending
```

This avoids naming collisions across packages.

---

✅ Add helper methods like `String()` and `IsValid()`.

---

✅ Prefer string enums for APIs, JSON, and databases.

---

✅ Prefer `iota`-based integer enums for internal state machines and algorithms.

---

# Interview Questions

### Why doesn't Go have native enums?

Because Go favors a small language. Custom types and constants provide similar functionality without introducing another language feature.

---

### Are Go enums truly type-safe?

They are type-safe in the sense that different named types cannot be mixed accidentally. However, any value of the underlying type can still be explicitly converted, so runtime validation is often needed.

---

### Why create a custom type instead of using `int`?

A custom type makes APIs clearer, prevents mixing unrelated concepts, and allows methods such as `String()` and `IsValid()`.

---

### Why are string enums popular in REST APIs?

They are readable, self-describing, and remain stable even if the internal implementation changes.

---

# Related Notes

- [[Constants]]
---

# Summary

- Go has no built-in `enum` keyword.
- Enums are modeled using **custom types + constants**.
- Custom types improve readability and prevent mixing unrelated concepts.
- Integer enums are efficient for internal logic.
- String enums are ideal for APIs, JSON, and databases.
- Enum values should be validated when they come from external sources.
- Methods like `String()` and `IsValid()` make enums easier to use and maintain.