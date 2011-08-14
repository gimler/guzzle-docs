========
Overview
========

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

If you are going to use Guzzle web service clients in your projects (e.g. guzzle-aws), you will need to clone them into your installation of Guzzle.  You can use the phing build script to help with this::

    cd /path/to/guzzle/clone

    # Install/update the default Guzzle web service clients
    phing -f build/build.xml all-services

    # Install/update a particular Guzzle web service client (if it is not one of the default clients)
    phing -f build/build.xml service -Drepo=git@github.com:guzzle/guzzle-aws.git -Dpath=./src/Guzzle/Aws

You can build a phar file containing your clone of Guzzle and the cloned web service clients.  The phar file will automatically register an autoloader function to handle loading all Guzzle classes.  You can build the phar file by running the following command::

    # Rebuild the phar file to include these services
    phing -f build/build.xml phar

Available web service clients
-----------------------------

Guzzle web service clients are distributed separately from the Guzzle framework.  Guzzle officially supports a few web service clients, and hopefully there will be third-party created services coming soon:

* `Amazon web services (AWS) <https://github.com/guzzle/guzzle-aws>`_ - Amazon S3, SimpleDB, SQS, MWS web service client
* `Unfuddle <https://github.com/guzzle/guzzle-unfuddle>`_ - Unfuddle web service API client
* `Cardinal Commerce <https://github.com/guzzle/guzzle-cardinal-commerce>`_ - Cardinal Commerce web service client