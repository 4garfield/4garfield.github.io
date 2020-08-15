---
title: "Install shadowsocks-libev on Centos"
date: 2020-01-01 01:13:50 +0800
header:
  image: "/assets/images/headers/using-shadowsocks-header.jpg"
  teaser: "/assets/images/headers/using-shadowsocks-teaser.jpg"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com/photos/MSWD-PDMizQ)"
tags:
  - shadowsocks
toc: true
---

Shadowsocks is a fast tunnel proxy. This article will show how to install shadowsocks-libev on CentOS 7.

## Server

Install the shadowsocks server via the pre-bundled script:

```sh
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

Select the server type `Shadowsocks-libev`, then input password and listening port, wait after installation successfully.

The default config file location: `/etc/shadowsocks-libev/config.json`, check the shadowsocks-libev status: `/etc/init.d/shadowsocks-libev status`.

## Client

[ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG) is the shadowsocks UI client on MacOS. Installed by: `brew cask install shadowsocksx-ng`.

To configure chrome using socks5 proxy, you can use the `SwitchyOmega` chrome plugin.

## BBR & Fastopen

TCP [BBR](https://github.com/google/bbr) is congestion control algorithm by Google. BBR is only avaliable after Linux kernel 4.9 or newer. so, to use BBR, you need update the kernel to latest version.

```sh
# update kernel
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y

# use the letest kernel at boot
egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \
# makesure the latest kernel is at the first index
grub2-set-default 0
reboot
```

After reboot server, to check if bbr is already enabled: `lsmod | grep bbr`.

If not enabled, you need manually enable bbr:

```sh
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

Reboot server again, to check if bbr is enabled:

```sh
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
lsmod | grep bbr
```

It's better to enable tcp fastopen, to do so, you need install the latest kernel-headers:

```sh
yum remove kernel-headers
yum --enablerepo=elrepo-kernel install kernel-ml-headers kernel-ml-devel kernel-ml-tools kernel-ml-tools-libs
echo "net.ipv4.tcp_fastopen=3" >> /etc/sysctl.conf
sysctl -p
```

## v2ray Plugin

[v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin) is based on v2ray for proxies.

### v2ray plugin at server

Installation steps:

```sh
cd /etc/shadowsocks-libev
wget https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.1.0/v2ray-plugin-linux-amd64-v1.1.0.tar.gz
tar -zvxf v2ray-plugin-linux-amd64-v1.1.0.tar.gz
mv v2ray-plugin_linux_amd64 v2ray-plugin
```

v2ray-plugin will look for TLS certificates signed by [acme.sh](https://github.com/Neilpang/acme.sh) by default. If you want to use your own signed certificate for your host, you can use below command(without key passphrase):

```sh
openssl req -x509 -nodes -sha256 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```

Then, edit the config file `/etc/shadowsocks-libev/config.json` to enable the v2ray-plugin with tls enabled:

```json
{
  "server": "0.0.0.0",
  "server_port": "{{port}}",
  "password": "{{password}}",
  "timeout": 300,
  "user": "nobody",
  "method": "aes-256-gcm",
  "fast_open": true,
  "nameserver": "8.8.8.8",
  "mode": "tcp_and_udp",
  "no_delay": true,
  "plugin": "/etc/shadowsocks-libev/v2ray-plugin",
  "plugin_opts": "server;tls;host={{hostname}};cert={{cert_path}};key={{key_path}}"
}
```

The "host" name must be included in the "cert" common name in the "plugin_opts".

then, restart the service: `/etc/init.d/shadowsocks-libev restart`.

### v2ray plugin at client

From v1.9.0, the ShadowsocksX-NG has provide the v2-ray-plugin by default. If you're runing older version, you can follow below steps to install:

```sh
cd $HOME/Library/Application\ Support/ShadowsocksX-NG/plugins/
wget https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.1.0/v2ray-plugin-darwin-amd64-v1.1.0.tar.gz
tar -zvxf v2ray-plugin-darwin-amd64-v1.1.0.tar.gz
mv v2ray-plugin_darwin_amd64 v2ray-plugin
```

Be sure to transfer your server public cert to your client. Then, open ShadowsocksX-NG, click the server preference, in the "Plugin" input field, type `v2ray-plugin`; in the "Plugin Opts" input field, type `tls;host={{hostname}};cert={{cert_path}}`.

## Proxy optimization

In the server, edit `/etc/sysctl.conf`, add below lines:

```conf
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
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1
```

Reload the configuration file: `sysctl -p`.
