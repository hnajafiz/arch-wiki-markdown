fstab
=====

Related articles

-   Persistent block device naming
-   NTFS Write Support
-   Firefox Ramdisk
-   Boot Debugging
-   udev

The /etc/fstab file can be used to define how disk partitions, various
other block devices, or remote filesystems should be mounted into the
filesystem.

Each filesystem is described in a separate line. These definitions will
be converted into systemd mount units dynamically at boot, and when the
configuration of the system manager is reloaded. The default setup will
automatically fsck and mount filesystems before starting services that
need them to be mounted. For example, systemd automatically makes sure
that remote filesystem mounts like NFS or Samba are only started after
the network has been set up. Therefore, local and remote filesystem
mounts specified in /etc/fstab should work out of the box. See
man 5 systemd.mount for details.

The mount command will use fstab, if just one of either directory or
device is given, to fill in the value for the other parameter. When
doing so, mount options which are listed in fstab will also be used.

Contents
--------

-   1 File example
-   2 Field definitions
-   3 Identifying filesystems
    -   3.1 Kernel name
    -   3.2 Label
    -   3.3 UUID
-   4 Tips and tricks
    -   4.1 Automount with systemd
    -   4.2 Filepath spaces
    -   4.3 External devices
    -   4.4 atime options
    -   4.5 tmpfs
        -   4.5.1 Usage
            -   4.5.1.1 Improving compile times
                -   4.5.1.1.1 For one session
                -   4.5.1.1.2 Permanently
    -   4.6 Writing to FAT32 as Normal User
    -   4.7 Remounting the root partition
-   5 See also

File example
------------

A simple /etc/fstab, using kernel name descriptors:

    /etc/fstab

    # <file system>        <dir>         <type>    <options>             <dump> <pass>
    /dev/sda1              /             ext4      defaults,noatime      0      1
    /dev/sda2              none          swap      defaults              0      0
    /dev/sda3              /home         ext4      defaults,noatime      0      2

Field definitions
-----------------

The /etc/fstab file contains the following fields separated by a space
or tab:

     <file system>        <dir>         <type>    <options>             <dump> <pass>

-   <file system> - the partition or storage device to be mounted.
-   <dir> - the mountpoint where <file system> is mounted to.
-   <type> - the file system type of the partition or storage device to
    be mounted. Many different file systems are supported: ext2, ext3,
    ext4, btrfs, reiserfs, xfs, jfs, smbfs, iso9660, vfat, ntfs, swap
    and auto. The auto type lets the mount command guess what type of
    file system is used. This is useful for optical media (CD/DVD).
-   <options> - mount options of the filesystem to be used. Note that
    some mount options are filesystem specific. Some of the most common
    options are:

-   auto - Mount automatically at boot, or when the command mount -a is
    issued.
-   noauto - Mount only when you tell it to.
-   exec - Allow execution of binaries on the filesystem.
-   noexec - Disallow execution of binaries on the filesystem.
-   ro - Mount the filesystem read-only.
-   rw - Mount the filesystem read-write.
-   user - Allow any user to mount the filesystem. This automatically
    implies noexec, nosuid, nodev, unless overridden.
-   users - Allow any user in the users group to mount the filesystem.
-   nouser - Allow only root to mount the filesystem.
-   owner - Allow the owner of device to mount.
-   sync - I/O should be done synchronously.
-   async - I/O should be done asynchronously.
-   dev - Interpret block special devices on the filesystem.
-   nodev - Don't interpret block special devices on the filesystem.
-   suid - Allow the operation of suid, and sgid bits. They are mostly
    used to allow users on a computer system to execute binary
    executables with temporarily elevated privileges in order to perform
    a specific task.
-   nosuid - Block the operation of suid, and sgid bits.
-   noatime - Don't update inode access times on the filesystem. Can
    help performance (see atime options).
-   nodiratime - Do not update directory inode access times on the
    filesystem. Can help performance (see atime options).
-   relatime - Update inode access times relative to modify or change
    time. Access time is only updated if the previous access time was
    earlier than the current modify or change time. (Similar to noatime,
    but doesn't break mutt or other applications that need to know if a
    file has been read since the last time it was modified.) Can help
    performance (see atime options).
