MIPS TEE Project description
============================

Updated: Mar 23, 2018.

The MIPS TEE project combines the L4Re Hypervisor, mips-lk Trusted Execution
Environment (TEE) and Linux Rich Execution Environment (REE) to implement a TEE
meeting the requirements of the GlobalPlatform (GP) TEE API Specifications.

The MIPS TEE project leverages the features of the MIPS Virtualization (VZ)
architecture module to provide hardware-assisted isolation between the TEE and
REE components. The L4Re hypervisor uses the VZ features to support running the
REE and TEE operating systems in isolation in separate virtual machines.

- [MIPS Virtualization](https://www.mips.com/products/architectures/ase/virtualization/)
- [GlobalPlatform TEE Device Specifications](https://www.globalplatform.org/specificationsdevice.asp)
- [L4Re Hypervisor](https://l4re.org/)

This project is comprised of the following set of MIPS repositories and some
additional external repositories:

- [MIPS/mips-tee](https://github.com/MIPS/mips-tee.git)
- [MIPS/mips-lk](https://github.com/MIPS/mips-lk.git)
- [MIPS/mips-tee-linux](https://github.com/MIPS/mips-tee-linux.git)
- [MIPS/mips-tee-client](https://github.com/MIPS/mips-tee-client.git)
- [MIPS/mips-tee-test](https://github.com/MIPS/mips-tee-test.git)


Prerequisites
=============

- Linux distribution for building and running MIPS-TEE is a must at the moment.
  Ubuntu-based distributions are recommended.

- Bourne compliant shell environment.
  While most of the Bourne compliant shells should be usable for this project,
  BASH shell is recommended since the project is tested with it.

Install mips MTI toolchains and add to the PATH environment variable.

- mips-mti-elf toolchain (bare metal variant for little kernel)
- mips-mti-linux-gnu toolchain (for building L4Re and Linux)

The MIPS MTI codescape toolchains (suitable for mips32 R2-R5 instruction sets)
can be obtained here:

    wget https://codescape.mips.com/components/toolchain/2016.05-06/Codescape.GNU.Tools.Package.2016.05-06.for.MIPS.MTI.Bare.Metal.CentOS-5.x86_64.tar.gz
    wget https://codescape.mips.com/components/toolchain/2016.05-06/Codescape.GNU.Tools.Package.2016.05-06.for.MIPS.MTI.Linux.CentOS-5.x86_64.tar.gz

- dtc, the device tree compiler:
  In Ubuntu/Debian: `sudo apt-get install device-tree-compiler`

- If you want to build GlobalPlatform Compliance Test Suite you will need an
  XML parser. xalan and xsltproce are supported at the moment.


Getting Started
===============

To get started first clone the mips-tee.git repository.  To populate the
mips-tee directory with the necessary external repositories run the
initialization script from the top-level directory <mips_tee_directory>.

    cd <mips_tee_directory>
    ./scripts/do-mips-tee-mrconfig

The initialization script uses the myrepos tool `mr` to help clone the external
repositories and checkout the default branches.  Before building any of the
components run the `mr status` command and ensure each repository has the
desired branch checked out (e.g. master, development).  See [scripts/README-mr](./scripts/README-mr.md)
for more information about how `mr` can operate on these repositories.

Build the REE, TEE and L4Re components separately and finally package them
together in the L4Re Hypervisor bootstrap image. See the following HOWTO files
for further build instructions:

- REE: [ree/HOWTO-ree](./ree/HOWTO-ree.md)
- TEE and L4Re: [tee/mips-lk/HOWTO-tee](https://github.com/MIPS/mips-lk/blob/master/HOWTO-tee.md)


MIPS TEE project directory structure
====================================

Some directories are repositories for external projects. These are pulled
into the hierarchy below with the aid of an initialization script.


    (mips-tee/ mips-tee.git => mips.com repository)
    +--- tee/
    |  +--- (mips-lk/ mips-lk.git => mips.com repository)
    |
    +--- (l4re/ => svn.l4re.org repository)
    |
    +--- ree/
    |  +--- (linux/ mips-tee-linux.git => mips.com repository)
    |  +--- (buildroot/ buildroot.git => buildroot.org external repository)
    |  +--- board/
    |  |  +--- rootfs_overlay/
    |  |  +--- patches/
    |  |     +--- linux/
    |  |
    |  +--- configs/
    |  |  +--- mips_tee_defconfig
    |  |
    |  +--- package/
    |  |  +--- (libteec/ mips-tee-client.git => mips.com repository)
    |  |  +--- (mips-tee-test/ mips-tee-test.git => mips.com repository)
    |  |     +--- clientapps/
    |  |     +--- xtest/
    |  |
    |  +--- Config.in
    |  +--- external.mk
    |  +--- external.desc
    |
    +--- scripts/
    |  +--- .mips_tee_mrconfig
    |  +--- do-mips-tee-mrconfig
    |
    +--- tools/
    |  +--- bin/
    |  |  +--- (mr => myrepos.branchable.com external repository)
    |  +--- (myrepos => myrepos.branchable.com external repository)
    |
    +--- prebuilt/ (optional)
       +--- toolchain/
