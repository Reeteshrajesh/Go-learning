# **Go Map Concepts Cheat Sheet**

### **1. What is a Map in Go?**

* A **map** in Go is an **unordered** collection of key-value pairs.
* Keys are **unique**, and values can be duplicated.
* Under the hood, maps in Go are implemented as **hash tables**.

Example:

```go
var myMap map[string]int // Declaration (nil map)
myMap = make(map[string]int) // Initialization
myMap["apple"] = 5
myMap["banana"] = 3
fmt.Println(myMap) // map[apple:5 banana:3]
```

---

### **2. Declaring and Initializing Maps**

#### Method 1 – `make`

```go
m := make(map[string]int)
```

#### Method 2 – Map literal

```go
m := map[string]int{
    "a": 1,
    "b": 2,
}
```

#### Method 3 – `var` keyword (nil map)

```go
var m map[string]int // nil, can't set values yet
m = make(map[string]int)
```

---

### **3. Adding and Updating Values**

```go
m := make(map[string]int)
m["one"] = 1   // Add
m["one"] = 11  // Update
```

---

### **4. Accessing Values**

```go
val := m["one"]
fmt.Println(val) // If key not present → returns zero value (0 for int, "" for string)
```

---

### **5. Checking if a Key Exists**

Go returns **two values** when reading from a map: `value, exists`

```go
val, exists := m["two"]
if exists {
    fmt.Println("Key exists:", val)
} else {
    fmt.Println("Key not found")
}
```

---

### **6. Deleting a Key**

```go
delete(m, "one")
```

---

### **7. Iterating Over Maps**

```go
for key, value := range m {
    fmt.Println(key, value)
}
```

> **Note:** Map iteration order is **random** and changes in each run.

---

### **8. Length of a Map**

```go
fmt.Println(len(m))
```

---

### **9. Nil Maps**

```go
var m map[string]int // nil map
fmt.Println(m == nil) // true
// m["a"] = 1 // ❌ panic: assignment to entry in nil map
```

---

### **10. Map as Function Arguments**

Maps are **reference types** – modifications inside functions affect the original.

```go
func update(m map[string]int) {
    m["x"] = 99
}
```

---

### **11. Maps with Slices / Slices with Maps**

#### Map of slices:

```go
m := make(map[string][]int)
m["nums"] = []int{1, 2, 3}
```

#### Slice of maps:

```go
s := make([]map[string]int, 2)
s[0] = make(map[string]int)
s[0]["a"] = 1
```

---

### **12. Map Keys and Allowed Types**

* Keys must be **comparable** (bool, int, string, array, struct without slices/maps/functions).
* ❌ Can't use slices, maps, or functions as keys.

Example of struct key:

```go
type Point struct {
    X, Y int
}
m := make(map[Point]string)
m[Point{1, 2}] = "origin"
```

---

### **13. Initial Capacity**

```go
m := make(map[string]int, 100) // Preallocate space for ~100 elements
```

---

### **14. Maps are Not Thread Safe**

* If multiple goroutines write to a map, you **must** use sync.Mutex or sync.RWMutex.
* Or use `sync.Map`.

---

### **15. Example – Count Frequencies**

```go
func countFrequencies(nums []int) map[int]int {
    freqMap := make(map[int]int)
    for _, num := range nums {
        freqMap[num]++
    }
    return freqMap
}

func main() {
    nums := []int{1, 2, 2, 3, 1, 4}
    fmt.Println(countFrequencies(nums)) // map[1:2 2:2 3:1 4:1]
}
```

---

### **16. Common Pitfalls**

* **Zero value trap** – Reading non-existing key returns zero value, not an error.
* **Random order** – Don't depend on iteration order.
* **Concurrent writes** – Need locking.
* **Nil maps** – Must initialize before adding keys.

---

### **17. Useful Tricks**

#### Swap keys & values:

