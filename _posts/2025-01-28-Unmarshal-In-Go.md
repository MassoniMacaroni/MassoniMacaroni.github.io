---
title: "Using Unmarshal in Go with structs"
categories: Blogging
description: "How to use unmarshal with structs in go"
tags: [learning, programming, Go]
---

Dealing with JSON in Go? The `Unmarshal` function of Go's `encoding/json` package is a powerful tool for decoding JSON data into Go types. Whether you're fetching data from an API or processing JSON files, understanding how to use `Unmarshal` effectively can make your Go projects much smoother. This guide covers the basics, use cases, and examples of unmarshalling JSON into Go structs.

[See the official Go documentation for Unmarshal here](https://pkg.go.dev/encoding/json#Unmarshal). In essence, `Unmarshal` parses JSON-encoded data and stores it in the value pointed to by `v`. If `v` is nil, it allocates a new value for `v` to point to.

```go
func Unmarshal(data []byte, v any) error
```

#### Importing the Required Package
Before using Unmarshal, ensure you’ve imported the encoding/json package.

```go
import (
  "encoding/json"
)
```

## Why Use Unmarshal in Go?
When data is received from external sources like APIs or REST endpoints, it's often in JSON format. JSON data needs to be "unmarshalled" into native Go types like structs for efficient processing and usage. `Unmarshal` provides a straightforward and reliable way to handle this transformation.

## How to Use Unmarshal in Go
When working with Unmarshal, follow these steps:

* **Define a Struct:** Create a Go struct that mirrors the structure of your JSON data.
* **Prepare JSON Data:** Store the JSON as a byte slice, which is the format Unmarshal expects.
* **Call json.Unmarshal:** Convert the JSON data into a Go struct by passing the byte slice and a pointer to the struct.
* **Handle the Data:** After unmarshalling, access and manipulate the data as needed.

## Examples of Using Unmarshal

### 1. Basic JSON Object
Check out this simple JSON object:

```json
{
  "name": "Jimbo",
  "person_age": 39,
  "email": "jimbo@example.com",
  "Country": "Austria"
}
```
To process this JSON in Go, we define a struct that maps to its struct:

```go
type Person struct {
  Name  string `json:"name"`
  Age   int    `json:"person_age"`
  Email string `json:"-"`
}
```

#### Key Points:

* Only exported fields (uppercase first letter) are unmarshalled.
* JSON keys are matched to struct fields based on exact or case-insensitive matches.
* Use struct tags to map mismatched JSON keys to Go fields.

#### For example:

* "person_age" maps to Age using the tag json:"person_age".
* "email" is ignored with the tag json:"-".
* "Country" is ignored since it has no corresponding struct field.

#### Code Implementation

```go
func main() {
    jsonData := []byte(`{
        "name": "Jimbo",
        "person_age": 39,
        "email": "jimbo@example.com",
        "Country": "Austria"
    }`)

    var person Person
    err := json.Unmarshal(jsonData, &person)
    if err != nil {
        fmt.Println("Error unmarshalling JSON:", err)
        return
    }

    fmt.Println("Name:", person.Name)
    fmt.Println("Age:", person.Age)
    fmt.Println("Email:", person.Email)
}
```
#### Expected Output:

```
Name: Jimbo  
Age: 39  
Email:  
```
Notice how `Email` is ignored, returning its zero value `("")`.

### 2. JSON Objects with Nested Structures
Handling nested JSON requires nested structs. Check out this JSON:

```json
{
  "id": 101,
  "name": "Jimbo",
  "email": "jimbo@example.com",
  "address": {
    "street": "123 Elm Street",
    "city": "Springfield",
    "zip": "62704"
  }
}
```

Now we need to define structs that represent the nested structure.

```go
type Address struct {
    Street string `json:"street"`
    City   string `json:"city"`
    Zip    string `json:"zip"`
}

type Person struct {
    ID      int     `json:"id"`
    Name    string  `json:"name"`
    Email   string  `json:"email"`
    Address Address `json:"address"`
}
```

Then if we were to access the data we'd unmarshal like so:

```go
func main() {
    jsonData := []byte(`{
        "id": 101,
        "name": "Jimbo",
        "email": "jimbo@example.com",
        "address": {
            "street": "123 Elm Street",
            "city": "Springfield",
            "zip": "62704"
        }
    }`)

    var person Person
    err := json.Unmarshal(jsonData, &person)
    if err != nil {
        fmt.Println("Error unmarshalling JSON:", err)
        return
    }

    fmt.Println("ID:", person.ID)
    fmt.Println("Name:", person.Name)
    fmt.Println("Email:", person.Email)
    fmt.Println("Address:")
    fmt.Println("  Street:", person.Address.Street)
    fmt.Println("  City:", person.Address.City)
    fmt.Println("  Zip:", person.Address.Zip)
}
```

This is what we'd expected to get back from the above function.

```
ID: 101  
Name: Jimbo  
Email: jimbo@example.com  
Address:  
  Street: 123 Elm Street  
  City: Springfield  
  Zip: 62704  
```

## Best Practices for Unmarshal

* **Validate Your JSON Data:** Ensure the JSON format matches your struct.
* **Use Struct Tags:** Specify JSON keys explicitly with json:"key_name" tags.
* **Handle Errors Gracefully:** Always check for errors during unmarshalling.
* **Ignore Unnecessary Fields:** Use json:"-" to ignore fields.

## Conclusion
`Unmarshal` is an essential tool for working with JSON in Go, allowing you to convert JSON data into Go structs efficiently. By understanding how to map JSON keys to struct fields, handle nested objects, and leverage struct tags, you can process JSON data effortlessly in your Go applications.

### FAQs

#### How does Unmarshal handle unknown JSON fields?
Unmarshal ignores unknown fields by default, so they won't cause errors.

#### What happens if a JSON key doesn't match a struct field?
Unmarshal performs a case-insensitive search for a matching field. If none is found, the key is ignored.

#### Can I use Unmarshal for arrays or slices?
Yes, Unmarshal works with arrays and slices by mapping JSON arrays to Go slices.

#### What is the role of struct tags in Unmarshal?
Struct tags map JSON keys to specific struct fields, allowing flexibility and customization.

#### Can Unmarshal handle deeply nested JSON?
Yes, as long as you define nested structs that match the JSON structure.

#### How do I ignore fields during unmarshalling?
Use the tag json:"-" to ignore specific fields.
