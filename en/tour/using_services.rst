================================================
Consuming web services using web service clients
================================================

.. highlight:: php

Guzzle's awesome HTTP support provides the raw materials needed to build robust web service clients.  Guzzle's service layer provides the glue needed to bring it all together.

Command based web service clients
---------------------------------

Command based web service clients help to hide the underlying implementation of an API by following the `command pattern <http://en.wikipedia.org/wiki/Command_pattern>`_ and giving a concrete class or service description to each action that can be taken on a web service.

Instantiating web service clients using a ServiceBuilder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The best way to instantiate Guzzle web service clients is to let Guzzle handle building the clients for you using a ServiceBuilder.  A ServiceBuilder is responsible for creating concrete client objects based on configuration settings and helps to manage credentials for different environments.

A ServiceBuilder can source information from an array, an XML file, SimpleXMLElement, or JSON file::

    use Guzzle\Service\Builder\ServiceBuilder;

    // Source service definitions from a JSON file
    $builder = ServiceBuilder::factory('services.json');

Clients are referenced using a customizable name you provide in your service definition.  The ServiceBuilder is a sort of multiton object-- it will only instantiate a client once and return that client for subsequent retrievals.  You can get a "throwaway" client (a client that is not persisted by the ServiceBuilder) by passing ``TRUE`` in the second argument of ``ServiceBuilder::get()``.

Here's an example of retrieving an Unfuddle client from your ServiceBuilder::
    $client = $builder->get('unfuddle');
    // You can also use the ServiceBuilder object as an array
    $client = $builder['unfuddle'];

Sourcing data from XML
^^^^^^^^^^^^^^^^^^^^^^

A ServiceBuilder can get information from an XML file or a SimpleXMLElement.  The XML file includes ``<service>`` elements that describe each web service client you will use.  Parameters need to be specified in each ``<service>`` element to tell a ``Guzzle\Service\Builder\ServiceBuilder`` object how to build the web service client.  Clients are given names which are handy for using multiple accounts for the same service or creating development clients vs. production clients.  Here's an example of a services.xml that uses several `Amazon Web Services <http://aws.amazon.com/>`_ clients and the `Unfuddle <http://www.unfuddle.com/>`_ web service:

