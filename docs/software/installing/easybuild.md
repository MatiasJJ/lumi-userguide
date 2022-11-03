# EasyBuild

[Lmod_modules]: ../../runjobs/lumi_env/Lmod_modules.md
[softwarestacks]: ../../runjobs/lumi_env/softwarestacks.md
[helpdesk]: ../../helpdesk/index.md
[lumi-g]: ../../hardware/compute/lumig.md
[eap]: ../../hardware/compute/eap.md

Most software in the central LUMI [software stacks][softwarestacks] is
installed through [EasyBuild](https://easybuild.io/). The central software
stack is kept as compact as possible to ease maintenance and to avoid user
confusion. E.g., packages for which users request special customisations will
never be installed in the central software stack. Moreover, due to the
technical implementation of a software stack on a system the size of LUMI,
installing software can be disruptive, so new software is mostly made available
during maintenance intervals.

This, however, does not mean that you may have to wait for weeks before you can
get the software you need for your project on LUMI. We have made it very easy to
install additional software in your home or project directories (where the
latter is a better choice as you can then share it with the other people in your
project). After installing, using the software requires not much more than loading a module
that configures EasyBuild for local installations and running EasyBuild with a
few recipes that can be supplied by the [User Support Team][helpdesk] or your
national support team or that you may write yourself. And this software is then
built in exactly the same way as it would be in a central installation.


!!! Note
    Before continuing to read this page, make sure you are familiar with the
    [setup of the software stacks on LUMI][softwarestacks] and somewhat familiar
    with [the Lmod module environment][Lmod_modules].



## Beginner's guide to installing software on LUMI with EasyBuild

*If you are new to EasyBuild and LUMI, it might be a good idea to first read through this chapter once, and then start software installations.*

You can check for available EasyBuild installable software on LUMI [in this repocitory](https://klust.github.io/LUMI-EasyBuild-docs/). 

### Preliminary step: Set up the location for your EasyBuild installations
By default our EasyBuild setup will install software in `$HOME/EasyBuild`.
However, this location can be changed by pointing the environment variable
`EBU_USER_PREFIX` to the directory where you want to create the software
installation. In most cases a subdirectory in your `/project/project_<id>`
directory is the best location to install software as that directory is both
permanent for the duration of your project and shared with all users in your
project so that everybody can use the software. It is a very good idea to set
this environment variable in your `.profile` or `.bashrc` file, e.g.

```bash
export EBU_USER_PREFIX=/project/project_<id>/EasyBuild
```

where `project_<id>` is your project identification, and the `<id>` is a number with 9 digits. Now `/project/project_<id>/EasyBuild/` will be the path where EasyBuild creates the software installations.

??? Tip "Tip for users with multiple projects"
    If you participate in multiple projects, you'll have to either have only
    a very personal software setup in your home directory which no one else can
    use, or a setup in each of the project directories, as sharing of project
    directories across projects is not possible. Our modules can also support
    only one user software setup at a time. However, you can always switch to
    a different setup by changing the value of the `EBU_USER_PREFIX`
    environment variable, but you should only do so when no modules are loaded,
    not even the `LUMI` module. Hence you should always do a

    ```bash
    $ module --force purge
    ```
    
    of at the very least
    
    ```bash
    $ module --force unload LUMI
    ```
    
    immediately before changing the value of `EBU_USER_PREFIX`. If you fail to
    do so, the old user module directories will not be removed from the module
    search path, not even if you reload the `LUMI` module, and you may get very
    unexpected results from module load operations.

### 1. Software stack version and partition

We support installing software with EasyBuild only in the LUMI software stacks,
not in CrayEnv. 

Let's assume that we want to install software in the `LUMI/22.06` stack. To load this version of LUMI software stack (and to possibly replace a version that we might have loaded earlier), we then execute

```bash
$ module load LUMI/22.06
```
This should also automatically load the right `partition module` for the part
of LUMI you are on, as further detailed on the [software
stacks][softwarestacks] page.

??? Failure "Issue: Only partition/L and partition/C are currently fully supported"
    Note that in the initial versions of the software stack, only `partition/L`
    and `partition/C` are supported. The `partition/G` module is for all MI250X
    GPU nodes, whether in the regular [LUMI-G][lumi-g] partition or in the
    temporary [Early Access Platform][eap], and is only meant for users who
    install their own software and not supported by the
    [User Support Team][helpdesk] except for basic build tools until after
    the LUMI-G pilot phase.

Though it is technically possible to cross-compile software for a different
partition, it may not be without problems as not all install scripts that come
with software support cross-compiling and as tests may fail when compiling for
a CPU with instructions that the host CPU does not support.

To check what modules are currently loaded, use the command `module list`, and to change to the compute nodes partition, load the partition/C

```bash
$ module load partition/C
```

### 2. EasyBuild environment

To get the EasyBuild features available, load the `EasyBuild-user` module:

```bash
$ module load EasyBuild-user
```

This will print a line on the screen indicating where software will be installed
as a confirmation. It will also create the directory structure for the user
software installation if it does not yet exist, including the structure of the
user repository discussed below in the ["Advanced
guide"](#advanced-guide-to-easybuild-on-lumi), section ["Building your own
EasyBuild repository"](#building-your-own-easybuild-repository). 

Now all the EasyBuild eb-commands are available to be used. Try e.g. the following commands: 

To see a list of all eb-commands:

```bash
$ eb --help
```

If you want
more information about the full configuration of EasyBuild:

```bash
$ eb --show-config
```

To see a basic list of software that is available on LUMI with EasyBuild:

```bash
$ eb --list-software
```

To search for a specific software and all the build recipes available for it:

```bash
$ eb --search <some-name>
```

where `<some-name>` is a name of a software, e.g. Gromacs. The EasyBuild -build recipes are files that end with '.eb'. The names consist of several components that carry important information about the recipe configuration. 


!!! Warning
    When searching for the build recipes, you get a list of all available ones, also the ones that are not compatible with your current settings of software stacks and programming environment. 


The EasyBuild installable software and the build recipes are also documented on [this page](https://klust.github.io/LUMI-EasyBuild-docs/).


### 3. Compatibility of a build recipe with the environment settings

To have an insight on what build recipe to use, one needs to first understand the naming scheme for the EasyBuild -build recipes on LUMI. For example, let's consider one of the build recipes for the software GROMACS that we have found e.g. with using the `eb --search gromacs` command:

```text
GROMACS-2021.4-cpeGNU-22.06-PLUMED-2.7.4-CPU.eb
```
The table below opens up the naming scheme:

| part of the name   | explanation
|--------------------|-------------------------------------------------------------------------------------|
| `GROMACS`          | the name of the package                                                             | 
| `2021.4`           | the version of GROMACS, in this case the initial 2021 release                       |
| `cpeGNU-22.06`     | the so-called *toolchain* used for the build                                        |
| `cpeGNU`           | uses the `PrgEnv-gnu` programming environment                                       |
| `cpeCray`          | uses the `PrgEnv-cray` programming environment                                      |
| `cpeAOCC`          | uses the `PrgEnv-aocc` programming environment                                      |
| `cpeAMD`           | uses the `PrgEnv-amd` programming environment                                       |
| `-PLUMED-2.7.4-CPU`| the version suffix                                                                  |

_Version suffixes_ are typically used to distinguish different builds of the same version of the
package. In this case, it indicates that it is a build of the 2021.4 version
purely for CPU and also includes PLUMED as we have also builds without PLUMED
(which is not compatible with every GROMACS version).

**Note that the version of the toolchain should match the version of the LUMI software stack or the installation will fail.** (In fact, it is not
just the version in the file name that should match but the version of the
toolchain that is used in the recipe file.) 

Pay attention to pick an EasyBuild -build recipe that either matches you current settings (check with `module list`) or change your settings. For the version of software stack, you can check again the [bullet point 1](#1-software-stack-version-and-partition). About programming environment, read more [here](../../development/compiling/prgenv.md) or just check that the version of the EasyBuild recipe matches your currenlty loaded PrgEnv. You can change the programming environment e.g. from cray to gnu with the command:

```bash
$ module load PrgEnv-gnu
```

### 4. Installing software

EasyBuild is configured so that it searches in the user repository and two
repositories on the system. The current directory is not part of the default
search path but can be easily added with a command line option. By default,
EasyBuild will not install dependencies of a package and fails instead, if one or
more of the dependencies cannot be found, but that is also easily changed on
the command line. If all needed EasyBuild recipes are in one of those
repository or in the current directory, all you need to do to install the software
package is to run (when continuing with the same GROMACS example)

```bash
$ eb -r . GROMACS-2021.4-cpeGNU-22.06-PLUMED-2.7.4-CPU.eb
```

The `-r` tells EasyBuild to also install dependencies that may not yet be
installed, and, with the dot added to it, to also add the current directory to
the front of the search path. The `-r .` or `-r` flags should be omitted if you
want full control and install dependency by dependency before installing the
package (which may be very useful if building right away fails).

EasyBuild now does the installation to the location that you have [defined](#preliminary-step-set-up-the-location-for-your-easybuild-installations) with the EBU_USER_PREFIX varialbe. With our GROMACS example case, the location of the installed module would be:

`/project/project_<id>/EasyBuild/modules/LUMI/22.06/partition/C/GROMACS`

Take this module into use:

```bash
$ module use /project/project_<id>/EasyBuild/modules/LUMI/22.06/partition/C/GROMACS
```

Now if you type `module avail` you should see on the list:

```text
GROMACS/2021.4-cpeGNU-22.06-PLUMED-2.7.4-CPU
```

In some current cases (at least with some recipes of GROMACS), the name of the actual software is dropped out from the module name, and you might only see e.g. `2021.4-cpeGNU-22.06-PLUMED-2.7.4-CPU` for the module name. Then use the name for the module that you actually see on the list. 

Note the relation between the name of the EasyBuild recipe
and the module name and version of the module. This is only the case though if
the EasyBuild recipe follows the EasyBuild guidelines for naming. If the
guidelines are not followed and if EasyBuild needs to install this module as a
dependency of another package, EasyBuild will fail to locate the build recipe.

Finally, to load the now available module, type `module load <module name>`, e.g.:

```text
module load GROMACS/2021.4-cpeGNU-22.06-PLUMED-2.7.4-CPU
```

## Advanced guide to EasyBuild on LUMI

### Toolchains on Cray

Toolchains in EasyBuild contain at least a compiler, but can also contain an
MPI library and a number of mathematical libraries (BLAS, LAPACK, ScaLAPACK and
a FFT library). Programs compiled with different toolchains cannot be loaded
together (though the module system will not always prevent this on LUMI).

The toolchains on LUMI are different from what you may be used to from non-Cray
systems. On most systems, EasyBuild uses its own toolchains installed from
within EasyBuild, but on LUMI we use toolchains that are based on the Cray
Programming Environment. Four toolchains are currently implemented

- `cpeGNU` is the equivalent of the Cray `PrgEnv-gnu` programming environment
- `cpeCray` is the equivalent of the Cray `PrgEnv-cray` programming environment
- `cpeAOCC` is the equivalent of the Cray `PrgEnv-aocc` programming environment
- `cpeAMD` is the equivalent of the Cray `PrgEnv-amd` programming environment

All four toolchains use `cray-mpich` over the Open Fabric Interface library
(`craype-network-ofi`) and Cray LibSci for the mathematical libraries, with the
releases taken from the Cray PE release that corresponds to the version number
of the `cpeGNU`, `cpeCray`, `cpeAOCC`, or `cpeAMD` module.

??? note "cpeGNU/Cray/AOCC/AMD and PrgEnv-gnu/cray/aocc/amd"
    Currently the `cpeGNU`, `cpeCray`, `cpeAOCC`, and `cpeAMD` modules don't
    load the corresponding `PrgEnv-*` modules nor the `cpe/<version>` modules.
    This is because in the current setup of LUMI both modules have their
    problems and the result of loading those modules is not always as intended.

    If you want to compile software that uses modules from the LUMI stack,
    it is best to use one of the `cpeGNU`, `cpeCray`, `cpeAOCC`, or `cpeAMD`
    modules to load the compiler and libraries rather than the matching
    `cpe/<version>` and `PrgEnv-*` modules as those may not always load
    all modules in the correct version.

Since the LUMI software stack does not support the EasyBuild common toolchains
(such as the EasyBuild intel and foss toolchains), one cannot use the default
EasyBuild build recipes without modifying them. Hence they are not included in
the robot search path of EasyBuild so that you don't accidentally try to
install them (and also removed from the search path for `eb -S` or `eb
--search` to avoid any confusion that they might work).

### Building your own EasyBuild repository

We advise users to maintain their own repository of EasyConfig files which they
installed in their personal or project space. This may help to rebuild your
environment for a later project on LUMI. It may even be a good idea to keep the
repository on a personal GitHub or other version control service.

The repository is created automatically the first time `EasyBuild-user` is
loaded. The directory is called `UserRepo` and is in `$EBU_USER_PREFIX` (or the
default location `$HOME/EasyBuild` if you don't set the environment variable).
It must be structured similarly to [the main EasyBuild EasyConfig
repository](https://github.com/easybuilders/easybuild-easyconfigs). The
EasyBuild recipes (`.eb` files) should be in a subdirectory
`easybuild/easyconfigs`, leaving room for personal EasyBlocks also (which would
then go in the `easybuild/easyblocks` subdirectory) and even personal
configuration files that overwrite some system options. This setup also
guarantees compatibility with some EasyBuild features for very advanced users
that go way beyond this page.

To store this repository on GitHub, you can follow the GitHub documentation,
and in particular the page ["Adding an existing project to GitHub using the
command
line](https://docs.github.com/en/github/importing-your-projects-to-github/importing-source-code-to-github/adding-an-existing-project-to-github-using-the-command-line).

Technical documentation on the toolchains on LUMI and the directory structure
of EasyBuild can be found in [the documentation of the LUMI-SoftwareStack
GitHub repository](https://lumi-supercomputer.github.io/LUMI-SoftwareStack/).

## Further reading

If you want to get more familiar with EasyBuild and develop your own EasyBuild
recipes, we suggest the following sources of information:

- [EasyBuild manual on ReadtheDocs](https://docs.easybuild.io/)
- [EasyBuild tutorials](https://easybuilders.github.io/easybuild-tutorial/)
- The [EasyBuild YouTube channel](https://www.youtube.com/channel/UCqPyXwACj3sjtOho7m4haVA)
  contains recordings of a four-session tutorial
  given for the LUMI User Support Team by Kenneth Hoste (UGent), the lead developer
  of EasyBuild and Luca Marsella (CSCS)
  - [Part 1: Introduction](https://www.youtube.com/watch?v=JTRw8hqi6x0)
  - [Part 2: Using EasyBuild](https://www.youtube.com/watch?v=C3S8aCXrIMQ)
  - [Part 3: Advanced topics](https://www.youtube.com/watch?v=KbcvHa4uO1Y)
  - [Part 4: EasyBuild on Cray systems](https://www.youtube.com/watch?v=uRu7X_fJotA)
- [Technical documentation on our setup for developers](https://lumi-supercomputer.github.io/LUMI-SoftwareStack/)
- LUMI EasyBuild recipes
  - [Main LUMI software stack GitHub repository](https://github.com/Lumi-supercomputer/LUMI-SoftwareStack)
    contains the full EasyBuild setup for LUMI, including the EasyBuild recipes
    that we use for the central software stack and many others that we fully support
    and consider of good quality. The clone on the system is automatically searched
    by the `EasyBuild-user` module.
  - [LUMI contributed EasyBuild recipes GitHub repository](https://github.com/Lumi-supercomputer/LUMI-EasyBuild-contrib)
    contains contributed EasyBuild recipes and other recipes developed by LUST
    that haven't been as thoroughly checked or are deemed not appropriate for central
    installation at this point. However, they are fully compatible with the setup
    on LUMI, with correct dependency versions etc.
- Other EasyBuild recipes for the Cray Programming Environment
  - [CSCS GitHub repository](https://github.com/eth-cscs/production).
    Most of the recipes are for Piz Daint which uses slightly different toolchains.
    Moreover dependencies typically need updating as the software installation
    on LUMI is not in sync with the CSCS installation. The repository is particularly
    useful for CPU-only programs as the GPUs in their system are not compatible
    with those in LUMI.
- EasyBuild recipes that are not compatible with the Cray Programming Environment but that
  may sometimes be a good source to start developing compatible ones (if you're an
  EasyBuild expert):
  - [EasyBuilders repository](https://github.com/easybuilders/easybuild-easyconfigs/tree/develop/easybuild/easyconfigs),
    the repository of EasyConfig files that also come with EasyBuild.
  - [ComputeCanada repository](https://github.com/ComputeCanada/easybuild-easyconfigs)
  - [IT4Innovations repository](https://code.it4i.cz/sccs/easyconfigs-it4i)
  - [Fred Hutchinson Cancer Research Center repository](https://github.com/FredHutch/easybuild-life-sciences/tree/main/fh_easyconfigs)
  - [University of Antwerpen repository](https://github.com/hpcuantwerpen/UAntwerpen-easyconfigs)
  - [University of Leuven repository](https://github.com/hpcleuven/easybuild-easyconfigs/tree/master/easybuild/easyconfigs)
