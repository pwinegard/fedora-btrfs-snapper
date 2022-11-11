# layout
### boot
1Gb /boot/efi

### root
btrfs /

### subvolumes
- /usr/local
- /tmp
- /home
- /opt
- /var
- /.snapshots

## Post install
### Remove copy on write for /var
```bash
sudo -i && \
chattr -R -f +C /var
```

### Enable btrfs snapshots in grub
1. run below
```bash
echo 'SUSE_BTRFS_SNAPSHOT_BOOTING="true"' >> /etc/default/grub && \
grub2-mkconfig -o /boot/grub2/grub.cfg  && \
sed -i '1i set btrfs_relative_path="yes"' /boot/efi/EFI/fedora/grub.cfg
```
1. reboot

## Install snapper and setup
1. run below
```bash
sudo dnf install snapper python3-dnf-plugin-snapper && \
sudo umount /.snapshots && \
sudo rmdir /.snapshots && \
sudo snapper -c root create-config / && \
sudo btrfs subvolume delete /.snapshots && \
sudo mkdir /.snapshots && \
sudo systemctl daemon-reload && \
sudo mount -a && \
sudo snapper -c root set-config ALLOW_USERS=$USER SYNC_ACL=yes && \
sudo chown -R :$USER /.snapshots && \
sudo grub2-editenv - unset menu_auto_hide && \
snapper ls
```

## Install grub-btrfs and setup

1. run below in a src drc
```bash
sudo -i && \
git clone https://github.com/Antynea/grub-btrfs.git && \
cd grub-btrfs && \
make install && \
echo 'GRUB_BTRFS_SHOW_TOTAL_SNAPSHOTS_FOUND="true"' >> /etc/default/grub-btrfs/config && \
echo 'GRUB_BTRFS_GRUB_DIRNAME="/boot/grub2"' >> /etc/default/grub-btrfs/config && \
echo 'GRUB_BTRFS_MKCONFIG=/usr/sbin/grub2-mkconfig' >> /etc/default/grub-btrfs/config && \
echo 'GRUB_BTRFS_SCRIPT_CHECK=grub2-script-check' >> /etc/default/grub-btrfs/config && \
grub2-mkconfig -o /boot/grub2/grub.cfg && \
systemctl enable --now grub-btrfsd
```