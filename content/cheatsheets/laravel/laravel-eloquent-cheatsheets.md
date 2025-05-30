---
title: Laravel Eloquent Cheatsheets
date: 2019-06-16 
---

## Quick Operation

```php
<?php
User::all();
User::find($id);
User::find([$id1, $id2, $id3]);
User::findOrFail($id);
User::firstOrCreate(['key'=> 'value']);
?>
```

## Relationship


### HasMany

```php
<?php
class Barang extends Model
{
// ...
    public function barangs()
    {
        # default referencing foreign key is <classname_id>
        return $this->belongsTo('App\Category'); 
        # or if you need to refer to non standard column name
        return $this->belongsTo('App\Category', 'id_categories'); 
    }
// ...
}
?>
```
