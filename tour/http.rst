===========
Taming HTTP
===========

Guzzle gives PHP developers complete control over HTTP requests while utilizing HTTP/1.1 best practices.  Guzzle's HTTP functionality is a robust framework built on top of the `PHP libcurl bindings <http://www.php.net/curl>`_.

Quick overview of Guzzle's HTTP features
----------------------------------------

* Supports GET, HEAD, POST, DELETE, OPTIONS, PUT, and TRACE methods
* `Persistent connections <http://en.wikipedia.org/wiki/Persistent_connections>`_ are implicitly managed by Guzzle, resulting in huge performance benefits
* Allows custom entity bodies to be sent in PUT and POST requests, including sending data from a PHP stream
* Allows full access to request HTTP headers
* Responses can be cached and served from cache using the CachePlugin
* Failed requests can be retried using truncated `exponential backoff <http://en.wikipedia.org/wiki/Exponential_backoff>`_ using the ExponentialBackoffPlugin
* All data sent over the wire can be logged using the LogPlugin
* Cookies sessions can be maintained between requests using the CookiePlugin
* Send requests in parallel
* Supports HTTPS and SSL certificate validation
* Requests can be sent through a proxy
* Automatically requests compressed data and automatically decompresses data
* Supports authentication methods provided by cURL (Basic, Digest, GSS Negotiate, NTLM)
* Transparently follows redirects
* Subject/Observer signal slot system for modifying request behavior
* Request signal slot events for before/progress/complete/failure/etc...

Most of the functionality implemented in the libcurl bindings has been simplified and abstracted by Guzzle. Developers who need access to `cURL specific functionality <http://www.php.net/curl_setopt>`_ that is not abstracted by Guzzle (e.g. proxies and SSL) can still add cURL handle specific behavior to Guzzle HTTP requests using something like this: ``$request->getCurlOptions()->set(CURLOPT_SSL_VERIFYHOST, true);``

Sending requests and using responses
------------------------------------

Requests
~~~~~~~~

Guzzle provides full access to request and response messages.  Requests can be created manually or using the RequestFactory.  Using the RequestFactory is the preferred method of creating requests:

.. code-block:: php

    <?php

    use Guzzle\Http\Message\RequestFactory;

    // Create a GET/HEAD/DELETE/POST/PUT/OPTIONS request using the RequestFactory (preferred)
    $request = RequestFactory::get('http://www.example.com/path?query=123&value=abc');
    $request = RequestFactory::head('http://www.example.com/path?query=123&value=abc');
    $request = RequestFactory::delete('http://www.example.com/path?query=123&value=abc');
    $request = RequestFactory::post('http://www.example.com/path?query=123&value=abc');
    $request = RequestFactory::put('http://www.example.com/path?query=123&value=abc');
    $request = RequestFactory::options('http://www.example.com/path');

    // Create a request by method
    $request = RequestFactory::create('TRACE', 'http://www.example.com/');

    // Create a GET/HEAD/DELETE request manually
    $request = new Guzzle\Http\Message\Request('GET', 'http://www.example.com/path?query=123&value=abc');

    // Create a POST/PUT request using the EntityEnclosingRequest class
    $request = new Guzzle\Http\Message\EntityEnclosingRequest('PUT', 'http://www.example.com/path?query=123&value=abc');

    // Check if a resource supports the DELETE method
    $supportsDelete = RequestFactory::options('http://www.example.com/path')->send()->isMethodAllowed('delete');

If you know exactly what HTTP message you want to send, you can create request objects from messages:

.. code-block:: php

    <?php

    $request = RequestFactory::fromMessage(
        "PUT / HTTP/1.1\r\n" .
        "Host: test.com:8081\r\n" .
        "Content-Type: text/plain"
        "Transfer-Encoding: chunked\r\n" .
        "\r\n" .
        "this is the body"
    );

    $request->send();

Request objects are all about building an HTTP message.  Each part of an HTTP request message can be set individually using methods on the request object or set in bulk using the setUrl() method.  Here's the format of an HTTP request with each part of the request referencing the method used to change it::

    PUT(a) /path(b)?query=123(c) HTTP/1.1(d)
    X-Header(e): header
    Content-Length(e): 4

    data(f)

