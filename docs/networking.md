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
