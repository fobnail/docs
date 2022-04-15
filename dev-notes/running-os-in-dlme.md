# Running OS in DLME

In this document, we would like to describe the attempts to run a minimal
operating system in the Dynamically Launched Measured Environment. For this
purpose, we have prepared a meta-fobnail layer, which allows you to build a
system image using the Yocto Project. Following this document will allow
building an operating system running in DLME independently.

## Prerequisites

- PC Engines apu2 with a serial connection to host PC
- Host PC that meets the requirements outlined in the Yocto Project
  [documentation](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html#compatible-linux-distribution)
  (tested on Ubuntu 20.04)

## Background

In order to prepare a system that will be capable to running in DLME we want to
use [Trenchboot](https://trenchboot.org/) project. It is a framework that allows
to build security engines to perform launch integrity actions for their systems.
The framework builds upon Boot Integrity Technologies (BITs) that establish one
or more Roots of Trust (RoT) from which a degree of confidence that integrity
actions were not subverted.

Because we want to test on PC Engines apu2 we needed that our system consists
of:

* [GRUB SecureLaunch for
  AMD](https://github.com/TrenchBoot/grub/tree/trenchboot_support_2.04)
* [Linux 5.13 SecureLaunch for Intel and
  AMD](https://github.com/TrenchBoot/linux/tree/linux-sl-5.13-amd)
* [Secure Kernel Loader](https://github.com/TrenchBoot/secure-kernel-loader)

The above information was obtained from the website describing the [Trenchboot
repositories](https://trenchboot.org/code/).

## Building

As mentioned earlier, we prepared a project using Yocto. For layers management
and simplification of the build process, we use
[kas](https://kas.readthedocs.io/en/latest/) tool with
[kas-container](https://github.com/siemens/kas/blob/master/kas-container)
script. All build components are open and public, so everyone can try to
reproduce our effort if only the prerequisites are met.

### Instructions

To complete the build process please execute the following steps.

1. Download `kas-container` script, place it in PATH

```
$ wget https://raw.githubusercontent.com/siemens/kas/2.6.2/kas-container ~/bin/kas-container
```

2. Make it executable.

```
$ chmod +x ~/bin/kas-container
```

3. Make sure that script can be executed from any place.

```
$ kas-container --help
Usage: /home/tomzy/bin/kas-container [OPTIONS] { build | checkout | shell } [KASOPTIONS] [KASFILE]
       /home/tomzy/bin/kas-container [OPTIONS] for-all-repos [KASOPTIONS] [KASFILE] COMMAND
       /home/tomzy/bin/kas-container [OPTIONS] clean
       /home/tomzy/bin/kas-container [OPTIONS] menu [KCONFIG]

Positional arguments:
build                   Check out repositories and build target.
checkout                Check out repositories but do not build.
shell                   Run a shell in the build environment.
for-all-repos           Run specified command in each repository.
clean                   Clean build artifacts, keep downloads.
menu                    Provide configuration menu and trigger configured build.

Optional arguments:
--isar                  Use kas-isar container to build Isar image.
--with-loop-dev         Pass a loop device to the container. Only required if
                        loop-mounting is used by recipes.
--runtime-args          Additional arguments to pass to the container runtime
                        for running the build.
--docker-args           Same as --runtime-args (deprecated).
-d                      Print debug output.
-v                      Same as -d (deprecated).
--version               print program version.
--ssh-dir               Directory containing SSH configurations.
                        Avoid $HOME/.ssh unless you fully trust the container.
--aws-dir               Directory containing AWScli configuration.
--git-credential-store  File path to the git credential store
--no-proxy-from-env     Do not inherit proxy settings from environment.
--repo-ro               Mount current repository read-only
                        (default for build command)
--repo-rw               Mount current repository writeable
                        (default for shell command)

You can force the use of podman over docker using KAS_CONTAINER_ENGINE=podman.
```

4. Clone `meta-fobnail` repository.

```
$ git clone git@github.com:fobnail/meta-fobnail.git
```

> Note: use `git clone https://github.com/fobnail/meta-fobnail.git` if do not
  have SSH keys in GitHub account.

5. Start build.

```
$ kas-container build meta-fobnail/kas-debug.yml
```

> Note: `kas-debug.yml` configuration will generate a debug version of the
  image, which differs from prod in that it does not have a root password

Running this command will clone every needed Yocto layer and start the build
which last about 2 hours for the first run.

## Tests

We checked couple configurations in order to run our minimal OS in DLME.

### Test of latest Trenchboot

From Trenchboot sources

#### 3mdeb GRUB fork

#### W/A patch for SKL

Link to TB issue

### Working solution

linux v5.8
landing-zone
3mdeb fork for GRUB

## Summary
