## Release Cycles



Each Spacewalk release will be roughly two (2) months in duration. See [Roadmap](https://hosted.fedoraproject.org/spacewalk/roadmap) for upcoming releases.
## Mechanics

We build Spacewalk rpms using our instance of Koji builder.


The Koji server lives at [koji.spacewalkproject.org](http://koji.spacewalkproject.org/koji).
To submit a build you need a user certificate which is granted on a invite-only basis.

The Spacewalk koji config file in ~/.koji/spacewalk-config could then look like this:


    [koji]
    
    ;configuration for koji cli tool
    
    ;url of XMLRPC server
    server = http://koji.spacewalkproject.org/kojihub
    
    ;url of web interface
    weburl = http://koji.spacewalkproject.org/koji
    
    ;url of package download site
    pkgurl = http://koji.spacewalkproject.org/packages
    
    ;path to the koji top directory
    ;topdir = /mnt/koji
    
    ;configuration for SSL athentication
    
    ;client certificate
    cert = ~/.spacewalk.cert
    
    ;certificate of the CA that issued the client certificate
    ca = ~/.spacewalk-ca.cert
    
    ;certificate of the CA that issued the HTTP server certificate
    serverca = ~/.spacewalk-ca.cert
the ~/.spacewalk-ca.cert file can be found linked at the bottom of this page
## Get your packages tagged, get your packages built



The *tito* tool (available in Fedora) can be used to tag and build packages
from git.

To have a particular package in a Spacewalk release, you will need to tag
it in git. But stop, wait! Don't do `git tag`. Since we have many packages
in our Spacewalk repo, some non-standard housekeeping is needed when tagging,
and we use the *tito* for that.

But before you tag with *tito*, you should really check that what you
have is sane. Change to the directory of the package and run

            tito build --srpm --test

This will create a .src.rpm package from your local HEAD on
whatever branch you are. It will have that commit's SHA1 in name
and if will also appear in the .tar.gz within that .src.rpm. It's
done that way on purpose, so that .src.rpm from your local HEAD is
not confused with "official" package releases.

When you get .src.rpm, try to rpmbuild it on your Fedora and RHEL. Fix
any problems that appear, do more commits, revert commits, etc. To build
the rpm on your local machine, you can also do

            tito build --rpm --test
which will do it in one step. Or you can submit a scratch build to koji.

When you are happy with the .src.rpm, you should run (still in the directory
of the package you are tagging)

            git pull --rebase
            tito tag 

and tag your HEAD with name-version-release, and push that commit and
tag to the upstream repository. You will also have a chance to finetune
the rpm changelog.

After that, run

            gitk --all
            # review and verify that things look sane and that you did not
            # introduce unnecessary merges
            git push && git push --tags

*IMPORTANT*: Do not run rebase after tagging a package. If someone
pushed to master before you managed to push out your tag and your push failed,
you must revert your commit and delete your tag by running

            tito tag -u

then rebase on that remote head by

            git pull --rebase

and then

            tito tag
again.

If you do a rebase after you've tagged, your commit's SHA1 will change
(as it will be a brand new commit on top of the latest remote head) while your
tag will point to a commit which is not accessible from any branch. If you then
push, people will be confused because they will see a commit without tag or
misplaced tag ... bad things. Just `tito tag -u` and `git pull --rebase`.

After you have pushed the tag to the upstream repository, you
can do

    	tito build --srpm
which will create .src.rpm from the latest tag for that package, named
with the package-version-release of that latest tag.

To submit a build to whatever build system is configured for your current
git repository and branch, do:

    	tito release --all

In Spacewalk git branches, this will create .src.rpms with appropriate dist
tags and submit them to Spacewalk koji to Koji tags as configured in the
global `rel-eng/tito.props` config file.

The *tito* will invoke the *koji build* with `--nowait` by default.
If you need something more, like point koji to your config file, add it to your
`~/.spacewalk-build-rc` file:

    KOJI_OPTIONS=-c ~/.koji/spacewalk-config build --nowait

We do nightly builds (in facts 3 times per day) for every tagged package. So unless you need that rpm, then taggin may be sufficient for you.
## Branching a release

When we're nearing the end of a release cycle, we need to branch the release.


Make sure your `master` is clean and updated:

       git checkout master
       git pull (make sure things are clean)

Now create a local branch, where x.y is the release number, i.e. 0.2:

       git checkout -b spacewalk-x.y

Then, push your branch to the remote server, where x.y is the release number:

       git push origin spacewalk-x.y:refs/heads/SPACEWALK-x.y

Whenever doing any change and fixes in that branch, make sure they do not get
lost -- commit to master first and then `git cherry-pick -x` to that "release"
branch.