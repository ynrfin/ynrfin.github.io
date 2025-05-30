---
title: Lumen Response
date: 2019-06-16 
draft: true
---

## Overview First
We are trying to create response that adhere jsonapi.org standard.

## Laravel Resource, Why we Use this Resource
Yes, could create response using `json_encode()`. 

Buuut, there's a good practice of having a standard. So that everyone is in the same page.

    - Backend
    - Frontend
    - Mobile



https://medium.com/@dinotedesco/using-laravel-5-5-resources-to-create-your-own-json-api-formatted-api-2c6af5e4d0e8

## Exception Response

```php
<?php
public function render($request, Exception $e)
{
    $rendered = parent::render($request, $e);

    return response()->json([
        'error' => [
            'code' => $rendered->getStatusCode(),
            'message' => $e->getMessage(),
        ]
    ], $rendered->getStatusCode());
}
```

## How To Create Response

1. Create response Class