+-------------------------+---------------------------------------------------------------------------------+
| a. **Method**           | The request method can only be set when instantiating a request                 |
+-------------------------+---------------------------------------------------------------------------------+
| b. **Path**             | ``$request->setPath('/path');``                                                 |
+-------------------------+---------------------------------------------------------------------------------+
| c. **Query**            |``$request->getQuery()->set('query', '123'); // see ``Guzzle\Http\QueryString``  |
+-------------------------+---------------------------------------------------------------------------------+
| d. **Protocol version** | ``$request->setProtocolVersion('1.1');``                                        |
+-------------------------+---------------------------------------------------------------------------------+
| e. **Header**           | ``$request->setHeader('X-Header', 'header');``                                  |
+-------------------------+---------------------------------------------------------------------------------+
| f. **Entity Body**      |  ``$request->setBody('data'); // Only available with PUT and POST requests``    |
+-------------------------+---------------------------------------------------------------------------------+

PUT
^^^

Here's how to send a PUT request (substitute POST for PUT to send a custom POST request):

.. code-block:: php

    <?php

    // Create a new PUT request, setting headers and an entity body
    $request = RequestFactory::put('http://www.example.com/upload', array(
        'X-Guzzle-Test-Header' => 'header_value'
    ), 'this is the body');

    $response = $request->send();

POST
^^^^

Guzzle helps to make it extremely easy to send POST requests.  POST requests will be sent with an ``application/x-www-form-urlencoded`` Content-Type header if no files are being sent in the POST.  If files are specified in the POST, then the Content-Type header will become ``multipart/form-data``.  Here's how to send a multipart/form-data POST containing files and fields:

.. code-block:: php

    <?php

    $request = RequestFactory::post('http://www.example.com/upload')
        ->addPostFields(array(
            'custom_key' => 'value'
        ))
        ->addPostFiles(array(
            'file' => '/path/to/file.xml'
        ));

    $response = $request->send();

This can be achieved more succinctly using only the RequestFactory.  ``RequestFactory::post()`` accepts three arguments: the URL, optional headers, and the post fields.  To send files in the POST request, prepend the ``@`` symbol to the array value (just like you would if you were using the PHP ``curl_set_opt`` function).

.. code-block:: php

    <?php

    $request = RequestFactory::post('http://www.example.com/upload', null, array(
        'custom_key' => 'value',
        'file' => '@/path/to/file.xml'
    ));

Dealing with errors
^^^^^^^^^^^^^^^^^^^

Requests that receive a 4xx or 5xx response will throw a ``Guzzle\Http\Message\BadResponseException``.  Here's an example of catching a BadResponseException:

.. code-block:: php

    <?php

    try {
        $response = RequestFactory::get('http://www.test.com/not_found.xml')->send();
    } catch (BadResponseException $e) {
        echo 'Uh oh! ' . $e->getMessage();
    }

Throwing an exception when a 4xx or 5xx response is encountered is the default behavior of Guzzle requests.  This behavior can be overridden by specifying a custom onComplete method for your requests.  An onComplete function should follow this functional prototype::

    function onComplete(RequestInterface $request, Response $response, array $default);

The default onComplete method is passed to any custom onComplete method.  This is useful if you wish to override only certain responses and still utilize the default onComplete method.  Here's an example of logging all redirects, but still calling the default onComplete method:

.. code-block:: php

    <?php

    $request = RequestFactory::get('http://test.com/')
        ->setOnComplete(function(RequestInterface $request, Response $response, array $default) {
            if ($response->isRedirect()) {
                MyApplication::log((string) $request);
            }

            call_user_func($default, $request, $response);
        });

Connection problems and cURL specific errors can also occur when transferring requests using Guzzle.  When Guzzle encounters cURL specific errors, a ``Guzzle\Http\Curl\CurlException`` is thrown with an informative error message and access to the cURL error message.  Sending a request that cannot resolve a host name will result in a CurlException with an exception message similar to the following::

    [curl] 6: Couldn't resolve host 'www.nonexistenthost.com' [url] http://www.nonexistenthost.com/ [info] array (
      'url' => 'http://www.nonexistenthost.com/',
      'content_type' => NULL,
      'http_code' => 0,
      'header_size' => 0,
      'request_size' => 0,
      'filetime' => -1,
      'ssl_verify_result' => 0,
      'redirect_count' => 0,
      'total_time' => 0,
      'namelookup_time' => 0,
      'connect_time' => 0,
      'pretransfer_time' => 0,
      'size_upload' => 0,
      'size_download' => 0,
      'speed_download' => 0,
      'speed_upload' => 0,
      'download_content_length' => -1,
      'upload_content_length' => -1,
      'starttransfer_time' => 0,
      'redirect_time' => 0,
      'certinfo' =>
      array (
      ),
    ) [debug] * getaddrinfo(3) failed for www.nonexistenthost.com:80
    * Couldn't resolve host 'www.nonexistenthost.com'
    * Closing connection #0

