Routing
#######

.. php:namespace:: Cake\Routing

.. php:class:: Router

Routing provides you tools that map URLs to controller actions. By defining
routes, you can separate how your application is implemented from how its URL's
are structured.

Routing in CakePHP also encompasses the idea of reverse routing,
where an array of parameters can be transformed into a URL string.
By using reverse routing, you can more easily re-factor your application's
URL structure without having to update all your code.

.. index:: routes.php

Quick Tour
==========

This section will teach you by example the most common uses of the CakePHP
Router. Typically you want to display something as a landing page, so you add
this to your **routes.php** file::

    use Cake\Routing\Router;

    Router::connect('/', ['controller' => 'Articles', 'action' => 'index']);

This will execute the index method in the ``ArticlesController`` when the homepage
of your site is visited. Sometimes you need dynamic routes that will accept
multiple parameters, this would be the case, for example of a route for viewing
an article's content::

    Router::connect('/articles/*', ['controller' => 'Articles', 'action' => 'view']);

The above route will accept any url looking like ``/articles/15`` and invoke the
method ``view(15)`` in the ``ArticlesController``. This will not, though,
prevent people from trying to access URLs looking like ``/articles/foobar``. If
you wish, you can restring some parameters to conform to a regular expression::

    Router::connect(
        '/articles/:id',
        ['controller' => 'Articles', 'action' => 'view'],
        ['id' => '\d+', 'pass' => ['id']]
    );

The previous example changed the star matcher by a new placeholder ``:id``.
Using placeholders allows us to validate parts of the url, in this case we used
the ``\d+`` regular expression so that only digits are matched. Finally, we told
the Router to treat the ``id`` placeholder as a function argument to the
``view()`` function by specifying the ``pass`` option. More on using this
options later.

The CakePHP Router can also match routes in reverse. That means that from an
array containing similar parameters, it is capable of generation a URL string::

    use Cake\Routing\Router;

    echo Router::url(['controller' => 'Articles', 'action' => 'view', 'id' => 15]);
    // Will output
    /articles/15

Routes can also be labelled with a unique name, this allows you to quickly
reference them when building links instead of specifying each of the routing
parameters::

    use Cake\Routing\Router;

    Router::connect(
        '/login',
        ['controller' => 'Users', 'action' => 'login'],
        ['_name' => 'login']
    );

    echo Router::url(['_name' => 'login']);
    // Will output
    /login

To help keep your routing code DRY, the Router has the concept of 'scopes'.
A scope defines a common path segment, and optionally route defaults. Any routes
connected inside a scope will inherit the path/defaults from their wrapping
scopes::

    Router::scope('/blog', ['plugin' => 'Blog'], function ($routes) {
        $routes->connect('/', ['controller' => 'Articles']);
    });

The above route would match ``/blog/`` and send it to
``Blog\Controller\ArticlesController::index()``.

The application skeleton comes with a few routes to get you started. Once you've
added your own routes, you can remove the default routes if you don't need them.

.. index:: :controller, :action, :plugin
.. index:: greedy star, trailing star
.. _connecting-routes:
.. _routes-configuration:

Connecting Routes
=================

.. php:staticmethod:: connect($route, $defaults = [], $options = [])

To keep your code :term:`DRY` you should use 'routing scopes'. Routing
scopes not only let you keep your code DRY, they also help Router optimize its
operation. As seen above you can also use ``Router::connect()`` to connect
routes. This method defaults to the ``/`` scope. To create a scope and connect
some routes we'll use the ``scope()`` method::

    // In config/routes.php
    Router::scope('/', function ($routes) {
        $routes->fallbacks('InflectedRoute');
    });

The ``connect()`` method takes up to three parameters: the URL template you wish
to match, the default values for your route elements, and the options for the
route. Options frequently include regular expression rules to help the router
match elements in the URL.

The basic format for a route definition is::

    $routes->connect(
        'URL template',
        ['default' => 'defaultValue'],
        ['option' => 'matchingRegex']
    );

The first parameter is used to tell the router what sort of URL you're trying to
control. The URL is a normal slash delimited string, but can also contain
a wildcard (\*) or :ref:`route-elements`.  Using a wildcard tells the router
that you are willing to accept any additional arguments supplied. Routes without
a \* only match the exact template pattern supplied.

