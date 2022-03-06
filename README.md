# Single GPU passthrough with Manjaro Host and Windows Guest

Based on Muda's guide video online(https://www.youtube.com/watch?v=BUSrdUoedTo)

### Step 1
Enabling IOMMU in he grubfile.

sudo nano /etc/default/grub

add in "inteliommu=on" or "amd_iommu=on" to the 
