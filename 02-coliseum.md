
## Configure SSH Key Authentication

⚠️⚠️⚠️ This private key file is the root of all trust.

ℹ️ manually associate ssh key with borgbase account and repo

Do this whether creating backups or restoring them.

```sh
# generate ssh key ⚠️
ssh-keygen

# ensure horrea.lan is a known host
ssh-keygen -R horrea.lan
ssh-copy-id horrea.lan
```

## Create Backups

⚠️ generate a passphrase and save it to lastpass
ℹ️ manually create a repository on borgbase

```sh
# install borg on the target server
ssh horrea.lan
sudo pacman -S borg

# save passphrase via systemd
sudo su
systemd-ask-password -n | systemd-creds encrypt - /etc/credstore.encrypted/borgmatic.pw
```

Configure Borgmatic

```sh
sudo pacman -Syu borgmatic
mkdir ~/.config/borgmatic/
nano ~/.config/borgmatic/config.yaml # see below for contents
sudo nano /etc/systemd/system/borgmatic.service # see below for contents
sudo nano /etc/systemd/system/borgmatic.timer # see below for contents
sudo systemctl enable borgmatic.timer
```

Create Repositories and Backups

```sh
# create borg repos (in borgmatic config file config file) with borgmatic
borgmatic repo-create

# extract secret encryption key (distinct from passphrase)
# ⚠️ save to lastpass
borg key export ssh://marcus@horrea.lan/var/backups/borg/coliseum-charlie
borg key export ssh://d0im7qb1@d0im7qb1.repo.borgbase.com/./repo

# create backups
sudo systemctl start borgmatic.service

# check backups
mkdir ~/borgmnt
borg mount ssh://marcus@horrea.lan/home/marcus/bog-documents ~/borgmnt
umount ~/borgmnt
borg mount ssh://d0im7qb1@d0im7qb1.repo.borgbase.com/./repo ~/borgmnt
```

### Restoring Backups

```sh
# install borgmatic
sudo pacman -S borgmatic

# show timestamps of available backups
borg list ssh://marcus@horrea.lan//home/marcus/coliseum-bravo
borg list ssh://d0im7qb1@d0im7qb1.repo.borgbase.com/./repo

# create empty directrory to restore to
mkdir ~/Downloads/borgrestore
cd ~/Downloads/borgrestore

# restore backup
borg extract \
ssh://marcus@horrea.lan//home/marcus/coliseum-bravo::[latest backup name]
borg extract \
ssh://d0im7qb1@d0im7qb1.repo.borgbase.com/./repo::coliseum-2026-03-31T18:36:01.596852
```

## Configuration File Contents

```toml
sudo nano /etc/systemd/system/borgmatic.service

[Unit]
Description=borgmatic backup
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
RuntimeDirectory=borgmatic
StateDirectory=borgmatic
User=marcus
WorkingDirectory=/home/marcus

# Have systemd decrypt the encrypted blob and expose it at
# /run/credentials/<unit>/borgmatic.pw before switching to User=marcus.
LoadCredentialEncrypted=borgmatic.pw:/etc/credstore.encrypted/borgmatic.pw

# Optional debug to confirm the runtime credential file exists (safe: does not print secret)
ExecStartPre=/bin/sh -c 'ls -l "$CREDENTIALS_DIRECTORY" || true'

# Read the runtime credential file into the environment and exec borgmatic.
# This sets BORG_PASSPHRASE only for the borgmatic process and replaces the shell with borgmatic.
ExecStart=/bin/sh -c 'BORG_PASSPHRASE="$(cat "$CREDENTIALS_DIRECTORY/borgmatic.pw")" exec /usr/bin/borgmatic --verbosity 2'
```

```toml
sudo nano /etc/systemd/system/borgmatic.timer

[Unit]
Description=Run borgmatic backup

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```yml
nano ~/.config/borgmatic/config.yaml

source_directories_must_exist: true
source_directories:
   - ~/Camera
   - ~/Desktop Bethany
   - ~/Documents
   - ~/Family Photos
   - ~/Pictures
   - ~/Socials
   - ~/Videos
exclude_patterns:
   - '*/.stfolder'

repositories:
   - path: ssh://marcus@horrea/var/backups/borg/coliseum
     label: horrea
     encryption: repokey-blake2
   - path: ssh://d0im7qb1@d0im7qb1.repo.borgbase.com/./repo
     label: borgbase
     encryption: repokey-blake2

keep_daily: 7
keep_weekly: 5
keep_monthly: 12
keep_yearly: -1 # forever
```