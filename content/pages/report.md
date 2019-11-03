---
title: "Report"
date: 2019-10-06T14:56:20+02:00
---

![Group Photo!](/images/conf/groupphoto.jpg)

# Day 1

## Infrastructure

There was an small talk about the curent infrastructure we provide, the problems we currently have which are:

* User management
* Security
* Lack of overview
* Old unmaintained software in operation (flyspray, bbs, planet, mailman 2)
* Not ideal Ansible setup
* Mix between VPS/dedicated machines
* Unmanaged Luna Server (cgit, AUR, BBS)
* Hard to mirror betwene cgit and Github
* no CI/CD

### Discussions 


#### Mailman 2

Mailman 2 is still Python 2 which is soon EOL. We want to move to Mailman 3.
The new version has a nice (optional) webui where you can search in a mailing
list, reply and there is even an API. The current blocker for upgrading to
mailman 3 is that bounce handling is not implemented for emails sent from
mailman meaning other mailservers could blacklist us as a consequence of not
[implementing bounce handling](https://gitlab.com/mailman/mailman/issues/343).

Migrating to mailman 3 is documented
[here](https://docs.mailman3.org/en/latest/migration.html).

The actionable items where identified as:

* Package mailman 3 and [hyperkitty](https://gitlab.com/mailman/mailman/issues/343) (optionally)
* Try the migration process, notes from [Fedora](https://fedoraproject.org/wiki/Mailman3_Migration)
* Bug upstream about the status of the bounce handling implementation

#### User Management

Explored managing the users using Ansible to manage LDAP. Flaws with LDAP:
firstname/lastname should be set which some software does not handle, slow for
searching.  KDE is moving to openid-connect. Start with gitlab, and extend from
there.

Services which we need to move over:

* Grafana
* BBS
* Zabbix
* Mediawiki
* AUR
* Patchwork (gitlab?)
* Archweb (django has an module for it)
* [Mailman](http://docs.mailman3.org/en/latest/config*web.html#configure-login-to-django)
* Kanboard (gitlab?)
* Gitlab
* Matrix
* Quassel
* Email (dovecot) => unix user
* SSH/unix users

#### Questions
* To we put everyone in openid connect or not?
* How to do this gradually?
* How well does it scale?

#### Next Steps

* Package freeipa
* Package keycloak
* Getting Arch Users in Keycloak and decide on the attributes
* Make Gitlab use keycloak using the Arch Staff (Dev/TU) group.

#### Gitlab

The plan is to try out Gitlab for hosting our project git repositories and not the packages!

#### BBS Alternative

The current forum software uses Fluxbb which has it's last release more then a
year ago and might cause issues in the future regarding security, performance
and PHP compatibility. We have custom patches applied to our current fluxbb
instance so that searching does not DoS the MySQL database.

The main requirements for an alternative forum software have been identified as:

* Retain old posts from FluxBB
* GDPR (or make it easier then the current scenario)
* OAUTH or Plugin software
* Spam / signup protection such as we have now
* Moderation tools, mass removing posts. 
* Theme-ability
* Performance

Possible alternatives have been identified as:

* [dfeed](https://github.com/CyberShadow/DFeed)
* [discourse](https://www.discourse.org/)
* [flarium]( https://flarum.org)

For discourse there is a way to [migrate FluxBB](https://meta.discourse.org/t/migrating-to-discourse-from-another-forum-software/16616) users and posts to Discourse using a [Ruby import script](https://github.com/discourse/discourse/blob/master/script/import_scripts/fluxbb.r). This has been tested and has so far been able to import the users.

Testing migration to discourse:

```
mysqldump fluxbb fluxbb_users fluxbb_categories fluxbb_forums fluxbb_bans fluxbb_groups fluxbb_posts fluxbb_topics > fluxbb.sql
mysql -u root fluxbb < fluxbb.sql
git clone https://github.com/discourse/discourse.git
# Setup the docker container for discord or use a package
export RAILS_ENV=production
FLUXBB_HOST=127.0.0.1 FLUXBB_USER=fluxbb FLUXBB_DB=fluxbb FLUXBB_PW=fluxbb FLUXBB_PREFIX=fluxbb_ bundle exec ruby script/import_scripts/fluxbb.rb
```

The importer supports restarting on a previous half finished import.

#### Automate DNS

Currently our DNS is manually managed and provided by Hetzner. Since we use
terraform now it would be logical manage our DNS also with terraform.

[notes](https://pad.sleepmap.de/p/2019-arch-conf-infra)

## Package standards and Quality

Arch has packaging guidelines of varying quality and some of them are often
ignored. Below is a collection of findings from a discussion with various
developers and trusted users.
These data points need to be incorporated into the [package
guidelines](https://wiki.archlinux.org/index.php/Arch_package_guidelines) and
the [packager
howto](https://wiki.archlinux.org/index.php/DeveloperWiki:HOWTO_Be_A_Packager).

To increase the knowledge sharing on best practices it would be benificial to
add experienced maintainers for the various languages (e.g. Haskell, Go, Java,
Ruby, etc.) as contacts to the Wiki so people can have a look at "model"
PKGBUILDs instead of random ones in the repos.

Additionally, issues that come up during TU application reviews are a good
example of inconsistencies, that are worth documenting right away.

The following list is a (non-exhaustive) collection of examples, that still
require addition in the wiki.

- use HTTPS for sources whereever possible, "git+https://" for git
- don't use specific sourceforge mirror to download, as they might not be
  available (see [this forum
  entry](https://bbs.archlinux.org/viewtopic.php?id=22200) as an example)
- use PGP signatures where possible (may need to build from git tag instead of
  source tarball)
- rename sources to something unique for shared `$srcdir`
- be consistent with the use of cd "$srcdir/..." (which is not needed anyway)
  or use subshells `( )`
- don't use internal functions like `msg2` as they are subject to change and
  *will break*
- `autoreconf`, `autogen.sh`, `bootstrap`, etc. needs to be called in
  `prepare()`, not in  `build()`
- always do `autoreconf -vfi` (instead of e.g. `autogen.sh` or directly using a
  configure file), so that system flags (e.g. `--as-needed` in `LDFLAGS`) are
  honored
- avoid `sed` as it may fail silently
- report problems upstream right away and add a link to the upstream ticket in
  a comment. this improves visibility and allows for others to work with the
  package
- upstream patches where possible
- run tests whereever possible
- don't diminish the security or validity of a package (e.g. no checksum check,
  removing PGP signature verification), because an upstream release is broken
  or suddenly lacks a certain feature (e.g. PGP signature missing)
- don't build implicitly in `package()`
- use `/usr` instead of `/etc` where possible, e.g. `/usr/lib/systemd/system`
- use .install files only for warnings, not for documentation
- no need for provides=("$pkgname")
- add non-internal shared libraries to `provides` (e.g. `package` provides
  `libpackagename.so` and `libsomeotherthing.so`), so they can be used by other
  packages directly (document `devtools`' `find-libprovides` and `find-libdeps`
  for this use-case)
- use `sysusers.d` and `tmpfiles.d`, instead of adding users or chowning files
  manually in .install and *never* delete users on uninstall (see this
  [TODO](https://www.archlinux.org/todo/usergroup-management/) for an example)

Specific guidelines that should be improved by the currently most experienced
developers/ Trusted Users on the given language:
- [Haskell](https://wiki.archlinux.org/index.php/Haskell_package_guidelines)
  (completely empty)
- [Java](https://wiki.archlinux.org/index.php/Java_package_guidelines)
  (suggests copying binaries instead of building from source is ok)

## Transitive dependencies

Transitive dependencies are package dependencies, that abstract direct
dependencies. If `package_a` depends on `package_b` and `package_c`, but
`package_b` also depends on `package_c`, then making `package_a` depend only on
`package_b` introduces a transitive dependency.
Questions around this topic pop up on the mailing lists from time to time (e.g.
[here](https://lists.archlinux.org/pipermail/arch-projects/2017-August/004607.html)).

In a discussion round we identified a few problems around this topic and worked
out actionables to discourage the use of transitive dependencies.

### Problems

* packages break when a dependency they need is removed down the chain where
  they used to get it from
* Namcap encourages it (e.g. it will suggest to remove “already included”
  dependencies): https://bugs.archlinux.org/task/64022
* Raises issues like https://bugs.archlinux.org/task/54887
* Hard to grasp which packages actually depend on something just by looking at
  reverse dependencies

### Solutions

* namcap needs to be changed to output an error on all missing `NEEDED` library
  dependencies (see `find-libdeps` and `find-libprovides`)
* namcap needs to be extended to properly resolve dependency chains for python
  software (e.g. by utilizing setuptools configuration of a given project)
* use of `find-libdeps` and `find-libprovides` needs to be documented in the
  wiki and its use encouraged
* packaging guidelines need to be changed to make sure packagers add shared
  library names to provides array, which will extend the granularity of
  dependency resolution and visibility

## Versioned Kernel Installs

[notes](https://pad.sleepmap.de/p/2019-arch-conf-versioned-kernel-installs)

# Day 2

## svn2git migration

[notes](https://pad.sleepmap.de/p/2019-arch-conf-svn2git-migration)

## GSoC

Multiple members of the community have asked about doing a Google Summer of
Code project for Arch Linux. When this was brought up, we where most of the
time unable to comprehend a list of potential thought out projects users could
take on. We've held a small brainstorm session on potential GSoC projects:

* Pacman - [parallel transfers](https://bugs.archlinux.org/task/20056)
* Pacman - [iterator interface for reading databases](https://lists.archlinux.org/pipermail/pacman-dev/2011-July/013816.html)
* Archweb - Implement an official pkgstats
* Infra - Arch Integration Testing Framework for packages
* Infra - Rebuild order program, a program which determines the packages to rebuild when package X is updated
* Namcap - Go fmt for PKGBUILD
* Nmacap - Better tests?
* Reproducible Builds
* Installer - graphical installer
* AUR - Rewrite in Python, since parts of it is already Python
* AUR - Support Pull Requests for an AUR package
* AUR - Support OAUTH/SSO

All of these projects require a dedicated mentor who is wiling to review code,
provide feedback and help out a student.

[notes](https://pad.sleepmap.de/p/2019-arch-conf-gsoc)

## Project governance

[notes](https://pad.sleepmap.de/p/2019-arch-conf-project-governance)

## Base Package

[notes](https://pad.sleepmap.de/p/2019-arch-conf-base-package)

## Golang packaging

[notes](https://pad.sleepmap.de/p/2019-arch-conf-golang)

## Arch Security


### Reproducible builds

For Reproducible Builds the plan is to first reproduce the [core] repository
after the new pacman release (current version v5.1.3) which resolves the
filesystem issue. Then work on creating a rebuilder which is query'able and the
results are shown in the package view in archweb. After the creation of an API,
development can start on userland tools which shows the status of
reproducible packages on your system.

### Arch Security

Progress has been made on enabling PIE for Haskell packages by passing
`--ghc-option='-pie'` to runhaskell Setup configure.

It has been decided to integrate security scanning of packages in Archweb,
which will have a dashboard with packages lacking PIE, full RELRO and other
security mitigation techniques.