-   discard - Issue TRIM commands to the underlying block device when
    blocks are freed. Recommended to use if the filesystem is located on
    an SSD.
-   flush - The vfat option to flush data more often, thus making copy
    dialogs or progress bars to stay up until all data is written.
-   nofail - Mount device when present but ignore if absent. This
    prevents errors being reported at boot for removable media.
-   defaults - the default mount options for the filesystem to be used.
    The default options for ext4 are: rw, suid, dev, exec, auto, nouser,
    async.

-   <dump> - used by the dump utility to decide when to make a backup.
    Dump checks the entry and uses the number to decide if a file system
    should be backed up. Possible entries are 0 and 1. If 0, dump will
    ignore the file system; if 1, dump will make a backup. Most users
    will not have dump installed, so they should put 0 for the <dump>
    entry.

-   <pass> - used by fsck to decide which order filesystems are to be
    checked. Possible entries are 0, 1 and 2. The root file system
    should have the highest priority 1 (unless its type is btrfs, in
    which case this field should be 0) - all other file systems you want
    to have checked should have a 2. File systems with a value 0 will
    not be checked by the fsck utility.

Identifying filesystems
-----------------------

There are three ways to identify a partition or storage device in
/etc/fstab: by its kernel name descriptor, label or UUID. The advantage
of using UUIDs or labels is that they are not dependent on the order in
which the drives are (physically) connected to the machine. This is
useful if the storage device order in the BIOS is changed, or if you
switch the storage device cabling. Also, the BIOS may occasionally
change the order of storage devices. Read more about this in the
Persistent block device naming article.

To list basic information about the partitions, run:

    $ lsblk -f

    NAME   FSTYPE LABEL      UUID                                 MOUNTPOINT
    sda                                                           
    ├─sda1 ext4   Arch_Linux 978e3e81-8048-4ae1-8a06-aa727458e8ff /
    ├─sda2 ntfs   Windows    6C1093E61093B594                     
    └─sda3 ext4   Storage    f838b24e-3a66-4d02-86f4-a2e73e454336 /media/Storage
    sdb                                                           
    ├─sdb1 ntfs   Games      9E68F00568EFD9D3                     
    └─sdb2 ext4   Backup     14d50a6c-e083-42f2-b9c4-bc8bae38d274 /media/Backup
    sdc                                                           
    └─sdc1 vfat   Camera     47FA-4071                            /media/Camera

> Kernel name

Run lsblk -f to list the partitions, and prefix them with /dev.

See the example.

> Label

Note:Each label should be unique, to prevent any possible conflicts.

For detailed information on how to label a device or partition, see
Persistent block device naming#by-label. Renaming the root partition has
to be done from a "live" Linux distribution because the partition needs
to be unmounted first.

Run lsblk -f to list the partitions, and prefix them with LABEL= :

    /etc/fstab

    # <file system>        <dir>         <type>    <options>             <dump> <pass> 
    LABEL=Arch_Linux       /             ext4      defaults,noatime      0      1
    LABEL=Arch_Swap        none          swap      defaults              0      0

> UUID

All partitions and devices have a unique UUID. They are generated by
filesystem utilities (e.g. mkfs.*) when you create or format a
partition. See Persistent block device naming#by-uuid for details.

Run lsblk -f to list the partitions, and prefix them with UUID= :

Tip:If you would like to return just the UUID of a specific partition:

    $ lsblk -no UUID /dev/sda2

    /etc/fstab

    # <file system>                            <dir>     <type>    <options>             <dump> <pass>
    UUID=24f28fc6-717e-4bcd-a5f7-32b959024e26  /         ext4      defaults,noatime      0      1
    UUID=03ec5dd3-45c0-4f95-a363-61ff321a09ff  /home     ext4      defaults,noatime      0      2
    UUID=4209c845-f495-4c43-8a03-5363dd433153  none      swap      defaults              0      0

Tips and tricks
---------------

> Automount with systemd

