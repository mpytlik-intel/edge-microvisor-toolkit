# Deploying Edge Microvisor Toolkit on Bare Metal

Below you will find all methods of deployment on Bare Metal using ISO or RAW images.

## Requirements

You will need:

- ISO image of [Edge Microvisor Toolkit Developer Node 3.0](https://files-rs.edgeorchestration.intel.com/files-edge-orch/microvisor/iso/EdgeMicrovisorToolkit-3.0.iso).
- RAW image of Edge Microvisor Toolkit Standalone Node 3.0.
- USB flash drive (min. 8GB).
- Access to the target machine.
- Optional: Monitor and keyboard, or BMC/iDRAC/iKVM access.

## Create Bootable USB

Follow the steps below to create a bootable USB device to install Edge Microvisor Toolkit
on your bare metal system.

### Bootable USB from ISO

Ensure you have the microvisor ISO file you want to flash saved on your system. Insert the
USB drive and identify it.

#### Linux OS

```bash
lsblk
```

Compare the output before and after inserting your USB to identify its device name
(e.g., `/dev/sdb`). Flash the ISO Image. Use the `dd` command to write the ISO image.
Replace `/path/to/your.iso` with the ISO’s location and `/dev/sdb` with your USB device.

```bash
sudo dd if=/path/to/your.iso of=/dev/sdb bs=4M status=progress oflag=sync
# Warning: Double-check the device name. Using a wrong device can overwrite data.
```

Sync and Eject: Once `dd` has finished, run:

```bash
sudo sync
```

Then, safely remove the USB drive.

#### Windows OS

On Windows, download and install ISO writer software such as [Rufus](https://rufus.ie/en).

1. Insert the USB device (8GB or more).
2. Launch Rufus.
3. Select the USB drive from the dropdown list.
4. Boot selection: Select your EMT 3.0 ISO file.
5. Image option: Leave default or choose *Standard Installation*.
6. Partition scheme: MBR (for legacy BIOS) or GPT (for UEFI).
7. File system: FAT32 (recommended).
8. Click *Start*.
9. Confirm warnings about data being erased.
10. Wait for completion and safely eject the USB.


### Bootable USB from RAW

You can either [Build](../emt-building-howto.md) or download the RAW image that you will flash
on USB device used to install the immutable Standalone Node.


#### Linux OS

1. Navigate to the folder with the RAW image. Then, unpack the image by running the commands:

   ```bash
   gzip -d edge_microvisor_toolkit.raw.gz
   chmod -Rf 777 edge_microvisor_toolkit.raw
   ```

2. Flash the RAW image to a USB flash drive using the 'dd' command.

   ```bash
   sudo dd if=edge_microvisor_toolkit.raw of=/dev/sdc status=progress
   ```

   Or, you can choose to flash the USB device by using su

   > **Note:** Successful flashing of the image should produce partitions such as /dev/sdb and /dev/sdc

#### Windows OS



### Boot and Install Edge Microvisor Toolkit

#### Boot from USB

1. Insert the USB into the target machine.
2. Enter the BIOS/Boot menu.
3. Choose the USB drive as the boot device.

#### Install Edge Microvisor Toolkit Developer Node

1. Choose *Terminal Installer* or *Graphical Installer* when prompted

   ![Select installer](../../assets/01-select-installer.png)

   **Follow Installation Prompts**

2. Choose the installation type:

   ![Installation type](../../assets/02-installation-type.png).

3. Select the target disk for installation and choose the partitioning method.

   ![Partition](../../assets/03-partition-config.png).

4. Skip disk encryption (optional).
5. Create a username and a password. Keep the default *Hostname*.

   ![System config](../../assets/04-system-config.png).

6. Click *Install* and confirm by clicking *Install Now*.

7. When the installation has completed, click *Done* to close the installer.

   ![Complete](../../assets/05-install-complete.png).

   The system will reboot.

   **You are now ready to use Edge Microvisor Toolkit!**

### Post Installation Check

Check the version of Edge Microvisor Toolkit by running the following command:

```bash
cat /etc/os-release
```

#### Install Edge Microvisor Toolkit Standalone Node



