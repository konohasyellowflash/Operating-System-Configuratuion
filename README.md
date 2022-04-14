# Operating-System-Configuratuion

## Arch Linux SBG for ASUS-ROG-Zephyrus-G15-GA503QR_GA503QR
**0. System Specifications**

* CPU: AMD Ryzen 9 5900HS with Radeon Graphics (16) @ 4.680GHz
* Memory: 39584MiB
* Storage: 1.5TB
* Integrated GPU: AMD ATI 07:00.0 Cezanne 
* Dedicated GPU: NVIDIA GeForce RTX 3070 Mobile / Max-Q
* Resolution: 2560x1440 

**1. Download Latest Arch x86_64.iso**

https://archlinux.org/download/


**2. Create a bootable usb**
* In windows make use of https://rufus.ie/en/ 
* In linux use a dd command

```
dd if=archlinux-version-x86_64.iso of=/dev/sdb bs=4M
```
**Note**
Only issue which might appear is nouveau crashing the installation, this can be solved by adding a boot parameter **modprobe.blacklist=nouveau** to the kernel cmdline before booting the installation media

**3. Connect to Internet**

```
iwctl
```
```
device list
```
```
station wlan0 connect Zenos_5g
```

<details><summary>Troubleshoot</summary>

```
rfkill unblock all
```
</details>

**4. Ping some site on the Internet to verify connection**
```
ping -c 3 archlinux.org
```

**5. Update system clock**
```
timedatectl set-ntp true
```


<details><summary>Check status</summary>

```
timedatectl status
```
</details>

**6. Update your mirrorlist**
```
reflector --verbose --latest 200 --sort rate --save /etc/pacman.d/mirrorlist
```

**7. Install terminus-font and archlinux-keyring**
```
pacman -Syy terminus-font archlinux-keyring
```

<details><summary>setfont</summary>

```
setfont ter-v24b
```
</details>

**8. Create the encrypted filesystems**

**8A) Boot EFI Drive Setup**
```
gdisk /dev/nvme0n1
```
- Type 'o' to create a partition table
- Type 'n' for a new partition
- Enter
- Enter
- +1G
- EF00 This will create a 1 gb partition with the EFI format that we need for booting

**8B) Encrypted Arch Drive setup**
- Type 'n' to create a new partition
- Enter
- Enter
- Enter
- 8309

**8C) Check if everything looks right by pressing ‘p’. It should look like this:**
```
   Number    Start (sector)  End (sector)   Size          Code    Name
   1         2048            2099199        1024.0 MiB    EF00    EFI system partition
   2         2099200         2000409230     952.9 GiB     8309    Linux LUKS
```

**8D) Looks good?**
Press 'w' to write changes to disk.

**8E) Create the encrypted filesystems**
```
cryptsetup luksFormat /dev/nvme0n1p2
```
```
cryptsetup open /dev/nvme0n1p2 kurama
```
```
pvcreate /dev/mapper/kurama
```
```
vgcreate eighttrigramssealing /dev/mapper/kurama
```

**8F) Create encrypted partitions**
```
lvcreate -L 48G eighttrigramssealing -n swap
```
```
lvcreate -L 64G eighttrigramssealing -n root
```
```
lvcreate -l +100%FREE eighttrigramssealing -n home
```

**8G) Create filesystems**
```
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/eighttrigramssealing-root
mkfs.ext4 /dev/mapper/eighttrigramssealing-home
mkswap /dev/mapper/eighttrigramssealing-swap
```

**8H) Mount partitions**
```
mount /dev/mapper/eighttrigramssealing-root /mnt
```
```
mkdir /mnt/home
```
```
mkdir /mnt/boot
```
```
mount /dev/mapper/eighttrigramssealing-home /mnt/home
```
```
mount /dev/nvme0n1p1 /mnt/boot
```
```
swapon /dev/mapper/eighttrigramssealing-swap
```

**9) Install Arch linux base and vim packages**
```
pacstrap -i /mnt base vim
```

