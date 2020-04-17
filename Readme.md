# ft_linux
A simple LFS...

## Prerequisites

For this subject, i'll use a virtual machine (virtualbox) with 10 GB of space and bridged network (for ssh).
I also allowed 2048 MB of ram and 2 CPU.

I decided to install debian as host OS.

During installation of Debian, i decided to activate the ssh, to be able to connect to it more easily from my terminal.

I also set up 4 partitions, as the subject requires.

This is the list of my partitions:

- ext2 partition (100 mb) <- will be /boot
- swap partition (1 gb)   <- will be swap file
- ext4 partition (16.9 gb) <- root partition
- ext4 partition (2 gb)   <- host os partition

The total is 25gb

## Settings things up on the host

### Installing dependencies

Once Debian was finally installed and running, i could install the packages required to build LFS,
following this page:

http://www.linuxfromscratch.org/lfs/view/stable/chapter02/hostreqs.html

```
su
apt update && apt upgrade
apt install binutils bison bzip2 coreutils diffutils findutils gawk gcc grep m4 make patch perl python sed tar texinfo g++ libncurses5-dev
```

Many of these packages are already installed on debian, but some are missing.

### Mounting the partitions

We are gonna mount our disks into the host to be able to write inside them

we can see our disks by doing: ```fdisk -l``` as root.

In my case,

- /dev/sda5 is my swap
- /dev/sda6 is my /boot
- /dev/sda7 is my root

Now that we know our partitions, we can mount them:

```
su
mkdir /mnt/boot
mkdir /mnt/root
mount /dev/sda6 /mnt/boot
mount /dev/sda7 /mnt/root
```

we can now create env variable to easily remember those path
/!\ If you do the installation is more than one time, or reboot your computer, you will have to re set those variables /!\

```
export boot="/mnt/boot"
export root="/mnt/root"
```

## Downloading the sources

Now that everything is set up, we can begin to get the sources

First, let's create the folder for centralizing the sources:

```
mkdir $root/sources
chmod -v a+wt $root/sources

```

Then get the list of files to download

```
cd $root/sources
wget http://www.linuxfromscratch.org/lfs/view/stable/wget-list
wget http://www.linuxfromscratch.org/lfs/view/stable/md5sums
```

Download it all:

```
wget --input-file=wget-list --continue --directory-prefix=$root/sources
```

Then verify the integrity of the packages:

```
pushd $root/sources
md5sum -c md5sums
popd
```

If during the verification, one or multiple packages failed during verification, you can manually try to redownload them from this page:

http://www.linuxfromscratch.org/lfs/view/stable/chapter03/packages.html

and relaunch verification.

## Install temporary system and tools

We begin by creating a folder called 'tools'. It will contain all tools needed to build LFS
We also create a symbolic link at root, for easier usage

```
mkdir -v $root/tools
ln -sv $root/tools /
```

### Setting good work environnement

In order to build everything, we need a user that does not have root access, to be sure not to broke anything.  

```
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs
chown -v lfs $root/tools
chown -v lfs $root/sources
su - lfs
```

~/.bash_profile will contain informations about the user and the bash settings for this user.
In the code below, we make sure that each time the shell start with 'lfs' user, we empty the environnement, to create a fresh env

