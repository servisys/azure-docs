---
title: Prepare an Debian Linux VHD 
description: Learn how to create Debian VHD images for VM deployments in Azure.
author: gbowerman
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 11/13/2018
ms.author: guybo

---
# Prepare a Debian VHD for Azure
## Prerequisites
This section assumes that you have already installed a Debian Linux operating system from an .iso file downloaded from the [Debian website](https://www.debian.org/distrib/) to a virtual hard disk. Multiple tools exist to create .vhd files; Hyper-V is only one example. For instructions using Hyper-V, see [Install the Hyper-V Role and Configure a Virtual Machine](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## Installation notes
* See also [General Linux Installation Notes](create-upload-generic.md#general-linux-installation-notes) for more tips on preparing Linux for Azure.
* The newer VHDX format is not supported in Azure. You can convert the disk to VHD format using Hyper-V Manager or the **convert-vhd** cmdlet.
* When installing the Linux system, it is recommended that you use standard partitions rather than LVM (often the default for many installations). This will avoid LVM name conflicts with cloned VMs, particularly if an OS disk ever needs to be attached to another VM for troubleshooting. [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) or [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid) may be used on data disks if preferred.
* Do not configure a swap partition on the OS disk. The Azure Linux agent can be configured to create a swap file on the temporary resource disk. More information can be found in the steps below.
* All VHDs on Azure must have a virtual size aligned to 1MB. When converting from a raw disk to VHD, you must ensure that the raw disk size is a multiple of 1MB before conversion. For more information, see [Linux Installation Notes](create-upload-generic.md#general-linux-installation-notes).

## Use Azure-Manage to create Debian VHDs
There are tools available for generating Debian VHDs for Azure, such as the [azure-manage](https://github.com/credativ/azure-manage) scripts from [Credativ](https://www.credativ.com/). This is the recommended approach versus creating an image from scratch. For example, to create a Debian 8 VHD run the following commands to download the `azure-manage` utility (and dependencies) and run the `azure_build_image` script:

```console
# sudo apt-get update
# sudo apt-get install git qemu-utils mbr kpartx debootstrap

# sudo apt-get install python3-pip python3-dateutil python3-cryptography
# sudo pip3 install azure-storage azure-servicemanagement-legacy azure-common pytest pyyaml
# git clone https://github.com/credativ/azure-manage.git
# cd azure-manage
# sudo pip3 install .

# sudo azure_build_image --option release=jessie --option image_size_gb=30 --option image_prefix=debian-jessie-azure section
```


## Manually prepare a Debian VHD
1. In Hyper-V Manager, select the virtual machine.
2. Click **Connect** to open a console window for the virtual machine.
3. If you installed the OS using an ISO, then comment out any line relating to "`deb cdrom`" in `/etc/apt/source.list`.

4. Edit the `/etc/default/grub` file and modify the **GRUB_CMDLINE_LINUX** parameter as follows to include additional kernel parameters for Azure.

    ```config-grub
    GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8 earlyprintk=ttyS0,115200"
    ```

5. Rebuild the grub and run:

    ```console
    # sudo update-grub
    ```

6. Add Debian's Azure repositories to /etc/apt/sources.list for either Debian 8, 9 or 10:

    **Debian 8.x "Jessie"**

    ```config-grub
    deb http://debian-archive.trafficmanager.net/debian jessie main
    deb-src http://debian-archive.trafficmanager.net/debian jessie main
    deb http://debian-archive.trafficmanager.net/debian-security jessie/updates main
    deb-src http://debian-archive.trafficmanager.net/debian-security jessie/updates
    deb http://debian-archive.trafficmanager.net/debian jessie-updates main
    deb-src http://debian-archive.trafficmanager.net/debian jessie-updates main
    deb http://debian-archive.trafficmanager.net/debian jessie-backports main
    deb-src http://debian-archive.trafficmanager.net/debian jessie-backports main
    ```

    **Debian 9.x "Stretch"**

    ```config-grub
    deb http://debian-archive.trafficmanager.net/debian stretch main
    deb-src http://debian-archive.trafficmanager.net/debian stretch main
    deb http://debian-archive.trafficmanager.net/debian-security stretch/updates main
    deb-src http://debian-archive.trafficmanager.net/debian-security stretch/updates main
    deb http://debian-archive.trafficmanager.net/debian stretch-updates main
    deb-src http://debian-archive.trafficmanager.net/debian stretch-updates main
    deb http://debian-archive.trafficmanager.net/debian stretch-backports main
    deb-src http://debian-archive.trafficmanager.net/debian stretch-backports main
    ```
    
    **Debian 10.x "Buster"**
    ```config-grub
    deb http://debian-archive.trafficmanager.net/debian buster main
    deb-src http://debian-archive.trafficmanager.net/debian buster main
    deb http://debian-archive.trafficmanager.net/debian-security buster/updates main
    deb-src http://debian-archive.trafficmanager.net/debian-security buster/updates main
    deb http://debian-archive.trafficmanager.net/debian buster-updates main
    deb-src http://debian-archive.trafficmanager.net/debian buster-updates main
    deb http://debian-archive.trafficmanager.net/debian buster-backports main
    deb-src http://debian-archive.trafficmanager.net/debian buster-backports main
    ```

7. Install the Azure Linux Agent:

    ```console
    # sudo apt-get update
    # sudo apt-get install waagent
    ```

8. For Debian 9+, it is recommended to use the new Debian Cloud kernel for use with VMs in Azure. To install this new kernel, first create a file called /etc/apt/preferences.d/linux.pref with the following contents:

    ```config-pref
    Package: linux-* initramfs-tools
    Pin: release n=stretch-backports
    Pin-Priority: 500
    ```

    Then run "sudo apt-get install linux-image-cloud-amd64" to install the new Debian Cloud kernel.

9. Deprovision the virtual machine and prepare it for provisioning on Azure and run:

    ```console
    # sudo waagent -deprovision
    # export HISTSIZE=0
    # logout
    ```

10. Click **Action** -> Shut Down in Hyper-V Manager. Your Linux VHD is now ready to be uploaded to Azure.

## Next steps
You're now ready to use your Debian virtual hard disk to create new virtual machines in Azure. If this is the first time that you're uploading the .vhd file to Azure, see [Create a Linux VM from a custom disk](upload-vhd.md#option-1-upload-a-vhd).
