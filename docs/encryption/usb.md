# USB Drive Encryption

## Install `cryptsetup`

### Installation on ubuntu and debian based distributions

```bash
sudo apt install cryptsetup
```

### Installation on arch based distributions

```
sudo pacman -S cryptsetup
```

## Partition USB Drive

### Open a root shell

```bash
sudo su
```

### Create 2 partitions using fdisk

> [!IMPORTANT]
> The first partition should be unencrypted and have
> a rather small volume size compared to the second encrypted volume.

```bash
# find the device, typically /dev/sdX where X is unknown. See size of the device to correlate
fdisk /dev/sdX
```

Within Interactive mode

```{bash .no-copy}
# use d to delete all partitions if there any already on the device
Command (m for help): n # new parition
[Press <ENTER> twice]
# 5GB size for unencrypted volume
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-31703005, default 31703005): +5GB

# create 2nd parition with remaining space on device
Command (m for help): n
[Press <ENTER> three times]

Command (m for help): w
```

### Format the partitions

```bash
mkfs.ext4 /dev/sdX1
mkfs.ext4 /dev/sdX2
```

### Protect against pattern based encryption attacks using `dd`

```bash
dd if=/dev/urandom of=/dev/sdX2 status=progress
```

## Encrypt the 2nd Partition

> [!IMPORTANT]
> You need provide a passphrase to decrypt the partition after running this command

```bash
cryptsetup luksFormat \
--type luks2 --cipher aes-xts-plain64 --hash sha256 --iter-time 2000 \
--key-size 256 --pbkdf argon2id --use-urandom --verify-passphrase /dev/sdX2

```

## Post Encryption Commands

### Opening the encrypted partition

```bash
cryptsetup open /dev/sdX2 device_mapper_name
```

The partition is now mapped to `/dev/mapper/<device_mapper_name>`

```bash
mkfs.ext4 /dev/mapper/<device_mapper_name>
```

The device partition can now be **mount**ed like another partition.

### Closing the encrypted partition

**unmount** the partition like any other partition using the device mapped name and then close the connection.

```bash
# assuming root shell access. use sudo otherwise
umount /dev/mapper/<device_mapper_name>
cryptsetup lcose device_mapper_name
```

## Resources

- [Linux Config Tutorial](https://linuxconfig.org/usb-stick-encryption-using-linux)
- [Arch Wiki Device Encryption Guide](ihttps://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Cryptsetup_passphrases_and_keys)
