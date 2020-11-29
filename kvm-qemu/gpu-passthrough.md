# GPU Passthrough Guide

1. Get GPU ids from "sudo lspci -nn" (will be around 2). Save these ids somewhere.
2. Install the following packages:
~
sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager ovmf
~
3. Add kernel modules:
For grub system:
~
sudo nano /etc/default/grub

Add the following modules at the end:
GRUB_CMDLINE_LINUX_DEFAULT= "... intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=XXXX:XXXX,XXXX:XXXX"
~

For systemd-boot system:
~
sudo nano /boot/efi/loader/entries/loader.conf (there might be some different name of loader.conf, e.g, PopOS has Pop_OS-current.conf)

Add the following modules at the end of "options" line:
intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=XXXX:XXXX,XXXX:XXXX"
~

XXXX:XXXX are GPU ids that we acquired in step 1. Make sure not to leave any space in-between the comma.

Source and useful links:
https://www.youtube.com/watch?v=tDMoEvf8Q18&t=523s (Detailed video guide)
