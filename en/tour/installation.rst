============
Installation
============

.. highlight:: php

Guzzle is a PHP 5.3+ HTTP client library and framework for building RESTful web service clients.

Requirements
------------

#. PHP 5.3.2+ compiled with the cURL extension
#. A recent version of cURL 7.16.2+ compiled with OpenSSL and zlib
#. `PHPUnit <http://www.phpunit.de/manual/3.6/en/installation.html>`_ is required to run the unit tests
#. `node.js <http://nodejs.org>`_ is required to run the unit tests
#. `Phing <http://www.phing.info/trac/>`_ is required to run the build scripts

Installing Guzzle
-----------------

Adding Guzzle to a project using PHAR
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle is distributed using a PHAR file that compresses all of the required classes into a single file.  In order to simplify installing Guzzle, `Monolog <https://github.com/seldaek/monolog>`_, `Doctrine Common <https://github.com/doctrine/common>`_ Cache layer, `Symfony2's Validator <https://github.com/symfony/Validator>`_, and `Symfony2's EventDispatcher <https://github.com/symfony/EventDispatcher>`_ are included.  Including the Guzzle phar file in your application will automatically configure autoloading so everything should just work.  You can download the Guzzle phar file from https://github.com/downloads/guzzle/guzzle/guzzle.phar::

    $ wget --quiet https://github.com/downloads/guzzle/guzzle/guzzle.phar

And include it in your scripts::

    require '/path/to/guzzle.phar';

If you already have your own autoloader and don't want to install the extra libraries, you can install the minimal PHAR distribution of Guzzle at https://github.com/downloads/guzzle/guzzle/guzzle-min.phar.

Adding Guzzle to a project using Composer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create composer.json file in the project root::

    {
        "require": {
            "guzzle/guzzle": "*"
        }
    }

Then download composer.phar and run the install command::

    $ wget --quiet http://getcomposer.org/composer.phar
    $ php composer.phar install --install-suggests

Installing Guzzle from GitHub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guzzle can be installed from source by cloning the Guzzle github repository and installing the dependencies using composer::

    $ git clone https://github.com/guzzle/guzzle.git
    $ wget --quiet http://getcomposer.org/composer.phar
    $ php composer.phar install --install-suggests

Using your own autoloader
^^^^^^^^^^^^^^^^^^^^^^^^^

You will need to add Guzzle and Guzzle's dependencies to your application's autoloader.  If your application does not already use an autoloader, you can use the `Symfony2 ClassLoader <https://github.com/symfony/ClassLoader>`_ component::

    <?php

    require_once '/path/to/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';
    $classLoader = new \Symfony\Component\ClassLoader\UniversalClassLoader();
    $classLoader->registerNamespaces(array(
        'Guzzle' => '/path/to/guzzle/src',
        'Symfony' => '/path/to/symfony/src'
    ));
    $classLoader->register();

Running the unit tests
~~~~~~~~~~~~~~~~~~~~~~

Guzzle is unit tested using PHPUnit.  You will need to create your own phpunit.xml file in order to run the unit tests.  You can customize this file to suit your testing needs::

    $ cp phpunit.xml.dist phpunit.xml

You will need to install `node.js <http://nodejs.org/>`_ v0.5.0 or newer in order to test the cURL implementation.

Framework integrations
----------------------

Using Guzzle with Symfony
~~~~~~~~~~~~~~~~~~~~~~~~~

A `Guzzle Symfony2 bundle <https://github.com/ddeboer/GuzzleBundle>`_ is available on github thanks to `ddeboer <https://github.com/ddeboer>`_

Using Guzzle with Silex
~~~~~~~~~~~~~~~~~~~~~~~

A `Guzzle Silex service provider <https://github.com/guzzle/guzzle-silex-extension>`_ is available on github.
