============================
Building web service clients
============================

.. highlight:: php

Building web service clients using commands is better than creating requests manually.

#. Better choice for maintainability, features, and ease of use.
#. Best practices are inherently implemented for the end-developer.
#. Changes can be made to the underlying request and response processing without breaking the interface.
#. Can create :doc:`dynamic commands </guide/service/creating_dynamic_commands>` based on a service description.

Simple web service clients make the assumption that the end-developer has an intricate understanding of how to send requests to a web service and how that web service will respond.  If you want to build a reusable RESTful web service client for an application you plan on maintaining, then creating a simple client might not cut it.  Simple clients might work for extremely simple web services (like `Yahoo weather <http://developer.yahoo.com/weather/>`_ for example) or quickly prototyping, but when you're dealing with a huge web service with a ton of options (e.g. Amazon S3), then you're going to want to build a robust web service client that executes commands.

A command-based web service client that abstracts away the HTTP request and response makes a client more future-proof; if the API you are interacting with changes (for example, adds a required field), then you would only have to update the command in one place rather than every single file in your project that interacts with the web service.  Guzzle uses the `command pattern <http://en.wikipedia.org/wiki/Command_pattern>`_ for building requests and processing responses.  Abstracting the underlying implementation helps new developers to quickly send requests to an API using any of the best-practices coded into the command itself, rather than assuming that every developer on your team has an intricate understanding of the web service.

Commands can also be created for developers dynamically using a :doc:`service description </guide/service/creating_dynamic_commands>`.

The following document will describe how to build a command-based web service client for Guzzle.

Setting up
----------

The first thing you will need to do is create the directory structure of your project.  You can quickly create the required directory structure of your project by cloning the `guzzle client template <https://github.com/guzzle/guzzle-client-template>`_ and running the phing build script.

.. code-block:: bash

    $ git clone git://github.com/guzzle/guzzle-client-template.git
    $ cd guzzle-client-template
    $ phing

