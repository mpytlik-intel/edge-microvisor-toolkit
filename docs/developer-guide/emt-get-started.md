# Get Started

Edge Microvisor Toolkit is a lightweight, container-first Linux distribution,
optimized for Intel® architecture. It provides a secure and high-performing
environment for deploying edge workloads across multiple deployment models.

This section provides an overview of both the operating system and build
pipelines. Once you have decided on the usage scenarios presented below, you can
move on to:

- [Build a new Edge Microvisor Toolkit Image.](./get-started/emt-building-howto.md)
- [Install Edge Microvisor Toolkit from existing image.](./get-started/emt-installation-howto.md)

## Usage Scenarios

This section outlines the key usage models intended for the initial release of
Edge Microvisor Toolkit.

It can be used for standalone edge node deployments, or with Edge Manageability
Framework - a complete integrated system providing full lifecycle management for
your edge devices, including remote deployment and management of Kubernetes
applications.

### Build Your Own Edge Microvisor Toolkit

Edge Microvisor Toolkit is a downstream of Azure Linux. It is composed of multiple modules to
facilitate creating `rpm` based OS images supporting a variety of different image formats.

The toolkit has an `imageconfig` construct in the JSON format that defines the characteristics
of the resulting image, such as:

- Type and size of partitioning table.
- Partitions, their types (such as EFI, rootfs, etc.), settings, file system, and size.
- Reference to `packagelists` which defines what packages (i.e. `rpms`) should be included in
  the image.
- Additional configuration files that should be embedded in the image (e.g. network-, systemd
  configurations).
- Any required post-installation scripts that should be executed once the image has been
  generated.
- Kernel and command line options.
- Final configuration properties that should be applied (e.g. enable full disc encryption,
  immutable image, second stage bootloader provider, purge documentation etc.).

#### Build the Toolchain

