# Arch Linux Installation on Encrypted Root

This article tells about installing encrypted Arch Linux with desktop. [This](https://github.com/someoneinjd/dotfiles) is my goal.

## Encrypt Partition with LUKS

After downloading ISO from [tuna](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/) (super fast for me), burning ISO to a USB disk with [Win32DiskImager](https://sourceforge.net/projects/win32diskimager/), boot into it.
Following the official [installation guide](https://wiki.archlinux.org/title/installation_guide), connect to the network and update clock with `timedatectl set-ntp true`, then start partition.

### Layout

| Mount Point | Partition | Type                | Size           |
| ----------- | --------- | ------------------- | -------------- |
| /boot       | /dev/sda1 | EFI                 | 512M           |
| /           | /dev/sda2 | LUKS Encrypted ext4 | All space left |

### Prepare Encrypted Root Partition

After setting up the partitions with `fdisk`, follow the [instruction](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Preparing_the_disk) to prepare the encrypted root:

```
# cryptsetup -y -v luksFormat /dev/sda2
# cryptsetup open /dev/sda2 croot
# mkfs.ext4 /dev/mapper/croot
# mount /dev/mapper/croot /mnt
```

### Prepare the Boot Partition

```
# mkfs.fat -F32 /dev/sda1
# mkdir /mnt/boot
# mount /dev/sda1 /mnt/boot
```

## System Installation

### Edit Mirror

Add mirror Tuna to `/etc/pacman.d/mirrorlist` (fast in China):

```properties
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

### Install with Pacman

```
# pacstrap /mnt base linux linux-firmware neovim fish
```

### Fstab

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

```
# arch-chroot /mnt
```

### Set Fish as Default shell

```
# usermod --shell /bin/fish root
# fish
```

### Time zone & Localization

```
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# hwclock --systohc
# locale-gen
# echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
```

### Set Hostname

```
# echo myhostname > /etc/hostname
# echo '127.0.0.1 localhost
::1 localhost
127.0.0.1 myhostname' >> /etc/hosts
```

### Configuring Network (With Broadcom Driver)

```
# pacman -S dhcpcd iwd broadcom-wl-dkms
# systemctl enable dhcpcd iwd
```

### Configuring mkinitcpio

Add the encrypt hook to `mkinitcpio.conf`:

```properties
HOOKS=(...encrypt)
```

Recreate the initramfs image:

```
# mkinitcpio -P
```

### Change Password

```
# passwd
```

### Create Swap File

Create an 9000M swap file under root:

```
# dd if=/dev/zero of=/swapfile bs=1M count=9000 status=progress
# chmod 600 /swapfile
# mkswap /swapfile
# echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```

### Install Grub

```
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Add kernel parameters to `/etc/default/grub`:

```properties
GRUB_CMDLINE_LINUX="... root=/dev/mapper/croot resume=/swapfile cryptdevice=UUID=...:croot"
```

Generate `grub.cfg`:

```
# grub-mkconfig -o /boot/grub/grub.cfg
```

### Reboot

```
# exit
# exit
# reboot
```

## Configure the System

Follow the [General Recommendations](https://wiki.archlinux.org/title/General_recommendations) in arch wiki.

### Create User

```
# useradd -m -G wheel user
# passwd user
```

### sudo Configuration

```
# pacman -S sudo
```

Uncomment line:

```properties
%wheel ALL=(ALL) ALL
```
