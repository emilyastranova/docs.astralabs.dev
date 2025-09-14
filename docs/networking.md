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

SSH tunneling is a way to securely transport network traffic from one machine to another through an encrypted connection. This is useful for accessing services that are behind a firewall or for securing unencrypted protocols.

### SSH Port Forwarding

This method forwards a single port from one network to another. The optional `<bind_address>` parameter controls which network interface the tunnel listens on. If you omit it, the tunnel is only accessible to the machine it's listening on (`localhost`). If you specify an IP or `0.0.0.0`, other computers on that network can use the tunnel.

#### Local Port Forwarding

This makes a **remote** service available on your **local** machine or network.

**Syntax:** `ssh -L [<local_bind_address>:]<local_port>:<destination_host>:<destination_port> <user>@<ssh_server>`

**Example**: You want to connect to a database on `remoteserver.com` from your local network.

- **To make it accessible only to your machine:**

    ```bash
    # On your PC, you can now connect to the database via localhost:5432
    ssh -L 5432:localhost:5432 user@remoteserver.com
    ```

- **To make it accessible to your entire local network:**

    ```bash
    # On your PC (e.g., 192.168.1.50), run this.
    # Other devices can now connect to the database via 192.168.1.50:5432
    # Also works with 0.0.0.0 to bind the port to all of the interfaces
    ssh -L 192.168.1.50:5432:localhost:5432 user@remoteserver.com
    ```

#### Remote Port Forwarding

This makes a **local** service available on the **remote** server or network.

**Syntax:** `ssh -R [<remote_bind_address>:]<remote_port>:<destination_host>:<destination_port> <user>@<ssh_server>`

**Example**: You want to expose a web server running on your laptop at `localhost:8080` to the remote server.

- **To make it accessible only from the remote server itself:**

    ```bash
    # On the remote server, you can now access your site via 'curl http://localhost:8000'
    ssh -R 8000:localhost:8080 user@remoteserver.com
    ```

- **To make it accessible to the remote server's entire network:**

    ```bash
    # Anyone who can reach remoteserver.com can now browse to http://remoteserver.com:8000
    # NOTE: This requires 'GatewayPorts yes' in the remote server's sshd_config file.
    ssh -R 0.0.0.0:8000:localhost:8080 user@remoteserver.com
    ```

#### Dynamic Port Forwarding

This works by creating a SOCKS proxy on your local host. This allows you to route traffic from applications on your local machine through the remote host. Running as root is only required if you use a privileged port (a port number below 1024).

Forward all traffic from local host to remote host:

```bash
ssh -D <local_port> <remote_user>@<remote_host>
```

#### Reverse Dynamic Port Forwarding

This is the inverse of dynamic port forwarding. It opens a SOCKS proxy on the **remote** host that tunnels traffic back to your **local** machine. This is useful when you want applications on the remote server (or other machines on its network) to route their traffic through your local network.

This feature is implemented entirely on the SSH client side, meaning it works even with older SSH servers that don't explicitly support it.

Create a SOCKS proxy on the remote host:

```bash
ssh -R <remote_socks_port> <remote_user>@<remote_host>
```

For example, `ssh -R 9050 user@remoteserver.com` would start a SOCKS proxy on port `9050` of `remoteserver.com`. Any application on the remote server configured to use `localhost:9050` as its proxy will have its traffic sent through the SSH tunnel to your local machine, and then out to its final destination.

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
