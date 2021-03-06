## API Introduction
 


Spacewalk provides an Application Programming Interface (API) to enable creation of tools and programs to automate common tasks typically performed through the Spacewalk Web User Interface.  The tools/programs may be implemented in any programming language that provides an XML-RPC client interface; however, Python and Perl are the two most commonly used languages.
## API Development



We are in the process of defining and developing additional APIs.

To view the APIs currently available for Spacewalk, simply navigate to https://myspacewalkserver.com/rhn/apidoc/index.jsp, where _myspacewalk.com_ is your installed Spacewalk server.

If you have an idea or need for a new API, please add it to the API Proposal table below.  If you aren't sure exactly what an API call should be named, simply add it to the scratchpad below and a developer will add it to the Planned APIs table.
### Scratchpad



 * Be able to add CVE's to errata
 * Change base- and child-channel for system groups, not just single systems
 * idea2
 * idea3
### Proposed APIs




|   *Task*  |  *Target Release*  |   *Estimate (hours)*  |  *Developer*  |  *Status*  |  *Notes*  |
| --- | --- | --- | --- | --- | --- | --- |
|   ----    |    |    |    |    |    |  |


|   *Namespace: channel.software*  |    |    |    |    |    |  |
| --- | --- | --- | --- | --- | --- | --- |
|   channel.software.cloneByDate(org_label, CHANNEL_DETAILS, date)  |  ?  |  3  |    |    |    |  |


|   *Namespace: errata*  |    |    |    |    |    |  |
| --- | --- | --- | --- | --- | --- |
|   errata.linkCveToErrata(session, advisory_name, array(cveNames))  |  ?  |  4  |   |    |  return int (1) on success, exception thrown otherwise.  |
|   errata.createCve(session, cveName)  |  ?  |  4  |   |    |  return int (1) on success, exception thrown otherwise.  |


|   *Namespace: system.config*  |    |    |    |    |    |  |
| --- | --- | --- | --- | --- | --- |
|   system.config.deployFiles(auth, array(string filename), array(sid))  |  ?  |  3  |    |    |  schedule deployment of specific files to specific systems  |