```
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

~/.bash_rc contain informations about the configuration of out shell.
In the code below, we set right for access folders

```
cat > ~/.bashrc << "EOF"
set +h
umask 022
boot=/mnt/boot
root=/mnt/root
LFS=$root
LC_ALL=POSIX
alias ..='cd ../'
alias ex='tar -xvf'
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/tools/bin:/bin:/usr/bin
export LFS boot root LC_ALL LFS_TGT PATH
EOF
```

We export MAKEFLAGS in order to multithread compilation, we also add it to .bashrc

```
export MAKEFLAGS='-j 2'
echo "export MAKEFLAGS='-2 '" >> ~/.bashrc
```

Finally, we reload bash_profile

```
source ~/.bash_profile
```

### Building the tools

We go into the sources folder

```
cd $root/sources
```

#### Binutils - Pass 1

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/binutils-pass1.html

#### Gcc

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-pass1.html

#### Linux Api

```
make mrproper
make menuconfig
```

A menu should appear. Go into:
General setup -> Local version - append to kernel release
You can now type the name of your kernel.
I chose "Linux kernel 5.5.3 lubenard".
Save, then exit.

```
make headers
cp -rv usr/include/* /tools/include
```

#### Glibc

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/glibc.html

#### Gcc - libstdc++

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-libstdc++.html

#### Binutils - Pass 2

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/binutils-pass2.html

#### Gcc - Pass 2

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-pass2.html

#### Tcl

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/tcl.html

#### Expect

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/expect.html


#### Dejagnu

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/dejagnu.html

#### M4

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/m4.html

#### Ncurses

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/ncurses.html

#### Bash

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/bash.html

#### Bison

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/bison.html

#### Bzip2

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/bzip2.html

#### Coreutils

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/coreutils.html

#### Diffutils

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/diffutils.html

#### File

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/file.html

#### findutils

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/findutils.html

#### Gawk

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gawk.html

#### Grep

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/grep.html

#### Gzip

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gzip.html

#### Make

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/make.html

#### Patch

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/patch.html

#### Perl

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/perl.html

#### Python

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/Python.html

#### Sed

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/sed.html

#### Tar

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/tar.html

#### Texinfo

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/texinfo.html

#### Xz

http://www.linuxfromscratch.org/lfs/view/stable/chapter05/xz.html


### Changing right on tools folder

```
chown -R root:root $LFS/tools
```

## Let's build the system !

We create basic first folders

```
mkdir -pv $LFS/{dev,proc,sys,run}
```

We also create /dev/null and /dev/zero

```
mknod -m 600 $LFS/dev/console c 5 1
mknod -m 666 $LFS/dev/null c 1 3
```

We mount /dev

```
mount -v --bind /dev $LFS/dev
```

And mount other folders

```
mount -vt devpts devpts $LFS/dev/pts -o gid=5,mode=620
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```


```
if [ -h $LFS/dev/shm ]; then
  mkdir -pv $LFS/$(readlink $LFS/dev/shm)
fi
```

### I am chroot !

To transition from the host to the LFS, we use chroot

```
chroot "$LFS" /tools/bin/env -i \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
    /tools/bin/bash --login +h
```

We make basic Linux system folders at root of LFS partition

```
mkdir -pv /{bin,etc/{opt,sysconfig},home,lib/firmware,mnt,opt}
mkdir -pv /{media/{floppy,cdrom},sbin,srv,var}
install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
mkdir -pv /usr/{,local/}{bin,include,lib,sbin,src}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -v  /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -v  /usr/libexec
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -v  /usr/lib/pkgconfig

case $(uname -m) in
 x86_64) mkdir -v /lib64 ;;
esac

mkdir -v /var/{log,mail,spool}
ln -sv /run /var/run
ln -sv /run/lock /var/lock
mkdir -pv /var/{opt,cache,lib/{color,misc,locate},local}
```

Then create symbolic links

```
ln -sv /tools/bin/{bash,cat,chmod,dd,echo,ln,mkdir,pwd,rm,stty,touch} /bin
ln -sv /tools/bin/{env,install,perl,printf}         /usr/bin
ln -sv /tools/lib/libgcc_s.so{,.1}                  /usr/lib
ln -sv /tools/lib/libstdc++.{a,so{,.6}}             /usr/lib

ln -sv bash /bin/sh
```

```
ln -sv /proc/self/mounts /etc/mtab
```

We set basic users, such as root....

```
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/bin/false
daemon:x:6:6:Daemon User:/dev/null:/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/var/run/dbus:/bin/false
nobody:x:99:99:Unprivileged User:/dev/null:/bin/false
EOF
```

...and basic groups

```
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
usb:x:14:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
wheel:x:97:
nogroup:x:99:
users:x:999:
EOF
```

```
exec /tools/bin/bash --login +h
```

```
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664  /var/log/lastlog
chmod -v 600  /var/log/btmp
```
