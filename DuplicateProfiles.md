
# **DEPRECATED, NO LONGER USED**

## Issue

Customers during a reprovision will encounter a duplicate system profile getting created in their spacewalk instance eating up a management entitlement + leaving a dead system. This can become very unmanageable for example when several machines are reprovisioned through PXE or standard boot causing several duplicate system profiles.

## Implementation Summary



 * Added UI detection of duplicates by:
  * Mac address, IP address, Hostname

    If these match for multiple profiles, they're probably duplicates
 * Duplicate prevention during day-to-day workflow
  * Kickstarting through Spacewalk/Satellite's UI automatically reuses existing profiles if applicable
  * If detected that system is already registered (has a systemid) the server will issue a reactivation key automatically
  * System details tab has a new *_Spacewalk Profile*_ option
   * Reconnected to existing profile (the default behavioral option)
   * Delete existing profile (i.e., replace it)
   * Leave old profile in place and make a duplicate (i.e., the old behavior)
## Implementation Notes

 * Have the kickstart script send the system id in a pre script and store the returned reactivation key in a file and during a post script send that reactivation key along with rhnreg_ks. This will work for bare metal provisioning cases.

 * Have a UI + API script that allows a system administrator to show dupe systems based on a hardware criteria and allow her to compare and prune the list. 
 * Add a background task to do the deletions through the SSM asynchronous message queue (Tomcat stuff). Since system deletion is an expensive operation (expensive enough for 200 system deletion/unentitle to run into http time out), we would need to put it on the background. We need this because customers are likely to delete/unentitle a lot of dupes/dead systems at once.
 * (Low Priority) Have rhnreg_ks send hardware information/system id file on registration so that server automatically assigns the sent system id to server instead of saying "You system is already registered use --force."
### Kickstart File Changes

Four snippets will be automatically (non-optional) included in /var/lib/rhn/kickstarts/snippets to perform different actions. They 'll be appended to every spacewalk generated kickstart. The snippets perform actions such as,

   1. Copying /etc/sysconfig/rhn/systemid to /tmp/rhn  (Pre)
   1. Generating the reactivation key storing on to /tmp/key OR calling the 'deleteSystem' command. (Pre)
   1. Copying /tmp/key to /mnt/sysimage/tmp/key (Moving from Non Chroot since the registration snippet needs things to be chrooted) (Post --nochroot)
   1. Modified registration script to check if the key exists and registration using reactivation key (Post)

   The  customer will be able to modify the snippets if they want to turn off this behaviour due to bug or an edge case (which should in theory never happen).
### UI Changes

The initial UI mockups are located at

 * http://duffy.fedorapeople.org/spacewalk/dupe%20profiles/20Apr2010-Spacewalk-DuplicateSystemsMockup-List-2.png -> This page will show a grouping of systems which Spacewalk has identified as duplicates based on Same Hostname/ip address/mac address. The customer will be able to select systems for comparison and make a decision on the bad ones.
 * http://duffy.fedorapeople.org/spacewalk/dupe%20profiles/20Apr2010-Spacewalk-DuplicateSystemsMockup-Comparison-2.png -> This page shows a comparison of a group of systems which were identified as duplicates and lets the users decide which one to delete  
 * Add a check box  option in Systems-> Kickstart -> Select Profile-> System Details page. "[  ] Remove the system history of any systems using this kickstart file. All existing subscriptions, entitlements, and other system profile data will be deleted." This option will dictate if the profile adds a "deleteSystem" or "obtainReactivationKey"  in the pre script.
 * Redo the SSM systems deletion page to java -> http://<FQDN>/network/systems/ssm/misc/delete_systems_conf.pxt and make "deleteSystems" an asynchronous queued background task.
 * Make system header jspf check to see if the system is currently in the deletion queue and if so say "System has been deleted."
### Impact to upgrades of Spacewalk

Customers upgrading from previous versions of Spacewalk will have to regenerate the kickstart files. This may be as easy as deleting files in  /var/lib/rhn/kickstarts/wizard




|  Tasks  |  Days |  |
| --- | --- | --- |
|  Java Api handler call to obtain a reactivation key to given a system id file + unit test  |  1.5  |
|  Java Api handler call to delete a system given a system id file + unit test  |  1.5  |
|  ks file changes   |  2  |
|  Java Side changes to update the kickstart file to automatically append pre/post scripts  |  2   |
|  Api handler to return a list of duplicate systems based on ip, hostname, system id |  2  |
|  UI Changes to search on duplicates  |  4 |  |
|  UI Changes to compare on duplicates  |  4 |  |
|  Manager layer code to 'delete' systems asynchronously  | 3 |  |
|  UI Changes setup the background SSM asynchronous system delete task  |  3 (since its in perl and has to be converted to java) |
|  Changes needed on the system header to redirect one to say "system has been deleted"  | 1 |  |
