---
title: Laravel 7.x Seeding
date: 2020-09-14
tags: [laravel, laravel 7, php, migration, seeding, populate, artisan, command, faker]
draft: false
---

Laravel Version: 7.x(7.24)

Seeding is one of my favorite tools that helps my development tremendously.
This is how you could insert some(I mean whatever amount you want) records to database.
Using [Faker](https://github.com/fzaninotto/Faker) library to randomize the value, you could specify how far the
randomness of your record values are. Check [what Faker Provider could do](https://github.com/fzaninotto/Faker#formatters) and there's
additional [Faker Provider Collection](https://github.com/mbezhanov/faker-provider-collection) to step up your faking game.

- Just clone a project from the cloud? migrate then seed the database
- Think that your database values is confusing to manage? empty then reinsert data
- Make a new table but too lazy to populate it? just run Seeder
- Has migration with many tables and its a mess to seed the data? Use Seeder

All with a simple CLI command.

```
php artisan db:seed
```

Or for more granular control

```
php artisan db:seed --class=SomeTableSeeder
```

Interested? carry on reading!

# Installation

Faker is shipped default with Laravel & Lumen, But you need to install the
[Faker Provider Collection](https://github.com/mbezhanov/faker-provider-collection).
 If you are not sure what is provided, once again do check their providers, its helpful.

 ```
 composer require mbezhanov/faker-provider-collection
 # just in case for migration
 composer require doctrine/dbal
 ```

 That will do.

# Make an Empty Database

Make a database of your choice, then update you `.env` database detail to match your database setting.

```env
DB_CONNECTION=mysql
DB_HOST=192.168.10.10
DB_PORT=3306
DB_DATABASE=lara-seeding-tutorial
DB_USERNAME=homestead
DB_PASSWORD=secret
```

Here I use MySQL 8 for the database called `lara-seeding-tutorial` in a Laravel Homestead
which has ip of `192.168.10.10` and port `3306`. The MySQL username is `homestead` and
the password is `secret`

Update yours according to your database credential.

# Create Table

We will use laravel's migration tools to create the table, so in the future we could
run this migration script to create the tables instead of running an sql file dump.
Faster and easier.

We will create a migration file that create a table called `products`. Use this command
to generate that file.
```
php artisan make:migration create_products_table
php artisan make:migration create_categories_table
php artisan make:migration create_transactions_table
```

Which will create a new files

```
/database/migrations/[current_date]_[somenumber]_create_products_table.php
/database/migrations/[current_date]_[somenumber]_create_categories_table.php
/database/migrations/[current_date]_[somenumber]_create_transactions_table.php
```

This is the current file content and its modidification highlighted:

This file will create table called `products` with columns named:
```diff
<?php
# /database/migrations/2020_09_07_112121_create_categories_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProductsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
+            $table->string('name');
+            $table->text('description');
+            $table->integer('price');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('products');
    }
}

```

```diff
<?php
# /database/migrations/2020_09_07_112121_create_categories_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateCategoriesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('categories', function (Blueprint $table) {
+           $table->id();
+           $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('categories');
    }
}

```

```diff
<?php
# /database/migrations/020_09_07_112121_create_categories_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTransactionsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('transactions', function (Blueprint $table) {
            $table->id();
+           $table->foreignId('product_id')->nullable();
+           $table->unsignedBigInteger('price');
+           $table->integer('quantity');
            $table->timestamps();

+           $table->foreign('product_id')->references('id')->on('products')->onDelete('set null');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
+       Schema::table('transactions', function (Blueprint $table){
+           $table->dropForeign('transactions_product_id_foreign');
+       });
        Schema::dropIfExists('transactions');
    }
}

```


Let's create a simple table for product, we will have product `name`, `description`, and `price` on the table


Next step is test this migration file to be able to create table & de-create it a.k.a drop it
when necessary.

To make the migration take effect, use this script
```
php artisan migrate
```

Which will result in this(the user related & jobs tables are built in from laravel install)
```cli
vagrant@homestead:~/code/lara-seeding-tutorial$ php artisan migrate
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.1 seconds)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (0.09 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.06 seconds)
Migrating: 2020_09_06_160325_create_products_table
Migrated:  2020_09_06_160325_create_products_table (0.05 seconds)
```

Congratulation, from 2 last line, your table is migrated. You could check it in your database,
there will be a new table called `products` there, along with some other.

Now we check if the table could be deleted when you reset the migration by run this command:
```
php artisan migrate:rollback --step=1
```

This command will reset the last migrate command described in the `down()` function of your last
file(s) affected by the migration. In our case it will delete tables name `users`, `password_resets`,
`failed_jobs`, and `products` table.

# Create Models

Run this to create model class for products table.
```
php artisan make:model Models/Product
```

The result will be located at `app/Models/Product`. This class will seek table called `products`


# Define Your Columns Value

Now that we could create the table structure with ease, it is time for us to define what kind of values those tables columns have.
We will define this values in a class called Factory Class. In laravel you could create this class through command line

```
php artisan make:factory ProductFactory
```

This will make a new file `/database/factories/ProductFactory.php` with following content:

```php
<?php

/** @var \Illuminate\Database\Eloquent\Factory $factory */

use App\Model;
use Faker\Generator as Faker;

$factory->define(Model::class, function (Faker $faker) {
    return [
        //
    ];
});

```

Update it to look like this

```diff
<?php

/** @var \Illuminate\Database\Eloquent\Factory $factory */

- use App\Model;
+ use App\Models\Product;
+ use Bezhanov\Faker\ProviderCollectionHelper;
  use Faker\Generator as Faker;

- $factory->define(Model::class, function (Faker $faker) {
+ $factory->define(Product::class, function (Faker $faker) {
+     ProviderCollectionHelper::addAllProvidersTo($faker);
      return [
-         //
+         'name' => $faker->productName,
+         'description' => $faker->realText(),
+         'price' => $faker->numberBetween(200, 999999),
      ];
  });
```

This `ProductFactory` file will describe the value of your column

```php
ProviderCollectionHelper::addAllProvidersTo($faker);
```

Will add additional categories of things to choose from.

```php
        'name' => $faker->productName,
        'description' => $faker->realText(),
        'price' => $faker->numberBetween(200, 999999),
```
`productName` will make random product name that we could still relate

`realText()` will make random sentence

`numberBetween(200, 9000)` will make random number between 200 to 9000

# Your First Seeder

This is the class which you would run to insert your database records. This class could
contains the way you would insert your records to database.

There are more than one way to insert records to your table(s).

If you have a kind of fixed types table, maybe a list of genders, or a few status choices, You could use `DB::insert()` instead of the Factory file like above.
That would be simpler.

But when you have a table with many columns and records coming from user input(meaning this will be random within certain criteria),
the Factory file is your friend. Let your computer randomise those input, inserting hundreds or thousands of records  will be a breeze.

Now let's make a Seeder class.
My way of seeding tables is make a class named after the table name with `Seeder` ending.
 For example we wanted to make a Seeder for products table , I just call it ProductSeeder.
 Laravel has the command to make the class

 ```
 php artian make:seeder ProductSeeder
 ```

You will found the newly generated class in `/database/seeds/ProductSeeder.php`.
Let's take a look inside:
```php
<?php

use Illuminate\Database\Seeder;

class ProductSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        //
    }
}
```

We just need to call the factory that we have describe.
```diff
  <?php
+ use App\Models\Product;
  use Illuminate\Database\Seeder;

  class ProductSeeder extends Seeder
  {
      /**
       * Run the database seeds.
       *
       * @return void
       */
      public function run()
      {
-          //
+          factory(Product::class, 137)->create();
      }
  }
```

That is it. We will insert a 137 records on `products` table.

to run this,  use this command

```
php artisan db:seed --class=ProductSeeder
```

And when you check the database records of `products` table, 137 new records is there.

![inserted product records](https://i.postimg.cc/jjkWBBCw/2020-09-07-230104-1033x635-scrot.png)

Tadaaa. The `products` table has records now, exacly 137 records.

if you are not certain, run this command to reset your database structure

```
php artisan migrate:rollback
```

This command will revert back your database to only one table, `migrations` table

Then run this command to create those tables
```
php artisan migrate
```

which will migrate all your tables, you will see the `products` tables is back.
But there's no database records there. Run the seeding command again to fill it

```
php artisan db:seed --class=ProductSeeder
```

Refresh your database, and you will see records on your `products` table with different
values, but still inside the criteria

# Conclusion

In this article I demonstrate to you how migration & seeding work using laravel. This combination
could be used in other framework too using different tools. This is a simple example.
