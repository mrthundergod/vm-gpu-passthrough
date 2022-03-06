# Single GPU Passthrough with Manjaro Host and Windows Guest

Based on Muda's guide video online(https://www.youtube.com/watch?v=BUSrdUoedTo) and Arch Wiki Guide(https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

## Step 1: Enabling IOMMU in he grubfile.

Open up the grub file in nano:
```
sudo nano /etc/default/grub
```
Add in `intel_iommu=on` or `amd_iommu=on` to the parameter `GRUB_CMDLINE_LINUX_DEFAULT`.

Save the file and compile the grub file.

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Step 2: Check IOMMU grouping and isolate the GPU

Run `dmesg` in commad line tro check if IOMMU is enabled/working properly.
```
dmesg | grep -i -e DMAR -e IOMMU
```
Check groupings using snippet below and find the entries from NVIDIA Corp.
```
#!/bin/bash
shopt -s nullglob
for g in `find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V`; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```
Example of grouping:
```
IOMMU Group 1:
        01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP106 [GeForce GTX 1060 3GB] [10de:1c02] (rev a1)
        01:00.1 Audio device [0403]: NVIDIA Corporation GP106 High Definition Audio Controller [10de:10f1] (rev a1)
```
## Step 3: Installig the QEMU, VMM and Virtual network apps

Time to install all the Virtual manager packages and bits.
```
sudo pacman -S qemu libvirt edk2-ovmf virt-manager ebtables dnsmasq
```
Enable `libvirtd.service` and start it.
```
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
```
Enable `virtlogd.socket` and start it.
```
sudo systemctl enable virtlogd.socket
sudo systemctl start virtlogd.socket
```
Enable `virsh` to autostart and start. This will be the default network for our VM.
```
sudo virsh net-autostart default
sudo virsh net-start default
```