If you have a large /home partition, it might be better to allow
services that do not depend on /home to start while /home is checked by
fsck. This can be achieved by adding the following options to the
/etc/fstab entry of your /home partition:

    noauto,x-systemd.automount

This will fsck and mount /home when it is first accessed, and the kernel
will buffer all file access to /home until it is ready.

Note:This will make your /home filesystem type autofs, which is ignored
by mlocate by default. The speedup of automounting /home may not be more
than a second or two, depending on your system, so this trick may not be
worth it.

The same applies to remote filesystem mounts. If you want them to be
mounted only upon access, you will need to use the
noauto,x-systemd.automount parameters. In addition, you can use the
x-systemd.device-timeout=# option to specify a timeout in case the
network resource is not available.

If you have encrypted filesystems with keyfiles, you can also add the
noauto parameter to the corresponding entries in /etc/crypttab. systemd
will then not open the encrypted device on boot, but instead wait until
it is actually accessed and then automatically open it with the
specified keyfile before mounting it. This might save a few seconds on
boot if you are using an encrypted RAID device for example, because
systemd does not have to wait for the device to become available. For
example:

    /etc/crypttab

    data /dev/md0 /root/key noauto

> Filepath spaces

If any mountpoint contains spaces, use the escape character \ followed
by the 3 digit octal code 040 to emulate them:

    /etc/fstab

    UUID=47FA-4071     /home/username/Camera\040Pictures   vfat  defaults,noatime       0  0
    /dev/sda7          /media/100\040GB\040(Storage)       ext4  defaults,noatime,user  0  2

> External devices

External devices that are to be mounted when present but ignored if
absent may require the nofail option. This prevents errors being
reported at boot.

    /etc/fstab

    /dev/sdg1        /media/backup    jfs    defaults,nofail    0  2

> atime options

The use of noatime, nodiratime or relatime can impact drive performance.

-   The atime option updates the atime of the files every time they are
    accessed. This is more purposeful when Linux is used for servers; it
    does not have much value for desktop use. The drawback about the
    atime option is that even reading a file from the page cache
    (reading from memory instead of the drive) will still result in a
    write!

Using the noatime option fully disables writing file access times to the
drive every time you read a file. This works well for almost all
applications, except for a rare few like Mutt that needs such
information. For mutt, you should only use the relatime option.

The nodiratime option disables the writing of file access times only for
directories while other files still get access times written.

Note:noatime already includes nodiratime. You do not need to specify
both.

-   relatime enables the writing of file access times only when the file
    is being modified (unlike noatime where the file access time will
    never be changed and will be older than the modification time). The
    best compromise might be the use this option since programs like
    Mutt will continue to work, but you will still have a performance
    boost as the files will not get access times updated unless they are
    modified. This option is used when the defaults keyword option or no
    options at all are specified in fstab for a given mount point.

> tmpfs

tmpfs is a temporary filesystem that resides in memory and/or your swap
partition(s), depending on how much you fill it up. Mounting directories
as tmpfs can be an effective way of speeding up accesses to their files,
or to ensure that their contents are automatically cleared upon reboot.

Some directories where tmpfs is commonly used are /tmp, /var/lock and
/var/run. Do NOT use it on /var/tmp, because that folder is meant for
temporary files that are preserved across reboots. Arch uses a tmpfs
/run directory, with /var/run and /var/lock simply existing as symlinks
for compatibility. It is also used for /tmp by the default systemd setup
and does not require an entry in /etc/fstab unless a specific
configuration is needed.

Note:When using systemd, temporary files in tmpfs directories can be
recreated at boot by using tmpfiles.d.

By default, a tmpfs partition has its maximum size set to half your
total RAM, but this can be customized. Note that the actual memory/swap
consumption depends on how much you fill it up, as tmpfs partitions do
not consume any memory until it is actually needed.

To explicitly set a maximum size, in this example to override the
default /tmp mount, use the size mount option:

    /etc/fstab

    tmpfs   /tmp         tmpfs   nodev,nosuid,size=2G          0  0

