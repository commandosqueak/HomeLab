
## Create
lvcreate -L2G -s -n root-pre-nvidia /dev/pve/root

## Reverse
lvconvert --merge /dev/pve/root-pre-nvidia
reboot
