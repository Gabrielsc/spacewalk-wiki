# Search Improvements 0.6+ (US-06 1003)

### Reviewed by:

## Overview



Define why the feature should be added, this could be because of customer feedback, marketing reasons, etc.
## Requirements



Bugs from PRD:

| Req #    |  bug #  |  Title  |  Description                                                         |  Comments  |
| --- | --- | --- | --- | --- |
| 1003-00  |  [bug 452318 ](https://bugzilla.redhat.com/show_bug.cgi?id=452318)  |  Ability to filter errata so as to apply only filtered errata to a system  |  Would like the ability to selectively filter errata by type, and to apply errata by type. This would allow an organization to only apply certain types of patches (e.g. Security only). For example, for a given system there would be a method to apply only RHSA errata. The ability to export this list into a CSV file would be nice too. Finally, a status report summary (exportable as well) of the patching process would be a welcome addition  |  Filter errata by type and apply is already implemented.  Need to add CSV at: /rhn/systems/details/ErrataList.do and CSV to a summary page of what was deployed. When errata are deployed, do we get a status of what errata were successfully deployed, what errata failed?  BZ asks for a report of this activity, maybe a CSV list would acceptable?  |
| 1003-01  |  [bug 472889](https://bugzilla.redhat.com/show_bug.cgi?id=472889)   |  Filter by events by package  |  User would like to know how they can filter the history of what package has been installed and what package failed to be installed. Customer would like to be able to filter RHN Events to search for failed package installations. Currently, Event History is not filterable. This will allow finer grained control/visibility into system states regarding what needs to be upgraded. Currently there is an all or nothing approach - if one package fails to install, the entire event fails. The customer would like to see partial installations and failures. Currently, when building the transaction set, if one package fails to install the whole transaction won't install. In order for the customer to filter the results, we would have to change how the transaction set is carried out. Instead of building a complete transaction set and then trying to install all at once - in which case one package with a missing dependency would cause the whole transaction set to fail - we would have to install one package and its dependencies at a time. This would then allow for some installations/updates and some failures. At that point, Event History could then be filtered for package installation or failure.  |   Concern about request for breaking package install into fine grained/individual transactions, sounds similar to case with errata where individual errata actions cause excess network I/O and poor performance   |
| 1003-02  | [bug 483118](https://bugzilla.redhat.com/show_bug.cgi?id=483118)  |  Need Filter by Synopsis - while Applying errata when working with system groups  |  Requests "Filter by Synposis" for viewing Errata under system profile. "System" -> "Click System Profile" -> "Software" -> "Errata" Desire "Filter by Synposis". This has already been added as of 5.3.0 ISO 6/5/09 this is functional  2nd request is for "Filter by Synposis" to be added to System Set Manager "Errata" when working with a group. As of 530 ISO 6/5/09, this functionality is lacking.  Filter by errata synopsis, and apply results to a group of systems  |  Currently in perl (network/systems/ssm/errata/index.pxt).  Port to Java, add filter by type functionality and CSV  |
| 1003-03  |  [bug 481626](https://bugzilla.redhat.com/show_bug.cgi?id=481626)  |  Filtering and sorting of pending (and other) events  |  Need to be able to monitor progress of errata updates on satellite to verify when they complete   |  Probably can monitor taskomatic queue for errata actions and provide updates from that  |

Requirements this feature must fulfill. Include any use cases which are used to drive this feature. Be descriptive, how does it affect the command line tools? What about the web UI? How will the user interact with the feature?
## Other Feature Impact



Describe how this new feature / change will affect other features in the product, both existing and new ones.
* UI
* API
## Proposed Implementation



Describe how we plan on implementing or fixing the feature. Point out changes that need to occur in UI, server-side, command line clients, database, etc.

 * UI
  * Add a URL widget to the some page.
 * Server-side
  * Add a server-side task to download the data from some known location.
### Known Issues

issues that we know exist with the spec and we should write them down so that they are

known up front when the spec is approved.  If an issue is severe enough, it
could warrant the feature not being included in the release/roadmap or
delayed until those issues can be addressed. 
### Future Enhancements
## Mockups



Supply some pseudo code and/or links to actual sample code. Screenshots are also considered sample code.
## Tasks



Break down the work items probably in a table structure. Is there webui work? what about cli? database? etc.

|  Description  |  Estimate  |  Confidence  |
| --- | --- | --- |
|  Add support user role to database  |  1h  |  5  |
|  Make support user role available to UI  |  4h  |  4  |
## Test Notes



Describe a high-level test plan. Could contain links to testopia (except those links are internal so would look weird in a spec on Spacewalk's wiki).
## Risks/Concerns



List out the risks and concerns for a particular feature. I.e. will it make upgrades particularly difficult? Will it cause a full regression test? etc.
## Potential Work Items - Not mentioned in PRD

 * Add in support for handling deletion of errata (might be related to bug 510122)

 * Add in sanity check of search indexes, possibly create a uniq ID stored per index and in DB, compare to ensure that index files and DB are in sync.   Or do away with DB for storing state of indexes completely.  Store state in index directory.
 * Add support to recover from error cases of an index being locked.  Below is a result example which stopped new erratas from being indexed. 

     INFO  - Job updateIndex.errata threw a JobExecutionException: 
     org.quartz.JobExecutionException: com.redhat.satellite.search.index.IndexingException: org.apache.lucene.store.LockObtainFailedException: Lock obtain timed out: SimpleFSLock@/usr/share/rhn/search/indexes/errata/write.lock  [See nested exception: com.redhat.satellite.search.index.IndexingException: org.apache.lucene.store.Lock
    ObtainFailedException: Lock obtain timed out: SimpleFSLock@/usr/share/rhn/search/indexes/errata/write.lock]
            at com.redhat.satellite.search.scheduler.tasks.IndexErrataTask.execute(IndexErrataTask.java:81)
            at org.quartz.core.JobRunShell.run(JobRunShell.java:203)
            at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:520)
    * Nested Exception (Underlying Cause) ---------------
    com.redhat.satellite.search.index.IndexingException: org.apache.lucene.store.LockObtainFailedException: Lock obtain timed out: SimpleFSLock@/usr/share/rhn/search/i
    ndexes/errata/write.lock
            at com.redhat.satellite.search.index.IndexManager.addToIndex(IndexManager.java:241)
            at com.redhat.satellite.search.scheduler.tasks.IndexErrataTask.indexErrata(IndexErrataTask.java:146)
            at com.redhat.satellite.search.scheduler.tasks.IndexErrataTask.execute(IndexErrataTask.java:67)
            at org.quartz.core.JobRunShell.run(JobRunShell.java:203)
            at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:520)
    Caused by: 
    org.apache.lucene.store.LockObtainFailedException: Lock obtain timed out: SimpleFSLock@/usr/share/rhn/search/indexes/errata/write.lock
            at org.apache.lucene.store.Lock.obtain(Lock.java:85)
            at org.apache.lucene.index.IndexWriter.init(IndexWriter.java:692)
            at org.apache.lucene.index.IndexWriter.init(IndexWriter.java:672)
            at org.apache.lucene.index.IndexWriter.<init>(IndexWriter.java:524)
            at com.redhat.satellite.search.index.IndexManager.getIndexWriter(IndexManager.java:346)
            at com.redhat.satellite.search.index.IndexManager.addToIndex(IndexManager.java:222)
            ... 4 more
# PROTOSPEC DOCUMENT

### TOPIC: US-06 1003 – Search improvements


This is a dumping grounds for specific customer-driven search and filtering improvements.

### REQUESTOR: jmatthews

=== REVIEWED BY: ===

### RELEASE: Spacewalk 0.6
## BUSINESS DEFINITION

Define why the feature should be added, this could be because of customer feedback, marketing reasons, etc.

## OTHER FEATURE IMPACT

Describe how this new feature / change will affect other features in the product, both existing and new ones.

## PROPOSED IMPLEMENTATION

Describe how we plan on implementing or fixing the feature. Point out changes that need to occur in UI, server-side, command line clients, database, etc.


 * UI
  * Add a URL widget to the some page.
 * Server-side
  * Add a server-side task to download the data from some known location.
## USE CASES
 

Define the use cases which are used to drive this feature. Be descriptive, how does it affect the command line tools? What about the web UI? How will the user interact with the feature?

Personas
Directors of IT and Sysadmins

Preconditions (assumptions)
Installed Satellite with out of date managed systems.

Main Flow / Description
 * Search for errata, filtering by the various identified types.
 * Apply the filtered results to selected systems.
 * Event history is not filterable. There is a deep desire to be able to filter based on results. E.g. Show me only failed actions on a system or set of systems. Which packages failed to install?
 * Extend “filter by synopsis” to all listings of errata, to include errata information for a specific system.
 * The final bugzilla surrounds gleaning information about updating systems, both singly and as a group.
## SAMPLE CODE

Supply some pseudo code and/or links to actual sample code. Screenshots are also considered sample code.

## WORK ITEMS

Break down the work items probably in a table structure. Is there webui work? what about cli? database? etc.


* Fulfill the core goals of these bugzillas: 452318, 472889, 483118, 481626

Requirements:

| Req #    |  Description                                                         |  bug #  |  Comments  |
| --- | --- | --- | --- |
| 1003-00  |  Filter errata so as to apply only filtered errata to a system  |  [bug 452318 ](https://bugzilla.redhat.com/show_bug.cgi?id=452318) |  Functionality implemented already, just needs CSV at: /rhn/systems/details/ErrataList.do  |
| 1003-01  |  Filter by events by package  |  [bug 472889](https://bugzilla.redhat.com/show_bug.cgi?id=472889) |  Concern about request for breaking package install into fine grained/individual transactions, sounds similar to case with errata where individual errata actions cause excess network I/O and poor performance   |
| 1003-02  |  Filter by errata synopsis, and apply results to a group of systems  |  [bug 483118](https://bugzilla.redhat.com/show_bug.cgi?id=483118) |  Currently in perl (network/systems/ssm/errata/index.pxt).  Port to Java, add filter by type functionality and CSV  |
| 1003-03  |  Need to be able to monitor progress of errata updates on satellite to verify when they complete   |  [bug 481626](https://bugzilla.redhat.com/show_bug.cgi?id=481626) |  Probably can monitor taskomatic queue for errata actions and provide updates from that  |


Details:
 452318 
  Description of problem:
  Would like the ability to selectively filter errata by type, and to apply 
  errata by type. This would allow an organization to only apply certain types of 
  patches (e.g. Security only). 

  For example, for a given system there would be a method to apply only RHSA 
  errata. The ability to export this list into a CSV file would be nice too. 
  Finally, a status report summary (exportable as well) of the patching process 
  would be a welcome addition. 


 472889
  User would like to know how they can filter the history of what package has been
  installed and what package failed to be installed.

  Customer would like to be able to filter RHN Events to search for failed
  package installations.  Currently, Event History is not filterable.


  This will allow finer grained control/visibility into system states
  regarding what needs to be upgraded.  Currently there is an all or nothing
  approach - if one package fails to install, the entire event fails.  The
  customer would like to see partial installations and failures.

  Currently, when building the transaction set, if one package fails to
  install the whole transaction won't install.  In order for the customer to
  filter the results, we would have to change how the transaction set is
  carried out.

  Instead of building a complete transaction set and then trying to install
  all at once - in which case one package with a missing dependency would
  cause the whole transaction set to fail - we would have to install one
  package and its dependencies at a time.  This would then allow for some
  installations/updates and some failures.

  At that point, Event History could then be filtered for package
  installation or failure.

 483118 
  Requests "Filter by Synposis" for viewing Errata under system profile.
  "System" -> "Click System Profile" -> "Software" -> "Errata"
  Desire "Filter by Synposis".  This has already been added as of 5.3.0 ISO 6/5/09 this is functional

  2nd request is for "Filter by Synposis" to be added to System Set Manager "Errata" when working with a group.
  As of 530 ISO 6/5/09, this functionality is lacking.

 481626
  Need to be able to monitor progress of errata updates on satellite to verify when they complete
## TEST PLAN

Describe a high-level test plan. Could contain links to testopia (except those links are internal so would look weird in a spec on Spacewalk's wiki).

## RISKS/CONCERNS

List out the risks and concerns for a particular feature. I.e. will it make upgrades particularly difficult? Will it cause a full regression test? etc.


















