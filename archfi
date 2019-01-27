#!/bin/bash

# project   : https://github.com/asafemode/arch_install
# download  : wget https://raw.githubusercontent.com/asafemode/arch_install/master/archfi

#########################
#### Dialog function ####                                                       
#########################
boot_dialog() {
    DIALOG_RESULT=$(whiptail --clear --backtitle " Arch Linux / Parabola   Fast Install (GRUB SYSTEMD-BOOT)" "$@" 3>&1 1>&2 2>&3)
    DIALOG_CODE=$?
}

#################
#### Welcome ####                                                               
#################
clear
boot_dialog --title "Welcome" --msgbox "\nChoosing between GPT and MBR partition table." 10 60

##############################
#### UEFI / BIOS detection ####                                                 
##############################
efivar -l >/dev/null 2>&1

if [[ $? -eq 0 ]]; then
    UEFI_BIOS_text="UEFI detected."
    UEFI_radio="on"
    BIOS_radio="off"
else
    UEFI_BIOS_text="BIOS detected."
    UEFI_radio="off"
    BIOS_radio="on"
fi

boot_dialog --title "UEFI or BIOS" --radiolist "${UEFI_BIOS_text}\nPress <Enter> to accept." 10 60 2 1 GPT "$UEFI_radio" 2 MBR "$BIOS_radio"
[[ $DIALOG_RESULT -eq 1 ]] && GPT=1 || GPT=0
if [[ $GPT -eq 1 ]]; then
boot_dialog --title "UEFI boot loaders" --menu "" 10 60 2 "1" "GRUB" "2" "SYSTEMD-BOOT" 
bl="$DIALOG_RESULT"
fi

############################
#### Linux distribution ####                                                 
############################
boot_dialog --title "Linux distribution" --menu "" 10 60 2 "1" "ARCHLINUX" "2" "PARABOLA" 
os="$DIALOG_RESULT"

###################
#### Localtime ####
###################
items=$(ls -l /usr/share/zoneinfo/ | grep '^d' | gawk -F':[0-9]* ' '/:/{print $2}') 
  options=()
  for item in $items; do
    options+=("$item" "")
  done
boot_dialog --title "Timezone" --menu "" 16 60 7 "${options[@]}"  
region="$DIALOG_RESULT"

items=$(ls /usr/share/zoneinfo/$region/) 
  options=()
  for item in $items; do
    options+=("$item" "")
  done
boot_dialog --title "Timezone" --menu "" 16 60 7 "${options[@]}" 
city="$DIALOG_RESULT"

################
#### Locale ####
################
items=$(ls /usr/share/i18n/locales)
  options=()
  for item in $items; do
    options+=("$item" "")
  done
boot_dialog --title "Locale" --menu "" 16 60 7 "${options[@]}" 
locale="$DIALOG_RESULT"

################
#### Keymap ####
################
items=$(find /usr/share/kbd/keymaps/ -type f -printf "%f\n" | sort -V | awk -F'.map' '{print $1}')
  options=()
  for item in $items; do
    options+=("$item" "")
  done
boot_dialog --title "Keymap" --menu "" 16 60 7 "${options[@]}"  
keymap="$DIALOG_RESULT"  

