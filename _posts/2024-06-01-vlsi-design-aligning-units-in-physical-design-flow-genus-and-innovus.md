---
layout: post
title: VLSI DESIGN - Aligning Units in Physical Design Flow (Genus and Innovus)
date: 2024-06-01 09:31 +0200
author: me
categories: [hw, VLSI, Genus, Innovus]
tags: [Genus, Innovus, VLSI]
---

In the realm of physical synthesis, managing unit consistency across different tools and libraries is critical for achieving accurate and optimal results. This blog will show the challenges and solutions for aligning units in Genus and Innovus, particularly when dealing with mixed units in timing libraries.

## Understanding the Problem
1. Unit discrepancies between tools: when working Genus and Innovus, one often encounters unit mismatches because these tools might use different default units for timing and capacitance. For instance, Genus defaults to picoseconds (ps) for timing and femtofarads (fF) for capacitance, whereas Innovus might revert to nanoseconds (ns) and picofarads (pF) if it detects inconsistencies in the timing libraries.

2. Unit discrepancies in timing libraries: timing libraries may contain mixed units for timing and capacitance values. For example, standard cells in the library might have timing values in ps and capacitance values in fF, while the memory cells could use ns and pF.

Such discrepancies can cause significant issues during the physical design flow, leading to incorrect timing analysis, inaccurate power estimations, and suboptimal results in terms of area and performance.

## Solutions to Ensure Consistency
To address these challenges, it is essential to align the units across both tools. Here’s how you can manage unit consistency effectively:

### Using set_units Command in SDC Files
Modify the `SDC` files to explicitly set the desired units:
```tcl
set_units -time ns -capacitance pF
```


### Setting Units in Genus and Innovus
In Genus/Innovus, you can override the default unit settings using the following commands before initializing the design:

**Common UI**:
```tcl
set_db timing_time_unit 1ps
set_db timing_cap_unit 1ff
```
**Legacy UI**:
```tcl
setLibraryUnit -time 1ps
setLibraryUnit -cap 1ff
```

### Customizing Units for Reporting in Genus 
Besides setting the default units, Genus allows customization of the units used for reporting:

**Common UI**:
```tcl
set_db timing_report_time_unit ns
set_db timing_report_load_unit pF
```
**Legacy UI**:
```tcl
set_attribute timing_report_time_unit ns
set_attribute timing_report_load_unit pF
```