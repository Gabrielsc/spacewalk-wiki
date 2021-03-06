# Spacewalk Installation Instructions



These are installation instruction for new installations of Spacewalk 1.5. If you are upgrading from older versions, see HowToUpgrade.

Spacewalk 1.4 installation instructions are available at [[HowToInstall14]].

----
## Prerequisites



 * Outbound open ports 80, 443, 4545 (only if you want to enable monitoring)
 * Inbound open ports 80, 443, 5222 (only if you want to push actions to client machines) and 5269 (only for push actions to a Spacewalk Proxy), 69 udp if you want to use tftp
 * Storage for database: 250 KiB per client system +  500 KiB per channel + 230 KiB per package in channel (i.e. 1.1GiB for channel with 5000 packages)
 * Storage for packages (default /var/satellite): Depends on what you're storing; Red Hat recommend 6GB per channel for their channels
 * 2GB RAM minimum, 4GB recommended
 * Make sure your underlying OS up-to-date.
 * If you use LDAP as a central identity service and wish to pull user and group information from it, see [[SpacewalkWithLDAP]]
 * Make sure your operating system is fully up-to-date.
## Setting up Spacewalk repo



RPM downloads of the project are available through a yum repository

  * http://spacewalk.redhat.com/yum/ - Binary RPMs
  * http://spacewalk.redhat.com/source/ - Source RPMs and tarballs

To use this repository install spacewalk-repo with commands below:
### Red Hat Enterprise Linux 5, CentOS




    rpm -Uvh http://spacewalk.redhat.com/yum/1.5/RHEL/5/x86_64/spacewalk-repo-1.5-1.el5.noarch.rpm
### Red Hat Enterprise Linux 6, CentOS




    rpm -Uvh http://spacewalk.redhat.com/yum/1.5/RHEL/6/x86_64/spacewalk-repo-1.5-1.el6.noarch.rpm
### Fedora 14




    rpm -Uvh http://spacewalk.redhat.com/yum/1.5/Fedora/14/x86_64/spacewalk-repo-1.5-1.fc14.noarch.rpm
### Fedora 15




    rpm -Uvh http://spacewalk.redhat.com/yum/1.5/Fedora/15/x86_64/spacewalk-repo-1.5-1.fc15.noarch.rpm
### Nightly builds



If you want to use the nightly builds, add a .repo file pointing to the nightly repository.

Server:


    cat > /etc/yum.repos.d/spacewalk.repo << 'EOF'
    [spacewalk]
    name=Spacewalk
    # RHEL 5 / CentOS 5
    baseurl=http://spacewalk.redhat.com/yum/nightly/RHEL/5/$basearch/
    # RHEL 6
    #baseurl=http://spacewalk.redhat.com/yum/nightly/RHEL/6/$basearch/
    # Fedora 14
    #baseurl=http://spacewalk.redhat.com/yum/nightly/Fedora/14/$basearch/
    # Fedora 15
    #baseurl=http://spacewalk.redhat.com/yum/nightly/Fedora/15/$basearch/
    gpgkey=http://spacewalk.redhat.com/yum/RPM-GPG-KEY-spacewalk-2010
    enabled=1
    gpgcheck=0
    EOF

Client:


    cat > /etc/yum.repos.d/spacewalk-client.repo << 'EOF'
    [spacewalk-client]
    name=Spacewalk
    # RHEL 5 / CentOS 5
    baseurl=http://spacewalk.redhat.com/yum/nightly-client/RHEL/5/$basearch/
    # RHEL 6 / CentOS 6
    #baseurl=http://spacewalk.redhat.com/yum/nightly-client/RHEL/6/$basearch/
    # Fedora 14
    #baseurl=http://spacewalk.redhat.com/yum/nightly-client/Fedora/14/$basearch/
    # Fedora 15
    #baseurl=http://spacewalk.redhat.com/yum/nightly-client/Fedora/15/$basearch/
    gpgkey=http://spacewalk.redhat.com/yum/RPM-GPG-KEY-spacewalk-2010
    enabled=1
    gpgcheck=0
    EOF

*NOTE:* Nigthly repo contains developers snapshot and is not suitable in production environment.
## Additional repos & packages

### Jpackage repository


#### RHEL, CentOS 6 and Fedora 14 & 15




For Spacewalk on RHEL 6 and Fedora, a couple of additional dependencies are needed from jpackage.
If repo is broken, mirror list here: http://www.jpackage.org/mirroring.php
Please configure the following yum repository before beginning your Spacewalk installation:


    cat > /etc/yum.repos.d/jpackage-generic.repo << EOF
    [jpackage-generic]
    name=JPackage generic
    #baseurl=http://mirrors.dotsrc.org/pub/jpackage/5.0/generic/free/
    mirrorlist=http://www.jpackage.org/mirrorlist.php?dist=generic&type=free&release=5.0
    enabled=1
    gpgcheck=1
    gpgkey=http://www.jpackage.org/jpackage.asc
    EOF

