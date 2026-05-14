# cautious-couscous
Home Lab | Cadence | Apr 2026

## Configure SSH Key Authentication

⚠️⚠️⚠️ This private key file is the root of all trust.

ℹ️ manually associate ssh key with borgbase account and repo

Do this whether creating backups or restoring them.

⚠️ generate a passphrase and save it to lastpass

ℹ️ manually create a repository on borgbase

```sh
# generate ssh key ⚠️
ssh-keygen
```

```sh
# ensure horrea is a known, trusted, and accessible host
ssh-keygen -R horrea
ssh-copy-id horrea
ssh horrea

# save passphrase via systemd
sudo su
systemd-ask-password -n | systemd-creds encrypt - /etc/credstore.encrypted/borgmatic.pw
```

## Install and Configure Ansible on the Controller Machine

```sh
sudo pacman -S ansible
sudo apt install ansible-core

# clone this repo
cd ~/cautious-couscous/ansible

# configure this host to be the ansible controller
ansible-playbook 0_bootstrap.yml -i ./files/inventory.yml --ask-become-pass
# run additional playbooks
ansible-playbook --ask-become-pass
```

```sh
sudo brightnessctl set 100%
sudo brightnessctl set 0%
```

Lots of documentation is at https://github.com/VincentSaelzler/onebox/