All of the exceptions thrown during the transfer of a request will extend ``Guzzle\Http\HttpException``.  You can catch this exception only, or target each type of exception that can be encountered (BadResponseException and CurlException).

Entity Bodies
^^^^^^^^^^^^^

`Entity body <http://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html>`_ is the term used for the body of an HTTP message.  The entity body of requests and responses is inherently a `PHP stream <http://php.net/manual/en/book.stream.php>`_ in Guzzle.  The body of the request can be either a string or a PHP stream which are converted into a ``Guzzle\Http\EntityBody`` object using its factory method.  When using a string, the entity body is stored in a `temp PHP stream <http://www.php.net/manual/en/wrappers.php.php>`_.  The use of temp PHP streams will help to protect your application from running out of memory when sending or receiving enormous entity bodies in your messages.  When more than 2MB of data is stored in a temp stream, it automatically stores the data on disk rather than in memory.

EntityBody objects provide a great deal of functionality: compression, decompression, calculate the Content-MD5, calculate the Content-Length (when the resource is repeatable), chunked reading, guessing the Content-Type, determining if the entity body should be compressed, and more.  Guzzle doesn't need to load an entire entity body into a string when sending or retrieving data; entity bodies are streamed when being uploaded and downloaded.

Here's an example of gzip compressing a text file then sending the file to a URL:

.. code-block:: php

    <?php
    use Guzzle\Http\EntityBody;

    $body = EntityBody::factory('/path/to/file.txt');
    $body->compress();
    $request = $factory->put('http://localhost:8080/uploads', null, $body);

    $response = $request->send();

The body of the request can be specified in the ``RequestFactory::put()`` method, or, you can specify the body of the request by calling the ``setBody()`` method of any ``EntityEnclosingRequestInterface`` object.

Responses
~~~~~~~~~

Sending a request will return a ``Guzzle\Http\Message\Response`` object.  You can view the HTTP response message by casting the Response object to a string.  Casting the response to a string will return the entity body of the response as a string too, so this might be an expensive operation if the entity body is stored in a file or network stream.  If you only want to see the response headers, you can call ``getRawHeaders()``.

The Response object contains helper methods for retrieving common response headers.  These helper methods normalize the variations of HTTP response headers so that you will not need to check for the upper-case existence, lowercase existence, or if you aren't sure if the header will contain a hyphen:

.. code-block:: php

    <?php

    // A sample of some of the Response helper methods
    $response->getContentMd5();
    $response->getEtag();
    $response->getCacheControl();

    // Get a header explicitly from the Response
    $response->getHeader('Content-Length');

The entity body of a response can be retrieved by calling ``$response->getBody()``.  Pass TRUE to this method to retrieve the body as a string rather than an EntityBody object;  this is a convenience feature-- an EntityBody can be cast as a string.

Send HTTP requests in parallel
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sending many HTTP requests serially (one at a time) can cause an unnecessary delay in a script's execution. Each request must complete before a subsequent request can be sent. By sending requests in parallel, a pool of HTTP requests can complete at the speed of the slowest request in the pool, significantly reducing the amount of time needed to execute multiple HTTP requests. Guzzle provides a wrapper for the curl_multi functions in PHP.

Here's an example of sending three requests in parallel using a Pool object:

.. code-block:: php

    <?php
    use Guzzle\Http\Message\RequestFactory;
    use Guzzle\Http\Pool\PoolRequestException;
    use Guzzle\Http\Pool\Pool;

    $pool = new Pool();
    $pool->add(RequestFactory::get('http://www.google.com/'));
    $pool->add(RequestFactory::head('http://www.google.com/'));
    $pool->add(RequestFactory::get('https://www.github.com/'));

    try {
        $pool->send();
    } catch (PoolRequestException $e) {
        echo "The following requests encountered an exception: \n";
        foreach ($e as $exception) {
            echo $exception->getRequest() . "\n"
                 . $exception->getMessage() . "\n";
        }
    }

A single request failure will not cause the entire pool of requests to fail.  Any exceptions thrown while transferring a pool of requests will be aggregated into a ``Guzzle\Http\Pool\PoolRequestException``.

