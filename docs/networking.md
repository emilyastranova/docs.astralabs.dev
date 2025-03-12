# Networking

A deep dive into the world of networking, from basic concepts to advanced topics.

## Linux Network Interfaces setup

Modify the following file: `/etc/network/interfaces`

### DHCP

```bash
 auto eth0
 allow-hotplug eth0
 iface eth0 inet dhcp
```

### Static IP

```bash
auto eth0
iface eth0 inet static
    address 192.0.2.7/24
    gateway 192.0.2.254
```

### Restart Networking Service

Finally, as the `root` user or by running `sudo`, restart the `networking` service, usually accomplished with:

```shell
systemctl restart networking
```

or

```shell
service networking restart
```

## SSH Tunneling

SSH tunneling is a way to securely connect to a remote server through a local server. This is useful for accessing a remote server that is behind a firewall or NAT. It can also be used to access a remote server that is not directly accessible from the internet.

### SSH Port Forwarding

Forward port from local host to remote host:

```bash
ssh -R <remote_port>:<local_host>:<local_port> <remote_user>@<remote_host>
```

Forward port from remote host to local host:

```bash
ssh -L <local_port>:<remote_host>:<remote_port> <remote_user>@<remote_host>
```

### SSH Dynamic Port Forwarding

This works by creating a SOCKS proxy on the local host. This allows you to route all traffic through the remote host. This application-level port forwarding can only be run as root.

Forward all traffic from local host to remote host:

```bash
ssh -D <local_port> <remote_user>@<remote_host>
```

## Find Which Process is Using a Port

Sometimes you will find youself in a situation where you need to bind to a port (especially when working with Docker) and there is already another process taking it up. There's a few methods to find which process is listening on a port. For these examples we'll be looking for port 80.

### netstat

`netstat` is a command-line utility that displays network connections for both incoming and outgoing traffic. It can display TCP and UDP ports that the computer is listening on, as well as the state of TCP ports.

Package: `net-tools`

```bash
netstat -ltnp | grep -w ':80'
```

### lsof

`lsof` stands for "list open files". It is a command-line utility that displays information about files that are open by processes. It can display the process ID, user ID, file descriptor, and the path to the file.

Package: `lsof`

```bash
lsof -i :80
```

### fuser

`fuser` is a command-line utility that displays the process ID of processes that are using files or sockets. It can display the process ID, user ID, file descriptor, and the path to the file.

Package: `psmisc`

```bash
fuser 80/tcp
```

### ss

`ss` is a command-line utility that displays information about sockets. It can display TCP and UDP ports that the computer is listening on, as well as the state of TCP ports.

Package: `iproute2`

```bash
ss -ltnp | grep -w ':80'
```

## Forward real IP address in NGINX with Cloudflare

Simply put the following in your location block (or in Nginx Proxy Manager, put this in the Advanced section of your proxy host):

```bash
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 104.16.0.0/13;
set_real_ip_from 104.24.0.0/14;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 131.0.72.0/22;

set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2a06:98c0::/29;
set_real_ip_from 2c0f:f248::/32;

real_ip_header CF-Connecting-IP;
```

## Fix Nginx proxy_pass upstream SSL Issues

I was experiencing the following error in my Nginx setup after upgrading Nginx Proxy Manager one day:

```
SSL_do_handshake() failed (SSL: error:14094458:SSL routines:ssl3_read_bytes:tlsv1 unrecognized name:SSL alert number 112) while SSL handshaking to upstream
```

To fix this, I had to put the following in my location block (or in Nginx Proxy Manager, put this in the Advanced section of your proxy host):

```
proxy_ssl_server_name on;
proxy_ssl_name $host;
proxy_ssl_session_reuse off;
```
