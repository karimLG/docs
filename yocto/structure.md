# Architecture of the Yocto project for DCs

The project is spread over several git repositories containing a Yocto project layer or the sources of a software component.
These repositories are unified by the repo tool.

[TOC]

# Yocto structure
The Yocto structure goal is to be generic for all the existent and future data concentrators.
The Yocto mirror layers and the BSP layers are meant to be shared by all teams.
The French and Finland teams can then, in their own layers (meta-application-fr and meta-application-fi) define their own images, while sharing the maximum amount of work.
This should reduce duplicated work to a minimum.

Yocto project is managed using gerrit and therfore deposited in different git repositories.
There are two types:
* **sources repositories**: for tracking L+G sources and mirroring external sources.
* **layer repositories**: for tracking or mirroring layers recipes and relevant material.

![](/images/YoctoStructure.png)

## Manifests
The `DC/manifests` contains google repo manifests for DCs.
It contains repo manifest files describing the directories that are visible and where they should be obtained from with git.

## Generic layers
Generic Yocto layers which are mirrored on gerrit.
Latest LTS branch for those layers : kirkstone.

| Generic layers               |         |                                                                   |
|------------------------------|---------|-------------------------------------------------------------------|
| `DC/poky-mirror`             | layer   |Poky sources mirror for data concentrators                         |
| `DC/meta-openembedded-mirror`| layer   | OpenEmbedded layer sources mirror for data concentrators          |
| `DC/meta-java-mirror`        | layer   | Yocto Java layer mirror for data concentrators                    |
| `DC/meta-security-mirror`    | layer   | Yocto security layer sources mirror for data concentrators        |

## Landis BSP layer
Landis specific layers created and managed by the French DC team at Montluçon.
The Landis BSP layer contains contains the recipes to build the Linux kernel and the u-boot bootloader tools for the DC450S4 and other machines.
The Linux and u-boot tools sources are respectively hosted in DC/linux and DC/u-boot repositories.
The Linux 5.4 LTS revision and the 2012.02 u-boot revision are used.

| Landis BSP layer             |         |                                                                   |
|------------------------------|---------|-------------------------------------------------------------------|
| `DC/meta-bsp`                | layer   | Yocto BSP layer for data concentrators                            |
| `DC/u-boot`                  | sources | U-Boot bootloader for the data concentrators                      |
| `DC/linux`                   | sources | Linux kernel sources for data concentrators                       |

## Landis system layer
The meta-system layer contains the recipes which are hardware specific.
For now it contains the u-log recipe that is used to fetch u-boot logs on Linux.

|                              |         |                                                                   |
|------------------------------|---------|-------------------------------------------------------------------|
| `DC/meta-system`             | layer   | Yocto system layer for data concentrators.                        |
| `DC/u-log`                   | sources | Tool to read U-Boot logs from Linux userspace.                    |

## Landis common application layer
This layer contains the application recipes that will be shared between Finland and France DC teams.
For now it contains recipes dedicated to the HSM module and the security service on the Linux side.

|                              |         |                                                                   |
|------------------------------|---------|-------------------------------------------------------------------|
| `DC/meta-application-common` | layer   | Yocto France specific application layer for data concentrators    |
| `DC/admin-hsm-tool`          | sources | DC HSM admin tool program                                         |
| `DC/crypto-engine`           | sources | OpenSSL engines for the DC security modules                       |
| `DC/PLC-modem-firmware`      | sources | PLC modem management software                                     |

## Landis France layer
The meta-application-fr layer contains France specific application recipes for the DC450S4 Java and other machines.

| Landis France layer          |         |                                                                   |
|------------------------------|---------|-------------------------------------------------------------------|
| `DC/meta-application-fr`     | layer   | Yocto France specific application layer for data concentrators    |
| `DC/client-java-pik`         | sources | Java PIK client for data concentrators                            |
| `DC/csp-java`                | sources | Java CSP for data concentrators                                   |
| `DC/wdg-stat`                | sources | DC watchdog statistics program                                    |
| `DC/client-notifier`         | sources | DC client notifier programs                                       |
| `DC/core-firmware`           | sources | Data concentrator core French firmware                            |
| `DC/image-tools`             | sources | DC image modules tools                                            |
| `DC/update-tool`             | sources | DC update tools programs                                          |

## Landis Finland layer
The meta-application-fi layer contains Finland specific application recipes for the DC450S4 AIM.

| Landis Finland layer         |         |                                                                   |
|------------------------------|---------|-------------------------------------------------------------------|
| `DC/meta-application-fi`     | layer   | Yocto Finland specific application layer for data concentrators   |