# Welcome to the PHP MVC framework

This is a simple MVC framework for building web applications in PHP. It's free and [open-source](LICENSE).

## Starting an application using this framework

1. First, download the framework, either directly or by cloning the repo.
1. Run **composer update** to install the project dependencies.
1. Configure your web server to have the **public** folder as the web root.
1. Open [App/Config.php](App/Config.php) and enter your database configuration data.
1. Create routes, add controllers, views and models.

See below for more details.

## Configuration

Configuration settings are stored in the [App/Config.php](App/Config.php) class. Default settings include database connection data and a setting to show or hide error detail. You can access the settings in your code like this: `Config::DB_HOST`. You can add your own configuration settings in here.

## Routing

The [Router](Core/Router.php) translates URLs into controllers and actions. Routes are added in the [front controller](public/index.php). A sample home route is included that routes to the `index` action in the [Home controller](App/Controllers/Home.php).

Routes are added with the `add` method. You can add fixed URL routes, and specify the controller and action, like this:

```php
$router->add('', ['controller' => 'Home', 'action' => 'index']);
$router->add('posts/index', ['controller' => 'Posts', 'action' => 'index']);
```

Or you can add route **variables**, like this:

```php
$router->add('{controller}/{action}');
```

In addition to the **controller** and **action**, you can specify any parameter you like within curly braces, and also specify a custom regular expression for that parameter:

```php
$router->add('{controller}/{id:\d+}/{action}');
```

You can also specify a namespace for the controller:

```php
$router->add('admin/{controller}/{action}', ['namespace' => 'Admin']);
```

## Controllers

Controllers respond to user actions (clicking on a link, submitting a form etc.). Controllers are classes that extend the [Core\Controller](Core/Controller.php) class.

Controllers are stored in the `App/Controllers` folder. A sample [Home controller](App/Controllers/Home.php) included. Controller classes need to be in the `App/Controllers` namespace. You can add subdirectories to organise your controllers, so when adding a route for these controllers you need to specify the namespace (see the routing section above).

Controller classes contain methods that are the actions. To create an action, add the **`Action`** suffix to the method name. The sample controller in [App/Controllers/Home.php](App/Controllers/Home.php) has a sample `index` action.

You can access route parameters (for example the **id** parameter shown in the route examples above) in actions via the `$this->route_params` property.

### Action filters

Controllers can have **before** and **after** filter methods. These are methods that are called before and after **every** action method call in a controller. Useful for authentication for example, making sure that a user is logged in before letting them execute an action. Optionally add a **before filter** to a controller like this:

```php
/**
 * Before filter. Return false to stop the action from executing.
 *
 * @return void
 */
protected function before()
{
}
```

To stop the originally called action from executing, return `false` from the before filter method. An **after filter** is added like this:

```php
/**
 * After filter.
 *
 * @return void
 */
protected function after()
{
}
```

## Views

Views are used to display information (normally HTML). View files go in the `App/Views` folder. Views can be in one of two formats: standard PHP, but with just enough PHP to show the data. No database access or anything like that should occur in a view file. You can render a standard PHP view in a controller, optionally passing in variables, like this:

```php
View::render('Home/index.php', [
    'name'    => 'Dave',
    'colours' => ['red', 'green', 'blue']
]);
```

