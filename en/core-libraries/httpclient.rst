Http Client
###########

.. php:namespace:: Cake\Network\Http

.. php:class:: Client(mixed $config = [])

CakePHP includes a basic but powerful HTTP client which can be easily used for
making requests. It is a great way to communicate with webservices, and
remote APIs.

Doing Requests
==============

Doing requests is simple and straight forward.  Doing a get request looks like::

    use Cake\Network\Http\Client;

    $http = new Client();

    // Simple get
    $response = $http->get('http://example.com/test.html');

    // Simple get with querystring
    $response = $http->get('http://example.com/search', ['q' => 'widget']);

    // Simple get with querystring & additional headers
    $response = $http->get('http://example.com/search', ['q' => 'widget'], [
      'headers' => ['X-Requested-With' => 'XMLHttpRequest']
    ]);

Doing post and put requests is equally simple::

    // Send a POST request with application/x-www-form-urlencoded encoded data
    $http = new Client();
    $response = $http->post('http://example.com/posts/add', [
      'title' => 'testing',
      'body' => 'content in the post'
    ]);

    // Send a PUT request with application/x-www-form-urlencoded encoded data
    $response = $http->put('http://example.com/posts/add', [
      'title' => 'testing',
      'body' => 'content in the post'
    ]);

    // Other methods as well.
    $http->delete(...);
    $http->head(...);
    $http->patch(...);

Creating Multipart Requests with Files
======================================

You can include files in request bodies by including them in the data array::

    $http = new Client();
    $response = $http->post('http://example.com/api', [
      'image' => '@/path/to/a/file',
      'logo' => $fileHandle
    ]);

By prefixing data values with ``@`` or including a filehandle in the data.  If
a filehandle is used, the filehandle will be read until its end, it will not be
rewound before being read.

Sending Request Bodies
======================

When dealing with REST API's you often need to send request bodies that are not
form encoded. Http\\Client exposes this through the type option::

    // Send a JSON request body.
    $http = new Client();
    $response = $http->post(
      'http://example.com/tasks',
      json_encode($data),
      ['type' => 'json']
    );

The ``type`` key can either be a one of 'json', 'xml' or a full mime type.
When using the ``type`` option, you should provide the data as a string. If you're
doing a GET request that needs both querystring parameters and a request body
you can do the following::

    // Send a JSON body in a GET request with query string parameters.
    $http = new Client();
    $response = $http->get(
      'http://example.com/tasks',
      ['q' => 'test', '_content' => json_encode($data)],
      ['type' => 'json']
    );

.. _http_client_request_options:

Request Method Options
=======================

Each HTTP method takes an ``$options`` parameter which is used to provide
addition request information.  The following keys can be used in ``$options``:

- ``headers`` - Array of additional headers
- ``cookie`` - Array of cookies to use.
- ``proxy`` - Array of proxy information.
- ``auth`` - Array of authentication data, the ``type`` key is used to delegate to
  an authentication strategy. By default Basic auth is used.
- ``ssl_verify_peer`` - defaults to ``true``. Set to ``false`` to disable SSL certification
  verification (not advised)
- ``ssl_verify_depth`` - defaults to 5. Depth to traverse in the CA chain.
- ``ssl_verify_host`` - defaults to ``true``. Validate the SSL certificate against the host name.
- ``ssl_cafile`` - defaults to built in cafile. Overwrite to use custom CA bundles.
- ``timeout`` - Duration to wait before timing out.
- ``type`` - Send a request body in a custom content type. Requires ``$data`` to
  either be a string, or the ``_content`` option to be set when doing GET
  requests.

The options parameter is always the 3rd parameter in each of the HTTP methods.
They can also be use when constructing ``Client`` to create
:ref:`scoped clients <http_client_scoped_client>`.

Authentication
==============

Http\\Client supports a few different authentication systems.  Different
authentication strategies can be added by developers. Auth strategies are called
before the request is sent, and allow headers to be added to the request
context.

Using Basic Authentication
--------------------------

An example of basic authentication::

    $http = new Client();
    $response = $http->get('http://example.com/profile/1', [], [
      'auth' => ['username' => 'mark', 'password' => 'secret']
    ]);

By default Http\\Client will use basic authentication if there is no ``'type'``
key in the auth option.


Using Digest Authentication
---------------------------

An example of basic authentication::

    $http = new Client();
    $response = $http->get('http://example.com/profile/1', [], [
      'auth' => [
        'type' => 'digest',
        'username' => 'mark',
        'password' => 'secret',
        'realm' => 'myrealm',
        'nonce' => 'onetimevalue',
        'qop' => 1,
        'opaque' => 'someval'
      ]
    ]);

By setting the 'type' key to 'digest', you tell the authentication subsystem to
use digest authentication.

OAuth 1 Authentication
----------------------

