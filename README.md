# cautious-couscous
Home Lab | Cadence | Apr 2026

```sh
# first, clone this repo
sudo pacman -S ansible
ansible-playbook 0_bootstrap.yml -i ./files/inventory.yml --ask-become-pass
```