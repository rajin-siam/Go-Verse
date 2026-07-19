
## What is JSON Unmarshalling?

**JSON Unmarshalling** is the process of converting JSON data into Go objects (such as structs, maps, or slices).

It is the **opposite of JSON Marshalling**.

Go uses the `encoding/json` package to perform this conversion.

```
JSON String
     │
     ▼
Unmarshalling
     │
     ▼
Go Struct
```

---

# Why Do We Need It?

Applications often receive data from external sources:

- HTTP request bodies
- REST APIs
- Configuration files
- Databases
- Message queues

These sources usually provide data as JSON, but your Go program works with Go types.

Unmarshalling translates JSON into Go objects so your application can use the data.

---

# Basic Example

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name string
	Age  int
}

func main() {
	data := []byte(`{"Name":"Siam","Age":24}`)

	var user User

	err := json.Unmarshal(data, &user)
	if err != nil {
		panic(err)
	}

	fmt.Println(user.Name)
	fmt.Println(user.Age)
}
```

Output

```
Siam
24
```

---

# Why Do We Pass a Pointer?

Notice this line:

```go
json.Unmarshal(data, &user)
```

We pass **`&user`**, not `user`.

Why?

Because `Unmarshal()` needs permission to modify the variable.

If you pass:

```go
json.Unmarshal(data, user)
```

Go would only receive a copy of `user`.

Any changes would disappear after the function returns.

Passing a pointer allows `Unmarshal()` to fill the original struct.

```
user
 │
 ▼
Memory
+----------------+
| Name: ""       |
| Age: 0         |
+----------------+

      ▲
      │
json.Unmarshal()
      │
writes values
```

---

# Using JSON Tags

Suppose the JSON looks like this:

```json
{
    "name": "Siam",
    "age": 24
}
```

The struct should use matching JSON tags.

```go
type User struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}
```

Now the JSON keys correctly map to the Go fields.

---

# What Happens If a Field Is Missing?

```json
{
    "name": "Siam"
}
```

Struct

```go
type User struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}
```

Result

```go
User{
	Name: "Siam",
	Age: 0,
}
```

Missing fields simply keep their **zero value**.

---

# What If JSON Has Extra Fields?

```json
{
    "name": "Siam",
    "age": 24,
    "country": "Bangladesh"
}
```

Struct

```go
type User struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}
```

The `"country"` field is ignored.

Go only fills fields that exist in the struct.

---

# Unmarshalling Nested Structs

```go
type Address struct {
	City string `json:"city"`
}

type User struct {
	Name    string  `json:"name"`
	Address Address `json:"address"`
}
```

JSON

```json
{
    "name": "Siam",
    "address": {
        "city": "Dhaka"
    }
}
```

Result

```go
fmt.Println(user.Address.City)
```

Output

```
Dhaka
```

---

# Unmarshalling into a Slice

JSON

```json
[
    {"name":"Siam"},
    {"name":"John"}
]
```

Go

```go
type User struct {
	Name string `json:"name"`
}

var users []User

err := json.Unmarshal(data, &users)
```

Result

```go
fmt.Println(users[0].Name)
```

Output

```
Siam
```

---

# Unmarshalling into a Map

Sometimes you don't know the JSON structure beforehand.

```json
{
    "name":"Siam",
    "age":24
}
```

```go
var result map[string]interface{}

err := json.Unmarshal(data, &result)
```

Now you can access values dynamically.

```go
fmt.Println(result["name"])
```

Output

```
Siam
```

---

# Common Data Types

JSON values map to Go types like this:

| JSON | Go |
|-------|----|
| string | string |
| number | float64 (when using `interface{}`) |
| boolean | bool |
| object | map or struct |
| array | slice |
| null | nil |

> **Note:** When unmarshalling into a `map[string]interface{}`, all JSON numbers become `float64` by default.

Example:

```go
var data map[string]interface{}

json.Unmarshal([]byte(`{"age":24}`), &data)

fmt.Printf("%T\n", data["age"])
```

Output

```
float64
```

---

# Common Errors

## Invalid JSON

```json
{
"name":"Siam"
```

Missing `}`

Produces

```
unexpected end of JSON input
```

---

## Type Mismatch

JSON

```json
{
    "age":"twenty"
}
```

Struct

```go
type User struct {
	Age int `json:"age"`
}
```

Produces

```
cannot unmarshal string into Go struct field User.age of type int
```

---

## Passing a Non-Pointer

Wrong

```go
json.Unmarshal(data, user)
```

Produces an error similar to:

```
json: Unmarshal(non-pointer main.User)
```

Always pass a pointer:

```go
json.Unmarshal(data, &user)
```

---

# Marshal vs Unmarshal

| Marshal | Unmarshal |
|----------|------------|
| Go → JSON | JSON → Go |
| `json.Marshal()` | `json.Unmarshal()` |
| Returns `[]byte` | Fills an existing variable |
| Used when sending data | Used when receiving data |

---

# Best Practices

- Always validate errors returned by `json.Unmarshal()`.
- Use JSON tags to ensure correct field mapping.
- Pass pointers to `Unmarshal()`.
- Prefer structs over `map[string]interface{}` when the JSON structure is known.
- Validate the resulting data after unmarshalling (for example, required fields or value ranges), since successful unmarshalling does not guarantee the data is logically valid.

---

# Related Notes

- [[JSON Marshalling]]
- [[Struct]]
- [[Struct Tags]]
- [[Pointers]]
- [[Slices]]
- [[Maps]]
- [[HTTP Request]]
- [[REST API]]
- [[DTO (Data Transfer Object)]]
- [[encoding/json Package]]

---

# Summary

- **Unmarshalling** converts JSON into Go objects.
- `json.Unmarshal()` takes JSON bytes and a pointer to the destination variable.
- Missing JSON fields become Go zero values.
- Extra JSON fields are ignored.
- JSON tags map JSON keys to struct fields.
- Use structs whenever the JSON format is known.
- Always check for errors and validate the resulting data.