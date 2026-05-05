# cautious-couscous
Home Lab | Cadence | Apr 2026

```sh
sudo pacman -S ansible
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

