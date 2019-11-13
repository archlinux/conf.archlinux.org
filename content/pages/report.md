---
title: "Report"
date: 2019-10-06T14:56:20+02:00
---

![Group Photo!](/images/conf/groupphoto.jpg)

[Recordings](https://static.conf.archlinux.org/archconf2019/recordings/)

Licensed under [CC-BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) by the Arch Linux team

[Pictures](https://static.conf.archlinux.org/archconf2019/pictures/) [(Cloud Link)](https://www.jottacloud.com/p/foxboron/_3032773dd72a47ddb602c724685e35f1/thumbs)

Licensed under [CC-BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) by the Arch Linux team

# Day 1

## Infrastructure

There was an small talk about the current infrastructure we provide, the problems we currently have which are:

* User management
* Security
* Lack of overview
* Old unmaintained software in operation (flyspray, bbs, planet, mailman 2)
* Not ideal Ansible setup
* Mix between VPS/dedicated machines
* Unmanaged Luna Server (cgit, AUR, BBS)
* Hard to mirror between cgit and Github
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

## Package standards and Quality

Arch has packaging guidelines of varying quality and some of them are often
ignored. Below is a collection of findings from a discussion with various
developers and trusted users.
These data points need to be incorporated into the [package
guidelines](https://wiki.archlinux.org/index.php/Arch_package_guidelines) and
the [packager
howto](https://wiki.archlinux.org/index.php/DeveloperWiki:HOWTO_Be_A_Packager).

To increase the knowledge sharing on best practices it would be beneficial to
add experienced maintainers for the various languages (e.g. Haskell, Go, Java,
Ruby, etc.) as contacts to the Wiki so people can have a look at "model"
PKGBUILDs instead of random ones in the repos.

Additionally, issues that come up during TU application reviews are a good
example of inconsistencies, that are worth documenting right away.

The following list is a (non-exhaustive) collection of examples, that still
require addition in the wiki.

- use HTTPS for sources wherever possible, "git+https://" for git
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
- run tests wherever possible
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

A group of participants was interested in discussing versioned kernel installs.
A [long-standing bug report](https://bugs.archlinux.org/task/16702) evolves
around the topic.

### Problems

* Upon installing a new kernel, the module tree is removed, meaning that
  subsequent loading of modules without a reboot fail (e.g. pluging an USB
  drive)
* If there is an issue with the new version, there is no "old" kernel to boot
  into (e.g. when linux-lts or any other kernel might not be a suitable
  fallback)

### Open Questions

* Up until lately we installed /boot/vmlinuz-linux and generated an associated
  /boot/initramfs-linux.img from it, which led to the question how to deal with
  rewriting a specific bootloader configuration on each kernel update.
* Symlinking the initramfs images: A default name links to newest version (this
  does not work on FAT)
* kernel-install from systemd can manage multiple kernels (but assumes UEFI
  systems)
* Not all bootloaders have a way to specify glob pattern in kernel
  image/ initramfs so that they grab all versioned kernels

### Solutions

[Changes to the kernel packages and
mkinitcpio](https://www.archlinux.org/news/new-kernel-packages-and-mkinitcpio-hooks/)
en route to a possible move to Dracut enable us to at least leave old kernels +
initramfs around, making rescue boots possible:

* /boot/ is no longer part of the kernel packages
* Hooks in mkinitcpio and/or Dracut (possibly using kernel-install) are now
  able to copy kernels from /lib/modules to /boot/ in different naming schemes
  (this needs further work)
* Seblu versioned kernels with kernel-install and mkinitcpio:
  * https://git.seblu.net/archlinux/linux-seblu-meta
  * https://git.seblu.net/archlinux/kernel-install-poc

# Day 2

## svn2git migration

[arch-repo-management](https://github.com/archlinux/arch-repo-management)

### Team members / participants:

* heftig (Project lead)
* alad
* maximbaz
* jleclanche
* Pierre
* daurnimator?

### Goals

* Move our current svn approach for repository management to git
* Include support for debug packages

### New implementation from scratch

* dbscripts is old, difficult to understand and extend
  * Allows theoretical extensions for `git` (as of recent; no actual support included)
  * Documentation of dbscripts behavior
    * What does it do?
      * Handling of edge cases
    * What can it do? What can it not do?
      * Issues with any/x86_64 in split packages
      * Cannot properly (db-)move split packages
  * Possible languages
    * Python: easy to (unit-)test, library support - **AGREED ON THIS**
    * Bash: most people were against this (hard to test, general language difficulties)

### Monorepo approach

* Packages (pkgbase) are maintained in separate git repositories
  * Tags: `pkgname-(epoch:)pkgver-pkgrel`
    * Not tagged by arch (a PKGBUILD may work for several archictectures)
    * Signed tags + verification
      * Option (A) Tags can be force pushed until a "release"
      * Option (B) Do not allow force pushing tags - **AGREED ON THIS**
  * Split in subdirs by first two letters?
* A separate git repository is kept for management of the (binary) arch repos/mirrors
  * Layout proposals
    * arch -> repo -> pkgbase
      https://github.com/dvzrv/arch-repo-management#git-repository-layout
    * arch -> repo -> pkgbase -> pkgname
  * Advantage: keep history of precise state of Arch Linux repositories at any given time
    * Possible mismatches between monorepo and built database?
      * Alternative: "repo-add" that works untarred tree, release == tar
  * Option (1) Keep a large monorepo with full copies of all packages<br />
    Only needed if requiring access to the source files<br />
    Has the nice side-effect of containing an ABS-style tree for users to clone
    * `git submodule`
      * Least amount of possible data
      * More difficult to use (but automated anyway by scripts)
    * `git subtree`
      * Hard to remove files a-posteri (example: distribution/legality issues)
    * `git read-tree`
      * "git-subtree with less history"
  * Option (2)  Keep a monorepo with only metadata of packages - **AGREED ON THIS**
    * May offer performance improvements over full copy monorepo (1)
    * Lets us generate DB files directly from the repo state, without manipulating them using repo-add
      * replace `repo-add` (in dbscripts) with custom code that generates a DB file from the monorepo contents alone
    * folder structure:
      ```
      x86_64/
        community/
          foo.json (where foo is pkgbase)
          bar.json (where bar is pkgbase)
        community-staging/
        core/
        core-debug/
       ```
    * file format: pretty-printed with sorted keys JSON with fields:
      * version
      * makedepends (list) 
      * checkdepends (list)
        * packages (list of objects with fields below)
        * filename
        * name
        * desc
        * groups (list)
        * csize (int)
        * isize (int)
        * md5sum
        * sha256sum
        * pgpsig
        * url
        * license (list)
        * arch (because can also be "any")
        * builddate (int)
        * packager
        * replaces (list)
        * conflicts (list)
        * provides (list)
        * depends (list)
        * optdepends (list)
        * backup (list)
        * files (list)

### Operations

* Add/Update (integrate new .pkg.tar.xz for one or multiple pkgbases into the DB)
  * ✅ Collect packages from staging dir, parse .PKGINFO
    * ✅ Group by repo and pkgbase
      * ✅ `{'extra': {'foo': data, 'bar': data}, 'testing': { ... }}`
      * ✅ if pkgbase already seen but common fields (version, makedepends, checkdepends) differ, error out
      * ✅ do GPG verification?
  * ✅ For each repo to process:
    * ✅ Load repo JSON data
  * For each pkgbase:
    * ✅ Ensure version is increasing (pyalpm vercmp)
    * Shallow clone tag named "$version" from package repository named "$pkgbase" to get PKGBUILD
    * GPG-verify tag?
    * Run `makepkg --packagelist` to get list of expected packages
      * Verify against packages collected
    * Do other verification checks between `PKGBUILD` and packages? Check current dbscripts
    * Get rid of clone
    * Copy the packages into the FTP pool
      * Existing file is an error
    * Link the packages from the FTP repo dir
      * Existing file is an error
    * Copy package data into repo data
  * ✅ Write out JSON
  * ✅ Write out DB files
  * git commit
  * Remove old symlinks
* Remove (remove existing pkgbases)
  * ✅ For each repo to process: (existing db-remove operates on a single repo)
    * ✅ Load repo JSON data
  * ✅ For each pkgbase:
    * ✅ Remove pkgbase from data
  * ✅ Delete JSON files
  * ✅ Write out DB files
  * `git commit`
  * Remove old symlinks
* Move (move existing pkgbases from e.g. testing to extra)
  * ✅ For each repo to process:
    * ✅ Load repo JSON data
  * For each pkgbase:
    * ✅ Move data
    * Add new symlinks
  * ✅ Write out JSON
  * ✅ Write out DB files
  * `git commit`
  * Remove old symlinks

### Tasks

* Code to load JSON and stream out a database - heftig
  * Generates a tar written to the `foo.db.tar.gz`
* Code to load packages and write out JSON - maximbaz
* ✅ For testing purposes, code to convert a `foo.db.tar.gz` into JSON - alad
* Rewrite devtools' commitpkg to use git instead of svn

### (Unit) testing

* All code should be properly tested
* https://docs.python.org/3/library/unittest.html

### Performance testing

* Currently ~10k packages in the official repositories
  * Initial testing with small amount of packages (100-500 packages)
  * Extend to multiple of packages to ensure scaling for the foreseeable future
    * Example: AUR (~55k packages)
  * Compare multiple approaches (and their different implementations like subtree vs submodule)

### Documentation
  
* Let's document everything along the way
* Avoid the dbscripts scenario where very few people actually understand the code

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

## Project governance

During Arch Conf we discussed the topic of project governance and its many
facets
([presentation](https://pkgbuild.com/~dvzrv/presentation/archconf2019-project-governance.pdf)).
Historically, Arch Linux has always been somewhat of a loosely coupled
meritocracy, lead by a Benevolent Dictator For Life (BDFL).

### Problems

We have identified the following (non-exhaustive) list of problems, that
bother us as a team and that block us from developing the future of Arch Linux:

- Infrastructure
  - Many bigger infrastructure topics (some topics go unidentified for years,
    others get bikeshedded to death and then nothing ever happens)
- Intransparancy
  - Financial intransparency (we have no real treasure guidelines, sometimes
    the team can't decide on _what_ is worth spending money on, even if it's
    not much)
  - Large parts of the team are not really aware of the general "Arch status"
    (e.g. who are all these people and what are they actually doing? How is
    Arch doing?)
  - Developer role intransparency (how is the developer role defined? how are
    they voted in place, how are they demoted?)
- Team Chemistry
  - People discouraging each other from working on projects (team toxicity has
    been building up)
  - Some members of the team are very secluded and disconnected from the rest
- Missing Ownership
  - We have no PR management (which is currently a problem as sponsors would
    like Twitter shoutouts but we've lost access to our Twitter account, for
    instance)
  - Bus factor = 1 (trademark, SPI, several internal projects, packaging)
  - money spending is dealt with via Aaron + developer consent (which can take
    long)
  - No Arch Game Rules / RFC / PEP (we have no written rules and process to
    update them, such as TU bylaws) Review process: leadership?

### Actionables

During the discussion round we came up with a few actions to take.

- Team
  - do (community centric) Arch Conf yearly!
  - meet at FOSDEM (hackathon!)
  - create a Conf, that is more user centric (requires sponsors)
  - encourage (and do!) online meetings, pair programming
- Increase Ownership
  - lower bus factor by actively encouraging single-person projects to find
    co-maintainers
  - encourage developers to (be able to) work on all internal projects
  - encourage everyone to be mindful to opt out properly (not silently), if
    there's no time
  - think about codified "allowances" for yearly events (e.g. Arch Conf) where
    community members get travel refunds
  - delegation of spending, e.g. any developer can spend up to $500 without
    approval (e.g. a dinner); 3 developers required for up to $5000; all + Aaron
    required for more?
  - allowing for e.g. infrastructure team to spend
  - get Hetzner to bill SPI instead of Ionuţ (aka wonder)
- Transparency
  - share / publish / highlight SPI report
  - define and open up developer role
  - define e.g. an RFC based process to codify Arch's internals and decision
    making processes
- Infrastructure
  - find responsible team lead for major infrastructure topics, which will be
    responsible for it

In a follow-up work group to the initial presentation and discussion we
discussed the idea of Arch Linux' project leadership further, as many of the
brought up topics fell under the categories of *"Missing Ownership"* and
*"Intransparency"*.

We came up with the following wishlists for a general leadership role and
leadership roles per project.

#### Arch Linux Project Leadership

- Identify (technical) problems in Arch
- Conduct regular meetings in which to discuss problems and find resolutions to
  those (assign to subleaders, defer, close)
- Increase transparency amongst the team by doing regular general status
  reports
- Organize a list of ongoing processes/ subprojects and keep track of those
  (regularly poll involved people)
- Improve the decision making process by e.g. guiding an RFC based definition
  of Arch's internal workings
- Be electable
- More actively seek sponsorship

#### Leadership for Subprojects

- Appointed by project leadership during meetings to tackle a specific
  technical problem and/ or voted into place by the members of a specific
  subproject
- Identify issues and tasks in a specific technical domain
- Conduct regular meetings in which project specifics are discussed, progress
  is documented and shared with everyone afterwards
- Track progress on the subproject's tasks

## Base Package

The idea of turning the `base` group into a `base` package [has been proposed in
January this
year](https://lists.archlinux.org/pipermail/arch-dev-public/2019-January/029435.html)
(while [follow ups lasted well into
March](https://lists.archlinux.org/pipermail/arch-dev-public/2019-March/029491.html)).

### Problems

* Base as a group contains many (unneeded) packages
* Some assume it being installed, others don’t (as the members of a group can't
  be tracked without reinstalling it, this is difficult)

### Solutions

As a follow up on the proposed change, several modifications were made (during Arch Conf):

* Create a new `base` meta-package, which is assumed being installed on all systems.
* Base as a package means dependencies can be added and removed on the fly and
  being more transparent
  ([PKGBUILD](https://git.archlinux.org/svntogit/packages.git/tree/trunk/PKGBUILD?h=packages/base))
* The new package was pushed to `[core]` and a
  [TODO](https://www.archlinux.org/todo/base-group-removal/) created to remove
  all packages from the `base` group
* The [installation
  guide](https://wiki.archlinux.org/index.php/Installation_guide#Install_essential_packages)
  was modified to reflect this change.
* The kernel packages (e.g.
  [linux](https://git.archlinux.org/svntogit/packages.git/commit/trunk/PKGBUILD?h=packages/linux&id=f1c97a49a09b0fd8d7d1c3f6e8e635ef45974373))
  were modified to add linux-firmware as an optional dependency.
* A [news
  item](https://www.archlinux.org/news/base-group-replaced-by-mandatory-base-package-manual-intervention-required/)
  was posted to notify of the change while [a message to the arch-dev-public
  mailing
  list](https://lists.archlinux.org/pipermail/arch-dev-public/2019-October/029693.html)
  elaborated further on the change and potential pitfalls.

### Further follow ups

* Evaluate removal of `bzip2`, `gzip` and `xz` as they are a dependency for
  something else (e.g. pacman) or [not useful on their
  own](https://bugs.archlinux.org/task/64028)
* Evaluate removal of [network tools](https://bugs.archlinux.org/task/64030),
  [pciutils](https://bugs.archlinux.org/task/64029)

## Golang packaging

Participants:

* Foxboron
* Felixonmars

The main goal of the packaging of go is to get rid of vendored dependencies. To
make sure we are able to onboard new team members to package go, this should be
tooling assisted.

Debian has spent a [lot of time
detailing](https://go-team.pages.debian.net/packaging.html) their efforts
packaging golang proper. The idea is to utilize these guidelines and have our
own tool to aid in the packaging.

Debian also has their own tool to support packagers,
[dh-make-golang](https://github.com/Debian/dh-make-golang), which we will
utilize to provide our own tool, [gopkg](https://github.com/Foxboron/gopkg).

Rest of the session was spent introducing the tool and discussing potential
migration plans. The packages shouldn't be arbitrarily dropped into the given
repository as they could be hundreds. Instead we will do packaging in a
temporary repository to ensure we are able to properly maintain golang packages
this way.

We also fixed a [bug in
glider](https://git.archlinux.org/svntogit/community.git/commit/trunk?h=packages/glider&id=7b33bb1c722dd358557ed2158f6babf5f4cd94e9) together :)

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
