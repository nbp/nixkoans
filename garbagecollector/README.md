# Garbage Collector

Most other package managers (dpkg) have a way to remove unused software.  Nix is a lot more precise compared to other systems. With other package managers, one will most likely end up with some unncessary package installed or dangling files.  With nix, this does not happen.

We decide whether a store path is still needed the same way garbage collector in programming languages decide whether an object is alive.  GC roots are our store paths.

* GC roots are stored under `/nix/var/nix/gcroots`.
* If there's a symlink to a store path, then that store path is a GC root.
* Nix allows this directory to have subdirectories and it will recursively search for symlinks to store paths.

With a list of GC roots, knowing which dead store path to delete can be achieved by comparing to all our live store paths.  All dead store paths are moved to `/nix/store/trash` as an atomic operation; afterwards, the trash is emptied.

## Run the Garbage Collector

```
$ nix-collect-garbage
```

This cleans out all dead links.

Another example:

```
$ nix-env -iA nixos.pkgs.bsdgames
...

$ readlink -f `which fortune`
/nix/store/x7v07qyf20h2h91rzhvq9b3dvpzxg5ar-bsd-games-2.17/bin/fortune

$ nix-store -q --roots $(readlink -f `which fortune`)
/nix/var/nix/profiles/default-1-link

$ nix-env --list-generations
   1   2015-05-26 04:09:01   (current)

$ nix-env -e bsd-games
uninstalling ‘bsd-games-2.17’
building path(s) ‘/nix/store/ivzdnyirgssbvvwfb3hac2mm7nhdfld2-user-environment’
created 0 symlinks in user environment


$ nix-env --list-generations
   1   2015-05-26 04:09:01
   2   2015-05-26 04:12:40   (current)```

$ nix-collect-garbage
finding garbage collector roots...
deleting garbage...
deleting ‘/nix/store/fpli8mgny3qs77z5i27q1an08639m4nl-bsd-games-2.17.drv’
deleting ‘/nix/store/byfxsa2zh6g9l5i5mwjf960nxy5rdcdd-miscfiles-1.5.drv’
deleting ‘/nix/store/l63944g20xwrw3p8kf51dz8k7lsnwfkf-miscfiles-1.5.tar.gz.drv’
deleting ‘/nix/store/v0ryfpkkrinblifawswlad2lknaxs3sg-bsd-games-2.17.tar.gz.drv’
deleting ‘/nix/store/cihfwjd7yrngr05d7s66b1ilvll5f0jm-dm-noutmpx.patch.drv’
deleting ‘/nix/store/trash’
deleting unused links...
note: currently hard linking saves -0.00 MiB
5 store paths deleted, 0.02 MiB freed

$ ls /nix/store/x7v07qyf20h2h91rzhvq9b3dvpzxg5ar-bsd-games-2.17
bin  share
```
