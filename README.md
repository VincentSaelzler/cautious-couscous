# cautious-couscous

Home Lab | Cadence | Apr 2026

Lots of documentation is at <https://github.com/VincentSaelzler/onebox/>

## OS Provisioning (`tabula`)

Boot the laptop (`tabula`) into BIOS. Erase the NVMe drive using the "cryptographic keys" option. Restart

Boot the laptop (`tabula`) into the Arch Linux live environment (`archiso`) and set a temporary root password:

```sh
passwd
```

From the controller, connect to the live environment 

```sh
ssh-keygen -R archiso
ssh root@archiso
```

Install Arch Linux using the saved configuration on the Ventoy USB (it evolves often, so it is not in source control), then power off:

```sh
partprobe
mkdir /usb
lsblk
mount /dev/mapper/sda1 /usb  # Adjust path based on lsblk

archinstall --config /usb/tabula/user_configuration.json --creds /usb/tabula/user_credentials.json
poweroff
```

*see instructions below related to the controller PC. these i instructions should actually be above horrea provisioning.*

## OS Provisioning (`horrea`)

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

## SSH Authentication

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
# add to borgbase repo ssh keys (append-only)
# establish trust with Borgbase public key
ssh -o RequestTTY=no repo_id@repo_id.repo.borgbase.com
```

## Borgbase Repository Setup

- **Create Repository:** Manually create a new repository on Borgbase.
- **Add Key:** Associate `horrea`'s new SSH key with your Borgbase account and grant it append-only access to the repository.
- **Update Config:** Update the repository URL in `./ansible/templates/borgmatic_config.yml.j2`.
- **Encryption:** Generate a strong passphrase for the repository and save it in LastPass.

## Ansible Deployment

Install Ansible on the controller PC and deploy the configuration:

```sh
# Install Ansible
sudo pacman -S ansible # Arch
# sudo apt install ansible-core # Ubuntu/Debian

git clone https://github.com/vincentsaelzler/cautious-couscous.git
cd ~/cautious-couscous/ansible

# Configure the local controller host
ansible-playbook 0_bootstrap.yml -i ./files/inventory.yml --ask-become-pass
```

## Provision Horrea and Restore Files from Backup

It is safe to sequentially run all playbooks **before** `horrea_borgmatic_backup`.

Critically, the `horrea_syncthing` playbook must be run before `horrea_borgmatic_backup`, otherwise borgmatic could be backing up files in a directory which are missing files (or even subdirectories) on coliseum which were updated after horrea was shut down.

```sh
ansible-playbook --ask-become-pass <playbook_name.yml>
```

## Mutually Connect Hosts via Syncthing

Use the Web UI. On each side, click "Add Remote Device".

The following table assumes all hosts have already been configured to share their own folders locally (via Ansible).

| Setting | Horrea Web UI | Coliseum Web UI | Surface Web UI |
| --- | --- | --- | --- |
| Hosts to add | coliseum + surface | horrea | horrea |
| Introducer | no | **YES** | no | no |
| Auto accept | no | no | no | no |
| Folders to share | all | all | Downloads + Documents |

## Misc

Adjust laptop brightness manually:

```sh
sudo brightnessctl set 100%
sudo brightnessctl set 0%
```
