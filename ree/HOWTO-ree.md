HOWTO build Linux and other REE packages using Buildroot
========================================================

Updated: Mar 23, 2018.


Prerequisites
=============

Install mips toolchain and add to path
- mips-mti-linux-gnu toolchain

The MIPS codescape toolchain can be obtained here:

    wget https://codescape.mips.com/components/toolchain/2016.05-06/Codescape.GNU.Tools.Package.2016.05-06.for.MIPS.MTI.Linux.CentOS-5.x86_64.tar.gz


It is recommended to become familiar with the
[Buildroot documentation](https://buildroot.org/downloads/manual/manual.html),
in particular 'Chapter 8. General Buildroot usage' and '8.2. Understanding when
a full rebuild is necessary'. Buildroot does not attempt to detect what parts
of the system should be rebuilt when the system configuration is changed and
some user expertise is required.


Build instructions
==================

These instructions assume the following directory structure where the external
linux and buildroot project directories have been fetched.

    +--- scripts/do-mips-tee-mrconfig
    +--- ree/
       +--- (linux/ mips-tee-linux.git => mips.com repository)
       +--- (buildroot/ buildroot.git => buildroot.org external repository)
       +--- configs/
       |  +--- mips_tee_defconfig
       |
       +--- package/
          +--- (libteec/ mips-tee-client.git => mips.com repository)
          +--- (mips-tee-test/ mips-tee-test.git => mips.com repository)
             +--- clientapps/
             +--- xtest/

The `do-mips-tee-mrconfig` script can help populate the external repositories
and set the required branches (if you were following mips-tee/README, this step
is already finished).

Make buildroot specifying the configs/mips_tee_defconfig file to enable:
external build root, external toolchain (pre-installed toolchain), and external
linux source tree (LINUX_OVERRIDE_SRCDIR). Specify BR2_EXTERNAL the first time,
afterwards it is saved in output/.br-external.mk and does not need to be
specified any more.

    cd ree/buildroot
    make BR2_EXTERNAL=../../ree mips_tee_defconfig

Using `make menuconfig`, edit the path to the external pre-installed toolchain
and ensure the toolchain version matches the BR2_TOOLCHAIN_EXTERNAL_* config
option.

Use the [External options] menu item for enabling the GlobalPlatform TEE Client
API, sample client applications, and basic xtest testing framework.  Note
however that the extended xtest GlobalPlatform Client Test Suite is proprietary
and has to be purchased first in order to use the xtest_gp option.

The mips_tee_defconfig enables BR2_TARGET_ENABLE_ROOT_LOGIN and sets
BR2_TARGET_GENERIC_ROOT_PASSWD to its default. It also includes Busybox
and specifies the ree/board/busybox.config.

    # Set the external toolchain version and absolute path
    # (BR2_TOOLCHAIN_EXTERNAL_PATH).
    # Change the default root password (BR2_TARGET_GENERIC_ROOT_PASSWD).
    # Enable the GP Client API (BR2_PACKAGE_LIBTEEC).
    # Optionally enable the test_starter_ca app (BR2_PACKAGE_TEST_STARTER_CA),
    # the xtest framework (BR2_PACKAGE_XTEST) and the extended GP test suite
    # (BR2_PACKAGE_XTEST_GP).
    make menuconfig
    make

To update and save the default buildroot configuration run:

    make savedefconfig

To start a clean buildroot build run:

    make clean all

Note: You should never use `make -jN` with Buildroot. Use the BR2_JLEVEL option
to tell Buildroot to run the compilation of each individual package with `make
-jN`.

After building linux and the rootfs (with libteec and xtest), build L4Re and
the mips-lk TEE (see tee/mips-lk/HOWTO-tee) in order to package linux and the
rootfs into the final boot image.


Working with Linux and OVERRIDE_SRCDIR
======================================

When buildroot has been setup to use the LINUX_OVERRIDE_SRCDIR feature (see
override.mk), use the commands below to re-run Linux specific commands.  When
rebuilding, buildroot will rsync any custom Linux SRCDIR changes to the
buildroot/output/build directory.  For more information refer to the Buildroot
documentation.

    cd ree/buildroot
    make linux-menuconfig
    make linux-reconfigure
    make linux-rebuild all

After checking out a differing linux branch or editing any files, run
`make linux-rebuild` (and not just `make`) to get buildroot to rsync the most
recent source changes into the build directory.

To save the default linux configuration to BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE
run:

    make linux-update-defconfig


Rebuilding root file system components with Buildroot
=====================================================

After modifying any of the root file system components (such as libteec, xtest,
the test_starter_ca app, busybox) it is necessary to both rebuild the modified
component and re-package it into the root file system. Some configuration
changes require that the component output directory be deleted for a clean
build, or in some cases a full buildroot `make clean all` is required.  For
more information refer to the Buildroot documentation.

Use the following commands to selectively rebuild any of these components which
have changed and finally run make to include them in the rootfs.

    cd ree/buildroot
    make libteec-rebuild
    make test_starter_ca-rebuild
    make busybox-rebuild
    make                   # include the re-built component in the rootfs

The xtest extended test suite uses a script to add the test suite source files
into the build. Use these commands to rebuild and clean the xtest directories.

    cd ree/buildroot
    make xtest-clean
    make xtest-rebuild
    make                   # include the re-built component in the rootfs


Common errors
=============

- Buildroot does not attempt to detect what parts of the system should be
  rebuilt when the system configuration is changed e.g. using `make menuconfig`
  or `git checkout`. It is the responsibility of the user to know when a full
  rebuild is necessary. Refer to the Buildroot documentation.

- You should never use `make -jN` with Buildroot

- If the Linux default configuration arch/mips/configs/mach_virt_defconfig is
  not found verify that the mips-virt patchset is present in the current Linux
  branch.

- Linux `make` should be run from the buildroot directory. When run from the
  ree/linux directory instead this error occurs:

> ***
> *** Configuration file ".config" not found!
> ***
