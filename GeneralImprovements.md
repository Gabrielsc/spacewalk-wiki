
    #!div class="important" style="border: 2pt solid; text-align: center" 
    '''''DEPRECATED, NO LONGER USED''''' 
# Enhanced Channel Removal

### Background


=== Requirements ===

 1. Remove any channel via
  * Script (API).
  * UI.
 1. When removing a channel, provide option to:
  * Remove associated errata.
  * Remove associated content.
 1. Log channel removal action - to include:
  * user
  * channel
  * content removed
  * systems affected
### Specification

=== Notes ===
# Spacewalk on KVM

### Background


=== Requirements ===

### Specification

=== Notes ===
## Provide Channel ISO in DVD Format

### Background


=== Requirements ===

### Specification

=== Notes ===
## Errata Management Polish

### Background


=== Requirements ===

### Specification

=== Notes ===
## Email Notification For Role Changes

### Background


=== Requirements ===

### Specification

=== Notes ===
## Monitoring

== Background ==

A lot of monitoring code is on place where it should not be.
### Requirements

 1. Libraries should be located in proper place.

 2. Service should be service.
### Specification

 1. /etc/rc.d/np.d/sysvStep should be moved to %vendor_lib

 2. /var/lib/nocpulse/libexec should be probably moved to %vendor_lib too
 3. sysVstep install is not needed - code should be cleaned
 4. service Monitoring and MonitoringScout run several sub-serviceses. They should be independent services.
### Notes


