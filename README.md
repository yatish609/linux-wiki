# LINUX WIKI

## Index
1. [KVM](#kvm)
2. [Sound Improvements](#sound-improvements)
3. [VAAPI](#vaapi)
4. [Proprietary Bluetooth Codecs](#proprietary-bluetooth-codecs)
5. [ZSH](#zsh)
6. [Bluetooth Pairing for Dual Boot](#bluetooth-pairing-for-dual-boot)

## KVM

### GPU Passthrough Guide
1. Get GPU ids from "sudo lspci -nn" (will be around 2). Save these ids somewhere.
2. Install the following packages:
~~~
sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager ovmf
~~~
3. Add kernel modules:
For grub system:
~~~
sudo nano /etc/default/grub

Add the following modules at the end:
GRUB_CMDLINE_LINUX_DEFAULT= "... intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=XXXX:XXXX,XXXX:XXXX"
~~~

For systemd-boot system:
~~~
sudo nano /boot/efi/loader/entries/loader.conf (there might be some different name of loader.conf, e.g, PopOS has Pop_OS-current.conf)

Add the following modules at the end of "options" line:
intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=XXXX:XXXX,XXXX:XXXX
~~~

XXXX:XXXX are GPU ids that we acquired in step 1. Make sure not to leave any space in-between the comma.

Source and useful links:
https://www.youtube.com/watch?v=tDMoEvf8Q18&t=523s (Detailed video guide)

### Guest Config
Under CPUs, you should customize the topology, i.e. with my 8 Cores (4 real cores + 4 HT), you should set:
  - Sockets: 1
  - Cores: 4
  - Threads: 2
  - Then change current allocation (back to) to 8
- Under VirtIO Disk 1, change from SATA to VirtIO (increases disk throughput)
  - To get the Disk recognized under the Windows 10 installer, use an additional DVD drive
    with the VirtIO ISO (https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html)
    Then, in the Windows 10 installer-menu under the disks / media section, use "Load Drivers"; then the disk is recognized
- Switch off "scale display" under "View" -> "Scale Display" -> Never
- For a long time I thought that the blurry fonts in the VM were a result of the poor graphics performance but it was the VM trying to ajust the scaling.

### SPICE integration and performance steps

#### For Linux Guests:

1. To enable copy paste from host to guest, make sure to install "spice-vdagent".

2. To enable and improve SPICE QXL performance in guests, install "xserver-xorg-video-qxl". This will enable guest dynamic resizing and higher resolution with potentially better performance. 

3. Load kernel modules "qxl" and "bochs_drm" in order to gain a decent performance. This can be done by add the above modules in grub file which can be found at /etc/default/grub on grub booted systems.


Sources and useful links:  
https://wiki.archlinux.org/index.php/QEMU#qxl  
https://www.spice-space.org/download.html  
https://askubuntu.com/questions/858649/how-can-i-copypaste-from-the-host-to-a-kvm-guest  

### Host QEMU/KVM Wiki

#### Installation:

##### For Arch or Arch-based distributions:
1. Install the following packages:
~~~
sudo pacman -S virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat libguestfs
~~~
2. Enable libvirtd service:
~~~
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
~~~

3. For the first time, run virt-manager as root. Check if KVM/QEMU is connected.
If it shows only LXC as connection, go to File > Add Connection and choose QEMU/KVM as hypervisor and click add connection.

4. Import your previous existing machines or create new machines as required.

5. After reboot, if virt manager shows error "network 'default' is not active", click on any VM, then go to Edit > Connection Details > Virtual Networks and enable Autostart on boot option. Then reboot and it should work fine.

6. If virt-manager asks you to enter root password everytime, add yourself to the libvirt group:
~~~
sudo usermod -a -G libvirt $(whoami)
~~~
(Members of the libvirt group have the ability to manage virtual machines)

Useful links:  
https://wiki.manjaro.org/index.php/Virt-manager  
https://www.reddit.com/r/Fedora/comments/c0frj7/why_does_virtmanager_require_sudo_privileges/ (1st reply)  
https://github.com/kubernetes/minikube/issues/828 (1st reply)  


##### For Debian or debian-based distributions:

1. Install virt-manager using:
~~~
sudo apt install virt-manager bridge-utils
~~~


## Sound Improvements
### PulseAudio Improvements

Default linux config provides average sound experience. To improve sound quality on linux, certain pulseaudio parameters can be changed. 

Recommended Parameteric values:
* default-sample-format = float32le
* resample-method = soxr-vhq
* realtime-priority = 9
* default-sample-rate = 48000
* alternate-sample-rate = 44100

The following commands automatically apply the above recommended values to the pulseaudio file named daemon.conf:

    ~~~
    sudo sed -i 's/default-sample-format = s16le/default-sample-format = float32le/g' /etc/pulse/daemon.conf
    sudo sed -i 's/resample-method = speex-float-1/resample-method = soxr-vhq/g' /etc/pulse/daemon.conf
    sudo sed -i 's/realtime-priority = 5/realtime-priority = 9/g' /etc/pulse/daemon.conf
    sudo sed -i 's/default-sample-rate = 44100/default-sample-rate = 48000/g' /etc/pulse/daemon.conf
    sudo sed -i 's/alternate-sample-rate = 48000/alternate-sample-rate = 44100/g' /etc/pulse/daemon.conf
    ~~~

## Proprietary Bluetooth Codecs
Adds AptX, AptX HD, LDAC and AAC support in Linux. Greatly improves sound quality.

GitHub Repo Link - https://github.com/EHfive/pulseaudio-modules-bt

### Installation Instructions

#### Arch

    ~~~
    yay -S --noconfirm pulseaudio-modules-bt
    ~~~

#### Debian

    ~~~
    sudo add-apt-repository ppa:berglh/pulseaudio-a2dp
    sudo apt update
    sudo apt install pulseaudio-modules-bt libldac
    ~~~

## ZSH

### Installation