```go
func swap(m map[string]int) map[int]string {
    out := make(map[int]string)
    for k, v := range m {
        out[v] = k
    }
    return out
}
```

#### Merge maps:

```go
func merge(m1, m2 map[string]int) map[string]int {
    for k, v := range m2 {
        m1[k] = v
    }
    return m1
}
```

#### Remove duplicates from slice:

```go
func removeDuplicates(nums []int) []int {
    seen := make(map[int]bool)
    result := []int{}
    for _, n := range nums {
        if !seen[n] {
            seen[n] = true
            result = append(result, n)
        }
    }
    return result
}
```
---

## **1. Frequency Counting / Occurrence Tracking**

When you want to know how many times something occurs.

```go
words := []string{"go", "python", "go", "java", "go", "python"}
freq := make(map[string]int)

for _, word := range words {
    freq[word]++
}

fmt.Println(freq) // map[go:3 java:1 python:2]
```

**Why?**
Efficient O(1) lookups make it perfect for counting items.

---

## **2. Caching / Memoization**

Store results of expensive computations to reuse later.

```go
cache := make(map[int]int)

var fib func(n int) int
fib = func(n int) int {
    if n <= 1 {
        return n
    }
    if val, exists := cache[n]; exists {
        return val
    }
    cache[n] = fib(n-1) + fib(n-2)
    return cache[n]
}

fmt.Println(fib(10)) // 55
```

**Why?**
Maps make caching fast without using external DBs.

---

## **3. Lookup Tables (Configs, Mappings, Routing)**

Store key-value configurations for quick reference.

```go
httpStatus := map[int]string{
    200: "OK",
    404: "Not Found",
    500: "Internal Server Error",
}

fmt.Println(httpStatus[404]) // Not Found
```

**Why?**
Fast retrieval without loops.

---

## **4. Removing Duplicates**

Check if an item has been seen before.

```go
nums := []int{1, 2, 2, 3, 4, 1}
seen := make(map[int]bool)
unique := []int{}

for _, n := range nums {
    if !seen[n] {
        seen[n] = true
        unique = append(unique, n)
    }
}

fmt.Println(unique) // [1 2 3 4]
```

**Why?**
Map lookups are faster than scanning arrays repeatedly.

---

## **5. Grouping Data**

Group items by category or property.

```go
students := []struct {
    Name string
    Grade string
}{
    {"Alice", "A"}, {"Bob", "B"}, {"Charlie", "A"},
}

groups := make(map[string][]string)

for _, s := range students {
    groups[s.Grade] = append(groups[s.Grade], s.Name)
}

fmt.Println(groups) // map[A:[Alice Charlie] B:[Bob]]
```

**Why?**
Maps allow building complex groupings easily.

---

## **6. Relationship Mapping**

Mapping between IDs, relationships, or connections.

```go
friends := map[string][]string{
    "Alice": {"Bob", "Charlie"},
    "Bob":   {"Alice"},
}

fmt.Println(friends["Alice"]) // [Bob Charlie]
```

**Why?**
You can store complex structures inside maps.

---

## **7. Indexing Data by Unique Keys**

For quick retrieval without looping.

```go
type User struct {
    ID   int
    Name string
}

users := map[int]User{
    101: {ID: 101, Name: "Alice"},
    102: {ID: 102, Name: "Bob"},
}

fmt.Println(users[101]) // {101 Alice}
```

**Why?**
Avoids scanning arrays for matches.

---

## **8. Session / State Management**

Track user sessions or in-memory states.

```go
sessions := make(map[string]string)
sessions["token123"] = "Alice"
sessions["token456"] = "Bob"

fmt.Println("User:", sessions["token123"]) // Alice
```

**Why?**
Great for real-time applications where state must be tracked quickly.

---

## **9. In-Memory Database Simulation**

Store small datasets without external DB.

```go
inventory := map[string]int{
    "apple":  50,
    "banana": 30,
}

inventory["apple"] -= 5 // Sell 5 apples
fmt.Println(inventory) // map[apple:45 banana:30]
```

