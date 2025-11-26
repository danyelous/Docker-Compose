# Installing Docker and Portainer on a Proxmox LXC Container

# I created it with the help of Claude AI and ChatGPT

Guide for setting up Docker and Portainer CE in a Proxmox LXC container.

## Prerequisites

- Proxmox VE installed and running
- An LXC container template (tested with Ubuntu 22.04)

## Step 1: Configure the LXC Container

Before starting, the container needs specific permissions to run Docker. On the **Proxmox host**, edit the container configuration:
```bash
nano /etc/pve/lxc/<CTID>.conf
```

Add or modify these lines:
```
unprivileged: 0
lxc.apparmor.profile: unconfined
lxc.cap.drop:
lxc.cgroup2.devices.allow: a
lxc.mount.auto: cgroup:rw
```

> **Note:** Replace `<CTID>` with your container ID. Setting `unprivileged: 0` makes this a **privileged container**, which is required for Docker to function properly.

Alternatively, you can change the privileged status via command line:
```bash
pct set <CTID> -unprivileged 0
```

## Step 2: Install Docker

Start the container and run:
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

Verify Docker is working:
```bash
docker run hello-world
```

If successful, you'll see a "Hello from Docker!" message.

## Step 3: Install Portainer

Create a volume for Portainer data and run the container:
```bash
docker volume create portainer_data

docker run -d \
  --name="portainer" \
  --restart=on-failure \
  -p 9000:9000 \
  -p 8000:8000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

## Step 4: Access Portainer

Open your browser and navigate to:
```
http://<CONTAINER_IP>:9000
```

Create your admin user on first access.

## References

- [Video tutorial](https://www.youtube.com/watch?v=e2jVkcC_KCc)
