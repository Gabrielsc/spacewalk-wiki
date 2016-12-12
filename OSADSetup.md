# osad Installation Instructions



 * [Server Setup (osa-dispatcher and jabberd)](https://fedorahosted.org/spacewalk/wiki/OSADSetup#ServerSetup)
 * [Client Setup (osad)](https://fedorahosted.org/spacewalk/wiki/OSADSetup#ClientSetup)
# Server Setup



 * Install osa-dispatcher:

    yum install osa-dispatcher
    chkconfig osa-dispatcher on
    service osa-dispatcher start
# Client Setup



 * Install osad

    yum install osad
 * Change _osa_ssl_cert_ in /etc/sysconfig/rhn/osad.conf to:

    osa_ssl_cert = /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT
 * If you haven't already, download the trusted cert to your client

    cd /usr/share/rhn/
    wget http://yourspacewalk.example.org/pub/RHN-ORG-TRUSTED-SSL-CERT
 * Start osad

    service osad start