Many modern web-services require OAuth authentication to access their API's.
The included OAuth authentication assumes that you already have your consumer
key and consumer secret::

    $http = new Client();
    $response = $http->get('http://example.com/profile/1', [], [
      'auth' => [
        'type' => 'oauth',
        'consumerKey' => 'bigkey',
        'consumerSecret' => 'secret',
        'token' => '...',
        'tokenSecret' => '...',
        'realm' => 'tickets',
      ]
    ]);

Proxy Authentication
--------------------

Some proxies require authentication to use them. Generally this authentication
is Basic, but it can be implemented by any authentication adapter.  By default
Http\\Client will assume Basic authentication, unless the type key is set::

    $http = new Client();
    $response = $http->get('http://example.com/test.php', [], [
      'proxy' => [
        'username' => 'mark',
        'password' => 'testing',
        'port' => 12345,
      ]
    ]);

.. _http_client_scoped_client:

Creating Scoped Clients
=======================

Having to re-type the domain name, authentication and proxy settings can become
tedious & error prone.  To reduce the change for mistake and relieve some of the
tedium, you can create scoped clients::

    // Create a scoped client.
    $http = new Client([
      'host' => 'api.example.com',
      'scheme' => 'https',
      'auth' => ['username' => 'mark', 'password' => 'testing']
    ]);

    // Do a request to api.example.com
    $response = $http->get('/test.php');

The following information can be used when creating a scoped client:

* host
* scheme
* proxy
* auth
* port
* cookies
* timeout
* ssl_verify_peer
* ssl_verify_depth
* ssl_verify_host

Any of these options can be overridden by specifying them when doing requests.
host, scheme, proxy, port are overridden in the request URL::

    // Using the scoped client we created earlier.
    $response = $http->get('http://foo.com/test.php');

The above will replace the domain, scheme, and port.  However, this request will
continue using all the other options defined when the scoped client was created.
See :ref:`http_client_request_options` for more information on the options
supported.


Setting and Managing Cookies
============================

Http\\Client can also accept cookies when making requests. In addition to
accepting cookies, it will also automatically store valid cookies set in
responses. Any response with cookies, will have them stored in the originating
instance of Http\\Client. The cookies stored in a Client instance are
automatically included in future requests to domain + path combinations that
match::

    $http = new Client([
        'host' => 'cakephp.org'
    ]);

    // Do a request that sets some cookies
    $response = $http->get('/');

    // Cookies from the first request will be included
    // by default.
    $response2 = $http->get('/changelogs');

You can always override the auto-included cookies by setting them in the
request's ``$options`` parameters::

    // Replace a stored cookie with a custom value.
    $response = $http->get('/changelogs', [], [
        'cookies' => ['sessionid' => '123abc']
    ]);


Response Objects
================

.. php:class:: Response

Response objects have a number of methods for inspecting the response data.

.. php:method:: body($parser = null)

    Get the response body. Pass in an optional parser, to decode the response
    body. For example. `json_decode` could be used for decoding response data.

.. php:method:: header($name)

    Get a header with ``$name``. ``$name`` is case-insensitive.

.. php:method:: headers()

    Get all the headers.

.. php:method:: isOk()

    Check if the response was ok. Any valid 20x response code will be
    treated as OK.

.. php:method:: isRedirect()

    Check if the response was a redirect.

.. php:method:: cookies()

    Get the cookies from the response. Cookies will be returned as
    an array with all the properties that were defined in the response header.
    To access the raw cookie data you can use :php:meth:`header()`

.. php:method:: cookie($name = null, $all = false)

    Get a single cookie from the response. By default only the value of a cookie
    is returned. If you set the second parameter to ``true``, all the properties
    set in the response will be returned.

.. php:method:: statusCode()

    Get the status code.

.. php:method:: encoding()

    Get the encoding of the response. Will return null if the response
    headers did not contain an encoding.

In addition to the above methods you can also use object accessors to read data
from the following properties:

* cookies
* headers
* body
* code
* json
* xml


::

    $http = new Client(['host' => 'example.com']);
    $response = $http->get('/test');

    // Use object accessors to read data.
    debug($response->body);
    debug($response->code);
    debug($response->headers);

.. _http-client-xml-json:

Reading JSON and XML Response Bodies
------------------------------------

Since JSON and XML responses are commonly used, response objects provide easy to
use accessors to read decoded data. JSON data is decoded into an array, while
XML data is decoded into a ``SimpleXMLElement`` tree::

    // Get some XML
    $http = new Client();
    $response = $http->get('http://example.com/test.xml');
    $xml = $response->xml;

    // Get some JSON
    $http = new Client();
    $response = $http->get('http://example.com/test.json');
    $json = $response->json;

The decoded response data is stored in the response object, so accessing it
multiple times has no additional cost.

.. meta::
    :title lang=en: HttpClient
    :keywords lang=en: array name,array data,query parameter,query string,php class,string query,test type,string data,google,query results,webservices,apis,parameters,cakephp,meth,search results
