---
layout: post
title: "Self-Hosting Gitea: Docker Setup with Web and SSH Public Access via Cloudflare Tunnel"
date: 2024-12-28 14:12 +0100
author: me
categories: [server, docker]
tags: [Docker, Gitea, Cloudflare]
---

[![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=Cloudflare&logoColor=white)](https://www.cloudflare.com/)
[![Gitea](https://img.shields.io/badge/Gitea-330099?style=for-the-badge&logo=Gitea&logoColor=white)](https://gitea.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)

This guide will walk you through the process of setting up Gitea using Docker on an Ubuntu server and making it accessible via Cloudflare Tunnel.

## Prerequisites
- A system running Ubuntu
- A user account with sudo privileges
- Terminal access

## Installation Steps

### 1. Install Docker on server

Please refer to the [Server Setup - Install Docker on Ubuntu](https://icshare.work/posts/server-setup-install-docker/) guide for detailed instructions on how to install Docker on your Ubuntu server.

### 2. Install/Configure Cloudflare Tunnel on server

Make sure you have a domain added to your Cloudflare account and install the Cloudflare Tunnel on your server, please refer to the [Cloudflare Tunnel](https://icshare.work/posts/cloudflare-tunnel/) guide for detailed instructions on how to install Cloudflare Tunnel on your Ubuntu server.

You can first create a tunnel named `gitea` and run it on your server.

### 3. Docker install Gitea
Here is a offical page of [Gitea Docker Installation](https://docs.gitea.com/installation/install-with-docker). But you can also follow the steps below to install Gitea.

#### 3.1 Create a user `git` on your server to do SSH Container Passthrough
A. Create a user `git` and set the user ID to an unused ID, for example `1001`.
```bash
sudo adduser git -u 1001
```
> **Note**: 
> 
> a. Just create a user `git` and you do NOT have to do the following steps with login as `git` user.
> 
> b. You can use `id git` to check the user ID and group ID of the user `git`.

B. Create an SSH key pair on the host machine. This key pair will be used to authenticate the git user when connecting from the host to the container. Run the following command as a user with sudo privileges:
```bash
sudo -u git ssh-keygen -t rsa -b 4096 -C "Gitea Host Key"
```

C. Add the public key of the key you created above ("Gitea Host Key") to `/home/git/.ssh/authorized_keys`.

```bash
sudo -u git cat /home/git/.ssh/id_rsa.pub | sudo -u git tee -a /home/git/.ssh/authorized_keys
sudo -u git chmod 600 /home/git/.ssh/authorized_keys
```

D. Create a fake host gitea command that will forward commands from the host to the container.

```bash
cat <<"EOF" | sudo tee /usr/local/bin/gitea
#!/bin/sh
ssh -p 2222 -o StrictHostKeyChecking=no git@127.0.0.1 "SSH_ORIGINAL_COMMAND=\"$SSH_ORIGINAL_COMMAND\" $0 $@"
EOF
sudo chmod +x /usr/local/bin/gitea
```

#### 3.2 Install Gitea with Docker

A. Create a directory for gitea installation, for example `/opt/gitea`.

B. Create a `docker-compose.yml` file in the directory `/opt/gitea`.

```yaml
services:                                                                                                                                                                                                                                      
  server:
    image: docker.io/gitea/gitea:latest
    environment:
      - USER_UID=1001 # Set the user ID to the same as the user you created in step 3.1
      - USER_GID=1001 # Set the group ID to the same as the user you created in step 3.1
      - GITEA__database__DB_TYPE=mysql
      - GITEA__database__HOST=db:3306
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea_password
    restart: always
    volumes:
      - /home/git/.ssh/:/data/git/.ssh
      - ./data:/var/lib/gitea
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000" # Web port mapping HOST:CONTAINER
      - "127.0.0.1:2222:22"   # SSH port mapping HOST:CONTAINER
    depends_on:
      - db

  db:
    image: mysql:8
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea_password
      - MYSQL_DATABASE=gitea
    volumes:
      - ./mysql:/var/lib/mysql
```

C. Run the `docker-compose.yml` file to install Gitea.

```bash
docker compose up -d
```

Now when you `curl localhost:3000`, you should see the Gitea web page.

### 4. Configure Cloudflare Tunnels for Gitea
Suppose your domain is `yourgitea.com`. On the Cloudflare dashboard, navigate to Zero Trust > Networks > Tunnels to find the tunnel `gitea` you created, then add following public names for web and ssh.

A. HTTP Tunnel for Gitea Web

- Subdomain: `gitea_web` (or any subdomain you want)
- Type: `HTTP` 
- URL: `localhost:3000` (this should match the web port mapping in your docker-compose.yml)

Now you can access your Gitea web interface at `https://gitea_web.yourgitea.com`. Please initialize the database and create an admin user following the web page instructions.
After initialization, now you should be able to create a new repository. 

If you create a repository named `first-repo`, you should be able to access it by the HTTP link Gitea provided, something like `https://gitea_web.yourgitea.com/${your_username}/first-repo.git`.

But now the ssh link like `git@gitea_web.yourgitea.com:${your_username}/first-repo.git` is not accessible externally.


B. SSH Tunnel for Gitea SSH

- Subdomain: `gitea_ssh` (or any subdomain you want)
- Type: `SSH` 
- URL: `localhost:2222` (this should match the ssh port mapping in your docker-compose.yml)

Now we have a domain `gitea_ssh.yourgitea.com` which binds to the server port `2222`.

### 5. Add public SSH key to the Gitea
A. Generate a public SSH key on your local machine. For example, `~/.ssh/id_rsa.pub`.

B. Enter the Gitea web interface, click your avatar to `Settings` -> `SSH/GPG Keys`, and add the public key you created above.

C. After you add the public key, you can see a key has been added into the Sever's `/home/git/.ssh/authorized_keys` file, something like:
```bash
# gitea public key
command="/usr/local/bin/git ...
```

### 6. Configure SSH conifg on your local machine

A. Create a `~/.ssh/config` file on your local machine if it does not exist.

B. Add the following content to the `~/.ssh/config` file.

```bash
Host gitea_web.yourgitea.com # match the subdomain configured for HTTP (Gitea domain)
    HostName gitea_ssh.yourgitea.com
    User git
    IdentityFile ~/.ssh/id_rsa
    ProxyCommand ${cloudflared_path} access ssh --hostname %h # cloudflared_path is the path to the cloudflared executable (e.g. /usr/local/bin/cloudflared)
```

Now you should be able to use `git clone git@gitea_web.yourgitea.com:${your_username}/first-repo.git` to clone the repository.

## Conclusion

You now have a fully functional, self-hosted Gitea instance with:
- Secure web access through Cloudflare Tunnel
- SSH support for Git operations
- MySQL database for data persistence
- Automatic container restarts

This setup provides a robust, private Git server that's securely accessible from anywhere while being protected by Cloudflare's security features.