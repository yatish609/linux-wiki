# SPICE integration and performance steps

## For Linux Guests:

1. To enable copy paste from host to guest, make sure to install "spice-vdagent".

2. To enable and improve SPICE QXL performance in guests, install "xserver-xorg-video-qxl". This will enable guest dynamic resizing and higher resolution with potentially better performance. 

3. Load kernel modules "qxl" and "bochs_drm" in order to gain a decent performance. This can be done by add the above modules in grub file which can be found at /etc/default/grub on grub booted systems.


Sources and useful links:
https://wiki.archlinux.org/index.php/QEMU#qxl
