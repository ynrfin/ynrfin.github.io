---
title: Lumen Request
date: 2019-06-16 
draft: true
---

#### Get Query String

`$request->query('queryParams')`

```php

<?php
public function index(Request $request){
    dd($request->query('qqparams1'))
}
?>
```


#### Get Validated Body

``` php
<?php
public function index(Request $request){
    $validated = $this->validate($request, [
        'kode' => 'required|string',
        'stok' => 'required|integer|min:0',
        'harga' => 'required|integer|min:0',
        'nama' => 'required|string',
        'deskripsi' => 'required|string'
    ]);
}
?>
```
