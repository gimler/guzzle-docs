========
Overview
========

**Guzzle is a PHP 5.3+ framework for consuming web services and building RESTful web service clients.**

Most web service clients follow a specific pattern: create a client class, create methods for each action that can be taken on the API, create a cURL handle to transfer an HTTP request to the client, parse the response, implement error handling, and return the result. You've probably had to interact with an API that either doesn't have a PHP client or the currently available PHP clients are not up to an acceptable level of quality. When facing these types of situations, you probably find yourself writing a web service that lacks advanced features like over the wire logging, parallel requests, persistent connections, exponential backoff, or cookie management. It wouldn't make sense to spend all that time writing those features-- it's just a simple web service client for just one API... But then you build another client... and another. Suddenly you find yourself with several web service clients to maintain, each client a God class, each reeking of code duplication and lacking most, if not all, of the aforementioned features.  Enter Guzzle.

Guzzle provides the tools necessary to quickly build a testable web service client with complete control over preparing HTTP requests and processing HTTP responses.  Guzzle helps on the HTTP layer by allowing requests to be sent in parallel, automatically managing persistent cURL connections between requests for multiple hosts, and providing various pluggable behaviors for HTTP transactions including exponential backoff, over the wire logging, caching, and cookie management.

Guzzle makes writing web service clients an easy task by providing a simple pattern to follow:

#. Extend the default ``Guzzle\Service\Client`` class
#. Create commands for each API action.  Guzzle uses the `command pattern <http://en.wikipedia.org/wiki/Command_pattern>`_.
#. Add the web service client configuration to your services.xml file

Requirements
------------

#. PHP 5.3+ compiled with the cURL extension
#. A recent version of cURL 7.16.2+ compiled with OpenSSL and zlib
#. PHPUnit is required to run the unit tests
#. node.js is required to run the unit tests

Installing Guzzle
-----------------

Installing only Guzzle
~~~~~~~~~~~~~~~~~~~~~~

If you aren't using any Guzzle web service clients like guzzle-aws or guzzle-unfuddle, then you can just download the `guzzle phar file <http://build.guzzlephp.org/guzzle.phar>`_ and include it in your php scripts.

.. code-block:: php

    <?php

    require 'guzzle.phar';

Installing Guzzle and Guzzle web service clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle can be installed by cloning the Guzzle github repository::

    git clone https://github.com/guzzle/guzzle.git

You will need to add Guzzle to your application's autoloader.  Guzzle ships with a few select classes from other vendors, one of which is the `Symfony2 <http://symfony.com/>`_ universal class loader.  If your application does not already use an autoloader, you can use the Symfony2 autoloader distributed with Guzzle:

.. code-block:: php

    <?php

    require_once '/path/to/guzzle/library/vendor/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $classLoader = new \Symfony\Component\ClassLoader\UniversalClassLoader();
    $classLoader->registerNamespaces(array(
        'Guzzle' => '/path/to/guzzle/library'
    ));
    $classLoader->register();

*Substitute '/path/to/' with the full path to your Guzzle installation.*

If you are going to use Guzzle web service clients in your projects (e.g. guzzle-aws), you will need to add them to your main project as git submodules.  See the README of each project for more information on how to install a particular client.  After installing the web service clients, you can build a single phar file containing the Guzzle framework and all of your installed Guzzle web service clients using ``phing -f build/build.xml phar``.  Then you simply include the phar file in your PHP scripts.

Installing web service clients
------------------------------

Current web service clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle web service clients are distributed separately from the Guzzle framework.  Guzzle officially supports a few web service clients, and hopefully there will be third-party created services coming soon:

* `Amazon web services (AWS) <https://github.com/guzzle/guzzle-aws>`_ - Amazon S3, SimpleDB, SQS, MWS web service client
* `Unfuddle <https://github.com/guzzle/guzzle-unfuddle>`_ - Unfuddle web service API client
* `Cardinal Commerce <https://github.com/guzzle/guzzle-cardinal-commerce>`_ - Cardinal Commerce web service client

When installing a Guzzle web service client, check the service's installation instructions for specific examples on how to install the service.  Services can typically be installed using a git submodule within your Guzzle installation.  Here is an example of installing the AWS web service client::

    cd /path/to/guzzle
    git submodule add git://github.com/guzzle/guzzle-aws.git ./src/Guzzle/Aws

Autoloading Services
~~~~~~~~~~~~~~~~~~~~

Services that are installed within the path of Guzzle will be autoloaded automatically using the autoloader settings configured for the Guzzle library (e.g. /Guzzle/Aws).  If you install a Guzzle service outside of this directory structure, you will need to add the service to the autoloader separately.