Once you've specified a URL, you use the last two parameters of ``connect()`` to
tell CakePHP what to do with a request once it has been matched. The second
parameter is an associative array. The keys of the array should be named after
the route elements the URL template represents. The values in the array are the
default values for those keys.  Let's look at some basic examples before we
start using the third parameter of connect()::

    $routes->connect(
        '/pages/*',
        ['controller' => 'Pages', 'action' => 'display']
    );

This route is found in the routes.php file distributed with CakePHP.  It matches
any URL starting with ``/pages/`` and hands it to the ``display()`` action of
the ``PagesController``. A request to ``/pages/products`` would be mapped to
``PagesController->display('products')``.

In addition to the greedy star ``/*`` there is also the ``/**`` trailing star
syntax. Using a trailing double star, will capture the remainder of a URL as a
single passed argument. This is useful when you want to use an argument that
included a ``/`` in it::

    $routes->connect(
        '/pages/**',
        ['controller' => 'Pages', 'action' => 'show']
    );

The incoming URL of ``/pages/the-example-/-and-proof`` would result in a single
passed argument of ``the-example-/-and-proof``.

You can use the second parameter of ``connect()`` to provide any routing
parameters that are composed of the default values of the route::

    $routes->connect(
        '/government',
        ['controller' => 'Pages', 'action' => 'display', 5]
    );

This example shows how you can use the second parameter of ``connect()`` to
define default parameters. If you built a site that features products for
different categories of customers, you might consider creating a route. This
allows you link to ``/government`` rather than ``/pages/display/5``.

Another common use for the Router is to define an "alias" for a
controller. Let's say that instead of accessing our regular URL at
``/users/some_action/5``, we'd like to be able to access it by
``/cooks/some_action/5``. The following route easily takes care of
that::

    $routes->connect(
        '/cooks/:action/*', ['controller' => 'Users']
    );

This is telling the Router that any URL beginning with ``/cooks/`` should be
sent to the users controller. The action called will depend on the value of the
``:action`` parameter. By using :ref:`route-elements`, you can create variable
routes, that accept user input or variables. The above route also uses the
greedy star.  The greedy star indicates to ``Router`` that this route
should accept any additional positional arguments given. These arguments will be
made available in the :ref:`passed-arguments` array.

When generating URLs, routes are used too. Using
``['controller' => 'Users', 'action' => 'some_action', 5]`` as
a url will output ``/cooks/some_action/5`` if the above route is the
first match found.

.. _route-elements:

Route Elements
--------------

You can specify your own route elements and doing so gives you the
power to define places in the URL where parameters for controller
actions should lie. When a request is made, the values for these
route elements are found in ``$this->request->params`` in the controller.
When you define a custom route element, you can optionally specify a regular
expression - this tells CakePHP how to know if the URL is correctly formed or
not. If you choose to not provide a regular expression, any non ``/`` character will be
treated as part of the parameter::

    $routes->connect(
        '/:controller/:id',
        ['action' => 'view'],
        ['id' => '[0-9]+']
    );

The above example illustrates how to create a quick way to view
models from any controller by crafting a URL that looks like
``/controllername/:id``. The URL provided to ``connect()`` specifies two
route elements: ``:controller`` and ``:id``. The ``:controller`` element
is a CakePHP default route element, so the router knows how to match and
identify controller names in URLs. The ``:id`` element is a custom
route element, and must be further clarified by specifying a
matching regular expression in the third parameter of connect().

CakePHP does not automatically produce lowercased urls when using the
``:controller`` parameter. If you need this, the above example could be
rewritten like so::

    $routes->connect(
        '/:controller/:id',
        ['action' => 'view'],
        ['id' => '[0-9]+', 'routeClass' => 'InflectedRoute']
    );

The special ``InflectedRoute`` class will make sure that the ``:controller`` and
``:plugin`` parameters are correctly lowercased.

.. note::

    Patterns used for route elements must not contain any capturing
    groups. If they do, Router will not function correctly.

Once this route has been defined, requesting ``/apples/5`` would call the view()
method of the ApplesController. Inside the view() method, you would need to
access the passed ID at ``$this->request->params['id']``.

If you have a single controller in your application and you do not want the
controller name to appear in the URL, you can map all URLs to actions in your
controller. For example, to map all URLs to actions of the ``home`` controller,
e.g have URLs like ``/demo`` instead of ``/home/demo``, you can do the
following::

    $routes->connect('/:action', ['controller' => 'Home']);

If you would like to provide a case insensitive URL, you can use regular
expression inline modifiers::

    $routes->connect(
        '/:userShortcut',
        ['controller' => 'Teachers', 'action' => 'profile', 1],
        ['userShortcut' => '(?i:principal)']
    );

One more example, and you'll be a routing pro::

    $routes->connect(
        '/:controller/:year/:month/:day',
        ['action' => 'index'],
        [
            'year' => '[12][0-9]{3}',
            'month' => '0[1-9]|1[012]',
            'day' => '0[1-9]|[12][0-9]|3[01]'
        ]
    );

This is rather involved, but shows how powerful routes can be The URL supplied
has four route elements. The first is familiar to us: it's a default route
element that tells CakePHP to expect a controller name.

Next, we specify some default values. Regardless of the controller,
we want the index() action to be called.

Finally, we specify some regular expressions that will match years,
months and days in numerical form. Note that parenthesis (grouping)
are not supported in the regular expressions. You can still specify
alternates, as above, but not grouped with parenthesis.

Once defined, this route will match ``/articles/2007/02/01``,
``/articles/2004/11/16``, handing the requests to
the index() actions of their respective controllers, with the date
parameters in ``$this->request->params``.

There are several route elements that have special meaning in
CakePHP, and should not be used unless you want the special meaning

* ``controller`` Used to name the controller for a route.
* ``action`` Used to name the controller action for a route.
* ``plugin`` Used to name the plugin a controller is located in.
* ``prefix`` Used for :ref:`prefix-routing`
* ``_ext`` Used for :ref:`file-extensions` routing.
* ``_base`` Set to ``false`` to remove the base path from the generated URL. If your application
  is not in the root directory, this can be used to generate URLs that are 'cake relative'.
  cake relative URLs are required when using requestAction.
* ``_scheme``  Set to create links on different schemes like `webcal` or `ftp`. Defaults
  to the current scheme.
* ``_host`` Set the host to use for the link.  Defaults to the current host.
* ``_port`` Set the port if you need to create links on non-standard ports.
* ``_full``  If ``true`` the `FULL_BASE_URL` constant will be prepended to generated URLs.
* ``#`` Allows you to set URL hash fragments.
* ``_ssl`` Set to ``true`` to convert the generated URL to https or ``false``
  to force http.
* ``_method`` Define the HTTP verb/method to use. Useful when working with
  :ref:`resource-routes`.
* ``_name`` Name of route. If you have setup named routes, you can use this key
  to specify it.

Passing Parameters to Action
----------------------------

When connecting routes using :ref:`route-elements` you may want
to have routed elements be passed arguments instead. By using the 3rd
argument of :php:meth:`Cake\\Routing\\Router::connect()` you can define which route
elements should also be made available as passed arguments::

    // SomeController.php
    public function view($articleId = null, $slug = null)
    {
        // Some code here...
    }

    // routes.php
    Router::connect(
        '/blog/:id-:slug', // E.g. /blog/3-CakePHP_Rocks
        ['controller' => 'Blog', 'action' => 'view'],
        [
            // order matters since this will simply map ":id" to $articleId in your action
            'pass' => ['id', 'slug'],
            'id' => '[0-9]+'
        ]
    );

And now, thanks to the reverse routing capabilities, you can pass
in the URL array like below and CakePHP will know how to form the URL
as defined in the routes::

    // view.ctp
    // This will return a link to /blog/3-CakePHP_Rocks
    echo $this->Html->link('CakePHP Rocks', [
        'controller' => 'Blog',
        'action' => 'view',
        'id' => 3,
        'slug' => 'CakePHP_Rocks'
    ]);

    // You can also used numerically indexed parameters.
    echo $this->Html->link('CakePHP Rocks', [
        'controller' => 'Blog',
        'action' => 'view',
        3,
        'CakePHP_Rocks'
    ]);

.. _named-routes:

Using Named Routes
------------------

Sometimes you'll find typing out all the URL parameters for a route too verbose,
or you'd like to take advantage of the performance improvements that named routes
have. When connecting routes you can specifiy a ``_name`` option, this option
can be used in reverse routing to identify the route you want to use::

    // Connect a route with a name.
    $routes->connect(
        '/login',
        ['controller' => 'Users', 'action' => 'login'],
        ['_name' => 'login']
    );

    // Generate a URL using a named route.
    $url = Router::url(['_name' => 'login']);

    // Generate a URL using a named route,
    // with some query string args.
    $url = Router::url(['_name' => 'login', 'username' => 'jimmy']);

If your route template contains any route elements like ``:controller`` you'll
need to supply those as part of the options to ``Router::url()``.

.. index:: admin routing, prefix routing
.. _prefix-routing:

Prefix Routing
--------------

.. php:staticmethod:: prefix($name, $callback)

Many applications require an administration section where
privileged users can make changes. This is often done through a
special URL such as ``/admin/users/edit/5``. In CakePHP, prefix routing
can be enabled by using the ``prefix`` scope method::

    Router::prefix('admin', function ($routes) {
        // All routes here will be prefixed with `/admin`
        // And have the prefix => admin route element added.
        $routes->fallbacks('InflectedRoute');
    });

Prefixes are mapped to sub-namespaces in your application's ``Controller``
namespace. By having prefixes as separate controllers you can create smaller and
simpler controllers. Behavior that is common to the prefixed and non-prefixed
controllers can be encapsulated using inheritance,
:doc:`/controllers/components`, or traits.  Using our users example, accessing
the URL ``/admin/users/edit/5`` would call the ``edit`` method of our
``src/Controller/Admin/UsersController.php`` passing 5 as the first parameter. The
view file used would be ``src/Template/Admin/Users/edit.ctp``

You can map the URL /admin to your ``index`` action of pages controller using
following route::

    Router::prefix('admin', function ($routes) {
        // Because you are in the admin scope,
        // you do not need to include the /admin prefix
        // or the admin route element.
        $routes->connect('/', ['controller' => 'Pages', 'action' => 'index']);
    });

You can define prefixes inside plugin scopes as well::

    Router::plugin('DebugKit', function ($routes) {
        $routes->prefix('admin', function ($routes) {
            $routes->connect('/:controller');
        });
    });

The above would create a route template like ``/debug_kit/admin/:controller``.
The connected route would have the ``plugin`` and ``prefix`` route elements set.

When defining prefixes, you can nest multiple prefixes if necessary::

    Router::prefix('manager', function ($routes) {
        $routes->prefix('admin', function ($routes) {
            $routes->connect('/:controller');
        });
    });

The above would create a route template like ``/manager/admin/:controller``.
The connected route would have the ``prefix`` route element set to
``manager/admin``.

The current prefix will be available from the controller methods through
``$this->request->params['prefix']``

When using prefix routes it's important to set the prefix option. Here's how to
build this link using the HTML helper::

    // Go into a prefixed route.
    echo $this->Html->link(
        'Manage articles',
        ['prefix' => 'manager', 'controller' => 'Articles', 'action' => 'add']
    );

    // Leave a prefix
    echo $this->Html->link(
        'View Post',
        ['prefix' => false, 'controller' => 'Articles', 'action' => 'view', 5]
    );

.. note::

    You should connect prefix routes *before* you connect fallback routes.

.. index:: plugin routing

Plugin Routing
--------------

.. php:staticmethod:: plugin($name, $options = [], $callback)

Plugin routes are most easily created using the ``plugin()`` method. This method
creates a new routing scope for the plugin's routes::

    Router::plugin('DebugKit', function ($routes) {
        // Routes connected here are prefixed with '/debug_kit' and
        // have the plugin route element set to 'DebugKit'.
        $routes->connect('/:controller');
    });

When creating plugin scopes, you can customize the path element used with the
``path`` option::

    Router::plugin('DebugKit', ['path' => '/debugger'], function ($routes) {
        // Routes connected here are prefixed with '/debugger' and
        // have the plugin route element set to 'DebugKit'.
        $routes->connect('/:controller');
    });

When using scopes you can nest plugin scopes within prefix scopes::

    Router::prefix('admin', function ($routes) {
        $routes->plugin('DebugKit', function ($routes) {
            $routes->connect('/:controller');
        });
    });

The above would create a route that looks like ``/admin/debug_kit/:controller``.
It would have the ``prefix``, and ``plugin`` route elements set.

You can create links that point to a plugin, by adding the plugin key to your
URL array::

    echo $this->Html->link(
        'New todo',
        ['plugin' => 'Todo', 'controller' => 'TodoItems', 'action' => 'create']
    );

Conversely if the active request is a plugin request and you want to create
a link that has no plugin you can do the following::

    echo $this->Html->link(
        'New todo',
        ['plugin' => null, 'controller' => 'Users', 'action' => 'profile']
    );

By setting ``plugin => null`` you tell the Router that you want to
create a link that is not part of a plugin.

SEO-Friendly Routing
--------------------

Some developers prefer to use dashes in URLs, as it's perceived to give
better search engine rankings. The ``DashedRoute`` class can be used in your
application with the ability to route plugin, controller, and camelized action
names to a dashed URL.

For example, if we had a ``ToDo`` plugin, with a ``TodoItems`` controller, and a
``showItems`` action, it could be accessed at ``/to-do/todo-items/show-items``
with the following router connection::

    Router::plugin('ToDo', ['path' => 'to-do'], function ($routes) {
        $routes->fallbacks('DashedRoute');
    });

.. index:: file extensions
.. _file-extensions:

Routing File Extensions
-----------------------

.. php:staticmethod:: extensions(string|array|null $extensions, $merge = true)

To handle different file extensions with your routes, you need one
extra line in your routes config file::

    Router::extensions(['html', 'rss']);

This will enable the named extensions for all routes connected **after** this
method call. Any routes connected prior to it will not inherit the extensions.
By default the extensions you passed will be merged with existing list of extensions.
You can pass ``false`` for the second argument to override existing list.
Calling the method without arguments will return existing list of extensions.
You can set extensions per scope as well::

    Router::scope('/api', function ($routes) {
        $routes->extensions(['json', 'xml']);
    });

.. note::

    Setting the extensions should be the first thing you do in a scope, as the
    extensions will only be applied to routes connected **after** the extensions
    are set.

By using extensions, you tell the router to remove any matching file extensions,
and then parse what remains. If you want to create a URL such as
/page/title-of-page.html you would create your route using::

    Router::scope('/api', function ($routes) {
        $routes->extensions(['json', 'xml']);
        $routes->connect(
            '/page/:title',
            ['controller' => 'Pages', 'action' => 'view'],
            [
                'pass' => ['title']
            ]
        );
    });

Then to create links which map back to the routes simply use::

    $this->Html->link(
        'Link title',
        ['controller' => 'Pages', 'action' => 'view', 'title' => 'super-article', '_ext' => 'html']
    );

File extensions are used by :doc:`/controllers/components/request-handling`
to do automatic view switching based on content types.

.. _resource-routes:

Creating RESTful Routes
=======================

.. php:staticmethod:: mapResources($controller, $options)

Router makes it easy to generate RESTful routes for your controllers.
If we wanted to allow REST access to a recipe database, we'd do
something like this::

    // In config/routes.php...

    Router::scope('/', function ($routes) {
        $routes->extensions(['json']);
        $routes->resources('Recipes');
    });

The first line sets up a number of default routes for easy REST
access where method specifies the desired result format (e.g. xml,
json, rss). These routes are HTTP Request Method sensitive.

=========== ===================== ==============================
HTTP format URL.format            Controller action invoked
=========== ===================== ==============================
GET         /recipes.format       RecipesController::index()
----------- --------------------- ------------------------------
GET         /recipes/123.format   RecipesController::view(123)
----------- --------------------- ------------------------------
POST        /recipes.format       RecipesController::add()
----------- --------------------- ------------------------------
PUT         /recipes/123.format   RecipesController::edit(123)
----------- --------------------- ------------------------------
PATCH       /recipes/123.format   RecipesController::edit(123)
----------- --------------------- ------------------------------
DELETE      /recipes/123.format   RecipesController::delete(123)
=========== ===================== ==============================

CakePHP's Router class uses a number of different indicators to
detect the HTTP method being used. Here they are in order of
preference:

#. The \_method POST variable
#. The X\_HTTP\_METHOD\_OVERRIDE
#. The REQUEST\_METHOD header

The \_method POST variable is helpful in using a browser as a
REST client (or anything else that can do POST easily). Just set
the value of \_method to the name of the HTTP request method you
wish to emulate.

Creating Nested Resources
-------------------------

Once you have connected resources in a scope, you can connect routes for
sub-resources as well. Sub-resource routes will be prepended by the original
resource name and a id parameter. For example::

    Router::scope('/api', function ($routes) {
        $routes->resources('Articles', function ($routes) {
            $routes->resources('Comments');
        });
    });

Will generate resource routes for both ``articles`` and ``comments``. The
comments routes will look like::

    /api/articles/:article_id/comments
    /api/articles/:article_id/comments/:id

You can get the ``article_id`` in ``CommentsController`` by::

    $this->request->params['article_id']

.. note::

    While you can nest resources as deeply as you require, it is not recommended to
    nest more than 2 resources together.

Limiting the Routes Created
---------------------------

By default CakePHP will connect 6 routes for each resource. If you'd like to
only connect specific resource routes you can use the ``only`` option::

    $routes->resources('Articles', [
        'only' => ['index', 'view']
    ]);

Would create read only resource routes. The route names are ``create``,
``update``, ``view``, ``index``, and ``delete``.

Changing the Controller Actions Used
------------------------------------

You may need to change the controller action names that are used when connecting
routes. For example, if your ``edit`` action is called ``update`` you can use
the ``actions`` key to rename the actions used::

    $routes->resources('Articles', [
        'actions' => ['update' => 'update', 'add' => 'create']
    ]);

The above would use ``update`` for the update action, and ``create`` instead of
``add``.

.. _custom-rest-routing:

Custom Route Classes for Resource Routes
----------------------------------------

You can provide ``connectOptions`` key in the ``$options`` array for
``resources()`` to provide custom setting used by ``connect()``::

    Router::scope('/', function ($routes) {
        $routes->resources('books', [
            'connectOptions' => [
                'routeClass' => 'ApiRoute',
            ]
        ];
    });

.. index:: passed arguments
.. _passed-arguments:

Passed Arguments
================

Passed arguments are additional arguments or path segments that are
used when making a request. They are often used to pass parameters
to your controller methods. ::

    http://localhost/calendars/view/recent/mark

In the above example, both ``recent`` and ``mark`` are passed arguments to
``CalendarsController::view()``. Passed arguments are given to your controllers
in three ways. First as arguments to the action method called, and secondly they
are available in ``$this->request->params['pass']`` as a numerically indexed
array. When using custom routes you can force particular parameters to go into
the passed arguments as well.

If you were to visit the previously mentioned URL, and you
had a controller action that looked like::

    class CalendarsController extends AppController
    {
        public function view($arg1, $arg2)
        {
            debug(func_get_args());
        }
    }

You would get the following output::

    Array
    (
        [0] => recent
        [1] => mark
    )

This same data is also available at ``$this->request->params['pass']``
and ``$this->passedArgs`` in your controllers, views, and helpers.
The values in the pass array are numerically indexed based on the
order they appear in the called URL::

    debug($this->request->params['pass']);

Either of the above would output::

    Array
    (
        [0] => recent
        [1] => mark
    )

When generating URLs, using a :term:`routing array` you add passed
arguments as values without string keys in the array::

    ['controller' => 'Articles', 'action' => 'view', 5]

Since ``5`` has a numeric key, it is treated as a passed argument.

Generating URLs
===============

.. php:staticmethod:: url($url = null, $full = false)

Generating URLs or Reverse routing is a feature in CakePHP that is used to allow you to
easily change your URL structure without having to modify all your code.
By using :term:`routing arrays <routing array>` to define your URLs, you can
later configure routes and the generated URLs will automatically update.

If you create URLs using strings like::

    $this->Html->link('View', '/articles/view/' . $id);

And then later decide that ``/articles`` should really be called
'articles' instead, you would have to go through your entire
application renaming URLs. However, if you defined your link like::

    $this->Html->link(
        'View',
        ['controller' => 'Articles', 'action' => 'view', $id]
    );

Then when you decided to change your URLs, you could do so by defining a
route. This would change both the incoming URL mapping, as well as the
generated URLs.

When using array URLs, you can define both query string parameters and
document fragments using special keys::

    Router::url([
        'controller' => 'Articles',
        'action' => 'index',
        '?' => ['page' => 1],
        '#' => 'top'
    ]);

    // Will generate a URL like.
    /articles/index?page=1#top

Router will also convert any unknown parameters in a routing array to
querystring parameters.  The ``?`` is offered for backwards compatibility with
older versions of CakePHP.

You can also use any of the special route elements when generating URLs:

* ``_ext`` Used for :ref:`file-extensions` routing.
* ``_base`` Set to ``false`` to remove the base path from the generated URL. If your application
  is not in the root directory, this can be used to generate URLs that are 'cake relative'.
  cake relative URLs are required when using requestAction.
* ``_scheme``  Set to create links on different schemes like `webcal` or `ftp`. Defaults
  to the current scheme.
* ``_host`` Set the host to use for the link.  Defaults to the current host.
* ``_port`` Set the port if you need to create links on non-standard ports.
* ``_full``  If ``true`` the `FULL_BASE_URL` constant will be prepended to generated URLs.
* ``_ssl`` Set to ``true`` to convert the generated URL to https or ``false``
  to force http.
* ``_name`` Name of route. If you have setup named routes, you can use this key
  to specify it.

.. _redirect-routing:

Redirect Routing
================

.. php:staticmethod:: redirect($route, $url, $options = [])

Redirect routing allows you to issue HTTP status 30x redirects for
incoming routes, and point them at different URLs. This is useful
when you want to inform client applications that a resource has moved
and you don't want to expose two URLs for the same content

Redirection routes are different from normal routes as they perform an actual
header redirection if a match is found. The redirection can occur to
a destination within your application or an outside location::

    $routes->redirect(
        '/home/*',
        ['controller' => 'Articles', 'action' => 'view'],
        ['persist' => true]
        // Or ['persist'=>['id']] for default routing where the
        // view action expects $id as an argument.
    );

Redirects ``/home/*`` to ``/articles/view`` and passes the parameters to
``/articles/view``. Using an array as the redirect destination allows
you to use other routes to define where a URL string should be
redirected to. You can redirect to external locations using
string URLs as the destination::

    $routes->redirect('/articles/*', 'http://google.com', ['status' => 302]);

This would redirect ``/articles/*`` to ``http://google.com`` with a
HTTP status of 302.

.. _custom-route-classes:

Custom Route Classes
====================

Custom route classes allow you to extend and change how individual routes parse
requests and handle reverse routing. Route classes have a few conventions:

* Route classes are expected to be found in the ``Routing\\Route`` namespace of your application or plugin.
* Route classes should extend :php:class:`Cake\\Routing\\Route`.
* Route classes should implement one or both of ``match()`` and/or ``parse()``.

The ``parse()`` method is used to parse an incoming URL. It should generate an
array of request parameters that can be resolved into a controller & action.
Return ``false`` from this method to indicate a match failure.

The ``match()`` method is used to match an array of URL parameters and create a string URL.
If the URL parameters do not match the route ``false`` should be returned.

You can use a custom route class when making a route by using the ``routeClass``
option::

    Router::connect(
         '/:slug',
         ['controller' => 'Articles', 'action' => 'view'],
         ['routeClass' => 'SlugRoute']
    );

This route would create an instance of ``SlugRoute`` and allow you
to implement custom parameter handling. You can use plugin route classes using
standard :term:`plugin syntax`.

Default Route Class
-------------------

.. php:staticmethod:: defaultRouteClass($routeClass = null)

If you want to use an alterate route class for all your routes besides the
default ``Route``, you can do so by calling ``Router::defaultRouteClass()``
before setting up any routes and avoid having to specify the ``routeClass``
option for each route. For example using::

    Router::defaultRouteClass('DashedRoute');

will cause all routes connected after this to use the ``DashedRoute`` route class.
Calling the method without an argument will return current default route class.

Fallbacks method
----------------

.. php:method:: fallbacks($routeClass = null)

The fallbacks method is a simple shortcut for defining default routes. The method
uses the passed routing class for the defined rules or if no class is provided the
class returned by ``Router::defaultRouteClass()`` is used.

Calling fallbacks like so::

    $routes->fallbacks('InflectedRoute');

Is equivalent to the following explicit calls::

    $routes->connect('/:controller', ['action' => 'index'], ['routeClass' => 'InflectedRoute']);
    $routes->connect('/:controller/:action/*', [], , ['routeClass' => 'InflectedRoute']);

.. note::

    Using the default route class (``Route``) with fallbacks, or any route
    with ``:plugin`` and/or ``:controller`` route elements will result in
    inconsistent URL case.

Handling Named Parameters in URLs
=================================

Although named parameters were removed in CakePHP 3.0, applications may have
published URLs containing them.  You can continue to accept URLs containing
named parameters.

In your controller's ``beforeFilter()`` method you can call
``parseNamedParams()`` to extract any named parameters from the passed
arguments::

    public function beforeFilter()
    {
        parent::beforeFilter();
        Router::parseNamedParams($this->request);
    }

This will populate ``$this->request->params['named']`` with any named parameters
found in the passed arguments.  Any passed argument that was interpreted as a
named parameter, will be removed from the list of passed arguments.

.. _request-action:

RequestActionTrait
==================

.. php:trait:: RequestActionTrait

    This trait allows classes which include it to create sub-requests or
    request actions.

.. php:method:: requestAction(string $url, array $options)

    This function calls a controller's action from any location and
    returns data from the action. The ``$url`` passed is a
    CakePHP-relative URL (/controllername/actionname/params). To pass
    extra data to the receiving controller action add to the $options
    array.

    .. note::

        You can use ``requestAction()`` to retrieve a fully rendered view
        by passing 'return' in the options:
        ``requestAction($url, ['return']);``. It is important to note
        that making a requestAction using 'return' from a controller method
        can cause script and css tags to not work correctly.

    .. warning::

        If used without caching ``requestAction`` can lead to poor
        performance. It is seldom appropriate to use in a controller.

    ``requestAction`` is best used in conjunction with (cached)
    elements – as a way to fetch data for an element before rendering.
    Let's use the example of putting a "latest comments" element in the
    layout. First we need to create a controller function that will
    return the data::

        // Controller/CommentsController.php
        class CommentsController extends AppController
        {
            public function latest()
            {
                if (!$this->request->is('requested')) {
                    throw new ForbiddenException();
                }
                return $this->Comments->find('all', [
                    'order' => 'Comment.created DESC',
                    'limit' => 10
               ]);
            }
        }

    You should always include checks to make sure your requestAction methods are
    actually originating from ``requestAction``.  Failing to do so will allow
    requestAction methods to be directly accessible from a URL, which is
    generally undesirable.

    If we now create a simple element to call that function::

        // View/Element/latest_comments.ctp

        $comments = $this->requestAction('/comments/latest');
        foreach ($comments as $comment) {
            echo $comment->title;
        }

    We can then place that element anywhere to get the output
    using::

        echo $this->element('latest_comments');

    Written in this way, whenever the element is rendered, a request
    will be made to the controller to get the data, the data will be
    processed, and returned. However in accordance with the warning
    above it's best to make use of element caching to prevent needless
    processing. By modifying the call to element to look like this::

        echo $this->element('latest_comments', [], ['cache' => '+1 hour']);

    The ``requestAction`` call will not be made while the cached
    element view file exists and is valid.

    In addition, requestAction now takes array based cake style URLs::

        echo $this->requestAction(
            ['controller' => 'Articles', 'action' => 'featured'],
            ['return']
        );

    The URL based array are the same as the ones that :php:meth:`HtmlHelper::link()`
    uses with one difference - if you are using passed parameters, you must put them
    in a second array and wrap them with the correct key. This is because
    requestAction merges the extra parameters (requestAction's 2nd parameter)
    with the ``request->params`` member array and does not explicitly place them
    under the ``pass`` key. Any additional keys in the ``$option`` array will
    be made available in the requested action's ``request->params`` property::

        echo $this->requestAction('/articles/view/5');

    As an array in the requestAction would then be::

        echo $this->requestAction(
            ['controller' => 'Articles', 'action' => 'view', 5],
        );

    You can also pass querystring arguments, post data or cookies using the
    appropriate keys. Cookies can be passed using the ``cookies`` key.
    Get parameters can be set with ``query`` and post data can be sent
    using the ``post`` key::

        $vars = $this->requestAction('/articles/popular', [
          'query' => ['page' = > 1],
          'cookies' => ['remember_me' => 1],
        ]);

    .. note::

        Unlike other places where array URLs are analogous to string URLs,
        requestAction treats them differently.

    When using an array URL in conjunction with requestAction() you
    must specify **all** parameters that you will need in the requested
    action. This includes parameters like ``$this->request->data``.  In addition
    to passing all required parameters, passed arguments must be done
    in the second array as seen above.

.. toctree::
    :glob:
    :maxdepth: 1

    /development/dispatch-filters

.. meta::
    :title lang=en: Routing
    :keywords lang=en: controller actions,default routes,mod rewrite,code index,string url,php class,incoming requests,dispatcher,url url,meth,maps,match,parameters,array,config,cakephp,apache,router
