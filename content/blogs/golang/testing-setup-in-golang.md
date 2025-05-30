---
author: "Yanuar Arifin"
title: "Testing Set Up in Golang"
date: "2021-11-30"
description: "How to setup your test in golang and some of gotcha"
tags: [
    "golang",
    "testing",
    "tdd",
]
draft: false
---

File conventions, function naming conventions, how to separate test to test cases, send output (fmt.Println-ish), mark the test as fail
<!--more-->

# Setting Up Test in Golang

## 1.  Where to put and How to Name Test class

Golang have specific way to create test classes. 

**where to put?** in the same directory as the file we want to test

**naming convention**: filename_test.go

ex.
```
|- hello
    |- Greeting.go
    |- Greeting_test.go
```

## 2.  How to test a function

The name of your test function have a relationship with function name on your actual code. The template is  the `<functionName>Test.go`  and has one parameter `(t *testing.T){}`

To test this function from `Greeting.go`, 

```go
# hello/Greeting.go
func Hello(){
    fmt.Println("hello there")
}
```

create  this

```go
# hello/Greeting_test.go
func TestHello(t *testing.T){
    fmt.Println("hello there")
}
```

## 3.  how to separate each test case

Use `t.Run(description, callback)` 

```go
# hello/Greeting_test.go
func TestHello(t *testing.T){
    # test case  1
    t.Run("can call Hello()", func(t *testing.T){
    })
    # test case  2
    t.Run("warn when the param is empty", func(t *testing.T){
    })
    
    # test case n
    # ...
}
```

## 4.  helper function when testing a method

When there's repetition, we DRY it by creating function. But golang could not create function inside function(nested function). The solution is to create variable with type  of function

```go
func TestHello(t *testing.T){
    callApi := func(URL string, params string) (string, errors){
        # ...
        return var, err
    }
    # ...
```

## 5. Show Output

Print output and continue the test

```go
t.Log("string")
t.Logf("string with param %v %d", a, b)
```

Mark current test or `t.Run(...)`  as fail and on to next test

```go
t.Error("string")
t.Errorf("string with param %v %d", a, b)
```

## Assertion?

Golang's standard library does not have assertions, check out this library if you need assertion https://github.com/stretchr/testify