.. code-block:: xml

    <?xml version="1.0" ?>
    <guzzle>

        <!-- You can optionally provide a list of XML file to include -->
        <includes>
            <include path="/path/to/other/services.xml" />
        </includes>

        <!-- You can optionally specify a ServiceBuilderInterface class to instantiate -->
        <class>MyService.Awesome.ServiceBuilder</class>

        <services>
            <!-- Abstract service to store AWS account credentials -->
            <service name="abstract.aws">
                <param name="access_key" value="12345" />
                <param name="secret_key" value="abcd" />
            </service>
            <!-- Amazon S3 client that extends the abstract client -->
            <service name="s3" classs="Guzzle\Aws\S3\S3Client" extends="abstract.aws">
                <param name="devpay_product_token" value="XYZ" />
                <param name="devpay_user_token" value="123" />
            </service>
            <service name="simple_db" class="Guzzle\Aws\SimpleDb\SimpleDbClient" extends="abstract.aws" />
            <service name="sqs" class="Guzzle.Aws.Sqs.SqsClient" extends="abstract.aws" />
            <!-- Unfuddle client ( "." in class names are converted to "\" )-->
            <service name="unfuddle" class="Guzzle.Unfuddle.UnfuddleClient">
                <param name="username" value="test-user" />
                <param name="password" value="my-password" />
                <param name="subdomain" value="my-subdomain" />
            </service>
        </services>
    </guzzle>

Let's dissect what's going on in the above XML file.  The first client defined, ``abstract.aws``, is an **abstract client** that can be used by other clients to share configuration values among a number of clients.  This can be useful when clients share the same username and password.

The next client is an Amazon S3 client.  Each ``<service>`` nodes must contain a ``class`` attribute that references the full class name of the client being created (you can substitute PHP's namespace separator, ``\``, with a period ``.``).  Client nodes can inherit parameters from other previously defined nodes.  The above Amazon S3 client is inheriting configuration settings from the abstract.aws client and adding `Amazon DevPay <http://aws.amazon.com/devpay/>`_ related parameters.  As you can see from the `Amazon SimpleDB <http://aws.amazon.com/simpledb/>`_ and `Amazon SQS <http://aws.amazon.com/sqs/>`_ clients, not all clients will require additional parameters.

Sourcing from an Array
^^^^^^^^^^^^^^^^^^^^^^

Web service clients can be defined using an array of data.::

    $builder = ServiceBuilder::factory(array(
        'aws' => array(
            'access_key' => 'xyz',
            'secret'     => 'abc'
        ),
        's3' => array(
            'class'   => 'Guzzle\\Aws\\S3\\S3Client',
            'extends' => 'aws',
            'params'  => array(
                'subdomain' => 'michael',
            ),
        ),
        'unfuddle' => array(
            'class'  => 'Guzzle\\Unfuddle\\UnfuddleClient',
            'params' => array(
                'username'  => 'test-user',
                'password'  => 'test-password',
                'subdomain' => 'test'
            )
        )
    ));

Sourcing from a JSON document
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The above array could be represented as a JSON array.  Note that ``includes`` and ``class`` are optional.

.. code-block:: javascript

    {
        "includes": ["/path/to/file.json"],
        "class": "Foo\Bar"
        "services": {
            "aws": {
                "access_key": "xyz",
                "secret": "abc"
            },
            "s3": {
                "class": "Guzzle\\Aws\\S3\\S3Client",
                "extends": "aws",
                "params": {
                    "subdomain": "michael"
                }
            },
            "unfuddle": {
                "class": "Guzzle\\Unfuddle\\UnfuddleClient",
                "params": {
                    "username": "test-user",
                    "password": "test-password",
                    "subdomain": "test"
                }
            }
        }
    }

Referencing other clients in parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If one of your clients depends on another client as one of its parameters, you can reference that client by name by enclosing the client's reference key in ``{ }``.

.. code-block:: javascript

    {
        "services": {
            "token": {
                "class": "My\Token\TokenFactory",
                "params": {
                    "access_key": "xyz"
                }
            },
            "client": {
                "class": "My\Client",
                "params": {
                    "token_client": "{token}",
                    "version": "1.0"
                }
            }
        }
    }

Using Client objects
--------------------

Web service clients are the central point of interaction with a web service.  They hold service configuration data and help to ready HTTP requests to be sent to a web service.  Web service clients don't know much about the service itself-- they just create requests and execute commands.  Configuration settings can be retrieved from a client by passing a configuration key to the ``getConfig()`` method of a client (e.g. ``$token = $client->getConfig('devpay_product_token')``).

Executing commands using a client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Commands are used to take action on a web service and format the response from the web service into something useful.  Commands can send single HTTP requests or send a complex series of requests to a web service.

Commands can be instantiated and configured by a client by calling the ``getCommand()`` method on a client and using the short form of a command's name.  The short form of a command's name is calculated based on the folder hierarchy of a command and converting the CamelCased named commands into snake_case.  Here are some examples on how the command names are calculated:

#. ``Guzzle\Aws\S3\Command\Bucket\ListBucket`` **->** bucket.list_bucket
#. ``Guzzle\Aws\S3\Command\GetAcl`` **->** get_acl
#. ``Guzzle\Unfuddle\Command\People\GetCurrentPerson`` **->** people.get_current_person

Notice how any sub-namespace beneath ``Command`` is converted from ``\`` to ``.`` (a period).  CamelCasing is converted to lowercased snake_casing (e.g. GetAcl == get_acl).

Here's how you would get the Amazon S3 client from the ServiceBuilder and execute a GetObject command to retrieve an object from Amazon S3::

    // Retrieve the client by name
    $client = $serviceBuilder['s3'];

    $command = $client->getCommand('bucket.get_bucket');
    $command->setBucket('mybucket')->setKey('mykey');

    // The result of the GetObject command returns a Guzzle\Http\Message\Response object
    $httpResponse = $client->execute($command);

    // Get the body of the Amazon S3 object
    echo $httpResponse->getBody();

The GetObject command just returns the HTTP response object when it is executed.  This is the default behavior of Guzzle commands unless specified otherwise in the docblock of the ``getResult()`` method of a specific command.  Commands don't have to just return the HTTP response; commands might return more valuable information when executed::

    // Get a command from the Amazon S3 client
    $command = $client->getCommand('bucket.list_bucket');
    $command->setBucket('mybucket');

    // Execute the command and get a BucketIterator object
    $objects = $client->execute($command);

    // Iterate over every single object in the bucket.  Subsequent requests
    // will be issued to retreive the next result of a truncated response.
    foreach ($objects as $object) {
        echo "{$object['key']} {$object['size']}\n";
    }

    // You can get access to the HTTP request issued by the command and the response
    echo $command->getRequest();
    echo $command->getResponse();

The ListBucket command above returns a ``Guzzle\Aws\S3\Model\BucketIterator`` which will iterate over the entire contents of a bucket.

You can take some shortcuts in your code by passing key-value pair arguments to a command::

    $objects = $client->getCommand('bucket.list_bucket', array('bucket' => 'my_bucket'))->execute();

Executing commands in parallel using CommandSets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Commands can be sent in parallel using ``Guzzle\Service\Command\CommandSet`` objects::

    $client = $serviceBuilder['simple_db'];
    $client->execute(array(
        $client->getCommand('get_attributes', array(
            'domain' => 'test',
            'item_name' => 'item1'
        )),
        $client->getCommand('get_attributes', array(
            'domain' => 'test',
            'item_name' => 'item2'
        )),
        $client->getCommand('delete_domain', array(
            'domain' => 'test_2'
        ))
    ));
    foreach ($set as $command) {
        echo $command->getName() . ': ' . $command->getResponse()->getStatusCode() . "\n";
    }

Guzzle doesn't require that all of the commands in a CommandSet originate from the same client.  This allows you to write extremely efficient code when you need to send several requests to multiple services::

    use Guzzle\Service\Command\CommandSet;

    // Get all of the commands from a registered client object
    $set = new CommandSet(array(
        $serviceBuilder['simple_db']->getCommand('get_attributes', array(
            'domain' => 'test',
            'item_name' => 'item1'
        )),
        $serviceBuilder['s3']->getCommand('bucket.head_bucket', array(
            'bucket' => 'my_bucket'
        )),
        $serviceBuilder['unfuddle']->getCommand('people.get_current_person'),
    ));

    $set->execute();

    foreach ($set as $command) {
        // Do something with the results of each command
        switch ($command->getName()) {
            case 'get_attributes':
                break;
            case 'bucket.head_bucket':
                break;
            case 'people.get_current_person':
                break;
        }
    }

Next steps
~~~~~~~~~~

Check the documentation of the web service client you are using to see the available commands for the client.  Some clients will mix :doc:`service descriptions </guide/service/service_descriptions>` with concrete commands, so will need to check if an XML or JSON service description is present.
