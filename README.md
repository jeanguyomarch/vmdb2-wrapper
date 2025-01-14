# VMDB2-Wrapper


[![actions build ](https://github.com/Jerome-Maurin/vmdb2-wrapper/workflows/Build%20images/badge.svg)](https://github.com/Jerome-Maurin/vmdb2-wrapper/actions)

VMDB2-Wrapper is a simple wrapper for [`vmdb2`](https://vmdb2.liw.fi/), to build armhf & arm64 board images for SD-card using `u-boot` Debian packages, `flash-kernel` and Debian kernels. 

Source code for `vmdb2` can be found on [Lars Wirzenius' Gitlab](https://gitlab.com/larswirzenius/vmdb2/) or [his own Gitano server](http://git.liw.fi/vmdb2/).

The Raspberry Pi different models are already [supported by Debian](https://raspi.debian.net).<br>
FYI these images are also [build using `vmdb2`](https://salsa.debian.org/raspi-team/image-specs/).

# Table of contents

- [Supported Platforms](#supported-platforms)
- [TL;DR Getting started](#tldr-getting-started)
- [How to build an image](#how-to-build-an-image)
  * [Setting up the environment](#setting-up-the-environment)
  * [Choosing the right target](#choosing-the-right-target)
  * [Building the image](#building-the-image)
- [Writing the image to an SD-card](#writing-the-image-to-an-sd-card)
- [Customizing the image by using an Ansible-playbook](#customizing-the-image-by-using-an-ansible-playbook)
- [Needed packages](#needed-packages)
- [FAQ](#faq)
  * [Why must the build be run as root ?](#why-must-the-build-be-run-as-root-)
  * [How to add the support for a new board ?](#how-to-add-the-support-for-a-new-board-)
- [Potential issues](#potential-issues)
  * [Cleanup old cache](#cleanup-old-cache)

# Supported Platforms

FIXME

# TL;DR Getting started

You can either download an already built image from [the projetct's Github Releases](https://github.com/Jerome-Maurin/vmdb2-wrapper/releases) or [the project's Github Actions (nigthly builds)](https://github.com/Jerome-Maurin/vmdb2-wrapper/actions) (needs to be logged-in) and then skip to the [**Writing the image to an SD-card**](#writing-the-image-to-an-sd-card) section.

### Or

You can build the image yourself using a Debian (or an Ubuntu, you'll need to adapt the Debian procedure).

# How to build an image

## Setting up the environment

On a freshly installed minimalist Debian (with sudo installed), use this command to install needed packages:

    sudo apt install vmdb2 curl ansible python3-distutils qemu-user-static binfmt-support

The purpose behind each of those packages is explained in the [**Needed packages**](#needed-packages) section.

At the moment, the `vmdb2` version in Debian Buster lacks a critical feature which forces the installation of Bullseye's version.<br>
Either add the Bullseye repository to your `sources.list` (be carefull to [limit the package to vmdb2](https://wiki.debian.org/AptConfiguration#apt_preferences_.28APT_pinning.29)) or retrieve and install [the Bullseye package](https://packages.debian.org/bullseye/all/vmdb2/download) manually.<br>
For example (manual install):

    wget http://ftp.de.debian.org/debian/pool/main/v/vmdb2/vmdb2_0.22-1_all.deb
    sudo dpkg -i vmdb2_0.22-1_all.deb

## Choosing the right target

Each Yaml file corresponds to a single board using the naming convention BOARD_RELEASE_ARCH_vmdb2-MINVERSION.yaml where:
  - BOARD is the board's name.
  - RELEASE is the expected Debian release.
  - ARCH is the expected architecture (armhf or arm64 for example).
  - MINVERSION is vmdb2's minimum required version for the Yaml file to work.<br>
    FYI versions of `vmdb2` are retro-compatible with older yaml files versions (0.14.1 yaml files will work with version 0.14.1+).

## Building the image

For the time being `vmdb2` needs to be run as root, see [**Why must the build be run as root ?**](#why-must-the-build-be-run-as-root-) for more details.

Use `head` on the file corresponding to your board and run the command present in the comment on the first line.

`vmdb2` command generic example:

    sudo vmdb2 BOARD_RELEASE_ARCH_vmdb2-MINVERSION.yaml --output BOARD_RELEASE_ARCH.img --rootfs-tarball RELEASE_ARCH_rootfs.tgz --log=stderr

### Or

You can run the following oneliner if you prefer:

    eval $(head -n1 BOARD_RELEASE_ARCH_vmdb2-MINVERSION.yaml | sed 's/^.*: \(.*\)$/\1/g')

The resulting image will be called `BOARD_RELEASE_ARCH.img`

# Writing the image to an SD-card

To write img to sdcard, use `dd`.

For example:

    sudo dd bs=64k status=progress oflag=dsync if=cubietruck_buster_armhf.img of=/dev/mmcblk1

Make sure `/dev/mmcblkN` is the correct SD-card (by using `lsblk` for example).

In case the image comes from Github Releases or Github Actions you could use something like that:

    zcat cubietruck_buster_armhf.img.bz2.zip | bunzip2 -c -d | sudo dd bs=64k status=progress oflag=dsync of=/dev/mmcblk1

# Customizing the image by using an Ansible-playbook

For Ansible use `vmdb2-ansible.yaml.exemple` as a starting point, create a file named `vmdb2-ansible.yaml` to write an Ansible-playbook that will be used by `vmdb2`.

# Needed packages 

Why do I need thoses packages on my system to run a image build ?

`curl` is needed to fetch some binaries from the internet.

******************************

`ansible` is needed for the build to support customizing the image with an ansible playbook.

In some cases the needed package `python3-distutils` might not be installed, which can trigger an error in the ansible part.<br>
Make sure it is installed.

You can always comment or remove the call to ansible roles in the yaml files if you don't want to install it.

******************************

Extra packages needed for cross-compile build (use of qemu-debootstrap in yaml, default):

`qemu-user-static` and `binfmt-support`

You can always remplace `qemu-debootstrap` by `debootstrap` to build natively without needing `qemu-user-static` & `binfmt-support`, but in case you don't want to change the yaml files and you don't mind having `qemu-user-static` & `binfmt-support` on your system, `qemu-debootstrap` will also work for native builds with almost no overhead.

# FAQ

## Why must the build be run as root ?

For the time being it is easier to be root for the abilities to create /dev/loops and mount/unmount them.<br>
An alternative could be available later.

## How to add the support for a new board ?

The best starting point is the `cubieboard2_buster_armhf_vmdb2-0.14.1.yaml` file which is the simplest.<br>
FIXME<br>
Is the card supported by flash kernel ?<br>
If not, ..., comment the rm of /etc/flash-kernel/machine, if not kernel update wont work<br>
Same if flash-kernel cannot retrieve the card's name by looking in /proc/device-tree/model<br>
For example in case something else than U-Boot is used as bootloader<br>
FIXME

# Potential issues

## Cleanup old cache

If you face any issue when running the built image, try removing the corresponding cache file `RELEASE_ARCH_rootfs.tbz` and rebuilding the image.
