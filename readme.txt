1) etckeeper (http://etckeeper.branchable.com/)
  * distribution agnostic, commonly used with Debian, but also many others
  * written by Joey Hess (quality software! :)
  * actively maintained & developed
  * default VCS is git, but mercurial and darcs are also possible
  * does extra accounting for file permissions, empty directories and symlinks
    those attributes are only partially supported by different VCS
  * all files under version control, gitignore is possible to exclude special
    files (e)g. changed by daemons, security concerns, etc..
  * Configurable through hook files
  * All the goodies that come with git
    remotes, branches, diffing, history, ).
  * sudo integration
    records actions as the user who invoked sudo

2) etcupdate (http://www.freebsd.org/cgi/man.cgi?query=etcupdate)
  * FreeBSD
  * part of userland: actively maintained & developed
  * three-way merge of changes
  * keeps copies of current & previous file versions
  * automatic merge when there is no conflict
  * all conflicts must be resolved before new merges can be performed

3) the Arch way (https://wiki.archlinux.org/index.php/Pacnew_and_Pacsave_files)
  * actually kinda distribution agnostic, similar to other distributions
  * packages can define files which need a backup for update or removal
    `backup` array in `PKGBUILD`
  * three backup files: *)pacnew, *.pacsave and *.pacorig
  * compares `md5sum`s of current, previous and new
  * *)pacnew:
    current version is different from previous
    => user modifications, unable to merge, install new file as *)pacnew
  * *)pacsave:
    package is removed, but current file has user modifications
    => rename to *)pacsave, leave on filesystem
  * *)pacorig:
    does not belong to a package, but listed in `backup`
    => move to *)pacorig and replace with version from package
  * user is prompted with a hint to check & solve the situation manually
  * tools for automatically diffing and editing (e)g. with vimdiff. exist
  * oftentimes simple bash scripts, or such, e)g.:

  ```
    for *.pac{new,save,orig}; do
      vimdiff $f $(echo $f | sed -e 's/\(\.pacnew\|\.pacsave\|\.pacorig\)$//');
    done
  ```

  * more advanced scripts are etc-update and dispatch-conf from Gentoo:
    http://www.gentoo-wiki.info/HOWTO_etc-update
    http://www.gentoo-wiki.info/TIP_dispatch-conf

4) fillup (https://github.com/openSUSE/fillup)
  * code base largely from the 90's
  * looks like it's unmaintained
  * tries to be clever about comments
  * rpm macros: %fillup_and_insserv, %fillup_only
  * functionality is a mix of 2) and 3)

5) Nix (https://nixos.org/nixos/about.html)
  * declarative system configuration model
  * purely functional description of a system state
  * multiple configurations can coexist
  * reliable system updates because the initial state is well known
  * configuration changes are atomic
    transactional approach allows rollbacks
  * well studied: http://nixos.org/~eelco/pubs/hotos-final.pdf

6) Debian (https://wiki.debian.org/ConfigPackages)
  there are multiple ways of dealing with configuration files:

    a) put config files into package and install scripts
      + single object
      + global config can be systematically removed and reversed
      + config can be tested with `dpkg -i`
      + config can be securely distributed with the package
      - basic package setup takes about half an hour
      - existing pkgs need to be signed and uploaded to local repository
      - it's necessary to check how other pkgs handle a particular file

    b) dpkg interactive conflict resolution
      - runs automatically when a config file is replaced
        solutions:
        => c)
        => d)

    c) divert config into a separate package
      + handles multiple config states with symlinks
      - complex interaction if pkg removed, but config pkg remains
      - requires symlink handling in `postinst` and `prerm` script hooks

    d) replace both file and checksum, so dpkg doesn't recognize the change
      - new config files will replace locally customized files

Proposal
========

Plan a)
  Reuse the Nix package management system
  + Well thought out, scientifically studied solution
  + Functional, declarative awesomeness
  - Many people won't like it, because it breaks their habitual way of thinking

Therefore, Plan b)
  Use etckeeper and build some git porcelain around it.
    (e.g. `etc edit /etc/motd` would launch `$EDITOR` and automatically commit
    changes afterwards. Idea is taken from https://github.com/RichiH/vcsh)

  + Reuse existing tools, don't invent the wheel again
  + git is well known, many people are familiar with it
  + Full version history:
    Log of all changes to the system configuration
  + diffing and merging of (remote) branches:
    Easily compare configuration of two or more systems
  + Extending with shell scripts hooks is trivial

  - All of etc needs to be under version control
    Files changed by deamons or with security concerns need to be excluded
  - Users need to learn how to properly use the new toolchain

Therefore, Plan c)
  Reuse etcupdate from FreeBSD
  + Does almost the same as Debian, but looks less complex
  + No license trouble, because BSD
