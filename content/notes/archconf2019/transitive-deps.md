---
title: "Transitive Deps"
date: 2019-10-12T16:54:23+02:00
draft: true
---

# transitive dependencies

# Related
* https://lists.archlinux.org/pipermail/arch-projects/2017-August/004607.html


# Problems: 
* packages break when a dependency they need is removed down the chain where they used to get it from
* Namcap doesn't care (e.g. it will suggest to remove “already included” dependencies) : https://bugs.archlinux.org/task/64022
* Raises issues like https://bugs.archlinux.org/task/54887
* Hard to grasp which packages actually depend on something just by looking at reverse dependencies

# Solutions
* all NEEDED library requirements should be in depends/optdepends (change namcap)
* packaging guidelines need to be changed to make sure packagers add shared library names to provides array
-> can be automatic, at least the provides= (only if the soname is specific(?))
* add symbols to provides (e.g. rpm)?
* patch projects using libtool to use as-needed (which apparently is discarded)
-> always use `autoreconf -vfi` to recreate `./configure` should be added to packaging guidelines

