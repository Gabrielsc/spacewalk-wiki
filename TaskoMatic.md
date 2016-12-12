## What is Taskomatic?

Taskomatic is a job scheduler which closely resembles cron. Taskomatic jobs are the "background" parts of Spacewalk which must be performed on a regular basis.


The Java port of Taskomatic is based on the [Quartz](http://quartz-scheduler.org/) scheduler library. The daemon part of Taskomatic uses the [Tanuki Service Wrapper](http://wrapper.tanukisoftware.org/doc/english/introduction.html) library.
## Tasks, bunches, schedules



Tasks are grouped into bunches (e.g. package-cleanup-bunch contains one task package-cleanup). If you want to run task bunches on regular basis, then you have Task schedule (e.g. package-cleanup-default).

To check whether the engine is running and what is the last time each bunch was ran and its state, check _Admin -> Task Engine Status_ and note "Scheduling Service: ON" and status and execution date of individual bunches.
## Reconfiguring Taskomatic



Changes to the *frequency* of the taskomatic schedules is not recommended but changing the time of day when the daily tasks take place can be done.

Navigate to _Admin -> Task Schedules_. There is a list of defined task schedules with given frequencies as [Quartz strings](http://www.quartz-scheduler.org/documentation/quartz-1.x/tutorials/crontrigger). After navigating to selected schedule, you can edit Quartz string manually or select values from form.
## Tasks




    #!html
    <table cols="3" border="1" cellpadding="2" cellspacing="2">
      <tr>
        <th>Task Name</th>
        <th>DB Name</th>
        <th>Purpose</th>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.DailySummary</td>
        <td>daily-summary</td>
        <td>Sends out email on a daily basis with a summary of out-of-date systems</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.ErrataMailer</td>
        <td>errata-mailer</td>
        <td>Sends email to org admins for each errata alert, eg: "RHN Errata Alert: RHBA-2008:0959-3.."</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.ErrataQueue</td>
        <td>errata-queue</td>
        <td>Processes errata that are synced from satellite-sync and sets them up to be ready for the ErrataMailer to send the email notifications.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.ErrataCacheTask</td>
        <td>errata-cache</td>
        <td>Calculates the 'errata cache' information which is used in the GUI to display what systems need what errata.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.KickstartCleanup</td>
        <td>kickstart-cleanup</td>
        <td>Cleans up old kickstart sessions that have gone stale.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.PackageCleanup</td>
        <td>package-cleanup</td>
        <td>Physically deletes rpms from /var/satellite that are 'marked' for deletion in the GUI.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.ChannelRepodata</td>
        <td>channel-repodata</td>
        <td>Prototype to be used to regenerate channel metadata.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.SatelliteCertificateCheck</td>
        <td>sat-cert-check</td>
        <td>Checks to see if the Satellite Certificate is expired and if so starts sending email with warning.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.SandboxCleanup</td>
        <td>sandbox-cleanup</td>
        <td>Deletes stale Configuration Management system sandboxes.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.SessionCleanup</td>
        <td>session-cleanup</td>
        <td>Deletes expired GUI Http sessions from the pxtsessions database table.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.SummaryPopulation</td>
        <td>summary-population</td>
        <td>Works in tandem with "DailySummary" to determine what orgs should get the daily summary email.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.RepoSyncTask</td>
        <td>repo-sync</td>
        <td>Used for syncing repos (like yum repos) to a channel. Calls a python script.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.SatSyncTask</td>
        <td>satellite-sync</td>
        <td>Calls satellite-sync tool to sync channels.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.KickstartFileSyncTask</td>
        <td>kickstartfile-sync</td>
        <td>Syncs kickstart profiles that were generated using the wizard.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.CobblerSyncTask</td>
        <td>cobbler-sync</td>
        <td>Syncing cobbler data. Communicates with cobbler through XMLRPC.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.CompareConfigFilesTask</td>
        <td>compare-config-files</td>
        <td>Schedules a comparison of config files on all systems.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.ClearLogHistory</td>
        <td>clear-log-history</td>
        <td>Clears taskomatic run log history.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.ChangeLogCleanUp</td>
        <td>cleanup-packagechangelog-data</td>
        <td>Deletes orphaned packages changelog data.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.RebootActionCleanup</td>
        <td>reboot-action-cleanup</td>
        <td>Cleans up stale reboot action data if it is older than 6 hours.</td>
      </tr>
      <tr>
        <td>com.redhat.rhn.taskomatic.task.AutoErrataTask</td>
        <td>auto-errata</td>
        <td>This is what automatically schedules automatic errata update actions.</td>
      </tr>
    </table>
## Troubleshooting Taskomatic



All Taskomatic logging is controlled by the log4j configuration file /usr/share/rhn/classes/log4j.properties. Taskomatic task engine and individual task log output is configured to go to /var/log/rhn/rhn_taskomatic_daemon.log.

Make sure that these lines exist in the log4j.properties file


     log4j.appender.TaskomaticAppender=org.apache.log4j.FileAppender
     log4j.appender.TaskomaticAppender.file=/var/log/rhn/rhn_taskomatic.log
     log4j.appender.TaskomaticAppender.layout=org.apache.log4j.PatternLayout
     log4j.appender.TaskomaticAppender.layout.ConversionPattern=[%d] %-5p %c - %m%n

To enable Taskomatic task engine logging add or edit the following line in log4j.properties:

     log4j.logger.com.redhat.rhn.taskomatic=DEBUG,TaskomaticAppender
To enable coarse-grained task logging add or edit the following line in log4j.properties:

     log4j.logger.com.redhat.rhn.task=DEBUG,TaskomaticAppender
To enable task-specific execution logging add or edit the following line, as needed, in log4j.properties:

     log4j.logger.<insert_task_name_here>=DEBUG,TaskomaticAppender
The Taskomatic daemon process can be started in console mode as well. This is helpful in diagnosing problems where Taskomatic itself will not run:

      sudo /sbin/service taskomatic stop
      sudo /sbin/service taskomatic console

See also: [LogDriverSetup](LogDriverSetup)

