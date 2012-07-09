====================
Service Descriptions
====================

Guzzle allows you to create commands for your web service client based on a document called a service description.  As seen in :doc:`Building Web Service Clients </tour/building_services>`, Guzzle web service clients execute commands on a web service.  You can build concrete commands for every action you want to take on a web service, but you might find yourself writing a lot of boiler plate code if the actions you take on a web service are relatively the same, but just use a different path or method.  If this is the case for your client, you should consider utilizing Guzzle's dynamic commands based on service descriptions.

Creating a service description
------------------------------

Service descriptions can be representing using a PHP array, XML document, or JSON document.

Let's say you're interacting with a web service that allows for the following routes and methods::

    GET/POST /users
    GET/PUT/DELETE /users/:id
    GET/POST /comments
    GET/PUT/DELETE /comments/:id

This looks like a fairly simple API.  The following XML service description implements this simple web service using nothing but XML:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <client>
        <commands>
            <command name="get_users" method="GET" uri="/users">
                <doc>Get a list of users</doc>
            </command>
            <command name="create_user" method="POST" uri="/users">
                <doc>Create a user</doc>
                <param name="data" type="string" location="body" doc="User XML" />
            </command>
            <command name="delete_user" method="DELETE" uri="/users/{{id}}">
                <doc>Delete a user by ID</doc>
                <param name="id" type="string" required="true" />
            </command>
            <command name="get_user" method="GET" uri="/users/{{id}}">
                <param name="id" type="string" required="true" />
            </command>
            <command name="update_user" method="PUT" uri="/users">
                <doc>Update a user</doc>
                <param name="id" type="string" required="true" />
                <param name="data" type="string" location="body" doc="User XML" />
            </command>
            <command name="get_comments" method="GET" uri="/comments">
                <doc>Get a list of comments</doc>
            </command>
            <command name="create_comment" method="POST" uri="/comments">
                <doc>Create a comment</doc>
                <param name="data" type="string" location="body" doc="Comment XML" />
            </command>
            <command name="get_comment" method="GET" uri="/comments/{{id}}">
                <doc>Get a comment by ID</doc>
                <param name="id" type="string" required="true" />
            </command>
            <command name="update_comment" method="PUT" uri="/comments/{{id}}">
                <doc>Update a comment</doc>
                <param name="id" type="string" required="true" doc="Comment ID" />
                <param name="data" type="string" location="body" doc="Comment XML" />
            </command>
            <command name="delete_comment" method="DELETE" uri="/comments/{{id}}">
                <doc>Delete a comment by ID</doc>
                <param name="id" type="string" required="true" />
            </command>
        </commands>
    </client>

If you attach this service description to a client, you would be able to execute commands by name.

.. code-block:: php

    use Guzzle\Service\Description\ServiceDescription;

    $description = ServiceDescription::factory('/path/to/client.xml');
    $client->setDescription($description);

    $command = $client->getCommand('delete_comment', array(
        'id' => 123
    ));

    $httpResponse = $client->execute($command);

.. note::

    You can add the service description to your client's factory method or constructor.

How to write an XML service description
---------------------------------------

XML service descriptions are stored in a separate XML file for each web service client.  The XML file should be stored in the same location as the client.  The root node of the XML file must be ``<client>``.  The ``<client>`` node must contain one or more ``<command>`` nodes which define each dynamic command that can be sent by the client.

Define commands using ``<command>`` nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Commands are defined using ``<command>`` nodes.  A command node is a single command that can be executed on a web service.  Command nodes can either reference a concrete command class that will receive extra parameters from the command node or be a completely dynamic command that builds an HTTP request based on the command definition.

Dynamic commands
^^^^^^^^^^^^^^^^

Dynamic commands are commands that build HTTP requests completely based on the command definition and do not require a concrete command class.  Dynamic command nodes utilize the following attributes:

