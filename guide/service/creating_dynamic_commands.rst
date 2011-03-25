=========================
Creating Dynamic Commands
=========================

Guzzle allows you to create commands for your web service client based on an XML description of what your command should do.  As seen in :doc:`Building Web Service Clients </tour/building_services>`, Guzzle web service clients execute commands on a web service.  You can build concrete commands for every action you want to take on a web service, but you might find yourself writing a lot of boiler plate code if the actions you take on a web service are relatively the same, but just use a different path or method.  If this is the case for your client, you should consider utilizing Guzzle's dynamic commands based on XML descriptions.

Creating an XML service description
-----------------------------------

Let's say you're interacting with a web service that allows for the following routes and methods::

    GET/POST /users
    GET/PUT/DELETE /users/:id
    GET/POST /comments
    GET/PUT/DELETE /comments/:id

This looks like a fairly simple API.  The following XML service description implements this simple web service using nothing but XML:

*/path/to/client.xml*

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <client>
        <commands>
            <command name="get_users" method="GET" path="/users">
                <doc>Get a list of users</doc>
            </command>
            <command name="create_user" method="POST" path="/users">
                <doc>Create a user</doc>
                <param name="data" type="string" location="body" doc="User XML" />
            </command>
            <command name="delete_user" method="DELETE" path="/users/{{id}}">
                <doc>Delete a user by ID</doc>
                <param name="id" type="string" required="true" location="path" />
            </command>
            <command name="get_user" method="GET" path="/users/{{id}}">
                <param name="id" type="string" required="true" location="path" />
            </command>
            <command name="update_user" method="PUT" path="/users">
                <doc>Update a user</doc>
                <param name="id" type="string" required="true" location="path" />
                <param name="data" type="string" location="body" doc="User XML" />
            </command>
            <command name="get_comments" method="GET" path="/comments">
                <doc>Get a list of comments</doc>
            </command>
            <command name="create_comment" method="POST" path="/comments">
                <doc>Create a comment</doc>
                <param name="data" type="string" location="body" doc="Comment XML" />
            </command>
            <command name="get_comment" method="GET" path="/comments/{{id}}">
                <doc>Get a comment by ID</doc>
                <param name="id" type="string" required="true" location="path" />
            </command>
            <command name="update_comment" method="PUT" path="/comments/{{id}}">
                <doc>Update a comment</doc>
                <param name="id" type="string" required="true" location="path" doc="Comment ID" />
                <param name="data" type="string" location="body" doc="Comment XML" />
            </command>
            <command name="delete_comment" method="DELETE" path="/comments/{{id}}">
                <doc>Delete a comment by ID</doc>
                <param name="id" type="string" required="true" location="path" />
            </command>
        </commands>
    </client>

If you attach this service description to a client, you would be able to execute commands by name:

.. code-block:: php

    <?php

    $command = $client->getCommand('delete_comment', array(
        'id' => 123
    ));

    $httpResponse = $client->execute($command);

Adding an XML service description to a client
---------------------------------------------

You should add the service description to your client in the client's factory method using a ``Guzzle\Service\Description\XmlDescriptionBuilder``.

