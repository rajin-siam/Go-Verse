
## What is an Anonymous Function?

An **anonymous function** is a function **without a name**.

Unlike a normal function:

```go
func Add(a, b int) int {
	return a + b
}
```

an anonymous function has no identifier.

```go
func(a, b int) int {
	return a + b
}
```

Notice that there is no function name after the `func` keyword.

---

# Why Do Anonymous Functions Exist?

Sometimes you need a function only once.

Instead of creating a named function that will never be reused, you can create it inline.

Imagine writing:

```go
func PrintHello() {
	fmt.Println("Hello")
}
```

If this function is called only once, giving it a name may be unnecessary.

Instead:

```go
func() {
	fmt.Println("Hello")
}()
```

---

# Real World Analogy

Imagine hiring a full-time employee just to deliver one package.

That's unnecessary.

Instead, you hire a courier for one delivery.

Named function

```
Permanent Employee
```

Anonymous function

```
Temporary Worker
```

Use it once, then it's gone.

---

# Basic Syntax

Anonymous function:

```go
func() {

	fmt.Println("Hello")

}
```

Notice:

- No name
- Can have parameters
- Can return values
- Behaves like any other function

---

# Executing Immediately

Creating an anonymous function does **not** execute it.

```go
func() {
	fmt.Println("Hello")
}
```

Nothing happens.

To execute it immediately, add `()` after the function.

```go
func() {
	fmt.Println("Hello")
}()
```

Output

```
Hello
```

This is called an **Immediately Invoked Function Expression (IIFE)**.

---

# Anonymous Function with Parameters

```go
func(name string) {

	fmt.Println("Hello", name)

}("Siam")
```

Flow

```
"Siam"

↓

Parameter

↓

Print
```

Output

```
Hello Siam
```

---

# Returning Values

Anonymous functions can return values.

```go
result := func(a, b int) int {

	return a + b

}(10,20)
```

Output

```
30
```

The function is created, called immediately, and its result is assigned to `result`.

---

# Assigning to Variables

Functions are values.

You can store an anonymous function in a variable.

```go
add := func(a,b int) int {

	return a+b
}
```

Later

```go
fmt.Println(add(10,20))
```

Output

```
30
```

Think of `add` as a variable that happens to hold a function.

---

# Function Values

Just like this:

```go
x := 10
```

stores an integer,

```go
add := func(a,b int) int {

	return a+b
}
```

stores a function.

Conceptually:

```
Variable

↓

Function
```

Instead of

```
Variable

↓

Integer
```

---

# Passing Anonymous Functions

Functions can be passed as arguments.

```go
func Execute(f func()) {

	f()
}
```

Call

```go
Execute(func() {

	fmt.Println("Running")

})
```

Output

```
Running
```

This pattern is common for callbacks and middleware.

---

# Returning Anonymous Functions

Functions can also return functions.

```go
func GetPrinter() func() {

	return func() {

		fmt.Println("Hello")
	}
}
```

Usage

```go
printer := GetPrinter()

printer()
```

Output

```
Hello
```

This is the foundation of **higher-order functions**.

---

# Anonymous Functions and Goroutines

One of the most common uses is with goroutines.

```go
go func() {

	fmt.Println("Running")

}()
```

Instead of creating

```go
func Worker() {

}
```

and calling

```go
go Worker()
```

the function is written exactly where it's used.

This is very common in concurrent Go programs.

---

# Anonymous Functions Can Capture Variables

Example

```go
name := "Siam"

func() {

	fmt.Println(name)

}()
```

Output

```
Siam
```

The anonymous function can access variables from the surrounding scope.

This behavior is called a **closure**.

We'll study closures in the next note.

---

# Compiler Internals

Consider

```go
add := func(a,b int) int {

	return a+b
}
```

The compiler creates a function object.

Conceptually:

```
Machine Code

↓

Function Value

↓

Stored in Variable

↓

Called Later
```

If the anonymous function captures surrounding variables, the compiler generates additional structures to hold that captured state.

---

# Memory Model

An anonymous function itself is code.

If it **doesn't capture** any outer variables, it's similar to a normal function.

If it **does capture** variables:

```go
count := 0

f := func() {

	count++
}
```

the compiler creates a **closure object** that stores the captured variables.

Depending on how the closure is used, those captured variables may be allocated on the heap.

This is why closures are closely related to **escape analysis**.

---

# Common Use Cases

## Callbacks

```go
Execute(func() {

	fmt.Println("Done")

})
```

---

## Goroutines

```go
go func() {

	fmt.Println("Working")

}()
```

---

## Short One-Time Logic

```go
result := func() int {

	return 10+20

}()
```

---

## Sorting

```go
sort.Slice(users, func(i,j int) bool {

	return users[i].Age < users[j].Age
})
```

Instead of creating a separate comparison function, the logic is written inline.

---

# Common Mistakes

## Forgetting to Call the Function

Wrong

```go
func() {

	fmt.Println("Hello")

}
```

Nothing happens.

Correct

```go
func() {

	fmt.Println("Hello")

}()
```

---

## Confusing Anonymous Functions with Closures

Not every anonymous function is a closure.

This is **not** a closure:

```go
func() {

	fmt.Println("Hello")

}
```

This **is** a closure:

```go
name := "Siam"

func() {

	fmt.Println(name)

}()
```

Because it captures `name` from the surrounding scope.

---

## Making Large Anonymous Functions

Avoid writing hundreds of lines inside an anonymous function.

If the logic is reusable or complex, create a named function instead.

---

# Best Practices

✅ Use anonymous functions for short, localized behavior.

---

✅ Use them with goroutines and callbacks.

---

✅ Keep them small and focused.

---

✅ Use named functions when logic is reused or lengthy.

---

# Interview Questions

### What is an anonymous function?

A function declared without a name.

---

### Can anonymous functions have parameters?

Yes.

They support parameters, return values, and methods of invocation just like named functions.

---

### Can they be assigned to variables?

Yes.

Functions are first-class values in Go.

---

### What is an IIFE?

An **Immediately Invoked Function Expression**—an anonymous function that is defined and executed immediately.

Example:

```go
func() {
	fmt.Println("Hello")
}()
```

---

### Is every anonymous function a closure?

No.

Only anonymous (or named) functions that capture variables from an outer scope are closures.

---

# Related Notes

- [[Functions]]
- [[Closures]]
- [[First-Class Functions]]
- [[Higher-Order Functions]]
- [[Function Types]]
- [[Goroutines]]
- [[Escape Analysis]]
- [[Compiler Internals of Functions]]

---

# Summary

- Anonymous functions are functions without names.
- They can be created, assigned to variables, passed as arguments, and returned from functions.
- Adding `()` after an anonymous function executes it immediately (IIFE).
- Anonymous functions are commonly used for callbacks, goroutines, sorting, and short one-time logic.
- Functions become **closures** when they capture variables from their surrounding scope.
- Small anonymous functions improve readability, but large reusable logic should usually be moved to named functions.