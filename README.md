LMongo [![Build Status](https://secure.travis-ci.org/navruzm/lmongo.png?branch=master)](https://travis-ci.org/navruzm/lmongo)
======

LMongo is [MongoDB](http://www.mongodb.org/) package for [Laravel 4](http://laravel.com/). Most part of LMongo is based on [Illuminate/Database](https://github.com/illuminate/database) (Thanks to @taylorotwell)

**Please note that LMongo project is under heavy development, some functionalities are not yet tested. Please report if you find any bug.**

- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Query Builder](#query-builder)
    * [Wheres](#wheres)
    * [Advanced Wheres](#advanced-wheres)
    * [Aggregates](#aggregates)
    * [Inserts](#inserts)
    * [Updates](#updates)
    * [Deletes](#deletes)
    * [Pagination](#pagination)
- [Eloquent for MongoDB](#eloquent-for-mongodb)
- [License](#license)

Installation
============

Add `navruzm/lmongo` as a requirement to composer.json:

```json
{
    "require": {
        "navruzm/lmongo": "*"
    }
}
```
And then run `composer update`

Once Composer has installed or updated your packages you need to register LMongo. Open up `app/config/app.php` and find the `providers` key and add:

```php
'LMongo\LMongoServiceProvider'
```

Then find the `aliases` key and add following line to the array:

```php
'LMongo'          => 'LMongo\Facades\LMongo',
'EloquentMongo'   => 'LMongo\Eloquent\Model',
```

Finally you need to publish a configuration file by running the following Artisan command.

```terminal
$ php artisan config:publish navruzm/lmongo
```
This will copy the default configuration file to app/config/packages/navruzm/lmongo/config.php


Basic Usage
===========
You may get a MongoDB instance by calling the `LMongo::connection` method:

```php
$LMongo = LMongo::connection();
```
This will give you an instance of the default MongoDB server. You may pass the server name to the `connection` method to get a specific server as defined in your mongodb configuration:

```php
$LMongo = LMongo::connection('othermongodbserver');
```
LMongo uses magic method to pass the collection name to the Database class and return MongoCollection instance. Then you can use any of [MongoCollection methods](http://php.net/manual/en/class.mongocollection.php):

```php
$item = $LMongo->collection_name->findOne(array('key' => 'value'));

$items = $LMongo->collection_name->find(array('key' => 'value'))->limit(5);

$LMongo->collection_name->remove(array('key' => 'value'));
```
Get the [MongoDB](http://php.net/manual/en/class.mongodb.php) object:

```php
$mongodb = $LMongo->getMongoDB();

$collection_names = $mongodb->getCollectionNames();
```
Get the [MongoClient](http://php.net/manual/en/class.mongoclient.php) object:

```php
$mongo = $LMongo->getMongoClient();

$databases = $mongo->listDBs();
```

Select/switch the database:

```php
$LMongo->selectDB($dbName);
```

Create the database:

```php
$LMongo->createDB($dbName);
```

Query Builder
=============

Wheres
------
**Retrieving All Rows From A Collection**

```php
$users = LMongo::collection('users')->get();

foreach ($users as $user)
{
    var_dump($user['name']);
}
```

**Retrieving A Single Document From A Collection**

```php
$user = LMongo::collection('users')->where('name', 'John')->first();

var_dump($user['name']);
```

**Retrieving A Single Column From A Document**

```php
$name = LMongo::collection('users')->where('name', 'John')->pluck('name');
```

**Specifying A Fields**

```php
$users = LMongo::collection('users')->get(array('name', 'email'));
```

**Using Where Operators**

```php
$users = LMongo::collection('users')->where('votes', 100)->get();
```

**Or Statements**

```php
$users = LMongo::collection('users')
                ->where('votes', 100)
                ->orWhere('name', 'John')
                ->get();
```

**Nor Statements**

```php
$users = LMongo::collection('users')
                ->where('votes', 100)
                ->norWhere('name', 'John')
                ->get();
```

**Using Where In And Where Not In With An Array**

```php
$users = LMongo::collection('users')
                ->whereIn('id', array(1, 2, 3))->get();
```

**Using Where All With An Array**

```php
$users = LMongo::collection('users')
                ->whereAll('tags', array('php','mongodb'))->get();

$users = LMongo::collection('users')
                ->whereNin('id', array(1, 2, 3))->get();
```

**Using Where Exists**

```php
$users = LMongo::collection('users')
                ->whereExists('updated_at')->get();
```

**Using Where Gt**

```php
$users = LMongo::collection('users')
                ->whereGt('votes', 1)->get();
```

**Using Where Gte**

```php
$users = LMongo::collection('users')
                ->whereGte('votes', 1)->get();
```

**Using Where Lt**

```php
$users = LMongo::collection('users')
                ->whereLt('votes', 1)->get();
```

**Using Where Lte**

```php
$users = LMongo::collection('users')
                ->whereLte('votes', 1)->get();
```

**Using Where Between**

```php
$users = LMongo::collection('users')
                ->whereBetween('votes', 1, 100)->get();
```

**Using Where Ne**

```php
$users = LMongo::collection('users')
                ->whereNe('name', 'John')->get();
```

**Using Where Regex**

```php
$users = LMongo::collection('users')
                ->whereRegex('name', '/John/i')->get();
//or
$users = LMongo::collection('users')
                ->whereRegex('name', new MongoRegex('/John/im'))->get();
```
**Using Where Like**

```php
$users = LMongo::collection('users')
                ->whereLike('name', 'John','im')->get();
```
There are more where methods in [Query/Builder.php](src/LMongo/Query/Builder.php) file.

**Order By**

```php
$users = LMongo::collection('users')
                ->orderBy('name', 'desc')
                ->get();

$users = LMongo::collection('users')
                ->orderBy('name', -1)
                ->get();
```

**Offset & Limit**

```php
$users = LMongo::collection('users')->skip(10)->take(5)->get();
```

Advanced Wheres
---------------

**Parameter Grouping**

```php
LMongo::collection('users')
            ->where('name', 'John')
            ->orWhere(function($query)
            {
                $query->whereGt('votes', 100)
                      ->whereNe('title', 'Admin');
            })
            ->get();
```

Aggregates
----------

```php
$users = LMongo::collection('users')->count();

$price = LMongo::collection('orders')->max('price');

$price = LMongo::collection('orders')->min('price');

$price = LMongo::collection('orders')->avg('price');

$total = LMongo::collection('users')->sum('votes');
```

Distinct
----------

```php
$emails = LMongo::collection('users')->distinct('email');
```

Inserts
-------

**Inserting Document Into A Collection**

```php
$id = LMongo::collection('users')->insert(
    array('email' => 'john@example.com', 'votes' => 0),
);
```

**Inserting Multiple Documents Into A Collection**

```php
$ids = LMongo::collection('users')->batchInsert(
            array('email' => 'john@example.com', 'votes' => 0),
            array('email' => 'henry@example.com', 'votes' => 0),
        );
```

Updates
-------

**Updating Documents In A Collection**

```php
LMongo::collection('users')
            ->where('id', 1)
            ->update(array('votes' => 1));
```

**Incrementing or decrementing a value of a column**

```php
LMongo::collection('users')->increment('votes');

LMongo::collection('users')->decrement('votes');
```

Deletes
-------

**Deleting Documents In A Collection**

```php
LMongo::collection('users')->where('votes', 100)->delete();
//or
LMongo::collection('users')->where('votes', 100)->remove();
```

**Deleting All Documents From A Collection**

```php
LMongo::collection('users')->delete();
//or
LMongo::collection('users')->truncate();
```

Pagination
---------

LMongo has pagination support like Laravel's Query Builder.

```php
$users = LMongo::collection('users')->orderBy('name')->paginate(10);

foreach ($users as $user)
{
    echo $user['name'];
}

echo $user->links();
```

Eloquent for MongoDB
====================

It's similar to Eloquent, except few differences:
* It has a `collection` property, not `table`
* Primary key field is `_id`, not `id`
* No Pivot collections for "Many To Many" relationship. So don't use "Many To Many" relationships on a large datasets.

See [Eloquent Docs](http://four.laravel.com/docs/eloquent).

License
=======

Licensed under the [MIT License](http://cheeaun.mit-license.org/).


TODO
====
* Aggregate/group support
* Indexes
* GridFS