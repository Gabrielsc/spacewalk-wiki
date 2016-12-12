# Spacewalk with PostgreSQL backend



The PostgreSQL database support now covers all areas of the Spacewalk project, including monitoring.

Historically, RHN Satellite and thus Spacewalk servers required Oracle RDBMS as database backend. In order to make the whole Spacewalk project use open source software, support for PostgreSQL as database backend has been added.

This is a page describing the status of the PostgreSQL support as of 7th March, 2012, Spacewalk version 1.7.
## Get the PostgreSQL server running



Follow [[PostgreSQLServerSetup]].
## Install the spacewalk-postgresql and configure it



When installing Spacewalk, you install `spacewalk-postgresql` which should give you correct backend and dependencies.

    # yum install spacewalk-postgresql

Then, when you run `spacewalk-setup`, you'll be asked for connection information:


    # spacewalk-setup --disconnected
    ** Database: Setting up database connection for PostgreSQL backend.
    Hostname (leave empty for local)? 
    Database? spaceschema
    Username? spaceuser
    Password? spacepw
    ** Database: Populating database.

The `spacewalk-setup` finishes and you are good to go -- enjoy your Spacewalk on PostgreSQL!
## Migrating existing Spacewalk on Oracle



If you have an existing Spacewalk with Oracle backend installation, you can use it as a base of your Spacewalk on PostgreSQL setup -- see [[DatabaseMigrations]].
## When you hit error



If you want to do a bit of coding, send a patch to the spacewalk-devel@redhat.com mailing list. See below for porting hints.

You can also just report the problem on either spacewalk-devel@redhat.com mailing list or (the more user-oriented) spacewalk-list@redhat.com. Please make sure you have the relevant log parts ready.

You can also file bugzilla. 
## Info for developers

### Porting to PostgreSQL




At [[PostgreSQLPortingGuide]] we list issues that we were addressing while making Spacewalk work with the PostgreSQL database backend. Please use it as a reference of fixes when you hit an issue and want to create a patch for it.
### Database schema



When Spacewalk is installed and `spacewalk-setup` run for the first time, database is populated with necessary database objects via `psql` from `/etc/sysconfig/rhn/postgres/main.sql`. So this is the file you want to edit if you need to fix something in the schema on your installation. Then rerun `spacewalk-setup`, it will prompt to clear the schema. Watch `/var/log/rhn/populate_db.log` for any errors during population of the schema in the PostgreSQL database.

Of course, to preserve the changes, you need to get them to the Spacewalk git repo, so that they get to the next nightly spacewalk-schema package. For PostgreSQL, the schema is compiled from files in the `schema/spacewalk/common/` and `schema/spacewalk/postgres/` directories in the Spacewalk git repo. A file should be either in the `common/` or in `postgres/` subdirectory, not in both.
## Slides for Developer Conference 2011



 * [Spacewalk on PostgreSQL by Jan Pazdziora](http://www.adelton.com/docs/spacewalk/spacewalk-on-postgresql)
   * The history and state of the port, issues that we hit, examples, and hopefully an inspiration for you to contribute or use our experience in migration of your legacy applications.
