---
title: "The Beginner VPS setup guide"
date: 2019-07-13 00:01:42 +0800
tags:
  - vps
  - centos
toc: true
---

VPS is short for Virtual Private Server. Essentially, a VPS is a virtualized server that acts like an independent physical (dedicated) server.

## System update

Simply run `yum update` to update CentOS. You can also run `yum check-update` to check avaliables before update all.

After update completed, run `yum clean all` to clean yum caches, then reboot server.

## Create new privilege user

Add user through below command:

```bash
adduser username
passwd username
```

Your CentOS installation may or may not have the wheel group enabled. Open the configuration file by entering the command: `visudo`. Scroll through the configuration file to enable the following entry:

```config
%wheel        ALL=(ALL)       ALL
```

then, add the user to `wheel` group:

```bash
usermod -aG wheel username
```

## SSH

Generate your own SSH key pairs and upload public key to VPS:

```bash
ssh-keygen -t rsa -b 4096

cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

In client, edit `~/.ssh/config` to add the new key:

```config
Host IP
  User username
  IdentityFile ~/.ssh/rsa_key
```

SSH supports two protocols, 1 and 2. Version 1 is considered to be less secure and should no longer be used. Open the configuration file `/etc/ssh/sshd_config` and insert a new line `Protocol 2`. save the change and restart the sshd service: `systemctl restart sshd`

to disable the root login, edit `/etc/ssh/sshd_config`, change the line `PermitRootLogin yes` to `PermitRootLogin no`. then insert a new line to allow other user: `AllowUsers user`

## SELinux

Verify which SELinux packages are installed on your system: `sudo rpm -aq | grep selinux`
Install the following packages and their associated dependencies: `sudo yum install policycoreutils policycoreutils-python setools setools-console setroubleshoot`

check if SELinux is enabled or not: `sudo sestatus`.
if disabled, edit `/etc/selinux/config` to enforcing mode: `SELINUX=enforcing`. then reboot server.

Then, check the SELinux mode: `sudo getenforce`

Use the following command to view SELinux policy modules currently loaded into memory:`sudo semodule -l`

Use the `sealert` utility to generate a report from your audit log. `sudo sealert -a /var/log/audit/audit.log`

View the list of SELinux Boolean variables by using `getsebool -a` command, e.g. to view the current httpd boolean variables: `sudo getsebool -a | grep "httpd_can"`. enable the httpd for network connect `sudo setsebool -P httpd_can_network_connect ON`

## FirewallD

FirewallD is frontend controller for iptables used to implement persistent network traffic rules.

install firewalld: `sudo yum install firewalld`.

To start the service and enable FirewallD on boot:

```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
```

Check the firewall status: `sudo firewall-cmd --state`

To see the zones used by network interface(s): `sudo firewall-cmd --get-active-zones`.

Add the http/https traffic to the current session and the permanent rule-set by:

```bash
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --zone=public --add-service=https
sudo firewall-cmd --zone=public --permanent --add-service=https
```

verify the services: `sudo firewall-cmd --zone=public --permanent --list-services`

## Apache & SSL

Install Apache 2.4:`sudo yum install httpd`. Enable and start the Apache service:

```bash
sudo systemctl enable httpd.service
sudo systemctl start httpd.service
```

Change the permissions on the `/var/www` directory: `sudo chmod -R 755 /var/www`.

We can generate the let's encrypt SSL certs through `acme.sh`:

```bash
## get the script
curl https://get.acme.sh | sh
source ~/.bashrc

## generate the cert
acme.sh --issue -d mydomain.com --webroot /var/www/html

## install the cert
acme.sh --install-cert \
        --domain example.com \
        --cert-file /certpath/cert.cer \
        --key-file /certpath/key.key \
        --fullchain-file /certpath/fullchain.cer \
        --reloadcmd "sudo systemctl reload httpd.service"
```

Install apache httpd ssl module: `sudo yum install mod_ssl`, then, edit `/etc/httpd/conf.d/ssl.conf` to add cert file path.

```config
SSLCertificateFile /certpath/cert.cer
SSLCertificateKeyFile /certpath/key.key
SSLCertificateChainFile /certpath/fullchain.cer
```

Reload the httpd service: `systemctl reload httpd.service`.

Add below configuration for http to https redirection.

```config
<VirtualHost *:80>
Redirect permanent / https://yourdomain.com/
</VirtualHost>
```

## Cockpit

Cockpit makes it easy to administer your GNU/Linux servers via a web browser.

Installation: `sudo yum install cockpit`. Enable cockpit: `sudo systemctl enable --now cockpit.socket`

Open the firewall:

```bash
sudo firewall-cmd --permanent --zone=public --add-service=cockpit
sudo firewall-cmd --reload
```

Then, open browser for `https://SERVER_IP:9090`.

To disable root login, edit `/etc/pam.d/cockpit` to add the first line: `auth requisite pam_succeed_if.so uid >= 1000`.

Cockpit will load a certificate from the `/etc/cockpit/ws-certs.d` directory. It will use the last file with a `.cert` or `.crt` extension in alphabetical order. So, just manually install the cert to that directoty:

```bash
acme.sh --install-cert --domain example.com --key-file /etc/cockpit/ws-certs.d/cockpit.key --fullchain-file /etc/cockpit/ws-certs.d/cockpit.cert
```

then, restart the cockpit.
