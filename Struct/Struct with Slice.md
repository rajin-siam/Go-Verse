

```go
package main

import "fmt"

type Student struct {
	Name   string
	Scores []int
}

func main() {
	s := Student{
		Name:   "John",
		Scores: []int{80, 90, 95},
	}

	fmt.Println(s)
}
```

Output

```
{John [80 90 95]}
```


[[Struct]]