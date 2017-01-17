# OpenSCAP



SCAP provides standardized approach to maintaining the security of systems. Beside others, OpenSCAP tool can evaluate machine compliance with a given profile. For further information about OpenSCAP please navigate to project's [homepage](http://www.open-scap.org/page/Main_Page) or Fedora's feature [page](https://fedoraproject.org/wiki/Features/OpenSCAP)
## Spacewalk and OpenSCAP



Project to integrate OpenSCAP to Spacewalk has started. The aim is to provide an easy way for admins to scan whole infrastructure and view results and trends. More info and tutorials are available at [blog](http://isimluk.livejournal.com/631.html).

With Spacewalk 1.7, user is able:
 * schedule XCCDF scan for a given machine through FrontendAPI
 * obtain summary of the scan results through spacewalk-reports tool.

With Spacewalk nightly (and post 1.7 releases) user is able:
 * schedule XCCDF scan through Spacewalk web (either for a single machine or for SSM)
 * view all scans summary on web (per single machine or whole infrastructure)
 * view a structured scan result on Spacewalk web
 * fetch scan results via FrontendAPI
 * perform search via Spacewalk Web
## How to perform scan via WEB UI

 * Prerequisites:

   * For the server side: Spacewalk 1.8 or higher
   * For the client: spacewalk-oscap package from spacewalk-client repo
   * Distribute the XCCDF content to client machines (e.g by rpm)
 * On overview page of the targeted system, select 'Audit' tab and 'Schedule' subtab.
 * Fill in the scan's form
   * The 'path' parameter points to content location on the client system.
   * The 'parameter' represents additional arguments for oscap tool, e.g (--profile to select security xccdf:profile)
 * Wait for action to be picked-up
 * Investigate results under the systems's 'Audit' tab.
## How to perform scan via API

 * Prerequisites:

   * For the server side: Spacewalk 1.7 or higher
   * For the client: spacewalk-oscap package from spacewalk-client repo
   * Distribute the XCCDF content to client machines (e.g by rpm)
 * Schedule scan through FrontendAPI (system.scap.scheduleXccdfScan)
   * The 'path' parameter points to content location on the client system.
   * The 'parameter' represents additional arguments for oscap tool, e.g (--profile to select security xccdf:profile)
 * Wait for action to be picked-up
 * Get results by spacewalk-reports tool.
   * # /usr/bin/spacewalk-reports system-history-scap
   * # /usr/bin/spacewalk-reports scap-scan
   * # /usr/bin/spacewalk-reports scap-scan-results
### Example

The following is an example of python script which will schedule XCCDF scan for machine with sid=1000010001. The script assumes USGSB content in `/usr/local/share/scap/usgcb-rhel5desktop-xccdf.xml` on the client filesystem. Parameters indicate which profile is to be evaluated; it is `united_states_government_configuration_baseline`. The content used for this example can be found at [NIST](https://usgcb.nist.gov/usgcb/content/scap/USGCB-rhel5desktop-1.2.5.0.zip)


    #!/usr/bin/python
    client = xmlrpclib.Server('https://spacewalk.example.com/rpc/api')
    key = client.auth.login('username', 'password')
    client.system.scap.scheduleXccdfScan(key, 1000010001,
        '/usr/local/share/scap/usgcb-rhel5desktop-xccdf.xml',
        '--profile united_states_government_configuration_baseline')
