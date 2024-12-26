---
layout: post
title: Server Setup - Install Docker on Ubuntu
date: 2024-12-26 17:09 +0100
author: me
categories: [server, ubuntu, docker]
tags: [docker]
---

[![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)

This guide will walk you through the process of installing Docker on Ubuntu. Docker is a platform that enables you to automate the deployment of applications inside lightweight containers.

## Prerequisites
- A system running Ubuntu
- A user account with sudo privileges
- Terminal access

## Installation Steps

### 1. Install Required Packages
First, update your package index and install packages needed to use the Docker repository:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
```

### 2. Add Docker's Official GPG Key
Add Docker's official GPG key to ensure package authenticity:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### 3. Set Up Docker Repository
Add the Docker repository to your system's package sources:

```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 4. Install Docker Engine
Install Docker Engine, containerd, and Docker Compose:

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 5. Configure User Permissions
Add your user to the Docker group to run Docker commands without sudo:

```bash
sudo usermod -aG docker $USER
```

> **Note**: You'll need to log out and back in for the group changes to take effect.

### 6. Verify Installation
Test your Docker installation by running the hello-world container:

```bash
docker run hello-world
```

If successful, you'll see a message like this:

```plaintext
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Congratulations! You've successfully installed Docker on your Ubuntu server.