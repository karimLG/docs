 # [Visual Studio] and [VisualGDB] usage for [Yocto] / [DC]

{#toc}
[TOC]

# Presentation
[Visual Studio] + [VisualGDB] can conveniently be used as [Windows] cross-development suite for the [Yocto] environment.

# Installation

>**/!\ TODO Write installation section**

# Requirements
For [Windows] cross-development, a [Linux] build machine and a [DC] target must be available.
[DC] target can be a stand-alone device or can run on [NFS].
Both build machine and target must have a [SSH] server running.

# Clone [gerrit] sources
For an project, sources must be present on the [build host] or on the [Windows] computer.\
    `<repository url>`: URL of the repository to clone), for example *ssh://gerrit-eu.landisgyr*\
    `<directory>`: cloning location, for example *C:\sources\PLC-modem-firmware*
```bash
git clone <repository url> <directory>
```

# Create a [Visual Studio] project

|   |   |
|---|---|
| Create a new project<br/>Select **Linux Project Wizard**<br>select the [SDK] tool chain `/opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi`                     | ![](/images/visualgdb_000.png)
| Select **name** and **location**                                          | ![](/images/visualgdb_001.png)
| **Project type**: for example **Import a project**                        | ![](/images/visualgdb_002.png)
| Select **build machine** and **target**                                   | ![](/images/visualgdb_003.png)
| **Source location**: on the [build host] or on the [Windows] computer     | ![](/images/visualgdb_004.png)
| **Source code access**: select convenient option                          | ![](/images/visualgdb_005.png)
| **Building and debugging**: executable may be selected later              | ![](/images/visualgdb_006.png)

# Setup the [Visual Studio] project
[VisualGDB] can be tuned to the project, in particular to its construction parameters.\
![](/images/visualgdb_007.png)\
![](/images/visualgdb_013.png)
>**Warning**: Debug symbols have to be build for debugging with sources

# Build the [Visual Studio] project
Simply use `Build/Build Solution [F6]`menu\
![](/images/visualgdb_008.png)\
![](/images/visualgdb_009.png)

# Debug the [Visual Studio] project
Simply use `Debug/Start Debug [F5]`menu, or click on the green arrow.\
![](/images/visualgdb_010.png).
[Visual Studio] debug features are available (breakpoints, watches...)
![](/images/visualgdb_012.png)

>Note: The first time, the executable can be prompted.\
![](/images/visualgdb_011.png)


[build host]: /glossary.md#buildhost
[DC]: /glossary.md#dc
[gerrit]: /glossary.md#gerrit
[Linux]: /glossary.md#linux
[NFS]: /yocto/development.md#nfs-start
[SDK]: /glossary.md#sdk
[SSH]: /glossary.md#ssh
[Yocto]: /glossary.md#yocto
[VisualGDB]: /glossary.md#visualgdb
[Visual Studio]: /glossary.md#visualstudio
[Windows]: /glossary.md#windows