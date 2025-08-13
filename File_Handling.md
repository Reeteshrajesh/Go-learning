# 📄 Go File Handling — Read, Write, and Buffered Reading

This document covers different approaches to working with files in Go, with **code examples, use cases, and performance notes**.

---

## **1. Reading an Entire File into Memory (`os.ReadFile`)**

```go
package main

import (
    "fmt"
    "log"
    "os"
)

func main() {
    content, err := os.ReadFile("example.txt")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(content))
}
```

### **How it Works**

* **`os.ReadFile(path)`**:

  * Opens the file.
  * Reads **all bytes** into memory.
  * Closes the file automatically.
* Returns:

  * `[]byte` → file content.
  * `error` → non-`nil` if something goes wrong.
* `string(content)` converts the byte slice into a readable string.

### **When to Use**

✅ Best for **small to medium files** where reading everything at once is fine.
❌ Avoid for **very large files** due to high memory usage.

---

## **2. Writing to a File (`os.WriteFile`)**

```go
package main

import (
    "log"
    "os"
)

func main() {
    err := os.WriteFile("output.txt", []byte("Hello, file!"), 0644)
    if err != nil {
        log.Fatal(err)
    }
}
```

### **How it Works**

* **`os.WriteFile(path, data, perm)`**:

  * Creates or **overwrites** the file.
  * Writes the `[]byte` data in one operation.
  * `0644` → File permissions (owner can read/write, others read-only).

### **When to Use**

✅ For **quick writes** of small to moderate data.
❌ Not ideal for **appending** or writing very large streams — use `os.OpenFile` and buffered writers instead.

---

## **3. Buffered Reading Line by Line (`bufio.Scanner`)**

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "os"
)

func main() {
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
}
```

### **How it Works**

1. **Open the file** with `os.Open` (does not read immediately).
2. **Wrap in a Scanner** with `bufio.NewScanner(file)`.
3. **Loop through lines** with `scanner.Scan()` until EOF.
4. **Retrieve each line** with `scanner.Text()`.
5. **Check for errors** after reading.

### **When to Use**

✅ Ideal for **large files** or **streaming data** without loading the whole file into memory.
✅ Great for reading logs, CSVs, or any line-based data format.

---

## **4. Appending to a File**

If you want to **add** data without overwriting:

```go
f, err := os.OpenFile("output.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
if err != nil {
    log.Fatal(err)
}
defer f.Close()

if _, err := f.WriteString("New line\n"); err != nil {
    log.Fatal(err)
}
```

---

## **5. Comparison Table**

| Method          | Memory Usage | Reads Whole File?  | Suitable For                 |
| --------------- | ------------ | ------------------ | ---------------------------- |
| `os.ReadFile`   | High         | ✅ Yes              | Small files, quick reads     |
| `os.WriteFile`  | Depends      | ✅ Yes (writes all) | Small writes                 |
| `bufio.Scanner` | Low          | ❌ No               | Large files, streaming reads |
| `os.OpenFile`   | Low          | ❌ No               | Appending, controlled writes |

---

## **Performance Tips**

* Use **`os.ReadFile`** for small configs or short text files.
* Use **`bufio.Scanner`** or **`bufio.Reader`** for large datasets.
* For binary data, avoid `string()` conversion and work with `[]byte` directly.
* Always `defer file.Close()` to avoid resource leaks.

---
---

## **1. `os` Package**

Think of the `os` package as Go’s direct interface to your **operating system**.

It deals with:

* Files & directories
* Environment variables
* Processes & system signals
* Permissions

You use `os` when you want **low-level, direct access** to system resources.

### Commonly used `os` functions and methods:

| Function / Method                | What it does                                       |
| -------------------------------- | -------------------------------------------------- |
| `os.Open(name)`                  | Opens a file for reading. Returns `*os.File`.      |
| `os.Create(name)`                | Creates a file for writing (overwrites if exists). |
| `os.ReadFile(name)`              | Reads the entire file into memory.                 |
| `os.WriteFile(name, data, perm)` | Writes data to a file.                             |
| `os.Remove(name)`                | Deletes a file.                                    |
| `os.Stat(name)`                  | Gets file info (size, modification time, etc.).    |
| `os.Mkdir(name, perm)`           | Creates a directory.                               |
| `os.Getenv(key)`                 | Gets an environment variable.                      |
| `os.Exit(code)`                  | Immediately exits the program.                     |

---

## **2. `bufio` Package**

`bufio` stands for **Buffered I/O**.
It doesn’t replace `os` — instead, it **wraps an `io.Reader` or `io.Writer`** (like a file) to make reading/writing more efficient.

Why?
Without buffering, every `Read` or `Write` hits the disk or network directly. That’s slow.
`bufio` stores data in memory temporarily and processes it in chunks.

---

### Common `bufio` types & functions:

| bufio Type / Function           | What it does                                                               |
| ------------------------------- | -------------------------------------------------------------------------- |
| `bufio.NewReader(r)`            | Wraps an `io.Reader` (e.g., `os.File`) with buffering for efficient reads. |
| `bufio.NewWriter(w)`            | Wraps an `io.Writer` for efficient writes.                                 |
| `bufio.NewScanner(r)`           | Reads data **line-by-line** or token-by-token.                             |
| `(*Reader).ReadString('\n')`    | Reads until a newline character.                                           |
| `(*Reader).ReadBytes(delim)`    | Reads until a delimiter byte.                                              |
| `(*Writer).WriteString("data")` | Writes a string to buffer.                                                 |
| `(*Writer).Flush()`             | Forces any buffered data to be written out.                                |
| `(*Scanner).Scan()`             | Moves to the next token/line.                                              |
| `(*Scanner).Text()`             | Gets the current scanned text.                                             |

---

## **3. How they work together**

You almost always **open a file with `os`** first, then wrap it with `bufio` for efficiency.

Example: Reading line by line

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "os"
)

func main() {
    file, err := os.Open("example.txt") // OS-level file open
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    scanner := bufio.NewScanner(file) // Buffered reader
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }

    if err := scanner.Err(); err != nil {
        log.Fatal(err)
    }
}
```

Example: Writing efficiently

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "os"
)

func main() {
    file, err := os.Create("output.txt") // OS-level file create
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    writer := bufio.NewWriter(file) // Buffered writer
    for i := 1; i <= 5; i++ {
        fmt.Fprintf(writer, "Line %d\n", i)
    }
    writer.Flush() // IMPORTANT: write buffered data to file
}
```

---

✅ **Rule of Thumb**:

* Use `os` for **creating, opening, deleting, and getting info about files**.
* Use `bufio` when you **read/write a lot of data** (especially in loops) to improve performance.
