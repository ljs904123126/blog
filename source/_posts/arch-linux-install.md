---
title: arch linux install
date: 2021-05-27 10:03:41
tags:
category:
---



##### down 

http://mirrors.163.com/archlinux/iso/2021.05.01/archlinux-2021.05.01-x86_64.iso



##### set keyboard 

list keyboard 

ls /usr/share/kbd/keymaps/**/*.map.gz

######  set 

loadkeys



##### verify uefi

ls /sys/firmware/efi/efivars



##### partition

###### BIOS Boot 1M  

no hand

###### EFI 512M

mkfs.fat -F32 /dev/efi_partition(sda2)

###### SWAP

 mkswap /dev/swap_partition(sda3)

###### /,/home

mkfs.ext4 /dev/root_partition(sda4)



###### mount

mount /dev/root_partition(sda4) /mnt

swapon /dev/swap_partition（交换空间分区）



##### network configuration

ip link

man 8 ip-link 



wifi pacman -S iw wireless_tools wpa_supplicant



##### install system

pacstrap /mnt base  linux linux-firmware  vim grub efibootmgr grub [ base-devel dhcpcd net-tools]



##### generate a fstab file 

genfstab -U /mnt >> /mnt/etc/fstab



arch-chroot /mnt



##### localtime configuration synchronization

 timedatectl set-ntp true



##### set time zone

 ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

or

timedatectl list-timezones 

timedatectl set-timezone Aisa/Shanghai

 hwclock --systohc



mount  efi 

mkdir /boot/efi

mount /dev/sda2 /boot/efi



mkinitcpio -P

grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi

grub-mkconfig -o /boot/grub/grub.cfg



grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch
grub-mkconfig -o /boot/grub/grub.cfg



https://wiki.gentoo.org/wiki/Initramfs/Guide/zh-cn



network configuration 

https://wiki.archlinux.org/title/Network_configuration

ip link set dev ens33 up 

ip a add 192.168.84.200/24 broadcast 192.168.84.255 dev ens33

ip route add default via  192.168.0.254  dev eth0   



network soft manager

NetworkManager

pacman -S networkmanager

systemctl enable NetworkManager

systemctl start NetworkManager

systemctl status NetworkManager



###### systemctl

list-dependencise

list-unit





###### user and group 

groupadd ljs

useradd -m -G ljs -s /bin/bash ljs

`-m create home -G group `



###### install xfce4



install xorg

sudo pacman -S  xfce4 mousepad parole ristretto thunar-archive-plugin thunar-media-tags-plugin xfce4-battery-plugin xfce4-datetime-plugin xfce4-mount-plugin xfce4-netload-plugin xfce4-notifyd xfce4-pulseaudio-plugin xfce4-screensaver xfce4-taskmanager xfce4-wavelan-plugin xfce4-weather-plugin xfce4-whiskermenu-plugin xfce4-xkb-plugin file-roller network-manager-applet leafpad epdfview galculator lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings capitaine-cursors arc-gtk-theme xdg-user-dirs-gtk



###### install vmware tools

sudo pacman -S open-vm-tools
sudo pacman -S gtkmm
sudo pacman -S xf86-video-vmware
sudo pacman -S xf86-input-vmmouse
systemctl enable vmtoolsd



###### install chinese font

pacman -S wqy-zenhei ttf-fireflysung 



## Device driver

### Check the status

[udev](https://wiki.archlinux.org/title/Udev) should detect your [network interface controller](https://en.wikipedia.org/wiki/Network_interface_controller) (NIC) and automatically load the necessary [kernel module](https://wiki.archlinux.org/title/Kernel_module) at startup. Check the "Ethernet controller" entry (or similar) from the `lspci -v` output. It should tell you which kernel module contains the driver for your network device. For example:

```
$ lspci -v
02:00.0 Ethernet controller: Attansic Technology Corp. L1 Gigabit Ethernet Adapter (rev b0)
 	...
 	Kernel driver in use: atl1
 	Kernel modules: atl1
```

Next, check that the driver was loaded by running `dmesg | grep *module_name*` as root. For example:

```
# dmesg | grep atl1
...
atl1 0000:02:00.0: eth0 link is up 100 Mbps full duplex
```