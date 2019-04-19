# Shadowsocks with simple-obfs (Deprecated)

Setup Shadowsocks:
```bash
# Installation of libsodium
export LIBSODIUM_VER=1.0.16
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig

# Installation of MbedTLS
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

# Start building
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
mkdir -m 755 /etc/shadowsocks
```

Create `/etc/shadowsocks/shadowsocks.json`:
```bash
{
    "server":"0.0.0.0",
    "server_port":443,
    "password":"FILL_IN_HERE",
    "timeout":300,
    "method":"chacha20-ietf-poly1305",
    "fast_open": true,
    "plugin": "obfs-server",
    "plugin_opts": "obfs=http"
}
```


Drop config file into `/lib/systemd/system`:
```
[Unit]
Description=Shadowsocks proxy server

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


Add optimizations:
```bash
sudo nano /etc/sysctl.d/local.conf

<< ////

# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096
# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1
# for high-latency network
net.ipv4.tcp_congestion_control = hybla
# for low-latency network, use cubic instead
net.ipv4.tcp_congestion_control = cubic

////

sudo sysctl --system
```