We specifically want 5.0 generic directory in the above URL.
Make sure, you do not use the jpackage-generic repository on RHEL 5.
### CentOS - CR repository



selinux update need CR repository:


    cat > /etc/yum.repos.d/cr.repo << EOF
    [CR-repository]
    name=CR Repository
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=cr
    baseurl=http://mirror.centos.org/centos/$releasever/cr/$basearch/
    enabled=1
    gpgcheck=1
    EOF
### EPEL repository



Spacewalk requires a Java Virtual Machine with version 1.6.0 or greater.  [EPEL - Extra Packages for Enterprise Linux](http://fedoraproject.org/wiki/EPEL) contains a version of the openjdk that works with Spacewalk.  Other dependencies can get installed from EPEL as well.
To get packages from EPEL just install this RPM:
#### EPEL 5 (use for RHEL 5)



    rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm
#### EPEL 6 (use for RHEL 6)



    rpm -Uvh http://download.fedora.redhat.com/pub/epel/6/i386/epel-release-6-5.noarch.rpm

Please make sure you have only one java version installed. 
Otherwise you get error messeges like :
 * " bad major version at off set=6" or 
 * "INFO: validateJarFile(/usr/share/tomcat5/webapps/rhn/WEB-INF/lib/jspapi.jar) - jar not loaded."

When using RHEL 6, make sure you are subscribed to following channels as well:
 * *Red Hat High Availability Server 6*
 * *Red Hat Optional Server 6*
### CentOS 5



Follow the instructions to use *EPEL 5* with the additions:
 * Necessary packages rhn-client-tools and rhnlib were removed from CentOS, they can be found in spacewalk-client repo. Setup it by installing spacewalk-client-repo package.

    rpm -ihv http://spacewalk.redhat.com/yum/1.5-client/RHEL/5/x86_64/spacewalk-client-repo-1.5-1.el5.noarch.rpm
 *  Import Redhat's RPM GPG key:

    # wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release http://www.redhat.com/security/37017186.txt
    # rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
## Oracle Pre-Requisites



In order to get Spacewalk to run with Oracle database backend, you need:
 * the Oracle Instant Client
 * an oracle database, this can either be:
  * Oracle 10g or 11g database server see the [Full Oracle Setup](FullOracleSetup) page. (we assume here you use a seperate host running oracle)
  * the free alternative Oracle XE. Instructions on how to do this can be found on the [Oracle 10g Express Edition Setup](OracleXeSetup) page.
## PostgreSQL Pre-Requisites



In order to get Spacewalk ro run with PostgreSQL database backend, you need PostgreSQL 8.4 server installed on the same or different machine. Use [[PostgreSQLServerSetup]] as a guide to get the server installed and setup.

Please note that the functionality of Spacewalk on PostgreSQL is limited and sooner than later you may hit Internal Server Error.
## Conflicting Packages

There is an open [bug 593402](https://bugzilla.redhat.com/show_bug.cgi?id=593402) that identifies the package cobbler-web as conflicting with Spacewalk's Taskomatic process. Until the issue is addressed, it is recommended that the cobbler-web package be removed from the Spacewalk server.


    yum remove cobbler-web

Yum can also be configured to exclude that package in the future by adding the following entry to the [[main]] section in /etc/yum.conf:

    vi /etc/yum.conf
    
    [main]
    ....
    exclude=cobbler-web

There is a workaround to be able to install cobbler-web and use it alongside Spacewalk.  Instructions can be found at CobblerWebWorkaround.
## Installing Spacewalk



Just ask yum to install the necessary packages. This will pull down and install the set of RPMs required to get Spacewalk to run.

If you tend to use the Oracle backend:

    yum install spacewalk-oracle 

If you prefer the PostgreSQL backend:

    yum install spacewalk-postgresql 

You may want to take a look at the [[PostgreSQL]] page for further information about the PostgreSQL backend.
## Configuring the firewall



Spacewalk needs various inbound ports to be connectible. Assuming you have a default RHEL/CentOS set up, proceed as following:

For RHEL 5/CentOS 5/Scientific Linux 5

    iptables -D RH-Firewall-1-INPUT -j REJECT --reject-with icmp-host-prohibited
    iptables -A RH-Firewall-1-INPUT -p tcp -m tcp --dport 443 -j ACCEPT
    iptables -A RH-Firewall-1-INPUT -p tcp -m tcp --dport 80 -j ACCEPT
    iptables -A RH-Firewall-1-INPUT -j REJECT --reject-with icmp-host-prohibited
    service iptables save
    service iptables restart

For RHEL 6/Scientific Linux 6

    iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited
    iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
    iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
    iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
    service iptables save
    service iptables restart


Add port 5222 if you want to push actions to client machines and 5269 for push actions to a Spacewalk Proxy, 69 udp if you want to use tftp.
## Configuring Spacewalk



Your Spacewalk server should have a resolvable FQDN such as 'hostname.domain.com'.  If the installer complains that the hostname is not the FQDN, do not use the --skip-fqdn-test flag to skip !  

The setup requires that the database account has a password.

*Note:* Please don't use * '#' * (number sign/pound/hash) and * '@' * in your database password otherwise installation will fail.

Once the Spacewalk RPM is installed you need to configure the application, you can run:


    spacewalk-setup --disconnected



An example session is as follows:


    # spacewalk-setup --disconnected
    * Setting up Oracle environment.
    * Setting up database.
    ** Database: Setting up database connection for Oracle backend.
    Database service name (SID)? XE
    Username? spacewalk
    Password? 
    ** Database: Testing database connection.
    ** Database: Populating database.
    *** Progress: ####
    * Setting up users and groups.
    ** GPG: Initializing GPG and importing key.
    ** GPG: Creating /root/.gnupg directory
    You must enter an email address.
    Admin Email Address? root@localhost
    * Performing initial configuration.
    * Activating Spacewalk.
    ** Loading Spacewalk Certificate.
    ** Verifying certificate locally.
    ** Activating Spacewalk.
    * Enabling Monitoring.
    * Configuring apache SSL virtual host.
    Should setup configure apache's default ssl server for you (saves original ssl.conf) [Y]? 
    ** /etc/httpd/conf.d/ssl.conf has been backed up to ssl.conf-swsave
    * Configuring tomcat.
    ** /etc/tomcat5/tomcat5.conf has been backed up to tomcat5.conf-swsave
    ** /etc/tomcat5/server.xml has been backed up to server.xml-swsave
    ** /etc/tomcat5/web.xml has been backed up to web.xml-swsave
    * Configuring jabberd.
    * Creating SSL certificates.
    CA certificate password? 
    Re-enter CA certificate password? 
    Organization? Fedora
    Organization Unit [spacewalk.server.com]? Spacewalk Unit
    Email Address [root@localhost]? 
    City? Brno
    State? CZ
    Country code (Examples: "US", "JP", "IN", or type "?" to see a list)? CZ
    ** SSL: Generating CA certificate.
    ** SSL: Deploying CA certificate.
    ** SSL: Generating server certificate.
    ** SSL: Storing SSL certificates.
    * Deploying configuration files.
    * Update configuration in database.
    * Setting up Cobbler..
    Cobbler requires tftp and xinetd services be turned on for PXE provisioning functionality. Enable these services [Y/n]?
    cobblerd does not appear to be running/accessible
    * Restarting services.
    Installation complete.
    Visit https://spacewalk.server.com to create the Spacewalk administrator account.
### Configuring Spacewalk with an Answer File



You can also configure Spacewalk by using an answer file, by running `spacewalk-setup` like


    spacewalk-setup --disconnected --answer-file=<FILENAME>

An example answer file for the Oracle database backend:


    admin-email = root@localhost
    ssl-set-org = Spacewalk Org
    ssl-set-org-unit = spacewalk
    ssl-set-city = My City
    ssl-set-state = My State
    ssl-set-country = US
    ssl-password = spacewalk
    ssl-set-email = root@localhost
    ssl-config-sslvhost = Y
    db-backend=oracle
    db-user=spacewalk
    db-password=spacewalk
    db-name=xe
    db-host=localhost
    db-port=1521
    enable-tftp=Y

If you do not supply a value or leave out a key you will be prompted to supply that answer.

For PostgreSQL, you need to create something like this:


    admin-email = root@localhost
    ssl-set-org = Spacewalk Org
    ssl-set-org-unit = spacewalk
    ssl-set-city = My City
    ssl-set-state = My State
    ssl-set-country = US
    ssl-password = spacewalk
    ssl-set-email = root@localhost
    ssl-config-sslvhost = Y
    db-backend=postgresql
    db-name=spaceschema
    db-user=spaceuser
    db-password=spacepw
    db-host=host
    db-port=5432
    enable-tftp=Y

After `spacewalk-setup` is complete your application is ready to go!

Once you've made sure you can login to the Spacewalk WebUI, you can then proceed to the next step: [Uploading Content](UploadFedoraContent).
## Managing Spacewalk



Spacewalk consists of several services. Each of them has its own init.d script to stop/start/restart. If you want manage all spacewalk services at once use 

    /usr/sbin/spacewalk-service [stop|start|restart].
## Known Issues



 * For issues with Oracle XE, see [Oracle XE Setup Troubleshooting](OracleXeSetup)

