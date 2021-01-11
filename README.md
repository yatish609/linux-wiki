# LINUX WIKI

## Index
1. [KVM](#kvm)
    * [Host config](#host-config)
    * [Guest config](#guest-config)
    * [SPICE Integration](#spice-integration)
    * [GPU Passthrough](#gpu-passthrough)
2. [Sound Improvements](#sound-improvements)
    * [Pulseaudio Improvements](#pulseaudio-improvements)
    * [Proprietary Bluetooth Codecs](#proprietary-bluetooth-codecs)
3. [Bluetooth Pairing for Dual Boot](#bluetooth-pairing-for-dual-boot)
4. [VAAPI](#vaapi)
5. [ZSH](#zsh)

## KVM

### Host Config
------

#### For Arch or Arch-based distributions:
1. Install the following packages:
    ```
    sudo pacman -S virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat libguestfs
    ```
2. Enable libvirtd service:
    ```
    sudo systemctl enable libvirtd.service
    sudo systemctl start libvirtd.service
    ```

3. For the first time, run virt-manager as root. Check if KVM/QEMU is connected.
If it shows only LXC as connection, go to File > Add Connection and choose QEMU/KVM as hypervisor and click add connection.

4. Import your previous existing machines or create new machines as required.

5. After reboot, if virt manager shows error "network 'default' is not active", click on any VM, then go to Edit > Connection Details > Virtual Networks and enable Autostart on boot option. Then reboot and it should work fine.

6. If virt-manager asks you to enter root password everytime, add yourself to the libvirt group:
    ```
    sudo usermod -a -G libvirt $(whoami)
    ```

    (Members of the libvirt group have the ability to manage virtual machines)

#### For Debian or debian-based distributions:

1. Install virt-manager using:
    ```
    sudo apt install qemu-kvm virt-manager bridge-utils
    ```

Useful links:  
https://wiki.manjaro.org/index.php/Virt-manager  
https://www.reddit.com/r/Fedora/comments/c0frj7/why_does_virtmanager_require_sudo_privileges/ (1st reply)  
https://github.com/kubernetes/minikube/issues/828 (1st reply)  

### Guest Config
------

Under CPUs, you should customize the topology, i.e. with my 8 Cores (4 real cores + 4 HT), you should set:
  - Sockets: 1
  - Cores: 4
  - Threads: 2
  - Then change current allocation (back to) to 8
- Under VirtIO Disk 1, change from SATA to VirtIO (increases disk throughput)
  - To get the Disk recognized under the Windows 10 installer, use an additional DVD drive
    with the VirtIO ISO (https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html)
    Then, in the Windows 10 installer-menu under the disks / media section, use "Load Drivers"; then the disk is recognized
- Switch off "scale display" under "View" -> "Scale Display" -> Never (This is to prevent blurry fonts in the VM)


### SPICE Integration
------
#### For Linux Guests:

1. To enable copy paste from host to guest, make sure to install `spice-vdagent`.

2. To enable and improve SPICE QXL performance in guests, install `xserver-xorg-video-qxl. This will enable guest dynamic resizing and higher resolution with potentially better performance. 

3. Load kernel modules `qxl` and `bochs_drm` in order to gain a decent performance. This can be done by add the above modules in grub file which can be found at `/etc/default/grub` on grub booted systems.


Sources and useful links:  
https://wiki.archlinux.org/index.php/QEMU#qxl  
https://www.spice-space.org/download.html  
https://askubuntu.com/questions/858649/how-can-i-copypaste-from-the-host-to-a-kvm-guest  


### GPU Passthrough
------

1. Get GPU ids from "sudo lspci -nn" (will be around 2). Save these ids somewhere.
2. Install the following packages:
    ```
    sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager ovmf
    ```
3. Add kernel modules:
   * For grub system:
            
        ```
        sudo nano /etc/default/grub

        Add the following modules at the end:
        GRUB_CMDLINE_LINUX_DEFAULT= "... intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=XXXX:XXXX,XXXX:XXXX"
         ```

   * For systemd-boot system:

        ```
        sudo nano /boot/efi/loader/entries/loader.conf (there might be some different name of loader.conf, e.g, PopOS has Pop_OS-current.conf)

        Add the following modules at the end of "options" line:
        intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=XXXX:XXXX,XXXX:XXXX
        ```

     XXXX:XXXX are GPU ids that we acquired in step 1. Make sure not to leave any space in-between the comma.

    Source and useful links:
    https://www.youtube.com/watch?v=tDMoEvf8Q18&t=523s (Detailed video guide)


## Sound Improvements
### PulseAudio Improvements
------

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


### Proprietary Bluetooth Codecs
------
Adds AptX, AptX HD, LDAC and AAC support in Linux. Greatly improves sound quality.

GitHub Repo Link - https://github.com/EHfive/pulseaudio-modules-bt

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


## Bluetooth Pairing for Dual Boot

Basically, when you pair your device, your Bluetooth service generates a unique set of pairing keys. First, your computer stores the Bluetooth device's MAC address and pairing key. Second, your Bluetooth device stores your computer's MAC address and the matching key. This usually works fine, but the MAC address for your Bluetooth port will be the same on both Linux and Windows (it is set on the hardware level). Thus, when you re-pair the device in Windows or Linux and it generates a new key, that key overwrites the previously stored key on the Bluetooth device. Windows overwrites the Linux key and vice versa.

### How to fix
Using the instructions below, we'll first pair your Bluetooth devices with Ubuntu/Linux Mint, and then we'll pair Windows. Then we'll go back into our Linux system and copy the Windows-generated pairing key(s) into our Linux system.

1. Pair all devices w/ Mint/Ubuntu
2. Pair all devices w/ Windows
3. Copy your Windows pairing keys in one of two ways:
    * Use `psexec -s -i regedit.exe` from Windows (harder)

        1. Go to "Device & Printers" in Control Panel and go to your Bluetooth device's properties. Then, in the Bluetooth section, you can find the unique identifier. Copy that (you will need it later).
        2. Download PsExec from http://technet.microsoft.com/en-us/sysinternals/bb897553.aspx.
        3. Unzip the zip you downloaded and open a cmd window with elevated privileges. (Click the Start menu, search for cmd, then right-click the CMD and click "Run as Administrator".)
        4. cd into the folder where you unzipped your download.
        5. Run `psexec -s -i regedit.exe`
        6. Navigate to find the keys at `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\BTHPORT\Parameters\Keys`.  If there is no CurrentControlSet, try ControlSet001.
        7. You should see a few keys labels with the MAC addresses - write down the MAC address associated with the unique identifier you copied before.

    * Use `chntpw` from your Linux distro (easier). Start in a terminal then:

        1. `sudo apt-get install chntpw`

        2. Mount your Windows system drive

        3. `cd /[WindowsSystemDrive]/Windows/System32/config`

        4. `chntpw -e SYSTEM` opens a console

        5. Run these commands in that console:

            ```

            > cd CurrentControlSet\Services\BTHPORT\Parameters\Keys
            > # if there is no CurrentControlSet, then try ControlSet001
            > # on Windows 7, "services" above is lowercased.
            > ls
            # shows you your Bluetooth port's MAC address
            Node has 1 subkeys and 0 values
              key name
              <aa1122334455>
            > cd aa1122334455  # cd into the folder
            > ls  
            # lists the existing devices' MAC addresses
            Node has 0 subkeys and 1 values
              size     type            value name             [value if type DWORD]
                16  REG_BINARY        <001f20eb4c9a>
            > hex 001f20eb4c9a
            => :00000 XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX ...ignore..chars..
            # ^ the XXs are the pairing key

            ```

        6. Make a note of which Bluetooth device MAC address matches which pairing key. The Mint/Ubuntu one won't need the spaces in-between.  Ignore the :00000.

4. Go back to Linux (if not in Linux) and add the Windows key to our Linux config entries. Note that the Bluetooth port's MAC address is formatted differently when moving from Windows to Linux - referenced as aa1122334455 in Windows as in the example above. The Linux version will be in all caps and punctuated by ':' after every two characters - for example AA:11:22:33:44:55.

    1. Switch to root: `sudo su`

    2. cd to your Bluetooth config location `/var/lib/bluetooth/[bth port  MAC addresses]`

    3. Here you'll find folders for each device you've paired with. The folder names being the Bluetooth devices' MAC addresses and contain a single file `info`. In these files, you'll see the link key you need to replace with your Windows ones, like so:

        ```
            [LinkKey]
            Key=B99999999FFFFFFFFF999999999FFFFF
        ```

5. Once updated, restart your Bluetooth service in one of the following ways, and then it works!
    * Either restart bluetooth: `sudo systemctl restart Bluetooth`
    * Alternatively, reboot your machine into Linux.

6. Reboot into Windows - it works!



## VAAPI

|    Browser    | VAAPI Support |
| ------------- | ------------- |
| Google Chrome | :negative_squared_cross_mark:  |
| Chromium  | :heavy_check_mark:  |




## ZSH

### Installation
