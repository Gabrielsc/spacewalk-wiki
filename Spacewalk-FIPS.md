# Spacewalk and FIPS-140-2

## Introduction




_FIPS 140-2 (FIPS = Federal Information Processing Standard)_ is a U.S. government security standard used to accredit cryptographic modules. It outlines a set of requirements that must be adhered to by any US-government certified cryptographic module (hardware or software).

* FIPS 140-2 Annexes: *

* Annex A: Approved Security Functions
  * http://csrc.nist.gov/publications/fips/fips140-2/fips1402annexa.pdf
* Annex B: Approved Protection Profiles
  * http://csrc.nist.gov/publications/fips/fips140-2/fips1402annexb.pdf
* Annex C: Approved Random Number Generators
  * http://csrc.nist.gov/publications/fips/fips140-2/fips1402annexc.pdf
* Annex D: Approved Key Establishment Techniques
  * http://csrc.nist.gov/publications/fips/fips140-2/fips1402annexd.pdf

Operating systems supported by Spacewalk (RHEL, CentOS, Fedora) allow to turn FIPS mode on. In the FIPS mode, operating system is doing integrity checking and enforces minimum configurations for FIPS-aware cryptography
software (OpenSSL, libgcrypt).
## Scope of support



Spacewalk 2.2 introduced support for FIPS 140-2 standard. Spacewalk support for the FIPS standard allows:

* Installing and running your Spacewalk on an operating system (RHEL, CentOS, Fedora) with FIPS mode on.
* Running your registered clients in FIPS mode.
## Implementation notes



The FIPS 140-2 standard does not allow to use MD5 algorithm for cryptographic / security purposes, nor would a FIPS enabled operating system allow to use MD5 algorithm for cryptography.

This restriction lead to two important implementation changes in Spacewalk:
* User passwords, previously encrypted with MD5 method, will be (starting with Spacewalk 2.2) encrypted with SHA-256 algorithm.
* Client certificates (_/etc/sysconfig/rhn/systemid_), which the registered systems use to authenticate with the parent server, contain an MD5 digest string. In Spacewalk 2.2, the MD5-based authentication method had to be changed to SHA-256.
## Operations



Beginning with Spacewalk 2.2, all newly registered clients will be issued a certificate with SHA-256 authentication digest. Also, starting with Spacewalk 2.2, passwords for all newly created user accounts will be encrypted
with SHA-256 method. Though whenever a user with MD5-encrypted password would change his password or even just successfully authenticate with Spacewalk (a successful webui login for example), his MD5-encrypted password
would be automatically converted to a SHA-256 encrypted password.

How will this change affect Spacewalk functioning?
### Not interested in FIPS



If you're not planning to turn the FIPS mode on your Spacewalk server on, you won't have to do any changes in your Spacewalk operations:
* For Spacewalk installations, you will follow standard installation instructions.
* For Spacewalk upgrades, you will follow standard upgrade instructions.
* For client registrations, you will follow standard registration instructions.

You will hardly notice any changes in your Spacewalk's behavior (MD5s changes to SHA-256).
### I want to turn the FIPS mode on

#### New Spacewalk installations




It is possible to install Spacewalk 2.2 (or later) on an operating system already in FIPS mode. Standard Spacewalk installation instructions would apply here.
From the very beginning, all user passwords will be encrypted with SHA-256 method, all clients will be issued a SHA-256 certificate.
#### Spacewalk upgrades



If you're upgrading from a previous Spacewalk version, you will need to update user passwords and client certificates from the MD5 scheme to SHA-256.

Your procedure for turning the FIPS mode on would be:
1. Upgrade your Spacewalk, following the standard upgrade procedure. Don't turn the FIPS mode on just yet. If you'd turn the FIPS mode on now, your users would no longer be able to log into your Spacewalk (Spacewalk would not be able to authenticate them), nor would your registered clients be able to check in with the parent server (_rhn_check_, _rhnsd_, _osad_).
2. Update user passwords and client certificates from the MD5 scheme to SHA-256 (see the detailed procedure below).
3. After the password and certificate update successfully finishes, you can turn the FIPS mode on.
## Update of user passwords and client certificates

### User passwords