**10) Generate the /etc/fstab file**
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```

**11) Copy systemd network configuration files**
```
cp /etc/systemd/network/* /mnt/etc/systemd/network
```

**12) Change root to new system**
```
arch-chroot /mnt
```

**13) get the repo add to your /etc/pacman.conf at the end**
```
vim /etc/pacman.conf
```
```
[g14]
SigLevel = DatabaseNever Optional TrustAll
Server = https://arch.asus-linux.org
```

**14) Installing all the required packages**
**i) Basic pckages**
```
pacman -Sy linux-g14 linux-firmware linux-g14-headers base-devel efibootmgr mtools \
dosfstools openssh iwd zsh ntfs-3g amd-ucode xf86-video-amdgpu git ark unrar sshfs \
networkmanager asusctl nvidia-dkms supergfxctl gdm
```
**ii) Choosing Desktop Environment**
* Gnome
```
pacman -S gnome gnome-tweaks
```
* Plasma
```
pacman -Sy plasma xdg-desktop-portal kdeconnect  dolphin okular konsole kate gwenview \
spectacle libappindicator-gtk3 appmenu-gtk-module
```
```
pacman -Rnsc discover oxygen plasma-vault
```
* Cinnamon
```
pacman -S cinnamon nemo-fileroller
```
* XFCE
```
pacman -S xfce4 xfce4-goodies
```
* MATE
```
pacman -S mate mate-extra
```
**iii) Additional packages**
```
pacman -Sy firefox deepin-icon-theme deepin-gtk-theme virtualgl vlc \
pulseaudio-bluetooth signal-desktop qbittorrent kdenlive gimp obs-studio
```

**15) Set the timezone**
```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```

**16) Update the Hardware clock**
```
hwclock --systohc
```

**17) Adjust system clock from real time clock**
```
timedatectl set-local-rtc 1 --adjust-system-clock
```

**18) Set locale**
```
sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen (uncomment en_US.UTF-8)
```
```
locale-gen
```

**19) Create locale.conf**
```
vim /etc/locale.conf
```
Add the below line with your locale info
```
LANG=en_US.UTF-8
```

**20) Set your hostname**
```
vim /etc/hostname
```
Add something like below line
```
ai
```

**21) Set your hosts**
```
vim /etc/hosts
```
Add the below lines by making required changes
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   ai.localdomain   ai
```

**22) Configure mkinitcpio**
```
vim /etc/mkinitcpio.conf
```
Update "HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)" to
```
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt lvm2 filesystems resume fsck)
```

**23) Generate initramfs & install bootctl**
```
mkinitcpio -p linux-g14
```
```
bootctl install
```

**24) Create linux boot entry**
Check for the name and note it down, most likely it is /dev/nvme0n1p2
```
lsblk -lh
```
Then look for the UUID string in the output of following command at the device id 
```
blkid
```
you will get something like 33f6b22c-fc1e-47fe-8901-5ccbe7c64dda
```
vim /boot/loader/entries/arch.conf
```
Add the below lines by making required changes by replacing UUID with appropriate value
```
title arch
linux /vmlinuz-linux-g14
initrd /amd-ucode.img
initrd /initramfs-linux-g14.img
options cryptdevice=UUID=33f6b22c-fc1e-47fe-8901-5ccbe7c64dda:lvm:allow-discards resume=/dev/mapper/eighttrigramssealing-swap root=/dev/mapper/eighttrigramssealing-root rw pci=noaer nvidia-drm.modeset=0 quiet splash
```

**25) Enable boot menu item for windows**
```
lsblk
```
You will see something like 
```
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    nvme0n1     259:0    0 953.9G  0 disk 
    ...
    nvme1n1     259:4    0 465.8G  0 disk 
    -> nvme1n1p1 259:5    0   100M  0 part 
       nvme1n1p2 259:6    0    16M  0 part 
    ...
```
```
mount /dev/nvme1n1p1 /mnt
```
```
cp -R /mnt/EFI/Boot /boot/EFI/
```
```
cp -R /mnt/EFI/Microsoft /boot/EFI/
```
```
umount /mnt
```

**26) configure bootloader**
```
vim /boot/loader/loader.conf
```
Add below mentioned lines
```
timeout 10
default arch 
```

**26) Set root password and create a user**
```
passwd
```
```
useradd -m -g wheel krishna
```
```
passwd krishna
```
```
EDITOR=vim visudo
```
uncomment below line
```
%wheel ALL=(ALL) ALL
```

**27) Enable required services**
```
systemctl enable systemd-networkd
systemctl enable systemd-resolved
systemctl enable systemd-timesyncd
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable power-profiles-daemon
systemctl enable supergfxd
systemctl enable gdm
```
**28) Finish installation**
```
exit
umount -a
reboot
```
login as krishna

**29) Setup ZSH**
```
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

**30) Setup paru (AUR Helper)**
```
cd /opt
```
```
sudo git clone https://aur.archlinux.org/paru-git.git
```
```
sudo chown -R $USER ./paru-git
```
```
cd paru-git
```
```
makepkg -si
```

**31) Install required fonts**
```
paru -S ttf-comic-mono-git freetype2 ttf-ms-fonts
```
```
sudo pacman -S freetype2 fontconfig cairo ttf-ubuntu-font-family \
noto-fonts noto-fonts-cjk ttf-dejavu \
ttf-liberation ttf-opensans ttf-cascadia-code
```
```
sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
```
```
sudo ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
```
```
sudo ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
```
```
sudo vim /etc/profile.d/freetype2.sh
```
Add/uncomment the following line to/in it, save and close 
```
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
```
```
mkdir -p ~/.config/fontconfig/conf.d/
```
```
vim ~/.config/fontconfig/conf.d/20-no-embedded.conf
```
Add the following lines to it, save and close 
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<match target="font">
<edit name="embeddedbitmap" mode="assign">
<bool>false</bool>
</edit>
</match>
</fontconfig>
```
```
reboot

```

**32) Setup touch pad**
```
sudo pacman -S xdotool wmctrl
```
```
paru -S libinput-gestures
```
* libinput-gestures will detect touchpad gestures and 
  run a command when any set gesture is detected. 
* xdotool will react to this by simulating keyboard events. 
  To allow xdotool to work, the user you're using should be added to the input group:
```
sudo gpasswd -a $USER input
```
```
libinput-gestures-setup autostart
```
```
libinput-gestures-setup start
```

**33) Installing Razer software for keyboard, mouse configuration**
```
paru -S openrazer-driver-dkms openrazer-daemon openrazer-meta razercommander
```
```
sudo gpasswd -a $USER plugdev
```

**34) GPU Status tools**
* nvidia-smi
* glxspheres64

**35) Only for plasma**
- Open KRunner using meta key
```
kwriteconfig5 --file kwinrc --group ModifierOnlyShortcuts --key Meta "org.kde.krunner,/App,,toggleDisplay"
```
```
reboot
```
* Install kde-gtk-config kvantum-qt5
```
paru -S kde-gtk-config kvantum-qt5
```

**36) Installing zsh plugins**
```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
```
vim ~/.zshrc
```
update plugins
```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

**37) Installing powerlevel10k**
```
paru -S ttf-meslo-nerd-font-powerlevel10k
```
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
```
vim  ~/.zshrc
```
Edit the below mentioned line
```
ZSH_THEME="powerlevel10k/powerlevel10k" 
```
```
sudo vim /etc/pacman.conf      
```
Add below lines
```
Color
ILoveCandy
```

**38) Installing required IDEs**
```
paru -S visual-studio-code-bin android-studio intellij-idea-community-edition pycharm-community-edition
```

**39) Installing only office**
```
paru -S  onlyoffice-bin
```

**40) Installing virtualbox**
```
paru -S libvirt virtualbox virtualbox-ext-vnc virtualbox-guest-iso virtualbox-guest-utils \
virtualbox-host-dkms virtualbox-ext-oracle
```

**41) Installing teamviewer**
```
paru -S teamviewer
```
before using run the below command
```
sudo systemctl start teamviewerd 
```

**42) Installing google-chrome**
```
paru -S google-chrome
```

**43) Installing snapd**
```
paru -Sy snapd
```
```
systemctl restart apparmor.service
```
```
systemctl restart snapd.service
```
```
systemctl enable --now snapd.apparmor.service
```

**44) Installing Microsoft Office 365**
```
sudo snap install office365webdesktop --beta
```


**45) Supergfxctl - graphics switching**
* integrated, uses the iGPU only and force-disables the dGPU
```
sudo supergfxctl --mode integrated
```
* dedicated, uses the Nvidia gpu only
```
sudo supergfxctl --mode dedicated
```
* hybrid, enables Nvidia prime-offload mode
```
sudo supergfxctl --mode hybrid
```
* compute, enables Nvidia without Xorg. Useful for ML/Cuda
```
sudo supergfxctl --mode compute
```
vfio, binds the Nvidia gpu to vfio for VM pass-through
```
sudo supergfxctl --mode vfio
```
need to relogin after switching

* get current mode
```
sudo supergfxctl -g 
```
**46) Machine Learning Setup**
```
sudo pacman -Syu cuda cudnn jupyter-notebook python-tensorflow-cuda
```
To test gpu availability
```
import tensorflow as tf
tf.test.is_gpu_available()
```
