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
