=======
Caching
=======

Guzzle can leverage HTTP's caching specifications using the ``Guzzle\Http\Plugin\CachePlugin``.  The CachePlugin provides a private transparent proxy cache that caches HTTP responses.  The caching logic, based on `RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html>`_, uses HTTP headers to control caching behavior, cache lifetime, and supports automatic ETag and Last-Modified based revalidation.

.. code-block:: php

    <?php

    use Doctrine\Common\Cache\ArrayCache;
    use Guzzle\Http\Client;
    use Guzzle\Common\Cache\DoctrineCacheAdapter;
    use Guzzle\Http\Plugin\CachePlugin;

    $adapter = new DoctrineCacheAdapter(new ArrayCache());
    $cache = new CachePlugin($adapter, true);

    $client = new Client('http://www.wikipedia.org');
    $client->getEventDispatcher()->addSubscriber($cache);

    $response1 = $client->get('/');

    // The next request will revalidate against the origin server to see if it
    // has been modified.  If a 304 response is recieved the response will be
    // served from cache
    $response2 = $client->get('/')->send();

The cache plugin intercepts GET and HEAD requests before they are actually transferred to the origin server.  The cache plugin then generates a hash key based on the request message and checks to see if a response exists in the cache.  If a response exists in the cache, the cache adapter then checks to make sure that the response is still fresh.  If the response is fresh and passes any required revalidation, then the cached response is served instead of contacting the origin server.

Cache options
-------------

There are several options you can add to requests or clients to modify the behavior of the cache plugin.

Override cache TTL
~~~~~~~~~~~~~~~~~~

You can override the number of seconds a cacheable response is stored in the cache by setting the ``cache.override_ttl`` parameter on the params object of a request::

    <?php

    // If the response to the request is cacheable, then the response will be cached for 100 seconds
    $request->getParams()->set('cache.override_ttl', 100);

Revalidation options
~~~~~~~~~~~~~~~~~~~~

You can change the revalidation behavior of a request using the ``cache.revalidate`` parameter.  Setting this parameter to ``never`` will ensure that a revalidation request is never sent, and the response is always served from the origin server.  Setting this parameter to ``skip`` will never revalidate and uses the response stored in the cache.

Normalizing requests for caching
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the ``cache.key_filter`` parameter if you wish to strip certain headers or query string parameters from your request before creating a unique hash for the request.  This parameter can be useful if you send a special authorization header that changes based on the date, but still wish to cache the responses of those requests.  The cache.key_filter format can contain a list of ``query`` and ``header`` values to remove from the request hash.  You are not required to specify both a query or header list, and either list can contain one or more keys to ignore.  For example, here we are saying that the ``a`` and ``q`` query string variables and the ``Date`` header should be ignored when generating a hash key for the request::

    <?php

    $request->getParams()->set('cache.key_filter', 'query=a, q; header=Date, Host');

Setting Client-wide cache settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify cache settings for every request created by a client by adding cache settings to the configuration options of a client.

.. code-block:: php

    <?php

    $client = new Guzzle\Http\Client('http://www.test.com', array(
        'cache.override_ttl' => 3600,
        'cacche.revalidate' => 'never'
    ));

    echo $client->get('/')->getParams()->get('cache.override_ttl');
    // Prints 3600
    echo $client->get('/')->getParams()->get('cache.revalidate');
    // Prints never

Cache revalidation
------------------

If the cache plugin determines that a response to a GET request needs revalidation, a conditional GET is transferred to the origin server.  If the origin server returns a 304 response, then a response containing the merged headers of the cached response with the new response and the entity body of the cached response is returned.


Cache adapters
--------------

The cache plugin requires a cache adapter so that is can store responses in a cache.  Guzzle currently supports cache adapters for `Doctrine 2.0 <http://www.doctrine-project.org/>`_ and the Zend Framework 1.0 or 2.0.

Doctrine
~~~~~~~~

.. code-block:: php

    <?php

    use Doctrine\Common\Cache\ArrayCache;
    use Guzzle\Common\Cache\DoctrineCacheAdapter;
    use Guzzle\Http\Plugin\CachePlugin;

    $backend = new ArrayCache();
    $adapter = new DoctrineCacheAdapter($backend);
    $cache = new CachePlugin($adapter);

Zend Framework
~~~~~~~~~~~~~~

.. code-block:: php

    <?php

    use Guzzle\Common\Cache\ZendCacheAdapter;
    use Zend\Cache\Backend\TestBackend;

    $backend = new TestBackend();
    $adapter = new ZendCacheAdapter($backend);
    $cache = new CachePlugin($adapter);
