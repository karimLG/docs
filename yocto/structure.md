# Architecture of the Yocto project for DCs

[TOC]

# Yocto structure
The Yocto structure is presented hereafter.
The goal is to have a structure generic to all the existent and future data concentrators.
The Yocto mirror layers and the BSP layers are meant to be shared by all teams.
The French and Finland teams can then, in their own layers (meta-application-fr and meta-application-fi) define their own images, while sharing the maximum amount of work.
This should reduce duplicated work to a minimum.
Here is a description of all the involved layers.
![](/images/YoctoStructure.png)

## Manifests
The `DC/manifests` coàntains google repo manifests for DCs.

## Generic layers
Generic Yocto layers which are mirrored on gerrit.
Latest LTS branch for those layers : kirkstone.
|                              |                                                                   |
|------------------------------|-------------------------------------------------------------------|
| `DC/poky-mirror`             | Poky sources mirror for data concentrators                        |
| `DC/meta-openembedded-mirror`| OpenEmbedded layer sources mirror for data concentrators          |
| `DC/meta-java-mirror`        | Yocto Java layer mirror for data concentrators                    |
| `DC/meta-security-mirror`    | Yocto security layer sources mirror for data concentrators        |

## Landis BSP layer
Landis specific layers created and managed by the French DC team at Montluçon.
The Landis BSP layer contains contains the recipes to build the Linux kernel and the u-boot bootloader tools for the DC450S4 and other machines.
The Linux and u-boot tools sources are respectively hosted in DC/linux and DC/u-boot repositories.
The Linux 5.4 LTS revision and the 2012.02 u-boot revision are used.
|                              |                                                                   |
|------------------------------|-------------------------------------------------------------------|
| `DC/meta-bsp`                | Yocto BSP layer for data concentrators                            |
| `DC/u-boot`                  | U-Boot bootloader for the data concentrators                      |
| `DC/linux`                   | Linux kernel sources for data concentrators                       |

## Landis system layer
The meta-system layer contains the recipes which are hardware specific.
For now it contains the u-log recipe that is used to fetch u-boot logs on Linux.
|                              |                                                                  |
|------------------------------|------------------------------------------------------------------|
| `DC/meta-system`             | Yocto system layer for data concentrators.                       |
| `DC/u-log`                   | Tool to read U-Boot logs from Linux userspace.                   |

## Landis common application layer
This layer contains the application recipes that will be shared between Finland and France DC teams.
For now it contains recipes dedicated to the HSM module and the security service on the Linux side.
|                              |                                                                  |
|------------------------------|------------------------------------------------------------------|
| `DC/meta-application-common` | Yocto common application layer for data concentrators            |
| `DC/admin-hsm-tool`          | DC HSM admin tool program                                        |
| `DC/crypto-engine`           | OpenSSL engines for the DC security modules                      |
| `DC/PLC-modem-firmware`      | PLC modem management software                                    |

## Landis France layer
The `DC/meta-application-fr` layer contains France specific application recipes for the DC450S4 Java and other machines.
|                              |                                                                  |
|------------------------------|------------------------------------------------------------------|
| `DC/client-java-pik`         | Java PIK client for data concentrators                           |
| `DC/csp-java`                | Java CSP for data concentrators                                  |
| `DC/wdg-stat`                | DC watchdog statistics program                                   |
| `DC/client-notifier`         | DC client notifier programs                                      |
| `DC/core-firmware`           | Data concentrator core French firmware                           |
| `DC/image-tools`             | DC image modules tools                                           |
| `DC/update-tool`             | DC update tools programs                                         |

## Landis Finland layer
The `DC/meta-application-fi` layer contains Finland specific application recipes for the DC450S4 AIM.