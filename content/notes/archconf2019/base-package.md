---
title: "Base Package"
date: 2019-10-12T16:58:52+02:00
---

# Base package

# Current situation
base is a group, with a lot of stuff inside
some people assumed it installed, some don’t: as it’s a group, this is difficult

# Proposal
New base meta-package (assumed installed): being a package means we can add/remove things later and propagate that : https://git.archlinux.org/svntogit/packages.git/tree/trunk/PKGBUILD?h=packages/base
See https://lists.archlinux.org/pipermail/arch-dev-public/2019-January/029435.html and https://lists.archlinux.org/pipermail/arch-dev-public/2019-March/029491.html for included packages and further discussion
Push the package to [core]
Done
Create a TODO list for base group removal
Sent
Update documentation
install base and linux in the default install
Installation Guide edited
https://git.archlinux.org/svntogit/packages.git/commit/trunk/PKGBUILD?h=packages/linux&id=f1c97a49a09b0fd8d7d1c3f6e8e635ef45974373
explicit base as a mandatory package
News item
Sent

# For later
Remove 'bzip2' 'gzip' 'xz' : they either are dependency for something else (e.g. pacman) or not useful by themselves: https://bugs.archlinux.org/task/64028
Remove network tools, pciutils ? https://bugs.archlinux.org/task/64029, https://bugs.archlinux.org/task/64030


News post:
    
Following recent mailing list discussions, we have decided to replace the `base` group by a metapackage of the same name. We advise users to install this package, as it is effectively mandatory from now on. Users requesting support are expected to be running a system with the base package.

We have decided to replace the `base` group by a metapackage of the same name, we advise users to install this package (`pacman -Syu base`), as it is effectively mandatory from now on.
Users requesting support are expected to be running a system with the base package.

