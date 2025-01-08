---
layout: post
title: Understanding Round and Saturation in Hardward Design
date: 2024-03-21 7:52 +0100
author: me
categories: [hw, verilog]
tags: [verilog, fixed-point, rounding, saturation]
---

In many digital signal processing applications, it's often necessary to perform operations that involve fixed-point arithmetic. When working with fixed-point numbers, we need to be mindful of potential overflow and precision loss situations. Two common techniques used to handle these scenarios are rounding and saturation. In this blog post, we'll explore what these techniques are and how they can be implemented in Verilog.

## Rounding
Rounding is a technique used to approximate a fixed-point number to a desired precision level. It's particularly useful when you need to convert a number from a higher precision format to a lower precision format, or when you want to reduce the number of fractional bits after a mathematical operation.

Consider the following example: You have a 16-bit fixed-point number with 8 fractional bits, and you want to convert it to a 16-bit fixed-point number with 4 fractional bits. In this case, you need to round the least significant 4 fractional bits to the nearest value representable with the new format.

The Verilog code for a rounding unit is as follows:

```verilog

module round #(
    parameter DIN_WIDTH = 16,
    parameter DIN_FRAC_WIDTH = 8,
    parameter DOUT_FRAC_WIDTH = 4
)(
    input  signed [DIN_WIDTH-1:0] din,
    output signed [DIN_WIDTH-1:0] dout
);

localparam DOUT_WIDTH = DIN_WIDTH;
localparam TRUNCATE_FRAC_WIDTH = DIN_FRAC_WIDTH - DOUT_FRAC_WIDTH;

// Rounding
assign dout = $signed((din + { {(DOUT_WIDTH-TRUNCATE_FRAC_WIDTH){1'b0} }, 
               {din[TRUNCATE_FRAC_WIDTH], {(TRUNCATE_FRAC_WIDTH-1){!din[TRUNCATE_FRAC_WIDTH]}}}}))
               >>> TRUNCATE_FRAC_WIDTH;

endmodule
```

In this code, we're adding a value to the input din based on the least significant bit of the bits that will be truncated. If this bit is 1, the added value is 0.5, and if it's 0, the added value is approch to 0.5 but not exactly 0.5. By doing this, if the fractional part of `din` is 0.5, then `dout` is the even integer nearest to `din`. For example, 23.5 becomes 24, as does 24.5; however, −23.5 becomes −24, as does −24.5. The resulting sum is then right-shifted by TRUNCATE_FRAC_WIDTH bits to obtain the rounded output dout.

This type of rounding is known as `rounding half to even` or `bankers rounding`. It is the default rounding mode used in many numerical systems, including IEEE 754 floating-point arithmetic. In matlab, this also called [`convergent rounding`](https://mathworks.com/help/fixedpoint/ref/convergent.html).

## Saturation
Saturation is a technique used to handle overflow situations in fixed-point arithmetic. When the result of an operation exceeds the representable range of the output format, saturation clamps the result to the maximum or minimum value representable in that format.

For example, if you have a 16-bit signed input and want to saturate it to a 12-bit signed output, any input value greater than the maximum 12-bit signed value (2047) or less than the minimum 12-bit signed value (-2048) will be clamped to the corresponding maximum or minimum value.

The Verilog code for a saturation unit is as follows:

```verilog

module saturate #(
    parameter DIN_WIDTH             = 16,
    parameter DOUT_WIDTH            = 12
)(
    input  signed       [DIN_WIDTH-1:0]            din,
    output signed       [DOUT_WIDTH-1:0]           dout
);

localparam TRUNCATE_MSB = DIN_WIDTH - 1;
localparam TRUNCATE_LSB = DOUT_WIDTH -1;

wire not_overflow;

assign not_overflow = (&din[TRUNCATE_MSB:TRUNCATE_LSB]) | !(|din[TRUNCATE_MSB:TRUNCATE_LSB]);

/* if overflow, saturate din to MAX or MIN per DOUT_WIDTH
*/
assign dout = not_overflow ? din[DOUT_WIDTH-1:0]:
                            {din[DIN_WIDTH-1], {(DOUT_WIDTH-1){!din[DIN_WIDTH-1]}}};

endmodule

```
In this code, we first determine if an overflow has occurred after truncating DIN_WIDTH to DOUT_WIDTH by checking if any of the bits in the truncated range are different from the sign bit. If all the bits are the same as the sign bit, then no overflow has occurred. If any of the bits are different, then an overflow has occurred, and we need to saturate the output.

If an overflow is detected, we assign the output dout to the maximum or minimum value representable in the output format, based on the sign bit of the input din. If no overflow occurs, we simply truncate the input to the desired output width.

By employing rounding and saturation techniques, we can ensure that our fixed-point arithmetic operations produce meaningful results, even when dealing with precision loss or overflow scenarios. These techniques are essential for designing robust and reliable hardware systems that can handle a wide range of input conditions and maintain accuracy in their computations.

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
