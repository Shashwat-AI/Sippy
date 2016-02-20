# Sippy

An easy to use PHP Framework for building websites and web apps.

# Documentation

Sippy’s architecture bares striking resemblance to  Codeigniter and another small framework called PIP. However remember that Sippy is a lightweight framework and does not include a lot of the functionality that comes with Codeigniter.  However, with the help of other developers, maybe we can add a lot of functionality through plug-ins and libraries.

# Model-View-Controller

Sippy is based on the Model-View-Controller development pattern.

# Folder Structure

Your application specific files go in the "application" folder (you don't need to touch the system folder). Inside the application folder there are folders for all of the specific application entities:

config
controllers
helpers
logs
models
plugins
views

When Sippy loads files it assumes they are in the corresponding folders. So make sure you place your files in the correct folders.

We encourage you to use the "assets" folder in the root to store you static resource files (CSS, JS etc) however you can put them anywhere. You can also use the `site_url()` function to help include files in your HTML. For example:
```html
<link rel="stylesheet" href="<?php echo site_url(); ?>assets/css/style.css" type="text/css" media="screen" />
```
# Naming Conventions

All classes in Sippy use PascalCase naming. The associated file names must be the same except all lower case. So for example the class MainClass would have the filename mainclass.php. Underscores in classes must be included in file names as well.

# URL Structure

By default, URLs in Sippy are designed to be search-engine and human friendly. Rather than using the standard "query string" approach to URLs that is synonymous with dynamic systems, Sippy uses a segment-based approach:

`example.com/class/function/param`
By default index.php is hidden in the URL. This is done using the .htaccess file in the root directory.

## Controllers

Controllers are the driving force of a Sippy application. As you can see from the URL structure, segments of the URL are mapped to a class and function. These classes are controllers stored in the "application/controller" directory. So for example the URL...

example.com/main/login
...would map to the following Controller with the filename main.php:
```php
<?php

class Main extends Controller {

    function index()
    {
        // This is the default function (i.e. no function is set in the URL)
    }
    
    function login()
    {
        echo 'Hello World!';
    }
}
```

...and the output would be "Hello World!".

The default controller and error controller can be set in application/config/config.php

Note that if you need to declare a constructor you must also call the parent constructor like:
```php
<?php

class Example extends Controller {

    public function __construct()
    {
        parent::__construct();
        // Your own constructor code
    }

}

?>
```
There are several helper functions that can also be used in controllers. Most of these functions take the parameter $name of the corresponding class:

* Model($name) - Load a model
* View($name) - Load a view
* Plugin($name) - Load a plugin
* Helper($name) - Load a helper
* redirect($location) - Redirect to a page without having to include the base URL. E.g:
* $this->redirect('some_class/some_function');

Log Errors and Debugging information
* In your config file, you must enable logging, set to TRUE.

Then you can add this code to your controller.

* Errors: `$this->logger->log_message(‘error’,’Your log message‘);`
* Debugging: `$this->logger->log_message(‘debug’,’Your log message‘);` 


## Views

In Sippy a view is simply a web page. They can contain everything a normal web page would include. Views are almost always loaded by Controllers. So for example if you had a view called main_view.php that contained the following HTML:
```php
<html>
<head>
<title>My Site</title>
</head>
<body>
    <h1>Welcome to my Site!</h1>
</body>
</html>
... you would load it in a controller by doing the following:
// Controller file...
<?php

class Main extends Sippy_controller {
	
    function index()
    {
        $template = $this->View('main_view');
        $template->render();
    }
    
}
```

The View class has a helper function called `set($key, $val)` that allows you to pass variables from the Controller to the View.
```php
$template = $this->View('main_view');
$template->set('someval', 17);
$template->render();
```
...then in the view you could do:

`<?php echo $someval; ?>`

...and the output would be 17. Any kind of PHP variable can be passed to a view in this way: arrays, etc.

## Models

In Sippy models are classes that deal with data (usually from a database). For example:

```php
<?php

class Example_model extends Sippy_model {
	
    public function getSomething($id)
    {
        $id = $this->escapeString($id);
        $result = $this->query('SELECT * FROM something WHERE id="'. $id .'"');
        return $result;
    }

}

?>
```
...then in a controller you would do:

```php
function index()
{
    $example = $this->Model('Example_model');
    $something = $example->getSomething($id);
    
    $template = $this->View('main_view');
    $template->set('someval', $something);
    $template->render();
}
```
Now the results of your database query would be available in your view in $someval. Connecting to the MySQL Database can be done in your `config/config.php` file:
```php
$config['db_host'] = ''; // Database host (e.g. localhost)
$config['db_name'] = ''; // Database name
$config['db_username'] = ''; // Database username
$config['db_password'] = ''; // Database password
```
There are several helper functions that can also be used in models:

* `query($query)` - Returns an array of results from a query
* `execute($query)` - Returns the direct result from a query
* `escapeString($string)` - Escape strings before using them in queries

# Helpers & Plug-ins

There are two types of additional resources you can use in Sippy.

Helpers are classes which you can use that don't fall under the category of "controllers". These will usually be classes that provide extra functionality that you can use in your controllers. Sippy comes with two helper classes (Session_helper and Url_helper) which are examples of how to use helpers.

Plugins are literally any PHP files and can provide any functionality you want. By loading a plugin you are simply including the PHP file from the "plugins" folder. This can be useful if you want to use third party libraries in your Sippy application.

# Extending Sippy

To extend Sippy, there’s an option to add “hooks”.  Hooks allow you to extend the whole framework and/or add functionality.  To activate ‘hooks’ go into your config.php file and enable it.  Then in the config/hooks.php file add your hook function.  Hooks are global and there are three of them; `before_system` (loads before system calls), `before_controller` (loads before controllers), and `after_controller` (loads after controllers).  Here’s an example...
```php
// in hooks.php file
$hook['before_system'] = function() {
	echo "hello world";
};
```
This hook will globally load this function that says “hello world”.  Since you can only load a hook one time, if you have more than one function to call, you can stack them like so...
```php
//... your functions ... function func1() {...do stuff...}
$hook['before_system'] = function() {
	func1();
	func2();
	func3();
};
```
