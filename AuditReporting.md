# Audit Reporting



With Spacewalk 2.0 we started to log Spacewalk users' events, so we can track what user has done what and at what time.
E.g. who has deleted a system from Spacewalk server. 
## What gets logged



To catch all the events, the logging is implemented on the database level.

Currently the auditing is enabled on tables:
 - rhnUser
 - rhnServer
 - rhnServerGroup

so we can track changes within these tables.
## Usage



Currently the only way, how to check the logged events is the *spacewalk-report* tool. Make sure you install the *spacewalk-reports* package.
### users




    # spacewalk-report audit-users
    user_id,audit_stamp,audit_action,audit_user_id,audit_username,organization_id,username,password
    1,2013-07-19 07:57:31,INSERT,-1,SETUP,1,admin,$1$PRZpragf$qn0ADgKNUA8RUK6jWPttG1
    2,2013-07-19 10:30:42,INSERT,1,admin,1,mike,$1$7PS2LMpf$3za7fb24Ha0rgPeo39jQM/
    2,2013-07-19 10:31:10,DELETE,1,admin,1,mike,$1$7PS2LMpf$3za7fb24Ha0rgPeo39jQM/

The satellite admin with id 1 was created by (virtual) 'SETUP' as the first satellite user.
He then created user with login mike and id 2 and a minute later he deleted him.
### servers




    # spacewalk-report audit-servers
    server_id,audit_stamp,audit_action,audit_user_id,audit_username,organization_id,architecture,os,release,name,description,info,creator_id,creator_login,auto_update,running_kernel,last_boot,provision_state,channels_last_changed,cobbler_id
    1000010000,2013-07-19 07:58:09,INSERT,1,admin,1,x86_64-redhat-linux,fedora-release,19,ibm-ls22-04.rhts.eng.brq.redhat.com,"Initial Registration Parameters:
    OS: fedora-release
    Release: 19
    CPU Arch: x86_64",,1,admin,N,,0,,,
    1000010000,2013-07-19 07:58:09,UPDATE,1,admin,1,x86_64-redhat-linux,fedora-release,19,ibm-ls22-04.rhts.eng.brq.redhat.com,"Initial Registration Parameters:
    OS: fedora-release
    Release: 19
    CPU Arch: x86_64",,1,admin,N,,0,,,
    1000010000,2013-07-19 07:58:38,UPDATE,-2,CLIENT,1,x86_64-redhat-linux,fedora-release,19,ibm-ls22-04.rhts.eng.brq.redhat.com,"Initial Registration Parameters:
    OS: fedora-release
    Release: 19
    CPU Arch: x86_64",,1,admin,N,3.9.9-302.fc19.x86_64,1374229018.255403,,,
    1000010000,2013-07-19 08:00:37,UPDATE,1,admin,1,x86_64-redhat-linux,fedora-release,19,ibm-ls22-04.rhts.eng.brq.redhat.com,"Initial Registration Parameters:
    OS: fedora-release
    Release: 19
    CPU Arch: x86_64",,1,admin,N,3.9.9-302.fc19.x86_64,1374229018,,,
    1000010000,2013-07-19 10:36:11,DELETE,1,admin,1,x86_64-redhat-linux,fedora-release,19,ibm-ls22-04.rhts.eng.brq.redhat.com,"Initial Registration Parameters:
    OS: fedora-release
    Release: 19
    CPU Arch: x86_64",,1,admin,N,3.9.9-302.fc19.x86_64,1374229018,,,

Server with systemid 1000010000 was registered under the admin account (id 1). The first update happens within the registration process.
Then either rhn_check, rhn-profile-sync or any other 'CLIENT' tool was run on the server that caused an update. After the next admin's update the admin user deleted the system completely at 10:36.
### server-groups



Use

    # spacewalk-report audit-server-groups
    group_id,audit_stamp,audit_action,audit_user_id,audit_username,organization_id,group_name,group_description,max_members,current_members
    7,2013-07-22 10:29:23,INSERT,1,admin,1,test-group,test-group,,0
    7,2013-07-22 10:30:01,UPDATE,1,admin,1,test-group,test-group,,1
    7,2013-07-22 10:30:45,UPDATE,1,admin,1,test-group,test-group,,0
    7,2013-07-22 10:30:45,DELETE,1,admin,1,test-group,test-group,,0

Server group test-group was created by user admin, then he added one server into this group. Last two rows of report is deleting server group, last update is removing server from server group as this happens during deletion and the DELETE audit_action is deleting the server group. According to the timestamps of last two actions the removal of the server from the group occurred during deletion of the group.