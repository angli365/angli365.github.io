---
layout: post
title: Cloudflare Tunnel
date: 2024-02-22 21:37 +0100

author: me
categories: [cloudflare, tunnel]
tags: [cf]
---
# Guide to Setting Up Cloudflare Tunnel

Cloudflare Tunnel offers a secure and efficient method to expose services running on a local network to the internet. This feature is a game-changer for developers and IT professionals looking to make their services accessible globally without compromising on security. Let's dive into how you can leverage Cloudflare Tunnel for your needs.

## Prerequisites for Using Cloudflare Tunnel

Before you embark on using Cloudflare Tunnel, there are a few requirements you should be aware of:

1. **Domain Registration**: Ensure you have a domain added to your Cloudflare account.
2. **Service Exposure**:
   - For HTTP/HTTPS services, installation of the Cloudflared application on the server is sufficient. Clients can access the service without needing Cloudflared once it's exposed.
   - TCP services require the Cloudflared application to be installed on both the server and the client.

## Setting Up Your Tunnel and Exposing Services

### Creating a Tunnel

1. Navigate to `Zero Trust` -> `Networks` -> `Tunnels` in your Cloudflare dashboard, and follow the provided instructions. It's recommended to choose a `Cloudflared` connector for optimal performance.
2. After creating your tunnel, click on its name and select `Configure`. Here, you'll find detailed instructions on **installing and running a connector** tailored to your operating system.
3. On the same page, you can **assign a public hostname to your tunnel**. For example, `vnc.hostname.com`, and specify the service you wish to expose by providing a type and a URL, such as `tcp` and `localhost:5901`. 
![Cloudflare Tunnel Configuration](https://cfzero-telegraph.pages.dev/file/42055fc8f9cd870e93b40.png)

### Connecting to the Service on the Client Side

- **HTTP/HTTPS Services**: Simply copy the domain you specified earlier into your browser to access the service.
- **TCP Services**: If you're exposing a service like VNC, follow these steps to connect to it from your client machine:
  ```bash
  cloudflared access tcp --hostname vnc.hostname.com --listener localhost:25565
  ```
    The listener address determines where on your client machine the service can be accessed. In this case, you would start a VNC client and connect using the server address `localhost:25565`.
