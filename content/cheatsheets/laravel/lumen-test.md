---
title: Lumen Test
date: 2019-08-26 
draft: true
---
# JSON Test

## create json test request

inside test case

```php
<?
    $this->json(HTTP_method, request_body);
    $this->json('POST', []);
```

The request above will return `Illuminate\Http\JsonResponse`


## Get Plain Response Body

    $this->response->content();
