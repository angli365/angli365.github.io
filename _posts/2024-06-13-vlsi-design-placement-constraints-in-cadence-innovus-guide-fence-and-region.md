---
layout: post
title: 'VLSI DESIGN - Placement Constraints in Cadence Innovus: Guide, Fence, and
  Region'
date: 2024-06-13 23:52 +0200
author: me
categories: [hw, VLSI, Innovus]
tags: [Innovus, VLSI]
---
In VLSI physical design, specifically during the Place and Route (PnR) phase, controlling the placement of standard cells or modules is crucial for achieving optimal performance and meeting design constraints. Cadence Innovus, a leading tool for automated place and route, offers three types of placement constraints: Guide, Fence, and Region. Each serves a unique purpose and has distinct characteristics.

## Guide: Soft Placement Suggestion
A Guide in Cadence Innovus is essentially a soft constraint. It is used to suggest a preferred area for certain cells in the design, but it does not strictly enforce this placement. This means that:

- The assigned cells can be placed outside the defined box.
- Other cells can also be placed inside the box.
- This flexibility can be useful when you want to influence the placement of certain cells without strictly enforcing it, allowing the tool some freedom to optimize the overall design.

Command Example:

```tcl
createGuide obj_name 100.000 200.000 200.000 400.000
```

This command creates a guide for the specified object within the coordinates (100, 200) to (200, 400).

## Fence: Hard Exclusive Constraint
A Fence is a hard constraint that strictly controls the placement of specific cells. When you define a fence:

- The assigned cells are not allowed to be placed outside the defined box.
- No other cells are allowed inside the box.
- This makes the fenced area exclusively reserved for the assigned cells, ensuring that they remain within the specified region. This can be particularly useful for critical cells that need to be isolated for performance or noise considerations.

Command Example:

```tcl
createFence obj_name 100.000 200.000 200.000 400.000 
```
This command creates a fence for the specified object within the coordinates (100, 200) to (200, 400).

## Region: Hard Inclusive Constraint
A Region is similar to a fence in that it is a hard constraint, but with a key difference:

- The assigned cells must be placed within the defined box.
- Other cells are allowed to be placed inside the box.
- This means that while the specified cells are constrained to the region, the area can also accommodate other cells, potentially leading to congestion if not managed wisely. This constraint is useful when you want to cluster certain cells together but still allow for some flexibility in cell placement.

Command Example:
```tcl
createRegion obj_name 100.000 200.000 200.000 400.000
```
This command creates a region for the specified object within the coordinates (100, 200) to (200, 400).

## Deleting Placement Constraints
To remove any of these placement constraints, you can use the unplaceGuide command. This command deletes the guide, fence, or region associated with the specified module.

Command Example:
```tcl
unplaceGuide obj_name
```
This command deletes the placement constraint for the specified object.

By leveraging these placement constraints effectively in Cadence Innovus, designers can guide the tool to achieve the desired placement results while ensuring compliance with design rules and constraints.
