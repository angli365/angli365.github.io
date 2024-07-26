---
layout: post
title: VLSI Design - Engineering Change Order (ECO)
date: 2024-07-26 19:43 +0200
author: me
categories: [hw, VLSI]
tags: [ECO, VLSI]
---

Let's talk about something that keeps chip designers up at night: making last-minute changes to their designs. In the world of integrated circuit (IC) design, mistakes happen, and sometimes you need to tweak things even when you're almost at the finish line. That's where ECO, or **Engineering Change Order**, comes into play. Today, we're going to break down ECO, with a special focus on pre-mask and post-mask approaches.

## What's ECO all about?
Think of ECO as the "oops, let me fix that real quick" of chip design. It's a way to manually modify an integrated circuit, usually by directly changing the netlist. When you're knee-deep in the design process and spot an issue, ECO lets you make necessary changes without throwing the whole design out the window.

## Why should you care about ECO?
It's all about saving time and money. As you move through the stages of chip design, making changes becomes more and more expensive and time-consuming. ECO is like a safety net that lets designers make critical fixes without starting from scratch. It's a real lifesaver when you're racing against the clock and trying to stay within budget.

## Types of ECO

ECOs can be categorized in several ways, but one of the most important distinctions is between `pre-mask` and `post-mask` ECO.

1. The ECOs that occur between the freeze and tapeout stages are called **`pre-mask ECOs`**; 

2. After tapeout, the ECOs that take place during the period when the chip's transistors have been fabricated but the interconnections have not yet been made are called **`post-mask ECOs`**.


### Pre-mask ECO

Pre-mask ECO occurs before the chip's masks are created for manufacturing. This type of ECO is more flexible and allows for more extensive changes.

Key characteristics of pre-mask ECO:
- Can be performed anytime before tapeout
- Allows for addition, removal, or modification of cells
- Can address issues found in static timing analysis or post-simulation
- Requires formal verification to ensure functionality

### Post-mask ECO

Post-mask ECO happens after the chip's transistor layers have been manufactured but before the metal layers are added. This type of ECO is more limited but can still be crucial for fixing last-minute issues.

Key characteristics of post-mask ECO:
- Utilizes pre-designed spare cells or freed cells
- Limited to modifying metal layers for connections
- Cannot add new cells, only repurpose existing ones
- Typically addresses functional bugs or timing issues discovered during testing

## ECO Implementation Techniques

When implementing ECOs, designers use various techniques:

1. **Cell Resizing**: Changing the size of existing cells to adjust their drive strength or timing characteristics.

2. **Buffer Insertion**: Adding buffers to improve signal integrity or meet timing requirements.

3. **Logic Modification**: Altering the logical function of the circuit, often using spare cells or freed cells.

4. **Routing Changes**: Modifying the interconnections between cells, particularly in post-mask ECO.

## Considerations for Successful ECO

To ensure successful ECO implementation, designers should keep the following in mind:

1. **Minimal Physical Impact**: Strive for changes that cause minimal displacement of existing cells to maintain overall design stability.

2. **Timing Closure**: Carefully consider the impact of changes on timing across all corners and modes (MCMM - Multi-Corner Multi-Mode).

3. **Metal Fill**: For advanced processes, remember that changes may require adjustments to metal fill, which can affect timing.

4. **VT Cell Usage**: Leverage different threshold voltage (VT) cells for fine-tuning timing without extensive rerouting.

5. **Incremental Design**: Maintain an incremental approach to minimize the ripple effect of changes.

## Conclusion

ECO is a critical tool in the IC designer's toolkit, allowing for necessary modifications late in the design process. By understanding the differences between pre-mask and post-mask ECO and following best practices, designers can efficiently address issues and improve their designs without incurring excessive costs or delays. As chip designs become increasingly complex, mastering ECO techniques will remain an essential skill for IC designers.