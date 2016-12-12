## Telemetry



Telemetry is the code name for a simple template-based reporting engine for Spacewalk.  This is *only* a prototype at this point and needs a significant amount of work/help to flush out the idea/code. Telemetry uses the xmlrpc scripting [API](http://www.redhat.com/spacewalk/documentation/api/0.2/) to fetch data from Spacewalk servers.  It is written completely in Python, leveraging [Cheetah](http://www.cheetahtemplate.org/) for templating, cron for scheduling, and [Django](http://www.djangoproject.com/) for the webui.
### Requirements



 1. Must be able to leverage Satellite authentication/authorization (user/role).
 2. Must be able to execute reports immediately, or scheduled.
 3. Must be able to group reports logically.
 4. Must be able to generate different formats (.csv, .html, .txt, ...)
 5. Must be able to easily add/modify reports (including report formatting).
 6. Must be able to run reports against multiple satellites, and aggregate data if desired.
 7. Must be able to notify users upon report completion (email).
 8. Must be able to accept report criteria input from user.
 9. Reports need to be able to be packaged and updated separately from Reporting Engine.
### Architecture Diagram

![Alt](images/telemetry-diagram.jpg?raw=True)

### Source

Telemetry is current located in: /playpen/Telemetry in the spacewalk git repo -> [[https://fedorahosted.org/spacewalk/browser/playpen/Telemetry]]

### Installation



Telemetry is designed to run on any client-system.  It can be run on the Spacewalk server, but not necessarily.

*Fedora 9*

 * Django (webui)
 * Cheetah (report templating)
 * PyYAML (report config)
 * httpd 
 * mod_python


    yum install Django python-cheetah PyYAML httpd mod_python

 * python-crontab


    cd /tmp
    wget http://pypi.python.org/packages/any/p/python-crontab/python-crontab-0.7.linux-i686.tar.gz#md5=10ce70d894d6d2b1bffd1a8a339e1ed4
    tar -xvzf python-crontab-0.7.linux-i686.tar.gz -C /


 * Telemetry


    rpm -Uhv http://tsanders.fedorapeople.org/telemetry-0.1-2.noarch.rpm

 * or (I have added the dependent rpms to the .spec file)


    yum install http://tsanders.fedorapeople.org/telemetry-0.1-2.noarch.rpm  --nogpgcheck
### Sample Reports



Telemetry contains a few sample canned reports.  The reports are installed in _/usr/share/telemetry_.

 * /scripts   -> contains the xmlrpc client reports scripts (these are executable on the cmd-line or via the Telemetry webui.
 * /config    -> contains the report configuration files.
 * /templates -> contains the output format templates.

The report configuration files will *need* to be updated with the proper url for your Spacewalk server(s), and report directory.



    cd /usr/share/telemetry/config
    vi systemDetails.conf


    name: System Details Report
    description: This report provides a list of systems and supporting details (entitlements, channel subscriptions, & system groups.
    
    script: systemDetailsReport.py
    
    satellites:
    - http://rlx-1-16.rhndev.redhat.com/rpc/api  # Change this to point to the api url for your Spacewalk server(s). #
    
    api_versions:
    - 5.1.0 Java
    - 5.1.1 Java
    
    aggregate: no
    
    templates:
        txt: SystemDetails.txt
    
    prefix: SystemDetails
### Config Data Dictionary



 name::
     The name of the Report.

 description::
     A brief description of the Report.

 script::
     The .py xmlrpc client script file used to gather report data.  Script needs to be in the ''/usr/share/telemetry/scripts'' directory.

 satellites::
     The url of the xmlrpc API interface on your Spacewalk server(s).  Multiple servers can be added on separate lines.

 api_versions::
     The supported versions of the Spacewalk xmlrpc API specification.

 aggregate::
     A simple boolean value used when specifying multiple Spacewalk servers to indicate whether or not to aggregate the data into a single report.

 templates::
     The report templates that can be used with this report.  The templates need to be in the ''/usr/share/telemetry/templates'' directory. Currently, the supported types are: txt, csv, & html.

 prefix::
     The name that will be given to the generated reports.  The format will be: Report_Name + datetimestamp + output type (.txt, .csv, .html).
 
 criteria::
     (''Optional'') User supplied criteria for the generated report. 

 * name: The name of the criterion.
 * label The label of the criterion (used for the webui form field input.)
 * type  The type of the criterion (csv, txt, bool).
 * cols  The number of columns or horizontal size of the text field or text area for webui input.
 * rows  The number of rows or vertical size of the text field or text area for webui input.
 * required Boolean value to specify whether the criterion is optional.
### Web UI



The webui is leverages apache + mod_python.

*Starting*


    telemetry --setup
    /sbin/service httpd restart

*Browsing*

Browse to the following URL: http://127.0.0.1/telemetry/ or http://localhost/telemetry/
### Screen Shots



![Alt](images/Telemetry-index.jpg?raw=True)

_Telemetry webui - Index Page_

![Alt](images/Telemetry-reportlist.jpg?raw=True)

_Telemetry webui - Report List_

![Alt](images/Telemetry-reportdetails.jpg?raw=True)

''Telemetry webui - Report Details'

![Alt](images/Telemetry-reportresults.jpg?raw=True)

''Telemetry webui - Report Results'
### Step-by-Step Report Creation Walkthrough



The plan here is to walkthrough (step-by-step) the creation of a telemetry report.  We will take a simple use case:

Joe is a system administrator and would like to generate a report that lists all of the systems in his organization that have firefox installed, and report back the specific NVREA.
#### Step 1 - Writing the Report Script



First, we need to examine the Spacewalk [API](http://www.redhat.com/spacewalk/documentation/api/0.2/) and determine which methods will be used to gather the necessary information.

In our case, we will use the following:

 * system.listUserSystems(string sessionKey)
 * system.listPackages(string sessionKey, int serverId)


The psuedo-code is as follows:

 1. Prompt the user for a csv list of package names (required criteria)
 2. Get all the systems available to the logged-in user.
 3. For each system, get the list of installed packages.
 4. Compare
 5. Report Results (to include name, version, release, epoch, and arch)

We are now ready to open our favorite editor and start coding:
#### systemsByPackageReport.py
 

[[https://fedorahosted.org/spacewalk/attachment/wiki/PlayPen/Telemetry/systemsByPackageReport.py]]



    #!/usr/bin/env python
    
    import sys
    import xmlrpclib
    from telemetry import Report

telemetry.Report is a helper class that handles the xmlrpc connections, template rendering, etc.


    class System():
        pass
    
    class Package():
        pass

Define data structures to hold our result sets.  In this case we will create and array of System objects, which will contain an array of Package objects.


    # Main Processing
    if len(sys.argv) != 6:
        print "Usage %s <config> <type> <username> <password> <packagelist>" % (sys.argv[0])
        sys.exit(1)

Ensure that *all* 5 required arguments have been supplied to our report script.


    # Arguments
    config = sys.argv[1]
    type = sys.argv[2]
    username = sys.argv[3]
    password = sys.argv[4]
    packages = sys.argv[5].split(",")

Define local variables to hold script arguments.


    # Create Report Instance
    report = Report(config)
    
    # Get ServerProxies
    clients = report.connect()

Create a report instance, passing the report configuration file, then call the connect method to create ServerProxy objects for each Spacewalk server defined in the config.


    systems = []

Define an empty array to hold the system objects that have matching packages installed.


    _count = 0

Define a variable to keep track of the Spacewalk servers processed.


    for client in clients:

Iterate over Spacewalk servers.


        # get the session key
        try:
            key = client.auth.login(username, password)
        except xmlrpclib.Fault, fault:
            print "Error in systemsByPackageReport:"
            print fault.faultCode
            print fault.faultString

Attempt to login with provided credentials and obtain the session key. 


        if not (report.parameters['Aggregate']):
            systems = []
Handle the aggregation parameter as specified in the configuration file.  Essentially this means, create a single report or multiple reports.


            systems = processData(client,key,systems)
Call the processData method -> which returns the data structure that will be passed to the template engine.


            satellites = [report.parameters['Satellites'][_count]]
            vars = {'systems': systems, 'satellites': satellites}
            report.templatify(vars, type)
Pass the datastructure to the template rendering method of the helper class.


        else:
            systems = processData(client,key,systems)
    
        _count = _count + 1
    
    if (report.parameters['Aggregate']):
        vars = {'systems': systems, 'satellites': report.parameters['Satellites']}
        report.templatify(vars, type)
Next, define the processData method.


    def processData(client, key, systems):
    
        try:
    
            user_systems = client.system.listUserSystems(key)
    
            for system in user_systems:
    
                print system
    
                sys_packages = client.system.listPackages(key, system['id'])
    
                    #returns a array of packages installed on the system that are in common with the user provided list
                    common_pkgs = comparePkgs(sys_packages, packages)
    
                    if len(common_packages) > 0:
                    
                        sys = System()
                        sys.id = system['id']
                        sys.name = system['name']
                        sys.packages = common_pkgs
                        systems.append(sys)
    
        except xmlrpclib.Fault, fault:
            print "Error in systemByPackageReport.processData():"
            print fault.faultCode
            print fault.faultString
    
        return systems
    

The comparePkgs() method will compare the installed packages of a given system against the user supplied package list, and return the package details for any matches.

    def comparePkgs(system_packages, packageList)
    
        common_packages = []
    
        for pkg in packageList:
    
            for package in system_packages:
    
                if pkg == package['name']:
    
                    common_packages.append(package)
    
        return common_packages
#### Step 2 - Writing the Template



In this case, we are going to create a .txt template for the report output.

You'll recall from the script we just completed that we are passing to data elements to our template rendering method:

 * systems -> data structure consisting of an array of System objects; which in turn contain and array of matching Package objects.
 * satellites -> data structure consisting of spacewalk servers used to generate this report.
#### SystemsByPackage.txt



[[https://fedorahosted.org/spacewalk/attachment/wiki/PlayPen/Telemetry/SystemsByPackage.txt]]


    #from datetime import datetime
    Systems By Package Report
    Report Date: $datetime.now().strftime("%Y-%m-%d:%H:%M:%S")
    
    Satellites:
    #for satellite in $satellites
                 $satellite
    #end for
    
    #for system in $systems
    System ID:   $system.id
    System Name: $system.name
    ******************************************************************************************
    ******************************************************************************************
    Packages:
    #for package in $system.packages
                 $package.name   $package.version   $package.release   $package.epoch
    #end for
    
    
    #end for
#### Step 3 - Writing the Configuration File


#### systemsByPackage.conf

[[https://fedorahosted.org/spacewalk/attachment/wiki/PlayPen/Telemetry/systemsByPackage.conf]]



    name: Systems By Package Report
    description: This report shows systems with packages installed that match the user supplied csv, along with the specific NVRE.
    script: systemsByPackageReport.py
    
    satellites:
    - http://rlx-1-16.rhndev.redhat.com/rpc/api
    - http://rlx-1-16.rhndev.redhat.com/rpc/api
    
    api_versions:
    - 5.1.0 Java
    - 5.1.1 Java
    
    criteria:
    - name: Packages
      label: packages
      type: csv
      cols: 75
      rows: 10
      required: yes
    
    
    aggregate: yes
    
    templates:
        txt: SystemsByPackage.txt
    
    prefix: SystemsByPackage
### Tasks



1. Webui

 * style/css
 * navigation/flow
 * notification integration (email)
 * improved scheduling options (one-time-only, daily, weekly, etc)
 * crud functions for report config and templates

2. Infrastructure

 * integration with apache/mod-python
 * store reports on web server
 * single rpm installation with dependendencies

3. Other

 * Handle reporting across orgs (multiple users) -> Need a mapping file for user/password to servers.
### Design

#### Web UI Mockups




![Alt](images/telemetry-overview-0.1.png?raw=True)
(source SVG image in list of attachments at the bottom of the page)

This mockup is just a brainstorm to try to work out a general structure/layout, it can completely change. It's more meant to be a discussion starter. Here is what I starting thinking so far by sketching out the mockup:

 * Overview: just little preview widgets to give you a feel for what data lay in the other screens, with links to jump to the full data sets... (now that I'm looking at it, each report could have csv / txt / html / pdf icons next to it to let you download them in one click directly from the overview page)
 * Run a report: a wizard that walks you through running a report; it'll let you choose which report type you'd like to run, enter in your username/pass for the report, and schedule the report or let you run it directly.
 * Create a new report: This is probably the part of the ui that reaches most far and would likely be lower priority / out-of-scope at first. But, I think if possible:
   * it would be nice to have a wizard that asks you all the questions that you get asked in the individual report config files and generates a config file for you.
   * It could have an upload form for the api script. One idea here too could be a text field for the api script, to write it in the page, and it could be pre-filled with some stub code and have some kind of basic API cheatsheet along the bottom with a link to the full API guide.
   * there could be a templates section at the end that shows the various templates associated with the report and lets you at least view them if not modify them.
 * Report library: a directory showing all the various types of reports, ideally categorized (not sure how best to handle the categorization), with maybe a calendar for each report that will let you see which days of which months that report was generated and will let you click to see them, as crude as it is i was thinking something like this (but prettier!): 
http://nicubunulj.livejournal.com/calendar
   * maybe by default the calendar for this current month is shown and you click a link to zoom in on all the monthly calendars for that report type.
 * Report schedule: a schedule-based view of reports, to show you which reports run on which basis (hourly / daily / weekly / monthly) and to let you modify the schedules for those reports or stop them from running
 * Report templates: similar to the report-library view, but per report it'll list out what templates are available (html/ pdf/ csv/ etc) and let you add another template type

In addition to this, reports will probably need two views:

 * Report Details view: report type's name, what satellites the report runs on, (a lot of the .conf file details), a listing of the templates available for the report, and a full calendar for that report (the report schedule page would link specifically to that full calendar section)
 * Specific report run details: the results from a particular run of that report, with date/timestamp, user who ran the report, and the report output

This hopefully can be refined and simplified a lot more. Next we need to draw up a flow diagram for the UI and then based on feedback from the above and that do some better screen mockups. 