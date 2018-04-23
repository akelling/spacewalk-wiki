# Downloads


## Binary



Releases: The binaries for Fedora and RHEL and derivates can be downloaded from the [copr repositories](https://copr.fedorainfracloud.org/coprs/g/spacewalkproject/spacewalk-2.8/). Please [follow installation information](HowToInstall) on how to best setup yum to have dependencies resolved properly.

Nightly Builds: Composes of packages built for the upcoming release are generated a couple of times per day and available as [nightly builds](HowToInstallNightly).
## Source

### Getting source code with git




Spacewalk uses [git](http://git.or.cz) for source control.

To checkout the latest version from git:


    git clone git@github.com:spacewalkproject/spacewalk.git/

If you have a firewall that is blocking the git clone operation, you can also clone over http, although it is slower:


    git clone https://github.com/spacewalkproject/spacewalk.git/

You can view the source via [github](https://github.com/spacewalkproject/spacewalk)
### Commiter Git Information



In general, we want folks to send [github pull requests](PatchProcess).  If you have commit access to Spacewalk, however, you can use either of the above URLs to push your changes. The https URL forces you to type your username / password each time, whereas if you use the git URL you can upload your ssh keys to github and not have to authenticate each time.
### Source RPMs



Source packages are available from [[http://yum.spacewalkproject.org/]]version/OS/release/source directories.
## RPM Package Signing



We use a number of GNU Privacy Guard (GPG) keys to sign our software packages. The necessary public keys are included in relevant products and are used automatically to verify software updates. You can also check the packages manually using the keys on this page.

To verify a RPM package for Spacewalk, run the command

    rpm --checksig -v <filename>.rpm

The output of this command will show you if the package is signed, and which key was used to sign it.

The public key is here: [[https://copr-be.cloud.fedoraproject.org/results/@spacewalkproject/spacewalk-2.8/pubkey.gpg]]