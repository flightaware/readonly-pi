# readonly-pi

This is a set of initrd scripts that run early during boot and, based on
the boot commandline, arrange for the "real" root filesystem to be kept
readonly. An overlayfs is used as the runtime root filesystem, with the
on-disk filesystem as a readonly lower layer and tmpfs used to store changes
in RAM at runtime.

## CAUTION

Similar scripts are used in FlightAware's FlightFeeder software, where they
work well, but this separate package has not been tested - consider it a
starting point.

Also, this has only ever been used on Raspberry Pis - it may need changes
for other systems.

Pull requests to improve it are welcome!

## Prerequisites

You need a Pi setup that uses an initrd. The FlightFeeders have some custom
logic to do this as Raspbian did not support it at the time. Subsequently
Raspbian seems to be adding direct support for initrds, but I haven't looked
into the details.

## Build it

```
$ dpkg-buildpackage -b
```

## Install it

```
# dpkg -i readonly-pi_*.deb
```

## Use it

Add `overlay=yes` to /boot/cmdline.txt and reboot.
To switch back to a normal root FS, change it to `overlay=no` or remove the
overlay arg.

## Helper scripts

`switch-to-readonly` and `switch-to-readwrite` will handle updating cmdline.txt
for you. Pass `-r` if you want to reboot after making a change. The scripts
are no-ops (and will tell you about it) if the system is already in the
requested mode.

## How overlay=yes works

When in readonly mode, the filesystem layout looks like this:

```
/          overlayfs using /ro (lower layer) and /rw (upper layer)
/ro        readonly-mounted real root FS
/rw        tmpfs that will be used for changes
/rw/upper  overlayfs upper dir
/rw/work   overlayfs work dir
```

The tmpfs on /rw is sized as: 25% of RAM in excess of 128MB, constrained to [64MB,256MB]

## Gotchas

Most obviously, _all changes you make to the root filesystem in readonly mode will be lost
on reboot_.

If you do lots of filesystem changes on a readonly system, you might run out of tmpfs space.

Make sure you switch to readwrite mode before doing kernel/firmware upgrades.
/boot is still read-write in readonly mode, so if you forget to do this then
changes will be made in /boot but the corresponding package accounting
information on the root filesystem won't be updated.