##############
#### Font ####
##############
items=$(find /usr/share/kbd/consolefonts/*.psfu.gz -printf "%f\n" | cut -f1 -d.)
  options=()
  for item in $items; do
    options+=("$item" "")
  done
boot_dialog --title "Font" --menu "" 16 60 7 "${options[@]}" 
font="$DIALOG_RESULT"

############################
#### Display drivers ####                                                 
############################
boot_dialog --title "Display drivers" --menu "" 16 60 6 "1" "VIRTUALBOX" "2" "VMware" "3" "INTEL" "4" "ATI" "5" "AMD" "6" "NVIDIA"
dd="$DIALOG_RESULT"

#############################
#### Desktop environment ####                                                 
#############################
boot_dialog --title "Desktop environment" --menu "" 10 60 4 "1" "XFCE" "2" "MATE" "3" "KDE" "4" "WITHOUT DE"
de="$DIALOG_RESULT"

######################
#### File systems ####                                                 
######################
boot_dialog --title "File systems" --menu "" 10 60 2 "1" "EXT4" "2" "BTRFS" 
fs="$DIALOG_RESULT"

###########################
#### Hostname Username ####
###########################
boot_dialog --title "Hostname" --inputbox "\nPlease enter a name for this host.\n" 10 60
hostname="$DIALOG_RESULT"

boot_dialog --title "User name" --inputbox "Please enter a name for this user.\n" 10 60
username="$DIALOG_RESULT"

##########################
#### Password prompts ####                                                    
##########################
boot_dialog --title "Root password" --passwordbox "Please enter a strong password for the root user.\n" 10 60
root_password="$DIALOG_RESULT"

boot_dialog --title "User password" --passwordbox "Please enter a strong password for the user.\n" 10 60
user_password="$DIALOG_RESULT"

boot_dialog --title "Disk encryption" --passwordbox "\nEnter a strong passphrase for the disk encryption.\nLeave blank if you don't want encryption.\n" 10 60
encryption_passphrase="$DIALOG_RESULT"

#################
#### Warning ####
#################
boot_dialog --title "WARNING" --yesno "\nYou have chosen to remove all partition(ALL DATA) on the following drives:/dev/sda.\nPress <Yes> to continue or <No> to cancel.\n" 10 60
clear
if [[ $DIALOG_CODE -eq 1 ]]; then
    boot_dialog --title "Cancelled" --msgbox "\nScript was cancelled at your request." 10 60
    exit 0
fi

##########################
#### Reset the screen ####
##########################
reset
timedatectl set-ntp true
##################
#### Packages ####
##################     
    core_packages=''
    is_intel_cpu=$(lscpu | grep 'Intel' &> /dev/null && echo 'yes' || echo '')
    # Xserver
    core_packages+=' xorg-server xorg-apps xorg-xinit xorg-twm xterm'
if [[ "$os" = "1" ]]; then
    # linux-headers 
    core_packages+=' linux-headers'
else
    # linux-libre-headers 
    core_packages+=' linux-libre-headers'    
fi
if [[ "$os" = "1" && -n "$is_intel_cpu" ]]; then
   # https://wiki.archlinux.org/index.php/microcode
   core_packages+=' intel-ucode' 
fi
if [[ "$dd" = "1" && "$os" = "1" ]]; then    
    # Virtualbox
    core_packages+=' virtualbox-guest-utils'
fi
if [[ "$dd" = "2" ]]; then    
    # Vmware
    core_packages+=' xf86-video-vmware xf86-input-vmmouse'
fi
if [[ "$dd" = "3" ]]; then    
    # Intel
    core_packages+=' xf86-video-intel'
fi
if [[ "$dd" = "4" ]]; then    
    # Ati for older cards
    core_packages+='  xf86-video-ati'
fi
if [[ "$dd" = "5" ]]; then    
    # Amdgpu for newer cards
    core_packages+='  xf86-video-amdgpu'
fi
if [[ "$dd" = "6" ]]; then    
    # Nvidia
    core_packages+=' xf86-video-nouveau'
fi    
if [[ "$de" = "1" ]]; then 
    # Desktop environment
    core_packages+=' xfce4 xfce4-goodies'
    # Display manager
    core_packages+=' lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings'
fi    
    if [[ "$de" = "2" ]]; then 
    # Desktop environment
    core_packages+=' mate mate-extra'
    # Display manager
    core_packages+=' lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings'
fi 
    if [[ "$de" = "3" && "$os" = "1" ]]; then 
    # Desktop environment,localization for applications - kde-l10n-lt,kde-l10n-de ...
    core_packages+=' plasma-meta'
    # Display manager
    core_packages+=' sddm'
fi    
    if [[ "$de" = "3" && "$os" = "2" ]]; then 
    # Desktop environment,Parabola KDE pakages-https://www.parabola.nu/packages/?sort=&q=kde&maintainer=&flagged= ,localization for applications - kde-l10n-lt,kde-l10n-de ...
    core_packages+=' plasma kdebase-meta kdemultimedia-meta'
    # Display manager
    core_packages+=' sddm'
fi
    # Fonts
    core_packages+=' ttf-dejavu ttf-liberation ttf-freefont noto-fonts'
    # Audio
    core_packages+=' pulseaudio pulseaudio-alsa pavucontrol'
    # General utilities/libraries
    core_packages+=' mesa lib32-mesa xf86-input-libinput xdg-user-dirs networkmanager network-manager-applet ppp dialog wpa_supplicant'
    packages=''
if [[ "$os" = "1" ]]; then    
    # Browser
    packages+=' chromium'
    # General utilities/libraries
    packages+=' unace unrar'    
else
    # Browser ,language iceweasel-l10n-lt iceweasel-l10n-de iceweasel-l10n-ru 
    packages+=' iceweasel iceweasel-noscript iceweasel-ublock-origin'    
fi    
    # Image viewer
    packages+=' feh imagemagick'
    # Video
    packages+=' ffmpeg ffmpegthumbnailer mpv'
    # General utilities/libraries
    packages+=' neofetch git wget curl openssh file-roller p7zip unzip w3m mc htop ntfs-3g os-prober'

if [[ "$de" = "1" ]]; then    
    display_manager=" lightdm.service"
fi
if [[ "$de" = "2" ]]; then   
    display_manager=" lightdm.service"
fi
if [[ "$de" = "3" ]]; then   
    display_manager=" sddm.service"
fi

##################
#### Parabola ####
##################
#############################################################################################################################################
#### Install the keyring and mirror list for free repositories.If this doesn't work,see you https://wiki.parabola.nu/Migration_from_Arch ####
#############################################################################################################################################
migration_to_parabola(){
sed -i 's/^#RemoteFileSigLevel = Required/RemoteFileSigLevel = Never/' /etc/pacman.conf
#pacman -U --noconfirm --force https://www.parabola.nu/packages/libre/any/parabola-keyring/download
#pacman -U --noconfirm --force https://www.parabola.nu/packages/libre/any/pacman-mirrorlist/download
pacman -U --noconfirm --force https://repo.parabola.nu/pool/parabola/parabola-keyring-20190125-1-any.pkg.tar.xz
pacman -U --noconfirm --force https://repo.parabola.nu/pool/parabola/pacman-mirrorlist-20181227-1.par1-any.pkg.tar.xz
sed -i 's/^RemoteFileSigLevel = Never/#RemoteFileSigLevel = Required/' /etc/pacman.conf
cp -vr /etc/pacman.d/mirrorlist.pacnew /etc/pacman.d/mirrorlist
cat << PACMAN > /etc/pacman.conf
#
# /etc/pacman.conf
#
# See the pacman.conf(5) manpage for option and repository directives
#
# GENERAL OPTIONS
#
[options]
# The following paths are commented out with their default values listed.
# If you wish to use different paths, uncomment and update the paths.
#RootDir     = /
#DBPath      = /var/lib/pacman/
#CacheDir    = /var/cache/pacman/pkg/
#LogFile     = /var/log/pacman.log
#GPGDir      = /etc/pacman.d/gnupg/
#HookDir     = /etc/pacman.d/hooks/
HoldPkg     = pacman glibc
#XferCommand = /usr/bin/curl -C - -f %u > %o
#XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u
#CleanMethod = KeepInstalled
#UseDelta    = 0.7
Architecture = auto
#IgnorePkg   = 
#IgnorePkg   =
#IgnoreGroup =
#NoUpgrade   =
#NoExtract   =
# Misc options
#UseSyslog
#Color
#TotalDownload
CheckSpace
#VerbosePkgLists

# By default, pacman accepts packages signed by keys that its local keyring
# trusts (see pacman-key and its man page), as well as unsigned packages.
SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional
#RemoteFileSigLevel = Required
# NOTE: You must run `pacman-key --init` before first using pacman; the local
# keyring can then be populated with the keys of all official Arch Linux
# packagers with `pacman-key --populate archlinux`.
#
# REPOSITORIES
#   - can be defined here or included from another file
#   - pacman will search repositories in the order defined here
#   - local/custom mirrors can be added here or in separate files
#   - repositories listed first will take precedence when packages
#     have identical names, regardless of version number
#   - URLs will have $repo replaced by the name of the current repo
#   - URLs will have $arch replaced by the name of the architecture
#
# Repository entries are of the format:
#       [repo-name]
#       Server = ServerName
#       Include = IncludePath
#
# The header [repo-name] is crucial - it must be present and
# uncommented to enable the repo.
#
# The testing repositories are disabled by default. To enable, uncomment the
# repo name header and Include lines. You can add preferred servers immediately
# after the header, and they will be used before the default mirrors.
#[testing]
#Include = /etc/pacman.d/mirrorlist
#[pcr]
#Include = /etc/pacman.d/mirrorlist
[kernels]
Include = /etc/pacman.d/mirrorlist
[nonprism]
Include = /etc/pacman.d/mirrorlist
[libre]
Include = /etc/pacman.d/mirrorlist
[core]
Include = /etc/pacman.d/mirrorlist
[extra]
Include = /etc/pacman.d/mirrorlist
#[community-testing]
#Include = /etc/pacman.d/mirrorlist
[community]
Include = /etc/pacman.d/mirrorlist
# If you want to run 32 bit applications on your x86_64 system,
# enable the multilib repositories as required here.
#[multilib-testing]
#Include = /etc/pacman.d/mirrorlist
#[multilib]
#Include = /etc/pacman.d/mirrorlist
# An example of a custom package repository.  See the pacman manpage for
# tips on creating your own repositories.
#[custom]
#SigLevel = Optional TrustAll
#Server = file:///home/custompkgs
PACMAN
pacman -Scc --noconfirm
pacman -Syy --noconfirm
pacman-key --refresh
pacman -Suu pacman --noconfirm
pacman -Sy archlinux-keyring parabola-keyring --noconfirm
pacman-key --populate archlinux parabola
#pacman-key --refresh-keys
}  

############################################
#### Set up disk partitions and install ####
############################################
      swapsize=$(cat /proc/meminfo | grep MemTotal | awk '{ print $2 }')
      swapsize=$(($swapsize/1000))"M"
if [[ $GPT -eq 1 ]]; then
      parted /dev/sda mklabel gpt
      sgdisk /dev/sda -n=1:0:+1024M -t=1:ef00 
      sgdisk /dev/sda -n=2:0:+$swapsize -t=2:8200
      sgdisk /dev/sda -n=3:0:0
if [[ "$bl" = "1" ]]; then
      efi_bootmgr=" grub efibootmgr"
else
      efi_bootmgr=" dosfstools gptfdisk"
fi
else
      parted /dev/sda mklabel msdos
      sleep 2
      echo -e "n\np\n\n\n+512M\na\nw" | fdisk /dev/sda
      sleep 2
      echo -e "n\np\n\n\n+$swapsize\nt\n\n82\nw" | fdisk /dev/sda
      sleep 2
      echo -e "n\np\n\n\n\nw" | fdisk /dev/sda
      sleep 2
      mbr_grub=" grub"
fi
if [[ $GPT -eq 1 ]]; then      
      mkfs.vfat -F32 -n EFI /dev/sda1 
else      
      mkfs.ext2 -L boot /dev/sda1
fi      
      mkswap -L swap /dev/sda2
      swapon /dev/sda2
if [[ ! -z $encryption_passphrase ]]; then
      echo "Setting up encryption"
      printf "%s" "$encryption_passphrase" | cryptsetup luksFormat /dev/sda3
      printf "%s" "$encryption_passphrase" | cryptsetup open /dev/sda3 cryptroot
      physical_volume="/dev/mapper/cryptroot"
      encrypt_mkinitcpio_hook=" encrypt"
      sd_encrypt_mkinitcpio_hook=" sd-encrypt"
      crypt_uuid=$(blkid -s UUID -o value /dev/sda3)
      cryptdevice_grub="cryptdevice=UUID=${crypt_uuid}:cryptroot"
      cryptdevice_systemd="rd.luks.name=${crypt_uuid}=cryptroot"
else
      physical_volume="/dev/sda3"
      resume_mkinitcpio_hook=" resume"
      resume_grub_systemd=" resume=UUID=$(blkid -s UUID -o value /dev/sda2)"
fi
if [[ "$fs" = "1" ]]; then      
      mkfs.ext4 -L root $physical_volume
      mount $physical_volume /mnt
      mkdir /mnt/boot
      mount /dev/sda1 /mnt/boot
      fsck_mkinitcpio_hook=" fsck"
      root_systemd=" root=UUID=$(blkid -s UUID -o value ${physical_volume})"
else      
      mkfs.btrfs -L root $physical_volume
      mount $physical_volume /mnt
      btrfs subvolume create /mnt/@
      btrfs subvolume create /mnt/@home
      btrfs subvolume create /mnt/@snapshots
      btrfs sub set-default /mnt
      umount /mnt
      mount -o compress=lzo,subvol=@ $physical_volume /mnt
      mkdir /mnt/boot
      mount /dev/sda1 /mnt/boot
      mkdir /mnt/home
      mount -o compress=lzo,subvol=@home $physical_volume /mnt/home
      mkdir /mnt/snapshots
      mount -o compress=lzo,subvol=@snapshots $physical_volume /mnt/snapshots
      btrfs_progs=" btrfs-progs"
      root_systemd=" root=UUID=$(blkid -s UUID -o value ${physical_volume}) rootflags=subvol=@"
fi      
if [[ "$bl" = "2"  && ! -z $encryption_passphrase ]]; then     
      root_uuid=${cryptdevice_systemd}${root_systemd}
else
      root_uuid=${root_systemd}
fi
if [[ "$os" = "2" ]]; then
      migration_to_parabola
fi
      echo "pacstrap /mnt base base-devel bash-completion${btrfs_progs}${efi_bootmgr}${mbr_grub}"
      pacstrap /mnt base base-devel bash-completion$btrfs_progs$efi_bootmgr$mbr_grub  
      genfstab -U -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt /bin/bash <<EOF  
echo $hostname > /etc/hostname
ln -svf /usr/share/zoneinfo/$region/$city /etc/localtime
hwclock --systohc
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen
echo $locale".UTF-8 UTF-8" >> /etc/locale.gen
echo 'LANG="'$locale'.UTF-8"' > /etc/locale.conf
echo 'KEYMAP='$keymap'' >> /etc/vconsole.conf
echo 'FONT='$font'' >> /etc/vconsole.conf
locale-gen
echo "root:$root_password" | chpasswd
useradd -m -g users -G wheel -s /bin/bash $username
echo "$username:$user_password" | chpasswd
sed -i 's/^# %wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/g' /etc/sudoers
sed -i 's/^HOOKS=.*/HOOKS=(base udev autodetect keyboard keymap consolefont modconf block${encrypt_mkinitcpio_hook}${resume_mkinitcpio_hook} filesystems${fsck_mkinitcpio_hook})/g' /etc/mkinitcpio.conf
EOF
if [[ "$fs" = "2" ]]; then
arch-chroot /mnt /bin/bash <<EOF 
sed -i 's/^BINARIES=.*/BINARIES=(\/usr\/bin\/btrfs)/g' /etc/mkinitcpio.conf
EOF
fi
if [[ "$bl" = "2" ]]; then
arch-chroot /mnt /bin/bash <<EOF
sed -i 's/^HOOKS=.*/HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block${sd_encrypt_mkinitcpio_hook} filesystems${fsck_mkinitcpio_hook})/g' /etc/mkinitcpio.conf
EOF
fi
if [[ "$os" = "1" ]]; then
arch-chroot /mnt /bin/bash <<EOF
mkinitcpio -p linux
EOF
else
arch-chroot /mnt /bin/bash <<EOF
mkinitcpio -p linux-libre
EOF
fi
if [[ "$bl" = "2" ]]; then
arch-chroot /mnt /bin/bash <<EOF
bootctl install 
cat << LINUX > /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options ${root_uuid}${resume_grub_systemd} rw quiet splash
LINUX
cat << LINUXFL > /boot/loader/entries/arch-fallback.conf
title Arch Linux Fallback
linux /vmlinuz-linux
initrd /initramfs-linux-fallback.img
options ${root_uuid}${resume_grub_systemd} rw quiet splash
LINUXFL
cat << GRUB > /boot/loader/loader.conf
default arch
timeout 3
editor 0
GRUB
cat /boot/loader/entries/arch.conf
cat /boot/loader/entries/arch-fallback.conf
cat /boot/loader/loader.conf
EOF
else
arch-chroot /mnt /bin/bash <<EOF
sed -i "s/^GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT=\"quiet splash${resume_grub_systemd}\"/g" /etc/default/grub
sed -i "s/^GRUB_CMDLINE_LINUX=.*/GRUB_CMDLINE_LINUX=\"${cryptdevice_grub}\"/g" /etc/default/grub
EOF
fi
if [[ "$os" = "1" &&"$bl" = "2" && -n "$is_intel_cpu" ]]; then
arch-chroot /mnt /bin/bash <<EOF
sed -i "s/^initrd.*/initrd \/intel-ucode.img\\ninitrd \/initramfs-linux.img/g" /boot/loader/entries/arch.conf
sed -i "s/^initrd.*/initrd \/intel-ucode.img\\ninitrd \/initrd \/initramfs-linux-fallback.img/g" /boot/loader/entries/arch-fallback.conf
EOF
fi
if [[ "$bl" = "2" && "$os" = "2" ]]; then
arch-chroot /mnt /bin/bash <<EOF
sed -i 's/Arch Linux/Parabola/g' /boot/loader/entries/arch.conf
sed -i 's/vmlinuz-linux/vmlinuz-linux-libre/g' /boot/loader/entries/arch.conf
sed -i 's/initramfs-linux.img/initramfs-linux-libre.img/g' /boot/loader/entries/arch.conf
sed -i 's/Arch Linux/Parabola/g' /boot/loader/entries/arch-fallback.conf
sed -i 's/vmlinuz-linux/vmlinuz-linux-libre/g' /boot/loader/entries/arch-fallback.conf
sed -i 's/initramfs-linux-fallback.img/initramfs-linux-libre-fallback.img/g' /boot/loader/entries/arch-fallback.conf
EOF
fi
if [[ "$os" = "2" ]]; then
arch-chroot /mnt /bin/bash <<EOF
sed -i '/^#\[nonprism\]/{n;s/^#Include.*/Include = \/etc\/pacman.d\/mirrorlist/g}' /etc/pacman.conf
sed -i 's/^#\[nonprism\]/\[nonprism\]/g' /etc/pacman.conf
EOF
fi
arch-chroot /mnt /bin/bash <<EOF
sed -i "s/^#VerbosePkgLists/ILoveCandy\\n#VerbosePkgLists/g" /etc/pacman.conf
sed -i '/^#\[multilib\]/{n;s/^#Include.*/Include = \/etc\/pacman.d\/mirrorlist/g}' /etc/pacman.conf
sed -i 's/^#\[multilib\]/\[multilib\]/g' /etc/pacman.conf
pacman -Syy --noconfirm --needed $core_packages
EOF
if [[ "$bl" = "1" ]]; then
arch-chroot /mnt /bin/bash <<EOF
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
EOF
fi
if [[ $GPT -eq 0 ]]; then
arch-chroot /mnt /bin/bash <<EOF
grub-install --target=i386-pc --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
EOF
fi
if [[ $GPT -eq 1 &&  "$bl" = "1" && "$dd" = "1" ]]; then
arch-chroot /mnt /bin/bash <<EOF
mkdir /boot/EFI/boot
cp /boot/EFI/arch/grubx64.efi /boot/EFI/boot/bootx64.efi
EOF
fi
arch-chroot /mnt /bin/bash <<EOF
xdg-user-dirs-update
systemctl enable$display_manager NetworkManager
pacman -S --noconfirm --needed $packages
EOF

#################
#### The end ####                                                               
#################
printf "The script has completed boot Arch Linux or Parabola.\nIf it looks good you >> umount -R /mnt and shutdown now  or  reboot.\n"
