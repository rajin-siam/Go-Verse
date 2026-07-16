
```go
package main

import "fmt"

type Product struct {
	Name  string
	Stock map[string]int
}

func main() {
	p := Product{
		Name: "Coffee",
		Stock: map[string]int{
			"Small": 20,
			"Large": 10,
		},
	}

	fmt.Println(p.Stock["Small"])
}
```

Output

```
20
```


[[Struct]]