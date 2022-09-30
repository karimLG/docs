This is the DC Yocto manifests repository. It contains the Google

[repo](https://gerrit.googlesource.com/git-repo/) manifests needed to build a

DC image.



# Repositories



The Yocto image is built from the following Git repositories:



- [DC/poky-mirror](https://gerrit-eu.landisgyr.net/admin/repos/DC/poky-mirror)

This is a mirror of the Poky layer repository hosted

[here](https://git.yoctoproject.org/poky/).



- [DC/meta-openembedded-mirror](https://gerrit-eu.landisgyr.net/admin/repos/DC/meta-openembedded-mirror)

This is a mirror of the OpenEmbedded layer repository hosted

[here](https://git.openembedded.org/meta-openembedded/).



- [DC/meta-security-mirror](https://gerrit-eu.landisgyr.net/admin/repos/DC/meta-security-mirror)

This is a mirror of the Poky security layer repository hosted

[here](https://git.yoctoproject.org/meta-security/).



- [DC/meta-bsp](https://gerrit-eu.landisgyr.net/admin/repos/DC/meta-bsp)

This is the BSP layer. It contains the hardware dependant recipes such as the

Linux kernel, the u-boot bootloader and the machine definition files.



- [DC/meta-system](https://gerrit-eu.landisgyr.net/admin/repos/DC/meta-system)

This layer contains the customization of the mirror layer recipes that are

common to all DC images. The OpenSSH patches needed to interact with the

security module are for instance hosted in this layer.



- [DC/meta-application-common](https://gerrit-eu.landisgyr.net/admin/repos/DC/meta-application-common)

This layer contains the recipes for all the DC applications that are common to

the different DC teams, Finland and France for instance.



- [DC/meta-application-fr](https://gerrit-eu.landisgyr.net/admin/repos/DC/meta-application-fr)

This layer contains the recipes specific to the France DC team.



# Build a DC image



The following instructions have been tested on Ubuntu 20.04 but they should

work on most Linux distributions.



## Install Google repo



The Google repo tool can be installed this way:



```sh

sudo apt update && sudo apt install curl

mkdir -p ~/.bin

curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo

chmod a+rx ~/.bin/repo



Add the following line to ~/.bashrc:

export PATH=~/.bin/:$PATH

```



## Configure the SSH access



Generate SSH keys:



```sh

ssh-keygen -t ed5519

```



Authorize the generated SSH public key on [Gerrit](https://gerrit-eu.landisgyr.net/settings/#SSHKeys):



```

cat ~/.ssh/id_ed25519.pub

```



Add the following entry to the ~/.ssh/config file:



```

Host gerrit-eu.landisgyr.net

     User <windows username> ;dupondm for instance

     IdentityFile ~/.ssh/id_ed25519

     Port 29418

```



## Configure the HTTPS access



The SSH access to the Landis Gerrit server is faulty. Connection often hangs up unexpectedly.

For this reason the Landis Gerrit repositories are fetched by Yocto using HTTPS.



Note that over VPN, both HTTPS and SSH access are problematic.



To configure the HTTPS access, you need to run the following commands:



```

sudo apt update && sudo apt install git python-is-python3 docker.io



git config --global user.email "martin.dupond@landisgyr.net"

git config --global user.name "Martin Dupond"

git config --global credential.helper store



git clone https://gerrit-eu.landisgyr.net/DC/manifests /tmp/.manifests

Cloning into '/tmp/.manifests'...

Username for 'https://gerrit-eu.landisgyr.net': extothacehm

Password for 'https://extothacehm@gerrit-eu.landisgyr.net':

```



The password needs to be generated here: https://gerrit-eu.landisgyr.net/settings/#HTTPCredentials.

You need to click on "Generate new password".



The above git clone command purpose is to store HTTPS crendentials in the Git credentials store.

This will allow Yocto to fetch Gerrit repositories using HTTPS without prompting for any credential.



## Fetch the repositories



To fetch the DC sources for the kirkstone branch, run the following command:



```sh

sudo usermod -aG docker <user>

```



Then log-out of your session and log-in again,



```sh

mkdir ~/dc && cd $_

repo init -u ssh://gerrit-eu.landisgyr.net/DC/manifests -b kirkstone -m fr.xml

repo sync

```



You can select a different branch, or a different manifest: *fi.xml* for the

Finland manifest for instance.



### Configure the workspace



To configure the workspace, one need to run:



```

docker build -t landis_yocto sources/meta-application-fr/scripts

. init-build-env

```



### Build the image



Once configured, bitbake can be used to build images:



```

MACHINE=cr5-g1 bitbake dc-image

MACHINE=dc450-s4 bitbake dc-image

```



The available machines are:



- cr5-g1

- dc450-g3

- cr6-g3

- dc450-s4



and the available images are:



- dc-base-image (minimalistic image ~2MiB)

- dc-base-debug-image (ditto plus debug packages such as gdbserver)

- dc-image (full featured image)

- dc-debug-image (ditto plus debug packages)



# Run the image



The images can be tested using a machine running any Linux distribution. The

following instructions have been validated on Ubuntu 22.04.



The idea is to setup a *tftp* server that will host the Linux kernel image and

the device tree as well as an *NFS* server that hosts the image root

filesystem.



The data concentrator bootloader is then setup to fetch the Linux kernel and

device tree from the *tftp* server.  The Linux kernel command line is

configured by the bootloader to mount the *NFS* server share as the root

filesystem.



A prerequisite is to have the data concentrator ILOC Ethernet port connected

to the Linux machine. The corresponding Linux IP address should be static and

set to *100.0.0.50/24*.  The data concentrator will use the *100.0.0.1* IP

address on its side.



The Linux machine must also be connected to the Internet as some packages need

to be installed. It can be handy to configure two network interfaces:



- A first network interface connected to the concentrator and configured to use

  the static IP address *100.0.0.50* as mentioned above.



- A second network interface connected to the Internet.



You can now follow those steps to configure the data concentrator network boot.



## Install tftpd



Run the following command:



```sh

sudo apt install xinetd tftpd tftp

```



then create an */etc/xinetd.d/tftp* file that reads:



```

service tftp

{

protocol        = udp

port            = 69

socket_type     = dgram

wait            = yes

user            = mathieu #use your Linux username

server          = /usr/sbin/in.tftpd

server_args     = /tftpboot

disable         = no

}

```



You can then create the */tftpboot* directory:



```sh

sudo mkdir /tftpboot

sudo chmod -R 777 /tftpboot

sudo chown -R mathieu /tftpboot #use your Linux username

```



and restart the *xinetd* service, this way:



```sh

sudo service xinetd restart

```



## Install NFS



You can install NFS by running:



```sh

sudo apt install nfs-kernel-server

```



you then need to add the directory containing your root filesystem to

*/etc/exports*:



```sh

/nfsroot/bsp                 0.0.0.0/0.0.0.0(rw,no_root_squash,no_subtree_check,sync)

```



and create a symlink between */nfsroot/bsp* and the place where you will store

your root filesystem:



```sh

mkdir ~/bsp

sudo mkdir /nfsroot/

sudo ln -sv /home/mathieu/bsp /nfsroot/bsp #use your Linux username

```



you can then restart the corresponding service:



```sh

sudo systemctl restart nfs-kernel-server

```



## Extract the image



You now need to extract the image you would like to test in your home

directory, this way:



```sh

cd ~/bsp

sudo tar -xvf dc-image-cr5-g1.tar.gz

```



you also need to create symbolic links from the */tftpboot* directory to the

Linux kernel image and device tree originating from the just extracted image:



```sh

ln -sv /home/mathieu/bsp/boot/zImage /tftpboot/dc-kernel #use your Linux username

ln -sv /home/mathieu/bsp/boot/cr5.dtb /tftpboot/dc-dtb #use your Linux username



or for the dc450-s4:



ln -sv /home/mathieu/bsp/boot/dc450s4.dtb /tftpboot/dc-dtb #use your Linux username

```



## Configure the bootloader



At this step, you can start the data concentrator and interrupt the u-boot

boot process by pressing "Enter".



You then need to run in u-boot prompt:



```sh

set nfsroot root=/dev/nfs rootwait nfsroot=100.0.0.50:/nfsroot/bsp,nolock,vers=3,tcp

set nboot 'run bootargs_leds; setenv bootargs "console=ttyO0,115200n8 ${nfsroot} ip=100.0.0.1:100.0.0.50::255.255.255.0::eth0:off"; tftp dc-kernel; tftp $fdtaddr dc-dtb;bootz $loadaddr - $fdtaddr'

saveenv

```



alternatively, you can create a */tftpboot/nboot* file containing:



```sh

nfsroot=root=/dev/nfs rootwait nfsroot=100.0.0.50:/nfsroot/bsp,nolock,vers=3,tcp

nboot=run bootargs_leds; setenv bootargs "console=ttyO0,115200n8 ${nfsroot} ip=100.0.0.1:100.0.0.50::255.255.255.0::eth0:off"; tftp dc-kernel; tftp $fdtaddr dc-dtb;bootz $loadaddr - $fdtaddr

```



and run those commands to load it:



```sh

tftp nboot

env import -t $loadaddr $filesize

saveenv

```



finally, you can run this command to trigger a data concentrator NFS boot:



```sh

run nboot

```



# Build a SDK



You can produce a SDK image this way:



```sh

bitbake dc-image -c populate_sdk

```



it is also possible to produce an extended SDK this way:



```sh

bitbake dc-image -c populate_sdk_ext

```



To install the SDK, you can run:



```sh

~/dc/build/tmp/deploy/sdk/landis-glibc-x86_64-dc-image-cortexa8t2hf-neon-cr5-g1-0.1.sh

```



and to source the installed SDK:



```sh

. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi

```



# Debug the image



## Debug images



In order to debug the firmware, you can use the *dc-debug-image* or *dc-base-debug-image* images.

Those images are including some useful packages for debugging such as gdbserver and strace.



It can also be handy to add debug symbols and application sources, to get meaningful backtraces

within gdb. To do so you need to edit the *dc/build/conf/local.conf* file this way:



```

EXTRA_IMAGE_FEATURES = "dbg-pkgs src-pkgs"

```



Don't forget to restore the initial *EXTRA_IMAGE_FEATURES* value once you are done debugging

as the sources and symbols are making the produced images way bigger.



## Gdbserver



To debug a binary (firmware.elf for example), you can run on the target:



```sh

gdbserver :3389 /usr/lib/titulaire/ldt/bin/firmware.elf --no-background --verbose

```



on the host machine, you can use gdb from the SDK:



```sh

. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi

$GDB

(gdb) target remote 100.0.0.1:3389

(gdb) continue

```

See the *SDK images* section below for the SDK generation procedure.



## Devtool



You can use this tool to build and deploy the programs you are willing to debug.

From the extended SDK environment or the traditional build environment, run:



```sh

devtool modify dc-core-firmware

devtool build dc-core-firmware

devtool deploy-target dc-core-firmware cr5-g1

```



The cr5-g1 argument refers to the *~/.ssh/config* file:



```sh

Host cr5-g1

  User root

  Hostname 100.0.0.1

  IdentityFile /home/mathieu/dc-key.pem

```



You can find more details on the *devtool* program [here](https://docs.yoctoproject.org/ref-manual/devtool-reference.html).



# Create a release



From a clean workspace, run:



```sh

cd ~/dc

repo manifest -r -o .repo/manifests/releases/version-xxx.xml

```



this creates a manifest with fixed SHA-1s for all the layers.  This manifest

then needs to be commited to the *DC/manifests* Git repository.