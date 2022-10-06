# Yocto for DCs setup
This document describes how to set the working environment for DC development using the Yocto project.

[TOC]

[![][home]](#yocto-for-dcs-setup)

# Build host / development host
It has been decided to use a native Linux host, therefore a Linux PC or virtual machine.
It is recommended to use Ubuntu 22.04.
Regular setting uses Ubuntu 22.04 virtualized on VirtualBox.
It can be setup as follows.

[![][home]](#yocto-for-dcs-setup)

## Setting up Ubuntu on VirtualBox
Install VirtualBox from [virtualbox.org](https://www.virtualbox.org)\
Add VirtualBox Extension Pack from [virtualbox.org](https://www.virtualbox.org) (must be administrator).
This adds additional features.

Download Ubuntu desktop 22.04 image (ubuntu-20.04.4-desktop-amd64.iso) from [releases.ubuntu.com/22.04](https://releases.ubuntu.com/22.04)

|   |   |
|---|---|
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

[![][home]](#yocto-for-dcs-setup)

# Tools

## repo
repo is a tool built on top of Git.
repo helps manage many Git repositories, does the uploads to revision control systems, and automates parts of the development workflow.
Repo is not meant to replace Git, only to make it easier to work with Git.
The repo command is an executable Python script that can be put anywhere in user path.

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
## git
git is the version control system used by the Yocto project.

Install git:
```bash
sudo apt update && sudo apt install git
```
Configure git:\
    `<email>`: relevant email address, for example *martin.dupond<nolink>@landisgyr.com*\
    `<full name>` user full name, for example *Martin Dupond*\
    Please note that quotes are mandatory
```bash
git config --global user.email "<email>"
git config --global user.name "<full name>"
git config --global credential.helper store
```

### git customization
Git can **eventually** be customized to facilitate the work.
That section can be skipped without concern.

#### git completion
This helps building git command line by pressing `Tab` key.\
Download script from github at [github.com/git/git/blob/master/contrib/completion/git-completion.bash](https://github.com/git/git/blob/master/contrib/completion/git-completion.bash) and move it to `~/.git-completion.bash`.\
Add the following lines to `.bash_profile` or `bash_rc` in `~/`:
```bash
# activate git command line completion
if [ -f ~/.git-completion.bash ]; then
  source ~/.git-completion.bash
fi 
```
#### git prompt
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
  local __cur_location="$BLUE\W"              # capital 'W': current directory, small 'w': full file path
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
## python-is-python3
python-is-python3 determines the use of the python3 version for unversioned python.

Install python-is-python3:
```bash
sudo apt update && sudo apt install python-is-python3
```
## docker
docker is a platform for virtualized runtime environments (RTE).

Install docker:\
    `<email>`: relevant email address, for example *martin.dupond<nolink>@landisgyr.com*\
    Please note that quotes are mandatory
```bash
sudo apt update && sudo apt install docker.io
git config --global user.email "<email>"
```
## TFTP
For image testing, a TFTP server transfers the Linux kernel image and the device tree.

Install tftp:
```bash
sudo apt install xinetd tftpd tftp
```
Create an `/etc/xinetd.d/tftp` file that reads:\
    `<username>`: build host username, for example *mdupond*
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
Create the `/tftpboot` directory:\
    `<username>`: build host username, for example *mdupond*
```bash
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R <username> /tftpboot
```
Restart the xinetd service:
```bash
sudo service xinetd restart
```

## NFS
For image testing, a NFS server provides the root file system.

Install NFS:
```bash
sudo apt install nfs-kernel-server
```
Add the directory containing the root filesystem to `/etc/exports`:
```bash
/nfsroot/bsp                 0.0.0.0/0.0.0.0(rw,no_root_squash,no_subtree_check,sync)
```
Create a symlink between `/nfsroot/bsp` and the place to will store the root filesystem:\
    `<username>`: build host username, for example *mdupond*
```bash
mkdir ~/bsp
sudo mkdir /nfsroot/
sudo ln -sv /home/<username>/bsp /nfsroot/bsp
```
Restart the NFS service:
```bash
sudo systemctl restart nfs-kernel-server
```

[![][home]](#yocto-for-dcs-setup)

# Gerrit SSH access
The SSH Gerrit access have to be configured.
>The SSH access to the Landis Gerrit server is faulty.
Connection often hangs up unexpectedly.
For this reason, the Landis Gerrit repositories are fetched by Yocto using HTTPS.\
Note that over VPN, both HTTPS and SSH access are problematic.

Generate SSH keys (default file and no passphrase):
```bash
ssh-keygen -t ed25519
```
Configure build host by adding the following entry to the `~/.ssh/config` file:\
    `~/.ssh/config` may have to be created\
    `<username>`: L+G username, for example *dupondm*
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
![](/images/SSH_000.png)

[![][home]](#yocto-for-dcs-setup)

# Gerrit HTTPS access
The HTTPS Gerrit access has to be configured.

>The SSH access to the Landis Gerrit server is faulty.
Connection often hangs up unexpectedly.
For this reason, the Landis Gerrit repositories are fetched by Yocto using HTTPS.\
Note that over VPN, both HTTPS and SSH access are problematic.

A password needs to be generated in [gerrit-eu.landisgyr.net/settings/#HTTPCredentials](https://gerrit-eu.landisgyr.net/settings/#HTTPCredentials).
It may be securely stored in a convenient location to be retrieved if needed.\
![](/images/HTTPS_000.png)

The next git clone command's purpose is to store HTTPS credentials (username and password) in the Git credentials store.
This allows Yocto to fetch Gerrit repositories using HTTPS without prompting for any credential.
```bash
git clone https://gerrit-eu.landisgyr.net/DC/manifests /tmp/.manifests
```




[home]: /images/ArrowUp.png