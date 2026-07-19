
## What are String Enums?

A **string enum** is a custom type whose underlying type is `string`, with a predefined set of constant values.

Example:

```go
type OrderStatus string

const (
	StatusPending    OrderStatus = "pending"
	StatusPreparing OrderStatus = "preparing"
	StatusReady      OrderStatus = "ready"
	StatusDelivered  OrderStatus = "delivered"
)
```

Instead of storing

```
0

1

2
```

we store

```
pending

preparing

ready
```

---

# Why Use String Enums?

Suppose your API returns:

```json
{
    "status": 2
}
```

A frontend developer asks:

> What does `2` mean?

You answer:

> It means Ready.

Now imagine instead:

```json
{
    "status": "ready"
}
```

No explanation needed.

The value explains itself.

---

# Basic Example

```go
package main

import "fmt"

type OrderStatus string

const (
	StatusPending    OrderStatus = "pending"
	StatusPreparing OrderStatus = "preparing"
	StatusReady      OrderStatus = "ready"
)

func main() {

	var status OrderStatus

	status = StatusReady

	fmt.Println(status)
}
```

Output

```
ready
```

---

# Why Create a Custom Type?

Without a custom type:

```go
const Pending = "pending"
```

Everything is just a string.

You could accidentally write:

```go
status := "banana"
```

Nothing stops you.

Instead:

```go
type OrderStatus string
```

Now your APIs clearly communicate:

```
This variable represents an order status.
```

---

# Type Safety

Consider:

```go
type PaymentStatus string

type OrderStatus string
```

These are different types.

```go
func Update(status OrderStatus) {}
```

Calling

```go
Update(PaymentStatus("paid"))
```

Produces a compile-time error.

Even though both use `string` underneath.

---

# String Enums and JSON

This is where string enums shine.

Struct

```go
type Order struct {
	Status OrderStatus `json:"status"`
}
```

Value

```go
order := Order{
	Status: StatusReady,
}
```

Marshalling

```go
json.Marshal(order)
```

Output

```json
{
    "status":"ready"
}
```

No extra work is required.

---

# Compare with Integer Enums

Integer enum

```go
type Status int

const (
	Pending Status = iota
	Ready
)
```

Produces

```json
{
    "status":1
}
```

Frontend developers now need documentation.

String enum

```json
{
    "status":"ready"
}
```

Much clearer.

---

# String Enums in Databases

Imagine opening a database.

Integer

| id | status |
|----|---------|
|1|2|

What is 2?

Nobody knows.

String

| id | status |
|----|-----------|
|1|ready|

Immediately understandable.

---

# Methods on String Enums

Because this is a custom type, methods can be added.

```go
func (s OrderStatus) IsFinished() bool {

	return s == StatusDelivered
}
```

Usage

```go
if status.IsFinished() {

}
```

This makes business logic more expressive.

---

# Validation

Go cannot stop someone from writing:

```go
status := OrderStatus("pizza")
```

The type is correct.

The value is not.

Always validate.

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

Usage

```go
if !status.IsValid() {

	return errors.New("invalid status")
}
```

---

# String Method

Sometimes you want pretty output.

```go
func (s OrderStatus) String() string {
	return string(s)
}
```

Now

```go
fmt.Println(StatusReady)
```

prints

```
ready
```

Even without a `String()` method, `fmt.Println` prints the underlying string value. A `String()` method becomes more useful if you want a different display format or if your enum's underlying type is not `string`.

---

# Parsing User Input

Suppose the client sends

```json
{
    "status":"ready"
}
```

After unmarshalling

```go
status := order.Status
```

Validate it.

```go
if !status.IsValid() {

	return errors.New("invalid status")
}
```

Never trust client input.

---

# Using String Enums in switch

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

default:

	fmt.Println("Unknown")
}
```

Readable and expressive.

---

# Helper Functions

Some teams prefer a helper.

```go
func ParseStatus(s string) (OrderStatus, error) {

	status := OrderStatus(s)

	if !status.IsValid() {
		return "", errors.New("invalid status")
	}

	return status, nil
}
```

Usage

```go
status, err := ParseStatus("ready")
```

This keeps validation logic in one place.

---

# Compiler Internals

Remember

```go
type OrderStatus string
```

creates

```
Named Type

↓

Underlying Type

↓

string
```

The compiler stores it exactly like a string.

There is no additional runtime cost for creating the custom type.

---

# Performance

Comparison

```go
status == StatusReady
```

is a string comparison.

This is generally slightly slower than comparing integers.

However,

```
Database

↓

HTTP

↓

JSON

↓

Network
```

are usually **much** slower than a single string comparison, so the difference is rarely significant in typical business applications.

Choose readability unless profiling shows a real bottleneck.

---

# Real Project Pattern

A common production layout:

```go
type OrderStatus string

const (
	StatusPending    OrderStatus = "pending"
	StatusPreparing OrderStatus = "preparing"
	StatusReady      OrderStatus = "ready"
	StatusDelivered  OrderStatus = "delivered"
)
```

Methods

```go
func (s OrderStatus) IsValid() bool

func (s OrderStatus) String() string

func ParseStatus(string) (OrderStatus, error)
```

Nearly every production backend has something similar.

---

# Common Mistakes

## Using Plain Strings

```go
status := "ready"
```

Bad.

Use

```go
status := StatusReady
```

---

## Comparing with String Literals

Bad

```go
if status == "ready"
```

Good

```go
if status == StatusReady
```

---

## Skipping Validation

Never assume client input is valid.

Always call

```go
status.IsValid()
```

or

```go
ParseStatus()
```

---

## Changing Existing Values

Suppose your database contains

```
pending
```

Don't rename it to

```
waiting
```

without a migration strategy.

External systems may depend on the original value.

---

# Best Practices

✅ Use custom types.

```go
type Status string
```

---

✅ Keep constant values lowercase.

```go
"pending"
```

This is the common Go convention for JSON and APIs.

---

✅ Add `IsValid()`.

---

✅ Add a parsing helper for external input.

---

✅ Compare against constants, never raw string literals.

---

# Interview Questions

### Why are string enums preferred in REST APIs?

Because they are human-readable, self-describing, and stable across versions.

---

### Can invalid values still exist?

Yes.

```go
OrderStatus("banana")
```

is legal.

Validation is still required.

---

### Do string enums use more memory than integer enums?

Yes, but the difference is usually insignificant for API-facing applications. Favor clarity unless you have measured performance reasons to do otherwise.

---

### Why use a custom type instead of plain strings?

It improves API clarity, enables methods, and prevents mixing unrelated concepts like `OrderStatus` and `PaymentStatus`.

---

# Related Notes

- [[Constants]]
---

# Summary

- String enums are custom types whose underlying type is `string`.
- They are the preferred choice for JSON, REST APIs, databases, logs, and message queues.
- They make data self-describing and easier to debug.
- Go does not enforce valid enum values, so validation is essential.
- Add helper methods like `IsValid()` and parsing functions to centralize validation logic.
- Use constants instead of raw string literals throughout your code.