---

## **10. Counting Combinations / Pairs**

When you need to count specific patterns.

```go
pairs := []string{"a-b", "b-c", "a-b", "a-c"}
pairCount := make(map[string]int)

for _, p := range pairs {
    pairCount[p]++
}

fmt.Println(pairCount) // map[a-b:2 b-c:1 a-c:1]
```

---

💡 **Summary Table — Map Use Cases for Developers**

| Use Case                 | Key Benefit             |
| ------------------------ | ----------------------- |
| Frequency Counting       | O(1) lookup             |
| Caching/Memoization      | Avoid recomputation     |
| Lookup Tables            | Instant mapping         |
| Removing Duplicates      | No extra loops          |
| Grouping Data            | Natural bucket creation |
| Relationship Mapping     | Store complex data      |
| Indexing by Unique Key   | Fast direct access      |
| Session/State Management | Real-time tracking      |
| In-memory DB Simulation  | No extra dependencies   |
| Counting Combinations    | Easy aggregation        |


-------


# ✅ **DSA Questions on Maps (Go)**

1. **Count Frequencies**
   Given a slice of integers, count the frequency of each element.
   *Input:* `[1, 2, 2, 3, 1, 4]` → *Output:* `map[1:2 2:2 3:1 4:1]`

2. **Find First Non-Repeating Character**
   Given a string, find the first character that does not repeat.
   *Input:* `"gogopher"` → *Output:* `'p'`

3. **Check for Duplicates**
   Given a slice of integers, return `true` if any value appears at least twice.
   *Input:* `[1, 2, 3, 4, 1]` → *Output:* `true`

4. **Is Anagram**
   Given two strings, check if they are anagrams using maps.
   *Input:* `"listen", "silent"` → *Output:* `true`

5. **Group Words by Anagrams**
   Given a list of words, group them into anagram sets.
   *Input:* `["bat", "tab", "cat", "act"]`
   *Output:* `[[bat tab] [cat act]]`

6. **Top K Frequent Elements**
   Find the top `k` frequent elements in a slice.
   *Input:* `[1,1,1,2,2,3], k=2` → *Output:* `[1,2]`

7. **Two Sum Problem**
   Given an array and a target, return indices of two numbers that add up to the target using a map.
   *Input:* `[2,7,11,15], target=9` → *Output:* `[0,1]`

8. **Find Common Elements Between Two Arrays**
   Use maps to find common elements.
   *Input:* `[1,2,3]`, `[2,3,4]` → *Output:* `[2,3]`

9. **Find Missing Number**
   An array contains numbers from `1` to `n` with one missing. Find the missing number using a map.
   *Input:* `[1, 2, 4, 5]` → *Output:* `3`

10. **Find Element with Single Occurrence**
    Every element appears twice except one. Find that one using map.
    *Input:* `[4,1,2,1,2]` → *Output:* `4`

-----------

