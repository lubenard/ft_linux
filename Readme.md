# ft_linux
A simple LFS...

[Subject](ft_linux.en.pdf)

![ft_linux](https://user-images.githubusercontent.com/42534397/113270889-3be01880-92da-11eb-9f8d-e845e7f88b07.png)

/!\ Be careful ! This lfs might have broken links due to lfs updating their tutorial.

/!\ This LFS has been done with all links working !

/!\ If some links are broken, that might be a good idea to compare with the website to avoid errors

## Prerequisites

For this subject, i'll use a virtual machine (virtualbox) with 32 GB of space and bridged network (for ssh).
I also allowed 4096 MB of ram and 2 CPU.

I decided to install debian as host OS.

During installation of Debian, i decided to activate the ssh, to be able to connect to it more easily from my terminal.

I also set up 4 partitions, as the subject requires.

This is the list of my partitions:

- ext2 partition (100 mb) <- will be /boot
- swap partition (1 gb)   <- will be swap file
- ext4 partition (25 gb) <- root partition
- ext4 partition (5 gb)   <- host os partition

The total is ~31gb

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

/!\ If you do the installation is more than one time, or reboot your computer, you will have to set those variables 
again /!\

```
export boot="/mnt/boot"
export root="/mnt/root"
export LFS="/mnt/root"
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
wget https://ftp.gnu.org/gnu/wget/wget-1.20.3.tar.gz
wget http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.2p1.tar.gz
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
export MAKEFLAGS='-j 8'
echo "export MAKEFLAGS='-8 '" >> ~/.bashrc
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

Binutils - Pass 1 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/binutils-pass1.html)

Gcc - Pass 1 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-pass1.html)

Linux Api Headers [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/linux-headers.html)

Glibc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/glibc.html)

Gcc - libstdc++ - Pass 1 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-libstdc++.html)


M4 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/m4.html)

Ncurses [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/ncurses.html)

Bash [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/bash.html)

Coreutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/coreutils.html)

Diffutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/diffutils.html)

File [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/file.html)

Findutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/findutils.html)

Gawk [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/gawk.html)

Grep [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/grep.html)

Gzip [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/gzip.html)

Make [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/make.html)

Patch [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/patch.html)

Sed [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/sed.html)

Tar [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/tar.html)

Binutils - Pass 2 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/binutils-pass2.html)

Gcc - Pass 2 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/gcc-pass2.html)

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

If ```chroot``` is not found, check the PATH contains ```/usr/sbin```
If ```chroot``` is not in this folder, you can check with ```whereis chroot```

### Let's simplify that

If you are planning to do this LFS in multiple times,
we can create a small script that we will launch each time to simplify

```
nano startup.sh
```

Then enter:

```
#!/bin/bash
su
mount /dev/sda6 /mnt/boot
mount /dev/sda7 /mnt/root
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts $LFS/dev/pts -o gid=5,mode=620
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```

Make it executable:

```
chmod +x startup.sh
```

This way, when you will relaunch your VM, juste type:

```
./startup.sh
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

Initialise log files:

```
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664  /var/log/lastlog
chmod -v 600  /var/log/btmp
```

### Install packages

If you are short in space during the installation of thoses packages, you can free some space by deleting the folder of packages you have already installed:

(This will not clean the archives files)

```
cd /sources
rm -rf $(ls -d */)
```

Man pages [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/man-pages.html)

Linux API Headers [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/linux-headers.html)

Iana-etc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/iana-etc.html)

Glibc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/glibc.html)

Zlib [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/zlib.html)

Bzip2 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/bzip2.html)

Xz [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/xz.html)

Zstd [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/zstd.html)

File [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/file.html)

Readline [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/readline.html)

M4 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/m4.html)

Bc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/bc.html)

Flex [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/flex.html)

Tcl [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/tcl.html)

Expect [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/expect.html)

DejaGNU [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/dejagnu.html)

Binutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/binutils.html)

Gmp [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gmp.html)

Mpfr [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/mpfr.html)

Mpc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/mpc.html)

Attr [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/attr.html)

Acl [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/acl.html)

Libcap [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/libcap.html)

Shadow [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/shadow.html)

Gcc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gcc.html)

If during this test:

```
echo 'int main(){}' > dummy.c
cc dummy.c -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
```

your output is different than:

```
[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

try launching and see if it works 

```
gcc -dumpspecs | sed -e 's@/tools@@g' > `dirname $(gcc --print-libgcc-file-name)`/specs
```

and redo the test, it should now works

Pkg-config [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/pkg-config.html)

Ncurses [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/ncurses.html)

Sed [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/sed.html)

Psmisc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/psmisc.html)

Gettext [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gettext.html)

Bison [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/bison.html)

Grep [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/grep.html)

Libtool [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/libtool.html)

Gdbm [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gdbm.html)

Gperf [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gperf.html)

Expat [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/expat.html)

Inetutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/inetutils.html)

Perl [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/perl.html)

Xml::Parser [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/xml-parser.html)

Intltool [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/intltool.html)

Autoconf [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/autoconf.html)

Automake [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/automake.html)

Kmod [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/kmod.html)

Libelf from ElfUtils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/libelf.html)

Libffi [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/libffi.html)

OpenSSL [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/openssl.html)

Python [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/Python.html)

Ninja [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/ninja.html)

Meson [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/meson.html)

Coreutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/coreutils.html)

Check [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/check.html)

Diffutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/diffutils.html)

Gawk [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gawk.html)

Findutils [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/findutils.html)

Groff [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/groff.html)

Grub [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/grub.html)

Less [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/less.html)

Gzip [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/gzip.html)

Iproute2 [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/iproute2.html)

Kbd [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/kbd.html)

Libpipeline [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/libpipeline.html)

Make [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/make.html)

Patch [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/patch.html)

Man-db [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/man-db.html)

Tar [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/tar.html)

Texinfo [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/texinfo.html)

Vim [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/vim.html)

Eudev [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/eudev.html)

Procps-ng [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/procps-ng.html)

Util-linux [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/util-linux.html)

E2fsprogs [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/e2fsprogs.html)

Sysklogd [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/sysklogd.html)

Sysvinit [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter08/sysvinit.html)

Personnally, i like to install thoses packages in my lfs (even if they come from BLFS)

Wget [Here](http://www.linuxfromscratch.org/blfs/view/stable/basicnet/wget.html)

Ssh [Here](http://www.linuxfromscratch.org/blfs/view/stable/postlfs/openssh.html)

Personally, I have chosen to be able to log at root, at least at the beginning

## Everything is ready ? No sir !

### Cleaning things up

We remove files that could remain for tests:

```
rm -rf /tmp/*
```

Logout and renter the chroot using the bash we just installed

```
logout

chroot "$LFS" /usr/bin/env -i          \
    HOME=/root TERM="$TERM"            \
    PS1='(lfs chroot) \u:\w\$ '        \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin \
    /bin/bash --login
```

We can now remove last bits of files that could remain during compilation, such as library:

```
rm -f /usr/lib/lib{bfd,opcodes}.a
rm -f /usr/lib/libbz2.a
rm -f /usr/lib/lib{com_err,e2p,ext2fs,ss}.a
rm -f /usr/lib/libltdl.a
rm -f /usr/lib/libfl.a
rm -f /usr/lib/libz.a
find /usr/lib /usr/libexec -name \*.la -delete
```

### Configuration for boot 

Install Lfs-bootscripts followings [those](http://www.linuxfromscratch.org/lfs/view/stable/chapter07/bootscripts.html) steps (the package should be in source directory)

We can now create custom udev rules:

```
bash /lib/udev/init-net-rules.sh
```

and create cd-rom links:

```
udevadm test /sys/block/hdd
sed -i -e 's/"write_cd_rules"/"write_cd_rules mode"/' \
    /etc/udev/rules.d/83-cdrom-symlinks.rules
udevadm info -a -p /sys/class/video4linux/video0

cat > /etc/udev/rules.d/83-duplicate_devs.rules << "EOF"
# Persistent symlinks for webcam and tuner
KERNEL=="video*", ATTRS{idProduct}=="1910", ATTRS{idVendor}=="0d81", \
    SYMLINK+="webcam"
KERNEL=="video*", ATTRS{device}=="0x036f", ATTRS{vendor}=="0x109e", \
    SYMLINK+="tvtuner"

EOF
```

### Network Configuration

We create a interface enp0s3 for the network (because we are on virtualbox)

with a static ip:

```
cd /etc/sysconfig/
cat > ifconfig.enp0s3 << "EOF"
ONBOOT=yes
IFACE=enp0s3
SERVICE=ipv4-static
IP=192.168.1.2
GATEWAY=192.168.1.1
PREFIX=24
BROADCAST=192.168.1.255
EOF
```

and the resolv.conf file:

```
cat > /etc/resolv.conf << "EOF"
# Begin /etc/resolv.conf

domain <Your Domain Name>
nameserver 8.8.8.
nameserver 1.1.1.1

# End /etc/resolv.conf
EOF
```

Customizing hostname and hosts file

```
echo "lfs-lubenard" > /etc/hostname
cat > /etc/hosts << "EOF"
127.0.0.1 localhost lfs-lubenard 
EOF
```

### Configuring Sysvinit

```
cat > /etc/inittab << "EOF"
id:3:initdefault:

si::sysinit:/etc/rc.d/init.d/rc S

l0:0:wait:/etc/rc.d/init.d/rc 0
l1:S1:wait:/etc/rc.d/init.d/rc 1
l2:2:wait:/etc/rc.d/init.d/rc 2
l3:3:wait:/etc/rc.d/init.d/rc 3
l4:4:wait:/etc/rc.d/init.d/rc 4
l5:5:wait:/etc/rc.d/init.d/rc 5
l6:6:wait:/etc/rc.d/init.d/rc 6

ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now

su:S016:once:/sbin/sulogin

1:2345:respawn:/sbin/agetty --noclear tty1 9600
2:2345:respawn:/sbin/agetty tty2 9600
3:2345:respawn:/sbin/agetty tty3 9600
4:2345:respawn:/sbin/agetty tty4 9600
5:2345:respawn:/sbin/agetty tty5 9600
6:2345:respawn:/sbin/agetty tty6 9600

EOF
```

### Configuring the system clock

```
cat > /etc/sysconfig/clock << "EOF"
UTC=1

# Set this to any options you might need to give to hwclock,
# such as machine hardware clock type for Alphas.
CLOCKPARAMS=
EOF
```

### Configuring keyboard

This code below will configure the keyboard layout: 

The layout of my keyboard is french, so the KEYMAP variable is fr. 

Replace it with what corresponds to your keyboard layout.

```
cat > /etc/sysconfig/console << "EOF"
# Begin /etc/sysconfig/console

UNICODE="1"
KEYMAP="fr-latin1"
KEYMAP_CORRECTIONS="euro2"
LEGACY_CHARSET="iso-8859-15"
FONT="LatArCyrHeb-16 -m 8859-15"

# End /etc/sysconfig/console
EOF
```

### Shell startup script

In my case, i chose french language

```
LC_ALL=fr_FR locale charmap
LC_ALL=fr_FR locale language
LC_ALL=fr_FR locale charmap
LC_ALL=fr_FR locale int_curr_symbol
LC_ALL=fr_FR locale int_prefix

cat > /etc/profile << "EOF"
# Begin /etc/profile

export LANG=fr_FR.UTF8@euro

# End /etc/profile
EOF
```

### Inputrc and shells file

Inputrc [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter07/inputrc.html)

Shells [Here](http://www.linuxfromscratch.org/lfs/view/stable/chapter07/etcshells.html)

### Configure the swap file

We configure the swap partition (in my case, sda5)

```
mkswap /dev/sda5
swapon /dev/sda5
```

to see if the swap is active, type:

```
swapon
```

## It's alive !

### Create the /etc/fstab file

```
cat > /etc/fstab << "EOF"
# file system  mount-point  type     options             dump  fsck
#                                                              order

/dev/sda7     /            ext4    defaults            1     1
/dev/sda5     swap         swap     pri=1               0     0
proc           /proc        proc     nosuid,noexec,nodev 0     0
sysfs          /sys         sysfs    nosuid,noexec,nodev 0     0
devpts         /dev/pts     devpts   gid=5,mode=620      0     0
tmpfs          /run         tmpfs    defaults            0     0
devtmpfs       /dev         devtmpfs mode=0755,nosuid    0     0

EOF
```

### Time to compile the Kernel !

```
cd /sources/
tar -xvf linux-5.5.3.tar.gz
cd linux-5.5.3
make mrproper
make menuconfig
```

A menu should appear. Go into:
General setup -> Local version - append to kernel release
You can now type the name of your kernel.
I chose "Linux kernel-5.10.17-lubenard".
Save, then exit.

```
make
make modules_install
```

We mount /boot to set it as boot partition

```
mkdir -p /boot
mount /dev/sda6 /boot
```

And copy files inside of it

```
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-5.10.17-lubenard
cp -iv System.map /boot/System.map-5.10.17
cp -iv .config /boot/config-5.10.17
install -d /usr/share/doc/linux-5.10.17
cp -r Documentation/* /usr/share/doc/linux-5.10.17
```

```
install -v -m755 -d /etc/modprobe.d
cat > /etc/modprobe.d/usb.conf << "EOF"
# Begin /etc/modprobe.d/usb.conf

install ohci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i ohci_hcd ; true
install uhci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i uhci_hcd ; true

# End /etc/modprobe.d/usb.conf
EOF
```

### Setting grub up !

We are creating two entry to be able to boot from our lfs or our debian

```
grub-install /dev/sda
cat > /boot/grub/grub.cfg << "EOF"
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5

insmod ext2

menuentry "GNU/Linux, Linux 5.5.3-lubenard" {
        set root=(hd0,6)
        linux   /vmlinuz-5.5.3-lubenard root=/dev/sda7 ro
}
menuentry "Good old debian" {
        set root=(hd0,1)
        linux   /vmlinuz root=/dev/sda1 ro
        initrd /initrd.img
}
EOF
```

### Final reboot

Logout and umount all mounted disks, then reboot

```
logout
umount -v $LFS/dev/pts
umount -v $LFS/dev
umount -v $LFS/run
umount -v $LFS/proc
umount -v $LFS/sys
umount -v $LFS
umount -v $LFS/usr
umount -v $LFS/home
umount -v $LFS
shutdown -r now
```

You now have a working command line Linux !

If you want to free some space, you can remove all sources folders by doing

```
cd /sources
rm -rf */
```

This will delete the folders, but not the originals tar if you need them again

## And after ?

What ? You want more than just a CLI ? This is for you !

First of all, let's create a user that is not root:
```
useradd lubenard --create-home --shell /bin/bash
passwd lubenard
<Type new passd for user lubenard>
```

Since we do not want only root to be able to access the source folder, we can also let our user access it:
```
su root -c "chmod -R 777 /sources"
```

Then, we export the PS1 environment variable, in order to have a more visible prompt:

```
echo 'export PS1="\u@\h:\w# \[$(tput sgr0)\]"' >> ~/.bashrc
source ~/.bashrc
```

If you want to customize your prompt, you can use [this](http://bashrcgenerator.com/) website

We can also add some useful aliases to our user's config:
```
alias l="ls -l"
alias ll="ls -lha"
alias ..="cd ../"
alias ...="cd ../../"
alias e="tar -xvf"
```

Obviously, we want the bashrc to load when our shell starts up. To do this, we need to create a profile for each user associated to our shell.

For our user, we set up the minimal profile:

```
echo '[[ -f ~/.bashrc ]] && . ~/.bashrc' > ~/.bash_profile
```

This simple script means that if the .bashrc file exists, then bash has to load it

### Lxde power ?

We will now install Lxde. Lxde is working with Xorg, so we need to install it first

#### Installing Xorg

To install Xorg, we need to fill it's dependencies, beginning with Cmake.

Install Libuv first following [this](http://www.linuxfromscratch.org/blfs/view/stable/general/libuv.html) page,

then [libarchive](http://www.linuxfromscratch.org/blfs/view/stable/general/libarchive.html) and [curl](http://www.linuxfromscratch.org/blfs/view/stable/basicnet/curl.html), and finally [Cmake](http://www.linuxfromscratch.org/blfs/view/stable/general/cmake.html)

Then i proceed to install Xorg following the BLFS tutorial, availabale [here](http://www.linuxfromscratch.org/blfs/view/stable/x/xorg7.html)

#### Problems with LLVm

If you encounter problems during compilation of LLVM, and the error message is saying

```
g++: fatal error: Killed signal terminated program cc1plus
```

, type ```dmesg```

If the last line begin with ```Out of memory```, just shutdown the virtual machine and increase the ram allowed.

I've personally gone from 2gb to 5gb of ram.

Then, relaunch the virtual machine and relaunch the compilation.

#### Test to check if evrything is working correctly

When you're done installing all the packets, you can check if X server is launching correctly by typing:

```
startx /usr/bin/xclock
```

If, on your vm screen, a clock is displayed, it means that everything is working.

You can then test with xterm, by doing:

```
startx /usr/bin/xterm
```

If you get a error message saying that xterm cannot get monospace font, install font from [this](http://www.linuxfromscratch.org/blfs/view/stable/x/TTF-and-OTF-fonts.html#dejavu-fonts) page.

I personally chose FreeFont. To install it, type:

```
mkdir -pv /usr/share/fonts/default/TrueType
tar -xvf freefont-ttf.tar.gz
mv sdf/* /usr/share/fonts/default/TrueType/
```

You should normally be able to start xterm

You can follow the guide to install Lxde [here](http://www.linuxfromscratch.org/blfs/view/stable/lxde/lxde.html)

I choose to install the following packages:
 - git [here](http://www.linuxfromscratch.org/blfs/view/stable/general/git.html)
 - nmap [here](http://www.linuxfromscratch.org/blfs/view/stable/basicnet/nmap.html)
 - terminator [here](https://github.com/gnome-terminator/terminator)(If you are having trouble with org.gnome.desktop.interface, try (re)installing [this](https://download.gnome.org/sources/gsettings-desktop-schemas/3.38/gsettings-desktop-schemas-3.38.0.tar.xz))
 - Firefox [here](http://www.linuxfromscratch.org/blfs/view/stable/xsoft/firefox.html)
 - xarchiver [here](http://www.linuxfromscratch.org/blfs/view/stable/xsoft/xarchiver.html)
 - sudo [here](http://www.linuxfromscratch.org/blfs/view/stable/postlfs/sudo.html)
 - libreoffice [here](http://www.linuxfromscratch.org/blfs/view/stable/xsoft/libreoffice.html)
 - baobab [here](http://www.linuxfromscratch.org/blfs/view/stable/gnome/baobab.html)

For sudo, i decided to create a sudo group, give it all rights (with visudo), and adding my normal account to the sudo group.