Managed persistent HTTP connections
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Persistent HTTP connections is an extremely important aspect of the HTTP/1.1 protocol that is often overlooked by PHP web service clients. Persistent connections allows data to be transferred between a client and server without the need to reconnect each time a subsequent request is sent, providing a significant performance boost to applications that need to send many HTTP requests to the same host.  Guzzle implicitly manages persistent connections for all requests across all services.

HTTP requests and cURL handles are completely separate entities in Guzzle. In order for a request to get a cURL handle to transfer its message to a server, a request retrieves a cURL handle from a cURL handle factory. The default cURL handle factory will maintain a pool of open cURL handles and return an already existent cURL handle (with a persistent HTTP connection) if available, or create a new cURL handle if needed.  Unless you override the curl factory of a request, all requests in Guzzle use the default ``Guzzle\Http\Curl\CurlFactory``.

Guzzle is pretty good about managing cURL handles.  A handle will be closed if the server closes the connection, if a cURL handle has an option that is not easily removed and would corrupt a subsequent request, or if the cURL handle has been idle for too long.  Guzzle limits the number of concurrent idle connections to a host to 2 connections by default.  These connection limits can be adjusted for specific hosts if needed:

.. code-block:: php

    <?php
    use Guzzle\Http\Curl\CurlFactory;

    $factory = CurlFactory::getInstance();

    // Allow 10 idle connections to be managed for mywebsite.com on port 80
    $factory->setMaxIdleForHost('mywebsite.com:80', 10);

To disable connection reuse entirely, set the max idle time of the CurlFactory to 0: ``$factory->setMaxIdleTime(0);``.

Plugins for common HTTP request behavior
----------------------------------------

Guzzle provides easy to use request plugins that add behavior to requests based on signal slot event notifications.

Over the wiring logging
~~~~~~~~~~~~~~~~~~~~~~~

Use the ``Guzzle\Http\Plugin\LogPlugin`` to view all data sent over the wire, including entity bodies and redirects:

.. code-block:: php

    <?php
    use Guzzle\Http\Message\RequestFactory;
    use Guzzle\Common\Log\ZendLogAdapter;
    use Guzzle\Http\Plugin\LogPlugin;

    $adapter = new ZendLogAdapter(new \Zend_Log(new \Zend_Log_Writer_Stream('php://output')));
    $logPlugin = new LogPlugin($adapter, LogPlugin::LOG_VERBOSE);
    $request = RequestFactory::get('http://google.com/');

    // Attach the plugin to the request
    $request->getEventManager()->attach($logPlugin);

    $request->send();