```go

package main

import (
	"fmt"
	"sort"
)

// 1️⃣ Count Frequencies
func countFrequencies(nums []int) map[int]int {
	freqMap := make(map[int]int)
	for _, num := range nums {
		freqMap[num]++ // Increment count for each number
	}
	return freqMap
}

// 2️⃣ Find First Non-Repeating Character
func firstNonRepeatingChar(s string) rune {
	charCount := make(map[rune]int)
	for _, ch := range s {
		charCount[ch]++
	}
	for _, ch := range s {
		if charCount[ch] == 1 {
			return ch
		}
	}
	return 0 // means none found
}

// 3️⃣ Check for Duplicates
func containsDuplicate(nums []int) bool {
	seen := make(map[int]bool)
	for _, num := range nums {
		if seen[num] {
			return true
		}
		seen[num] = true
	}
	return false
}

// 4️⃣ Is Anagram
func isAnagram(s, t string) bool {
	if len(s) != len(t) {
		return false
	}
	count := make(map[rune]int)
	for _, ch := range s {
		count[ch]++
	}
	for _, ch := range t {
		count[ch]--
		if count[ch] < 0 {
			return false
		}
	}
	return true
}

// 5️⃣ Group Words by Anagrams
func groupAnagrams(words []string) [][]string {
	anagramMap := make(map[string][]string)
	for _, word := range words {
		// Convert to a sorted string as the key
		runes := []rune(word)
		sort.Slice(runes, func(i, j int) bool {
			return runes[i] < runes[j]
		})
		key := string(runes)
		anagramMap[key] = append(anagramMap[key], word)
	}

	result := [][]string{}
	for _, group := range anagramMap {
		result = append(result, group)
	}
	return result
}

// 6️⃣ Top K Frequent Elements
func topKFrequent(nums []int, k int) []int {
	freqMap := countFrequencies(nums)
	type kv struct {
		Key   int
		Value int
	}
	var freqSlice []kv
	for key, val := range freqMap {
		freqSlice = append(freqSlice, kv{key, val})
	}
	// Sort by frequency in descending order
	sort.Slice(freqSlice, func(i, j int) bool {
		return freqSlice[i].Value > freqSlice[j].Value
	})
	var result []int
	for i := 0; i < k && i < len(freqSlice); i++ {
		result = append(result, freqSlice[i].Key)
	}
	return result
}

// 7️⃣ Two Sum Problem
func twoSum(nums []int, target int) []int {
	indexMap := make(map[int]int)
	for i, num := range nums {
		if idx, found := indexMap[target-num]; found {
			return []int{idx, i}
		}
		indexMap[num] = i
	}
	return nil
}

// 8️⃣ Find Common Elements Between Two Arrays
func commonElements(arr1, arr2 []int) []int {
	elemMap := make(map[int]bool)
	for _, num := range arr1 {
		elemMap[num] = true
	}
	var result []int
	for _, num := range arr2 {
		if elemMap[num] {
			result = append(result, num)
		}
	}
	return result
}

// 9️⃣ Find Missing Number
func missingNumber(nums []int, n int) int {
	exists := make(map[int]bool)
	for _, num := range nums {
		exists[num] = true
	}
	for i := 1; i <= n; i++ {
		if !exists[i] {
			return i
		}
	}
	return -1
}

// 🔟 Find Element with Single Occurrence
func singleOccurrence(nums []int) int {
	count := make(map[int]int)
	for _, num := range nums {
		count[num]++
	}
	for key, val := range count {
		if val == 1 {
			return key
		}
	}
	return -1
}

func main() {
	// 1️⃣ Count Frequencies
	fmt.Println("1:", countFrequencies([]int{1, 2, 2, 3, 1, 4}))

	// 2️⃣ First Non-Repeating Char
	fmt.Printf("2: %c\n", firstNonRepeatingChar("gogopher"))

	// 3️⃣ Contains Duplicate
	fmt.Println("3:", containsDuplicate([]int{1, 2, 3, 4, 1}))

	// 4️⃣ Is Anagram
	fmt.Println("4:", isAnagram("listen", "silent"))

	// 5️⃣ Group Anagrams
	fmt.Println("5:", groupAnagrams([]string{"bat", "tab", "cat", "act"}))

	// 6️⃣ Top K Frequent Elements
	fmt.Println("6:", topKFrequent([]int{1, 1, 1, 2, 2, 3}, 2))

	// 7️⃣ Two Sum
	fmt.Println("7:", twoSum([]int{2, 7, 11, 15}, 9))

	// 8️⃣ Common Elements
	fmt.Println("8:", commonElements([]int{1, 2, 3}, []int{2, 3, 4}))

	// 9️⃣ Missing Number
	fmt.Println("9:", missingNumber([]int{1, 2, 4, 5}, 5))

	// 🔟 Single Occurrence
	fmt.Println("10:", singleOccurrence([]int{4, 1, 2, 1, 2}))
}


```
