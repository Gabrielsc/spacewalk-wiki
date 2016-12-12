
    #!div class="important" style="border: 2pt solid; text-align: center"
    '''''DEPRECATED, NO LONGER USED'''''


Here's my two cents.
# Languages

Most of the configuration management engines use yet another

language to the ones already used in Spacewalk/Satellite. Puppet for
example uses ruby. Support would mean learning yet another language,
creating more libraries, etc. I thought we where trying to reduce
the amount of languages?
# Conflicts

Also interfaces will have to be written to allow people to work with

the CME without breaking the required configuration of Spacewalk/
Satellite. Which would be a possibility e.g. if one routines put's a
file and the other changes it.
# Dependency chain

Projects like these are often in flux, which is good when that product

is the only thing you are going to use. If it wasn't you might be
better of not using it all together.

However if your code depends on functionality and configuration to
remain static this is another story.
# One way street



Most of the CME's don't allow for the checking or updating of a single
package, file or process. Getting reports on success is also tricky.

Whatever solution there might be the server needs to "know" what the
system looks like and what is supposed to look like. Using a CME is
more or less like a "fire and forget" way of managing a server, You're
"pretty sure" you know what is happening on the server.
# Conclusion

Yes there is a requirement to "Lay down the law" as it where. But the

spacewalk should be the sheriff not a CME.
# Proposal



I propose a to offer that what is required, but offer it in a way that is closer to the satellite/spacewalk's original design. 
## Func

[func](http://fedorahosted.org/func) and the regular file distribution

mechanism. Func is written in python and can be scripted, reducing actions to functions call's
allowing for extensive control on files, packages applications etc.. It's requirement of python
is neither an issue on server or client side since this requirement already exists in other
spacewalk/satellite tools.
## Pre-defined actions and triggers



A store of pre-defined calls and/or scripts should be available for (re)use later on. For example:

 * "Turn of and shutdown the foo service" (`chkconfig foo off; service foo stop`)
 * "Force update on `/etc/foo.conf`" (`rhncfg-client get /etc/foo.conf; service foo reload`)
 * "Reload the foo service"
 * "Halt server"
 * etc.

These actions can be linked to triggers. Which would mean that e.g. if `/etc/foo.conf` or
variables relevant to that file have changed "Force update on `/etc/foo.conf`" would be run.

Optionally add different types of triggers more or less like the "pre, post, prun, etc" switches in RPM. This way if a file or some of the related parameters have been changed the required actions can also be taken.
## Volatile Files



One of the most irritating (imho) part of the config channel way of working is that when I delete a file from a config channel it still exists on the server. Adding a "volatile" function would help.

If a file is removed from a channel and there is not other channel left offering that file and that file has been marked as "volatile", issue a file test + unlink of that file. The changes on that file should trigger any required actions.
## Evaluated file content



A (python based) macro language, home grown or preexisting like e.g.[mako](http://www.makotemplates.org/)
to be used in config files with variables that can be obtained from information in the server. This would add a switch to the file "evaluate" or "is template".

Using a set of predefined variables combined with internal variables linked to config channels would mean that drastic changes could be done in a controlled way.

A static image of the file would be available for file verification and caching purposes. Updating variables would automatically mean the static image is updated and that image is distributed. This way the server still "knows" what the file should look like.

--- John van Zantvoort (JYDawg)