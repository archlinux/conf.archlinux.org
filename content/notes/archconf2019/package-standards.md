---
title: "Package Standards"
date: 2019-10-12T16:53:57+02:00
---

Some issues that came up during previus TU application reviews that might be worth documenting e.g. in https://wiki.archlinux.org/index.php/Arch_package_guidelines:

- use HTTPS for sources whereever possible, "git+https://" for Git 
- don't use specific sourceforge mirror to download: https://bbs.archlinux.org/viewtopic.php?id=22200 
- use PGP signatures where possible (may need to build from Git tag instead of source tarball) 
- rename sources to something unique for shared $SRCDIR

- be consistent with the use of cd "$srcdir/..." (which is not needed anyway) 
- don't use internal functions like msg2

- autoreconf/autogen.sh/bootstrap/... in prepare(), not build() 
- always do autoreconf -if 
- don't use sed as it fails silently 
- upstream patches where possible 

- run tests whereever possible

- don't build implicitly in package() 
- use /usr instead of /etc where possible, e.g. /usr/lib/systemd/system 
- only warnings, no documentation in install files

- no need for provides=("$pkgname") 
- use sysusers.d and tmpfiles.d instead of doing stuff manually, don't delete users on uninstall: https://www.archlinux.org/todo/usergroup-management/

Specific guidelines that should be improved:
    - https://wiki.archlinux.org/index.php/Haskell_package_guidelines (completely empty)
    - https://wiki.archlinux.org/index.php/Java_package_guidelines (suggests copying binaries instead of building from source is ok)

Add experienced maintainers for Haskell, Go, Java, ... as contacts to the Wiki so people can look at "model" PKGBUILDs instead of random ones in the repos?

