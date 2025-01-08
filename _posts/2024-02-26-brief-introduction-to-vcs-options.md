---
layout: post
title: Brief Introduction to VCS Options
date: 2024-02-26 21:52 +0100
author: me
categories: [eda, vcs]
tags: [vcs]
---
# Introduction to Key VCS Options

VCS is a widely used HDL simulator from Synopsys. It supports compiling and simulating Verilog and SystemVerilog designs. VCS provides many compilation and simulation options to control and customize the simulation. This post introduces some of the commonly used VCS options.

## VCS Compilation Options

VCS compilation options control how the source code is compiled into an executable simulation executable. Some key compilation options include:

| Option | Description |
|--------|-------------|
| `-cm` | Enables code coverage metrics |
| `-debug` | Enables UCLI debug commands |
| `-l` | Logs compilation messages to file |
| `-ntb` | Enables Native Testbench |
| `-o` | Simulation executable name |
| `-v` | Verilog library file |
| `-y` | Verilog library directory |
| `-f` | File containing list of source files |
| `-file` | Similar to `-f`, but can contain options for controlling compilation, PLI options and object files. |
| `+incdir+<directory>` | Adds directory to include path |
| `+libext+<extension>` | Searches for library files with extension; use with `-y` |
| `+define+` | Defines macro for `ifdef` |
| `+protect` | Encrypts source code |
| `+cli` | Enables CLI debugging |


## VCS Simulation Options

VCS simulation options control the simulation execution. Some key options are:

| Option | Description |
|--------|-------------|
| `-sverilog` | Supports SystemVerilog |
| `-ucli` | Enables UCLI debug commands |
| `-vcd` | Dumps VCD file |
| `-i <filename>` | Runs CLI commands from file |
| `-k <filename>` | Logs UCLI and Virsim commands |

## VCS Debug Options

Some options useful for interactive debugging include:

| Option | Description |
|--------|-------------|
| `-RI` | Compiles and starts Virsim for debugging |
| `-RIG` | Starts Virsim with existing executable |
| `+sim+` | Specifies executable for `-RIG` |
| `-RPP` | Starts Virsim for post-processing with VCD+ |
| `+cfgfile+<filename>` | Specifies scenario configuration file |
| `+vslogfile+<filename>` | Saves Virsim command log |

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
