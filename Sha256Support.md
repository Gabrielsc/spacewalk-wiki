# SHA256 Support for Spacewalk

## Overview


 * Fedora 11 Clients with sha256 supported content should be able to interact with spacewalk and proxy as expected

 * spacewalk server should be able to support newer rpm content in addition to old
 * proxy should be able to serve and accept newer rpm content.
 * webui should display the right checkSum based on the rpm
 * API calls should be be able to handle sha256 checksum both as input args and output results
 * Both proxy and spacewalk tools should handle shasum.
 * if possible move to shasum replacing md5 for other non-rpm verifications on spacewalk/proxy
## Background - RPM changes



There are several places in rpm format where a kind of checksum/signature is used. An old rpm format (used e.g. RHEL5) uses MD5 and SHA1. A new rpm format (e.g. in Fedora 11) uses MD5, SHA1 and SHA256.

|| || old rpm ||  || new rpm || ||||
|| || hash || rpm tag || hash || rpm tag ||
|| Header digest || SHA1 || || SHA1 ||
|| Header signature || V3 DSA (SHA1) || SIGTAG_GPG SIGTAG_DSA || V3 RSA/SHA256 || SIGTAG_PGP SIGTAG_RSA
|| (Payload) digest || MD5 || || MD5 ||
|| (Payload) signature || V3 DSA (SHA1) || SIGTAG_GPG SIGTAG_DSA || V3 RSA/SHA256 || SIGTAG_PGP SIGTAG_RSA

|  (Payload) files checksum  |  MD5  |  RPMTAG_FILEMD5S  |  SHA256  |   RPMTAG_FILEDIGESTS RPMTAG_FILEDIGESTALGO  |
| --- | --- | --- | --- | --- |


