# ****DEPRECATED, NO LONGER USED****

For up-to-date information about the PostgreSQL port, see [[PostgreSQL]]. This page is now obsolete.

----

Currently we have no yum repo setup with customized pgsql packages, due to the way we're branched off master this is a tricky task to accomplish and involves significant overhead, for now we'll attempt to get by with the following steps:

 * Install latest spacewalk release. (0.4)
   * NOTE: For now, do not subscribe the system to the spacewalk development repo. (though this may be required in the future)
 * Install and configure postgresql, create a database. (leaving this as an exercise for the user :))
 * Clone the Spacewalk git repository and create a local branch to track the pgsql branch:
    {{{
    git branch --track pgsql origin/pgsql
    git checkout pgsql
    }}}
 * Use deploy script to copy files from git tree out into installed locations on the system:
   {{{
   cd ~/src/spacewalk/
   SWHOST=root@yourspacewalkhost scripts/pgsql-deploy.sh
   }}}
   * Please note this is somewhat dangerous and should never be run on a Spacewalk system you need functioning reliably.
   * Should you decide to go back to stock Spacewalk code, use the commands printed to the console at the end of the deploy script to remove the packages we're affecting and re-install them fresh.
   * Re-run the script often to get the latest changes in pgsql branch. Add new files if you modify something that needs to be deployed.
 * Run spacewalk-setup to connect to PostgreSQL and populate the auto-generated ora2pg schema:
   {{{
(dgoodwin@elaine)[[~_src_spacewalk]] % sudo spacewalk-setup --disconnected
Available database backends:
   oracle
   postgresql
Database? postgresql
** Database: Setting up database connection.
DB User? spacewalk
DB Password? 
DB SID? spacewalk
DB hostname? localhost
DB port [[1521]]? 5432
DB protocol [[TCP]]? 
** Database: Populating database.
*** Installing PostgreSQL schema.
*** Progress: ###############################################################################################################
* Setting up users and groups.
** GPG: Initializing GPG and importing key.
* Performing initial configuration.
Charset = UTF8
* Activating Spacewalk.
** Loading Spacewalk Certificate.
** Verifying certificate locally.
** Activating Spacewalk.
There was a problem validating the satellite certificate: 1

(dgoodwin@elaine)[[~_src_spacewalk]] % 
   }}}
   * Process dies at this point pending completion of the rhnSQL Python driver.
   * Technically your oracle based spacewalk install is still functional.


 