+-----------+----------------------------------------------------------------------+
| Attribute | Description                                                          |
+===========+======================================================================+
|  name     | The key used to reference the command.  Use snake_casing.            |
+-----------+----------------------------------------------------------------------+
|  method   | The HTTP method the command will execute (GET, HEAD, DELETE, POST,   |
|           | PUT, OPTIONS).                                                       |
+-----------+----------------------------------------------------------------------+
|  uri      | The URI template of the request (e.g. ``/path/to/users``).  The path |
|           | can be absolute or relative.  A relative path will append to the path|
|           | set on the base_url of the service.  This attribute can contain      |
|           | ``{key_name}`` URI templates, where ``key_name`` is a parameter in   |
|           | command or set in the associated client's configuration data.        |
+-----------+----------------------------------------------------------------------+
|  extends  | Extend a previously defined command in the same XML description to   |
|           | inherit every attribute of the parent command including params.  Any |
|           | settings specified in the chile command will override settings from  |
|           | inherited from the parent.                                           |
+-----------+----------------------------------------------------------------------+
|  class    | Optional.  Specify a ``concrete command`` class that will be         |
|           | instantiated when the command is created.  This is useful for        |
|           | implementing complex response processing                             |
+-----------+----------------------------------------------------------------------+

.. code-block:: xml

    <command name="my_command" method="GET" uri="/path/to/users">

``<command>`` nodes can contain an optional ``<doc>`` node that describes what the command does.

.. code-block:: xml

    <command name="my_command" method="GET" uri="/path/to/users">
        <doc>Documentation about the command</doc>
    </command>

``<param>`` nodes are used within ``<command>`` nodes to specify each parameter that the command will take into account when building an HTTP request.  Param nodes can contain the following attributes:

===============  =================================================================  ===========================================
Attribute        Description                                                        Example
===============  =================================================================  ===========================================
location         The location in which the parameter will be added to the           ``location="query"`` or
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
filters          CSV list of functions or static functions that modifies a string   ``@guzzle key filters="strtoupper,strrev"``
===============  =================================================================  ===========================================

The **location** attribute can be one of the following values:

+------------+------------------------------------------------------------------------------------------------+
| query      | Sets a query string value using the key and value of the parameter.  A custom query string key |
|            | can be used by providing the custom key after the query location separated by a colon          |
|            | (e.g. ``location="query:QueryKey"``)                                                           |
+------------+------------------------------------------------------------------------------------------------+
| header     | The parameter will be added as a header.  The header will be set as the name of the parameter  |
|            | or you can specify a custom header by providing the custom header after the header location    |
|            | separated by a colon (e.g. ``location="header:X-Custom-Header"``)                              |
+------------+------------------------------------------------------------------------------------------------+
| body       | The parameter value will be used as the body of the generated HTTP request                     |
+------------+------------------------------------------------------------------------------------------------+
| data       | This is the default location of parameters that do not contain a location attribute            |
+------------+------------------------------------------------------------------------------------------------+
| post_field | Sets a post string value using the key and value of the parameter.  A custom post string key   |
|            | can be used by providing the custom key after the post location separated by a colon           |
|            | (e.g. ``location="post_field:FieldKey"``)                                                      |
+------------+------------------------------------------------------------------------------------------------+
| post_file  | Sets a path to file that should be send as post file                                           |
+------------+------------------------------------------------------------------------------------------------+

Use custom ``<types>`` for data validation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Custom types can be registered to create shortcut references to type implementations or custom type classes that can be registered with the ``Guzzle\Service\Inspector`` class.  The ``<client>`` node can contain a ``<types>`` node which contains one or more ``<type>`` nodes.

You can use the ``type`` attribute on command parameters to enforce parameter values match a certain filter or are of a certain type.  For example, you could create a command parameter that must match a regular expression using the following snippet of code:

.. code-block:: xml

    <client>
        <commands>
            <command name="example_command" method="GET" uri="/{{username}}">
                <param name="my_parameter" type="regex:/[0-9a-zA-z_\-]+/" />
            </command>
        </commands>
    </client>

When an end-developer creates this command, they will need to pass a value that matches the ``/[0-9a-zA-z_\-]+/`` regular expression.  If a supplied parameter does not match this regular expression, an exception will be thrown.  If you use this same pattern in various parts of your XML service description, then you could create a shortcut ``<type>`` node and reference your custom type in each command.

.. code-block:: xml

    <client>
        <types>
            <type name="username" class="Guzzle\Common\Validation\Regex" pattern="/[0-9a-zA-z_\-]+/" />
        </types>
        <commands>
            <command name="example_command" method="GET" uri="/{{username}}">
                <param name="my_parameter" type="username" />
            </command>
        </commands>
    </client>

Sending PUT and POST requests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Service descriptions allow for a flexible way to send PUT and POST requests where the entity body of the request needs to be in a specific format.  You may have noticed that the PUT/POST commands in the example XML service description force the end-developer to build an XML entity body from scratch.  A better way of implementing these entity enclosing requests would be to allow the end-developers set body parameters using a SimpleXMLElement object.  This can be achieved by using the "type" parameter type and specifying a class:

.. code-block:: xml

    <client>
        <commands>
            <command name="create_user" method="POST" uri="/users">
                <param name="data" type="type:SimpleXMLElement" location="body" />
            </command>
        </commands>
    </client>

If you are sending JSON data, you should consider allowing end-developers to set body parameters using an array.  You can then convert an array to a JSON string by using the ``filters`` attribute of a parameter:

.. code-block:: xml

    <client>
        <commands>
            <command name="create_user" method="POST" uri="/users">
                <param name="data" type="type:array" filters="json_encode" location="body" />
            </command>
        </commands>
    </client>

Including other service descriptions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can include other service descriptions in your service description files as long as the service description you are including uses the same format (e.g. XML can include XML and JSON can include JSON).

XML
^^^

.. code-block:: xml

    <client>
        <includes>
            <include path="/path/to/service.xml" />
            <include path="../../relative/path/to/service.xml" />
        </includes>
    </client>

JSON
^^^^

.. code-block:: javascript

    {
        "includes": [
            "/path/to/service.json",
            "../../relative/path/to/service.json"
        ]
    }

Creating JSON service descriptions
----------------------------------

You can create service descriptions using .json files.  The JSON document must match the following template:

.. code-block:: javascript

    {
        "includes": [],
        "types": {},
        "commands": {}
    }

We covered including other service descriptions in the previous section.  Adding custom types in a JSON description must include a class value and can include any number of custom arguments to pass to the Guzzle constraint object when it is called.

.. code-block:: javascript

    {
        "types": {
            "regex": {
                "class": "regex",
                "pattern": "/[A-Z]+/"
            }
        }
    }

Commands will follow this format:

.. code-block:: javascript

    {
        "commands": {
            "abstract": {
                "uri": "/",
                "class": "Service\Command\Default"
            },
            "concrete": {
                "extends": "abstract",
                "params": {
                    "test": {
                        "type": "string",
                        "required": true,
                        "filters": "strtolower"
                    }
                }
            }
        }
    }

Use Dynamic and Concrete Commands
---------------------------------

Web service clients can utilize both concrete and dynamic commands.  When retrieving a command by name (``$command = $client->getCommand('command_name')``), the client will first check if it has a service description and if the service descriptions has a command defined by the name of 'command_name.'  If the client has a dynamic command named 'command_name', then a dynamic command will be created and returned.  If the client does not have a service description or its service description does not have a command defined by that name, it will see if a concrete command class maps to that name.  If it does, it will create the concrete command class and return it.  Whether or not the command is a concrete command or dynamic command doesn't matter to the end-developer as long as the developer can execute the command and get back a valuable response.

Concrete commands
~~~~~~~~~~~~~~~~~

Concrete commands pass the values specified in ``<param>`` nodes to concrete command objects.  This is useful if you want to create an abstracted concrete command that accepts a collection of parameters that it uses to build a request but still allows for custom response processing so that the command can return a valuable result.  Concrete commands require a ``class`` attribute that references the class name to instantiate when the command is created.  The class attribute can use the PHP namespace separator or periods for namespace separators (e.g. both ``Guzzle.Service.Command.ClosureCommand`` and ``Guzzle\Service\Command\ClosureCommand`` are acceptable). Concrete command nodes don't use a ``method`` or ``uri`` attribute; however, these parameters can be specified as ``<param>`` nodes which will be passed to the concrete command as parameters.

This example command will instantiate a ``Guzzle\Service\MyService\Command\DefaultDynamicCommand`` when it is executed from a client (e.g. ``$client->getCommand('my_concrete_command')->execute()``).  The instantiated command will receive the ``<param>`` node values as a ``Guzzle\Common\Collection`` object that it can use to build an HTTP request.

Response processing
~~~~~~~~~~~~~~~~~~~

The default behavior of a command is to automatically set the result of a command to a SimpleXMLElement if the response received by the command has a Content-Type of ``application/xml`` or an array if the Content-Type is ``application/json``.

You can extend ``Guzzle\Service\Command\DynamicCommand`` and implement a custom ``process()`` method to leverage dynamically generated commands while still providing customized results to commands.  For example, you can use a ``get_user`` concrete command that generates an HTTP request based on a service description, but validates the HTTP response and sets the result of the command to an easy to use ``User`` object.
