

This page is now obsolete. See [Registering Clients](RegisteringClients) for up-to-date instructions.

----
# __Managing Fedora Systems Via Spacewalk__

If you've got a few Fedora workstations you're likely to want to have some sort of centralised management for them. Spacewalk is one tool which can provide this.  We assume you have a spacewalk server and the need to deploy multiple Fedora clients. This guide is based around Fedora 9, please feel free to add any additional notes for other versions.

## Channel Setup

We'll create a channel holding all the Fedora rpms you may want to deploy to clients.  It is encouraged that you use a sub-channel for each repository you pull from.  For example we have a "Fedora 9 x86_64" channel.  This channel contains all the Fedora 9 packages from Fedora.  A "Livna" sub-channel contains packages from the livna repository.  Essentially you want the following structure


* Fedora 9 x86_64 chanel
  * Livna sub-channel
  * My custom packages sub-channel

The [[UploadFedoraContent]] page describes how to create a channel, reposync your required repositories and then upload the content to spacewalk.  At the end of this process you'll end up with a copy of the entire Fedora repository on your disk and another copy living inside of spacewalk (actually in /var/satellite).  This consumes quite a bit of space.  Furthermore rhnpush'ing an entire Fedora repository to a spacewalk server can take several days.  Some have managed it in a single day, others find it takes 2-3 days on more modest hardware.

You'll periodically want to resync the Fedora repository and upload the updated packages to spacewalk.  This is a repeat of the 2-3 day rhnpush process.  You then have to push the updated packages to your clients.  This pushing practice is described below.
## Provisioning Clients

Kickstart is possibly the easiest way to provision your clients.  It allows you to define the packages to install, authentication configuration etc... in a central spot.  Then each of your clients will be deployed with the same configuration.  It's a lot more manageable than creating a system imaging and rolling that out over multiple clients.  Spacewalk will allow you to define a kickstart profile.  You'll then have to follow [[KickstartDistro]] to upload Fedora content.  A kickstart will only install content from the kickstart tree.  It will not install content that is available in the "Fedora 9 x86_64" channel.  We find that kickstart is useful for provisioning your systems with the base and X Windows packages and with our authentication mechanism.  We deploy other packages using Spacewalk.  This allows us to update the packages on the system more easily.


Then using a Fedora boot CD in the client (advanced users can check out PXE booting) you can specify "ks= http://my.spacewalk.server/kickstart/ks/org/1x6454525425678f7ae02542542541cf/label/fedora-9-x86_64" as a kernel boot parameter.  The client will automatically download the kickstart file and configure itself accordingly.  This is a bare-metal kickstart, so it's important you use the kickstart URL as given to you by "Systems->Kickstart Details->Bare Metal Kickstart".  Furthermore, that big ugly number that you have to type is your organisation id.  We created a USB boot disk (using Fedora's Revisor application) with the bare metal kickstart URL in the boot parameters.  This means that we simply plug the USB disk into the machine, turn it on and go get some coffee.
## Registering Clients

For this we want an activation key.  Create a spacewalk activation key and associate it with your "Fedora 9 x86_64" channel.  It's important that the key is associated with a specific channel and not just the default.  You then have to install packages on the client and register with your server.  The process is described in [[RegisteringClients]].  However, if you've been kickstarting your clients you may want this to happen as part of the kickstart process.


