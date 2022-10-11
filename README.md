# [Yocto] project documentation for [DC]

This repository contains **[documentation](/presentation.md#documentation-content)** on the [Yocto] project for data concentrators.

# Devices concerned:
|                       |                                               |
|-----------------------|-----------------------------------------------|
|**[CR5]-[G1]**         | [DC] for French market, [G1] [PLC] technology |
|**[CR6]-[G3]**         | [DC] for French market, [G3] [PLC] technology |
|**[DC450]-S3**         | [L+G] [DC], penultimate version               |
|**[DC450]-s4**         | [L+G] [DC], latest version                    |

# [DC] presentation
In a smart grid, the [DC] is a bridge between the information system and the meters.
It is responsible for collecting data from meters and providing it to the information system.
The [DC] is used to control each meter and to manage the [PLC] network.

Currently the project is dedicated to [DC450] and [CRx] devices.

# [Yocto] project presentation
The [Yocto] project is an open-source collaboration project that helps developers create custom [Linux]-based systems.
The [Yocto] project's focus is on improving the software development process for embedded [Linux] distributions.
The [Yocto] project provides interoperable tools, metadata, and processes that enable the rapid, repeatable development of [Linux]-based embedded systems in which every aspect of the development process can be customized.

Cf. [yoctoproject.org](https://www.yoctoproject.org)\, [docs.yoctoproject.org](https://docs.yoctoproject.org)

# Documentation content

|                                               |                                  |
|-----------------------------------------------|----------------------------------|
|**[Presentation](/presentation.md)**           | [Yocto] project presentation     |
|**[Structure](/yocto/structure.md)**           | [Yocto] project structure        |
|**[Setup](/yocto/setup.md)**                   | Workspace configuration          |
|**[Development](/yocto/development.md)**       | Basic development operations     |
|**[VSCode](/vscode/vscode.md)**                | Development using [VSCode]       |
|**[Cross development](/cross/visualstudio.md)**| Cross Development                |
|**[Glossary](/glossary.md)**                   | Key terms used                   |


[CRx]: /glossary.md#crx
[CR5]: /glossary.md#crx
[CR6]: /glossary.md#crx
[DC]: /glossary.md#dc
[DC450]: /glossary.md#dc450
[G1]: /glossary.md#g1
[G3]: /glossary.md#g3
[L+G]: /glossary.md#lg
[Linux]: /glossary.md#linux
[PLC]: /glossary.md#plc
[VSCode]: /glossary.md#vscode
[Yocto]: /glossary.md#yocto