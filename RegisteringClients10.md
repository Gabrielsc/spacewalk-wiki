# Registering Clients



Instructions for registering client systems you wish to manage with Spacewalk

These are instruction for Spacewalk 1.0. 

----
# Before Starting

  1. Create a base channel within Spacewalk (Channels > Manage Software Channels > Create New Channel)

  2. Create an activation key with the new base channel. When creating a registration key do not use the generate function, create a human-readable version. eg: fedora-server-channel. This makes your installation more understandable and provides greater logical consistency to the whole system. On the other hand, if you want to prevent people from getting access to your channels, letting Spacewalk to generate random activation key name is the best way to go.
## Note about rhnreg_ks --force



rhnreg_ks is used for registration of clients to Spacewalk. If you need to re-register a client to your Spacewalk server or change registration from one environment or server to another Spacewalk server then use the "--force" flag with rhnreg_ks, otherwise there is no need to use "--force".
# Fedora 11



 1. Install the Spacewalk client yum repository
   
    # rpm -Uvh http://spacewalk.redhat.com/yum/1.0/Fedora/11/$(uname -i)/spacewalk-client-repo-1.0-2.fc11.noarch.rpm
       }}}
     1. Install client packages
       {{{
    # yum install rhn-client-tools rhn-check rhn-setup rhnsd m2crypto yum-rhn-plugin
       }}}
     1. Register your Fedora system to Spacewalk using the activation key you created earlier
       {{{
    # rhnreg_ks --serverUrl=http://YourSpacewalk.example.org/XMLRPC --activationkey=<key-with-fedora-custom-channel> 
       }}}
# Fedora 12

    

      1. Install the Spacewalk yum repository
       {{{
    # rpm -Uvh http://spacewalk.redhat.com/yum/1.0/Fedora/12/$(uname -i)/spacewalk-client-repo-1.0-2.fc12.noarch.rpm
       }}}
      1. Install client packages
        {{{
    # yum install rhn-client-tools rhn-check rhn-setup rhnsd m2crypto yum-rhn-plugin
        }}}
      1. Register your Fedora system to Spacewalk using the activation key you created earlier
       {{{
    # rhnreg_ks --serverUrl=http://YourSpacewalk.example.org/XMLRPC --activationkey=<key-with-fedora-custom-channel> 
       }}}
# CentOS 5 or Red Hat Enterprise Linux 5

    

    '''Warning:''' If you are installing these packages on a Red Hat Enterprise Linux installation it will override some of the original base packages and you may well be invalidating your support agreement with Red Hat!
    
      1. Install the Spacewalk yum repository
       {{{
    # BASEARCH=$(uname -i)
    # rpm -Uvh http://spacewalk.redhat.com/yum/1.0/RHEL/5/$BASEARCH/spacewalk-client-repo-1.0-2.el5.noarch.rpm
       }}}
      1. The latest client tools bring the upstream development to your client boxes. That means that the packages may have dependencies that are not found in core Red Hat Enterprise Linux 5. These dependencies can be found in EPEL, just like for the Spacewalk server:
       {{{
    # BASEARCH=$(uname -i)
    # rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/$BASEARCH/epel-release-5-3.noarch.rpm
       }}}
      1. Install client packages
        {{{
    # yum install rhn-client-tools rhn-check rhn-setup rhnsd m2crypto yum-rhn-plugin
        }}}
      1. Register your CentOS or Red Hat Enterprise Linux system to Spacewalk using the activation key you created earlier
       {{{
    # rhnreg_ks --serverUrl=http://YourSpacewalk.example.org/XMLRPC --activationkey=<key-with-rhel-custom-channel> 
       }}}
# CentOS 4

    

    Registering a CentOS 4 server to Spacewalk is exactly the same as it would be for CentOS 5, but rhnreg_ks, rhn_check and other related scripts are located in the package '''up2date''', and not in '''rhn-setup'''.
    
      1. Enable '''spacewalk-tools''' repo for Yum and install '''up2date''' package:
    {{{
    # rpm -ivh http://stahnma.fedorapeople.org/spacewalk-tools/spacewalk-client-tools-0.0-1.noarch.rpm
    # yum install up2date 
  1. Register your CentOS system to Spacewalk using the activation key you created earlier:

    # rhnreg_ks --serverUrl=http://YourSpacewalk.example.org/XMLRPC --activationkey=<key-with-centos-custom-channel>
# Solaris 8, 9 or 10



See [Solaris Support](Solaris).