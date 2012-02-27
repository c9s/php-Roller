Roller
======

PHP Roller is a simple/fast router for PHP5.3.

Roller API is really simple, eazy to use.

<img src="https://github.com/c9s/Roller/raw/master/misc/img1.png" width="500">

INSTALL
-------

Clone this repository, pick up a PSR-0 classloader, and add `src/` to
class path.

To use console dumper, you will need ezc/ConsoleTools, please use PEAR
installer to install:

    sudo pear channel-discover components.ez.no
    sudo pear install -a ezc/ConsoleTools

SYNOPSIS
--------

Initialize a router:

    $router = new Roller\Router;

Add a new route with simple callback

    $router->add( '/path/to/blog'  , function() { 
        return 'Blog';
    });

Add a new route with class callback

    $router->add( '/path/to/blog'  , array('Controller','method') );

Add a new route with class/action callback string:

    $router->add( '/path/to/blog'  , 'Controller:methodName' );

To add a new route with requirement:

    $router->add( '/path/to/:year' , array('Callback','method') , array( 
        'year' => '\d+',
    ));

To add a new route with requirement and closure:

    $router->add( '/path/to/:year' , function($year) { 
        return $year;
    },array( 
        'year' => '\d+',
    ));

META ATTRIBUTE
--------------
Meta attributes are started without ':' prefix. currently, Roller supports: 
`method`, `default`, `requirement`, `post`, `get` attributes.

To add a new route with requirement and default value:

    $router->add( '/path/to/:year' , array('Callback','method') , array( 
        'year' => '\d+',
        ':default' => array(
            'year' => 2000,
        ),
    ));

To add a new route with request method (like POST method):

    $router->add( '/path/to/:year' , array('Callback','method') , array( 
        'year' => '\d+',

        ':method' => 'post,
        ':default' => array(
            'year' => 2000,
        ),
    ));

Dispatch
--------

To dispatch path:

    $r = $router->dispatch( $_SERVER['PATH_INFO'] );

To evalulate response content:

    if( $r !== false )
        echo $r();
    else
        die('page not found.');


Mount paths
-----------

To mount route set:

    $routes = new Roller\RouteSet;
    $routes->add( '/path/to/:year' , array( 'Callback', 'method' ) );

    $routes = new Roller\RouteSet;
    $routes->mount( '/root' , $routes );

    $router = new Roller\Router( $routes );

Cache
-----

To enable apc cache:

    $router = new Roller\Router( null , array( 
        'cache_id' => '_router_testing_'
    ));
    
To enable file cache:

    $router = new Roller\Router( null , array( 
        'cache_id' => '_router_testing_',
        'cache_dir' => 'tests/cache',
    ));


RESTful
-------

Roller Router has a built-in RESTful route generator, it's pretty easy to
define a bunch of RESTful routes, Roller Router also provides a simple RESTful
route generator, which is pretty easy to customize your own RESTful routes.

First, initalize a RESTful plugin object:
	
        $router = new Roller\Router;
        $restful = new Roller\Plugin\RESTful(array( 
				'prefix' => '/restful' 
		));

Add RESTful plugin to your router manager:

        $router->addPlugin($restful);

Register your resource id to your resource handler:

        $restful->registerResource( 'blog' , 'BlogResourceHandler' );

Define your resource handler, here is a simple example that defines how CRUD works:

```php
	<?php
	use Roller\Plugin\RESTful\ResourceHandler;

	class BlogResourceHandler extends ResourceHandler
	{
		public function create()    { return array( 'id' => 1 );
		}

		public function update($id) { return array( 'id' => 1 );
		}

		// delete a record.
		public function delete($id) { return array( 'id' => 'delete' ); }

		// load one record
		public function load($id)   { return array( 'id' => $id , 'title' => 'title' ); }

		// find records
		public function find()      { 
			return array( 
				array( 'id' => 0 ),
				array( 'id' => 1 ),
				array( 'id' => 2 ),
			);
		}
	}
```

Before you dispatching URLs, router object will calls `expand` method from `ResourceHandler` class, which
generates RESTful routes into the routeset of router object. And below is the generated URLs:

	GET /restful/blog        - get blog list
	GET /restful/blog/:id    - get one blog record
	POST /restful/blog       - create one blog record
	PUT /restful/blog/:id    - update one blog record
	DELETE /restful/blog/:id - delete one blog record

You can override the `expand` method to define your own style RESTful URLs.

You should be able to dispatch RESTful urls:

        $_SERVER['REQUEST_METHOD'] = 'get';
        $r = $router->dispatch('/restful/blog/1');

		// returns {"success":true,"data":{"id":"1","title":"title"},"message":"Record 1 loaded."}
		$r();   

        $_SRVER['REQUEST_METHOD'] = 'get';
        $r = $router->dispatch('/restful/blog');

        // {"success":true,"data":[{"id":0},{"id":1},{"id":2}],"message":"Record find success."}
		$r();


## Customize Resource Handler

Here is how RESTful route generator works:

```php
<?php

	public function expand($routes, $h, $r)
	{
		$routes->add( "/$r(\.:format)" , array($h,'handleFind'), 
			array( 
				':get' => true , 
				':default' => array( 'format' => 'json' ) 
			));

		$routes->add( '/' . $r . '(\.:format)' , array($h,'handleCreate'), 
			array( 
				':post' => true, 
				':default' => array( 'format' => 'json' ) 
			));

		$routes->add( '/' . $r . '/:id(\.:format)' , array($h,'handleLoad'),
			array( 
				':get' => true, 
				':default' => array( 'format' => 'json' )
			));

		$routes->add( '/' . $r . '/:id(\.:format)' , array($h,'handleUpdate'),
			array( 
				':put' => true, 
				':default' => array( 'format' => 'json' ) 
			));

		$routes->add( '/' . $r . '/:id(\.:format)' , array($h,'handleDelete'),
			array( 
				':delete' => true, 
				':default' => array( 'format' => 'json' ) 
			));
	}
```

To define your own RESTful Resource Handler (Generator), you can simply inherit class from
`Roller\Plugin\RESTful\ResourceHandler`:

```php
<?php
use Roller\Plugin\RESTful\ResourceHandler;

class YourResourceHandler extends ResourceHandler {

	// define your own expand method
	function expand( $routes , $handlerClass, $resourceId ) {

	}

}
```


Hacking
-------
Get Onion and install it:

    $ curl -s http://install.onionphp.org/ | bash

Run onion to install depedencies:

    $ onion bundle

Now you should be able to run phpunit =)

    $ phpunit tests

