
## What is a Function?

A **function** is a reusable block of code that performs a specific task.

Instead of writing the same code multiple times, you write it once inside a function and call it whenever needed.

Think of a function as a **machine**:

```
Input
   │
   ▼
+-----------+
| Function  |
+-----------+
   │
   ▼
Output
```

Example:

```go
func Add(a int, b int) int {
	return a + b
}
```

Calling it:

```go
sum := Add(10, 20)

fmt.Println(sum)
```

Output

```
30
```

---

# Why Do We Need Functions?

Imagine writing this:

```go
fmt.Println(10 + 20)

fmt.Println(15 + 30)

fmt.Println(7 + 12)
```

Now suppose the addition logic becomes more complex.

Instead of changing it everywhere, we create a function.

```go
func Add(a, b int) int {
	return a + b
}
```

Now we simply write

```go
Add(10, 20)

Add(15, 30)

Add(7, 12)
```

One change in the function updates every caller.

---

# Real World Analogy

Imagine ordering coffee.

You don't go into the kitchen and prepare it yourself.

You ask the barista.

```
Customer

↓

Order Coffee

↓

Barista

↓

Coffee
```

The customer doesn't care **how** the coffee is made.

Similarly,

```
Caller

↓

Add(10,20)

↓

30
```

The caller doesn't need to know the implementation.

---

# Function Syntax

Basic syntax:

```go
func FunctionName(parameters) returnType {

	// function body

}
```

Example:

```go
func Greet() {

	fmt.Println("Hello")
}
```

Call it:

```go
Greet()
```

---

# Anatomy of a Function

```go
func Add(a int, b int) int {

	return a + b

}
```

Let's break it down.

### `func`

The keyword that tells Go you're declaring a function.

---

### `Add`

The function's name.

Choose descriptive names.

Good:

```go
CalculateTax()

SendEmail()

ValidateUser()
```

Bad:

```go
Do()

Test()

A()
```

---

### Parameters

```go
a int

b int
```

These are inputs to the function.

---

### Return Type

```go
int
```

Specifies what the function returns.

---

### Function Body

Everything inside the braces.

```go
return a + b
```

---

# Calling a Function

Declaring a function does nothing by itself.

```go
func Hello() {
	fmt.Println("Hello")
}
```

The code runs only when called.

```go
Hello()
```

---

# Parameters

Functions receive input through parameters.

```go
func Square(number int) int {

	return number * number
}
```

Call

```go
Square(5)
```

Output

```
25
```

---

# Multiple Parameters

```go
func Multiply(a int, b int) int {

	return a * b
}
```

Go lets you shorten this:

```go
func Multiply(a, b int) int {

	return a * b
}
```

Both are identical.

---

# Return Values

Functions can return results.

```go
func Add(a, b int) int {

	return a + b
}
```

Usage

```go
sum := Add(10,20)
```

---

# Functions Without Returns

Not every function returns a value.

```go
func PrintWelcome() {

	fmt.Println("Welcome")
}
```

The purpose is to perform an action.

---

# Multiple Return Values

Go supports returning multiple values.

```go
func Divide(a, b int) (int, int) {

	return a / b, a % b
}
```

Usage

```go
quotient, remainder := Divide(10,3)
```

Output

```
3

1
```

This feature is heavily used in Go, especially for returning a result and an error.

---

# Returning Errors

One of Go's design principles is explicit error handling.

```go
func ReadFile(name string) ([]byte, error) {

	// ...

}
```

Usage

```go
data, err := ReadFile("notes.txt")

if err != nil {

	return err
}
```

Instead of exceptions, Go functions usually return an `error`.

---

# Functions Are Reusable

Instead of:

```go
fmt.Println("Hello Siam")

fmt.Println("Hello John")

fmt.Println("Hello Alice")
```

Write

```go
func Greet(name string) {

	fmt.Println("Hello", name)
}
```

Now

```go
Greet("Siam")

Greet("John")

Greet("Alice")
```

---

# Functions Can Call Other Functions

```go
func Multiply(a,b int) int {

	return a*b
}

func Square(n int) int {

	return Multiply(n,n)
}
```

Output

```go
Square(5)
```

```
25
```

Functions can build on each other.

---

# Scope

Variables declared inside a function belong only to that function.

```go
func Demo() {

	x := 10

	fmt.Println(x)

}
```

Outside

```go
fmt.Println(x)
```

Compile error.

The variable exists only within the function's scope.

---

# Functions Are First-Class Values

In Go, functions are values.

They can be assigned to variables.

```go
func Add(a,b int) int {

	return a+b
}

f := Add

fmt.Println(f(2,3))
```

Output

```
5
```

This enables powerful patterns like callbacks and middleware.

> We'll explore this in detail in [[First-Class Functions]].

---

# Function Signatures

A function's **signature** consists of:

- Parameter types
- Return types

Example:

```go
func Add(a int,b int) int
```

Signature

```
(int,int) → int
```

Two functions cannot have the same name and signature in the same package.

Unlike languages like Java or C++, Go **does not support function overloading**.

---

# Function Overloading

This is **not allowed**:

```go
func Add(a int,b int) int

func Add(a float64,b float64) float64
```

Compile error.

Use different names or interfaces/generics instead.

---

# Compiler Internals

When you write

```go
Add(10,20)
```

the compiler roughly performs these steps:

```
Source Code

↓

Parse Function

↓

Type Check

↓

Generate Machine Code

↓

Program Calls Function
```

Depending on optimization, the compiler may even **inline** very small functions, replacing the call with the function body to avoid call overhead.

We'll study this in [[Compiler Internals of Functions]].

---

# Common Mistakes

## Forgetting to Call the Function

Wrong

```go
fmt.Println(Add)
```

Correct

```go
fmt.Println(Add(2,3))
```

---

## Ignoring Returned Errors

Bad

```go
data,_ := os.ReadFile("a.txt")
```

Good

```go
data,err := os.ReadFile("a.txt")

if err != nil {

	return err
}
```

---

## Writing Huge Functions

Avoid functions with hundreds of lines.

Prefer small, focused functions that do one thing well.

---

# Best Practices

✅ Give functions descriptive names.

---

✅ Keep functions focused on a single responsibility.

---

✅ Return errors instead of panicking for expected failures.

---

✅ Reuse functions instead of duplicating logic.

---

✅ Keep function bodies small and readable.

---

# Interview Questions

### What is a function?

A reusable block of code that performs a specific task.

---

### Why does Go support multiple return values?

To make returning both a result and an error simple and explicit.

---

### Does Go support function overloading?

No.

Function names must be unique within a package.

---

### Are functions values in Go?

Yes.

Functions are first-class values and can be assigned to variables, passed as arguments, and returned from other functions.

---

### Can a function return another function?

Yes.

Functions are first-class values.

We'll explore this in [[Higher-Order Functions]].

---

# Related Notes

- [[Function Parameters]]
- [[Return Values]]
- [[Named Return Values]]
- [[Variadic Functions]]
- [[Anonymous Functions]]
- [[Closures]]
- [[First-Class Functions]]
- [[Higher-Order Functions]]
- [[Methods]]
- [[Deferred Function Calls]]
- [[Compiler Internals of Functions]]

---

# Summary

- A function is a reusable block of code that performs a specific task.
- Functions improve code reuse, readability, and maintainability.
- They can accept parameters and return one or more values.
- Go commonly uses multiple return values for result and error handling.
- Functions are first-class values and can be assigned, passed, and returned.
- Go does not support function overloading.
- Small, focused functions are easier to test and maintain.