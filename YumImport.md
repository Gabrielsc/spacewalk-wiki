
# **DEPRECATED, NO LONGER USED**

# PROTOSPEC DOCUMENT

### TOPIC: Import content from a Yum repo


=== REQUESTOR: jmrodri ===

### REVIEWED BY:

=== RELEASE: Spacewalk 0.6 ===
## Overview

Make it easier to import content into Spacewalk. Today a user must download a repo locally then upload the packages via `rhnpush`.

See UploadFedoraContent for an explanation of what is involved.
## Requirements/Use Cases
 


 * User wants to sync Fedora repo to a channel
 * User has private yum repo and wants to bring that into his spacewalk server
## Other Feature Impact

None

## Proposed Implementation



 * UI
  * Channel Creation Page
    * add URL widget to allow one to define the yum repo.
    * add a check box to indicate if channel should be synced after creation
  * Channel Edit Page 
    * Add a URL widget to allow one to change the yum repo.
    * Add a sync button
    * Add last synced time
  * Channel Details page
    * Show yum repo info
    * Show last synced time
  * sync task will be performed asynchronously.  
  * Sync only be additive, packages removed from the yum repo will remain in the channel
  * To initiate sync, task is added to new Taskomatic queue
 * Taskomatic task will pick up from queue and use new yum repo module
 * Yum Repo interaction will use Yum module from RHQ
 * Api support for syncing channel, setting repo (working titles)
  * channel.software.repo.set
  * channel.software.repo.sync  
  * channel.software.repo.syncAll  
 * Schema Changes
  * Add rhnChannelRepo table to store  repo -> channel (many-to-one) mapping
 * Config option will be added to disable all Yum UI snippets and API calls
#### Domain Package <--> Yum Metadata



Mapping between com.redhat.rhn.domain.rhnpackage.Package _domain_ object and YUM metadata.

|| *Package Field* || *Description* || *Yum Metadata* _<document>_:_<xpath>_ ||
|| id || unique package identifier || primary.xml:/package/checksum ||
|| rpmVersion || The version of the RPM tool? || _none_ ||
|| description || Description || primary.xml:package/description ||
|| summary || Summary || primary.xml:package/summary ||
|| packageSize || Package file size (bytes) || primary.xml:/package/size/@package ||
|| payloadSize || Package installed content size (bytes) || primary.xml:/package/size/@installed ||
|| buildHost || Hostname where package was built || primary.xml:/package/format/rpm:buildhost ||
|| buildTime || Package build datetime || primary.xml:/package/time/@build ||
|| md5sum || The package md5 || primary.xml:/package/checksum[[@type=_md5_]] ||
|| vendor || The package vendor || primary.xml:/package/format/rpm:vendor ||
|| payloadFormat || ?? || ?? ||
|| compat || ?? || ?? ||
|| path || Filesystem path within Spacewalk? || ?? ||
|| headerSignature || The package header signature || ?? ||
|| copyright || Package copyright || primary.xml:/package/format/rpm:license ?? ||
|| cookie || ?? || ?? ||
|| lastModified || File last modified date/time || primary.xml:/package/time/@file ||
|| sourcePath || RPM Source0 path? || ?? ||
|| org || orgid || n/a ||
|| packageName || Package name || primary.xml:/package/name ||
|| packageEvr || EVR || primary.xml:/package/version/@epoch, primary.xml:/package/version/@ver, primary.xml:/package/version/@rel ||
|| packageGroup || RPM package group || primary.xml:/package/format/rpm:group ||
|| sourceRpm || Source RPM || primary.xml:/package/format/rpm:sourcerpm ||
|| packageArch || Architecture || primary.xml:/package/arch ||
|| packageKeys || Keys used to sign the package? || ?? ||
|| headerStart || RPM header (start) offset (bytes) || primary.xml:/package/format/rpm:header-range/@start ||
|| headerEnd || RPM header (end) offset (bytes) || primary.xml:/package/format/rpm:header-range/@end ||
|| publishedErrata || List of published errata || n/a ||
|| unpublishedErrata || List of published errata || n/a ||
|| channels || List of associated channels || n/a ||
|| packageFiles || List of files owned by package || primary.xml:/package/format/file ||
|| changeLog || The rpm changelog || other.xml:/otherdata/package[[@pkgid=__checksum__]]/changelog ||
|| provides || List of provided items || primary.xml:/package/format/provides ||
|| requires || List of required packages || primary.xml:/package/format/requires ||
|| obsoletes || List of obsoleted packages || primary.xml:/package/format/obsoletes ||
|| conflicts || List of conflicting packages || primary.xml:/package/format/conflicts ||
## SAMPLE CODE

### Channel creation page




Sample image of what kickstart details will look like (minus the 'sync schedule feature')

[[20%)](Image(channel-creation.png,)]
## WORK ITEMS




|  Task  |    Estimate  |  Confidence  |
| --- | --- | --- |
|  Schema changes (repo & taskomatic)  |  1 day  |  5   |
|  Port/Integrate RHQ's yum syncing module  |    3 days    |  4    |
|  Adding support code for repo syncing  |       |      |  |
|  Taskomatic Additions               |   2 days       |  4    |
|  UI Changes       |   2 Days  |  5  |
|  Apis             |   2 Days  |  5  |
|  Update Spacewalk Docs  |  1 Day  |  5  |
## TEST PLAN

Need to test syncing mechanism, as well as the resulting packages and channels to ensure they were created properly

## RISKS/CONCERNS

Reading GPG key from package file

## Future Features

[link](YumImport2)


