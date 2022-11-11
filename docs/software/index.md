# Overview

[developing-overview]: ../development/index.md
[lumi-p]: ../hardware/storage/lumip.md
[easybuild]: ./installing/easybuild.md
[spack]: ./installing/spack.md
[contwrapper]: ../software/installing/container-wrapper.md
[singularity-container]: ../software/containers/singularity.md
[singularity-jobs]: ../runjobs/scheduled-jobs/container-jobs.md
[software-stacks]: ../runjobs/lumi_env/softwarestacks.md
[module-env]: ../runjobs/lumi_env/Lmod_modules.md

---
Under this heading are presented the ways to bring/install software on LUMI as well as guides on using some specific software packages. If you are looking for ways to
optimize your software for use on LUMI, consult the [developing section][developing-overview] instead.

---

!!! warning

    The home and project directories reside on the Lustre based parallel
    file system on [LUMI-P][lumi-p] which does not perform well with
    installations of software containing **a lot of small files**, e.g. Python or
    R environments installed via Conda or pip. For such software **a container
    based approach must be used.**



## Bringing your scientific software to LUMI

### 1. Install the software in your home or project directory
 - #### [EasyBuild][easybuild]
The [EasyBuild][easybuild] package manager is used to manage the [central software stack][software-stacks] on LUMI, which makes it easy to install additional software that extends it. For this reason we also recommend using EasyBuild for the software installations that it is suitable. 
To identify the software for which we already provide EasyBuild recipes, visit the [LUMI Software Library](https://klust.github.io/LUMI-EasyBuild-docs/)

 - #### [Spack][spack]
 An alternative way to install software on LUMI is with [Spack][spack]. Consult the Spack pages for available Spack installable software recipes on LUMI.

 - #### [Container wrapper][contwrapper]
 An alternative way to install software on LUMI that should be used especially with Conda/Pip installations, because otherwise they can cause a heavy load to the Lustre filesystem with large number of small files.


### 2. Singulairy/Apptainer container
Bring a [Singularity/Apptainer software container][singularity-container] and run it using the [Singularity container runtime][singularity-jobs] provided by LUMI.


