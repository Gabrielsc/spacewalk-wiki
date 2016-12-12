# Comps Syncing



The comps.xml file is an XML file describing groups of packages that yum can use for group operations, things like groupinstall or groupinfo. With Fedora yum repositories, the comps.xml file is "attached" to the repository via

  <data type="group">

entry in repomd.xml.

For repos/channel served from RHN Satellite and Spacewalk servers, we need to add correct information about correct comps file to the repomd.xml file, so that the client machines are able to do the group operations.

To support group operations for RHEL 5 clients fast, we have hardcoded paths to comps files in the kickstart trees in the server code, done in [bugzilla 247156](https://bugzilla.redhat.com/show_bug.cgi?id=247156). However, that means that the kickstart trees need to be synced, and the paths always point to the 5.0 GA files, so for example RHEL 5.5 still gets 5.0 comps.xml file.

In [bugzilla 585233](https://bugzilla.redhat.com/show_bug.cgi?id=585233) we aim at supporting the comps syncing properly, both from RHN hosted to RHN Satellite, to channel dumps, and across Satellites.
## RHN hosted support



RHN hosted has added the support for exposing the comps data for channels in [bugzilla 249141](https://bugzilla.redhat.com/show_bug.cgi?id=249141). It has two parts:

 * The <rhn-channel> element in the channel XML now has <rhn-channel-comps-last-modified> element, with timestamp of the comps information, taken from hosted's rhnChannelComps table. If there is no comps information associated with the channel, the element is not present.
 * In the case when attribute or element in <rhn-channel> indicates that comps is associated, the comps file can be retrieved on the URL http://satellite.rhn.redhat.com/SAT/$RHN/$CHANNEL_LABEL/repodata/comps.xml, with correct authentication HTTP headers.
## RHN Satellite / Spacewalk syncing

### Live sync from RHN hosted




On RHN Satellite / Spacewalk side, we process the rhn-channel-comps-last-modified element. If it is present, we check if file /var/satellite/rhn/comps/$CHANNEL_LABEL/comps-$COMPS_LAST_MODIFIED.xml already exist and retrieve it with GET if it does not. Then we update the rhnChannelComps table to hold the correct record with correct relative_filename value.
### Generating channel dumps



When rhn-satellite-exporter generates the export and the channel has value in rhnChannelComps, it creates the rhn-channel-comps-last-modified element accordingly, and stores the comps.xml file gzipped in the channels/$CHANNEL_LABEL/comps.xml.gz file.
### Sync from channel dumps



When the <rhn-channel> element has the rhn-channel-comps-last-modified subelement, the comps.xml.gz file is ungzipped from channels/$CHANNEL_LABEL to /var/satellite/rhn/comps/$CHANNEL_LABEL/comps-$COMPS_LAST_MODIFIED.xml, and rhnChannelComps record is set.
### Inter-sat-sync



During inter-sat-sync, the rhn-channel-comps-last-modified subelement is created, and the satellite-sync then retrieves the comps file using new dump.get_comps XMLRPC call and stores it to /var/satellite and to rhnChannelComps like in the previous cases.
## Where are these changes available



The above changes were tagged as spacewalk-backend-1.1.11-1.

The spacewalk-java-1.1.7-1 with other comps-related fixes is also needed unless use_taskomatic_repomd is set to 0 to use the Python repomd generator.

For Satellite 5.3.*, errata [RHBA-2010:0443-1](http://rhn.redhat.com/errata/RHBA-2010-0443.html) includes the changes.