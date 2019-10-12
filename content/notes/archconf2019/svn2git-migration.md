---
title: "Svn2git Migration"
date: 2019-10-12T16:56:08+02:00
---

# svn2git migration
https://github.com/archlinuxarch-repo-management
 
Team members / participants:
heftig (Project lead)
alad
maximbaz
jleclanche
Pierre
daurnimator?
 
Goals
Move our current svn approach for repository management to git
Include support for debug packages
 
New implementation from scratch
dbscripts is old, difficult to understand and extend
Allows theoretical extensions for git (as of recent; no actual support included)
Documentation of dbscripts behavior
What does it do?
Handling of edge cases
What can it do? What can it not do?
Issues with any/x86_64 in split packages
Cannot properly (db-)move split packages
Possible languages
Python: easy to (unit-)test, library support - AGREED ON THIS
Bash: most people were against this (hard to test, general language difficulties)
 
Monorepo approach
https://github.com/archlinux/arch-repo-management
Packages (pkgbase) are maintained in separate git repositories
Tags: pkgname-(epoch:)pkgver-pkgrel
Not tagged by arch (a PKGBUILD may work for several archictectures)
Signed tags + verification
Option (A) Tags can be force pushed until a "release"
Option (B) Do not allow force pushing tags - AGREED ON THIS
Split in subdirs by first two letters?
A seperate git repository is kept for management of the (binary) arch repos/mirrors
Layout proposals
arch -> repo -> pkgbase
https://github.com/dvzrv/arch-repo-management#git-repository-layout
arch -> repo -> pkgbase -> pkgname
Advantage: keep history of precise state of Arch Linux repositories at any given time
Possible mismatches between monorepo and built database?
Alternative: "repo-add" that works untarred tree, release == tar
Option (1) Keep a large monorepo with full copies of all packages
Only needed if requiring access to the source files
Has the nice side-effect of containing an ABS-style tree for users to clone
git submodule
Least amount of possible data
More difficult to use (but automated anyway by scripts)
git subtree
Hard to remove files a-posteri (example: distribution/legality issues)
git read-tree
"git-subtree with less history"
Option (2)  Keep a monorepo with only metadata of packages - AGREED ON THIS
May offer performance improvements over full copy monorepo (1)
Lets us generate DB files directly from the repo state, without manipulating them using repo-add
replace repo-add (in dbscripts) with custom code that generates a DB file from the monorepo contents alone
folder structure:
 
x86_64/
    community/
        foo.json (where foo is pkgbase)
        bar.json (where bar is pkgbase)
    community-staging/
    core/
    core-debug/
 
file format: pretty-printed with sorted keys JSON with fields:
version
makedepends (list) 
checkdepends (list)
packages (list of objects with fields below)
filename
name
desc
groups (list)
csize (int)
isize (int)
md5sum
sha256sum
pgpsig
url
license (list)
arch (because can also be "any")
builddate (int)
packager
replaces (list)
conflicts (list)
provides (list)
depends (list)
optdepends (list)
files (list)
 
Operations
Add/Update (integrate new .pkg.tar.xz for one or multiple pkgbases into the DB)
Collect packages from staging dir, parse .PKGINFO
Group by repo and pkgbase
{'extra': {'foo': data, 'bar': data}, 'testing': { ... }}
if pkgbase already seen but common fields (version, makedepends, checkdepends) differ, error out
do GPG verification?
For each repo to process:
Load repo JSON data
For each pkgbase:
Ensure version is increasing (pyalpm vercmp)
Shallow clone tag named "$version" from package repository named "$pkgbase" to get PKGBUILD
GPG-verify tag?
Run makepkg --packagelist to get list of expected packages
Verify against packages collected
Do other verification checks between PKGBUILD and packages? Check current dbscripts
Get rid of clone
Copy the packages into the FTP pool
Existing file is an error
Link the packages from the FTP repo dir
Existing file is an error
Copy package data into repo data
Write out JSON
Write out DB files
git commit
Remove old symlinks
Remove (remove existing pkgbases)
For each repo to process:
Load repo JSON data
For each pkgbase:
Remove pkgbase from data
Write out JSON
Write out DB files
git commit
Remove old symlinks
Move (move existing pkgbases from e.g. testing to extra)
For each repo to process:
Load repo JSON data
For each pkgbase:
Move data
Add new symlinks
Write out JSON
Write out DB files
git commit
Remove old symlinks
 
Tasks
Code to load JSON and stream out a database - heftig
Generates a tar written to the foo.db.tar.gz
Code to load packages and write out JSON - maximbaz
For testing purposes, code to convert a foo.db.tar.gz into JSON - alad
Rewrite devtools' commitpkg to use git instead of svn
 
(Unit) testing
All code should be properly tested
https://docs.python.org/3/library/unittest.html
 
Performance testing
Currently ~10k packages in the official repositories
Initial testing with small amount of packages (100-500 packages)
Extend to multiple of packages to ensure scaling for the foreseeable future
Example: AUR (~55k packages)
Compare multiple approaches (and their different implementations like subtree vs submodule)
 
Documentation
Let's document everything along the way
Avoid the dbscripts scenario where very few people actually understand the code

