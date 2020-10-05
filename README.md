# Introduction
`update-initramfs` updates your initramfs using bash and generates it via provided kernel scripts.

# Prerequisites
* Kernel source files in `/usr/src/linux/`, where `linux` is a symlink to the directory of the kernel which is currently in use
* Enabled kernel settings to work with an initramfs. Consult [this page](https://wiki.gentoo.org/wiki/Custom_Initramfs) for more information.
* The following packages are installed:
```no-highlight
awk
chmod
cp
date
echo
grep
ldd
logrotate
make
md5sum
mkdir
rm
sh
tee
touch
tree
uname
```

* The path `/usr/local/sbin/` exists in the `${PATH}` variable:
```bash
$ echo "${PATH//:/\n}"
```
```
/home/ramon/bin
/usr/local/bin
/usr/bin
/bin
/usr/local/sbin
/usr/sbin
/sbin
/usr/local/games
/usr/games
/usr/lib/llvm/6/bin
/opt/bin
```

# Installation
Clone the repository into your current working directory:
```bash
$ git clone "https://codeberg.org/keks24/update-initramfs.git"
```
Copy all necessary files:
```bash
$ cd "update-initramfs/"
$ cp --recursive "usr/local/etc/update_initramfs/" "/usr/local/etc/"
$ cp "usr/local/sbin/update_initramfs" "/usr/local/sbin/"
$ cp "etc/logrotate.d/update_initramfs" "/etc/logrotate.d/"
$ chown --recursive root:root "/usr/local/etc/update_initramfs/" "/usr/local/sbin/update_initramfs" "/etc/logrotate.d/update_initramfs"
```

# Usage
1. Make sure you have set the appropriate kernel settings. Consult [this page](https://wiki.gentoo.org/wiki/Custom_Initramfs) for more information.
2. Enter the needed `binaries` and `shared objects` in the `update_initramfs.conf`:
```bash
[...]
bin_file_array=("busybox")
dev_file_array=("console" "null" "random" "tty" "urandom")
etc_file_array=()
init_file_array=("init")
lib_file_array=()
lib64_file_array=()
proc_file_array=()
root_file_array=()
sbin_file_array=("cryptsetup" "fsck")
sys_file_array=()
usr_file_array=()
```

The script analyses the defined arrays and determines, if a binary file was statically or dynamically compiled using `ldd`. It then copies all related files from your system (e.g. `/bin/busybox`) to `/usr/src/initramfs/` and uses `/usr/src/linux/scripts/gen_initramfs_list.sh` and `/usr/src/linux/usr/gen_init_cpio`, which are provided by the kernel source files, to build the initramfs in `/usr/src/custom-initramfs-$(/usr/bin/uname --kernel-release).cpio.gz`.

Custom scripts, like the `init` script, reside in `/usr/local/etc/update_initramfs/init_files`.

3. Check the content of the configuration file and update its md5 sum in the script:
```bash
$ md5sum "/usr/local/etc/update_initramfs/update_initramfs.conf"
```
```no-highlight
074f88a3f572df072b35d2e2cbd83d15  /usr/local/etc/update_initramfs/update_initramfs.conf
```
```bash
$ vi "/usr/local/sbin/update_initramfs"
```
```bash
[...]
config_file_md5_sum="074f88a3f572df072b35d2e2cbd83d15"
[...]
```

4. Execute `update_initramfs`:
```bash
$ update_initramfs
```
```no-highlight

-------------------------------------
-Copied/Updated the following files:-
-------------------------------------

'/bin/busybox' -> '/usr/src/initramfs/bin/busybox'
'/dev/console' -> '/usr/src/initramfs/dev/console'
'/dev/null' -> '/usr/src/initramfs/dev/null'
'/dev/random' -> '/usr/src/initramfs/dev/random'
'/dev/tty' -> '/usr/src/initramfs/dev/tty'
'/dev/urandom' -> '/usr/src/initramfs/dev/urandom'
'/usr/local/etc/update_initramfs/init_files/init' -> '/usr/src/initramfs/init'
mode of '/usr/src/initramfs/init' changed from 0644 (rw-r--r--) to 0744 (rwxr--r--)
'/usr/lib64/libcryptsetup.so.4' -> '/usr/src/initramfs/usr/lib64/libcryptsetup.so.4'
'/usr/lib64/libpopt.so.0' -> '/usr/src/initramfs/usr/lib64/libpopt.so.0'
'/lib64/libc.so.6' -> '/usr/src/initramfs/lib64/libc.so.6'
'/lib64/libuuid.so.1' -> '/usr/src/initramfs/lib64/libuuid.so.1'
'/lib64/libdevmapper.so.1.02' -> '/usr/src/initramfs/lib64/libdevmapper.so.1.02'
'/usr/lib64/libgcrypt.so.20' -> '/usr/src/initramfs/usr/lib64/libgcrypt.so.20'
'/usr/lib64/libgpg-error.so.0' -> '/usr/src/initramfs/usr/lib64/libgpg-error.so.0'
'/lib64/ld-linux-x86-64.so.2' -> '/usr/src/initramfs/lib64/ld-linux-x86-64.so.2'
'/lib64/librt.so.1' -> '/usr/src/initramfs/lib64/librt.so.1'
'/lib64/libudev.so.1' -> '/usr/src/initramfs/lib64/libudev.so.1'
'/lib64/libpthread.so.0' -> '/usr/src/initramfs/lib64/libpthread.so.0'
'/lib64/libm.so.6' -> '/usr/src/initramfs/lib64/libm.so.6'
'/sbin/cryptsetup' -> '/usr/src/initramfs/sbin/cryptsetup'
'/lib64/libmount.so.1' -> '/usr/src/initramfs/lib64/libmount.so.1'
'/lib64/libblkid.so.1' -> '/usr/src/initramfs/lib64/libblkid.so.1'
'/sbin/fsck' -> '/usr/src/initramfs/sbin/fsck'

-------------------------------------
-Current structure of the initramfs:-
-------------------------------------

/usr/src/initramfs
├── bin/
│   └── busybox*
├── dev/
│   ├── console
│   ├── null
│   ├── random
│   ├── tty
│   └── urandom
├── etc/
├── init*
├── lib/
├── lib64/
│   ├── ld-linux-x86-64.so.2*
│   ├── libblkid.so.1*
│   ├── libc.so.6*
│   ├── libdevmapper.so.1.02*
│   ├── libmount.so.1*
│   ├── libm.so.6*
│   ├── libpthread.so.0*
│   ├── librt.so.1*
│   ├── libudev.so.1*
│   └── libuuid.so.1*
├── mnt/
│   └── root/
├── proc/
├── root/
├── sbin/
│   ├── cryptsetup*
│   └── fsck*
├── sys/
└── usr/
    ├── lib/
    └── lib64/
        ├── libcryptsetup.so.4*
        ├── libgcrypt.so.20*
        ├── libgpg-error.so.0*
        └── libpopt.so.0*

-------------------------------------
----Generated initramfs location:----
-------------------------------------

/usr/src/custom-initramfs-4.14.83-gentoo.cpio.gz

Next steps:
1. Move the file 'custom-initramfs-4.14.83-gentoo.cpio.gz' to '/boot/'
2. Update your bootloader entries
   When using GRUB, you may refer to: https://wiki.gentoo.org/wiki/Custom_Initramfs#Using_GRUB
```

Content of `/var/log/update_initramfs/update_initramfs.log`:
```no-highlight
-------------
20181215-2055
-------------

        Copied the following files for /bin/busybox:
'/bin/busybox' -> '/usr/src/initramfs/bin/busybox'
        Copied the following files for /dev/console:
'/dev/console' -> '/usr/src/initramfs/dev/console'
        Copied the following files for /dev/null:
'/dev/null' -> '/usr/src/initramfs/dev/null'
        Copied the following files for /dev/random:
'/dev/random' -> '/usr/src/initramfs/dev/random'
        Copied the following files for /dev/tty:
'/dev/tty' -> '/usr/src/initramfs/dev/tty'
        Copied the following files for /dev/urandom:
'/dev/urandom' -> '/usr/src/initramfs/dev/urandom'
        Copied the following files for /usr/local/etc/update_initramfs/init_files/init:
'/usr/local/etc/update_initramfs/init_files/init' -> '/usr/src/initramfs/init'
mode of '/usr/src/initramfs/init' changed from 0644 (rw-r--r--) to 0744 (rwxr--r--)
        Copied the following files for /sbin/cryptsetup:
'/usr/lib64/libcryptsetup.so.4' -> '/usr/src/initramfs/usr/lib64/libcryptsetup.so.4'
'/usr/lib64/libpopt.so.0' -> '/usr/src/initramfs/usr/lib64/libpopt.so.0'
'/lib64/libc.so.6' -> '/usr/src/initramfs/lib64/libc.so.6'
'/lib64/libuuid.so.1' -> '/usr/src/initramfs/lib64/libuuid.so.1'
'/lib64/libdevmapper.so.1.02' -> '/usr/src/initramfs/lib64/libdevmapper.so.1.02'
'/usr/lib64/libgcrypt.so.20' -> '/usr/src/initramfs/usr/lib64/libgcrypt.so.20'
'/usr/lib64/libgpg-error.so.0' -> '/usr/src/initramfs/usr/lib64/libgpg-error.so.0'
'/lib64/ld-linux-x86-64.so.2' -> '/usr/src/initramfs/lib64/ld-linux-x86-64.so.2'
'/lib64/librt.so.1' -> '/usr/src/initramfs/lib64/librt.so.1'
'/lib64/libudev.so.1' -> '/usr/src/initramfs/lib64/libudev.so.1'
'/lib64/libpthread.so.0' -> '/usr/src/initramfs/lib64/libpthread.so.0'
'/lib64/libm.so.6' -> '/usr/src/initramfs/lib64/libm.so.6'
'/sbin/cryptsetup' -> '/usr/src/initramfs/sbin/cryptsetup'
        Copied the following files for /sbin/fsck:
'/lib64/libmount.so.1' -> '/usr/src/initramfs/lib64/libmount.so.1'
'/lib64/libblkid.so.1' -> '/usr/src/initramfs/lib64/libblkid.so.1'
'/sbin/fsck' -> '/usr/src/initramfs/sbin/fsck'

        Current structure of the initramfs:
/usr/src/initramfs
├── bin/
│   └── busybox*
├── dev/
│   ├── console
│   ├── null
│   ├── random
│   ├── tty
│   └── urandom
├── etc/
├── init*
├── lib/
├── lib64/
│   ├── ld-linux-x86-64.so.2*
│   ├── libblkid.so.1*
│   ├── libc.so.6*
│   ├── libdevmapper.so.1.02*
│   ├── libmount.so.1*
│   ├── libm.so.6*
│   ├── libpthread.so.0*
│   ├── librt.so.1*
│   ├── libudev.so.1*
│   └── libuuid.so.1*
├── mnt/
│   └── root/
├── proc/
├── root/
├── sbin/
│   ├── cryptsetup*
│   └── fsck*
├── sys/
└── usr/
    ├── lib/
    └── lib64/
        ├── libcryptsetup.so.4*
        ├── libgcrypt.so.20*
        ├── libgpg-error.so.0*
        └── libpopt.so.0*

        Generated the following initramfs file:
/usr/src/custom-initramfs-4.14.83-gentoo.cpio.gz
```
