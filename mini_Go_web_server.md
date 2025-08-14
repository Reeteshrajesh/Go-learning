# 🛠️ 1. Project Setup

Create a folder:

```bash
mkdir go-backend-basics && cd go-backend-basics
touch main.go
```

Paste the code below into `main.go`.

---

## ✅ Full Code: Covers All 5 Concepts

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
)

type Message struct {
    Name    string `json:"name"`
    Message string `json:"message"`
}

// GET handler
func getHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)

    response := map[string]string{"message": "GET request successful!"}
    json.NewEncoder(w).Encode(response)
}

// POST handler
func postHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")

    if r.Method != http.MethodPost {
        http.Error(w, "Only POST is allowed", http.StatusMethodNotAllowed)
        return
    }

    // Read body
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Cannot read body", http.StatusBadRequest)
        return
    }

    var msg Message
    err = json.Unmarshal(body, &msg)
    if err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }

    // Set status and return response
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(map[string]string{
        "received": fmt.Sprintf("Hello %s, you said: %s", msg.Name, msg.Message),
    })
}

func main() {
    http.HandleFunc("/get", getHandler)
    http.HandleFunc("/post", postHandler)

    fmt.Println("Server running on http://localhost:8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## 🚀 How to Run

```bash
go run main.go
```

---

## 🧪 Test with curl or Postman

### ✅ GET Request:

```bash
curl http://localhost:8080/get
```

Response:

```json
{"message":"GET request successful!"}
```

---

### ✅ POST Request:

```bash
curl -X POST http://localhost:8080/post \
  -H "Content-Type: application/json" \
  -d '{"name":"Reetesh","message":"Learning Go!"}'
```

Response:

```json
{"received":"Hello Reetesh, you said: Learning Go!"}
```

---

## ✅ Summary of Concepts Practiced:

| Concept              | Implemented In                          |
| -------------------- | --------------------------------------- |
| Create GET endpoint  | `/get` route with `getHandler`          |
| Create POST endpoint | `/post` route with `postHandler`        |
| Return JSON response | Using `json.NewEncoder(w).Encode(...)`  |
| Read request body    | `io.ReadAll(r.Body)` + `json.Unmarshal` |
| Set headers          | `w.Header().Set("Content-Type", ...)`   |
| Use status codes     | `w.WriteHeader(http.StatusCreated)`     |

---



# **📘 HTTP Status Codes in Go — Complete Guide**

## **1. What is `net/http`?**

`net/http` is the **standard Go package** for building HTTP servers and clients.

When you build a web server in Go, you typically:

* Create **handlers** for different routes
* Write responses
* Use HTTP status codes to tell clients what happened

Import it like:

```go
import "net/http"
```

---

## **2. HTTP Status Codes in Go**

Instead of remembering numbers like `404` or `500`, Go defines **named constants** in `net/http`.

Example:

```go
http.StatusOK               // 200
http.StatusNotFound         // 404
http.StatusInternalServerError // 500
```

This improves readability:

```go
w.WriteHeader(http.StatusNotFound) // Clear meaning: "Not Found"
```

vs.

```go
w.WriteHeader(404) // Less clear
```

---

## **3. Full Table — Common HTTP Status Codes in Go**

| Constant                          | Number | Meaning                              |
| --------------------------------- | ------ | ------------------------------------ |
| **✅ Success (2xx)**               |        |                                      |
| `http.StatusOK`                   | 200    | Request successful                   |
| `http.StatusCreated`              | 201    | Resource created                     |
| `http.StatusAccepted`             | 202    | Request accepted for processing      |
| `http.StatusNoContent`            | 204    | No content to return                 |
| **⚠️ Client Errors (4xx)**        |        |                                      |
| `http.StatusBadRequest`           | 400    | Client sent invalid request          |
| `http.StatusUnauthorized`         | 401    | Authentication required              |
| `http.StatusForbidden`            | 403    | No permission to access              |
| `http.StatusNotFound`             | 404    | Resource not found                   |
| `http.StatusMethodNotAllowed`     | 405    | HTTP method not supported            |
| `http.StatusConflict`             | 409    | Conflict with current resource state |
| `http.StatusGone`                 | 410    | Resource permanently removed         |
| `http.StatusUnsupportedMediaType` | 415    | Unsupported request content type     |
| **💥 Server Errors (5xx)**        |        |                                      |
| `http.StatusInternalServerError`  | 500    | Server error                         |
| `http.StatusNotImplemented`       | 501    | Method not implemented               |
| `http.StatusBadGateway`           | 502    | Bad gateway                          |
| `http.StatusServiceUnavailable`   | 503    | Service temporarily unavailable      |
| `http.StatusGatewayTimeout`       | 504    | Gateway timeout                      |

---

## **4. Using Status Codes in Go**

### **4.1. Sending a status code**

```go
w.WriteHeader(http.StatusCreated)
w.Write([]byte("New resource created!"))
```

---

### **4.2. Sending an error with status code**

`http.Error()` is a shortcut:

```go
http.Error(w, "Not Found", http.StatusNotFound)
```

This:

* Sets `Content-Type: text/plain`
* Sets status code
* Writes the message

---

### **4.3. Example — Handling Method Not Allowed**

```go
func handler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
        return
    }
    w.Write([]byte("You made a GET request"))
}
```

---

## **5. Related Constants — HTTP Methods**

These are also defined in `net/http`:

```go
http.MethodGet     // "GET"
http.MethodPost    // "POST"
http.MethodPut     // "PUT"
http.MethodDelete  // "DELETE"
http.MethodPatch   // "PATCH"
http.MethodOptions // "OPTIONS"
```

Example check:

```go
if r.Method != http.MethodPost {
    http.Error(w, "Only POST allowed", http.StatusMethodNotAllowed)
}
```

---

## **6. Logging Status Codes with `log`**

Use `log` to keep track of requests:

```go
import "log"

func handler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        log.Printf("Invalid method: %s for %s", r.Method, r.URL.Path)
        http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
        return
    }
    log.Printf("Successful GET request on %s", r.URL.Path)
    w.Write([]byte("OK"))
}
```

---

## **7. Best Practices**

* **Always** send the correct status code — don’t just send `200` for errors.
* Use Go constants (`http.StatusNotFound`) instead of raw numbers.
* Log unexpected methods, paths, or errors for debugging.
* Keep status code logic close to your business logic for readability.
* For APIs, send JSON errors instead of plain text:

```go
w.Header().Set("Content-Type", "application/json")
w.WriteHeader(http.StatusBadRequest)
w.Write([]byte(`{"error": "Invalid input"}`))
```

---

## **8. Quick Reference Cheat Sheet**

```go
// 2xx Success
http.StatusOK
http.StatusCreated
http.StatusNoContent

// 4xx Client Errors
http.StatusBadRequest
http.StatusUnauthorized
http.StatusForbidden
http.StatusNotFound
http.StatusMethodNotAllowed

// 5xx Server Errors
http.StatusInternalServerError
http.StatusBadGateway
http.StatusServiceUnavailable
```

---
---

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
)

// Message represents a single message
type Message struct {
    Name    string `json:"name"`
    Message string `json:"message"`
}

// Global slice to store messages
var (
    messages []Message
    mu       sync.Mutex
)

// GET /messages — Return all messages
func getMessages(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)

    mu.Lock()
    defer mu.Unlock()
    json.NewEncoder(w).Encode(messages)
}

// POST /messages — Accept a new message
func postMessage(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")

    if r.Method != http.MethodPost {
        http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
        return
    }

    var msg Message
    if err := json.NewDecoder(r.Body).Decode(&msg); err != nil {
        http.Error(w, "Bad Request", http.StatusBadRequest)
        return
    }

    if msg.Name == "" || msg.Message == "" {
        http.Error(w, "Both 'name' and 'message' fields are required", http.StatusBadRequest)
        return
    }

    mu.Lock()
    messages = append(messages, msg)
    mu.Unlock()

    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(map[string]string{"status": "Message received!"})
}

func main() {
    http.HandleFunc("/messages", func(w http.ResponseWriter, r *http.Request) {
        if r.Method == http.MethodGet {
            getMessages(w, r)
        } else if r.Method == http.MethodPost {
            postMessage(w, r)
        } else {
            http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
        }
    })

    fmt.Println("Server started at http://localhost:8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```
