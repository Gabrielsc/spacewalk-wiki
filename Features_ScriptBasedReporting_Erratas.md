# Basic FISMA Report: Software inventory report - errata out of compliance information



2009-10-23: spacewalk-backend 0.7.9-1 contains two errata reports.

2009-10-30: spacewalk-backend 0.7.11-1 adds column type to the errata-list report. 

Report fields:


    # spacewalk-report --info --list-fields-info errata-list
    Errata out of compliance information - errata details
    
        List of erratas that are applicable to at least one registered
        system, together with basic info about the errata. Also see
        errata-systems report to get list of systems that are affected.
    
    Fields in the report:
    
        advisory: Advisory / errata identifier
        type: Advisory type (Enhancement, Bug Fix, Security)
        cve: List of CVE names (Common Vulnerabilities and Exposures Identifiers) addressed by the errata
        synopsis: Synopsis of the errata
        systems_affected: Number of systems to which this errata is applicable


    # spacewalk-report --info --list-fields-info errata-systems
    Errata out of compliance information - erratas for systems
    
        List of applicable erratas and systems that are affected.
    
    Fields in the report:
    
        advisory: Advisory / errata identifier
        server_id: System identifier
        profile_name: Profile name, as stored on server
        hostname: Hostname, as reported by the system
        ip_address: IP address, as reported by the system

Note that there is no severity field in the errata-list report, as the severity table is empty on Satellite (5.3.0).

The output:


    # spacewalk-report errata-list | tail
    RHSA-2009:1504,Security Advisory,CVE-2009-3603;CVE-2009-3608;CVE-2009-3609,Important: poppler security and bug fix update,43
    RHSA-2009:1512,Security Advisory,CVE-2009-0791;CVE-2009-1188;CVE-2009-3604;CVE-2009-3608;CVE-2009-3609,Important: kdegraphics security update,3
    RHSA-2009:1513,Security Advisory,CVE-2009-3608;CVE-2009-3609,Moderate: cups security update,115
    RHSA-2009:1522,Security Advisory,CVE-2005-4881;CVE-2009-3228,Moderate: kernel security and bug fix update,38
    RHSA-2009:1528,Security Advisory,CVE-2009-2906,Moderate: samba security and bug fix update,3
    RHSA-2009:1529,Security Advisory,CVE-2009-1888;CVE-2009-2813;CVE-2009-2906;CVE-2009-2948,Moderate: samba security update,90
    RHSA-2009:1530,Security Advisory,CVE-2009-1563;CVE-2009-3274;CVE-2009-3370;CVE-2009-3372;CVE-2009-3373;CVE-2009-3374;CVE-2009-3375;CVE-2009-3376;CVE-2009-3380;CVE-2009-3382,Critical: firefox security update,167
    RHSA-2009:1531,Security Advisory,CVE-2009-1563;CVE-2009-3274;CVE-2009-3375;CVE-2009-3376;CVE-2009-3380,Critical: seamonkey security update,9
    RHSA-2009:1535,Security Advisory,CVE-2009-2703;CVE-2009-3083;CVE-2009-3615,Moderate: pidgin security update,2
    RHSA-2009:1536,Security Advisory,CVE-2009-3615,Moderate: pidgin security update,18
    # spacewalk-report --multival-on-rows errata-list | tail
    RHSA-2009:1530,Security Advisory,CVE-2009-3382,Critical: firefox security update,167
    RHSA-2009:1531,Security Advisory,CVE-2009-1563,Critical: seamonkey security update,9
    RHSA-2009:1531,Security Advisory,CVE-2009-3274,Critical: seamonkey security update,9
    RHSA-2009:1531,Security Advisory,CVE-2009-3375,Critical: seamonkey security update,9
    RHSA-2009:1531,Security Advisory,CVE-2009-3376,Critical: seamonkey security update,9
    RHSA-2009:1531,Security Advisory,CVE-2009-3380,Critical: seamonkey security update,9
    RHSA-2009:1535,Security Advisory,CVE-2009-2703,Moderate: pidgin security update,2
    RHSA-2009:1535,Security Advisory,CVE-2009-3083,Moderate: pidgin security update,2
    RHSA-2009:1535,Security Advisory,CVE-2009-3615,Moderate: pidgin security update,2
    RHSA-2009:1536,Security Advisory,CVE-2009-3615,Moderate: pidgin security update,18
    # spacewalk-report errata-systems | tail
    RHSA-2009:1522,1000012916,xen12.example.com,xen12.example.com,10.12.24.12
    RHSA-2009:1522,1000012918,xen69.example.com,xen69.example.com,10.12.24.69
    RHSA-2009:1522,1000013269,jasan-chroot-rhel4,slanina.virtual.jsn,192.168.1.254
    RHSA-2009:1522,1000013401,dell-pe1950-04.example.com,dell-pe1950-04.example.com,10.12.23.69
    RHSA-2009:1522,1000013428,xen84.example.com,xen84.example.com,10.12.24.84
    RHSA-2009:1522,1000013667,rhel4.virtual.jsn,rhel4.virtual.jsn,192.168.1.2
    RHSA-2009:1522,1000013927,rhel4s-x86_64-work,,
    RHSA-2009:1522,1000014067,hp-dl360-05.example.com,hp-dl360-05.example.com,10.12.23.59
    RHSA-2009:1522,1000014547,vmware146.example.com,vmware146.example.com,10.12.24.146
    RHSA-2009:1522,1000014787,nahan,unknown,192.168.122.145

----

Back to [[Features_ScriptBasedReporting]].