For a proper Spacewalk functioning in FIPS mode, your user passwords need to be encrypted with SHA-256 algorithm. Users with MD5-encrypted password would no longer be able to authenticate in FIPS mode.

The list of users with MD5-encrypted password can be obtained with _spacewalk-report_ (as _root_ on your Spacewalk server):


    $ spacewalk-report users-md5 > users-md5.csv

Once you have the list of users requiring the password encryption update, you can either:
1. Reset their password using _satpasswd_ utility (as _root_ on your Spacewalk server):

    $ for i in $(cat users-md5.csv | awk -F, 'NR>1 { print $4 }'); do
          echo "Changing password for user $i";
          satpasswd $i;
          echo;
    done

2. Instruct all of the users on the list to log into your Spacewalk's webui (no other action is needed). Any time a user with MD5-encrypted password successfully authenticates, Spacewalk will automatically convert his password from MD5 to SHA-256.

3. Take advantage of external authentication mechanisms supported by Spacewalk. Please consult https://fedorahosted.org/spacewalk/wiki/SpacewalkAndIPA for further information.
### Client certificates



Systems registered to Spacewalk use a client certificate (_/etc/sysconfig/rhn/systemid_) to authenticate with their parent server. This certificate is being issued to the clients during system registration and contains an authentication digest. Starting with Spacewalk 2.2, the algorithm used for creating the authentication digest was changed from MD5 to SHA-256.

The list of client systems using certificate with MD5 digest can be obtained with _spacewalk-report_ (as _root_ on your Spacewalk server):

    $ spacewalk-report system-md5-certificates > system-md5-certificates.csv

Once you have the list of systems requiring certificate update, you can:
1. Re-register the systems (_rhn_register_, _rhnreg_ks_).
2. Schedule update of the client certificate using _spacewalk-client-cert_ package.
#### Update of client certificate with _spacewalk-client-cert_ package



Updating client certificates with the help of _spacewalk-client-cert_ package is a two stage process:

1. In the first step, you need to install _spacewalk-client-cert_ package (part of the Spacewalk client yum repository) on the clients requiring certificate update. This can be done either:
  * manually (_rpm -ivh spacewalk-client-cert.rpm && rhn-profile-sync_, _yum install spacewalk-client-cert_)
  * via Spacewalk XML-RPC API. Spacewalk 2.2 comes with a script called _spacewalk-fips-tool_, which will help you schedule installations of the _spacewalk-client-cert_ package:
    
    $ spacewalk-fips-tool --help
    Usage: spacewalk-fips-tool -i|-c [options] systemid01 [systemid02 ...]
    
    Options:
      -h, --help   show this help message and exit
      -i           schedule installation of packages required for certificate
                   update
      -c           schedule update of client certificates for specified systems
      -u USERNAME  username (organizational administrator)
      -p PASSWORD  password
      -s HOSTNAME  server hostname to use, defaults to localhost
      -d DATE      date to schedule the conversion for, defaults to now, format:
                   %Y-%m-%d %H:%M:%S (e.g. 2014-10-30 19:30:00)
      -o OUTPUT    output file in csv format to store the details about all
                   scheduled actions, use '-' for stdout
    
    $ ORG_ID=1
    $ for system in $(awk -F, "NR>1 { if (\$3 == $ORG_ID) print \$1 }" system-md5-certificates.csv); do systems="$systems $system"; done
    $ spacewalk-fips-tool -i \
                          -u admin \
                          -d 2014-10-25\ 14:00:00 \
                          -o /tmp/scheduled-installations.csv \
                          $systems
        }}}
        Important notes about the above procedure:
          * The package installations need to be scheduled on a per-organization basis, i.e. you will need org-admin credentials for each of your organizations.
          * Successfully scheduled actions are recorded in the output csv file for later tracking (-o option).
          * You can schedule the package installations for a specific date (-d option).
    
    2. In the second step, after the ''spacewalk-client-cert'' packages are successfully installed, you can schedule updates of client certificates:
    {{{
    $ spacewalk-client-cert -c \
                            -u admin
                            -d 2014-10-25\ 14:00:00 \
                            -o /tmp/scheduled-certificate-updates.csv \
                            $systems
After the certificate updates all finish, you can switch the operating system under your Spacewalk installation to FIPS mode.