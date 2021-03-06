OneDB [![Build Status](https://travis-ci.org/cvgellhorn/OneDB.svg?branch=master)](https://travis-ci.org/cvgellhorn/OneDB)
===========

> A lightweight/single file PHP database framework

## Overview
OneDB is using the PDO_MYSQL extension and is based on three classes:

* <b>OneDB</b> - Main database framework
* <b>OneExpr</b> - Database expression
* <b>OneException</b> - Exception

All tests are based on the [PHPUnit](http://phpunit.de/) testing framework. You can easily set up your own phpunit.xml, for local unit testing. It's also very lightweight, only around 13 kb and all packed in a single PHP file.

### Server Requirements:

* PHP >= 5.4
* PDO_MYSQL extension

## Getting started
```php
// Include OneDB
require_once 'OneDB.php';

// Create OneDB instance and have fun
$database = OneDB::load([
    'database'  => 'application',
    'user'      => 'root',
    'password'  => 'admin123#'
]);

// After initializing, you can always get the current instance with
$database = OneDB::load();


// Or create a new connection by name (for multiple connections)
$dbWrite = OneDB::getConnection('write', [
    'database'  => 'application',
    'user'      => 'root',
    'password'  => 'admin123#'
]);

// Reload connection again later
$dbWrite = OneDB::getConnection('write');
```

## Configuration
You can also set the database host, port and charset.
```php
$database = OneDB::load([
	'host'      => 'sql.mydomain.com',
    'port'      => '3307',
    'charset'   => 'utf16',
    'database'  => 'application',
    'user'      => 'root',
    'password'  => 'admin123#'
]);
```

Default settings
```php
'host'    => 'localhost'
'port'    => '[default_mysql_port]'
'charset' => 'utf8'
```

## Basic Usage
### Insert
Insert new records in table, returns LAST_INSERT_ID.

```php
insert($table : string, $data : array)
```

Example:
```php
$lastInsertId = $database->insert('user', [
	'name'  => 'John Doe',
    'email' => 'john@doe.com',
    'tel'   => 12345678
]);
```

### Update
Edit data in table. You can use any given operator in the WHERE clause to filter the records. The ? represents the placeholder for the given param.

```php
update($table : string, $data : array, [$where : array])
```

Example:
```php
$database->update('user',
    [
		'name'   => 'John Smith',
    	'email'  => 'john@smith.com',
    	'tel'    => 87654321
    ],
    [
    	'id = ?' => 23
    ]
);
```

### Delete
Remove data from table. Just as update, the ? represents the placeholder for the given param.

```php
delete($table : string, [$where : array])
```

Example:
```php
$database->delete('user', [
	'id = ?' => 23
]);
```

### Fetch All
Retrieve all the rows of the result set in one step as an array.
```php
fetchAll($sql : string)
```

Example:
```php
$database->fetchAll('SELECT * FROM `user`');
```

### Fetch Assoc
Retrieve all the rows of the result set in one step as an array, using the first column or the given key as the array index.
```php
fetchAssoc($sql : string, [$key : string])
```

Example:
```php
$database->fetchAssoc('SELECT * FROM `user`', 'username');
```

### Fetch Row
Retrieve a single row of the result set as an array.
```php
fetchRow($sql : string)
```

Example:
```php
$database->fetchRow('SELECT * FROM `user` WHERE `id` = 1');
```

### Fetch One
Retrieve a single result value.
```php
fetchOne($sql : string)
```

Example:
```php
$database->fetchOne('SELECT `username` FROM `user` WHERE `id` = 1');
```

### Query
Send an SQL query. If there is a result, you will automatically get the matched result type: fetch all, fetch row or fetch one.
```php
query($sql : string)
```

Example:
```php
$database->query('DELETE FROM `user` WHERE `id` = 1');

// With result
$result = $database->query('SELECT * FROM `user`');
```

### Last Insert ID
Returns the ID of the last inserted row.
```php
lastInsertId()
```

Example:
```php
$database->lastInsertId();
```


## Advanced Usage
### Expression
You can also use database expressions in your statement, by using the OneExpr object.
```php
$lastInsertId = $database->insert('user', [
    'name'    => 'John Doe',
    'email'   => 'john@doe.com',
    'tel'     => 12345678,
    'created' => new OneExpr('NOW()')
]);
```

### Truncate
Truncate database table.
```php
truncate($table : string)
```

Example:
```php
$database->truncate('user');
```

### Drop
Drop database table.
```php
drop($table : string)
```

Example:
```php
$database->drop('user');
```

### Describe
Describe database table, returns the table attributes as array keys.
```php
describe($table : string)
```

Example:
```php
$database->describe('user');
```

### Transaction
Run a database transaction.
```php
try {
	// Start transaction
	$database->beginTransaction();

	// Do stuff
	$database->insert('user', [
		'name' => 'Skywalker'
	]);
	$database->delete('user', [
		'id = ?' => 3
	]);

    // Check transaction status, returns bool
    $status = $database->inTransaction();

	// Commit transaction if no error occurred
	$database->commit();
} catch (OneException $e) {
	// Rollback on error
	$database->rollBack();
}
```

### Quote
Add quotes to the given value.
```php
quote($val : string)
```

Example:
```php
$database->quote($value);
```

### Backtick
Add backticks to the given field name.
```php
btick($val : string)
```

Example:
```php
$database->btick('user');
```

### PDO
Returns the current PDO object.
```php
getPDO()
```

Example:
```php
$database->getPDO();
```


## Special Usage
### Multi Insert
Insert multiple records into database table.
```php
multiInsert($table : string, $keys : array, $data : array)
```

Example:
```php
$database->multiInsert('user',
    ['name', 'email', 'tel'],
    [
    	[
        	'John Doe',
            'john@doe.com',
            12345678
        ],
        [
        	'John Smith',
            'john@smith.com',
            11223344
        ),
        [
        	'Jack Smith',
            'jack@smith.com',
            87654321
        ]
	]
);
```

### Save
Update data if exist, otherwise insert new data. Using the ON DUPLICATE KEY UPDATE expression. Returns the ID of the last inserted or updated row.
```php
save($table : string, $data : array)
```
Example:
```php
$id = $database->save('user', [
	'id'	=> 1,
    'name'  => 'John Doe',
    'email' => 'john@doe.com',
    'tel'   => 12345678
]);
```


## Debug
You can activate the debug mode by using the following statement. It will show you all executed SQL queries and the parameter bindings.
```php
$database->debug();
```

It's also possible to change the debug style with the debugStyle attribute.
```php
$database->debugStyle = [
	'border: 2px solid #d35400',
	'border-radius: 3px',
	'background-color: #e67e22',
	'margin: 5px 0 5px 0',
	'color: #ffffff',
	'padding: 5px'
];
```
