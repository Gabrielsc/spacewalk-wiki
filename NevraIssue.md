
    #!div class="important" style="border: 2pt solid; text-align: center" 
    '''''DEPRECATED, NO LONGER USED''''' 
# NEVRA Issue

## Tasks




Md5Sum Changes

|  Task  |  Estimate hours  |  Assignee  |  Status  |  Notes  |
| --- | --- | --- | --- | --- |
|  DB Schema for md5sum  |  2  |  jsherrill  |  done  |   |  |
|  rhnPush   |  2  |  prad  |  done  |  (correct file system location)  |
|  sat-sync   |  4  |  prad  |  done  |  (correct file system location)  |

Gpg/Provider 

|  Task  |  Estimate hours  |  Assignee  |  Status  |  Notes  |
| --- | --- | --- | --- | --- |
|  DB Schema for gpg/provider linkage  |  2  |  jsherrill  |  done  |   |  |
|  rhnpush(server)  |  4  |  prad  |  done  |  read gpg id |
|  satellite-sync  |  16  |  prad  | done  |  read gpg id, import gpg info if available  |
|  webUi  |  24  | jsherrill  | in progress  |  includes re-writing some perl pages  |
|  migration-script |  8  | prad  | done  |  script to migrate existing pkg paths  |
## The Problems



At current, Spacewalk does not allow packages with the same NEVRA to be pushed
to the same org. For example, rootfiles-8.1-1.1.1.noarch is shipped with RHEL 5,
CentOS 5, and Fedora 9. All 3 packages have different GPG signatures and thus
different md5sums.

Also, CentOS rebuilds and resigns their packages across arches.  So even within
CentOS there may be two foo-1.0.i386 packages with the same GPG key but with
different md5sums.

In summary, we have the following:

1. Distinguish between two packages such that distribution to a client is
possible.

2. Easily manage packages from different sources and be able to differentiate
between where packages originated.  (Which i think is VERY important). 

Originally we were trying to lump the issues together under one problem with one
solution.
## Requirements



 1.  Customer should be able to push RHEL 5, Fedora 9, and CentOs 5 to separate channels and have their packages all stored on the satellite within the same org.
 2.  It should be fairly simple to know where a package originated from
 3.  Customer should be able to kickstart all 3 distro's from within the same org.
 4.  Customer should be able to upload more than 32,000 packages with different packages names to a single org.
## Specification




1. Solution to issue #1.
 Use the md5sum of a package as a differentiation.  If two packages have different md5sums, they are considered to be different packages.

Things that will need to be changed:  
 * DB
  * Unique constraint on md5sum,org for the rhnPackage table.
 * rhnpush 
  * Will need to be modified to push package to a new file system location (see below)
  * add support such that if a package has to be repushed on the file system, its path in the DB will be updated
 * Satellite-sync 
  * add support to push package to new file system location (see below)
  * add support such that if a package has to be redownloaded/replaced on the file system, its path in the DB will be updated (in case /var/satellite was blown away).
 * New file system location: (/var/satellite/)/redhat/ORG/MD5(0..2)/N/V-R-E/A/MD5SUM/file.rpm  to avoid hitting limits on number of entries in directory and avoid directories with only a single file.  (i.e. foomatic-1.1.i386 would be in /var/satellite/redhat/ORG/1/a3f/foomatic/1.1/i386/a3f198e89834a34/foomatic-1.1.i386.rpm). The older path was /var/satellite/redhat/ORG/1/N/V-R-E/arch/file.rpm.
  * An upgrade script will be provided to move packages to their new location and update the db


2. Solution to issue #2
  Whenever importing a package, store its GPG key id within the db in the rhnPackage table.  The user can then associate GPG keys to providers.  The idea here is that we are tracking package sources based on something relevant to the package (and something the user doesn't have to input).  
  

Things that will need to be changed:

 * DB
  * rhnPackage table will have new field rhnPackage.gpgId referencing rhnPackageGpgId
  * Table will be added rhnPackageGpgId which will have id, gpgId, modified, created
  * Table will be added rhnPackageProvider with an id, name, modified time, created time.  And will be pre-populated with equivalent labels/names:
   * Red Hat Inc.
   * Fedora
   * CentOS
   * OpenSuse
   * Novell
   * Scientific Linux
   * Oracle
  * Table will be added rhnPackageGpgProvider, linking rhnPackageProvider and rhnPackageGpgId with columns gpgId.id, provider.id...

 * rhnpush 
  * will need to grab the gpg id from the rpm and push it to the DB
 * satellite-sync
  * will need to grab the gpg id from the rpm and push it to the DB
 * webUi
  * Add provider/gpg information to certain package lists
  * Provide a way to link providers to gpg keys (optional)
 * Migration script
  * migrate existing package paths on the filer and db 
  * update/include gpg key id info in the db for existing packages.
  * fixes the pkg permissions to be readable for tomcat


Notes:
 * Apparently up2date handles multiple packages with the same NVREA within the same channel tree fine.  That surprised us both.  


Questions
 * Should we offer a way to add package providers easily? (immediate term or longer term)?

Bugzilla
 * There is a bugzilla open on this:  https://bugzilla.redhat.com/show_bug.cgi?id=453455
## GPG ids



    25dbef78a7048f8d  Scientific
    5326810137017186  Red Hat
    219180cddb42a60e  Red Hat
    b44269d04f2a6fd2  Fedora
    a8a447dce8562897  CentOs
    2802e89216ff0e46  CentOs
    a53d0bab443e1821  CentOs
    7049e44d025e513b  CentOs
    66ced3de1e5e0159  Oracle
    2e2bcdbcb38a8516  Oracle
    a84edae89c800aca  OpenSuse

