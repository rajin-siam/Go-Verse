
Structs are comparable if **all of their fields are comparable**.

```go
type User struct {
	Name string
	Age  int
}

u1 := User{"Alice", 20}
u2 := User{"Alice", 20}

fmt.Println(u1 == u2)
```

Output

```
true
```

This will **not compile**:

```go
type User struct {
	Name string
	Tags []string
}
```

Slices are not comparable.


[[Struct]]