As a precondition to this process you must ensure that your Fedora kickstart installs the m2crpto package.  This must be within your kickstart tree.  With Fedora 9 and Spacewalk 0.3 automatically registering clients requires creating a Post Script for your kickstart containing the following; do note that you must replace your spacewalk server URL and activation key below.


    # Set a proxy (if required)
    export http_proxy=http://proxy.brighton.ac.uk:80
    
    cd /tmp
    
    # Install keys
    # Fedora newkey
    # Livna
    # Rpm fusion
    # You should host these keys on your own web server.
    wget http://rpm.livna.org/RPM-LIVNA-GPG-KEY
    wget http://foss.it.brighton.ac.uk/~balor/RPM-GPG-KEY-fedora-x86_64
    wget http://foss.it.brighton.ac.uk/~balor/RPM-GPG-KEY-rpmfusion-free-fedora
    wget http://foss.it.brighton.ac.uk/~balor/RPM-GPG-KEY-rpmfusion-nonfree-fedora
    wget http://foss.it.brighton.ac.uk/~balor/RPM-GPG-KEY-fedora-8-and-9
    wget http://foss.it.brighton.ac.uk/~balor/RPM-GPG-KEY-fedora-8-and-9-x86_64
    
    rpm --import RPM-LIVNA-GPG-KEY
    rpm --import RPM-GPG-KEY-fedora-x86_64
    rpm --import RPM-GPG-KEY-rpmfusion-free-fedora
    rpm --import RPM-GPG-KEY-rpmfusion-nonfree-fedora
    rpm --import RPM-GPG-KEY-fedora-8-and-9
    rpm --import RPM-GPG-KEY-fedora-8-and-9-x86_64
    
    # Install packages necessary for registration
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhn-check-0.4.17-10.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhn-client-tools-0.4.17-10.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhn-setup-0.4.17-10.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhn-setup-gnome-0.4.17-10.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhncfg-0.2.2-1.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhncfg-actions-0.2.2-1.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhncfg-client-0.2.2-1.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhncfg-management-0.2.2-1.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhnlib-2.2.5-3.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhnpush-0.2.4-1.fc9.noarch.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/rhnsd-4.6.1-1.el5.i386.rpm
    wget http://spacewalk.redhat.com/yum/0.3/fedora/9/i386/yum-rhn-plugin-0.5.3-6.el5_2.6.noarch.rpm
    
    # This is _very_ necessary
    wget http://mirrors.vexs.net/pub/fedora/linux/releases/9/Everything/x86_64/os/Packages/m2crypto-0.18.2-4.x86_64.rpm
    
    wget http://dl.fedoraproject.org/pub/fedora/linux/releases/9/Everything/x86_64/os/Packages/libxml2-python-2.6.32-1.fc9.x86_64.rpm
    wget http://dl.fedoraproject.org/pub/fedora/linux/releases/9/Everything/x86_64/os/Packages/pyOpenSSL-0.6-4.fc9.x86_64.rpm
    
    # Don't check the signature of RPMs as the spacewalk key may change to the Fedora one soon(ish)
    # Don't check deps as there's a missing Python dep in rhnlib (it's not missing, the rpm file just thinks it is)
    rpm --install -v --hash --nosignature --nodeps *.rpm
    
    # install our ORG Spacewalk key from a http source.  Not a good idea when concerned with security, but it works.
    wget http://some.internal.sercer/RHN-ORG-TRUSTED-SSL-CERT
    mkdir /usr/share/rhn/
    cp RHN-ORG-TRUSTED-SSL-CERT /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT
    perl -npe 's/RHNS-CA-CERT/RHN-ORG-TRUSTED-SSL-CERT/g' -i /etc/sysconfig/rhn/*
    
    # Make up2date look at our server (my servers name is computersciencegames.net)
    perl -npe 's/xmlrpc.rhn.redhat.com/computersciencegames.net/' -i /etc/sysconfig/rhn/up2date
    mkdir -p /etc/sysconfig/rhn/allowed-actions/configfiles
    touch /etc/sysconfig/rhn/allowed-actions/configfiles/all
    
    # Clean up
    rm m2*.rpm
    rm rhn*.rpm
    rm yum*.rpm
    
    # Do the actual registration (put your actual activation key in place of the dummy one)
    rhnreg_ks --serverUrl=http://computersciencegames.net/XMLRPC --activationkey=1-dcdcdcdcdcdcdcdcdcdcdcdcdcd
## Pushing Packages to Clients


## Managing Client Configurations


## Related Documentation



* [Deployment, configuration and administration of Fedora 12](http://docs.fedoraproject.org/deployment-guide/f12/en-US/html/)