# Bug Handling in Spacewalk



If you want to commit fix for a bugzilla opened against Spacewalk, either because you came up with the fix or because you want to commit a patch from pull request created by community member, we'd like you to:

 * Fix in Spacewalk git repo in the *master* branch:
   * Use bugzilla number in the commit message of your fix.
   * The format of the commit message is '<bznumber> - <explanation of the fix>'.
   * You can provide more detailed description of why you are doing it and why you are doing it the way you do in the commit message, after an empty line.

 * Align bug to the next upcoming Spacewalk release if it's not already.
   * Tracker bugs are used for that in the form of spaceXY, so for example [space23](https://bugzilla.redhat.com/showdependencytree.cgi?id=space23) for the upcoming Spacewalk 2.3.

 * About a week or two before the next release, we create branch for the release. If you want to get the fix to this branch of soon-to-be-released or already released release, make sure the release nanny knows about your intention or that you will be able to tag and build the package and get it into the yum repo, to avoid disappointment.
   * If you want to go ahead with getting the fix there, cherry pick from master to that branch using *git cherry-pick -x <SHA1 of the commit in master>*. The -x option puts in reference to the original commit to the commit message in branch.

 * Put the SHA1 of the commit or commits you've done to a comment in the bugzilla, and change the bugzilla state to *MODIFIED*.
   * If you were cherry-picking to another side branch, put both the SHA1 of the master commit and of the branch commit (cherry-pick) to the comment.

 * If you have finished working with a set of bugzillas or with any give package or area of code, tag the package and optionally build it, using the tito command: https://fedorahosted.org/spacewalk/wiki/ReleaseProcess.

 * If you see packages built in copr or if they show up in the nigthly repo after an automatic build, fill in the *Fixed In Version* with package(s) name and version-release where the fix is included.

 * Move bugzilla to *ON_QA* when asked by the release nanny to do so, or when there is Spacewalk instance available for testing.

 * Move bugzilla to *VERIFIED* during Spacewalk developer QA cycles prior to release of Spacewalk release. For bugzillas reported by community members, the reporters are welcome to move their bugzillas to *VERIFIED* if they've verified the bug has been fixed with the new package.

 * Nanny for Spacewalk release moves bug to *CLOSED CURRENTRELEASE* on GA.
## Typical flow for Spacewalk bug states



   1. NEW - Newly filed bugzilla.
   2. ASSIGNED - Bug on developer's plate, the developer has acknowledged the existence of the bugzilla.
   3. MODIFIED - Bug has code checked into git repo, bugzilla has comment pointing to a specific commit SHA1.
   5. ON_QA - There are rpms available in our QA yum repository.
   6. VERIFIED - Testing/confidence complete in our QA yum repository.
   7. RELEASE_PENDING - Not used.
   8. CLOSED - Release nanny closes the bugzilla with the note in which Spacewalk release the bug was addressed.
## Commit Message With Associated Bugzilla Comment

### Commit Message


Within your git commit message please use the following format. 


If you're fixing a bug:

      bznumber - message

If no bug associated then:

      message

Here are some VALID examples:

     481767 - be more forgiving of busted kicstart ...
     Moving SystemDetailsHandler.java and its junit test ...

Some invalid ones:

     199560: Fix epoch being returned ...
     483815 minor messaging change ...

Why is this important you ask? Because when we tag the package using `tito tag`, we get nice changelog prepopulated.

You can try it yourself with:

     $ git log --pretty=oneline . 
    
     SHA1 Added note to make sure devs realize the han ...
     SHA1 484532 - api - fix description on ErrataHand ...
     SHA1 439834: Basically no system groups message w ...
     SHA1 484285 - Moved the SSM channel subscription  ...
### Commit Id (SHA1) into Bugzilla Comment



If you move your bugzilla to MODIFIED, please also include a comment about what commit you think has fix for the problem. It makes much easier for people looking at the bugzilla two years from now to figure our where you think you've fixed the issue.

Make sure to put the SHA1 (the commit id) to the bugzilla only after you're resolved any `git push` issues and the push actually went through. Especially when you rebase, SHA1s of your commits change, so take the commit id only after they are safely in the public repo.
### Fixed In Version



Once the package is tagged and built, put the tag value into the Fixed In Version in bugzilla (and maybe to bugzilla comment as well). Again, it makes it easier for people who don't (want to) know about git to figure our what rpm to try.
