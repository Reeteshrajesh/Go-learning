# 🔐 Full JWT Auth Project in Go (with Gin)

### 📁 Folder Structure

```
jwt-auth-go/
├── main.go
├── handlers/
│   └── auth.go
├── middleware/
│   └── auth.go
├── models/
│   └── user.go
├── utils/
│   └── token.go
└── go.mod
```

---

## 🧠 Dependencies

Install them with:

```bash
go mod init jwt-auth-go
go get github.com/gin-gonic/gin
go get github.com/golang-jwt/jwt/v5
go get golang.org/x/crypto/bcrypt
```

---

### 🧱 Step-by-step Code

#### `models/user.go`

```go
package models

type User struct {
	Username string
	Password string
}
```

---

#### `utils/token.go`

```go
package utils

import (
	"time"

	"github.com/golang-jwt/jwt/v5"
)

var secretKey = []byte("secret") // use env in real-world

func GenerateToken(username string) (string, string, error) {
	accessToken := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
		"username": username,
		"exp":      time.Now().Add(time.Minute * 15).Unix(),
	})
	accessString, err := accessToken.SignedString(secretKey)
	if err != nil {
		return "", "", err
	}

	refreshToken := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
		"username": username,
		"exp":      time.Now().Add(time.Hour * 24 * 7).Unix(),
	})
	refreshString, err := refreshToken.SignedString(secretKey)
	return accessString, refreshString, err
}

func ValidateToken(tokenString string) (string, error) {
	token, err := jwt.Parse(tokenString, func(t *jwt.Token) (interface{}, error) {
		return secretKey, nil
	})

	if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
		return claims["username"].(string), nil
	}

	return "", err
}
```

---

#### `handlers/auth.go`

```go
package handlers

import (
	"jwt-auth-go/models"
	"jwt-auth-go/utils"
	"net/http"

	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/bcrypt"
)

var users = map[string]string{} // in-memory user store

func Register(c *gin.Context) {
	var user models.User
	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "invalid input"})
		return
	}

	hashed, _ := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	users[user.Username] = string(hashed)
	c.JSON(http.StatusOK, gin.H{"message": "registered"})
}

func Login(c *gin.Context) {
	var user models.User
	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "invalid input"})
		return
	}

	stored, ok := users[user.Username]
	if !ok || bcrypt.CompareHashAndPassword([]byte(stored), []byte(user.Password)) != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "unauthorized"})
		return
	}

	access, refresh, err := utils.GenerateToken(user.Username)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "token error"})
		return
	}

	c.JSON(http.StatusOK, gin.H{
		"access_token":  access,
		"refresh_token": refresh,
	})
}

func Refresh(c *gin.Context) {
	var req struct {
		RefreshToken string `json:"refresh_token"`
	}
	if err := c.ShouldBindJSON(&req); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "invalid input"})
		return
	}

	username, err := utils.ValidateToken(req.RefreshToken)
	if err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid refresh token"})
		return
	}

	access, refresh, err := utils.GenerateToken(username)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "token generation failed"})
		return
	}

	c.JSON(http.StatusOK, gin.H{
		"access_token":  access,
		"refresh_token": refresh,
	})
}
```

---

#### `middleware/auth.go`

```go
package middleware

import (
	"jwt-auth-go/utils"
	"net/http"
	"strings"

	"github.com/gin-gonic/gin"
)

func JWTAuth() gin.HandlerFunc {
	return func(c *gin.Context) {
		auth := c.GetHeader("Authorization")
		if auth == "" || !strings.HasPrefix(auth, "Bearer ") {
			c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "unauthorized"})
			return
		}
		token := strings.TrimPrefix(auth, "Bearer ")

		username, err := utils.ValidateToken(token)
		if err != nil {
			c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "invalid token"})
			return
		}

		c.Set("username", username)
		c.Next()
	}
}
```

---

#### `main.go`

```go
package main

import (
	"jwt-auth-go/handlers"
	"jwt-auth-go/middleware"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	r.POST("/register", handlers.Register)
	r.POST("/login", handlers.Login)
	r.POST("/refresh", handlers.Refresh)

	auth := r.Group("/api")
	auth.Use(middleware.JWTAuth())
	auth.GET("/profile", func(c *gin.Context) {
		user, _ := c.Get("username")
		c.JSON(200, gin.H{"user": user})
	})

	r.Run(":8080")
}
```

---

### 🔃 JWT Flow Summary

1. `POST /register`: Save user with hashed password.
2. `POST /login`: Verify password, return access + refresh tokens.
3. `GET /api/profile`: Requires valid access token.
4. `POST /refresh`: Takes refresh token, returns new access + refresh tokens.

---

### ✅ Run the App

```bash
go run main.go
```

Test with Postman or curl:

```bash
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"username":"bob","password":"1234"}'
```
