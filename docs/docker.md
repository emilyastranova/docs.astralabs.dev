# Docker

Collection of useful Docker commands and utilities. Learn all about Docker, from the basics to advanced topics.

## Automated Install Script

=== "Debian"

    ```bash
    apt update && \
    apt install curl && \
    curl -sSL https://get.docker.com | sh
    ```

=== "Ubuntu"

    ```bash
    sudo apt update && \
    sudo apt install curl && \
    curl -sSL https://get.docker.com | sh
    ```

=== "Fedora/RHEL"

    ```bash
    sudo dnf update && \
    sudo dnf install curl && \
    curl -sSL https://get.docker.com | sh
    ```

=== "Arch"

    ```bash
    sudo pacman -Syu && \
    sudo pacman -S docker && \
    sudo systemctl start docker && \
    sudo systemctl enable docker
    sudo usermod -aG docker $USER
    ```

=== "Alpine"

    ```bash
    grep -qE '(http|https)://dl-cdn.alpinelinux.org/alpine/.*/community' /etc/apk/repositories || echo 'http://dl-cdn.alpinelinux.org/alpine/latest-stable/community' >> /etc/apk/repositories && \
    apk update && \
    apk add openrc && \
    apk add docker docker-compose && \
    rc-update add docker default && \
    service docker start
    ```

=== "Kali"

    ```bash
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    sudo chmod a+r /etc/apt/keyrings/docker.gpg && \
    sudo apt update && \
    sudo apt install -y docker-ce docker-ce-cli containerd.io
    ```

## Run Docker GUI applications

If you're looking to run graphical applications within a Docker container, you can follow these steps. This is particularly helpful for GUI applications, as it enables you to display their interfaces on your local machine while the application itself runs within the container.

To achieve this, you'll need to allow connections to your local X server, which handles graphical interfaces. The following commands show how to set up the necessary environment for running a Docker container with GUI applications:

```bash
xhost +local: && \
docker run -dit --pid=host -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:ro --name gui_application debian:12 && \
docker exec -it gui_application bash
```

## Portainer

Portainer is a web-based Docker management tool. It allows you to manage your Docker containers, images, volumes, networks and more! Portainer is meant to be as simple to deploy as it is to use. It consists of a single container that can run on any Docker engine (Docker for Linux and Docker for Windows are supported).

### Install/Uninstall

Install:

    docker volume create portainer_data && \
    docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

Uninstall:

    docker stop portainer && \
    docker rm portainer && \
    docker volume rm portainer_data

### Misc Notes

- Disable "Use SSO" if using Authentik

## SMB Volume Mount Failing at Reboot

If your Docker containers use SMB volume mounts and fail to start on boot since the share is not quite available, network isn't ready, etc. then this script is for you. This is meant for an Alpine container running OpenRC, feel free to adjust for `systemd`.

=== "Shell Script"

    Create the shell script
    
    ```shell
    mkdir /usr/share/scripts && nano /usr/share/scripts/share-watchdog.sh
    ```

    Shell script contents:
    
    ```bash
    #!/bin/sh
    while true; do
       if mountpoint -q "/var/lib/docker/volumes/plex-nas/_data"; then
            echo "Share mounted"
            #break # uncomment to stop the loop
       else
            echo "Share not mounted... Restarting container
            docker restart <container name>
       fi
       sleep 5
    done
    ```

=== "OpenRC Script"

    Create the OpenRC script file
    
    ```shell
    nano /etc/init.d/share-watchdog
    ```

    OpenRC file contents:
    
    ```shell
    #!/sbin/openrc-run
    
    description="Run share-watchdog script on boot"
    pidfile="/var/run/share-watchdog.pid"
    command="/usr/share/scripts/share-watchdog.sh"
    
    depend() {
        need localmount
        after bootmisc
    }
    
    start() {
        ebegin "Starting share-watchdog"
        start-stop-daemon --start --exec "$command" --background --make-pidfile --pidfile "$pidfile"
        eend $?
    }
    
    stop() {
        ebegin "Stopping share-watchdog"
        start-stop-daemon --stop --pidfile "$pidfile"
        eend $?
    }
    ```

    Add the service to `default` so it starts on boot, then start the service.
    ```shell
    rc-update add share-watchdog default && rc-service share-watchdog start
    ```

## Composerize

Convert Docker run commands to `docker-compose.yml` files

- Third-party host: [https://www.composerize.com/](https://www.composerize.com/)
- My backup: [https://github.com/emilyastranova/composerize](https://github.com/emilyastranova/composerize)

## Kasm Workspaces

Kasm Workspaces is a web-based Docker management tool. It allows you to manage your Docker containers, images, volumes, networks and more! Kasm Workspaces is meant to be as simple to deploy as it is to use. It consists of a single container that can run on any Docker engine (Docker for Linux and Docker for Windows are supported).

### Install

    cd /tmp && \
    curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.13.1.421524.tar.gz && \
    tar -xf kasm_release_1.13.1.421524.tar.gz && \
    sudo bash kasm_release/install.sh

### Uninstall

    sudo /opt/kasm/current/bin/stop && \
    sudo docker rm -f $(sudo docker container ls -qa --filter="label=kasm.kasmid") && \
    export KASM_UID=$(id kasm -u) && \
    export KASM_GID=$(id kasm -g) && \
    sudo -E docker compose -f /opt/kasm/current/docker/docker-compose.yaml rm && \
    sudo docker network rm kasm_default_network && \
    sudo docker volume rm kasm_db_1.13.1 && \
    sudo docker rmi redis:5-alpine && \
    sudo docker rmi postgres:9.5-alpine && \
    sudo docker rmi kasmweb/nginx:latest && \
    sudo docker rmi kasmweb/share:1.13.1 && \
    sudo docker rmi kasmweb/agent:1.13.1 && \
    sudo docker rmi kasmweb/manager:1.13.1 && \
    sudo docker rmi kasmweb/api:1.13.1 && \

    sudo docker rmi $(sudo docker images --filter "label=com.kasmweb.image=true" -q) && \
    sudo rm -rf /opt/kasm/ && \
    sudo userdel kasm_db -r && \
    sudo userdel kasm -r
