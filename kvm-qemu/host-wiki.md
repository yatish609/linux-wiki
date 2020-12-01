# Host QEMU/KVM Wiki

## Installation:

### For Arch or Arch-based distributions:
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


### For Debian or debian-based distributions:

1. Install virt-manager using:
~~~
sudo apt install virt-manager bridge-utils
~~~

