---
layout: post
title: 'GEM5: Docker Setup and Run Hello World'
date: 2024-07-04 18:38 +0200
author: me
categories: [hw, Simulation, Gem5]
tags: [Gem5, Docker, Simulation]
---

[Gem5](https://www.gem5.org/) is a popular open-source computer architecture simulator that allows you to model and analyze the performance of hardware systems. To get started with Gem5, you'll need to set up a working environment. This guide will walk you through setting up Gem5 in a Docker container.

## Make sure you have Docker installed

## Setting Up Docker
1. Create a directory to store your Gem5 files. For example, you can create a directory named 'gem5' in your home directory:
```bash
mkdir /home/<your_username>/gem5
```

2. We can pull a pre-built Docker image that contains all the dependencies needed to run Gem5. 
```bash
docker run -tid --hostname=gem5_dev --name=ubuntu24_gem5 -v /home/<your_username>/gem5:/root ghcr.io/gem5/ubuntu-24.04_all-dependencies:v24-0 /bin/bash    
```
Remember to replace `<your_username`> with your actual username.

## Entering Your Container
To access your new container and start working:
```bash
docker exec -ti ubuntu24_gem5 /bin/bash
cd /root
```

## Installing Gem5 and Building It
Now we'll get the Gem5 code and build it:
```bash
git clone https://github.com/gem5/gem5.git
cd gem5
scons build/X86/gem5.opt -j 8
```
Please note that this build process might take some time.

## Testing Gem5
To make sure everything is set up correctly, run this test:
```bash
build/X86/gem5.opt configs/learning_gem5/part1/simple.py
```
if you see the output, then you have successfully set up Gem5 in your Docker container.
> ...
>
> ...
>
> Beginning simulation!
>
> src/sim/simulate.cc:199: info: Entering event queue @ 0.  Starting simulation...
>
> Hello world!
>
> Exiting @ tick 493443000 because exiting with last active thread context


## Conclusion
You now have a working Gem5 environment set up in a Docker container. You can use this setup to explore the capabilities of Gem5 and run various simulations to analyze the performance of different hardware configurations. Happy simulating!

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