The second format uses the [Twig](http://twig.sensiolabs.org/) templating engine. Using Twig allows you to have simpler, safer templates that can take advantage of things like [template inheritance](http://twig.sensiolabs.org/doc/templates.html#template-inheritance). You can render a Twig template like this:

```php
View::renderTemplate('Home/index.html', [
    'name'    => 'Dave',
    'colours' => ['red', 'green', 'blue']
]);
```

A sample Twig template is included in [App/Views/Home/index.html](App/Views/Home/index.html) that inherits from the base template in [App/Views/base.html](App/Views/base.html).

## Models

Models are used to get and store data in your application. They know nothing about how this data is to be presented in the views. Models extend the `Core\Model` class and use [PDO](http://php.net/manual/en/book.pdo.php) to access the database. They're stored in the `App/Models` folder. A sample user model class is included in [App/Models/User.php](App/Models/User.php). You can get the PDO database connection instance like this:

```php
$db = static::getDB();
```

### PDO Database Class


A database class for PHP-MySQL which uses the PDO extension.

#### To use the class
##### 1. Edit the database settings in the settings.ini.php
### Note if PDO is loading slow change localhost to -> 127.0.0.1 !
```
[SQL]
host = 127.0.0.1
user = root
password = 
dbname = yourdatabase
```
##### 2. Import the class
```php
<?php

use Core\Database\Db;
```
##### 3. Create the instance 
```php
<?php

// The instance
$db = new Db();
```
##### 4.  Logs - Modify the read/write rights of the root folder

Everytime an exception is thrown by the database class a log file gets created or modified.
These logs are stored in the logs directory. Which means the database class needs write access for the logs folder.
If the files are on a webserver you'll have to modify the rights of the root folder otherwise you'll get a "Permission denied" error.

The log file is a simple plain text file with the current date('year-month-day') as filename.

## Examples
Below some examples of the basic functions of the database class. I've included a SQL dump so you can easily test the database
class functions. 
#### The persons table 
| id | firstname | lastname | sex | age
|:-----------:|:------------:|:------------:|:------------:|:------------:|
| 1       |        John |     Doe    | M | 19
| 2       |        Bob  |     Black    | M | 41
| 3       |        Zoe  |     Chan    | F | 20
| 4       |        Kona |     Khan    | M | 14
| 5       |        Kader|     Khan    | M | 56

#### Fetching everything from the table
```php
<?php
// Fetch whole table
$persons = $db->query("SELECT * FROM persons");
```
#### Fetching with Bindings (ANTI-SQL-INJECTION):
Binding parameters is the best way to prevent SQL injection. The class prepares your SQL query and binds the parameters
afterwards.

There are three different ways to bind parameters.
```php
<?php
// 1. Read friendly method  
$db->bind("id","1");
$db->bind("firstname","John");
$person   =  $db->query("SELECT * FROM Persons WHERE firstname = :firstname AND id = :id");

// 2. Bind more parameters
$db->bindMore(array("firstname"=>"John","id"=>"1"));
$person   =  $db->query("SELECT * FROM Persons WHERE firstname = :firstname AND id = :id"));

// 3. Or just give the parameters to the method
$person   =  $db->query("SELECT * FROM Persons WHERE firstname = :firstname",array("firstname"=>"John","id"=>"1"));
```

More about SQL injection prevention : http://indieteq.com/index/readmore/how-to-prevent-sql-injection-in-php

#### Fetching Row:
This method always returns only 1 row.
```php
<?php
// Fetch a row
$ages     =  $db->row("SELECT * FROM Persons WHERE  id = :id", array("id"=>"1"));
```
##### Result
| id | firstname | lastname | sex | age
|:-----------:|:------------:|:------------:|:------------:|:------------:|
| 1       |        John |     Doe    | M | 19
#### Fetching Single Value:
This method returns only one single value of a record.
```php
<?php
// Fetch one single value
$db->bind("id","3");
$firstname = $db->single("SELECT firstname FROM Persons WHERE id = :id");
```
##### Result
|firstname
|:------------:
| Zoe
#### Fetching Column:
```php
<?php
// Fetch a column
$names    =  $db->column("SELECT Firstname FROM Persons");
```
##### Result
|firstname | 
|:-----------:
|        John 
|        Bob  
|        Zoe  
|        Kona 
|        Kader
### Delete / Update / Insert
When executing the delete, update, or insert statement by using the query method the affected rows will be returned.
```php
<?php

// Delete
$delete   =  $db->query("DELETE FROM Persons WHERE Id = :id", array("id"=>"1"));

// Update
$update   =  $db->query("UPDATE Persons SET firstname = :f WHERE Id = :id", array("f"=>"Jan","id"=>"32"));

// Insert
$insert   =  $db->query("INSERT INTO Persons(Firstname,Age) VALUES(:f,:age)", array("f"=>"Vivek","age"=>"20"));

// Do something with the data 
if($insert > 0 ) {
  return 'Succesfully created a new person !';
}

```
## Method parameters
Every method which executes a query has the optional parameter called bindings.

The <i>row</i> and the <i>query</i> method have a third optional parameter  which is the fetch style.
The default fetch style is <i>PDO::FETCH_ASSOC</i> which returns an associative array.

Here an example :

```php
<?php
  // Fetch style as third parameter
  $person_num =     $db->row("SELECT * FROM Persons WHERE id = :id", array("id"=>"1"), PDO::FETCH_NUM);

  print_r($person_num);
  // Array ( [0] => 1 [1] => Johny [2] => Doe [3] => M [4] => 19 )
    
```
More info about the PDO fetchstyle : http://php.net/manual/en/pdostatement.fetch.php


###RetardORM

RetardORM is a class which you can use to easily execute basic SQL operations like(insert, update, select, delete) on your database. 
It uses the database class I've created to execute the SQL queries.

Actually it's just a little ORM class.

#### How to use RetardORM
##### 1. First, create a new class. Then require the easyCRUD class.
##### 2. Extend your class to the base class Crud and add the following fields to the class.
##### Example class :
```php
<?php

use Core\Database\RetardORM;
 
class YourClass  Extends RetardORM {
 
  # The table you want to perform the database actions on
  protected $table = 'persons';

  # Primary Key of the table
  protected $pk  = 'id';
  
}
```

##### RetardORM in action.

###### Creating a new person
```php
<?php
// First we"ll have create the instance of the class
$person = new person();
 
// Create new person
$person->Firstname  = "Kona";
$person->Age        = "20";
$person->Sex        = "F";
$created            = $person->Create();
 
//  Or give the bindings to the constructor
$person  = new person(array("Firstname"=>"Kona","age"=>"20","sex"=>"F"));
$created = person->Create();
 
// SQL Equivalent
"INSERT INTO persons (Firstname,Age,Sex) VALUES ('Kona','20','F')"
```
###### Deleting a person
```php
<?php
// Delete person
$person->Id  = "17";
$deleted     = $person->Delete();
 
// Shorthand method, give id as parameter
$deleted     = $person->Delete(17);
 
// SQL Equivalent
"DELETE FROM persons WHERE Id = 17 LIMIT 1"
```
###### Saving person's data
```php
<?php
// Update personal data
$person->Firstname = "John";
$person->Age  = "20";
$person->Sex = "F";
$person->Id  = "4"; 
// Returns affected rows
$saved = $person->Save();
 
//  Or give the bindings to the constructor
$person = new person(array("Firstname"=>"John","age"=>"20","sex"=>"F","Id"=>"4"));
$saved = $person->Save();
 
// SQL Equivalent
"UPDATE persons SET Firstname = 'John',Age = 20, Sex = 'F' WHERE Id= 4"
```
###### Finding a person
```php
<?php
// Find person
$person->Id = "1";
$person->Find();

echo $person->firstname;
// Johny
 
// Shorthand method, give id as parameter
$person->Find(1); 
 
// SQL Equivalent
"SELECT * FROM persons WHERE Id = 1"
```
###### Getting all the persons
```php
<?php
// Finding all person
$persons = $person->all(); 
 
// SQL Equivalent
"SELECT * FROM persons 
```

## Errors

If the `SHOW_ERRORS` configuration setting is set to `true`, full error detail will be shown in the browser if an error or exception occurs. If it's set to `false`, a generic message will be shown using the [App/Views/404.html](App/Views/404.html) or [App/Views/500.html](App/Views/500.html) views, depending on the error.

## Web server configuration

Pretty URLs are enabled using web server rewrite rules. An [.htaccess](public/.htaccess) file is included in the `public` folder. Equivalent nginx configuration is in the [nginx-configuration.txt](nginx-configuration.txt) file.

