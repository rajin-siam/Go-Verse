

Useful when a struct is only needed once.

```go
person := struct {
	Name string
	Age  int
}{
	Name: "Bob",
	Age:  30,
}

fmt.Println(person)
```

Output

```
{Bob 30}
```


[[Struct]]