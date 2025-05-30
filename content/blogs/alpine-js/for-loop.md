---
title: Using For Loop In Alpine JS
date: 2020-02-26
tags: [js, html, alpine, vue, react]
draft: false
---

The steps are:
- add alpine.js to head section of your HTML
- create loop data
- get data on template
- loop and show data

## Create Loop Data

Make a function inside a script tag just above the `</body>` tag
```html
...
<body>
...
<script>
  function theData() {
    return {
      produk: [
        {
          name: "Boaty McBoatface",
          price: "$300.000",
        },
        {
          name: "WD-40",
          price: "$45.000",
        },
        {
          name: "Bugs for lyfe",
          price: "$105.000",
        },
    ]};
  }
</script>
</body>
```

## Get Data to HTML

We need to make it available on the HTML. Use `x-data` from alpine.js to get the produk list above

```html
<div x-data="theData()">
  <!-- This wil make the arry of produk available inside this div -->
</div>
```

`x-data` will make all data and function(behaviour) inside `theData()` function available inside the div scope

## Loop it and print

```html
<div x-data="produk()">
  <!-- This wil make the arry of produk available inside this div -->
  <template>
    <div x-for="item in produk">
      <h1 x-text="item.name"></h1>
      <h4 x-text="item.price"></h4>
    </div>
  </template>
</div>
```
`x-for` --> will iterate the `produk` property inside that available from the `x-data` scope

`<template>` --> `x-for` **MUST** use this tag. Couldn't use standard HTML tags

`x-text` --> will update `innerText` of an element

## Use Case

I use this to create a list of product when create an e-commerce template
because creating template means that we need to change the HTML and CSS constantly.
Previously there are only full blown templating engine included on a web framework
or need a more complicated setup.

Using alpine.js is much more simple.
