# Startup Yocto for DCs

This document intends to introduce essential material for DC development using the Yocto project.
It describes how to set the working environment and how to build and test a DC image.

[TOC]

# L+G services

## Gerrit
Gerrit is a free, web-based team collaboration tool that facilitates code reviewing and version control as it integrates closely with Git.

Cf. [gerritcodereview.com](https://www.gerritcodereview.com)\
L+G Gerrit server address: [gerrit-eu.landisgyr.net](https://gerrit-eu.landisgyr.net)

## Jenkins
Jenkins is an open-source automation server.
It facilitates development by building and testing the project in an automated way.

Cf. [jenkins.io](https://www.jenkins.io)\
L+G Jenkins server address: [jenkins-eu.landisgyr.net](https://jenkins-eu.landisgyr.net)

[![][home]](#startup-yocto-for-dcs)

# Requirements

## Build host / development host
It has been decided to use a native Linux host, therefore a Linux PC or virtual machine.
It is recommended to use Ubuntu 22.04.
Regular setting uses Ubuntu 22.04 virtualized on VirtualBox.
It can be setup as follows.

## Setting up Ubuntu on VirtualBox
Install VirtualBox from [virtualbox.org](https://www.virtualbox.org)\
Add VirtualBox Extension Pack from [virtualbox.org](https://www.virtualbox.org) (must be administrator).
This adds additional features.

Download Ubuntu desktop 22.04 image (ubuntu-20.04.4-desktop-amd64.iso) from [releases.ubuntu.com/22.04](https://releases.ubuntu.com/22.04)


|  |  |
|--|--|
| **Start VirtualBox** (v6.1.36 for example) | ![](/images/VirtualBox_000.png)

| **Create a virtual machine** (Menu Machine/New…) <br> • Type: Linux <br> • Version: Ubuntu (64-bit) <br> • Memory size: at least 8GB <br> • Hard disk: Create a virtual hard disk now <br> | ![](/images/VirtualBox_001.png)
| • File Size: at least 150GB <br> • Hard disk file type: VDI (VirtualBox Disk Image) <br> • Storage on physical hard disk: Dynamically allocated | ![](/images/VirtualBox_002.png)
| **Settings** | ![](/images/VirtualBox_003.png)
| **General / Advanced** <br> • Shared Clipboard: Bidirectional <br> • Drag’n’Drop: Bidirectional | ![](/images/VirtualBox_004.png)
| **System / Processor** <br> • Processor(s): try just less than actual host processor count | ![](/images/VirtualBox_005.png)
| **Network** (internet access) <br> Attached to : NAT | ![](/images/VirtualBox_006.png)
| **Network** (DC access) <br> • Attached to : Bridged Adapter <br> • Name: *Select actual adapter* <br> •  Adapter Type: *try PCnet-FAST III* <br> • Promiscuous Mode: Allow All<br> • MAC Address: *The actual adapter address* | ![](/images/VirtualBox_007.png)
| **Insert Ubuntu image disk** <br> Storage <br> • Controller: IDE <br>  - Empty <br>   · Choose a disk file... <br>     Select Ubuntu desktop 20.04 image (*.iso) <br>   · Live CD/DVD: enabled | ![](/images/VirtualBox_010.png)
| **Start** the virtual machine | ![](/images/VirtualBox_011.png)
| Proceed to **Ubuntu installation** | ![](/images/VirtualBox_012.png)
| **Remove Ubuntu image disk** <br> Menu Devices<br>  Optical Drives<br>    Remove disk from virtual drive | ![](/images/VirtualBox_013.png)
| **Guest Additions** enables better performance and functionality<br>Menu Devices<br>  Insert **Guest Additions** CD image... <br>    Image (VBoxGuestAdditions.iso) is in VirtualBox installation folder | ![](/images/VirtualBox_014.png)


## L+G services access

[![][home]](#startup-yocto-for-dcs)
# Working environment setting

## DC connection
The DC can be connected to the development system on several ports.
### Ethernet connection
 ![](/images/DC_Ethernet_000.png)

### Console connection
Terminal setting:
115200 bps, 8 data bits, no parity, 1 stop bit, no flow control\
![](/images/DC_Console_000.png)

## Tools

### repo
Repo is a tool built on top of Git.
Repo helps manage many Git repositories, does the uploads to revision control systems, and automates parts of the development workflow.
Repo is not meant to replace Git, only to make it easier to work with Git.
The repo command is an executable Python script that can be put anywhere in user path.\
Cf. [android.googlesource.com/tools/repo](https://android.googlesource.com/tools/repo)

The Google repo tool can be installed this way:

```bash
sudo apt update && sudo apt install curl
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo
```
Then add the following line to ~/.bashrc:
```bash
# where to look for executables
export PATH=~/.bin/:$PATH
```
### Git

git is the version control system used by the Yocto project.

Install git:
```bash
sudo apt update && sudo apt install git
```
Configure git:\
    *\<email>*: relevant email address, for example *martin.dupond<nolink>@landisgyr.com*\
    *\<full name>*: user full name, for example *Martin Dupond*\
    Please note that quotes are mandatory
```bash
git config --global user.email "<email>"
git config --global user.name "<full name>"
git config --global credential.helper store
```

#### Git customization

Git can **eventually** be customized to facilitate the work.
That section can be skipped without concern.

##### Git completion

This helps building git command line by pressing `Tab` key.\
Download script from github at [github.com/git/git/blob/master/contrib/completion/git-completion.bash](https://github.com/git/git/blob/master/contrib/completion/git-completion.bash) and move it to `~/.git-completion.bash`.\
Add the following lines to `.bash_profile` or `bash_rc` in `~/`:\
```bash
# activate git command line completion
if [ -f ~/.git-completion.bash ]; then
  source ~/.git-completion.bash
fi 
```
##### Git prompt

Shell prompt can be customized for example to display the current git branch.\
Download script from github at [github.com/git/git/blob/master/contrib/completion/git-prompt.sh](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh) and move it to `~/.git-prompt.sh`.\
Adding the next lines to `.bash_profile` or `.bash_rc` in `~/` will display the current git branch with the following color code:
* red: unstashed 
* yellow: stashed 
* gray: untracked files
* cyan: staged files
* green: clean
```bash
# store colors
MAGENTA="\[\033[0;35m\]"
YELLOW="\[\033[01;33m\]"
BLUE="\[\033[00;34m\]"
LIGHT_GRAY="\[\033[0;37m\]"
CYAN="\[\033[0;36m\]"
GREEN="\[\033[00;32m\]"
RED="\[\033[0;31m\]"
VIOLET="\[\033[01;35m\]"
WHITE="\[\033[00m\]"
 
function color_my_prompt {
  local __user_and_host="$GREEN\u@\h"
  local __cur_location="$BLUE\W"           # capital 'W': current directory, small 'w': full file path
  local __git_branch_color="$GREEN"
  local __prompt_tail="$VIOLET$"
  local __user_input_color="$WHITE"
  local __git_branch=$(__git_ps1); 
  
  # colour branch name depending on state
  if [[ "${__git_branch}" =~ "*" ]]; then     # if repository is dirty
      __git_branch_color="$RED"
  elif [[ "${__git_branch}" =~ "$" ]]; then   # if there is something stashed
      __git_branch_color="$YELLOW"
  elif [[ "${__git_branch}" =~ "%" ]]; then   # if there are only untracked files
      __git_branch_color="$LIGHT_GRAY"
  elif [[ "${__git_branch}" =~ "+" ]]; then   # if there are staged files
      __git_branch_color="$CYAN"
  fi
   
  # Build the PS1 (Prompt String)
  PS1="$__user_and_host $__cur_location$__git_branch_color$__git_branch $__prompt_tail$__user_input_color "
}
 
# configure PROMPT_COMMAND which is executed each time before PS1
export PROMPT_COMMAND=color_my_prompt
 
# if .git-prompt.sh exists, set options and execute it
if [ -f ~/.git-prompt.sh ]; then
  GIT_PS1_SHOWDIRTYSTATE=true
  GIT_PS1_SHOWSTASHSTATE=true
  GIT_PS1_SHOWUNTRACKEDFILES=true
  GIT_PS1_SHOWUPSTREAM="auto"
  GIT_PS1_HIDE_IF_PWD_IGNORED=true
  GIT_PS1_SHOWCOLORHINTS=true
  . ~/.git-prompt.sh
fi
```
### python-is-python3

*python-is-python3* determines the use of the *python3* version for unversioned *python*.

Install *python-is-python3*:
```bash
sudo apt update && sudo apt install python-is-python3
```
### docker

*docker* is a platform for virtualized runtime environments (RTE).

Install *docker*:
```bash
sudo apt update && sudo apt install docker.io
git config --global user.email "<email>"
```
### TFTP

For image testing, a *TFTP* server transfers the Linux kernel image and the device tree.

Install *tftp*:
```bash
sudo apt install xinetd tftpd tftp
```
Create an */etc/xinetd.d/tftp* file that reads:
>*\<username>*: build host username, for example *mdupond*
```
service tftp
{
protocol        = udp
port            = 69
socket_type     = dgram
wait            = yes
user            = <username>
server          = /usr/sbin/in.tftpd
server_args     = /tftpboot
disable         = no
}
```
Create the */tftpboot* directory:
>*\<username>*: build host username, for example *mdupond*
```bash
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R <username> /tftpboot
```
Restart the xinetd service:
```bash
sudo service xinetd restart
```

### NFS
For image testing, a *NFS* server provides the root file system.

Install *NFS*:
```bash
sudo apt install nfs-kernel-server
```
Add the directory containing the root filesystem to */etc/exports*:
```bash
/nfsroot/bsp                 0.0.0.0/0.0.0.0(rw,no_root_squash,no_subtree_check,sync)
```
Create a symlink between */nfsroot/bsp* and the place to will store the root filesystem:\
>*\<username>*: build host username, for example *mdupond*
```bash
mkdir ~/bsp
sudo mkdir /nfsroot/
sudo ln -sv /home/<username>/bsp /nfsroot/bsp
```
Restart the NFS service:
```bash
sudo systemctl restart nfs-kernel-server
```

## Gerrit SSH access
The *SSH Gerrit* access have to be configured.
>The *SSH* access to the *Landis Gerrit* server is faulty.
Connection often hangs up unexpectedly.
For this reason, the *Landis Gerrit* repositories are fetched by *Yocto* using *HTTPS*.\
Note that over *VPN*, both *HTTPS* and *SSH* access are problematic.

Generate *SSH* keys (default file and no passphrase):
```bash
ssh-keygen -t ed25519
```
Configure build host by adding the following entry to the *~/.ssh/config* file:\
*~/.ssh/config* may have to be created\
>*\<username>*: L+G username, for example *dupondm*
```
Host gerrit-eu.landisgyr.net
     User <username>
     IdentityFile ~/.ssh/id_ed25519
     Port 29418
```
Read the generated SSH public key:
```bash
cat ~/.ssh/id_ed25519.pub
```
Configure Gerrit by adding the SSH public key in user setting in [gerrit-eu.landisgyr.net/settings/#SSHKeys](https://gerrit-eu.landisgyr.net/settings/#SSHKeys)
>![](/images/SSH_000.png)

## Gerrit HTTPS access
The *HTTPS Gerrit* access has to be configured.

>The *SSH* access to the *Landis Gerrit* server is faulty.
Connection often hangs up unexpectedly.
For this reason, the *Landis Gerrit* repositories are fetched by *Yocto* using *HTTPS*.\
Note that over *VPN*, both *HTTPS* and *SSH* access are problematic.

A password needs to be generated in [https://gerrit-eu.landisgyr.net/settings/#HTTPCredentials](https://gerrit-eu.landisgyr.net/settings/#HTTPCredentials).
It may be securely stored in a convenient location to be retrieved if needed
>![](/images/HTTPS_000.png)

The next git clone command's purpose is to store HTTPS credentials (username and password) in the Git credentials store.
This allows Yocto to fetch Gerrit repositories using HTTPS without prompting for any credential.
```bash
git clone https://gerrit-eu.landisgyr.net/DC/manifests /tmp/.manifests
```

## Fetch the repositories
To fetch the DC sources for the *kirkstone* branch, run the following command:\
>*\<username>*: build host username, for example *mdupond*
```bash
sudo usermod -aG docker <username>
```
Then log-out of the Ubuntu session and log-in again,
```bash
mkdir ~/dc && cd $_
repo init -u ssh://gerrit-eu.landisgyr.net/DC/manifests -b kirkstone -m fr.xml
repo sync
```
A different branch can be selected, or a different manifest: *fi.xml* for instance for the Finland manifest.

The docker images must be built by running in the *~/dc/* 
directory:
```bash
docker build -t landis_yocto sources/meta-application-fr/scripts
```

## Build host file system
Useful build host locations are reported here
|         |                       |
|---------|-----------------------|
|**~/**   | user’s home directory | 
|**~/dc/**| ...                   | 

[![][home]](#startup-yocto-for-dcs)
# DC project architecture

## Yocto structure
The Yocto structure is presented hereafter.
The goal is to have a structure generic to all the existent and future data concentrators.
The Yocto mirror layers and the BSP layers are meant to be shared by all teams.
The French and Finland teams can then, in their own layers (meta-application-fr and meta-application-fi) define their own images, while sharing the maximum amount of work.
This should reduce duplicated work to a minimum.
Here is a description of all the involved layers.
![](/images/Yocto_000.png)

### Generic layers
The yellow boxes are generic Yocto layers which are mirrored on our Gerrit.
We are using the latest LTS branch for those layers : kirkstone.
|                             |                                                                  |
|-----------------------------|------------------------------------------------------------------|
| DC/poky-mirror              | Poky sources mirror for data concentrators                       |
| DC/meta-openembedded-mirror | OpenEmbedded layer sources mirror for data concentrators         |
| DC/meta-java-mirror         | Yocto Java layer mirror for data concentrators                   |
| DC/meta-security-mirror     | Yocto security layer sources mirror for data concentrators       |

### Landis BSP layer
The blue boxes are Landis specific layers created and managed by the French DC team at Montluçon.
The Landis BSP layer contains contains the recipes to build the Linux kernel and the u-boot bootloader tools for the DC450S4 and other machines.
The Linux and u-boot tools sources are respectively hosted in DC/linux and DC/u-boot repositories.
The Linux 5.4 LTS revision and the 2012.02 u-boot revision are used.
|                             |                                                                  |
|-----------------------------|------------------------------------------------------------------|
| DC/meta-bsp                 | Yocto BSP layer for data concentrators                           |
| DC/u-boot                   | U-Boot bootloader for the data concentrators                     |
| DC/linux                    | Linux kernel sources for data concentrators                      |

### Landis system layer
The meta-system layer contains the recipes which are hardware specific.
For now it contains the u-log recipe that is used to fetch u-boot logs on Linux.
|                             |                                                                  |
|-----------------------------|------------------------------------------------------------------|
| DC/meta-system              | Yocto system layer for data concentrators.                       |
| DC/u-log                    | Tool to read U-Boot logs from Linux userspace.                   |

### Landis common application layer
This layer contains the application recipes that will be shared between Finland and France DC teams.
For now it contains recipes dedicated to the HSM module and the security service on the Linux side.
|                             |                                                                  |
|-----------------------------|------------------------------------------------------------------|
| DC/meta-application-common  | Yocto common application layer for data concentrators            |
| DC/admin-hsm-tool           | DC HSM admin tool program                                        |
| DC/crypto-engine            | OpenSSL engines for the DC security modules                      |

### Landis France layer
This layer contains France specific application recipes for the DC450S4 Java and other machines.
|                             |                                                                  |
|-----------------------------|------------------------------------------------------------------|
| DC/client-java-pik          | Java PIK client for data concentrators                           |
| DC/csp-java                 | Java CSP for data concentrators                                  |
| DC/wdg-stat                 | DC watchdog statistics program                                   |
| DC/client-notifier          | DC client notifier programs                                      |
| DC/core-firmware            | Data concentrator core French firmware                           |
| DC/image-tools              | DC image modules tools                                           |
| DC/update-tool              | DC update tools programs                                         |


### Landis Finland layer
The red boxes correspond to Landis Finland layer.
The meta-applicationfi layer is supposed to host all the Finland specific application recipes for the DC450S4 AIM.
|                             |                                                                  |
|-----------------------------|------------------------------------------------------------------|

**/!\ TODO where to put this ?**
* DC/PLC-modem-firmware   PLC modem management software
* DC/manifests        Google repo manifests for data concentrators.
* DC/meta-application-fi        
* DC/meta-application-fr        Yocto France specific application layer for data concentrators.

## Image variants

## Machines
The available machines are:
|                       |                                             |
|-----------------------|---------------------------------------------|
|**cr5-g1**             | DC for French market, G1 PLC technology     |
|**dc450-g3**           | L+G DC, penultimate version                 |
|**cr6-g3**             | DC for French market, G3 PLC technology     |
|**dc450-s4**           | L+G DC, latest version                      |

## Images
The available images are:
|                       |                                             |
|-----------------------|---------------------------------------------|
|**dc-base-image**      | minimalistic image ~2MiB                    |
|**dc-base-debug-image**| ditto plus debug packages such as gdbserver |
|**dc-image**           | full featured image                         |
|**dc-debug-image**     | ditto plus debug packages                   |

[![][home]](#startup-yocto-for-dcs)
# Build an image

## Initialize the workspace environment
To initialize the workspace, in the *~/dc/* directory, *init-build-env* is sourced :
```bash
. init-build-env
```
## Build the image
Once configured, bitbake can be used to build images.
Appropriate image and machine should be selected, for example *dc450-s4* machine and *dc-image* image.
>*\<machine>*: relevant machine, for example *dc450-s4*
*\<image>*: relevant image, for example *dc-image*
Usage:
```bash
MACHINE=<machine> bitbake <image>
```

# Add a DC SSH access
SSH a access is useful for remote debugging.\
 **/!\ TODO SSH access**

[![][home]](#startup-yocto-for-dcs)
# Test an image
The DC can be run using an image located on the build host.
* A TFTP server sends the Linux kernel image and the device tree
* A NFS server provides the image root filesystem

## Image extraction
Extract the image to test in the home directory:
>*\<machine>*: relevant machine, for example *dc450-s4*
```bash
cd ~/bsp
sudo tar -xvf ~/dc/build/tmp/deploy/images/<machine>/dc-image-<machine>.tar.gz

```
Create symbolic links from the */tftpboot* directory to the Linux kernel image and device tree from the extracted image:
>*\<username>*: build host username, for example *mdupond*\
*\<machine>*: relevant machine, for example *cr5* or *dc450s4*
```bash
ln -sv /home/<username>/bsp/boot/zImage /tftpboot/dc-kernel
ln -sv /home/<username>/bsp/boot/<machine>.dtb /tftpboot/dc-dtb
```

## Bootloader configuration
The bootloader environement variables of the DC must be set (**once**) to run the hosted image.

Create a */tftpboot/nboot* file containing:
```
nfsroot=root=/dev/nfs rootwait nfsroot=100.0.0.50:/nfsroot/bsp,nolock,vers=3,tcp
nboot=run bootargs_leds; setenv bootargs "console=ttyO0,115200n8 ${nfsroot} ip=100.0.0.1:100.0.0.50::255.255.255.0::eth0:off"; tftp dc-kernel; tftp $fdtaddr dc-dtb;bootz $loadaddr - $fdtaddr
bootdelay=15
```

IP addresses have to be set manually, other variables can be set from the */tftpboot/nboot* file.
>*100.0.0.50*: build host IP address

On a [console terminal](#console-connection):
|  |  |
|--|--|
| **Interrupt** the u-boot boot process by pressing a key | ![](/images/TFTP_000.png)
| Set the **IP addresses** <br>   setenv serverip 100.0.0.50  <br>   setenv ipaddr 100.0.0.1 <br>   setenv ethaddr 00:0F:93:0E:00:26 | ![](/images/TFTP_001.png)
| Update the **environement variables** <br>   tftp nboot  <br>   env import -t $loadaddr $filesize <br>   saveenv | ![](/images/TFTP_002.png)

## NFS start

Boot sequence must be stopped to boot on NFS.\
On a [console terminal](#console-connection):
|  |  |
|--|--|
|**Interrupt** the u-boot boot process by pressing a key | ![](/images/TFTP_000.png)
| Start **NFS** <br>   run nboot | ![](/images/NFS_000.png)

[![][home]](#startup-yocto-for-dcs)
# Flash an image

**/!\ TODO**

[![][home]](#startup-yocto-for-dcs)
# Debug an image

To improve development experiment, Yocto project builds SDK and extended SDK images.

All SDKs consist of the following:
* Cross-Development Toolchain: This toolchain contains a compiler, debugger, and various associated tools.
* Libraries, Headers, and Symbols: The libraries, headers, and symbols are specific to the image (i.e. they match the image against which the SDK was built).
* Environment Setup Script: This *.sh file, once sourced, sets up the cross-development environment by defining variables and preparing for SDK use.

Additionally, an extensible SDK has tools that allow you to easily add new applications and libraries to an image, modify the source of an existing component, test changes on the target hardware, and easily integrate an application.

## Build SDK

In the [initialized workspace](#initialize-the-workspace-environment):
```bash
bitbake dc-image -c populate_sdk
```

## Build extended SDK

In the [initialized workspace](#initialize-the-workspace-environment):
```bash
bitbake dc-image -c populate_sdk_ext
```

## Install SDK
To install the SDK, run:
```bash
~/dc/build/tmp/deploy/sdk/landis-glibc-x86_64-dc-image-cortexa8t2hf-neon-cr5-g1-0.1.sh
```

## Initialize SDK environment
SDK image is sourced to initialize the SDK environment:
```bash
. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi
```
>Note: */opt/landis/0.1/* is the default SDK installation folder.

# Develop an application

In an [initialized SDK environment](#initialize-sdk-environment), toolchain can be used directly with Makefile and Autotools-based projects.\
Or manually, for example:
```bash
$CC helloworld.c - o helloworld
```
# Debug an application

**/!\ TODO**

>**![warning] TODO content**


[home]: /images/ArrowUp.png
[warning]: /images/warning.png