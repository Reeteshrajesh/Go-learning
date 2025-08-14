# Go Learning 📚

A simple learning repository for Go basics, control flow, functions, and pointers (with a C++ pointer comparison).

---

## 📌 Code Syntax in Go

```go
package main

import "fmt"

// Package-level variable
var Name = "Alok"

// Package-level map
var person = map[string]string{
    "name": "Reetesh",
    "role": "DevOps",
}

func main() {
    fmt.Println(person["role"]) // Access inside main
    fmt.Println(Name)           // Access package-level variable
    fmt.Println(person["name"]) // Access package-level map
}
````

---

## 🔹 If-Else Conditional Statements in Go

```go
if age > 18 {
    fmt.Println("Adult")
} else {
    fmt.Println("Minor")
}
```

---

## 🔹 For Loop in Go

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

---

## 🔹 Functions in Go

```go
func add(a int, b int) int {
    return a + b
}
```

---

## 🖥 Pointers in C++

```cpp
#include <iostream>
using namespace std;

int main() {
    int x = 10;
    int* ptr = &x;

    cout << "Value of x: " << x << endl;
    cout << "Address of x: " << ptr << endl;
    cout << "Value via pointer: " << *ptr << endl;

    *ptr = 20; // change value through pointer
    cout << "Updated value of x: " << x << endl;

    return 0;
}
```

**Output:**

```
Value of x: 10
Address of x: 0x7ffee3564a8c
Value via pointer: 10
Updated value of x: 20
```

---

## 🖥 Pointers in Go

```go
package main
import "fmt"

func main() {
    x := 10        // normal variable
    p := &x        // p is a pointer to x

    fmt.Println("Value of x:", x)
    fmt.Println("Address of x:", p)
    fmt.Println("Value via pointer:", *p)

    *p = 20        // change value through pointer
    fmt.Println("Updated value of x:", x)
}
```

**Output:**

```
Value of x: 10
Address of x: 0xc0000140a8
Value via pointer: 10
Updated value of x: 20
```

---

## 🔍 Summary

* Go pointers are safe (no pointer arithmetic, automatic memory management).
* C++ pointers give low-level memory control but require manual memory management.
* Go syntax is minimal and beginner-friendly.
* C++ pointers allow more flexibility but also more room for bugs.
                     
## Error Handling

```go
import "errors"

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("cannot divide by zero")
    }
    return a / b, nil
}
```



# Go Language: Control Flow, Functions, and More

---

## Operators in Go

Go supports several types of operators:

### 1. **Arithmetic Operators**

* `+` Addition
* `-` Subtraction
* `*` Multiplication
* `/` Division
* `%` Modulus

### 2. **Comparison Operators**

* `==` Equal to
* `!=` Not equal to
* `>` Greater than
* `<` Less than
* `>=` Greater than or equal to
* `<=` Less than or equal to

### 3. **Logical Operators**

* `&&` Logical AND
* `||` Logical OR
* `!` Logical NOT

### 4. **Bitwise Operators**

* `&` AND
* `|` OR
* `^` XOR
* `&^` AND NOT
* `<<` Left shift
* `>>` Right shift

### 5. **Assignment Operators**

* `=` Assign
* `+=`, `-=`, `*=`, `/=`, `%=` Compound assignments
* `&=`, `|=`, `^=`, `<<=`, `>>=` Bitwise assignments

---

## Control Flow in Go

### if / else

```go
x := 10
if x > 5 {
    fmt.Println("x is greater than 5")
} else {
    fmt.Println("x is 5 or less")
}
```

### for loop

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

### while-like loop

```go
x := 0
for x < 5 {
    fmt.Println(x)
    x++
}
```

### Infinite loop

```go
for {
    fmt.Println("looping forever")
}
```

### switch

```go
value := 2
switch value {
case 1:
    fmt.Println("One")
case 2:
    fmt.Println("Two")
default:
    fmt.Println("Other")
}
```

---

## Functions in Go

### Basic Function

```go
func greet(name string) string {
    return "Hello, " + name
}
```

### Calling a Function

```go
message := greet("Reetesh")
fmt.Println(message)
```

### Multiple Return Values

```go
func divide(a, b int) (int, int) {
    return a / b, a % b
}
```

### Named Return Values

```go
func add(a, b int) (result int) {
    result = a + b
    return
}
```

### Variadic Function

```go
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}
```

### Anonymous Function

```go
add := func(a, b int) int {
    return a + b
}
fmt.Println(add(2, 3))
```

### Higher-order Function (passing functions as values)

```go
func apply(op func(int, int) int, a int, b int) int {
    return op(a, b)
}

func multiply(x, y int) int {
    return x * y
}

fmt.Println(apply(multiply, 3, 4)) // Outputs 12
```

---

## Input/Output in Go

### Reading Input from User

```go
var name string
fmt.Print("Enter your name: ")
fmt.Scanln(&name)
fmt.Println("Hello", name)
```

### Reading File

```go
content, err := os.ReadFile("example.txt")
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(content))
```

### Writing to File

```go
err := os.WriteFile("output.txt", []byte("Hello, file!"), 0644)
if err != nil {
    log.Fatal(err)
}
```

### Buffered Reader (Line by line)

```go
file, err := os.Open("example.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}
if err := scanner.Err(); err != nil {
    log.Fatal(err)
}
```

---