.. code-block:: php

    <?php

    namespace Guzzle\Service\MyService;

    use Guzzle\Common\Inspector;
    use Guzzle\Http\Message\RequestInterface;
    use Guzzle\Service\Client;
    use Guzzle\Service\Description\XmlDescriptionBuilder;

    /**
     * My example web service client
     *
     * @author My name <my_email@domain.com>
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
         *  * username - API username
         *  * password - API password
         *
         * @return MyServiceClient
         */
        public static function factory($config)
        {
            $config = Inspector::prepareConfig($config, array(
                'base_url' => 'http://www.test.com/',
            ), array('username', 'password', 'base_url'));

            $client = new self(
                $config->get('base_url'),
                $config->get('username'),
                $config->get('password')
            );
            $client->setConfig($config);

            // Add the XML service description to the client
            $builder = new XmlDescriptionBuilder('/path/to/client.xml');
            $client->setDescription($builder->build());

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

How to write an XML service description
---------------------------------------

XML service descriptions are stored in a separate XML file for each web service client.  The XML file should be stored in the same location as the client and, by convention, named the same as the web service client name with a .xml file extension (e.g. MyServiceClient.php => MyService.xml).  The root node of the XML file must be ``<client>``.  The ``<client>`` node must contain one or more ``<command>`` nodes which define each dynamic command that can be sent by the client.

Define commands using ``<command>`` nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Commands are defined using ``<command>`` nodes.  A command node is a single command that can be executed on a web service.  Command nodes can either reference a concrete command class that will receive extra parameters from the command node or be a completely dynamic command that builds an HTTP request based on the command definition.

Dynamic commands
^^^^^^^^^^^^^^^^

Dynamic commands are commands that build HTTP requests completely based on the command definition and do not require a concrete command class.  Dynamic command nodes require the following attributes:

+-----------+----------------------------------------------------------------------+
| Attribute | Description                                                          |
+===========+======================================================================+
|  name     | The key used to reference the command.  Please use snake_casing.     |
+-----------+----------------------------------------------------------------------+
|  method   | The HTTP method the command will execute (GET, HEAD, DELETE, POST,   |
|           | PUT, OPTIONS).                                                       |
+-----------+----------------------------------------------------------------------+
|  path     | The path of the request (e.g. ``/path/to/users``).  The path can be  |
|           | absolute or relative.  A relative path will append to the path set   |
|           | on the base_url of the service.  The path attribute can contain      |
|           | ``{{key_name}}`` injection points, where ``key_name`` is a parameter |
|           | in the command with a location of ``path``.                          |
+-----------+----------------------------------------------------------------------+
|  extends  | Extend a previously defined command in the same XML description to   |
|           | inherit every attribute of the parent command including params.  Any |
|           | settings specified in the chile command will override settings from  |
|           | inherited from the parent.                                           |
+-----------+----------------------------------------------------------------------+

.. code-block:: xml

    <command name="my_command" method="GET" path="/path/to/users">

``<command>`` nodes can contain an optional ``<doc>`` node that describes what the command does.

.. code-block:: xml

    <command name="my_command" method="GET" path="/path/to/users">
        <doc>Documentation about the command</doc>
    </command>

``<param>`` nodes are used within ``<command>`` nodes to specify each parameter that the command will take into account when building an HTTP request.  Param nodes can contain the following attributes:

===============  =================================================================  ===========================================
Attribute        Description                                                        Example
===============  =================================================================  ===========================================
location         The location in which the parameter will be added to the           ``location="path"`` or
                 generated request.                                                 ``location="header:X-Header"``
type             Type of variable (array, boolean, class, date, enum, float,        ``type="class:Guzzle\Common\Collection"``
                 integer, regex, string, timestamp).  Some type commands accept
                 arguments by separating the type and argument with a colon         ``type="array"``
                 (e.g. enum:lorem,ipsum).
required         Whether or not the argument is required.  If a required parameter  ``@guzzle key required="true"`` or
                 is not set and you try to execute a command, an exception will be  ``@guzzle key required="false"``
                 thrown.
default          Default value of the parameter that will be used if a value is
                 not provided before executing the command.                         ``default="default-value!"``
doc              Documentation for the parameter.                                   ``doc="This is the documentation"``
min_length       Minimum value length.                                              ``min_length="5"``
max_length       Maximum value length.                                              ``max_length="15"``
static           A value that cannot be changed.                                    ``static="this cannot be changed"``
prepend          Text to prepend to the value if the value is set.                  ``prepend="this_is_added_before."``
append           Text to append to the value if the value is set.                   ``append=".this_is_added_after"``
===============  =================================================================  ===========================================

The **location** attribute can be one of the following values:

+---------+------------------------------------------------------------------------------------------------+
| path    | Specifies the parameter as one that will inject into the ``path`` attribute of the command     |
+---------+------------------------------------------------------------------------------------------------+
| query   | Sets a query string value using the key and value of the parameter.  A custom query string key |
|         | can be used by providing the custom key after the query location separated by a colon          |
|         | (e.g. ``location="query:QueryKey``)"                                                           |
+---------+------------------------------------------------------------------------------------------------+
| header  | The parameter will be added as a header.  The header will be set as the name of the parameter  |
|         | or you can specify a custom header by providing the custom header after the header location    |
|         | separated by a colon (e.g. ``location="header:X-Custom-Header"``)                              |
+---------+------------------------------------------------------------------------------------------------+
| body    + The parameter value will be used as the body of the generated HTTP request                     |
+---------+------------------------------------------------------------------------------------------------+
| data    | This is the default location of parameters that do not contain a location attribute            |
+---------+------------------------------------------------------------------------------------------------+

Concrete commands
^^^^^^^^^^^^^^^^^

Concrete commands pass the values specified in ``<param>`` nodes to concrete command objects.  This is useful if you want to create an abstracted concrete command that accepts a collection of parameters that it uses to build a request but still allows for custom response processing so that the command can return a valuable result.  Concrete commands require a ``class`` attribute that references the class name to instantiate when the command is created.  The class attribute can use the PHP namespace separator or periods for namespace separators (e.g. both ``Guzzle.Service.Command.ClosureCommand`` and ``Guzzle\Service\Command\ClosureCommand`` are acceptable). Concrete command nodes don't use a ``method`` or ``path`` attribute; however, these parameters can be specified as ``<param>`` nodes which will be passed to the concrete command as parameters.

.. code-block:: xml

    <command name="my_concrete_command" class="Guzzle.Service.MyService.Command.DefaultDynamicCommand">
        <doc>Execute a command on the web service using a concrete class</doc>
        <param name="path" type="string" doc="Path of request" />
        <param name="method" type="enum:GET,HEAD,DELETE,POST,PUT,OPTIONS" doc="HTTP method. One of GET, HEAD, DELETE, PUT, POST, or OPTIONS" />
        <param name="other_data" type="string" doc="Give me something" />
        <param name="static_setting" static="dynamic" doc="This setting cannot be changed cause it is static" />
    </command>

This example command will instantiate a ``Guzzle\Service\MyService\Command\DefaultDynamicCommand`` when it is executed from a client (e.g. ``$client->getCommand('my_concrete_command')->execute()``).  The instantiated command will receive the ``<param>`` node values as a ``Guzzle\Common\Collection`` object that it can use to build an HTTP request.

Use custom ``<types>`` for data validation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Custom types can be registered to create shortcut references to type implementations or custom type classes that can be registered with the ``Guzzle\Common\Inspector`` class.  The ``<client>`` node can contain a ``<types>`` node which contains one or more ``<type>`` nodes.

You can use the ``type`` attribute on command parameters to enforce parameter values match a certain filter or are of a certain type.  For example, you could create a command parameter that must match a regular expression using the following snippet of code:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <client>
        <command name="example_command" method="GET" path="/{{username}}">
            <param name="my_parameter" type="regex:/[0-9a-zA-z_\-]+/" location="path" />
        </command>
    </client>

When an end-developer creates this command, they will need to pass a value that matches the ``/[0-9a-zA-z_\-]+/`` regular expression.  If a supplied parameter does not match this regular expression, an exception will be thrown.  If you use this same pattern in various parts of your XML service description, then you could create a shortcut ``<type>`` node and reference your custom type in each command.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <types>
        <type name="username" class="Guzzle.Common.InspectorFilter.Regex" default="/[0-9a-zA-z_\-]+/" />
    </types>
    <client>
        <command name="example_command" method="GET" path="/{{username}}">
            <param name="my_parameter" type="username" location="path" />
        </command>
    </client>

Use Dynamic and Concrete Commands
---------------------------------

Concrete commands are much better suited for interacting with complex web services or dealing with custom entity bodies that must be generated based on command parameters.  Never fear-- web service clients can utilize both concrete and dynamic commands.  When retrieving a command by name--

.. code-block:: php

    <?php

    $command = $client->getCommand('command_name');

The client will first check if it has a service description and if the service descriptions has a command defined by the name of 'command_name.'  If the client has a dynamic command named 'command_name', then a dynamic command will be created and returned.  If the client does not have a service description or its service description does not have a command defined by that name, it will see if a concrete command class maps to that name.  If it does, it will create the concrete command class and return it.  Whether or not the command is a concrete command or dynamic command doesn't matter to the end-developer as long as he/she can execute the command and get back a valuable response.

Caveats with service descriptions
---------------------------------

No code completion
~~~~~~~~~~~~~~~~~~

Dynamic commands cannot be directly instantiated and don't have setter methods to easily build up a command.  This means that you won't get code completion.

No custom response processing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Dynamic commands can't perform any type of response processing to create more valuable command results.  This means, for example, that the result of a command to retrieve a user will return a ``\SimpleXMLElement``, not a ``User`` object.

Weak support for custom entity bodies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

XML service descriptions are great for creating relatively simple commands, but you may have noticed that the PUT/POST commands in the example XML service description force the end-developer to build an XML entity body from scratch to send with those requests.  This is fine for relatively simple clients, but a better way of implementing these entity enclosing requests is to create models for the bodies of the commands and force end-developers to set the data parameter using a model by specifying the class in the parameter's type attribute:

.. code-block:: xml

    <!-- ... -->
    <command name="create_user" method="POST" path="/users">
        <doc>Create a user</doc>
        <param name="data" type="class:Guzzle\Service\MyService\Model\User" location="body" doc="User model" />
    </command>
    <command name="update_user" method="PUT" path="/users">
        <doc>Update a user</doc>
        <param name="id" type="string" required="true" location="path" />
        <param name="data" type="class:Guzzle\Service\MyService\Model\User" location="body" doc="User model" />
    </command>
    <command name="create_comment" method="POST" path="/comments">
        <doc>Create a comment</doc>
        <param name="data" type="class:Guzzle\Service\MyService\Model\Comment" location="body" doc="Comment model" />
    </command>
    <command name="update_comment" method="PUT" path="/comments/{{id}}">
        <doc>Update a comment</doc>
        <param name="id" type="string" required="true" location="path" doc="Comment ID" />
        <param name="data" type="class:Guzzle\Service\MyService\Model\Comment" location="body" doc="Comment model" />
    </command>
    <!-- ... -->