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
sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y && sudo apt-get clean
sudo apt-get install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev asciidoc xmlto libev-dev libc-ares-dev automake libmbedtls-dev libsodium-dev curl cron screen 
```

Setup NVM:

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
```

Setup Shadowsocks: https://github.com/shadowsocks/shadowsocks-libev#linux