The code sample above wraps a ``Zend_Log`` object using a ``Guzzle\Common\Log\ZendLogAdapter``.  After attaching the request to the plugin, all data sent over the wire will be logged to stdout.  The above code sample would output something like::

    2011-03-10T20:07:56-06:00 DEBUG (7): www.google.com - "GET / HTTP/1.1" - 200 0 - 0.195698 0 45887
    * About to connect() to google.com port 80 (#0)
    *   Trying 74.125.227.50... * connected
    * Connected to google.com (74.125.227.50) port 80 (#0)
    > GET / HTTP/1.1
    Accept: */*
    Accept-Encoding: deflate, gzip
    User-Agent: Guzzle/0.9 (Language=PHP/5.3.5; curl=7.21.2; Host=x86_64-apple-darwin10.4.0)
    Host: google.com

    < HTTP/1.1 301 Moved Permanently
    < Location: http://www.google.com/
    < Content-Type: text/html; charset=UTF-8
    < Date: Fri, 11 Mar 2011 02:06:32 GMT
    < Expires: Sun, 10 Apr 2011 02:06:32 GMT
    < Cache-Control: public, max-age=2592000
    < Server: gws
    < Content-Length: 219
    < X-XSS-Protection: 1; mode=block
    <
    * Ignoring the response-body
    * Connection #0 to host google.com left intact
    * Issue another request to this URL: 'http://www.google.com/'
    * About to connect() to www.google.com port 80 (#1)
    *   Trying 74.125.45.147... * connected
    * Connected to www.google.com (74.125.45.147) port 80 (#1)
    > GET / HTTP/1.1
    Host: www.google.com
    Accept: */*
    Accept-Encoding: deflate, gzip
    User-Agent: Guzzle/0.9 (Language=PHP/5.3.5; curl=7.21.2; Host=x86_64-apple-darwin10.4.0)

    < HTTP/1.1 200 OK
    < Date: Fri, 11 Mar 2011 02:06:32 GMT
    < Expires: -1
    < Cache-Control: private, max-age=0
    < Content-Type: text/html; charset=ISO-8859-1
    < Set-Cookie: PREF=ID=8a61470bce22ed5b:FF=0:TM=1299809192:LM=1299809192:S=axQwBxLyhXV7mbE3; expires=Sun, 10-Mar-2013 02:06:32 GMT; path=/; domain=.google.com
    < Set-Cookie: NID=44=qxXLtXgSKI2S9_mG7KbN7yR2atSje1B9Eft_CHTyjTuIivwE9kB1sATn_YPmBNhZHiNyxcP4_tIYnawjSNWeAepixK3CoKHw-RINrgGNSG3RfpAG7M-IKxHmLhJM6NeA; expires=Sat, 10-Sep-2011 02:06:32 GMT; path=/; domain=.google.com; HttpOnly
    < Server: gws
    < X-XSS-Protection: 1; mode=block
    < Transfer-Encoding: chunked
    <
    * Connection #1 to host www.google.com left intact
    <!doctype html><html><head>
    [...snipped]

Truncated exponential backoff
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``Guzzle\Http\Plugin\ExponentialBackoffPlugin`` automatically retries failed HTTP requests using truncated exponential backoff.  Single requests and requests sent in parallel are retried with this plugin.

.. code-block:: php

    <?php
    use Guzzle\Http\Message\RequestFactory;
    use Guzzle\Http\Plugin\ExponentialBackoffPlugin;

    $request = RequestFactory::get('http://google.com/');
    $request->getEventManager()->attach(new ExponentialBackoffPlugin());
    $request->send();

By default, the ExponentialBackoffPlugin will retry all 500 and 503 responses up to 3 times.  The number of retries and the HTTP status codes the are retried can be configured in the constructor of the plugin.

PHP-based caching forward proxy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle can leverage HTTP's caching specifications using the ``Guzzle\Http\Plugin\CachePlugin``.  The CachePlugin provides a private transparent proxy cache that caches HTTP responses.  The caching logic, based on `RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html>`_, uses HTTP headers to control caching behavior, cache lifetime, and supports ETag and Last-Modified based revalidation.

.. code-block:: php

    <?php
    use Doctrine\Common\Cache\ArrayCache;
    use Guzzle\Common\Cache\DoctrineCacheAdapter;
    use Guzzle\Http\Plugin\CachePlugin;
    use Guzzle\Http\Message\RequestFactory;

    $adapter = new DoctrineCacheAdapter(new ArrayCache());
    $cache = new CachePlugin($adapter, true);

    $request = RequestFactory::get('http://www.wikipedia.org/');
    $request->getEventManager()->attach($cache);
    $request->send();

    // Reset the request to new so it can be reused
    $request->setState('new');

    // The next request will revalidate against the origin server to see if it
    // has been modified.  If a 304 response is recieved the response will be
    // served from cache
    $request->send();

Guzzle doesn't try to reinvent the wheel when it comes to caching or logging.  Plenty of other frameworks, namely the `Zend Framework <http://framework.zend.com/>`_, have excellent solutions in place that you are probably already using in your applications.  Guzzle uses adapters for caching and logging.  Guzzle currently supports log adapters for the Zend Framework and cache adapters for `Doctrine 2.0 <http://www.doctrine-project.org/>`_ and the Zend Framework.

Cookie session plugin
~~~~~~~~~~~~~~~~~~~~~

Some web services require a Cookie in order to maintain a session.  The ``Guzzle\Http\Plugin\CookiePlugin`` will add cookies to requests and parse cookies from responses using a CookieJar object.

.. code-block:: php

    <?php
    use Guzzle\Http\Message\RequestFactory;
    use Guzzle\Http\Plugin\CookiePlugin;
    use Guzzle\Http\Plugin\CookieJar\ArrayCookieJar;

    $plugin = new CookiePlugin(new ArrayCookieJar());
    $request = RequestFactory::get('http://www.yahoo.com/');
    $request->getEventManager()->attach($plugin);

    // Send the request with no cookies and parse the returned cookies
    $request->send();

    // Send the request again, noticing that cookies are being sent
    $request->setState('new')->send();

    echo $request;

Wrapping it all up
~~~~~~~~~~~~~~~~~~

Phew!  That was a lot of information.  There's more to the Guzzle\Http namespace than what was described above.  As always, you can poke around the source and take a look at the unit tests for more information.