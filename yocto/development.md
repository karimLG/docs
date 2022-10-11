# Development with [Yocto] for [DC]
This document intends to introduce essential material for [DC] development using the [Yocto] project.
It describes how to set the working environment and how to build and test a [DC] image.

>Note: working environment must be correctly [setup](/yocto/setup.md)

[TOC]{#toc}

# [DC] project architecture

## [Yocto] structure
The [Yocto] structure is presented here: [xxx](\yocto\structure.md).

## Image variants

### Machines
The available machines are:

|                       |                                               |
|-----------------------|-----------------------------------------------|
|**cr5-g1**             | [DC] for French market, [G1] [PLC] technology |
|**dc450-g3**           | [L+G] [DC], penultimate version               |
|**cr6-g3**             | [DC] for French market, [G3] [PLC] technology |
|**dc450-s4**           | [L+G] [DC], latest version                    |

### Images
The available images are:

|                       |                                               |
|-----------------------|-----------------------------------------------|
|**dc-base-image**      | minimalistic image ~2MiB                      |
|**dc-base-debug-image**| ditto plus debug packages such as [gdbserver] |
|**dc-image**           | full featured image                           |
|**dc-debug-image**     | ditto plus debug packages                     |

[![][home]](#toc)


# [File system] of the [build host]
Useful [build host] locations are reported here:

|                     |                                  |
|---------------------|----------------------------------|
|**/nfsroot/bsp**     | file system for [NFS]            |
|**/opt/landis/0.1/** | default SDK installation folder  |
|**/tftpboot**        | [TFTP] server directory          |
|**~/**               | user’s home directory            |
|**~/bsp/**           | file system for [NFS]            |
|**~/dc/**            | [Yocto] working folder           |
|**~/dc/sources**     | [Yocto] layers working copy      |

[![][home]](#toc)

# Build an image

## Initialize the workspace environment
To initialize the workspace, in the `~/dc/` directory, `init-build-env` is sourced :
```bash
. init-build-env
```
>Note: before initialization, repositories synchronisation (`repo sync`, cf. [setup](/yocto/setup.md#fetch-the-repositories)) should be considered to update the [Yocto] layers.

## Build the image
Once configured, [bitbake] can be used to build images.
Appropriate image and machine should be selected.\
    `<machine>`: relevant [machine](#machines), for example *dc450-s4*\
    `<image>`: relevant [image](#images), for example *dc-image*
Usage:
```bash
MACHINE=<machine> bitbake <image>
```
>Note: cache cleanup (`MACHINE=<machine> bitbake <image> -c clean all`) should be considered if a cache-related problem is suspected.
>Note: `EXTRA_IMAGE_FEATURES` clearing (in `dc/build/conf/local.conf`) should be considered as the sources and symbols are making the produced images way bigger.

Building generates up to three different outputs:
* `dc-image.tar.gz`: an image tarball that can be used for [NFS] based developments.
* `zImage`: a [kernel] image that is also shipped in the image tarball.
* `dc-image.squashfs.tar.gz`: a tarball containing the image tarball break-out in several squashfs module that can be installed on the NAND memory (France only).

# Add a [DC] [SSH] access
[SSH] a access is useful for remote debugging.
 >**![warning] TODO [SSH] access**

[![][home]](#toc)

# Test an image
The [DC] can be run using an image located on the [build host].
* A [TFTP] server sends the [Linux] [kernel] image and the device tree
* A [NFS] server provides the image root filesystem

## Image extraction
Extract the image to test in the home directory:\
    `<machine>`: relevant [machine](#machines), for example *dc450-s4*
```bash
cd ~/bsp
sudo tar -xvf ~/dc/build/tmp/deploy/images/<machine>/dc-image-<machine>.tar.gz

```
Create symbolic links from the `/tftpboot/` directory to the [Linux] [kernel] image and device tree from the extracted image:\
    `<username>`: [build host] username, for example *mdupond*\
    `<machine>`: relevant [machine](#machines), for example *cr5* or *dc450s4*
```bash
ln -sv /home/<username>/bsp/boot/zImage /tftpboot/dc-kernel
ln -sv /home/<username>/bsp/boot/<machine>.dtb /tftpboot/dc-dtb
```

## Bootloader configuration
The bootloader environement variables of the [DC] must be set (**once**) to run the hosted image.

Create a `/tftpboot/nboot` file containing:
```
nfsroot=root=/dev/nfs rootwait nfsroot=100.0.0.50:/nfsroot/bsp,nolock,vers=3,tcp
nboot=run bootargs_leds; setenv bootargs "console=ttyO0,115200n8 ${nfsroot} ip=100.0.0.1:100.0.0.50::255.255.255.0::eth0:off"; tftp dc-kernel; tftp $fdtaddr dc-dtb;bootz $loadaddr - $fdtaddr
bootdelay=15
```

IP addresses have to be set manually, other variables can be set from the `/tftpboot/nboot` file.
>Note: *100.0.0.50* is the [build host] IP address

On a [console terminal](#console-connection):

|   |   |
|---|---|
| **Interrupt** the u-boot boot process by pressing a key | ![](/images/TFTP_000.png)
| Set the **IP addresses** (if needed)<br>   setenv serverip 100.0.0.50  <br>   setenv ipaddr 100.0.0.1 <br>   setenv ethaddr 00:0F:93:0E:00:26 | ![](/images/TFTP_001.png)
| Update the **environement variables** <br>   tftp nboot  <br>   env import -t $loadaddr $filesize <br>   saveenv | ![](/images/TFTP_002.png)

## [NFS] start
Boot sequence must be stopped to boot on [NFS].\
On a [console terminal](#console-connection):

|   |   |
|---|---|
|**Interrupt** the u-boot boot process by pressing a key | ![](/images/TFTP_000.png)
| Start **[NFS]** <br>   run nboot | ![](/images/NFS_000.png)

[![][home]](#toc)

# [SDK]
To improve development experiment, [Yocto] project builds [SDK] and [eSDK] (extended [SDK]) images.

All [SDK] consist of the following:
* Cross-Development Toolchain: This toolchain contains a compiler, debugger, and various associated tools.
* Libraries, Headers, and Symbols: The libraries, headers, and symbols are specific to the image (i.e. they match the image against which the [SDK] was built).
* Environment Setup Script: This *.sh file, once sourced, sets up the cross-development environment by defining variables and preparing for [SDK] use.

Additionally, an [eSDK] has tools that allow you to easily add new applications and libraries to an image, modify the source of an existing component, test changes on the target hardware, and easily integrate an application.

## Build [SDK]
In the [initialized workspace](#initialize-the-workspace-environment):
```bash
bitbake dc-image -c populate_sdk
```

## Build [eSDK]
In the [initialized workspace](#initialize-the-workspace-environment):
```bash
bitbake dc-image -c populate_sdk_ext
```

## Install [SDK]
To install the [SDK], run:
```bash
~/dc/build/tmp/deploy/sdk/landis-glibc-x86_64-dc-image-cortexa8t2hf-neon-cr5-g1-0.1.sh
```

## Initialize [SDK] environment
[SDK] image is sourced to initialize the [SDK] environment:
```bash
. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi
```
>Note: */opt/landis/0.1/* is the default SDK installation folder.

[![][home]](#toc)

## [devtool]
[devtool] is a convenient command for building and deploying programs to be debugged.
CF. [Yocto docs](https://docs.yoctoproject.org/ref-manual/devtool-reference.html)

Main options are:\
    `<recipe>`: [recipe] to modify\
    `<host>`: deployment target

To modify the source for a [recipe]:
```bash
devtool modify <recipe>
```
To build a [recipe]:
```bash
devtool modify <recipe>
```
To deploy [recipe] output files:
```bash
devtool deploy-target <recipe> <host>
```

The deployment target (`<host>`) must be specified in `~/.ssh/config`
    `<key>`: SSH private key for example */home/mdupond/dc-key.pem*
```
Host cr5-g1
   User root
   Hostname 100.0.0.1
   IdentityFile <key>
```

# Develop an application

## Build an application
In an [initialized SDK environment](#initialize-sdk-environment), toolchain can be used directly with Makefile and Autotools-based projects.\
Or manually, for example:
```bash
$CC helloworld.c - o helloworld
```

## Debug an application
[gdbserver] must be run in the [DC] to launch the application (built with the image or separately) to be debugged:\
    `<application>`: path to the application
```bash
gdbserver :3389 <application> --no-background --verbose
```
Then application can be debugged using [gdb] in an [initialized SDK environment](#initialize-sdk-environment):
```bash
. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi
$GDB
(gdb) target remote 100.0.0.1:3389
(gdb) continue
```
>Note: debug symbols are normaly generated during the [SDK] build

[![][home]](#toc)

# Debug the image

## Debug images
In order to debug the firmware, the `dc-debug-image` or `dc-base-debug-image` images must be used.
Those images are including some useful packages for debugging such as [gdbserver] and strace.

It can also be handy to add debug symbols and application sources, to get meaningful backtraces
within [gdb]. To do so you need to edit the `dc/build/conf/local.conf` file this way:

```
EXTRA_IMAGE_FEATURES = "dbg-pkgs src-pkgs"
```

>Note: initial `EXTRA_IMAGE_FEATURES` value must be restored after debugging
as the sources and symbols are making the produced images way bigger.

## Gdbserver
Cf. [Debug an application](#debug-an-application) for [gdb] debugging of an image binary  (firmware.elf for example).



# Create a release
From a clean workspace, run:

```bash
cd ~/dc
repo manifest -r -o .repo/manifests/releases/version-xxx.xml
```

This creates a manifest with fixed commit references ([SHA-1s]) for all the layers.  This manifest
then needs to be commited to the `DC/manifests` [git] repository.


[home]: /images/ArrowUp.png
[warning]: /images/warning.png

[bitbake]: /glossary.md#bitbake
[build host]: /glossary.md#buildhost
[devtool]: /glossary.md#devtool
[eSDK]: /glossary.md#esdk
[File system]: /glossary.md#filesystem
[DC]: /glossary.md#dc
[G1]: /glossary.md#g1
[G3]: /glossary.md#g3
[gdb]: /glossary.md#gdb
[gdbserver]: /glossary.md#gdbserver
[git]: /glossary.md#git
[kernel]: /glossary.md#kernel
[L+G]: /glossary.md#lg
[Linux]: /glossary.md#linux
[NFS]: /glossary.md#nfs
[PLC]: /glossary.md#plc
[recipe]: /glossary.md#recipe
[SDK]: /glossary.md#sdk
[SHA-1s]: /glossary.md#sha-1
[SSH]: /glossary.md#ssh
[TFTP]: /glossary.md#tftp
[Yocto]: /glossary.md#yocto