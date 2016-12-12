# Spacewalk Proxy Installation Instructions



*These instructions are for Spacewalk Proxy 0.2 onwards*

If you're looking for instructions for the RHN Proxy product see the [Red Hat Network Satellite documentation](http://docs.redhat.com/docs/en-US/Red_Hat_Network_Satellite/index.html).
## Prerequisites



  * Around 6GB storage per distribution under /var/spool/squid (or whereever you want your Squid cache to be)
  * Outbound open ports 80, 443, 4545 (only if you want to enable monitoring) and 5269
  * Inbound open ports 80, 443 and 5222
  * An upstream RHN Satellite server with an available Proxy entitlement or a Spacewalk server
  * Machine where you will install Spacewalk Proxy must be registered against Spacewalk Server, which you will proxy.
  * A provisioning entitlement for the Proxy server
  * Enable EPEL yum repository
## Repository



RPM downloads of the project are available through yum repositories. Set up your yum to point to Spacewalk 1.7 repository. For the repo setup specifics, see HowToInstall#SettingupSpacewalkrepo. 
## Installation



Ensure your machine is registered in Spacewalk and has a provisioning entitlement. Then just ask yum to install the application:


    yum install spacewalk-proxy-selinux spacewalk-proxy-installer

This will pull down and install the set of RPMs required for the installer to run. Below is an example of the set of packages pulled in:


    ========================================================================================
     Package                       Arch       Version            Repository            Size
    ========================================================================================
    Installing:
     spacewalk-proxy-installer     noarch     1.7.6-1.el5        spacewalk             50 k
    Installing for dependencies:
     rhncfg                        noarch     5.10.27-1.el5      spacewalk-client      68 k
     rhncfg-actions                noarch     5.10.27-1.el5      spacewalk-client      41 k
     rhncfg-client                 noarch     5.10.27-1.el5      spacewalk-client      37 k
     rhncfg-management             noarch     5.10.27-1.el5      spacewalk-client      48 k

If this is the first time installing an RPM from the Spacewalk repo, yum will prompt you to install the GPG key:


    Importing GPG key 0x863A853D "Spacewalk <spacewalk-devel@redhat.com>" from http://spacewalk.redhat.com/yum/RPM-GPG-KEY-spacewalk-2012
    Is this ok [y/N]: y

Then you need to configure the proxy. Run:


    configure-proxy.sh

and follow the prompts. This will download and configure the nessecery packages. Once installed use /usr/sbin/rhn-proxy to stop and start the service.

You should now be able to navigate to the hostname of your proxy in a web browser and see the Spacewalk Proxy page. If you see an error here there's likely something wrong with your install.

  * If all seems well you can now try [Registering Clients](RegisteringClients)
  * If you're having problems check [Debugging Proxy](DebuggingProxy)
## Automated Installation



The configure-proxy.sh install script supports an answer file to allow you to preanswer the questions. For the full list of variables see "man configure-proxy.sh".


    configure-proxy.sh --answer-file=proxyanswers.txt

proxyanswers.txt:


    VERSION="1.6"
    RHN_PARENT="spacewalk.example.com"
    TRACEBACK_EMAIL="admin@example.com"
    USE_SSL="Y"
    CA_CHAIN="/usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT"
    HTTP_PROXY=
    SSL_ORG="Example Org"
    SSL_ORGUNIT="proxy1.example.com"
    SSL_COMMON="proxy1.example.com"
    SSL_CITY="New York"
    SSL_STATE="New York"
    SSL_COUNTRY="US"
    SSL_EMAIL="admin@example.com"
    INSTALL_MONITORING="n"
    POPULATE_CONFIG_CHANNEL="n"

As this example doesn't answer all the questions (e.g. SSL_PASSWORD) you'd still be prompted for that value. If you know that you've answered all questions then you can run with --non-interactive.