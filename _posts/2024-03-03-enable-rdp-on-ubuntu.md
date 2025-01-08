---
layout: post
title: Enable RDP on Ubuntu
date: 2024-03-03 18:28 +0100
author: me
categories: [ubuntu, remote-desktop]
tags: [rdp]
---
# Enable RDP on Ubuntu
Xrdp (X Remote Desktop Protocol) is a free and open-source implementation of the Microsoft Remote Desktop Protocol (RDP) that allows users to remotely access graphical desktops running on Unix-based systems. This can be useful for system administration, running graphical applications on a remote server, or providing a graphical desktop environment for users who are not physically present at the computer.

In this blog post, we will walk you through the steps of installing and configuring Xrdp on Ubuntu 20.04.

## **Prerequisites**

- A computer running Ubuntu 20.04
- A user account with sudo privileges


## Step 1: Update the system

Before you begin, it is important to update your system to ensure that you have the latest packages and security fixes. You can do this by running the following command in your terminal:
``` bash
sudo apt update && sudo apt upgrade
```

## Step 2: Install Ubuntu desktop environment and Xrdp

Once your system is up to date, you can install Xrdp using the following command:
``` bash
sudo apt install ubuntu-desktop
sudo apt install xrdp
```
Once the installation is complete, xrdp service will start automatically. You can verify the status of the service by running the following command:
``` bash
sudo systemctl status xrdp
```

## Step 3: Configure Xrdp

After installing Xrdp, a new system user `xrdp` is created and an ssl key `ssl-cert-snakeoil.key` is placed in the `/etc/ssl/private/` folder. You need to add the xrdp user to the ssl-cert group to ensure that xrdp can read this ssl key:

```bash
sudo adduser xrdp ssl-cert
```

## Step 4: Restart Xrdp

Once you have made your changes to the configuration file, you need to restart the Xrdp service for the changes to take effect. You can do this by running the following command:
``` bash
sudo systemctl restart xrdp
```

## Step 5: Connect to your Xrdp session

Now you can connect to your Xrdp session from a remote Windows machine using a Remote Desktop Client. The hostname or IP address of your Ubuntu system and the port number (default: 3389) are required to connect.

##  **Fix the issue of the black screen**
Edit `/etc/xrdp/startwm.sh` script:
``` bash
sudo vim /etc/xrdp/startwm.sh
```

and add the following line before the line `test -x /etc/X11/Xsession && exec /etc/X11/Xsession`:
``` bash
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
```


----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
