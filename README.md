# How I build CentOS 8 VMs locally

## Why?
I use KVM to build a number of virtual servers.  I _like_ to have all
my files local which means mirroring.  Now with CentOS 6 and 7 you
could start with a copy of the CD and then just mirror the updates tree.

But with CentOS 8 there is no "update" tree.  Instead changes are
pushed straight into the BaseOS or AppStream trees, and these are
a super-set of the files on the DVD.  Everything on the DVD is in the
mirror, so it's a waste of disk space to store the files twice.

So, instead, I worked out what to mirror from upstream and what needs to
be done to make the result usable for kickstart purposes.

These are my scripts and suitable for my setup.  They will need changing
to work for you.

## Creating a local mirror

Firstly, create a root for the tree.  In my case I call it `/CentOS/DVD/CentOS-8`.   I have this available on my webserver as `http://repo/CentOS/DVD/CentOS-8/`


Now mirror the AppStream and BaseOS directories.  The file `MIRROR.kernel.org`
will do this from the `kernel.org` server.  Pick a server that you like!

First time you run this will take a while; it's around 12Gb.  You can
re-run this script as often as you like, to keep your tree local.

The script `make-treeinfo` will make a `.treeinfo` file based on the
downloaded files.  This will closely match what would have been on
the DVD, but should(!) automatically update if any of the images change.
This is called automatically from the `MIRROR` script, so you don't need
to run this by hand.

First time round you'll need to make some symbolic links:

```
ln -s BaseOS/EFI
ln -s BaseOS/images
ln -s BaseOS/isolinux
ln -s BaseOS/media.repo
```

The resulting tree should look something like:

```
% ls -lgo
total 48
drwxr-xr-x 4 4096 Sep 28 06:25 ./
drwxr-xr-x 9 4096 Sep 27 19:04 ../
-rw-r--r-- 1 1637 Sep 27 20:38 .treeinfo
drwxr-xr-x 4 4096 Sep 20 18:23 AppStream/
drwxr-xr-x 7 4096 Sep 24 16:42 BaseOS/
lrwxrwxrwx 1   10 Sep 27 17:03 EFI -> BaseOS/EFI
-rwxr-xr-x 1  274 Sep 28 06:25 MIRROR.xmission.com*
lrwxrwxrwx 1   13 Sep 27 17:03 images -> BaseOS/images
lrwxrwxrwx 1   15 Sep 27 17:03 isolinux -> BaseOS/isolinux
-rwxr-xr-x 1 1132 Sep 27 20:38 make-treeinfo*
lrwxrwxrwx 1   17 Sep 27 17:03 media.repo -> BaseOS/media.repo
```

## Creating the kickstart

The attached `centos8.cfg` is one that works for me and creates a
mostly minimal setup.   It's about 1.2Gb on the root partition.
Since my small VMs are typically only allocated 10Gb of disk I don't
do any fancy LVM setup or clever partitioning; just a boot, root, swap.

## Installing with virt-install

To make life easier for myself (and to handle other OS versions) I
have a wrapper script `virt_build`.  This will create file images or
LVMs, work out the correct `virt-install` command, resize memory as
necessary and then run the build.  Now CentOS 8 needs about 3Gb of
RAM to install 'cos it copies all the rpms to ramdisk, but headless (the
way I use it) it can easily run in under 500Mb.  So this script will
run the install with lots of memory, then after install has complete
it will resize the virtual machine and reboot it.

On my old machine the whole process takes around 7 minutes.
