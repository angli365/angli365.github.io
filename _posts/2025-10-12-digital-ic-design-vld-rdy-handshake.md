---
layout: post
title: Digital IC Design - VLD-RDY Handshake
date: 2025-10-12 17:23 +0200
categories: [hw, digital-ic-design]
tags: [valid-ready, handshake, pipeline, digital-design]
---

# Digital IC Design - VLD-RDY Handshake Protocol

The Valid-Ready (VLD-RDY) handshake protocol is a fundamental communication mechanism in digital IC design, particularly crucial for pipeline design and data flow control. This protocol ensures reliable data transfer between modules while preventing data loss and maintaining system integrity.

## Understanding the VLD-RDY Protocol

The VLD-RDY handshake uses two key signals:
- **Valid (VLD)**: Indicates that the current data is valid and ready to be consumed
- **Ready (RDY)**: Indicates that the receiving module is ready to accept new data


### VLD-RDY in Single-Cycle Pipeline Operations
For a single-cycle operation, which is the most common case, the handshake logic is straightforward:

1. **Ready Signal Logic**: `up_rdy_i = ~dn_vld_o || dn_rdy_i`
   - **Case 1**: `~dn_vld_o` - When this stage has no valid data (idle state), it can always accept new data
   - **Case 2**: `dn_rdy_i` - When the downstream stage is ready, this stage can accept new data even if it currently has valid data, because the current data will be consumed

2. **Valid Signal Logic**: The valid signal is simply registered from the upstream when ready is asserted
   - This ensures data is only updated when the stage can actually accept it

3. **Data Flow**: Data flows through in a single clock cycle, making this suitable for simple operations like addition, subtraction, or simple logic operations.


#### Example

```verilog
// Upstream ready: tells the upstream module this module is ready to receive data
// Enable when this stage is not valid (can receive data) OR 
// the next stage is ready to receive data (current data is consumed)
assign up_rdy_i = ~dn_vld_o || dn_rdy_i;

// Downstream valid: tells the downstream module this module's data is valid
// It is registered from up_vld_i when up_rdy_i is asserted
always_ff @(posedge clk or negedge arst_n) begin
  if (~arst_n) begin
    dn_vld_o <= {$bits(dn_vld_o){1'b0}};
  end
  else begin
    if (up_rdy_i) begin
      dn_vld_o <= up_vld_i;
    end
  end
end
```

### VLD-RDY in Multi-Cycle Pipeline Operations

For multi-cycle operations, we need additional logic to track computation progress:

1. **Stage Done Signal**: `stage_done` indicates when the computation is complete
2. **Modified Ready Logic**: `up_rdy_i = ~dn_vld_o || (stage_done && dn_rdy_i)`
   - Now we can only accept new data when the current computation is done AND downstream is ready
3. **Valid Signal Logic**: `dn_vld_o = recv_valid && stage_done`
   - Data is only valid when it has been received AND computation is complete

This prevents data corruption by ensuring new data isn't accepted until the current computation is finished.

## Multi-Cycle Operations

For stages that require more than one clock cycle to complete their computation, we need to add a `stage_done` signal:

```verilog
// Modified ready logic for multi-cycle operations
assign up_rdy_i = ~dn_vld_o || (stage_done && dn_rdy_i);
assign dn_vld_o = recv_valid && stage_done;

always_ff @(posedge clk or negedge arst_n) begin
    if (~arst_n) begin
      dn_vld_o <= {$bits(dn_vld_o){1'b0}};
    end
    else begin
      if (up_rdy_i) begin
        recv_valid <= up_vld_i;
      end
    end
end
```

## Design Example

Let's examine a practical example of a simple 2-stage pipeline that demonstrates both single-cycle and multi-cycle operations:

- **Stage 1**: Single-cycle increment operation (data + 1)
- **Stage 2**: Multi-cycle square operation (2 clock cycles)

### Key Design Principles

The core idea is to ensure data safety at each pipeline stage, preventing new data from overwriting current data before it's been consumed. Each pipeline stage can be thought of as a depth-1 FIFO.

### Pipeline Stage Analysis

#### Stage 1 (Single-Cycle Increment)

```verilog
// Stage 1: Single-cycle increment operation
assign stage1_done = 1'b1;  // Always done in one cycle
assign stage1_rdy = ~stage1_vld || (stage1_done && stage2_rdy);
assign stage1_to_stage2_vld = stage1_vld && stage1_done;

always_ff @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        stage1_vld <= 1'b0;
        stage1_data <= 32'h0;
    end
    else begin
        if(stage1_rdy) begin
            stage1_vld <= up_vld_i;
            if(up_vld_i) begin
                stage1_data <= up_data_i + 1'b1;  // Increment operation
            end
        end
    end
end
```

**Ready Logic Analysis:**
- `stage1_rdy` is asserted when:
  1. `~stage1_vld` - Stage is idle (no valid data), can accept new data
  2. OR `stage1_done && stage2_rdy` - Current data is processed AND next stage can accept it

#### Stage 2 (Multi-Cycle Square)

```verilog
// Stage 2: Multi-cycle square operation (2 cycles)
assign stage2_done = compute_done[1];  // Done after 2 cycles
assign stage2_rdy = ~stage2_vld || (stage2_done && dn_rdy_i);
assign stage2_to_dn_vld = stage2_vld && stage2_done;

// Computation done counter (2-bit shift register)
always_ff @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        compute_done <= 2'b00;
    end
    else begin
        if(stage2_rdy && stage1_to_stage2_vld) begin
            compute_done <= 2'b00;  // Reset when new data arrives
        end
        else if(stage2_vld) begin
            compute_done <= {compute_done[0], 1'b1};  // Shift register
        end
    end
end

always_ff @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        stage2_vld <= 1'b0;
        stage2_data <= 32'h0;
    end
    else begin
        if(stage2_rdy) begin
            stage2_vld <= stage1_to_stage2_vld;
            if(stage1_to_stage2_vld) begin
                stage2_data <= stage1_data;  // Store data for computation
            end
        end
    end
end

// Square computation (combinational)
assign square_result = stage2_data * stage2_data;
```

**Key Differences for Multi-Cycle Operations:**
- Uses a 2-bit shift register (`compute_done`) to track computation progress
- `stage2_rdy` considers both completion status and downstream readiness
- Data is only passed downstream when computation is complete (after 2 cycles)

<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>