See: [RPM file format changes to support SHA-256](https://fedoraproject.org/wiki/RPM_file_format_changes_to_support_SHA-256)
## Proposed Implementation
  


 * Create rhnChecksum table which is able to hold a checksum and it's type for any (general) content.
 * Replace md5 columns in various tables with reference to a row in rhnChecksum table.
 * Modify web pages so we can show different checksums (i.e. longer than md5 and also show type of checksum).
 * For old rpms we have to store md5 sums. For new sha256 rpm we have to store sha256 checksums.
 * Store checksum type along with channel date which will be used for (yum) metadata repo generation (sha1 for RHEL5 & Fedora 10, sha256 for Fedora 11).
 * Have repo checksum type be a channel variable (defined by user at channel creation time).
 * Switch md5 checksums to sha256 whereever it's possible (i.e. checksum is computed and used on spacewalk server / client so we can modify both data & checksum producer and consumer).
### Database(estimates: 30 hrs)



Current checksum usage in spacewalk:
 * rhnPackageFile 
 * rhnPackage
 * rhnKSTreeFile    
 * rhnPackageSource  
 * rhnErrataFile     
 * rhnErrataFileTmp      
 * rhnFile
 and may be more.

 * New tables rhnChecksum and rhnChecksumType will be added to accumulate checksum of any content in satellite:


    
    SQL> desc rhnChecksumType
     Name					   Null?    Type
     ----------------------------------------- -------- ----------------------------
     ID					   NOT NULL NUMBER
     LABEL					   NOT NULL VARCHAR2(32)
     DESCRIPTION				   NOT NULL VARCHAR2(64)
     CREATED				   NOT NULL DATE
     MODIFIED				   NOT NULL DATE
    
    Sample Data:
    1, md5, md5sum, 26-JUN-09, 28-JUN-09
    2, sha1, sha256sum, 26-JUN-09, 28-JUN-09
    3, sha224, sha224sum, 26-JUN-09, 28-JUN-09
    4, sha256, sha256sum, 26-JUN-09, 28-JUN-09
    5, sha384, sha384sum, 26-JUN-09, 28-JUN-09
    6, sha512, sha512sum, 26-JUN-09, 28-JUN-09
    
    SQL> desc rhnChecksum
     Name                                                  Null?    Type
     ----------------------------------------------------- -------- ------------------------------------
     ID                                                                                          NOT NULL NUMBER
     CHECKSUM_TYPE_ID			               NOT NULL NUMBER
     CHECKSUM				               NOT NULL VARCHAR2(128)
    
    Sample Data:
    1001, 1, 1g367fcfe8fb56662d2831d5cce8a5157db75
    1002, 1, 2f367fcfe8fb56662d2831d5cce8a5157db56
    1003, 2, 1906da045a9d4468e5d678a0cebe7ada3ae0f4f01ec7da0e39cba3277974becw
    

 * Move checksum(s) from all tables to rhnChecksum
 * replace current md5sum columns with a foreign key to rhnChecksum
### Spacewalk Server (estimates: 60 hours)

 

 * Repo Generation:
  * server side repo generation should include shasum support for channels with newer rpm content and be backward compatible with older clients.
  * What if a custom channel has mixed content? yum doesn't care about whats in the channel, it only cares about the repo format. So as long as the repomd.xml is of compatible format it should work. 
  * Custom channels have type of yum repo checksum associated by user at creation time.
  * For imported / synced channels we have to decide the checksum type at (their) creation time.

 * Importer/Exporter + ISS:
  * This should work as expected as all we're doing is extracting the content from the db and caching it into xml files. But we'll need to fix queries specifically dealing with above tables. 

 * App Handler:
  * compares md5sums to check if pkgs is already uploaded, move that to a general checksum instead
  * fix other getMd5Sum calls used on the server

 * xp Handler:
  * rhn-package-manager server side changes to support sha256

 * Other:
  * We have other places where we make assumptions about md5sum so migrate those to shasum.
  * make sure the /var/satellite paths are formed as expected with any checksum.
### Proxy (Estimates: 40 hours)

Checksum is used only in proxy during kickstart. Proxy replace /ty/_sessionid_ with /ty/_packagechecksum_ so squid can cache it. Proxy do not care if that checksum is MD5 or SHA256 or whatever. Only requirement is that it should be unique for each package.

Only small problem can be if one package is sent with MD5 checksum (checksum is taken from HTTP header) for older client and then for newer client with SHA256 checksum. Proxy will then treat this two as different content. So it will store one package twice. We may change the logic to always require only specific checksum.

* package uploads through proxy should work as expected for newer rpms.

Packages uploaded through rhn_package_manager to local proxy channel should handle sha256 properly.
### Web UI (Estimates: 20 hours)



* pages displaying checksum:
 * packageDetails
 * Errata
 * packageFilelists

* package details page will now show a new field called checksum type

We need to make sure the queries used by these pages reflect any new db changes that effect checksum display. We now assume md5sum by default.
### API( Estimates:  20 hours)

 * need to fix the hibernate objects and some cosmetic stuff by renaming the instances

  * packageHandler.getDetails - include checksum type as one of the return value
  * packageHandler.listFiles
  * errataHandler.listPackages
### Client( Estimates:  40 hours )

 * yum + rhn-plugin:

  * yum repodata (repodata file checksums, rpm file checksums)
  * yum-rhn-plugin plays well with new yum and rpm
  * older yum clients dont understand sha256 so they will fail if the repo is not sha1.
  * so we'll need to test that the repo generated on the server is compatible with newer/older clients. 

 * Scheduled actions:
  * scheduled actions need to be tested and made sure work on f11
  * fix rpmUtils and supported calls to incorporate any rpm api changes
  * rpm verify action verifies newer rpms as expected
   * this deals with rhnServerActionVerifyResult
### Tools ( Estimates: 20 hours x 2 = satellite(prad) + proxy(msuchy))

 * satellite sync:

  * Need to make sure syncing content works as expected for sha256 supported content
  * Make sure satellite-sync runs without issues on f11
  * populate shasum into rhnPackage for newer packages and md5sum for older
  * Populate checksum type for channels created by satellite-sync

 * rhnpush:
  * sha256 supported package can be uploaded  as expected
  * populate shasum into rhnPackage for newer packages and md5sum for older
  * might wanna change the package verify to shasum as well

 * rhn-package-manager(Proxy):
  * sha256 supported package can be uploaded as expected
  * Populate checksum type for channels created by rhn-package-manager

 * rhn-ssl-tool:
  * rhn-ssl-tool deploys rpms. It currently calls gen-rpm call to generate the spec and create and deploy em
  * need to test and make sure this area is able to generate f11(with sha256) and older rpms (as usual)
### Upgrades ( Estimates: 40 hours)

 * Any db changes should be upgradable from older satellites

 * Any existing content should be upgradable as well. 
 * An existing checksums have to be moved to new rhnChecksum table.
### Other Feature Impact:

 * Nvrea feature needs to be revisited to handle regressions
## Tasks (in order)




|  Task  |  Estimated Hours  |  Assignee  |  Status  |  Notes  |
| --- | --- | --- | --- | --- | --- |
|  Database Changes   |  40 hours  |  mmraka/prad  |    |   |  |
|  spacewalk Server + Tools |  60 hours  |  prad   |    |    |  |
|  Proxy Server + Tools  |  40 hours  |  msuchy   |   |   |  |
|  Client tools   |  40 hours  |  jmatthews + prad  |   |    |  |
|  Web UI + API   |  40 hours  |  jmatthews   |    |   |  |
|  Upgrades  |  30 hours  | mmraka/prad  |    |   |  |


The order here represents the dependency chain. Database changes would be the first to get done so we have those to develop and test the app code changes. spacewalk and proxy  development can go in parallel, but these should be done(at least partly) for client work to start so we have supporting server code for clients. Web UI and upgrades can go last I guess. 
## Future Enhancements:

N/A

## Mockups:

N/A

### Test Notes:

Will update as feature rolls.

### Risks/Concerns:

None So far.
