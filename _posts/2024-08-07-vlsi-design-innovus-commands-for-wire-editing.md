---
layout: post
title: 'VLSI DESIGN - Innovus commands for wire editing'
date: 2024-08-07 14:44 +0200
categories: [hw, VLSI, Innovus]
tags: [Innovus, VLSI, wire editing]
---

Cadence Innovus is a powerful tool for physical design of integrated circuits. In this post, we will see some useful commands for wire editing in Innovus.

Below commands are verified on Innovus `22.13`.

## editTrim

The `editTrim` command is used to trim the wires in the design. It can be used to trim the wires in the selected area, selected nets, selected layers, or all the wires in the design.

### Syntax
``` bash
editTrim 
 [-help]
 [-keep_floating_stubs] 
 [-status {COVER FIXED NOSHIELD ROUTED SHIELD}] 
 [-type {float}] 
 [-all | -selected | -nets net* | -layers layer_names | -area {x1 y1 x2 y2}]
```

- Removes or trims wires. Wires are trimmed or removed as follows:
    - Trims floating stubs of wires back to the closest connection point.
    - Removes wires that do not have electrical connections to other wires or pins (floating wires).
    - Removes wires with only one connection to another wire or pin, along with connecting vias. Removing a wire with one connection might cause the wire to which the just-removed wire is connected to change from being a wire with two connections to a wire with one connection and thus it will also be removed. This process is repeated until no wires with one connection are left.
    - Note: The editTrim command recognizes patch wires and detects the patch wire connection with the via. Therefore, it does not delete patch wires even if they are connected from only one side.
    - Trims shield wires along with signal wires.


### Diagram
Diagram below shows the effect of `editTrim` command on a wire.

![editTrim](/assets/figs/editTrim_example_diagram.jpg)

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
