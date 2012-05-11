===
FAQ
===

What should I do if I get this error: Fatal error: Maximum function nesting level of '100' reached, aborting!
-------------------------------------------------------------------------------------------------------------

You could run into this error if you have the XDebug extension installed and you execute a lot of requests in callbacks.  This error message comes specifically from the XDebug extension. PHP itself does not have a function nesting limit. Change this setting in your php.ini to increase the limit:

xdebug.max_nesting_level = 1000

[`source <http://stackoverflow.com/a/4293870/151504>`_]
