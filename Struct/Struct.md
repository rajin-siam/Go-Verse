

> [!INFO]
> **Struct** is a composite data type in Go that groups multiple related fields into a single unit. It is similar to a class in other languages **only for holding data**, but unlike classes, structs do not support inheritance.

---

## Why Do We Need Structs?

Imagine you're building a coffee shop application.

Without structs:

```go
var orderID int
var customerName string
var total float64
var status string
```

As the application grows, managing related variables separately becomes difficult.

With a struct:

```go
type Order struct {
	ID       int
	Customer string
	Total    float64
	Status   string
}
```

Now all information related to an order is grouped together.

> [!TIP]
> Use a struct whenever multiple values logically belong together.

---

## Syntax

```go
type StructName struct {
	Field1 Type
	Field2 Type
	Field3 Type
}
```

Example:

```go
type User struct {
	Name string
	Age  int
	City string
}
```

---


## Creating a Struct

### Method 1: Using Field Names (Recommended)

```go
package main

import "fmt"

type User struct {
	Name string
	Age  int
	City string
}

func main() {
	user := User{
		Name: "Alice",
		Age:  24,
		City: "Dhaka",
	}

	fmt.Println(user)
}
```

Output

```
{Alice 24 Dhaka}
```

> [!TIP]
> Named fields improve readability and prevent mistakes when field order changes.

---

### Method 2: Positional Initialization

```go
user := User{"Alice", 24, "Dhaka"}
```

Output

```
{Alice 24 Dhaka}
```

> [!WARNING]
> Avoid positional initialization in large projects because the meaning of each value is not obvious.

---

### Method 3: Zero Value

```go
var user User

fmt.Println(user)
```

Output

```
{ 0 }
```

Every field receives its zero value.

| Type | Zero Value |
|-------|------------|
| int | 0 |
| float | 0 |
| string | "" |
| bool | false |
| pointer | nil |
| slice | nil |
| map | nil |

---

## Accessing Fields

Use the dot (`.`) operator.

```go
fmt.Println(user.Name)
fmt.Println(user.Age)
```

Output

```
Alice
24
```

---

## Updating Fields

```go
user.Age = 25
user.City = "Chattogram"

fmt.Println(user)
```

Output

```
{Alice 25 Chattogram}
```

---




## Nested Struct

Structs can contain other structs.

```go
package main

import "fmt"

type Address struct {
	City    string
	Country string
}

type User struct {
	Name    string
	Address Address
}

func main() {
	user := User{
		Name: "Alice",
		Address: Address{
			City:    "Dhaka",
			Country: "Bangladesh",
		},
	}

	fmt.Println(user.Address.City)
}
```

Output

```
Dhaka
```

## Real-World Example

```go
package main

import "fmt"

type Order struct {
	ID       int
	Customer string
	Items    []string
	Total    float64
	Paid     bool
}

func main() {
	order := Order{
		ID:       101,
		Customer: "Rahim",
		Items: []string{
			"Latte",
			"Brownie",
		},
		Total: 8.50,
		Paid:  true,
	}

	fmt.Printf("%+v\n", order)
}
```

Output

```
{ID:101 Customer:Rahim Items:[Latte Brownie] Total:8.5 Paid:true}
```

---

## Common Mistakes

### 1. Using Positional Initialization

```go
User{"Alice", 20, "Dhaka"}
```

Prefer:

```go
User{
	Name: "Alice",
	Age: 20,
	City: "Dhaka",
}
```

---

### 2. Forgetting Zero Values

```go
var user User
fmt.Println(user.Name)
```

Output

```
""
```

The field exists, but it has its zero value.

---

### 3. Expecting Struct Embedding to Behave Like Inheritance

Embedding shares fields and methods but does not create an "is-a" relationship like inheritance in object-oriented languages.

---

## Best Practices

- Use structs to model real-world entities.
- Prefer named-field initialization.
- Keep structs focused on a single responsibility.
- Embed structs only when it expresses a clear "has-a" relationship.
- Add struct tags when working with JSON, databases, or validation libraries.
- Use pointer receivers on methods when the method modifies the struct or copying it would be expensive.

---

## Related Pages

- [[Types]]
- [[Pointers]]
- [[Methods]]
- [[Method Receivers]]
- [[Struct Embedding]]
- [[Composition]]
- [[Interfaces]]
- [[Zero Values]]
- [[JSON Marshalling]]
- [[Memory Layout]]