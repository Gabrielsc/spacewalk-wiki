## Core components

osad 

 Client-side service written in Python that responds to pings and runs rhn_check when told to by osa-dispatcher.

osa-dispatcher
 Server-side service written in Python that determines when an osad instance needs to be pinged or run rhn_check and sends a message telling them to do so.

jabberd 
 A daemon that implements the XMPP protocol defined in RFC 3920 and 3921. osad and osa-dispatcher both connect to this daemon. It handles authentication as well.
### What do they do?



osa-dispatcher periodically runs a query that checks to see if there are any clients overdue for a ping. If there are, it sends a message through jabberd to the osad instances running on the clients it needs responses from. The osad instances respond to the ping through the jabberd server. osa-dispatcher receives the response, and marks the client as 'online'. If osa-dispatcher doesn't receive a response from an osad instance in a certain amount of time, it is marked as offline.

The osa-dispatcher daemon also periodically performs a select within the database to see if any clients have any actions they need to perform. If there are, it sends a message through jabberd to osad telling it to run rhn_check on the client. rhn_check then takes over performing the actual action.

RHN Proxies have have a jabberd server running on them that will pass all messages it receives up to the jabberd server running on the satellite.
### Key Files for osad

The source code for osad lives in /usr/share/rhn/osad/ on the clients.


The configuration files are:
 /etc/sysconfig/rhn/osad.conf
 /etc/sysconfig/rhn/up2date

The actual script that is run by the service scripts for osad is /usr/sbin/osad.

Log files are placed in /var/log/osad by default, but the location is configurable in osad.conf.

osad is installed by the osad-*.rpm package
### Key Files for osa-dispatcher



The python code is in /usr/share/rhn/osad/ on the satellite.

osa-dispatcher is configured in the /etc/rhn/rhn.conf file, in the section titled "OSA Configuration".
Example entries

    # grep osa /etc/rhn/rhn.conf
    osa-dispatcher.jabber_server = yourspacewalk.example.org
    osa-dispatcher.osa_ssl_cert = /var/www/html/pub/RHN-ORG-TRUSTED-SSL-CERT
    osa-dispatcher.jabber_username = rhn-dispatcher-sat
    osa-dispatcher.jabber_password = rhn-dispatcher-a44631
    # 

The script run by the service commands is located at /usr/sbin/osa-dispatcher.

osa-dispatcher logs are dumped in /var/log/rhn/osa-dispatcher.log

Use osa-dispatcher.debug = X within the /etc/rhn/rhn.conf file to increase debug logging

osa-dispatcher is installed by the osa-dispatcher-*.rpm package and should be included on Spacewalk when it is installed.
### Key Files for jabberd

There are actually 6 programs that run when jabberd is started. Their executable are located in the following locations:

 /usr/bin/c2s
 /usr/bin/jabberd
 /usr/bin/resolver
 /usr/bin/router
 /usr/bin/s2s
 /usr/bin/sm

c2s handles communication between a client and the jabber server.
 config file: /etc/jabberd/c2s.xml
s2s handles communication with another jabber server.
 config file: /etc/jabberd/s2s.xml
sm manages sessions between the jabber components and the jabber router.
 config file: /etc/jabberd/sm.xml
router controls what component each message gets routed to.
 config file: /etc/jabberd/router.xml
resolver handles hostname resolution for the s2s component.
 config file: /etc/jabberd/resolver.xml
c2s and sm are configured to encrypt information that passes through them. The cert and key used to do this are located in the /etc/jabberd/server.pem file.

By default, all jabberd components append logging information to /var/log/messages.

The Berkeley database that jabberd uses is located in /var/lib/jabberd/. 
## Troubleshooting & Debugging

### Command-line usage


Osad can be run from the command line like this:


    /usr/sbin/osad -N -v -v -v -v

 The -N option forces osad to run in the foreground. 
 The -v option increases the verbosity of the output. 
 This is useful for seeing what exactly osad is doing, when it is doing it, and what errors are popping up.

The best way to figure out what is going on is to run osad on the clients in the foreground as described above, and tail the logfiles for osa-dispatcher and the jabberd components. When that is done, schedule some actions (package installs, remote commands, errata updates, etc.) and watch for errors that get generated. It can often be useful to compare to a known 'good' system as well, in both log files and configuration files. 
### General Troubleshooting Tips



If you have issues with osa-dispatcher, jabberd, or clients connecting make absolutely sure that your Spacewalk and/or Proxy use FQDN for their hostnames and that they're properly listed in /etc/hosts.  Also make sure that your Spacewalk SSL certificate uses the FQDN.  Jabber is very fragile when it comes to hostnames.
### use ntpd on server/clients



The authentication of osad instances to a jabberd server is time dependent. If the time on the client system is more than 120 seconds off from the time on the server, the osad instance on the client will fail authentication. This can make testing osad in a vmware instance a little painful. This is pretty easy to spot in the logs for osad, there will be an error complaining about the time drift being too large. This is a security feature and is working as intended, even if it can be annoying at times.
### osad stops responding



In some cases, your firewall or network settings may cause osad's to think they're still connected when they're not. A number of possible solution for this are discussed in [the Jabber and OSAD](JabberAndOSAD) section.
### Berkeley DB corruption



There is an intermittent problem with the interaction between jabberd and the Berkeley database it uses by default. Occasionally (not seen much any more), the database will become “wedged” and the osads on the clients will not pick up scheduled actions. Pings, however, still work. When/if this occurs, the only known solution is to restart jabberd - this will automatically remove the files in /var/lib/jabberd/. 
### Berkeley DB Maintenance



For more on how to maintain and clean up the bundled Berkeley DB for Jabber, go here:
  * [Jabber Berkeley Database](JabberDatabase) Maintenance
### Jabber and OSAD client connection issues

[Jabber and OSAD client connection issues](JabberAndOSAD)

### How to (re)confgure OSAD with different backend
  * [Configuring Spacewalk's jabberd to use an SQLite backend](https://omg.dje.li/2017/02/configuring-spacewalks-jabberd-to-use-an-sqlite-backend/)
  * [Configuring Spacewalk's jabberd to use a PostgreSQL backend](https://omg.dje.li/2017/03/configuring-spacewalks-jabberd-to-use-a-postgresql-backend/)

### Resources

 * http://jabberd.jabberstudio.org/2/docs/

 * http://www.ietf.org/rfc/rfc3920.txt
 * http://www.ietf.org/rfc/rfc3921.txt