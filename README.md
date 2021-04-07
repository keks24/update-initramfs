# Introduction
`update-initramfs` updates your initramfs using `bash`

# Prerequisites
* Kernel source files in `/usr/src/linux/`, where `linux` is a symlink to the directory of the kernel which is currently in use
* Enabled kernel settings to work with an initramfs. Consult [this page](https://wiki.gentoo.org/wiki/Custom_Initramfs) for more information.
* The following packages are installed:
```no-highlight
gawk
chmod
cp
cpio
date
find
grep
gzip
ldd
make
md5sum
mkdir
rm
sh
tee
touch
tree
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
$ vi "/usr/local/etc/update_initramfs.conf"
[...]
bin_file_array=("busybox")
dev_file_array=("console" "null" "random" "tty" "urandom")
etc_file_array=()
init_file_array=("init")
lib_file_array=()
# the symlink "/lib64/libgcc_s.so.1" to "/usr/lib/gcc/$(gcc_path=$(gcc-config --get-current-profile); echo ${gcc_path%-*}/${gcc_path##*-})/libgcc_s.so.1" is required in order to use
# "cryptsetup" with the new luks2 format ("libargon2", "pthread_cancel")
# "zfs" on "rootfs" (https://github.com/openzfs/zfs/issues/4749)
lib64_file_array=("libgcc_s.so.1")
proc_file_array=()
root_file_array=()
sbin_file_array=("cryptsetup")
sys_file_array=()
usr_file_array=("share/consolefonts/ter-112n.psf.gz" "lib/gcc/$(gcc_path=$(gcc-config --get-current-profile); echo ${gcc_path%-*}/${gcc_path##*-})/libgcc_s.so.1")
```

The script analyses the defined arrays and determines, if a binary file was `statically` or `dynamically` compiled using `ldd`. It then copies all related files from the system (e.g. `/bin/busybox`) to `/usr/src/initramfs/` and uses a combination of `find`, `cpio` and `gzip` to build the initramfs in `/usr/src/custom-initramfs-$(/usr/bin/uname --kernel-release).cpio.gz`.

Custom scripts, like the `init` script, are placed in `/usr/local/etc/update_initramfs/init_files`.

3. Make sure, that the content of the configuration file `/usr/local/etc/update_initramfs.conf` is properly configured and update its `md5 sum` in the script:
```bash
$ md5sum "/usr/local/etc/update_initramfs/update_initramfs.conf"
f2f93af83839fe9aebdfa8965fe52cea  /usr/local/etc/update_initramfs/update_initramfs.conf
$ vi "/usr/local/sbin/update_initramfs"
[...]
config_file_md5_sum="f2f93af83839fe9aebdfa8965fe52cea"
[...]
```

4. Execute `update_initramfs` in order to build the `initramfs`:
```bash
$ update_initramfs

-------------------------------------
-Copied/Updated the following files:-
-------------------------------------


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

Copied the following files for /lib64/libgcc_s.so.1:
'/lib64/libgcc_s.so.1' -> '/usr/src/initramfs/lib64/libgcc_s.so.1'

Copied the following files for /sbin/cryptsetup:
'/usr/lib64/libcryptsetup.so.12' -> '/usr/src/initramfs/usr/lib64/libcryptsetup.so.12'
'/usr/lib64/libpopt.so.0' -> '/usr/src/initramfs/usr/lib64/libpopt.so.0'
'/lib64/libuuid.so.1' -> '/usr/src/initramfs/lib64/libuuid.so.1'
'/lib64/libblkid.so.1' -> '/usr/src/initramfs/lib64/libblkid.so.1'
'/lib64/libc.so.6' -> '/usr/src/initramfs/lib64/libc.so.6'
'/lib64/libdevmapper.so.1.02' -> '/usr/src/initramfs/lib64/libdevmapper.so.1.02'
'/usr/lib64/libcrypto.so.1.1' -> '/usr/src/initramfs/usr/lib64/libcrypto.so.1.1'
'/usr/lib64/libargon2.so.1' -> '/usr/src/initramfs/usr/lib64/libargon2.so.1'
'/usr/lib64/libjson-c.so.5' -> '/usr/src/initramfs/usr/lib64/libjson-c.so.5'
'/lib64/ld-linux-x86-64.so.2' -> '/usr/src/initramfs/lib64/ld-linux-x86-64.so.2'
'/lib64/libudev.so.1' -> '/usr/src/initramfs/lib64/libudev.so.1'
'/lib64/libpthread.so.0' -> '/usr/src/initramfs/lib64/libpthread.so.0'
'/lib64/libm.so.6' -> '/usr/src/initramfs/lib64/libm.so.6'
'/lib64/libz.so.1' -> '/usr/src/initramfs/lib64/libz.so.1'
'/lib64/libdl.so.2' -> '/usr/src/initramfs/lib64/libdl.so.2'
'/sbin/cryptsetup' -> '/usr/src/initramfs/sbin/cryptsetup'

Copied the following files for /usr/share/consolefonts/ter-112n.psf.gz:
/usr/share -> /usr/src/initramfs/usr/share
/usr/share/consolefonts -> /usr/src/initramfs/usr/share/consolefonts
'/usr/share/consolefonts/ter-112n.psf.gz' -> '/usr/src/initramfs/usr/share/consolefonts/ter-112n.psf.gz'

Copied the following files for /usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1:
/usr/lib/gcc -> /usr/src/initramfs/usr/lib/gcc
/usr/lib/gcc/x86_64-pc-linux-gnu -> /usr/src/initramfs/usr/lib/gcc/x86_64-pc-linux-gnu
/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0 -> /usr/src/initramfs/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0
'/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1' -> '/usr/src/initramfs/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1'

-------------------------------------
-Current structure of the initramfs:-
-------------------------------------

/usr/src/initramfs
├── bin/
│   └── busybox*
├── boot/
├── dev/
│   ├── console
│   ├── null
│   ├── random
│   ├── tty
│   └── urandom
├── etc/
├── home/
├── init*
├── lib/
├── lib64/
│   ├── ld-linux-x86-64.so.2*
│   ├── libblkid.so.1*
│   ├── libc.so.6*
│   ├── libdevmapper.so.1.02*
│   ├── libdl.so.2*
│   ├── libgcc_s.so.1 -> ../usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1
│   ├── libm.so.6*
│   ├── libpthread.so.0*
│   ├── libudev.so.1*
│   ├── libuuid.so.1*
│   └── libz.so.1*
├── mnt/
│   └── root/
├── proc/
├── root/
├── run/
│   └── cryptsetup/
├── sbin/
│   └── cryptsetup*
├── sys/
└── usr/
    ├── lib/
    │   └── gcc/
    │       └── x86_64-pc-linux-gnu/
    │           └── 10.2.0/
    │               └── libgcc_s.so.1
    ├── lib64/
    │   ├── libargon2.so.1*
    │   ├── libcrypto.so.1.1*
    │   ├── libcryptsetup.so.12*
    │   ├── libjson-c.so.5*
    │   └── libpopt.so.0*
    └── share/
        └── consolefonts/
            └── ter-112n.psf.gz

-------------------------------------
----Generated initramfs location:----
-------------------------------------

/usr/src/custom-initramfs-5.10.27-gentoo.cpio.gz

Next steps:
1. Move the file '/usr/src/custom-initramfs-5.10.27-gentoo.cpio.gz' to '/boot/'
2. Update your bootloader entries
   When using GRUB, you may refer to: https://wiki.gentoo.org/wiki/Custom_Initramfs#Using_GRUB
```

Content of the log file `/var/log/update_initramfs/update_initramfs.log`:
```no-highlight
-------------
20210405-2036
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

Copied the following files for /lib64/libgcc_s.so.1:
'/lib64/libgcc_s.so.1' -> '/usr/src/initramfs/lib64/libgcc_s.so.1'

Copied the following files for /sbin/cryptsetup:
'/usr/lib64/libcryptsetup.so.12' -> '/usr/src/initramfs/usr/lib64/libcryptsetup.so.12'
'/usr/lib64/libpopt.so.0' -> '/usr/src/initramfs/usr/lib64/libpopt.so.0'
'/lib64/libuuid.so.1' -> '/usr/src/initramfs/lib64/libuuid.so.1'
'/lib64/libblkid.so.1' -> '/usr/src/initramfs/lib64/libblkid.so.1'
'/lib64/libc.so.6' -> '/usr/src/initramfs/lib64/libc.so.6'
'/lib64/libdevmapper.so.1.02' -> '/usr/src/initramfs/lib64/libdevmapper.so.1.02'
'/usr/lib64/libcrypto.so.1.1' -> '/usr/src/initramfs/usr/lib64/libcrypto.so.1.1'
'/usr/lib64/libargon2.so.1' -> '/usr/src/initramfs/usr/lib64/libargon2.so.1'
'/usr/lib64/libjson-c.so.5' -> '/usr/src/initramfs/usr/lib64/libjson-c.so.5'
'/lib64/ld-linux-x86-64.so.2' -> '/usr/src/initramfs/lib64/ld-linux-x86-64.so.2'
'/lib64/libudev.so.1' -> '/usr/src/initramfs/lib64/libudev.so.1'
'/lib64/libpthread.so.0' -> '/usr/src/initramfs/lib64/libpthread.so.0'
'/lib64/libm.so.6' -> '/usr/src/initramfs/lib64/libm.so.6'
'/lib64/libz.so.1' -> '/usr/src/initramfs/lib64/libz.so.1'
'/lib64/libdl.so.2' -> '/usr/src/initramfs/lib64/libdl.so.2'
'/sbin/cryptsetup' -> '/usr/src/initramfs/sbin/cryptsetup'

Copied the following files for /usr/share/consolefonts/ter-112n.psf.gz:
/usr/share -> /usr/src/initramfs/usr/share
/usr/share/consolefonts -> /usr/src/initramfs/usr/share/consolefonts
'/usr/share/consolefonts/ter-112n.psf.gz' -> '/usr/src/initramfs/usr/share/consolefonts/ter-112n.psf.gz'

Copied the following files for /usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1:
/usr/lib/gcc -> /usr/src/initramfs/usr/lib/gcc
/usr/lib/gcc/x86_64-pc-linux-gnu -> /usr/src/initramfs/usr/lib/gcc/x86_64-pc-linux-gnu
/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0 -> /usr/src/initramfs/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0
'/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1' -> '/usr/src/initramfs/usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1'

Current structure of the initramfs:
/usr/src/initramfs
├── bin/
│   └── busybox*
├── boot/
├── dev/
│   ├── console
│   ├── null
│   ├── random
│   ├── tty
│   └── urandom
├── etc/
├── home/
├── init*
├── lib/
├── lib64/
│   ├── ld-linux-x86-64.so.2*
│   ├── libblkid.so.1*
│   ├── libc.so.6*
│   ├── libdevmapper.so.1.02*
│   ├── libdl.so.2*
│   ├── libgcc_s.so.1 -> ../usr/lib/gcc/x86_64-pc-linux-gnu/10.2.0/libgcc_s.so.1
│   ├── libm.so.6*
│   ├── libpthread.so.0*
│   ├── libudev.so.1*
│   ├── libuuid.so.1*
│   └── libz.so.1*
├── mnt/
│   └── root/
├── proc/
├── root/
├── run/
│   └── cryptsetup/
├── sbin/
│   └── cryptsetup*
├── sys/
└── usr/
    ├── lib/
    │   └── gcc/
    │       └── x86_64-pc-linux-gnu/
    │           └── 10.2.0/
    │               └── libgcc_s.so.1
    ├── lib64/
    │   ├── libargon2.so.1*
    │   ├── libcrypto.so.1.1*
    │   ├── libcryptsetup.so.12*
    │   ├── libjson-c.so.5*
    │   └── libpopt.so.0*
    └── share/
        └── consolefonts/
            └── ter-112n.psf.gz

Generated the following initramfs file:
/usr/src/custom-initramfs-5.10.27-gentoo.cpio.gz
```
