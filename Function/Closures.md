# Closures

## What is a Closure?

A **closure** is a function that **captures and remembers variables from the surrounding scope**.

Unlike a normal function, a closure doesn't just receive parameters—it can also access variables that were defined outside of it.

Example:

```go
func main() {
	name := "Siam"

	greet := func() {
		fmt.Println("Hello", name)
	}

	greet()
}
```

Output

```
Hello Siam
```

The anonymous function uses `name` even though `name` is not one of its parameters.

---

# A Better Definition

A closure is:

> **A function together with the variables it has captured from its surrounding environment.**

Think of a closure as two things bundled together:

```
+-------------------+
| Function Code     |
+-------------------+
| Captured Variables|
+-------------------+
```

The function carries its environment wherever it goes.

---

# Real World Analogy

Imagine writing a reminder on a sticky note.

```
Reminder

↓

Buy Milk
```

You stick it inside your notebook.

Wherever the notebook goes, the reminder goes too.

Similarly,

```
Function

+

Captured Variables

↓

Closure
```

The function carries its remembered variables.

---

# How Does a Closure Work?

Example:

```go
func main() {
	count := 0

	increment := func() {
		count++
		fmt.Println(count)
	}

	increment()
	increment()
	increment()
}
```

Output

```
1
2
3
```

Why?

The closure remembers the same `count` variable.

It does **not** create a new one each time.

---

# Visualizing the Memory

Initially

```
count

↓

0
```

After first call

```
count

↓

1
```

Second call

```
count

↓

2
```

Third call

```
count

↓

3
```

The closure keeps using the same variable.

---

# Returning a Closure

Closures become more interesting when returned.

```go
func Counter() func() int {

	count := 0

	return func() int {
		count++
		return count
	}
}
```

Usage

```go
next := Counter()

fmt.Println(next())
fmt.Println(next())
fmt.Println(next())
```

Output

```
1
2
3
```

---

# Why Doesn't `count` Disappear?

Normally, local variables disappear when a function returns.

```
Counter()

↓

Returns

↓

Local variables destroyed
```

But here,

the returned function still needs `count`.

The compiler notices this and keeps `count` alive.

This is one of the key features of closures.

---

# Independent Closures

Each call creates its own environment.

```go
a := Counter()

b := Counter()
```

Memory

```
Closure A

count

↓

0
```

```
Closure B

count

↓

0
```

Calling

```go
a()
a()
```

Output

```
1
2
```

Calling

```go
b()
```

Output

```
1
```

Each closure has its own captured state.

---

# Capturing Multiple Variables

```go
func CreateUser(name string) func() {

	age := 24

	return func() {
		fmt.Println(name, age)
	}
}
```

The closure captures both:

```
name

↓

"Siam"

age

↓

24
```

---

# Closures Are Not Limited to Anonymous Functions

A closure is **about capturing variables**, not about being anonymous.

Anonymous functions are the most common way to create closures, but the important feature is variable capture.

---

# Closures and Goroutines

A common use case:

```go
name := "Siam"

go func() {
	fmt.Println(name)
}()
```

The goroutine captures `name`.

This is a closure.

---

# The Loop Variable Trap

One of the most famous Go mistakes.

```go
for i := 0; i < 3; i++ {

	go func() {
		fmt.Println(i)
	}()
}
```

Older versions of Go often printed:

```
3
3
3
```

because every closure captured the **same** loop variable.

### Go 1.22+

Starting with Go 1.22, when the loop variable is declared by the `range` clause or in the `for` statement, each iteration gets its own copy of the loop variable. This fixes many common closure bugs.

Even so, it's still useful to understand the old behavior because you'll encounter it in older code and discussions.

An explicit parameter is also a clear pattern:

```go
for i := 0; i < 3; i++ {

	go func(n int) {
		fmt.Println(n)
	}(i)
}
```

Output

```
0
1
2
```

---

# Closures vs Parameters

Without closure

```go
func Print(name string) {

	fmt.Println(name)
}
```

Uses parameters.

With closure

```go
name := "Siam"

func() {

	fmt.Println(name)

}()
```

Uses captured variables.

---

# Compiler Internals

Consider

```go
func Counter() func() int {

	count := 0

	return func() int {

		count++

		return count
	}
}
```

The compiler roughly transforms this into something like:

```
Closure Object

↓

Function Pointer

+

Captured Variables
```

Conceptually:

```
+----------------------+
| Function Code        |
| count = 0            |
+----------------------+
```

Every time the closure runs, it accesses the stored `count`.

The exact implementation is compiler-specific, but this mental model is useful.

---

# Escape Analysis

Normally

```
Local Variables

↓

Stack
```

But if a closure outlives its surrounding function,

```
Local Variable

↓

Heap
```

because it must remain alive.

Example

```go
func Counter() func() int {

	count := 0

	return func() int {

		count++

		return count
	}
}
```

The compiler detects that `count` escapes.

We'll study this in [[Escape Analysis]].

---

# Common Uses

## Counters

```go
counter := Counter()
```

---

## HTTP Middleware

```go
func Logger(next http.Handler) http.Handler {

	return http.HandlerFunc(func(w http.ResponseWriter,r *http.Request){

		next.ServeHTTP(w,r)

	})
}
```

The closure captures `next`.

---

## Callbacks

```go
button.OnClick(func(){

	fmt.Println("Clicked")

})
```

---

## Goroutines

```go
go func(){

	fmt.Println("Running")

}()
```

---

## Dependency Injection

```go
func Service(db *DB) func(){

	return func(){

		db.Save()

	}
}
```

The closure captures `db`.

---

# Common Mistakes

## Confusing Anonymous Functions with Closures

This is **not** a closure:

```go
func(){

	fmt.Println("Hello")

}
```

No variables are captured.

This **is** a closure:

```go
name := "Siam"

func(){

	fmt.Println(name)

}
```

---

## Capturing Loop Variables Incorrectly

Older Go versions had many bugs caused by sharing the same loop variable.

Understand the behavior, even though Go 1.22 fixed the common case.

---

## Capturing Large Objects

Closures may keep captured values alive longer than expected.

Be mindful when capturing large structs or other memory-intensive objects.

---

# Best Practices

✅ Keep closures small and focused.

---

✅ Prefer passing values as parameters when capture isn't necessary.

---

✅ Understand variable lifetime when returning closures.

---

✅ Be careful with captured mutable state in concurrent programs.

---

# Interview Questions

### What is a closure?

A function together with the variables it captures from its surrounding environment.

---

### Why doesn't a captured variable disappear?

Because the compiler ensures it remains alive for as long as the closure needs it, often by moving it to the heap.

---

### Is every anonymous function a closure?

No.

Only functions that capture variables from an outer scope are closures.

---

### Can multiple closures share the same variable?

Yes.

If they capture the same variable, changes made by one closure are visible to the others.

---

### What role does escape analysis play?

It determines whether captured variables must be allocated on the heap so they outlive their original stack frame.

---

# Related Notes

- [[Anonymous Functions]]
- [[Functions]]
- [[First-Class Functions]]
- [[Higher-Order Functions]]
- [[Escape Analysis]]
- [[Pointers]]
- [[Goroutines]]
- [[Compiler Internals of Functions]]
- [[Memory Model]]

---

# Summary

- A closure is a function that captures variables from its surrounding scope.
- A closure remembers those variables even after the surrounding function returns.
- Each closure has access to its captured environment.
- Closures are widely used for callbacks, middleware, goroutines, dependency injection, and stateful functions.
- Captured variables may escape to the heap if they need to outlive their original function.
- Understanding closures is essential for writing correct concurrent Go programs.