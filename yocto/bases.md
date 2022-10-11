# [Yocto] for [DC] project bases

{#toc}
[TOC]

# [Yocto] Principle

## Manifest
Manifests are tracked in a [gerrit] repository (in the `DC/manifests` repository).
It declares the project layers by specifying their repositories and eventually their commit identifier.

## Layers
Each layer is tracked in a [gerrit] repository and contains several [recipes].

## [Recipes]
Each [recipe] explains how to build a specific part of the image.
It can handle specific cases for every image or machine.
[Recipes] can be modified in all upper layer.
It contains the sources of application to be included by specifying their repositories and normally their commit identifier.

## Common layers
Most of the layers are common to all [DC] products and must remains.
Changes must then be made carefully to preserve all product integrity.
Products specificities must be handle int their own top layer, when possible.

## Specific layer
The top layer of each manifest is product specific (`DC/meta-application-fr` and `DC/meta-application-fi`).

## Work
Regular work consists in modifying a [recipe], for example the commit identifier of some sources.
Less often recipes have to be added or deleted.

When the change has been tested and validated, the change must be proposed to the layer's [gerrit] repository.

# [gerrit] principle
[gerrit] uses [git] to track [Yocto] layers and code sources.
It secures it by adding automatic tests and a dedicated code review tool.
It is not possible to directly push changes to the [git] repository, it has to be proposed first.
>Note when sources has changed, commit identifier update should be considered in relative [recipes]

Modifications are done in several steps:
1. modification on a local [git] clone
2. Proposal to [gerrit]
3. automatic verification by [jenkins]
4. code review by team members with [gerrit]
5. submission from [gerrit] to [git] 

## Proposing a change
On a local clone, and once the change is tested, the regular workflow is:

### Current status
get status:
```bash
git status
```
>Note: `git status` displays next proposed [git] command

### Add files
Add relevant modified files:\
    `<files>`: all the files that has changed 
```bash
git add <files>
```

### Commit change
commit change locally:
```bash
git commit
```
A Commit message must be created. It should contain:
* 1 line brief comment. Ideally indicating the concerned component<br>for example:*dc-modem-firmware: sources moved to DC/PLC-modem-firmware*
* empty line
* eventually multi lines comment + an empty line
* Identification, starting by `Identification: ` and indicating a JIRA or TFS entry<br>for example:*Identification: FRKLINST-1347*

### Push commit
Push to [gerrit]:\
    `<remote>`: for example:*landis* or *origin*\
    `<local commit>`: for example:*HEAD*\
    `<remote branch>`: for example:*master* or *kirkstone*
```bash
git push <remote> <local commit>:refs/for/<remote branch>
```
>Note: `<remote>` can be retrieved from `git remote`\
`<local commit>` from `git log --graph --oneline`\
`<remote branch>` from `git branch -r`

## Verification
Once proposed, verification is done by [jenkins].
It can include building and automatic tests.
On success change is proposed for reviewing.

## Review
Reviewers can comment or add or remove points.
When the score reaches +2 code can be tracked.


>**/!\ TODO continue**
### Assign reviewers


[DC]: /glossary.md#dc
[gerrit]: /glossary.md#{#gerrit}
[git]: /glossary.md#{#git}
[jenkins]: /glossary.md#{#jenkins}
[recipe]: /glossary.md#{#recipe}
[recipes]: /glossary.md#{#recipe}
[Yocto]: /glossary.md#yocto