# Eloquent PHPUnit

[![StyleCI](https://styleci.io/repos/65038313/shield)](https://styleci.io/repos/65038313)

#### Test your Laravel Eloquent model's and database schema

This package was inspired by the Ruby on Rails world and the testing framework RSpec. The Ruby on Rails community (for the most part) write tests for their models in a way that they check the model's attributes, relationships, database table and columns settings (defaults, nullable, etc.).

## Table of Contents

1. [What can be tested](#what-can-be-tested)
2. [Installation](#installation)
3. [Requirements](#requirements)
4. [Documentation](#documentation)
	1. [Properties](#test-class-properties)
	2. [Table Testing Methods](#database-testing-methods)
	3. [Model Testing Methods](#model-testing-methods)
5. [Example Model Test Class](#example-model-test)
6. [Contributing](#contributing)
7. [Version Release History](#history)
8. [Projects using Eloquent-PHPUnit](#projects-using-eloquent-phpunit)
9. [Author](#author)
10. [License](#license)

![Test a database table](docs/database-test-method.png "Test a database table.")

![Test an eloquent model's attributes/relationships](docs/model-test-method.png "Test an eloquent model's attributes/relationships.")

## What can be tested

- Casted attribute array
- Fillable attribute array
- Hidden attribute array
- Dates attribute array
- Relationship methods

You can also test your database tables such as:
- Table exists
- Table column exists
- Table column type (string, text, date, datetime, boolean, etc.).
- Column default value
- Null/Not Null
- Auto-incremented primary keys.
- Table indexes
- Unique indexes
- Foreign Key relationships

## Installation

1. The easiest way to use/install this package is by using composer in your terminal:
```
composer require erikgall/eloquent-phpunit
```
2. Or you can add the following line to your `require-dev` dependencies in your `composer.json` file
```json
{
	"require-dev": {
		"erikgall/eloquent-phpunit": "~1.0"
	}
}
```

## Requirements

This package requires `PHP 5.6` or `PHP 7+`. It has been tested and used with `Laravel 5.2` and `Laravel 5.3`. There should not be a problem using it with `Laravel 5.0/1` but it has not been tested or confirmed 100%.

## Documentation

### Test Class Properties

| Name | Type | Required | Default  | Description |
|---------------|-------------------------------------|----------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| defaultSeeder | string | false | DatabaseSeeder | The database seeder class name that calls the rest of your seeders (only used if seedDatabase property is not set to false). |
| data | array | false | - | Do not overwrite this property. It is used to store the model's data. You can access this data by calling any of the data array's keys like a class property ($this->fillable, $this->casts, $this->table) |
| model | string | true | - | The FQCN of the eloquent model that is to be tested (ex. App\User) |
| seedDatabase | boolean | false | true | Should the database be seeded before each test. If you are not running tests that require data in the database, you should set this to false to speed up your tests. |
| seeders | array | false | - | If you wish to only call certain seeder classes you can set them here (ex. `['UsersTableSeeder', 'PostsTableSeeder']` (only used if seedDatabase property is not set to false). |
| subject | Model** | false | -  | This is the instance of the model class that is being tested. When setting up a test, the EloquentTestCase class initializes a new empty model. |

**These settings are only used if the seedDatabase property is not set to false (the default value for the seedDatabase property is true).*

** The subject property is an instance of \Illuminate\Database\Eloquent\Model.

### Database Testing Methods

#### \EGALL\EloquentPHPUnit\Database\Table

Get the `EGALL\EloquentPHPUnit\Database\Table` class instance by calling the table property.

**Usage:**

```php
	$this->table
```

#### Table methods
---

##### column($columnName)

Initializes a new `EGALL\EloquentPHPUnit\Database\Column` class instance for table's column name that is passed in.

**Usage:**

```php
	$this->table->column('column_name')
```

Returns: `EGALL\EloquentPHPUnit\Database\Column`

--- 

##### exists()

Assert that the table exists in the database.

**Usage:**

```php
	$this->table->exists();
```

**Returns:** `EGALL\EloquentPHPUnit\Database\Table`

---

##### hasTimestamps()

Assert that the table has timestamp columns.

**Usage:**

```php
	$this->table->hasTimestamps();
```

**Returns:** `EGALL\EloquentPHPUnit\Database\Table`

--- 

##### resetTable($tableName)

Using this method it is possible to test multiple tables in one test class. 


**Usage:**

The usage code below is using a user/user-roles, example. The relationship is as follows: A user can have many role and a role can have many users (many-to-many). In eloquent, we would describe this relationship as a user `belongsToMany` roles through the user_role table and vice-versa. 

```php

	protected $model = 'App\Role';


	public function testDatabase() {

		// We are testing the roles table below
		$this->table->column('id')->increments();
        $this->table->column('name')->string()->notNull()->unique();
        $this->table->column('label')->string()->notNull();
        $this->table->hasTimestamps();

        // To switch to the user_role table we must reset the table.
        // You can method chain off of the resetTable() method as well.
        $this->resetTable('user_role');

        // Begin testing user_role table like normal.
        $this->table->column('role_id')->integer()->primary()->foreign('roles');
        $this->table->column('user_id')->integer()->primary()->foreign('roles');
        $this->table->hasTimestamps();


	}
```

**Returns:** `EGALL\EloquentPHPUnit\EloquentTestCase` / `$this`

---
### Model Testing Methods

// TODO

## Example Model Test

```php
Class UserModelTest extends \EGALL\EloquentPHPUnit\EloquentTestCase {
	
	protected $model = 'App\User';

	// If you want to run the DatabaseSeeder class
	protected $seedDatabase = true;

	// If you only want to run a specific seeder
	protected $seeders = ['UsersTableSeeder', 'SchoolsTableSeeder'];

	// Change the default seeder that calls the rest of your seeders.
	// The default is the default Laravel Seeder named: DatabaseSeeder. 
	// Ex. (You have a TestDatabaseSeeder and the default DatabaseSeeder).
	protected $defaultSeeder = 'TestDatabaseSeeder'

	/**
	 * Test the database table.
	 */
	public function testDatabaseTable() {
		$this->table->column('id')->integer()->increments();
		$this->table->column('name')->string()->nullable();
		$this->table->column('email')->string()->notNullable()->unique();
		$this->table->column('password')->string()->notNullable();
		$this->table->column('dob')->date()->nullable();
		$this->table->column('avatar_id')->integer()->foreign('images', 'id', $onUpdate = 'cascade', $onDelete = 'cascade');
		$this->table->column('is_verified')->boolean()->defaults(false);
		$this->table->column('is_admin')->boolean()->defaults(false);
		$this->table->column('verification_sent_at')->dateTime()->nullable();
		$this->table->column('invite_sent_at')->dateTime()->nullable();
		$this->table->column('api_token')->string()->index();
		$this->table->column('remember_token')->string()->nullable();
		$this->table->hasTimestamps();
	}

	/**
	 * Test the model's properties.
	 */
	public function testModelProperties() {
		$this->hasFillable('name', 'email', 'password', 'dob', 'avatar_id')
			 ->hasHidden('password', 'remember_token')
			 ->hasCasts('is_verified', 'boolean') 
			 // or
			 ->hasCasts(['is_verified' => 'boolean', 'is_admin' => 'boolean'])
			 ->hasDates('verification_sent_at', 'invite_sent_at')
			 ->belongsTo(Image::class) 
			 // if method name = 'image()' or
			 ->belongsTo(Image::class, $customMethod = 'avatar')
			 ->hasMany(Profile::class)
			 ->morphsTo($method = 'slug', $morphTo = 'sluggable') 
			 // or: example below assumes the db fields are: 'sluggable_id' and 'sluggable_type'
			 ->morphsTo('sluggable'); 
	}
}
```

## Contributing

1. Fork it.
2. Create your branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request.

## History

- v1.0.0 Released: 8/5/2016
- v1.0.3 Released: 8/9/2016
- v1.0.6 Released: 9/21/2016

## Projects using Eloquent-PHPUnit

- [Canvas ★803](https://github.com/austintoddj/canvas): A simple, powerful blog publishing platform built on top of Laravel 5 by [@austintoddj](https://github.com/austintoddj)

## Author

- [Erik Galloway](https://github.com/erikgall)


## License

Eloquent-PHPUnit is an open-sourced software licensed under the [MIT license](https://github.com/erikgall/eloquent-phpunit/blob/master/LICENSE).
