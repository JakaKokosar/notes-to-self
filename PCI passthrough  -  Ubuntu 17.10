# PCI passthrough on Ubuntu 17.10

Hardware:

 * Ryzen 1600x 
 * AsRock X370 TAICHI
 * Gigabyte GeForce GTX 1060 WINDFORCE OC 6G
 * MSI GeForce GTX 1030 2G
 * Corsair Vengeance LPX CMK16GX4M2B3200C16
 * Samsung 960 EVO (250GB)
 * Samsung 850 EVO (500GB)
 * Seasonic PRIME FOCUS Modular (80+Gold) 650 Watt
 * Noctua NH-D15 SE-AM4 140mm

System:

 * Ubuntu 17.10
 * kernel 4.14.0-rc7-custom (patched)


There is known NTP gpu performance issue using KVM on Ryzen platform. You need to apply [patch](https://patchwork.kernel.org/patch/10027525/) or wait for it to get into mainline Linux kernels.

## Enable IOMMU in grub bootloader

File location */etc/default/grub*
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash iommu=1 amd_iommu=on"
```
Then run <code>sudo update-grub</code>

## Add modules to initramfs
File location */etc/initramfs-tools/modules*
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

Then run <code>sudo update-initramfs -u</code>


## Create config files
In this setup we want to passthrough GPU device *(1060 nvidia series)* and whole NVME controller to the virtual machine.
+ Run <code> lspci -vnn</code> to get device IDs.

  Edit file */etc/modprobe.d/vfio.conf*
  ``` 
  options vfio-pci ids=10de:1c03,10de:10f1,144d:a804 
  ```

+ Make sure that GPU and NVME drivers will try to load after VFIO driver!

  Edit file */etc/modprobe.d/nvidia.conf*
  ``` 
  softdep nvidiafb pre: vfio vfio_pci 
  ```
  Edit file */etc/modprobe.d/nvme.conf*
  ``` 
  softdep nvme pre: vfio vfio_pci 
  ```

After rebooting the system run <code>lspci -vnn</code> and check if VFIO driver is in use.
 ```
 0f:00.0 VGA compatible controller [0300]: 
    NVIDIA Corporation GP106 [GeForce GTX 1060 6GB] [10de:1c03] (rev a1) (prog-if 00 [VGA controller])
	**Kernel driver in use: vfio-pci**
	Kernel modules: nvidiafb, nouveau, nvidia_384_drm, nvidia_384

0f:00.1 Audio device [0403]: 
    NVIDIA Corporation GP106 High Definition Audio Controller [10de:10f1] (rev a1)
	**Kernel driver in use: vfio-pci**
	Kernel modules: snd_hda_intel
    
01:00.0 Non-Volatile memory controller [0108]:
Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961 [144d:a804] (prog-if 02 [NVM Express])
	**Kernel driver in use: vfio-pci**
	Kernel modules: nvme
 ```
 
 ## Setup VM - CPU pinning (8 cores)
 Edit file */etc/libvirt/qemu/win10.xml*
 ``` xml
 <domain type='kvm'>
  <name>win10</name>
  <title>windows-pc</title>
  <description>Gaming machine</description>
  <vcpu placement='static'>8</vcpu>
  <cputune>
    <vcpupin vcpu='0' cpuset='0'/>
    <vcpupin vcpu='1' cpuset='1'/>
    <vcpupin vcpu='2' cpuset='2'/>
    <vcpupin vcpu='3' cpuset='3'/>
    <vcpupin vcpu='4' cpuset='4'/>
    <vcpupin vcpu='5' cpuset='5'/>
    <vcpupin vcpu='6' cpuset='6'/>
    <vcpupin vcpu='7' cpuset='7'/>
  </cputune>
  <features>
    <acpi/>
    <apic/>
    <hyperv>
      <relaxed state='on'/>
      <vapic state='on'/>
      <spinlocks state='on' retries='8191'/>
      <vendor_id state='on' value='whatever'/>
    </hyperv>
    <kvm>
      <hidden state='on'/>
    </kvm>
    <vmport state='off'/>
  </features>
  <cpu mode='host-passthrough' check='none'>
    <topology sockets='1' cores='4' threads='2'/>
  </cpu>
</domain>
 ```
 
### Usefull links:
 * [Reddit - Vfio](https://www.reddit.com/r/VFIO/)
 * [level1techs - npt patch](https://forum.level1techs.com/t/patch-npt-on-ryzen-for-better-performance-level-one-techs/120816)
 * [level1techs - fedora](https://forum.level1techs.com/t/ryzen-gpu-passthrough-setup-guide-fedora-26-windows-gaming-on-linux-level-one-techs/116959) 
 * [level1techs - ubuntu](https://forum.level1techs.com/t/ubuntu-17-04-vfio-pcie-passthrough-kernel-update-4-14-rc1/119639)
