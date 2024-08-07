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