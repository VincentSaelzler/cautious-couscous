# cautious-couscous

Home Lab | Cadence | Apr 2026

## Configure SSH Key Authentication

```sh
⚠️⚠️⚠️ ROOT OF ALL TRUST ⚠️⚠️⚠️
ssh-keygen
```

```sh
# ensure horrea is a known, trusted, and accessible host
ssh-keygen -R horrea
ssh-copy-id horrea
ssh horrea
⚠️ APPEND-ONLY AUTHENTICATION ON BORGBASE ⚠️
ssh-keygen
cat ~/.ssh/id_ed25519.pub
```

## Borgbase Repositories

ℹ️ manually create a repository on borgbase

ℹ️ manually associate `horrea` ssh key with borgbase account and repo

ℹ️ manually update repo url in `./ansible/templates/borgmatic_config.yml.j2`

## Repository Encryption

⚠️ generate a passphrase and save it to lastpass

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

Lots of documentation is at <https://github.com/VincentSaelzler/onebox/>
