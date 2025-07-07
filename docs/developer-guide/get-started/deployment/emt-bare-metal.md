# Deploying Edge Microvisor Toolkit on Bare Metal

Below you will find all methods of deployment on Bare Metal using ISO or RAW images.

## Mutable ISO Deployment

### Requirements

You will need:

- [Edge Microvisor Toolkit Developer Node 3.0](https://files-rs.edgeorchestration.intel.com/files-edge-orch/microvisor/iso/EdgeMicrovisorToolkit-3.0.iso).
- USB flash drive (min. 8GB).
- Access to the target machine.
- Optional: Monitor and keyboard, or BMC/iDRAC/iKVM access.

### Create Bootable USB (Linux)

Follow the steps below to create a bootable USB device to install Edge Microvisor Toolkit
on your bare metal system.

Ensure you have the microvisor ISO file you want to flash saved on your system. Insert the
USB drive and identify it.

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

### Create Bootable USB (Windows)

On Windows, download and install ISO writer software such as [Rufus](https://rufus.ie/en).

1. Insert the USB device (8GB or more).
1. Launch Rufus.
1. Select the USB drive from the dropdown list.
1. Boot selection: Select your EMT 3.0 ISO file.
1. Image option: Leave default or choose *Standard Installation*.
1. Partition scheme: MBR (for legacy BIOS) or GPT (for UEFI).
1. File system: FAT32 (recommended).
1. Click *Start*.
1. Confirm warnings about data being erased.
1. Wait for completion and safely eject the USB.

### Boot and Install Edge Microvisor Toolkit

**Boot from USB**

1. Insert the USB into the target machine.
2. Enter the BIOS/Boot menu.
3. Choose the USB drive as the boot device.

**Select Installer**

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

## Immutable RAW deployment

### Requirements

You will need:

- Edge Microvisor Toolkit Standalone Node 3.0 RAW image.
- USB flash drive (min. 8GB).
- Access to the target machine.
- Optional: Monitor and keyboard, or BMC/iDRAC/iKVM access.

### Create Bootable USB

1. Build or download the RAW image

2. Unpack the RAW image:

   Run:

   ```bash
   gzip -d edge_microvisor_toolkit.raw.gz
   chmod -Rf 777 edge_microvisor_toolkit.raw
   ```

3. Flash the RAW image to a the USB flash drive using the 'dd' command.

   Run:

   ```bash
   sudo dd if=edge_microvisor_toolkit.raw of=/dev/sdc status=progress
   ```

   > **Note:** Successful flashing of the image should produce partitions such as /dev/sdb and /dev/sdc

### Boot and Install Edge Microvisor Toolkit RAW

1. Configure the server to reboot with required disk/OS/partition

   Using the CLI method, run `sudo efibootmgr`:

   ```bash
   BootCurrent: 0002
   BootOrder: 0002,0012,0014,0015
   Boot0002* ubuntu
   Boot0012  EFI Fixed Disk Boot Device 2
   Boot0014  Cruzer Blade
   Boot0015  NIC in Slot 2 Port 2 Partition 1
   MirroredPercentageAbove4G: 0.00
   MirrorMemoryBelow4GB: false
   ```

   Find the ID of EMT boot device and run `sudo efibootmgr -o <ID of EMT boot device>`. Then run `sudo reboot`

   You can also reboot and go to the boot manager to select the flashed partition:

   ![Partition selection UI](../assets/emt_flashing_raw_partitionrebootui_image-2024-8-1_13-3-1-1.png)

2. Check date

   Run `sudo date 080509312024`.

   The string of numbers after `date` is the date in time in the following format: Month:08 Day:05 Hour:09 Minute:31 Year: 2024


3. Configure and enable ssh

   Run:

   ```bash
   echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
   echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
   ```

4. Restart sshd service to apply changes

   Run `sudo systemctl restart sshd`.

   > **Note**: If you want to use the bootable storage device for other purposes, you can [allocate the remaining storage space to one of the partitions](./emt-flashing-raw-partition-resize.md)

### Troubleshooting (Best Known Methods)

- **BIOS Security Settings**

   Disable the Secure Boot option in BIOS Settings if it was enabled.

- **Network is not working in XR12 (with x710 NIC)**

1. Install the drivers manually:

   ```bash
   modprobe i40e
   ```

2. Update the ssh configuration to ssh with the following information:

   ```bash
   vi /etc/ssh/sshd_config

   PermitRootLogin yes

   PasswordAuthentication yes
   ```

3. Restart sshd service:

   ```bash
   systemctl restart sshd
   ```

   To debug, run only:

   ```bash
   journalctl -u sshd -f
   ```

- **Switching between multiple OS disks**

1. Configure efibootmgr. Run `sudo efibootmgr`:

   ```bash
   BootCurrent: 0000
   BootOrder: 0000,0005,0006,0002
   Boot0000* EFI Fixed Disk Boot Device 2
   Boot0002* ubuntu
   Boot0005* Cruzer Blade
   Boot0006* NIC in Slot 2 Port 2 Partition 1
   MirroredPercentageAbove4G: 0.00
   MirrorMemoryBelow4GB: false
   ```

2. Select the boot device with the desired OS and run `efibootmgr -o <ID of boot device>`

3. Reboot to change the OS to boot

   ```bash
   reboot
   ```