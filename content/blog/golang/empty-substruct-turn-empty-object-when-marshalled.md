---
title: "Golang: Empty Object when Marshalling Struct Field"
date: 2024-03-24T18:01:56+07:00
description: "Output Empty Object `{}` rather than `{"field1":"", "field2":""}`"
---

# Initial Requirement
Imagine You want to have this output:
```json
{
    "name": "John Doe",
    "gender": "male",
    "occupation" : {
        "name" : "brick layer",
        "yearOfExperience" : "2"
    }
}
```

It is easy to create the struct, there's https://mholt.github.io/json-to-go/ . I copy the json above and then it turns into this code. I just change the struct name

```go
type Person struct {
    Name       string `json:"name"`
    Gender     string `json:"gender"`
    Occupation struct {
        Name             string `json:"name"`
        YearOfExperience string `json:"yearOfExperience"`
    } `json:"occupation"`
}
```

Then we separate the occupation struct 

```go
type (
    Person struct {
        Name       string `json:"name"`
        Gender     string `json:"gender"`
        Occupation Occupation  `json:"occupation"`
    }

    Occupation struct{ 
        Name             string `json:"name"`
        YearOfExperience string `json:"yearOfExperience"`
    }
)
```

If we just return empty person,
```go
p := Person{}

s, _ := json.MarshalIndent(p, "    ", "")
fmt.Println(string(s))

```

This is what it will return:
```json
{
    "name": "",
    "gender": "",
    "occupation":
    {
        "name": "",
        "yearOfExperience": ""
    }
}
```

# I Want `"occupation":{}` Instead Empty Fields

## First Solution, `omitempty`
When `Occupation` filled partially, it ONLY show filled fields
From :
```go
type Occupation struct {
    Name string `json:"name"`
    YearOfExperience string `json:"yearOfExperience"`
}
```
Add `omitempty` right behind its json field name.

To:
```go
type Occupation struct {
    Name string `json:"name,omitempty"`
    YearOfExperience string `json:"yearOfExperience,omitempty"`
}
```

Check this demo https://go.dev/play/p/3SuD_zHsctp

Full Code:
```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    type (
        Occupation struct {
            Name             string `json:"name,omitempty"`
            YearOfExperience string `json:"yearOfExperience,omitempty"`
        }
        Person struct {
            Name       string     `json:"name"`
            Gender     string     `json:"gender"`
            Occupation Occupation `json:"occupation"`
        }
    )
    p := Person{}

    s, _ := json.MarshalIndent(p, "    ", "")
    fmt.Println("=== Without omit")
    fmt.Println(string(s))
    p = Person{}
    p.Occupation.Name = "Lawyer"

    s, _ = json.MarshalIndent(p, "    ", "")
    fmt.Println("=== WITH omit")
    fmt.Println(string(s))
}
```

## Second Solution. make `Occupation` on `Person` a pointer representation

When occupation is nil, the `occupation` returns `{}`. 

When occupation is intialized, the `occupation` returns ALL fields.

From :
```go
    Occupation Occupation `json:"occupation"`
```

To:
```go
    // container for occupation pointer 
    type Poccupation struct {
        *Occupation
    }
    Occupation Poccupation `json:"occupation"`
```

Check this demo https://go.dev/play/p/UNwlE4D7sHt

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    type (
        Occupation struct {
            Name             string `json:"name"`
            YearOfExperience string `json:"yearOfExperience"`
        }

        Poccupation struct {
            *Occupation
        }
        Person struct {
            Name       string      `json:"name"`
            Gender     string      `json:"gender"`
            Occupation Poccupation `json:"occupation"`
        }
    )
    p := Person{}

    fmt.Println("=== NIL occupation")
    s, _ := json.MarshalIndent(p, "    ", "")
    fmt.Println("One")
    fmt.Println(string(s))

    fmt.Println("=== EMPTY occupation")
    p = Person{}
    p.Occupation = Poccupation{&Occupation{}}
    s, _ = json.MarshalIndent(p, "    ", "")
    fmt.Println(string(s))

    fmt.Println("=== Partial occupation")
    p = Person{}
    p.Occupation = Poccupation{&Occupation{Name: "layer"}}

    s, _ = json.MarshalIndent(p, "    ", "")
    fmt.Println(string(s))
}
```

