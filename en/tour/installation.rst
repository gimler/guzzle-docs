============
Installation
============

.. highlight:: php

Requirements
------------

#. PHP 5.3.2+ compiled with the cURL extension
#. A recent version of cURL 7.16.2+ compiled with OpenSSL and zlib
#. `PHPUnit <http://www.phpunit.de/manual/3.6/en/installation.html>`_ is required to run the unit tests
#. `node.js <http://nodejs.org>`_ is required to run the unit tests
#. `Phing <http://www.phing.info/trac/>`_ is required to run the build scripts

Installing Guzzle
-----------------

Phar
~~~~

Guzzle is distributed using a Phar file that compresses all of the required classes into a single file.  In order to simplify installing Guzzle, `Monolog <https://github.com/seldaek/monolog>`_, `Doctrine Common <https://github.com/doctrine/common>`_ Cache layer, and `Symfony2's EventDispatcher <https://github.com/symfony/EventDispatcher>`_ are included.  Including the Guzzle phar file in your application will automatically configure autoloading so everything should just work.  You can download the Guzzle phar file from http://guzzlephp.org/guzzle.phar and simply include it in your scripts::

    require '/path/to/guzzle.phar';

If you already have your own autoloader and don't want to install the extra libraries, you can install the minimal Phar distribution of Guzzle at http://guzzlephp.org/guzzle-min.phar.

Composer
~~~~~~~~

Create composer.json file in the project root:

.. code-block:: javascript

    {
        "require": {
            "guzzle/guzzle": "2.6.*"
        }
    }

.. note::

    It's always a good idea to specify at least a minor version that you know is compatible with your application.

Then download composer.phar and run the install command:

.. code-block:: bash

    curl -s http://getcomposer.org/installer | php && ./composer.phar install

Github
~~~~~~

Guzzle can be installed from source by cloning the Guzzle github repository and installing the dependencies using composer:

.. code-block:: bash

    git clone https://github.com/guzzle/guzzle.git
    cd guzzle
    curl -s http://getcomposer.org/installer | php && ./composer.phar install --dev

PEAR
~~~~

Guzzle can be installed through PEAR (`this needs your help! <https://github.com/guzzle/guzzle/issues/24>`_):

.. code-block:: bash

    pear channel-discover guzzlephp.org/pear
    pear install guzzle/guzzle

Using your own autoloader
~~~~~~~~~~~~~~~~~~~~~~~~~

If you aren't using the Phar file and you aren't using Composer, then you will need to add Guzzle and Guzzle's dependencies to your application's autoloader.  If your application does not already use an autoloader, you can use the `Symfony2 ClassLoader component  <https://github.com/symfony/ClassLoader>`_ component::

    require_once '/path/to/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $classLoader = new \Symfony\Component\ClassLoader\UniversalClassLoader();
    $classLoader->registerNamespaces(array(
        'Guzzle'  => '/path/to/guzzle/src',
        'Symfony' => '/path/to/symfony/src'
    ));
    $classLoader->register();

Running the unit tests
----------------------

Guzzle is unit tested with PHPUnit.  You will need to create your own phpunit.xml file in order to run the unit tests.  You can customize this file to suit your testing needs:

.. code-block:: bash

    cp phpunit.xml.dist phpunit.xml
    phpunit

You will need to install `node.js <http://nodejs.org/>`_ v0.5.0 or newer in order to test the cURL implementation.

Framework integrations
----------------------

Using Guzzle with Symfony
~~~~~~~~~~~~~~~~~~~~~~~~~

A `Guzzle Symfony2 bundle <https://github.com/ddeboer/GuzzleBundle>`_ is available on github thanks to `ddeboer <https://github.com/ddeboer>`_

Using Guzzle with Silex
~~~~~~~~~~~~~~~~~~~~~~~

A `Guzzle Silex service provider <https://github.com/guzzle/guzzle-silex-extension>`_ is available on github.