Before you can build OS images you need to build the toolchain and make sure to
[**install pre-requisites (Ubuntu)**](https://github.com/open-edge-platform/edge-microvisor-toolkit/blob/3.0/toolkit/docs/building/prerequisites-ubuntu.md).

> **Note:**
  Use the *stable* tag instead of *latest* for building the OS images with prebuilt packages.
  This is the recommended approach, as building the **entire toolchain** may take a lot of
  time. Adding the `REBUILD_TOOLCHAIN=y` parameter to the `make` command rebuilds
  the entire toolchain.


1. Clone the stable branch of the Edge Microvisor Toolkit repository.

   Check the [tags](https://github.com/open-edge-platform/edge-microvisor-toolkit/tags) for
   the `<stable_tag_name>`.

   ```bash
   git clone https://github.com/open-edge-platform/edge-microvisor-toolkit --branch=<stable_tag_name>
   ```

2. Navigate to the `toolkit` subdirectory.

   ```bash
   cd edge-microvisor-toolkit/toolkit
   ```

3. Build the tools.

   ```bash
   sudo make toolchain REBUILD_TOOLS=y
   ```

#### Build the Edge Microvisor Toolkit Image

Multiple image configurations are located in the `imageconfigs` folder:

```bash
microvisor/
├── docs/
├── LICENSES-AND-NOTICES/
├── SPECS/
├── SPECS-EXTENDED/
├── SPECS-SIGNED/
└── toolkit/
    ├── docs/
    └── imageconfigs/
      ├── edge-image-dev.json
      ├── edge-image-rt-dev.json
      ├── edge-image-rt.json
      ├── edge-image.json
      ...
      └──
  ...
```

Different image types can be built by using different JSON config files and parameters.
You can find more information about specific parameters [here.](https://github.com/open-edge-platform/edge-microvisor-toolkit/blob/3.0/toolkit/docs/building/building.md#local-build-variables)

To build an ISO image, run the following command:

```bash
sudo make iso -j8 REBUILD_TOOLS=y REBUILD_PACKAGES=n CONFIG_FILE=./imageconfigs/full.json
```

To build a RAW image without real-time extensions, run the following command:


```bash
sudo make image -j8 REBUILD_TOOLS=y REBUILD_PACKAGES=n CONFIG_FILE=./imageconfigs/edge-image.json
```

To build a RAW image with real-time extensions, use the following command:

```bash
RT build command: sudo make image -j8 REBUILD_TOOLS=y REBUILD_PACKAGES=n CONFIG_FILE=./imageconfigs/edge-image-rt.json
```

#### Customize Your Edge Microvisor Toolkit Image

To add packages to the default image, you can define your own `packagelist.json` file,
pointing to `rpms` that should be included in the image. The `edge-image.json` file points to
multiple `packagelist` files, located under `imageconfigs/packagelists`. The same `rpms` may
be included in an `imageconfig` file through the `packagelist` files.

The resulting image will include the set of all `rpms` specified within the array of
`packagelist` files from the `imageconfig`.

##### Example 1: Adding an existing RPM (Nano)

Note that you can only add the packages for which SPEC files exist. To add `nano` as an
alternative text editor to the image:

1. Define a new JSON file.

   ```bash
   # Create a new packagelist called utilities.json
   cat <<EOF > ./imageconfigs/packagelists/utilities.json
   {
       "packages": [
           "nano"
       ]
   }
   EOF
   ```

2. Include it in an existing `imageconfig` JSON file, for example `edge-image.json`.
   You can also create a new file, for example `edge-image-custom.json` and add it to the `imageconfigs` folder.

   ```bash
   # Edit the edge-image.json file. Add the custom packagelist and default login account for testing.
   ...
   "PackageLists": [
     "packagelists/core-packages-image-systemd-boot.json",
     "packagelists/ssh-server.json",
     "packagelists/virtualization-host-packages.json",
     "packagelists/agents-packages.json",
     "packagelists/tools-tinker.json",
     "packagelists/persistent-mount-package.json",
     "packagelists/fde-verity-package.json",
     "packagelists/selinux-full.json",
     "packagelists/intel-gpu-base.json",
     "packagelists/os-ab-update.json",
     "packagelists/utilities.json"
   ],
   "Users": [
     {
         "Name": "user",
         "Password": "user"
     }
   ],
   ...
   ```

3. Rebuild the image:

   ```bash
   sudo make image -j8 REBUILD_TOOLS=y REBUILD_PACKAGES=n CONFIG_FILE=./imageconfigs/edge-image.json
   ```

##### Example 2: Adding a new RPM package

To add a new package you need to generate for the package a SPEC file containing
all information required for the build infrastructure to generate `SRPM` and `RPM`
for the package. There are a few steps involved in creating a new package for Edge Microvisor Toolkit.

**Prerequisites**

Make sure you have the required build tools for `rpm`.
On Fedora, you can simply install the required packages with:

```bash
sudo dnf install rpm-build rpmdevtools
rpmdev-setuptree
```

where `rpmdev-setuptree` creates the necessary directories.

On Ubuntu, use the following command:

```bash
sudo apt-get install rpm
```

**Preparing the files**

1. Manually create the necessary directories:

   ```bash
   mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
   echo '%_topdir %(echo $HOME)/rpmbuild' > ~/.rpmmacros
   ```

2. Navigate to user home directory.

   ```bash
   cd
   ```

3. Define the SPEC file, using the example below.

   It will create a simple hello world RPM package, which will include a bash script that
   prints *"Hello, world!"*.

   ```bash
   Name:           helloworld
   Version:        1.0
   Release:        1%{?dist}
   Summary:        Simple Hello World script

   License:        MIT
   URL:            https://example.com/helloworld
   Source0:        helloworld-1.0.tar.gz

   BuildArch:      noarch

   %description
   A very basic "Hello, world!" script packaged as an RPM.

   %prep
   %setup -q

   %build
   # Nothing to build for a shell script

   %install
   mkdir -p %{buildroot}/usr/bin
   install -m 0755 helloworld.sh %{buildroot}/usr/bin/helloworld

   mkdir -p %{buildroot}/usr/share/helloworld
   install -m 0644 helloworld.signature.json %{buildroot}/usr/share/helloworld/

   %files
   /usr/bin/helloworld
   /usr/share/helloworld/helloworld.signature.json

   %changelog
   * Wed May 01 2025 Your Name <you@example.com> - 1.0-1
   - Initial package
   ```

4. Create the simple script and make it executable.

   ```bash
   mkdir -p ./helloworld-1.0
   cat > ./helloworld-1.0/helloworld.sh <<'EOF'
   #!/bin/bash
   echo "Hello, world!"
   EOF
   chmod +x ./helloworld-1.0/helloworld.sh
   ```

**Create the source archive and generate the sha256sum for the package.**

1. Compute the SHA-256 and generate the JSON signature for it.

   ```bash
   sum=$(sha256sum ./helloworld-1.0/helloworld.sh | awk '{print $1}')
   cat > ./helloworld-1.0/helloworld.signature.json <<EOF
   {
     "file": "helloworld.sh",
     "sha256": "$sum"
   }
   EOF
   ```

2. Create the tarball archive and generate its JSON signature.

   ```bash
   tar -czf helloworld-1.0.tar.gz ./helloworld-1.0
   sum=$(sha256sum helloworld-1.0.tar.gz | awk '{print $1}')
   cat > helloworld-1.0.tar.gz.signature.json <<EOF
   {
     "file": "helloworld-1.0.tar.gz",
     "sha256": "$sum"
   }
   EOF
   ```

3. Update the `cgmanifest.json` file.
4. Build an image with the package included and test locally.
5. Upload the tar.gz package to the source package repository after is has been tested locally.









4. Copy the RPM package files to the building directories and build it.

   ```bash
   cp helloworld-1.0.tar.gz ./rpmbuild/SOURCES
   cp helloworld.spec ./rpmbuild/SPECS
   rpmbuild -ba ./rpmbuild/SPECS/helloworld.spec
   ```

**Adding the package**

1. Create the `helloworld` folder in the `edge-microvisor-toolkit/SPECS` directory.

   ```bash
   mkdir ./edge-microvisor-toolkit/SPECS/helloworld
   ```

2. Copy the `helloworld.spec` and `helloworld.signature.json` files to the
   `helloworld` folder.

   ```bash
   cp ./helloworld.spec ./edge-microvisor-toolkit/SPECS/helloworld
   cp ./helloworld-1.0/helloworld.signature.json ./edge-microvisor-toolkit/SPECS/helloworld
   ```

3. Finally, update the `cgmanifest` by using the provided `python` script.

   ```bash
       cd ./edge-microvisor-toolkit/toolkit
       python3 -m pip install -r ./scripts/requirements.txt
       python3 ./scripts/update_cgmanifest.py first ../cgmanifest.json ../SPECS/helloworld.spec
   ```

<!--### Edge Microvisor Toolkit Developer Node

To create a custom developer build of Edge Microvisor Toolkit, follow these steps:

- [Download the mutable host ISO image](https://files-rs.edgeorchestration.intel.com/files-edge-orch/microvisor/iso/EdgeMicrovisorToolkit-3.0.iso) from
  Intel® Edge Software Catalog.
- Install the mutable host via ISO image. Choose one of several installation types that
  provide ready-to-use base environments:
  - **Standard Kernel** - that includes only essential pre-installed packages,
  - **RT Kernel** - that offers enhanced real-time performance with
    [Preempt RT Linux Kernel 6.12](./emt-architecture-overview.md#preempt-rt-kernel),
  - **Standard Kernel + Docker + K3s** - that is fitted with additional features:
    - Docker 25.07 - for deploying applications in lightweight and standalone containers,
    - K3s - Lightweight Kubernetes 1.32.4, installed as an RPM package, for a simplified
      deployment on resource-constrained edge devices,
  - **RT Kernel + Docker + K3s** - that supports real-time workloads and offers small
    footprint deployment.
- Install additional RPM packages, using DNF to tailor the OS to your specific needs.
- Update installed RPMs regularly to stay up-to-date in the OS in terms of package updates,
  kernel updates, security vulnerability fixes and bug fixes.
- Use the OS toolkit and available packages to build a custom OS image, which enables you to:
  - Configure the system for specialized workloads or environments.
  - Experiment with simplified or enhanced configurations tailored for your specific workloads.
  - Explore - use built-in monitoring tools to track system performance, resource
    usage, and log data for deeper insights into operational behavior.

| Item              | Details                                         |
| ------------------| ----------------------------------------------- |
| Packages          | approximately ~400                              |
| Core system tools | bash, coreutils, util-linux, tar, gzip          |
| Networking        | curl, wget, iproute2, iptables, openssh         |
| Package Management | tdnf, rpm                                      |
| Development       | gcc, make, python3, perl, cmake, git            |
| Security          | openssl, gnupg, selinux, cryptsetup, tpm2-tools |
| Filesystem        | e2fsprogs, mount                                |
| Included in kernel | iGPU, dGPU (Intel® Arc&trade;), SR-IOV, WiFi, Ethernet, Bluetooth, GPIO, UART, I2C, CAN, USB, PCIe, PWM, SATA, NVMe, MMC/SD, TPM, Manageability Engine, Power Management, Watchdog, RAS |

The supported package repository offers additional `rpm` for tailoring the image
to specific needs of container runtime, virtualization, orchestration software,
monitoring tools, standard cloud-edge (CNCF) software, and more.

### Edge Microvisor Toolkit Standalone Node

[Go to the Edge Microvisor Toolkit Standalone Node repository](https://github.com/open-edge-platform/edge-microvisor-toolkit-standalone-node).

### Edge Microvisor Toolkit with Edge Manageability Framework

Edge Microvisor Toolkit supports deployment of its two versions with Edge
Manageability Framework:

- Microvisor Immutable Image
- Microvisor Immutable Image with Real Time

For details on deploying Microvisor with Edge Manageability Framework, refer to
the [Edge Manageability Framework deployment guide](./emt-deployment-edge-orchestrator.md).

## Image Support

The toolkit comes pre-configured to produce different images, the table below
outlines the key differences between those.

|  Feature         | Edge Microvisor Toolkit Developer Node | Edge Microvisor Toolkit Standalone Node & Orchestrated                                   |
| -----------------| -------------------- | ------------------------------------------------- |
| Capabilities | <ul><li>Easy to install, bootable ISO image with precompiled packages for developer evaluation.</li> <li> Includes installable rpms with TDNF for extending baseline functionality.</li> <li>Complete with toolkit to build image with an opt-in data integrity and security features.</li></ul> | <ul><li>Designed for Open Edge Platforms and can be used to onboard and provision edge nodes at scale.</li><li>Can be used independently on bare-metal and as guest OS.</li><li>Fast atomic updates & rollback support with small image footprint and short boot time.|
| Image Type       | Mutable ISO          | Immutable RAW + VHD                               |
| Update Mechanism | RPM package updates with TDNF | Image based A/B updates + Rollback       |
| Linux Kernel     | Intel® Kernel 6.12   | Intel® Kernel 6.12                                |
| Real time        | Available for opt-in | Two images provided: one with RT kernel and one without |
| Add-on packages  | Available for opt-in: Docker + K3s | Built into image: Docker + K3s |
| OS Bootloader    | GRUB                 | systemd-boot                                      |
| Secure Boot      | Available for opt-in | Enabled                                           |
| Full Disc Encryption | Available for opt-in | Enabled                                       |
| dm-verity        | Available for opt-in | Enabled                                           |
| SELinux          | Permissive           | Permissive                                        |

## Next Steps

- [System Requirements](./emt-system-requirements)
- [Production Deployment with Edge Manageability Framework](./emt-deployment-edge-orchestrator.md)

:::{toctree}
./get-started/emt-building-howto.md
./get-started/emt-installation-howto.md
./get-started/emt-sb-howto.md
:::
-->