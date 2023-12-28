---
category: Fixes
---
# [Solution] Linux Mint Black Screen After Login Using Nvidia Drivers
Answer original provided by [generix](https://forums.developer.nvidia.com/t/linux-mint-nvidia-driver-loads-with-startx-but-not-on-initial-startup/168262)

Edit /etc/initramfs-tools/modules:
```bash
$ sudo nano /etc/initramfs-tools/modules
```
Add the following non-commented lines to the end of the file:
```
# Inside /etc/initramfs-tools/modules
----
----
nvidia
nvidia-modeset
nvidia-drm
```
`CTRL+X` then `Y` then `Enter` to exit.

Make changes take affect:
```bash
$ sudo update-initramfs -u
```
Restart machine
