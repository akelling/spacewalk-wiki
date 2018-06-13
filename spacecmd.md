# Description



spacecmd is a command-line interface to Satellite and Spacewalk servers.
# Downloads

Stable RPMs are available in the [Spacewalk server latest stable repositories](https://copr.fedorainfracloud.org/coprs/g/spacewalkproject/spacewalk-2.8/)

RPMs built nightly from git are available in the [Spacewalk server nightly repositories](https://copr.fedorainfracloud.org/coprs/g/spacewalkproject/nightly/)

The source is available in the [Spacewalk git repo at /spacecmd](http://git.fedorahosted.org/git/?p=spacewalk.git;a=tree;f=spacecmd).
# Install

To install spacecmd:
1. Install Spacewalk client repo. Example(EL 7):

        rpm -Uvh https://copr-be.cloud.fedoraproject.org/results/@spacewalkproject/spacewalk-2.8/epel-7-x86_64/00734156-spacecmd/spacecmd-2.8.25-1.el7.centos.noarch.rpm

2. Install spacecmd

        yum -y install spacecmd


# Bugs

Bugs should be reported to [Red Hat's Bugzilla](https://bugzilla.redhat.com) under the "spacecmd" component.

# Support

 * https://www.redhat.com/mailman/listinfo/spacewalk-list

 * https://www.redhat.com/mailman/listinfo/rhn-satellite-users
 * I've you are interested in development of spacecmd https://www.redhat.com/mailman/listinfo/spacewalk-devel
# Highlights

 * view and manage nearly every aspect of Spacewalk

 * tab completion
 * can be passed single commands (useful for shell scripts)
 * can be used as an interactive shell
 * advanced searching of systems allows systems to be managed without having to create system groups
 * all functionality implemented via the Spacewalk API

----
# Configure



Setting up spacecmd.
## Credential File



Spacecmd can be configured with a credentials file so you are not prompted for a username/password each time.
This allows for easier scripting.

1. Create a hidden spacecmd directory in your home. Lock down permissions.

        mkdir ~/.spacecmd
        chmod 700 ~/.spacecmd

2. Create a config file in the directory and give proper permissions.

        touch ~/.spacecmd/config

        chmod 600 ~/.spacecmd/config

3. Edit the config file and fill in the header, Spacewalk server fqdn, username, and password.

        vim ~/.spacecmd/config
    
>     [spacecmd]
>     server=my-spacewalk-server.local
>     username=usernamehere
>     password=passwordhere

4. Verify connectivity without a username/password prompt.

        spacecmd
## Alias



A useful alias to create is one that will make spacecmd quiet. Quiet mode will NOT print the connecting to server status messages, which get in the way when parsing system lists.

Add the following to your ~/.bashrc

    alias spacecmd='spacecmd -q'

----
# Spacecmd Modes



Spacecmd has two modes; interactive and single command.

  * Enter interactive mode

        spacecmd

  * Execute a single command

        spacecmd <COMMAND>

## Getting Help



To view command help/hints:

  * While already in Interactive Mode

        help <topic>

  * Single Command Mode

        spacecmd help <topic>

----
# Spacecmd Example Commands



Some examples of using spacecmd.

----
## Cache



A local cache is created to improve performance. This can leave you with outdated query information at some point.

To clear the local spacecmd cache:

    spacecmd clear_caches

----
## Group Commands

List system groups

    spacecmd group_list

List systems in a group

    spacecmd group_listsystems <GROUP>

Example Loop; Update systems in the "prod" group

    for NODE in $(spacecmd group_listsystems prod); do echo "=>${NODE}"; ssh -qt ${NODE} "sudo yum -y update"; done

Add systems to a group (system names space separated)

    spacecmd group_addsystems <GROUP> <SYSTEMS>

Remove systems from a group

    spacecmd group_removesystems <GROUP> <SYSTEMS>

----
## System Commands



List all systems

    spacecmd system_list

View system details

    spacecmd> system_details www01.example.com
    Name:          www01.example.com
    System ID:     1000010001
    Locked:        False
    Registered:    20100311 19:31:36
    Last Checkin:  20100621 18:31:53
    OSA Status:    online
    
    Hostname:      www01.example.com
    IP Address:    192.168.1.80
    Kernel:        2.6.18-164.el5
    
    Software Channels:
      custom-rhel-i386-server-5
        |-- custom-extras-i386-rhel5
        |-- clone-rhn-tools-rhel-i386-server-5
    
    Configuration Channels:
      sudoers
      base
      base-rhel5
    
    Entitlements:
      Management
      Monitoring
      Provisioning
    
    System Groups:
      all_linux_systems
      all_linux_VMs
      rhel5-i386

Delete system from Spacewalk

    spacecmd system_delete <SYSTEMS>

List a system's base channel

    spacecmd system_listbasechannel <SYSTEM>

List a system's child channels 

    spacecmd system_listchildchannels <SYSTEM>

Set an entire group's base channel 

    spacecmd -y system_setbasechannel group:<GROUP-NAME> <BASE-CHANNEL-NAME>

List a system's installed packages, grep for a specific one

    spacecmd system_listinstalledpackages <SYSTEM>  | grep glibc

----
## Install Packages and Errata

Add and remove packages from the commandline

    [user@sat]$ spacecmd -y system_installpackage www* mod_python
    Scheduled 6 system(s)
    
    [user@sat]$ spacecmd -y system_removepackage wiki02 mod_perl
    Scheduled 1 system(s)

Apply errata from the command line

    [user@sat]$ spacecmd -y errata_apply RHSA-2010:0423
    Scheduled 42 system(s)
    
    [user@sat]$ spacecmd -y system_applyerrata group:web_servers RHSA-2010:0040
    Scheduled 16 system(s)

----
## Channel Freezing



Some organizations have the requirement that the same updates that get applied in Development, must go through a Test and then Production environment.

In order to accomplish this, software channels can be cloned, essentially "freezing" them and preventing the cloned channel from receiving any new packages.

Clone an entire software channel tree:

    spacecmd softwarechannel_clonetree <base-channel-id> --prefix "prefix-name-here" --gpg-copy

Example: Clone the CentOS 6 tree "centos6_x86-64_base" and give it a date it was cloned prefix. This will copy gpg data and errata data.

    spacecmd softwarechannel_clonetree centos6_x86-64_base --prefix "20160111_" --gpg-copy
  * *Note:* This copies metadata of the channel and does NOT duplicate repo packages; so this does not take up much disk space.

Systems then can have their subscribed software channels changed to this "frozen channel" until the update cycle through production is complete. Once updated through production, subscribe systems back to the original channel or the next frozen channel. The old cloned channel can then be removed once no more systems are subscribed.

----
## Reporting



Systems not in any group

    spacecmd report_ungroupedsystems

Systems not checking in and their last check in time

    spacecmd report_inactivesystems

Entitlements

    [root@sat]# spacecmd report_entitlements | grep -e monitoring -e provisioning
    monitoring_entitled: 314/397
    provisioning_entitled: 322/397

Errata Reports

    spacecmd> system_listerrata ldap03
    System: ldap03
    
    Security Errata:
    RHSA-2010:0458  Moderate: perl security update                        6/7/10
    RHSA-2010:0449  Moderate: rhn-client-tools security update            6/1/10
    RHSA-2010:0423  Important: krb5 security update                      5/18/10
    
    spacecmd> report_errata
    # Systems       Errata
    ---------       ------
    CLA-2010:0474       88
    CLA-2010:0475        6
    CLA-2010:0488      183
    CLA-2010:0490      273
    CLA-2010:0500        4
    CLA-2010:0501        5
    RHBA-2010:0402       1
    RHSA-2010:0474       2
    RHSA-2010:0488       1
    RHSA-2010:0490       5

Out of Date Systems

    spacecmd> report_outofdatesystems
    System        Packages
    ------        --------
    monkey             310
    shark               63
    hedgehog            39
    pomeranian           4

IP Addresses

    spacecmd> report_ipaddresses
    System   Hostname                IP
    ------   --------                --
    dns01    dns01.dmz.example.com   192.168.254.53
    www01    www01.dmz.example.com   192.168.254.80
    ztest    ztest.test.example.com  192.168.42.111

Kernel Versions

    spacecmd> report_kernels
    System       Kernel
    ------       ------
    system01     2.6.9-89.0.25.ELsmp
    system02     2.6.9-89.0.3.ELsmp
    system03     2.6.9-89.0.26.ELsmp

----
## System Set Manager



Working with multiple systems via system set manager on the cli.

Make temporary groups on-the-fly

    spacecmd> ssm_add search:driver:bnx2
    Systems Selected: 111
    
    spacecmd> ssm_add search:device:vmware
    Systems Selected: 285
    
    spacecmd> ssm_add search:hostname:external.example.com
    Systems Selected: 16

----
 
