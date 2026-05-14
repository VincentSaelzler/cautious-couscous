
on the laptop

```sh
passwd
```

on the main pc

```sh
ssh-keygen -R archiso
ssh root@archiso

hdparm -I /dev/sda
# Commands/features:
# * SANITIZE feature set
# * BLOCK_ERASE_EXT command
hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sda
hdparm --sanitize-status /dev/sda
# read first gib and last gib
hexdump -C -n 2G /dev/sda
tail -c 2G /dev/sda | hexdump -C
```

only do this part if wanting to wipe the drive which contains the backups

```
hdparm -I /dev/sdb
hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sdb
hdparm --sanitize-status /dev/sdb
# read first gigabyte and last gigabyte
hexdump -C -n 2G /dev/sdb
tail -c 2G /dev/sdb | hexdump -C

mkfs.btrfs --label backups /dev/sdb
mount LABEL=backups /mnt
btrfs filesystem show /mnt
umount /mnt
```

grab (or save) the configuration file for archinstall. this lives on the ventoy usb stick. it's nice to have, but not critical. it evolves a lot, so i will not save outdated versions in source control or documentation.

```
partprobe
mkdir /usb
mount /dev/mapper/sdc1 /usb

archinstall --config /usb/horrea/user_configuration.json --creds /usb/horrea/user_credentials.json
poweroff
```

```sh
ssh horrea

sudo blkid /dev/sdb
# LABEL="backups"

ansible-playbook -c local playbook.yml --ask-become-pass

-rw-r--r-- 1 root root 190 Mar 20 00:05 /usr/lib/systemd/system/btrfs-scrub@.service
-rw-r--r-- 1 root root 160 Mar 20 00:05 /usr/lib/systemd/system/btrfs-scrub@.timer

sudo systemctl edit btrfs-scrub@var-backups-borg.timer
[Timer]
OnCalendar=
OnCalendar=hourly
AccuracySec=1s
RandomizedDelaySec=0
cat /etc/systemd/system/btrfs-scrub@var-backups-borg.timer.d/override.conf
sudo systemctl enable --now btrfs-scrub@var-backups-borg.timer

systemctl status btrfs-scrub@var-backups-borg.timer
systemctl status btrfs-scrub@var-backups-borg.service
# Apr 19 17:00:00 horrea btrfs[3409]: Error summary:    no errors found
sudo btrfs scrub status /var/backups/borg
# Error summary:    no errors found
```