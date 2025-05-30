---
title: Laravel Migration
date: 2019-06-16 
draft: true
---

artisan commands
```bash
# create migration
php artisan make:migration create_table_user

# redo migration
php artisan migrate:refresh

# redo migration and seed the records after re-migration
php artisan migrate:refresh --seed

# when seeder class is not detected
# (class autoload fail)
composer dump-autoload

```

## Table Definition
```php
<?php
    # Varchar
    $table->string('column_name', 100);

    # Soft delete record when deleted_at is not null
    $table->softDeletes();

?>
```

### Foreign Key

Step to create foreign key in migration

- Create column
- add index
- add foreign key

`down` function is reversed order

- remove foreign key
- remove index
- remove column

```php
<?php
    public function up()
    {
        Schema::table('barangs', function (Blueprint $table) {
            $table->unsignedBigInteger('id_categories')->after('deskripsi')->nullable(true)->default(null);
            $table->index('id_categories');
            $table->foreign('id_categories')->references('id')->on('categories');
        });
    }
                                                                                                                                                                                                                
    public function down()
    {
        Schema::table('barangs', function (Blueprint $table) {
            $table->dropForeign('barangs_id_categories_foreign');
            $table->dropIndex('barangs_id_categories_index');
            $table->dropColumn('id_categories');
        });
    }


```
