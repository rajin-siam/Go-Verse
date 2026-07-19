
## What is JSON Marshalling?

**JSON Marshalling** is the process of converting a Go object (struct, map, slice, etc.) into a JSON string.

Think of it as **translating Go data into a language that other systems can understand**.

Go uses the `encoding/json` package to perform this conversion.

---

## Why Do We Need It?

Go structs exist only inside a Go program. They cannot be sent directly over:

- HTTP APIs
    
- Network connections
    
- Files
    
- Databases that store JSON
    

JSON is a universal data format that almost every programming language understands.

Instead of sending a Go struct, we convert it into JSON.

```
Go Struct
     │
     ▼
JSON Marshalling
     │
     ▼
JSON String
```

---

## Basic Example

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
	user := User{
		Name: "Siam",
		Age:  24,
	}

	data, err := json.Marshal(user)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(data))
}
```

Output

```json
{"Name":"Siam","Age":24}
```

Notice that `Marshal()` returns **bytes**, not a string.

---

## Why Does Marshal Return `[]byte`?

Most JSON is:

- sent over the network
    
- written to files
    
- stored in buffers
    

All of these work with bytes.

```
Struct
   │
   ▼
Marshal()
   │
   ▼
[]byte
   │
   ├── Write to file
   ├── HTTP Response
   ├── Save to database
   └── Convert to string
```

To print JSON:

```go
fmt.Println(string(data))
```

---

# JSON Tags

By default, Go uses the field names.

```go
type User struct {
	Name string
	Age  int
}
```

Produces

```json
{
    "Name": "Siam",
    "Age": 24
}
```

Usually APIs prefer lowercase.

```go
type User struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}
```

Output

```json
{
    "name": "Siam",
    "age": 24
}
```

---

# How Tags Work

The tag

```go
`json:"name"`
```

tells the JSON package:

> "Use **name** as the JSON key instead of the Go field name."

Go field

```
Name
```

becomes

```json
"name"
```

---

# Exported vs Unexported Fields

Only **exported fields** are marshalled.

```go
type User struct {
	Name     string
	password string
}
```

Output

```json
{
    "Name": "Siam"
}
```

The `password` field is ignored because it starts with a lowercase letter and is **unexported**.

---

# Ignoring Fields

Use `-` if a field should never appear in JSON.

```go
type User struct {
	Name     string `json:"name"`
	Password string `json:"-"`
}
```

Output

```json
{
    "name": "Siam"
}
```

---

# Omitting Empty Fields

Sometimes you don't want zero values in the JSON.

```go
type User struct {
	Name string `json:"name"`
	Age  int    `json:"age,omitempty"`
}
```

```go
user := User{
	Name: "Siam",
}
```

Output

```json
{
    "name": "Siam"
}
```

Since `Age` is `0` (its zero value), it is omitted.

### Zero values

|Type|Zero Value|
|---|---|
|int|0|
|string|""|
|bool|false|
|slice|nil|
|map|nil|
|pointer|nil|

---

# Pretty Printing JSON

`Marshal()` creates compact JSON.

```json
{"name":"Siam","age":24}
```

To make it more readable:

```go
data, _ := json.MarshalIndent(user, "", "    ")

fmt.Println(string(data))
```

Output

```json
{
    "name": "Siam",
    "age": 24
}
```

---

# Marshalling Nested Structs

```go
type Address struct {
	City string `json:"city"`
}

type User struct {
	Name    string  `json:"name"`
	Address Address `json:"address"`
}
```

Output

```json
{
    "name": "Siam",
    "address": {
        "city": "Dhaka"
    }
}
```

The JSON package automatically marshals nested structs.

---

# Common Types That Can Be Marshalled

```go
json.Marshal(struct)
json.Marshal(map)
json.Marshal(slice)
json.Marshal(array)
json.Marshal(string)
json.Marshal(int)
json.Marshal(float64)
json.Marshal(bool)
```

Example:

```go
scores := map[string]int{
	"Math": 95,
	"Go":   100,
}

data, _ := json.Marshal(scores)
```

Output

```json
{
    "Go": 100,
    "Math": 95
}
```

---

# Common Errors

## Unsupported Types

Functions cannot be converted to JSON.

```go
type User struct {
	Name string
	Fn   func()
}
```

Produces:

```
json: unsupported type: func()
```

---

## Circular References

```go
type Node struct {
	Next *Node
}
```

If a node points back to itself, marshalling fails because JSON cannot represent infinite loops.

---

## Ignoring Errors

Bad practice:

```go
data, _ := json.Marshal(user)
```

Good practice:

```go
data, err := json.Marshal(user)
if err != nil {
	return err
}
```

---

# Best Practices

- Always use JSON tags for API structs.
    
- Keep sensitive fields (passwords, secrets, tokens) out of JSON using `json:"-"`.
    
- Check errors returned by `json.Marshal`.
    
- Use `MarshalIndent` only for debugging or human-readable output.
    
- Prefer API-specific DTOs over exposing internal domain entities directly.
    

---

# Related Notes

- [[JSON Unmarshalling]]
    
- [[Struct]]
    
- [[Struct Tags]]
    
- [[Slices]]
    
- [[Maps]]
    
- [[Pointers]]
    
- [[HTTP Request]]
    
- [[HTTP Response]]
    
- [[REST API]]
    
- [[DTO (Data Transfer Object)]]
    
- [[encoding/json Package]]
    

---

# Summary

- **Marshalling** converts Go data into JSON.
    
- `json.Marshal()` returns `[]byte`.
    
- Struct tags control JSON field names.
    
- Only exported fields are marshalled.
    
- Use `json:"-"` to hide fields.
    
- Use `omitempty` to omit zero-value fields.
    
- `MarshalIndent()` creates formatted JSON.
    
- Always handle errors from `json.Marshal()`.