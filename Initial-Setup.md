# Initial Setup

Begin by creating a new user, adding your SSH key, and modifying `sshd_config`. Remove root login.

```bash
adduser ptomerty
usermod -aG sudo ptomerty
mkdir .ssh && chmod 700 .ssh && cd .ssh
nano authorized_keys
chmod 600 authorized_keys


# Modify sshd_config to:
# PermitRootLogin no
# PasswordAuthentication no
sudo systemctl restart sshd 

sudo passwd -l root
```

Update all your packages and install some recommended ones.
```bash
sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y && sudo apt autoremove -y && sudo apt clean
sudo apt install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev libssl-dev asciidoc xmlto libev-dev libc-ares-dev automake libmbedtls-dev libsodium-dev curl cron screen git man-db wamerican ufw
```

Setup NVM:

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
nvm install 8.16.0
nvm use node 
npm install -g npm
node -v
sudo ln -s $(which node) /usr/bin/node
```

Setup an SSH key:
```bash
ssh-keygen -a 100 -t ed25519 -C "ptom@linode"
cat ~/.ssh/id_ed25519.pub
```

Setup UFW:
```bash
sudo ufw allow OpenSSH
sudo ufw allow proto tcp to 0.0.0.0/0 port 443 comment "Shadowsocks port"
sudo ufw enable
```

Add to `crontab`:
```
@weekly sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y && sudo apt autoremove -y && sudo apt clean && sudo reboot
```

### [Install Shadowsocks](./Shadowsocks-Obfs.md)
### [Install Kappbot](./Kappbot.md)