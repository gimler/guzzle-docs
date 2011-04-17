========
Overview
========

.. highlight:: php

**Guzzle is a PHP 5.3+ framework for consuming web services and building RESTful web service clients.**

Guzzle makes writing web service clients an easy task by providing a simple pattern to follow:

#. Extend the default ``Guzzle\Service\Client`` class
#. Create commands for each API action.  Guzzle uses the `command pattern <http://en.wikipedia.org/wiki/Command_pattern>`_.
#. Add the web service client configuration to your services.xml file

Or, more simply, run ``phing -f build/build.xml template`` to create a skeleton template for a new web service client.

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

If you aren't using any Guzzle web service clients like guzzle-aws or guzzle-unfuddle, then you can just download the `guzzle phar file <http://build.guzzlephp.org/guzzle.phar>`_ and include it in your php scripts::

    <?php

    require 'guzzle.phar';

Installing Guzzle and Guzzle web service clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle can be installed by cloning the Guzzle github repository:

.. code-block:: bash

    git clone https://github.com/guzzle/guzzle.git

You will need to add Guzzle to your application's autoloader.  Guzzle ships with a few select classes from other vendors, one of which is the `Symfony2 <http://symfony.com/>`_ universal class loader.  If your application does not already use an autoloader, you can use the Symfony2 autoloader distributed with Guzzle::

    <?php
    require_once '/path/to/guzzle/vendor/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $classLoader = new \Symfony\Component\ClassLoader\UniversalClassLoader();
    $classLoader->registerNamespaces(array(
        'Guzzle' => '/path/to/guzzle/src'
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

When installing a Guzzle web service client, check the service's installation instructions for specific examples on how to install the service.  Services can typically be installed using a git submodule within your Guzzle installation.  Here is an example of installing the AWS web service client:

.. code-block:: bash

    cd /path/to/guzzle
    git submodule add git://github.com/guzzle/guzzle-aws.git ./src/Guzzle/Aws

Autoloading Services
~~~~~~~~~~~~~~~~~~~~

Services that are installed within the path of Guzzle will be autoloaded automatically using the autoloader settings configured for the Guzzle library (e.g. /Guzzle/Aws).  If you install a Guzzle service outside of this directory structure, you will need to add the service to the autoloader separately.