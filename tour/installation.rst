============
Installation
============

.. highlight:: php

Guzzle is a PHP 5.3+ framework for HTTP and building RESTful web service clients.

Requirements
------------

#. PHP 5.3+ compiled with the cURL extension
#. A recent version of cURL 7.16.2+ compiled with OpenSSL and zlib
#. `PHPUnit <http://www.phpunit.de/manual/3.6/en/installation.html>`_ is required to run the unit tests
#. `node.js <http://nodejs.org>`_ is required to run the unit tests
#. `Phing <http://www.phing.info/trac/>`_ is required to run the build scripts

Installing Guzzle
-----------------

Installing only Guzzle
~~~~~~~~~~~~~~~~~~~~~~

If you aren't using any Guzzle web service clients like guzzle-aws or guzzle-unfuddle, then you can just download the pre-built `guzzle phar file <http://build.guzzlephp.org/guzzle.phar>`_ and include it in your php scripts::

    require '/path/to/guzzle.phar';

Installing Guzzle from source
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle can be installed from source by cloning the Guzzle github repository:

.. code-block:: bash

    git clone --recursive https://github.com/guzzle/guzzle.git

You will need to add Guzzle to your application's autoloader.  Guzzle ships with a few select classes from other vendors, one of which is the `Symfony2 <http://symfony.com/>`_ universal class loader.  If your application does not already use an autoloader, you can use the Symfony2 autoloader distributed with Guzzle::

    <?php

    require_once '/path/to/guzzle/vendor/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $classLoader = new \Symfony\Component\ClassLoader\UniversalClassLoader();
    $classLoader->registerNamespaces(array(
        'Guzzle' => '/path/to/guzzle/src'
    ));
    $classLoader->register();

Installing web service clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are going to use Guzzle web service clients in your projects (e.g. guzzle-aws), you will need to clone them into your installation of Guzzle into the correct folder (or use a git submodule).  Here's an example of adding guzzle-aws to your project::

    cd /path/to/guzzle
    git clone git@github.com:guzzle/guzzle-aws.git ./src/Guzzle/Aws

You can build a phar file containing your clone of Guzzle and the cloned web service clients.  The phar file will automatically register an autoloader function to handle loading all Guzzle classes.  You can build the phar file by running the following command::

    # Rebuild the phar file to include these services
    phing -f build/build.xml phar

Integrations
------------

Using Guzzle with Symfony
~~~~~~~~~~~~~~~~~~~~~~~~~

A `Guzzle Symfony2 bundle <https://github.com/ddeboer/GuzzleBundle>`_ is available on github thanks to `ddeboer <https://github.com/ddeboer>`_

Using Guzzle with Silex
~~~~~~~~~~~~~~~~~~~~~~~

A `Guzzle Silex service provider <https://github.com/guzzle/guzzle-silex-extension>`_ is available on github.

Registering
^^^^^^^^^^^

.. code-block:: php

    <?php

    require __DIR__ . '/../silex.phar';
    require __DIR__ . '/../vendor/Guzzle/GuzzleServiceProvider.php';

    use Silex\Application;
    use Guzzle\GuzzleServiceProvider;

    $app = new Application();

    $app->register(new GuzzleServiceProvider(), array(
        'guzzle.services' => '/path/to/services.js',
        'guzzle.class_path' => '/path/to/guzzle/src'
    ));

Example Usage
^^^^^^^^^^^^^

.. code-block:: php

    <?php

    // Get a command from your Amazon S3 client
    $command = $app['guzzle']['s3']->getCommand('bucket.list_bucket');
    $command->setBucket('mybucket');

    $objects = $client->execute($command);
    foreach ($objects as $object) {
        echo "{$object['key']} {$object['size']}\n";
    }

    // Using the Guzzle client:
    $response = $app['guzzle.client']->head('http://www.guzzlephp.org/)->send();

Available web service clients
-----------------------------

Guzzle web service clients are distributed separately from the Guzzle framework.  Guzzle officially supports a few web service clients, and hopefully there will be third-party created services coming soon:

* `Amazon web services (AWS) <https://github.com/guzzle/guzzle-aws>`_ - Amazon S3, SimpleDB, SQS, MWS web service client
* `Unfuddle <https://github.com/guzzle/guzzle-unfuddle>`_ - Unfuddle web service API client
* `Cardinal Commerce <https://github.com/guzzle/guzzle-cardinal-commerce>`_ - Cardinal Commerce web service client