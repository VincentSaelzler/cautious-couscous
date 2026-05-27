# cautious-couscous

Home Lab | Cadence | Apr 2026

Lots of documentation is at <https://github.com/VincentSaelzler/onebox/>

## 1. OS Provisioning (`horrea`)

Boot the laptop (`horrea`) into the Arch Linux live environment (`archiso`) and set a temporary root password:

```sh
passwd
```

From the controller, connect to the live environment to securely wipe the primary drive (`/dev/sda`):

```sh
ssh-keygen -R archiso
ssh root@archiso

# Securely erase the primary drive
hdparm -I /dev/sda
hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sda
hdparm --sanitize-status /dev/sda

# Verify wipe (check first and last 2GB)
hexdump -C -n 2G /dev/sda
tail -c 2G /dev/sda | hexdump -C
```

*(Optional)* Only do this if wiping the secondary drive (`/dev/sdb`) which contains the backups:

```sh
hdparm -I /dev/sdb
hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sdb
hdparm --sanitize-status /dev/sdb

# Verify wipe
hexdump -C -n 2G /dev/sdb
tail -c 2G /dev/sdb | hexdump -C

# Initialize the filesystem
mkfs.btrfs --label backups /dev/sdb
mount LABEL=backups /mnt
btrfs filesystem show /mnt
umount /mnt
```

Install Arch Linux using the saved configuration on the Ventoy USB (it evolves often, so it is not in source control), then power off:

```sh
partprobe
mkdir /usb
lsblk
mount /dev/mapper/sdc1 /usb  # Adjust path based on lsblk

archinstall --config /usb/horrea/user_configuration.json --creds /usb/horrea/user_credentials.json
poweroff
```

## 2. SSH Authentication

Establish trust with the newly installed host from the controller (the root of all trust):

```sh
# Ensure horrea is a known, trusted, and accessible host
ssh-keygen -R horrea
ssh-copy-id horrea
ssh horrea
```

Generate a dedicated SSH key on `horrea` for **append-only authentication to Borgbase**:

```sh
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub
ssh repo_id@repo_id.repo.borgbase.com # establish trust with Borgbase public key
```

## 3. Borgbase Repository Setup

1. **Create Repository:** Manually create a new repository on Borgbase.
2. **Add Key:** Associate `horrea`'s new SSH key with your Borgbase account and grant it append-only access to the repository.
3. **Update Config:** Update the repository URL in `./ansible/templates/borgmatic_config.yml.j2`.
4. **Encryption:** Generate a strong passphrase for the repository and save it in LastPass.

## 4. Ansible Deployment

Install Ansible on the controller PC and deploy the configuration:

```sh
# Install Ansible
sudo pacman -S ansible # Arch
# sudo apt install ansible-core # Ubuntu/Debian

cd ~/cautious-couscous/ansible

# Configure the local controller host
ansible-playbook 0_bootstrap.yml -i ./files/inventory.yml --ask-become-pass

# Run remaining deployment playbooks
ansible-playbook --ask-become-pass <playbook_name.yml>
```

### Misc

Adjust laptop brightness manually:

```sh
sudo brightnessctl set 100%
sudo brightnessctl set 0%
```

### Syncthing

on client: install syncthing

add remote (horrea)

# add folder(s) - note that this might not be required if auto-accepting shares, but don't do that right now so i don't accidentally wipe stuff out.

nope, instead tick both introducer and auto accept

check that things are syncing.