==============================
Creating Plugins and Observers
==============================

.. highlight:: php

Guzzle is extremely extendable because of the behavioral modifications that can be added to requests, clients, and commands using an event system.  Before and after the majority of actions are taken in the library, an event is emitted with the name of the event and context surrounding the event.  Observers can subscribe to a subject and modify the subject based on the events received.  Guzzle's event system is the backbone of it's plugin architecture.

Overview
--------

Plugins need to implement the ``Guzzle\Common\Event\Observer`` interface, which simply requires the ``update(Subject $subject, $event, $context = null)`` method.  The ``update()`` method will receive the subject that emitted the event, the name of the event, and any context surrounding the event.  You are free to modify the subject of an event or the context of the event (if the context is an object).

Subscribing an observer to a subject
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Observers subscribe to subjects and receive every event emitted from the subject.  You can subscribe an instantiated observer to an event by getting the ``EventManager`` object from the subject and calling  its ``attach()`` method.::

    <?php

    $testPlugin = new TestPlugin();
    $testPriority = 0;
    $client->getEventManager()->attach($testPlugin, $testPriority);

Priorities
~~~~~~~~~~

A priority specifies the order in which an observer will be notified in relation to other observers.  The higher the priority, the sooner an observer will be notified.  Priority values can be any value you want, including large negative numbers to ensure that an observer is notified last (e.g. -1000).  By default, each observer is attached with an equal priority of 0.

Filtering events
~~~~~~~~~~~~~~~~

Observers will receive every event emitted by the subject.  It is up to the observer to filter out updates that do not pertain to the observer.  For example, if you are creating an observer that is only interested in the 'request.complete' event of a request object, then you can write something like this::

    <?php

    namespace Guzzle\Http\Plugin;

    use Guzzle\Common\Event\Observer;
    use Guzzle\Common\Event\Subject;
    use Guzzle\Http\Message\RequestInterface;

    class TestPlugin implements Observer
    {
        public function update(Subject $subject, $event, $context = null)
        {
            if ($event != 'request.complete') {
                return;
            }

            // Do something here
        }
    }

Events emitted by different subjects
------------------------------------

Observers can be attached to any object implementing the ``Guzzle\Common\Event\Subject`` interface.  As a standard coding practice in Guzzle, information about all events emitted from a subject will be described in the docblock of the class.

The following is a listing of some of the key objects that can be subscribed to (this is not an exhaustive list).

``Guzzle\Http`` events
~~~~~~~~~~~~~~~~~~~~~~

``Guzzle\Http\Message\Request``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

============================   =====================   ==================================
Event                          Context                 Description
============================   =====================   ==================================
request.bad_response           null                    Received non-success response
request.before_send            null                    About to send the request
request.complete               Response                Completed HTTP transaction
request.curl.after_create      CurlHandle              Created a CurlHandle
request.curl.before_create     null                    About to create a CurlHandle
request.curl.release           CurlHandle              About to release a CurlHandle
request.exception              RequestException        Encountered an exception sending
request.failure                BadResponseException    The request failed
request.receive.header         array                   Received response header
request.receive.status_line    array                   Received response status line
request.sent                   null                    Sent the request
request.set_response           Response                Manually set a response
request.success                Response                The request succeeded
============================   =====================   ==================================


``Guzzle\Http\Message\EntityEnclosingRequest``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

============================   =========   =================================
Event                          Context     Description
============================   =========   =================================
request.prepare_entity_body    null        About to prepare the entity body
============================   =========   =================================

``Guzzle\Http\Pool\Pool``
^^^^^^^^^^^^^^^^^^^^^^^^^

================   ==================   =======================================
Event              Context              Description
================   ==================   =======================================
add_request        RequestInterface     A request was added to the pool
remove_request     RequestInterface     A request was removed from the pool
reset              null                 The pool was reset
before_send        array                The pool is about to be sent
complete           array                The pool finished sending the requests
polling_request    RequestInterface     A request is still polling
polling            null                 Some requests are still polling
exception          RequestException     A request exception occurred
================   ==================   =======================================

``Guzzle\Service events``
~~~~~~~~~~~~~~~~~~~~~~~~~

``Guzzle\Service\Client``
^^^^^^^^^^^^^^^^^^^^^^^^^

Any event attached to a client object will automatically be attached to all requests created by the client.

====================   =================   ===============================
Event                  Context             Description
====================   =================   ===============================
command.after_send     CommandInterface    A command executed
command.before_send    CommandInterface    A command is about to execute
command.create         CommandInterface    A command was created
request.create         RequestInterface    Created a new request
====================   =================   ===============================

``Guzzle\Service\ResourceIterator``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

============  =========  =======================================================
Event         Context    Description
============  =========  =======================================================
before_send   array      About to issue another command to get more results
after_send    array      Issued another command to get more results
============  =========  =======================================================

``Guzzle\Service\ResourceIteratorApplyBatched``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

============  =========  =======================================================
Event         Context    Description
============  =========  =======================================================
before_batch  array      About to send a batch of requests to the callback
after_batch   array      Finished sending a batch of requests to the callback
============  =========  =======================================================

Examples of the event system
----------------------------

Signing requests for an API
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are creating a web service client for a web service that requires that your requests are signed in a specific way (e.g. Amazon Web Services), you can create a plugin that will handle signing the request before the request is sent to the web service.  This type of observer would need to be attached to a ``Guzzle\Service\Client`` object which in turn will attach it to all requests created through the client.  This observer would only be interested in the ``request.before_send`` event of a request object.  Using this event, you will be able to calculate the signature need for the request and add any required headers or query string parameters.

**Note: You will need to ensure that you attach this observer with a very low priority so that it is updated last.**

Here's a snippet of the Amazon S3 request signing plugin from the `guzzle-aws <https://github.com/guzzle/guzzle-aws>`_ repo::

    <?php
    //...
    public function update(Subject $subject, $event, $context = null)
    {
        if ($event == 'request.before_send') {
            $path = $subject->getResourceUri() ?: '';
            $headers = array_change_key_case($subject->getHeaders()->getAll());
            if (!array_key_exists('Content-Length', $headers)) {
                $headers['Content-Type'] = $subject->getHeader('Content-Type');
            }
            $canonicalizedString = $this->signature->createCanonicalizedString(
                $headers, $path, $subject->getMethod()
            );
            $subject->setHeader(
                'Authorization',
                'AWS ' . $this->signature->getAccessKeyId(). ':' . $this->signature->signString($canonicalizedString)
            );
        }
    }