This phing build target will ask you a series of questions and generate a template for your web service client at your desired path.  The directory structure should resemble the following (of course substituting FootBar for your web service client's name):

.. code-block:: none

    src/FooBar/
    ........... FooBarClient.php
    ........... client.xml
    ........... Command/
    .................... CommandName.php
    tests/
    ...... mock/
    ...... FooBar/
    .............. Command/
    ....................... CommandNameTest.php
    .............. FooBarClientTest.php
    ...... bootstrap.php
    phpunit.xml.dist
    composer.json

After running the phing build target to generate the project's skeleton, you will need to modify the *FooBarClient.php* file by updating the factory method, adding a constructor if needed, and adding any class properties.

+--------------------------------------+------------------------------------------------------------------------------------------------------------------+
| src/FooBar/Command/                  | Place all of the commands for your web service in this folder.                                                   |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------+
| src/FooBar/FooBarClient.php          | Rename this class to the CamelCase name of the web service you are implementing followed by ``Client``.  Use     |
|                                      | strict CamelCasing (e.g. Xml is correct, XML is not).  A good client name would be something like                |
|                                      | ``FooBarClient.php``.                                                                                            |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------+
| phpunit.xml.dist                     | Different developers will configure their development environment differently.  A phpunit.xml file is required   |
|                                      | to run PHPUnit tests against your service.  ``phpunit.xml.dist`` provides a template for developers to copy and  |
|                                      | modify.                                                                                                          |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------+
| tests/                               | Place your ``bootstrap.php`` file in this folder.  The boostrap.php file is responsible for setting up PHPUnit   |
|                                      | tests.                                                                                                           |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------+
| tests/mock                           | Place mock response files in this folder.  Mock response files include HTTP response headers and the entity body |
|                                      | of a response.  Keep each HTTP response in a separate file.                                                      |
+--------------------------------------+------------------------------------------------------------------------------------------------------------------+

Create a client
---------------

Now that the directory structure is in place, you can start creating your web service client.  Rename Client.php to the CamelCase name of the web service you are interacting with.  Next you will need to create your client's constructor.  Your client's constructor can require any number of arguments that your client needs.  In order for a ServiceBuilder to create your client using a parameterized array, you'll need to implement a ``factory()`` method that maps an array of parameters into an instantiated client object.  Any class composition should be handled in your client's factory method.

.. note::

    Your client will not work with a service builder if you do not create a factory method.

Let's start creating a custom web service client.  First we will extend the ``Guzzle\Service\Client`` class.  Next we will create a constructor that accepts several web service specific arguments.  After creating your constructor, you must create a factory method that accepts an array of configuration data.  The factory method accepts parameters, adds default parameters, validates that required parameters are present, creates a new client, attaches any observers needed for the client, and returns the client object::

    <?php

    namespace Guzzle\MyService;

    use Guzzle\Service\Inspector;
    use Guzzle\Service\Client;

    /**
     * My example web service client
     */
    class MyServiceClient extends Client
    {
        /**
         * @var string Username
         */
        protected $username;

        /**
         * @var string Password
         */
        protected $password;

        /**
         * Factory method to create a new MyServiceClient
         *
         * @param array|Collection $config Configuration data. Array keys:
         *    base_url - Base URL of web service
         *    scheme - URI scheme: http or https
         *  * username - API username
         *  * password - API password
         *
         * @return MyServiceClient
         */
        public static function factory($config)
        {
            $default = array(
                'base_url' => '{{scheme}}://{{username}}.test.com/',
                'scheme' => 'https'
            );
            $required = array('username', 'password', 'base_url');
            $config = Inspector::prepareConfig($config, $default, $required);

            $client = new self(
                $config->get('base_url'),
                $config->get('username'),
                $config->get('password')
            );
            $client->setConfig($config);

            return $client;
        }

        /**
         * Client constructor
         *
         * @param string $baseUrl Base URL of the web service
         * @param string $username API username
         * @param string $password API password
         */
        public function __construct($baseUrl, $username, $password)
        {
            parent::__construct($baseUrl);
            $this->username = $username;
            $this->password = $password;
        }
    }

The ``Inspector::prepareConfig`` method is responsible for adding default parameters to a configuration object and ensuring that required parameters are in the configuration.   The factory method in the above example will be very similar to the code you will need in your client's factory method.  Any object composition required to build the client should be added in the factory method (for example, attaching event observers to the client based on configuration settings).

Miscellaneous helper methods for your web service can also be put in the client.  For example, the Amazon S3 client has methods to create a signed URL.

Create commands
---------------

Commands can be created in one of two ways: create a concrete command class that extends ``Guzzle\Service\Command\AbstractCommand`` or :doc:`create a dynamic command based on a service description </guide/service/creating_dynamic_commands>`.  We will describe how to create concrete commands below.

Commands help to hide complexity
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Commands are the method in which you abstract away the underlying format of the requests that need to be sent to take action on a web service.  Commands in Guzzle are meant to be built by executing a series of setter methods on a command object.  Commands are only validated when they are being executed.  A ``Guzzle\Service\Client`` object is responsible for executing commands.  Commands created for your web service must implement ``Guzzle\Service\Command\CommandInterface``, but it's easier to extend the ``Guzzle\Service\Command\AbstractCommand`` class and implement the ``build()`` method.  The ``build()`` method is responsible for using the arguments of the command to build one or more HTTP requests.

Docblock annotations for commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The required parameters of a command are validated based on docblock annotations on the command class.  Docblock annotations are also responsible for adding default parameters, setting static parameters on a command that cannot be changed, and enforcing type safety on different command parameters::

    <?php

    namespace Guzzle\MyService\Command;

    use Guzzle\Service\Command\AbstractCommand;

    /**
     * Sends a simple API request to an example web service
     *
     * @guzzle key doc="Destination object key" required="true"
     * @guzzle headers doc="Headers to set on the request" type="class:Guzzle\Common\Collection"
     * @guzzle other_value static="static value"
     */
    class Simple extends AbstractCommand
    {
        // ...
    }

In the above example, we are creating a simple command to send a web service request.  Docblock annotations for commands start with the ``@guzzle`` token.  The next token in is the parameter name (you must use snake_case parameter names).  After the @guzzle token and parameter name are a series of optional attributes.  These attributes are as follows:

===============  =================================================================  =============================================================
Attribute        Description                                                        Example
===============  =================================================================  =============================================================
``type``         Type of variable (array, boolean, class, date, enum, float,        ``@guzzle key type="class:Guzzle\Common\Collection"``
                 integer, regex, string, timestamp).  Some type commands accept
                 arguments by separating the type and argument with a colon         ``@guzzle key type="array"``
                 (e.g. enum:lorem,ipsum).
``required``     Whether or not the argument is required.  If a required parameter  ``@guzzle key required="true"`` or
                 is not set and you try to execute a command, an exception will be  ``@guzzle key required="false"``
                 thrown.
``default``      Default value of the parameter that will be used if a value is     ``@guzzle key default="default-value!"``
                 not provided before executing the command.
``doc``          Documentation for the parameter.                                   ``@guzzle key doc="This is the documentation"``
``min_length``   Minimum value length.                                              ``@guzzle key min_length="5"``
``max_length``   Maximum value length.                                              ``@guzzle key max_length="15"``
``static``       A value that cannot be changed.                                    ``@guzzle key static="this cannot be changed"``
``prepend``      Text to prepend to the value if the value is set.                  ``@guzzle key prepend="this_is_added_before."``
``append``       Text to append to the value if the value is set.                   ``@guzzle key append=".this_is_added_after"``
``filters``      CSV list of functions or static functions that modifies a string   ``@guzzle key filters="strtoupper,strrev"``
===============  =================================================================  =============================================================

When a command is being prepared for execution, the docblock annotations will be validated against the arguments present on the command.  Any default values will be added to the arguments, and if any required arguments are missing, an exception will be thrown.

As a general rule, most of the options for a command should essentially translate to an array key that the ``build()`` method takes into account when creating requests.  These keys should be specified in the docblock of the command's class header, and an end-developer should be able to set these values using setter methods with helpful docblocks or by passing the values to the command as an array.  This might not always be possible if you are building a complex command, but not allowing options to be set by array key in this manner will prevent end-developers from being able to use some shortcuts when calling your command (e.g. ``$client->getCommand('test', array('key' => 'value'));``).

Commands can turn HTTP responses into something more valuable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Commands can turn HTTP responses into something more valuable for your application.  After a command is executed, it calls the ``process()`` method of the command.  The AbstractCommand class will automatically set the result of the command to SimpleXMLElement if the response received by the command has a Content-Type of ``application/xml`` or an array if the Content-Type is ``application/json``.  If you want to provide more valuable results from your commands, you can override the ``process()`` method and return any value you want.  To help developers who use code completion, be sure to update the ``@return`` annotation of your ``getResult()`` method if you return a custom result (this will require you to override the ``getResult()`` method too)::

    <?php

    namespace Guzzle\MyService\Command;

    use Guzzle\Service\Command\AbstractCommand;

    /**
     * Sends a simple API request to an example web service
     *
     * @guzzle key doc="Destination object key" required="true"
     * @guzzle headers doc="Headers to set on the request" type="class:Guzzle\Common\Collection"
     * @guzzle other_value static="static value"
     */
    class Simple extends AbstractCommand
    {
        /**
         * Set the destination key
         *
         * @param string $key Destination key that will be added to the path
         *
         * @return Simple
         */
        public function setKey($key)
        {
            return $this->set('key', $key);
        }

        protected function build()
        {
            $this->request = $this->client->get('/{{key}}', $this);
            $this->request->setHeader('X-Header', $this->get('other_value'));
        }

        protected function process()
        {
            $this->result = new AwesomeObject($this->getResponse());
        }

        /**
         * {@inheritdoc}
         * @return AwesomeObject
         */
        public function getResult()
        {
            return parent::getResult();
        }
    }

There's our implemented command.  The ``build()`` method is responsible for creating an HTTP request to send to the web service.  This command will send a request to a web service that uses the ``key`` parameter as part of the path of the request, and adds an ``X-Header`` header value to the request using the ``other_value`` parameter of the command.  Parameters passed to a command can be referenced by calling ``$this->get($parameterName)`` or ``$this[$parameterName]``.  This command will return an ``AwesomeObject`` when the ``getResult()`` method is called on the command.  We are overriding the ``getResult()`` method in our command so that developers who use code completion will know what type of object is returned from the command.  You will notice that there are setter methods on the client for setting the keys referenced in the docblock.  These are strongly encouraged to help developers to quickly use your command with code completion.  You can also do fancy stuff to the values provided to setter methods, like creating objects or extra validation.  There's no need to create a setter method for the ``headers`` key, as that is implicitly managed by the ``AbstractCommand`` object.

Here's how you would execute this command using the client we created::

    <?php

    // Create your client using the factory method (use a service builder in your production app)
    $client = MyServiceClient::factory(array(
        'username' => 'test',
        'password' => 'shh!secret'
    ));

    $command = client->getCommand('simple');
    $command->setKey('test');

    // Result will be an instance of Awesomeobject
    $result = $client->execute($command);

    // You can also get the result of the command by calling getResult
    $result = $command->getResult();

Iterating over pages of results
-------------------------------

Some web services return paginated results.  For example, a web service might return the total number of results and a subset of the results in an API response.  Guzzle provides a couple of helpful classes that make it easy to work with web services that implement this type of result pagination.

The ``Guzzle\Service\ResourceIterator`` class should be used when dealing with results that can be iterated through by using some type of pagination controls like incrementing a page number or retrieving a list of resources using a next token returned from a web service.  You will need to extend the ResourceIterator class and implement the ``sendRequest()`` method that is responsible for sending a subsequent request when the results of the current page of resources is exhausted.  The ``sendRequest`` method is responsible for sending a request to fetch the next page of results and configuring the internal state of the iterator to begin iterating over the newly fetched results.  You will need to create a concrete command that instantiates your extended ResourceIterator in the command's ``process`` method.  Returning a ResourceIterator from a command object will help developers easily interact with a paginated result set-- all a developer needs to do is ``foreach`` over the result object, and every single resource from the API will be returned.

You might want to retrieve more than one page of results but not necessarily every page of results from a ResourceIterator.  In this case, you should allow end-developers to set a limit parameter on your command.  A limit parameter can be added to a ResourceIterator so that the iterator will not retrieve more resources than the limit amount.  For example, if you are retrieving 10 resources per page and your limit is set to 15, the resource iterator will retrieve a page of 10 resources followed by a page of 5 resources so that it will stay under the limit.  It is not guaranteed that the limit will limit the results to exactly the limit amount as this is dependent on the web service honoring the limit.

See ``Guzzle\Aws\S3\Model\BucketIterator`` and ``Guzzle\Aws\SimpleDb\Model\SelectIterator`` for examples of building resource iterators.

Unit test your service
----------------------

Unit testing a Guzzle web service client is not very difficult thanks to some of the freebies you get from the ``Guzzle\Tests`` namespace.  You can set mock responses on your requests or send requests to the test node.js server that comes with Guzzle.

You can learn more about unit testing guzzle web service clients by reading the :doc:`Unit testing web service clients </guide/service/testing_clients>` guide.