Here is a more advanced example showing how to add tmpfs mounts for
users. This is useful for websites, mysql tmp files, ~/.vim/, and more.
It's important to try and get the ideal mount options for what you are
trying to accomplish. The goal is to have as secure settings as possible
to prevent abuse. Limiting the size, and specifying uid and gid + mode
is very secure. For more information on this subject, follow the links
listed in the #See also section.

    /etc/fstab

    tmpfs   /www/cache    tmpfs  rw,size=1G,nr_inodes=5k,noexec,nodev,nosuid,uid=648,gid=648,mode=1700   0  0

See the mount command man page for more information. One useful mount
option in the man page is the default option. At least understand that.

Reboot for the changes to take effect. Note that although it may be
tempting to simply run mount -a to make the changes effective
immediately, this will make any files currently residing in these
directories inaccessible (this is especially problematic for running
programs with lockfiles, for example). However, if all of them are
empty, it should be safe to run mount -a instead of rebooting (or mount
them individually).

After applying changes, you may want to verify that they took effect by
looking at /proc/mounts and using findmnt:

    $ findmnt --target /tmp

    TARGET SOURCE FSTYPE OPTIONS
    /tmp   tmpfs  tmpfs  rw,nosuid,nodev,relatime

Usage

Generally, I/O intensive tasks and programs that run frequent read/write
operations can benefit from using a tmpfs folder. Some applications can
even receive a substantial gain by offloading some (or all) of their
data onto the shared memory. For example, relocating the Firefox profile
into RAM shows a significant improvement in performance.

Improving compile times

Compiling requires handling of many small files and involves many I/O
operations; therefore it is a prime activity to benefit from moving its
working directory to a #tmpfs.

For one session

The BUILDDIR value may be exported within a shell to temporarily set
makepkg build directory to an existing tmpfs:

    $ BUILDDIR=/tmp/makepkg makepkg

Permanently

Just uncomment the BUILDDIR line in /etc/makepkg.conf, see
Makepkg#Improving compile times for details.

> Writing to FAT32 as Normal User

  ------------------------ ------------------------ ------------------------
  [Tango-two-arrows.png]   This article or section  [Tango-two-arrows.png]
                           is a candidate for       
                           merging with USB Storage 
                           Devices#Mounting USB     
                           memory.                  
                           Notes: The linked        
                           section assumes that the 
                           partition on USB storage 
                           uses FAT32 or NTFS       
                           filesystem, so we have   
                           two sections covering    
                           the same topic. Either   
                           merge everything here or 
                           in the linked section.   
                           (Discuss)                
  ------------------------ ------------------------ ------------------------

To write on a FAT32 partition, you must make a few changes to your
/etc/fstab file.

    /etc/fstab

    /dev/sdxY    /mnt/some_folder  vfat   user,rw,umask=000              0  0

The user flag means that any user (even non-root) can mount and unmount
the partition /dev/sdX. rw gives read-write access; umask option removes
selected rights - for example umask=111 remove executable rights. The
problem is that this entry removes executable rights from directories
too, so we must correct it by dmask=000. See also Umask.

Without these options, all files will be executable. You can use the
option showexec instead of the umask and dmask options, which shows all
Windows executables (com, exe, bat) in executable colours.

For example, if your FAT32 partition is on /dev/sda9, and you wish to
mount it to /mnt/fat32, then you would use:

    /etc/fstab

    /dev/sda9    /mnt/fat32        vfat   user,rw,umask=111,dmask=000    0  0

> Remounting the root partition

If for some reason the root partition has been improperly mounted read
only, remount the root partition with read-write access with the
following command:

    # mount -o remount,rw /

See also
--------

-   Full device listing including block device
-   Filesystem Hierarchy Standard
-   30x Faster Web-Site Speed (Detailed tmpfs)
-   Adding Samba shares to /etc/fstab

Retrieved from
"https://wiki.archlinux.org/index.php?title=Fstab&oldid=296407"

Categories:

-   File systems
-   Boot process

-   This page was last modified on 6 February 2014, at 18:45.
-   Content is available under GNU Free Documentation License 1.3 or
    later unless otherwise noted.
-   Privacy policy
-   About ArchWiki
-   Disclaimers
