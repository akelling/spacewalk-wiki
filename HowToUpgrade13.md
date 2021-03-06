# Spacewalk Upgrade Instructions



These are upgrade instruction for upgrading Spacewalk 1.2 to Spacewalk 1.3.

These upgrade instruction apply to Spacewalk installations meeting the following criteria:

  *  Spacewalk 1.2 running on Red Hat Enterprise Linux 5 Server or CentOS 5, and on Fedora 13 and 14.
  *  Your Spacewalk uses either of Oracle 10g (including XE) / Oracle 11g / PostgreSQL 8.4+ as a database backend.
## Archive of older upgrade instructions



 * Spacewalk 1.1 to 1.2 upgrade instructions, are available at [[HowToUpgrade12]]
 * Spacewalk 1.0 to 1.1 upgrade instructions, are available at [[HowToUpgrade11]]



----
## Assumptions



  * For RHEL or CentOS, you will need the Base, EPEL 5 repositories enabled for dependencies.
  * For Fedora, your Fedora yum repositories are setup properly.
  * You had set up your yum to point to Spacewalk 1.3 repository. For the repo setup specifics, see HowToInstall#SettingupSpacewalkrepo.
    * In particular, for Fedora 13 and 14 make sure your jpackage repo is setup properly: https://fedorahosted.org/spacewalk/wiki/HowToInstall#Jpackagerepository
## Database and configuration backup




  *  For existing configuration files, create a backup of everything under /etc/sysconfig/rhn /etc/rhn and /etc/jabberd
  *  Backup your SSL build directory, ordinarily /root/ssl-build
  *  For instructions on how to create a backup of your existing Spacewalk database consult either Oracle / PostgreSQL documentation or contact your DBA
## Oracle Instant Client upgrade (Oracle DB backends only)



If you use PostgreSQL as a database backend, you may safely skip this step.

Spacewalk 1.3 requires Oracle Instant Client 11g. Before commencing with package upgrade, you need to upgrade oracle-instantclient* packages:

  1. Navigate to http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html choose correct architecture and download latest available versions of following packages:
    * oracle-instantclient11.2-basic
    * oracle-instantclient11.2-sqlplus

  2. Stop your Spacewalk and uninstall previous oracle-instantclient packages. Run as root:


    # spacewalk-service stop
    # rpm -e --nodeps oracle-instantclient-basic oracle-instantclient-sqlplus

  3. Install oracle-instantclient11.2* packages you downloaded in step 1. Run as root:


    # rpm -ivh oracle-instantclient11.2-*.rpm
    Preparing...                ########################################### [100%]
       1:oracle-instantclient11.########################################### [ 50%]
       2:oracle-instantclient11.########################################### [100%]
## Package upgrade



Your Spacewalk should be stopped at this point. Perform package upgrade using yum:


    # yum upgrade

During the upgrade, you may notice messages printed to the terminal when installing oracle-instantclient-selinux and spacewalk-selinux. These messages are produced by restorecon and do not pose any harm.

Check any `.rpmnew`/`.rpmsave` files that were created during the upgrade for your configuration files and make sure changes to your configuration files are preserved while new content from the distribution files is carried over.

    # yum install rpmconf
    # rpmconf -a
## Schema upgrade



Make sure your Spacewalk server is down:


    # /usr/sbin/spacewalk-service status

Make sure your database server is running. Run spacewalk-schema-upgrade script to upgrade database schema:


    # /usr/bin/spacewalk-schema-upgrade

Important notes:

  * The above command will inform you whether or not was the schema upgrade successful.
  * Log files from schema upgrade are being put into /var/log/spacewalk/schema-upgrade.
  * Should the schema upgrade fail because of insufficient space in some of the tablespaces, restore the database from backup, extend the tablespaces as needed and execute this step again.
## Upgrade of Spacewalk configuration



  1. Use spacewalk-setup to upgrade Spacewalk configuration. Run as root:


    # spacewalk-setup --disconnected --upgrade

  2. Restore some of the custom values you might have set previously in /etc/rhn/rhn.conf from the backup of your configuration files, such as:

  *  debug = 3
  *  pam_auth_service = rhn-satellite

  3. Enable (or re-enable) monitoring (and monitoring scout).

  * If you wish to enable (or re-enable) monitoring without enabling monitoring scout, run the following command as root:


    # /usr/share/spacewalk/setup/upgrade/rhn-enable-monitoring.pl

  * If you wish to enable (or re-enable) both monitoring and monitoring scout, run the following command:


    # /usr/share/spacewalk/setup/upgrade/rhn-enable-monitoring.pl --enable-scout
## Clean old rhn cache

Run as root:



    # rm -rf /var/cache/rhn/satsync
    # rm -rf /var/cache/rhn/xml-*
## Clean jabberd database



Run as root:


    # rm -f /var/lib/jabberd/db/*
## Restart Spacewalk




    # /usr/sbin/spacewalk-service start