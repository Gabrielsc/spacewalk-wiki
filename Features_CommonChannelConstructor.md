# __Common Channel Constructor__



Note that the spacewalk-common-channels command from spacewalk-utils package provides functionality more or less equivalent to what is wanted here.

----
## Overview

This feature proposal is to extend [create_channel.py](attachment_wiki_UploadFedoraContent_create_channel.py) so that when run with say a "--addcommon" flag it presents a simple CLI menu system that allows the selection and configuration of some very common channel configurations, making such API calls as required to set them up and installs appropriate entries into the crontab to keep them up to date.

### Sponsor and Reviewer

Sponsor: philip.mather
## Requirement

== Specification ==

### Functional

=== Non-functional ===
## Implementation

Common repositories to offer predefined channel configurations for...

 * RHEL
 * CentOS
 * Fedora
 * Scientific Linux - http://ftp.scientificlinux.org/linux/scientific/
 * EPEL - http://download.fedoraproject.org/pub/epel/
 * VMWare - http://packages.vmware.com/tools
 * Spacewalk - http://spacewalk.redhat.com/yum/
 * RPMFusion - http://rpmfusion.org/
 * AIX - ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/
 * Solaris?
### Technical

=== Architectural ===
## Mock Ups

== Tests ==

## Risks

 * Deluging a single under resourced repository with unnecessary requests.

 * Concentrating requests to a single repository into to small a time frame.
## Tasks

== Progress ==

|| *Phase* || *Task* || *Description* || *Estimate* || *Confidence* || *Progress* || *Owner* ||
||             ||            ||                   ||                ||                  || 0%             ||             ||||
||             ||            || *Subtotal:*   ||                ||                  || 0%             ||             ||||
||             ||            || *__Total:__*  ||                ||                  || 0%             ||             ||||