# XML-RPC Handlers

Spacewalk exports a significant chunk of operations through its XML-RPC API. This page is intended to get you up and running with sample client scripts, and implement a sample XML-RPC call in one of the handlers. To find the list of supported API calls goto ApiDocs.

## Using the API



Suppose one wants to obtain a list of users in his/her organization, the org admin would typically generate a client script like this in python.

    #!python
    #!/usr/bin/env python
    import xmlrpclib
    
    SATELLITE_HOST = "<your satellite address>"
    SATELLITE_URL = "http://%s/rpc/api" % SATELLITE_HOST
    login = "admin"
    password = "password"
    
    #connect to the server
    client = xmlrpclib.Server(SATELLITE_URL, verbose=0)
    # get the session key
    key = client.auth.login(login, password)
    # get the list of users
    print client.user.list_users(key)
## Parts of the server side making up the call



Lets trace through the example call `client.user.list_users(key)`. There are two parts to this call,
`.user` is the XML-RPC name space used to group logical methods together. The other part is `list_users(key)` which is the method called in the name space.
Alternatively, methods can be specified in [camelCase](http://en.wikipedia.org/wiki/CamelCase) such as `listUsers(key)`. The frontend API knows how to parse both types of calls.

In case you are user and suspect there is a bug in some call you would,

 * ensure you are calling the call with correct parameters (correct count and type (integer/string/list...) of them
 * investigate `/var/log/tomcat6/catalina.out` log file for tracebacks (easy way is to `tail -f /var/log/tomcat6/catalina.out` and recreate the failure to see what appears there)

 NOTE: we need to expand the section below

In the case of a bug in this call you would,

 * look at /eng/java/code/src/com/redhat/rhn/frontend/xmlrpc/handler-manifest.xml to figure out the class handling the "user" name space `<template name="user" classname="com.redhat.rhn.frontend.xmlrpc.user.UserHandler" />`
 * look `listUsers(String sessionKey)` method in /eng/java/code/src/com/redhat/rhn/frontend/xmlrpc/user/UserHandler.java and trace the bug from there