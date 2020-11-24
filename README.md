# Regolith installation on fully encrypted disk with LVM2
Installation was on Lenovo T14 Gen 1 Notebook.

## Download and create bootable USB stick

```
dd if=regolith-linux.iso of=/dev/sdx bs=1M
```

## Disk partitioning

You need to create 3 partitions

1. 1G for /boot
2. 128M (up to 256) for efi partition
3. The rest will be encrypted

# Create partitions
```
cgdisk /dev/sdX
1 100MB EFI partition # Hex code ef00
2 250MB Boot partition # Hex code 8300
3 100% size partiton # (to be encrypted) Hex code 8300
```
```
mkfs.vfat -F32 /dev/sdX1
mkfs.ext2 /dev/sdX2
```
# Setup the encryption of the system
```
cryptsetup -c aes-xts-plain64 -y --use-random luksFormat /dev/sdX3
cryptsetup luksOpen /dev/sdX3 luks
```
# Create encrypted partitions
# This creates one partions for root, modify if /home or other partitions should be on separate partitions
```
pvcreate /dev/mapper/luks
vgcreate vg0 /dev/mapper/luks
lvcreate -l +100%FREE vg0 --name root

# Create filesystems on encrypted partitions
mkfs.ext4 /dev/mapper/vg0-root

```

# Install system using manual disk partitioning
# After installing (without rebooting)

You need to know blkid of your encrypted partition
``` sudo blkid | grep LUKS
```
You'll need to setup `/mnt/etc/crypttab` by given example
```
luks  UUID=uuid   none luks,discard
```

```
mount /dev/mapper/vg0-root /mnt
mount --bind /dev/ /mnt/dev
mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devpts devpts /dev/pts

# Mount rest partitions
chroot /mnt /bin/bash
apt install cryptsetup lvm2 dmsetup
update-initramfs -k all -c

```

# After installation

```
apt install git build-essentials zsh tlp
chsh -s $(which zsh) # To change default shell to zsh
systemctl enable tlp # Battery saver
```

# Installation of Oh my zsh
```
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
sh install.sh
```

# Installation of Google chrome
```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
```

# Installation of solarized theme for terminal
```
git clone git://github.com/sigurdga/gnome-terminal-colors-solarized.git ~/.solarized
echo 'eval `dircolors ~/.dir_colors/dircolors`' >> ~/.zshrc

```

# Installation of Vundle package manager for ViM
```
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

```
# Terminal emulator with ligatures support. You can skip this step if you don't need ligatures
```
apt install kitty
sudo update-alternatives --config x-terminal-emulator   # To set kitty as default terminal
```

# Installing docker

```
sudo apt install curl

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

# Installing redshift and MS fonts
```
sudo apt install redshift gtk-redshift
sudo apt install ttf-mscorefonts-installer

```

# Lenovo throttle fix

```
sudo apt install git build-essential python3-dev libdbus-glib-1-dev libgirepository1.0-dev libcairo2-dev python3-venv python3-wheel
git clone https://github.com/erpalma/throttled.git
sudo ./throttled/install.sh
```

# Copy dotfiles from repo

```
xrdb -merge ~/.Xresources
regolith-look refresh
reboot
```

