---
title: "Golang: Keep JSON String as JSON. Or Dynamic JSON in Golang"
date: 2023-05-24T18:01:56+07:00
description: "Output JSON from JSON string"
---

## The Problem

I want to use response where the `content` field comes from external API response which will return varied formats. For example:

```json
{
    "souce": "http://guesthost.com/check-in",
    "payload": {
        "token" : "somerandomtoken",
        "id" : "1up"
    },
    "content": {
            "name": "John Doe",
            "occupation": "Doctor"
    }
}
```
or 
```json
{
    "souce": "http://localhost.com/hello-world",
    "payload": {
        "token" : "somerandomtoken",
    },
    "content": {
        "status": 404,
        "message": "Missing resource or wrong naming"
    }
}
```

If you come here you should know that the usual way for Golang to create JSON is using struct then convert it to JSON string using `json.Marshal()`. But the downside is we need to create every Struct for each JSON if we want to represent all those formats. Like my current problem.

ex.

```go
type ReturnStruct struct{
    Source string `json:"source"`
    Payload RequestPayload `json:"payload"`
    Content string `json:"content"`
}
```

this will return the `content` field on JSON as string. Which in turn will escape the JSON string response from 3rd party.  Having a string return instead of JSON is little bit pain especially when we want to process the data later as this kind of data will be stored to database(usually) and the data processor can be external app that reads JSON.

## What can I do? 

- Using map is out of question. We could make `map[string]interface{}` but the output will become an array instead of JSON.
- Creating each struct for each content is a no go too. If possible we could just automate it.


## The answer

Introducing `json.RawMessage`.

We only need to specify the type of the field to `*json.RawMessage` to make the JSON string is retained as JSON object instead of other types(ex. string) when we `json.Marshal()` it


```go
import (
	"encoding/json"
	"fmt"
	"os"
)

func main() {
    // Read JSON string
	h := json.RawMessage(`{"precomputed": true}`)

    // Declare the main JSON structure
    // NOTICE the header type and how its value is coming from
	c := struct {
		Header *json.RawMessage `json:"header"`
		Body   string           `json:"body"`
	}{Header: &h, Body: "Hello Gophers!"}

	b, err := json.MarshalIndent(&c, "", "\t")
	if err != nil {
		fmt.Println("error:", err)
	}
	os.Stdout.Write(b)

}
```

Looking by the example above. If we want to change the header value. We could do it in the first line  of the `main()` function which is string, not a struct. This is the flexibility of `json.RawMessage` that breaks the assumption that creating dynamic JSON in golang is hard.

## Conclusion

This is a great feature to compose dynamic JSON for your app and/or storage using Golang. 
