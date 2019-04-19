# Install Shadowsocks
Setup Shadowsocks:
```bash
export LIBSODIUM_VER=1.0.16
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig

export MBEDTLS_VER=2.6.0
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS="-O2 -fPIC"
sudo make DESTDIR=/usr install
popd
sudo ldconfig

rm -rf libsodium-$LIBSODIUM_VER.tar.gz libsodium-$LIBSODIUM_VER mbedtls-$MBEDTLS_VER-gpl.tgz mbedtls-$MBEDTLS_VER

cd /opt
sudo git clone --depth 1 https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
sudo git submodule update --init --recursive

sudo ./autogen.sh 
sudo ./configure --with-sodium-include=/usr/include --with-sodium-lib=/usr/lib --with-mbedtls-include=/usr/include --with-mbedtls-lib=/usr/lib
sudo make && sudo make install
```

Setup simple-obfs:
```bash
cd /opt
sudo git clone --depth 1 https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
sudo git submodule update --init --recursive

sudo ./autogen.sh
sudo ./configure
sudo make && sudo make install
```

Add a system user:
```bash
sudo adduser --system --no-create-home --group shadowsocks
sudo mkdir -m 755 /etc/shadowsocks
```

Create `/etc/shadowsocks/shadowsocks.json`:
```bash
{
    "server":"0.0.0.0",
    "server_port":443,
    "password":"FILL_IN_HERE",
    "timeout":300,
    "method":"chacha20-ietf-poly1305",
    "fast_open":true,
    "plugin":"obfs-server",
    "plugin_opts":"obfs=http"
}
```

Add optimizations to `/etc/sysctl.d/local.conf`:
```bash
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.rmem_default = 65536
net.core.wmem_default = 65536
net.core.netdev_max_backlog = 4096
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
net.ipv4.tcp_congestion_control = cubic
```

Update sysctl:
```bash
sudo sysctl --system
```

Drop config file into `/lib/systemd/system/shadowsocks.service`:
```
[Unit]
Description=Shadowsocks proxy server
After=network.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks/shadowsocks.json -a shadowsocks -v start
ExecStop=/usr/local/bin/ss-server -c /etc/shadowsocks/shadowsocks.json -a shadowsocks -v stop

[Install]
WantedBy=multi-user.target
```

Add the systemd service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable shadowsocks
sudo systemctl start shadowsocks
```
