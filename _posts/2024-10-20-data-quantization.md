---
layout: post
title: DATA Quantization
date: 2024-10-20 19:40 +0200
categories: [INTRO, AI, SW-HW]
tags: [quantization]
math: true
---

Data quantization is a crucial technique in the field of neural networks, particularly when it comes to deploying models on hardware with limited computational resources. In this blog post, we'll explore what data quantization is, why it's important, and how it's implemented in neural networks.

## What is Data Quantization?

At its core, quantization is the process of mapping a continuous range of values to a discrete set of values. In the context of neural networks, this typically involves converting floating-point numbers (which can have a wide range of values) to fixed-point or integer representations with a smaller number of bits.

For example, we might convert 32-bit floating-point numbers to 8-bit integers. This reduction in precision allows for more efficient computation and storage, which is especially beneficial for edge devices and mobile applications.

## The Mathematics of Quantization

Quantization can be described as an affine mapping from a continuous range $f \in [f_{min}, f_{max}]$ to a discrete set of values $q \in [q_{min}, q_{max}]$ using a scale factor $s$ and a zero-point $z$. The basic equation for this mapping is:

$$
q = \text{round}(\frac{f}{s} + z)
$$

The scale factor $s$ and zero-point $z$ are calculated as follows:

$$
s = \frac{f_{max} - f_{min}}{q_{max} - q_{min}}
$$

$$
z = \text{round}(q_{max} - \frac{f_{max}}{s})
$$

## Range Clipping in Quantization

To improve quantization efficiency, we often clip the input values to a defined range $[f_{min}, f_{max}]$. This is equivalent to clipping the quantized values to $[q_{min}, q_{max}]$. The quantization equation with clipping becomes:

$$
q = \text{clip}(\text{round}(\frac{f}{s} + z), q_{min}, q_{max})
$$

This approach focuses the available bits on common values rather than rare outliers, which can improve overall accuracy.

## Hardware Considerations

For efficient hardware implementation, fixed-point representation is often preferred. By defining the scale factor $s$ as a power-of-2 value ($2^{-frac\_bw}$) and the zero-point $z$ as 0, we can obtain a fixed-point value $fxq$:

$$
fxq = s * \text{clip}(\text{round}(\frac{f}{s}), q_{min}, q_{max})
$$

## Quantization Schemes

There are two main approaches to implementing quantization in neural networks:

1. **Quantization Aware Training (QAT)**: This involves training the neural network with quantization constraints from the beginning. It allows the network to adapt to the reduced precision during the training process, often resulting in better performance for the quantized model.

2. **Post-Training Quantization**: This technique involves quantizing a pre-trained neural network. It's simpler to implement but may result in some loss of accuracy compared to QAT.

## Implementing Quantization in Python

Here's a simple Python function that demonstrates how to quantize a floating-point number to a fixed-point representation:

```python
import numpy as np

def quantize(fx, bw, frac_bw):
    """
    Quantize a float number to a fixed number of bits.
    """
    # get the max and min value of the quantized number
    max_val = 2**(bw-1) - 1
    min_val = -2**(bw-1)
    
    scale_factor = 2**frac_bw
    
    # scale the float number to the range of [min_val, max_val]
    fx = fx * scale_factor
    fx = np.clip(fx, min_val, max_val)
        
    # round the float number to the nearest integer
    qx = np.round(fx)

    # scale back to the original range
    qx = qx / scale_factor
    
    return qx

```

## Quiz
A neural network has weights with value range $(-2, 2)$. To run it on a hardware only supports 8-bit fixed-point computation,
1. What is the best scale factor to do the weight quantization? *Cover the value range first, then use the remaining bits to maximize precision.*
2. Use the scale factor to quantize the floating point number $f$ = 0.101 to a fixed-point number.

<details>
<summary>Click to show answer 1</summary>

1. To cover the range of $(-2, 2)$, we need 2 bits for the integer part (including 1 sign bit), leaving 6 bits for the fractional part. Thus, the scale factor $s$ is $2^{-6} = 1/64$. 
</details>

<details>
<summary>Click to show answer 2</summary>

2. Quantizing $f = 0.101$: 
$$
\begin{align*}
fxq &= 1/64 * clip(round(0.101 * 64), -2^{7}, 2^{7}-1) \\
&= 1/64 * clip(round(6.464), -128, 127) \\
&= 1/64 * 6 \\
&= 0.09375
\end{align*}
$$
</details>

[FP to FXP Converter](https://venerable-biscuit-cefbbb.netlify.app/).

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
