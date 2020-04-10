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
