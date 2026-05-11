---
layout: post
title: Verdi Python NPI for RTL Debug
date: 2026-05-11 19:30 +0200
author: me
categories: [eda, vcs, verdi]
tags: [verdi, vcs, fsdb, npi, rtl-debug, python]
---

Most RTL debug still starts in a very familiar way: run VCS, open Verdi, load the FSDB, and stare at waveforms until the behavior makes sense. That works. I still do it. The problem is that this flow keeps too much information inside the GUI and inside my head.

If I want a script, regression system, or LLM agent to help with RTL debug, I need a way to ask small questions and get structured answers:

- What signals are dumped under this DUT?
- What value did this signal have at this time?
- When did this FSM move from one state to another?
- Which source line declares or drives this object?
- What are the drivers and loads of this net?

That is where Verdi NPI becomes useful.

NPI means **Native Programming Interface**. In practice, Verdi Python NPI is a Python interface to Verdi's internal debug databases: waveform, design hierarchy, HDL source objects, netlist connectivity, coverage, UPF, and transaction data. VCS is still the simulator. NPI is the programmatic debug handle on top of the databases produced by the simulation and Verdi flow.

## Mental Model

The important separation is:

| Tool or database | Role |
|------------------|------|
| VCS | Compile and run the simulation |
| KDB / debug database | Preserve design, hierarchy, and source debug information |
| FSDB | Store waveform values over time |
| Verdi GUI | Interactive human debug |
| Verdi Python NPI | Programmatic access to Verdi debug data |

So NPI is not a replacement for simulation. VCS still runs the design. NPI is the Python handle I can use after the run, when I want to turn the debug database into objects, queries, and evidence.

A useful RTL debug loop looks like this:

```text
RTL + TB + filelists
        |
        v
VCS compile with debug database
        |
        v
Simulation run with FSDB
        |
        v
Python NPI queries
        |
        v
JSON evidence for scripts, CI, or LLM agents
```

## Main Python NPI Models

The Verdi Python NPI package is organized into several models. For RTL debug, I usually start with these three:

| Model | Useful for |
|-------|------------|
| `waveform` | Open FSDB, list scopes/signals, query values, query value changes, search X/value events |
| `netlist` | Query instances, ports, nets, drivers, loads, fan-in, fan-out |
| `lang` | Map elaborated HDL objects back to source files and lines |

There are other models such as coverage, UPF, text, transaction waveform, and waveform writer. But for day-to-day RTL debugging, waveform + netlist + language already gives a very strong handle.

## Compile and Simulation Requirements

Before using NPI, the simulation flow needs to leave enough debug data behind. Otherwise Python has nothing useful to query.

For VCS, a practical baseline is:

```makefile
VCS_OPTS += -full64 -sverilog -timescale=1ns/1ps
VCS_OPTS += -debug_access+all -kdb
VCS_OPTS += -f filelists/rtl.f -f filelists/tb.f
```

For FSDB dumping, I prefer to guard the dumping code with a compile define:

```systemverilog
`ifdef TB_ENABLE_FSDB
initial begin
  string fsdb_file;
  if ($test$plusargs("dump_fsdb")) begin
    if (!$value$plusargs("fsdbfile=%s", fsdb_file)) begin
      fsdb_file = "sim.fsdb";
    end
    $fsdbDumpfile(fsdb_file);
    $fsdbDumpvars(0, tb_top);
  end
end
`endif
```

Then the Makefile can enable FSDB only when needed:

```makefile
ifeq ($(FSDB),1)
VCS_FSDB_OPTS += +define+TB_ENABLE_FSDB
SIM_FSDB_ARGS := +dump_fsdb +fsdbfile=$(FSDB_FILE)
endif
```

This avoids hard-wiring waveform dumping into every simulation.

## Minimal Python NPI Flow

The Python package is under the Verdi installation:

```python
import os
import sys

verdi_home = os.environ["VERDI_HOME"]
sys.path.append(os.path.join(verdi_home, "share", "NPI", "python"))
```

For waveform queries:

```python
from pynpi import npisys, waveform

npisys.init(["npi_wave_probe"])

fsdb = waveform.open("sim.fsdb")
print(fsdb.scale_unit())
print(fsdb.min_time(), fsdb.max_time())

sig = fsdb.sig_by_name("i2c_slave_tb.dut.state_q")
events = waveform.sig_hdl_value_between(
    sig,
    0,
    fsdb.max_time(),
    waveform.VctFormat_e.ObjTypeVal,
)

waveform.close(fsdb)
npisys.end()
```

For source and netlist queries, load the design:

```python
from pynpi import lang, netlist, npisys

argv = [
    "-sv",
    "-f", "filelists/rtl.f",
    "-f", "filelists/tb.f",
    "-top", "i2c_slave_tb",
]

npisys.init(argv)
npisys.load_design(argv)

hdl = lang.handle_by_name("i2c_slave_tb.dut.state_q", None)
print(hdl.file(), hdl.line_no())

net = netlist.get_net("i2c_slave_tb.dut.rx_valid_q")
print([pin.full_name() for pin in net.driver_list()])
print([pin.full_name() for pin in net.load_list()])

npisys.end()
```

## Build a JSON Debug Wrapper

The first wrapper I would build is intentionally boring. It should answer a few common debug questions, print clean JSON, and put any native tool chatter into a separate log file.

This matters because Synopsys shared libraries can print banners and messages to stdout or stderr. That is fine when I am running a command by hand. It is painful when another script, CI job, or LLM agent expects valid JSON.

The useful commands are:

| Command | Meaning |
|---------|---------|
| `info` | Show Verdi home, FSDB path, filelists, top, DUT |
| `wave-summary` | Show FSDB version, time range, time scale, top scopes |
| `signals` | List dumped signals in a scope |
| `events` | Show value changes for a signal in a time window |
| `value` | Show one signal value at one time |
| `find` | Search X or a specific value forward/backward |
| `source` | Map a hierarchical HDL object to source file and line |
| `design` | Summarize loaded netlist and DUT ports |
| `net` | Show drivers and loads for a net |

Example Makefile integration:

```makefile
NPI_DEBUG ?= python3 scripts/verdi_npi_debug.py
FSDB_FILE ?= sim.fsdb
RTL_FILELIST ?= filelists/rtl.f
TB_FILELIST ?= filelists/tb.f
TB_TOP ?= i2c_slave_tb
DUT_SCOPE ?= $(TB_TOP).dut

npi-wave: ## Summarize the current FSDB with Verdi Python NPI
	@$(NPI_DEBUG) --fsdb $(FSDB_FILE) \
	  --rtl-filelist $(RTL_FILELIST) \
	  --tb-filelist $(TB_FILELIST) \
	  --top $(TB_TOP) \
	  --dut $(DUT_SCOPE) \
	  wave-summary

npi-events: ## Show value changes for NPI_SIGNAL from NPI_BEGIN to NPI_END
	@$(NPI_DEBUG) --fsdb $(FSDB_FILE) \
	  events $(NPI_SIGNAL) $(NPI_BEGIN) $(NPI_END) \
	  --format $(NPI_FORMAT)
```

With that in place, the debug loop becomes command-line friendly:

```bash
make npi-wave
make npi-signals NPI_PATTERN='state|valid|ready|data|ack'
make npi-events NPI_SIGNAL=i2c_slave_tb.dut.state_q NPI_BEGIN=0 NPI_END=30000000
make npi-source NPI_SIGNAL=i2c_slave_tb.dut.rx_data_q NPI_CONTEXT=5
make npi-net NPI_NET=i2c_slave_tb.dut.rx_valid_q
```

## Case Study: I2C Slave Debug

I tested this flow on a small I2C slave design. The original smoke test passed:

```text
[14745000] [TEST] I2C_SLAVE_TB_PASS
```

But a passing smoke test does not prove the design is robust. It only says the happy path survived. NPI made it cheap to ask more annoying questions.

First, I listed the dumped DUT signals:

```bash
make npi-signals NPI_PATTERN='state|scl_|sda_|ack|read|rx_|tx_|stop|selected|bit_idx|shift'
```

Then I checked the FSM state sequence:

```bash
python3 scripts/verdi_npi_debug.py events \
  i2c_slave_tb.dut.state_q 0 14745000 \
  --format ObjTypeVal
```

The state trace showed the expected write, read, and address-miss sequence:

```json
[
  {"time": 175000, "value": "ST_ADDR"},
  {"time": 2095000, "value": "ST_ADDR_ACK"},
  {"time": 2415000, "value": "ST_WRITE"},
  {"time": 4255000, "value": "ST_WRITE_ACK"},
  {"time": 7225000, "value": "ST_READ"}
]
```

The useful bugs appeared when I stopped looking at the happy path and started testing valid-ready style corner cases.

## Bug 1: RX Data Was Overwritten Under Backpressure

The original testbench kept `rx_ready=1` all the time. I added a targeted test where `rx_ready=0`, so the downstream side was not ready to consume a received byte.

The expected behavior for a one-entry output buffer is:

1. Accept the first byte if the output slot is empty
2. Hold `rx_valid=1` and keep `rx_data` stable
3. NACK or ignore additional bytes until `rx_ready=1`

Before the fix, NPI showed this:

```json
{
  "signal": "rx_data_q",
  "events": [
    {"time": 4245000, "value": "a5"},
    {"time": 6595000, "value": "5a"}
  ]
}
```

`rx_valid_q` stayed high, `rx_ready` stayed low, but `rx_data_q` still changed from `0xa5` to `0x5a`. That means the unaccepted data was overwritten.

The fix was to make the receive side behave like a depth-1 buffer:

```verilog
assign rx_can_accept = !rx_valid_q || rx_ready;

if (bit_idx_q == 3'd0) begin
  write_ack_q <= selected_q && rx_can_accept;
  if (selected_q && rx_can_accept) begin
    rx_data_q <= {shift_q[7:1], sda_sync_q};
    rx_valid_q <= 1'b1;
  end
  state_q <= ST_WRITE_ACK;
end
```

After the fix, the NPI trace showed that `rx_data_q` stayed at the first accepted byte while `rx_ready=0`:

```json
{
  "signal": "rx_data_q",
  "events": [
    {"time": 13875000, "value": "c3"}
  ]
}
```

The blocked byte was NACKed and did not overwrite the pending payload.

## Bug 2: Read Data Depended on Live `tx_valid`

The original read path loaded `tx_shift_q` when the address phase completed:

```verilog
tx_shift_q <= tx_data;
tx_ready_q <= tx_valid;
state_q <= ST_READ;
```

But the actual SDA drive used live `tx_valid`:

```verilog
assign read_drive = (state_q == ST_READ) && selected_q && tx_valid && !tx_shift_q[bit_idx_q];
```

This creates an ambiguous handshake. If `tx_ready` means "the slave accepted the byte", then the producer is allowed to drop `tx_valid` after seeing `tx_ready`. In that case, the slave should continue shifting the already latched `tx_shift_q`.

Before the fix, when the producer dropped `tx_valid` after `tx_ready`, the master read `0xff` instead of the payload. NPI showed:

```json
{
  "tx_shift_q": [
    {"time": 2405000, "value": "3c"}
  ],
  "tx_valid": [
    {"time": 2415000, "value": "0"}
  ],
  "read_drive": [
    {"time": 2405000, "value": "1"},
    {"time": 2415000, "value": "0"}
  ]
}
```

The byte was latched, but drive stopped because `tx_valid` dropped.

The fix was to add a small `tx_active_q` state bit:

```verilog
tx_shift_q <= tx_data;
tx_active_q <= tx_valid;
tx_ready_q <= tx_valid;
state_q <= ST_READ;
```

And drive from the accepted transfer, not live `tx_valid`:

```verilog
assign read_drive = (state_q == ST_READ) && selected_q && tx_active_q && !tx_shift_q[bit_idx_q];
```

After the fix, NPI showed `tx_valid` dropping while `tx_active_q` stayed high for the read transfer:

```json
{
  "tx_valid": [
    {"time": 19155000, "value": "0"}
  ],
  "tx_active_q": [
    {"time": 19105000, "value": "1"},
    {"time": 21185000, "value": "0"}
  ]
}
```

The master then read the correct byte.

## Why NPI Helped

The key advantage was not only that NPI could read the waveform. A waveform GUI can do that. The advantage was that the debug evidence became structured:

- Signal names were exact hierarchical names
- Time windows were bounded
- Values were emitted as JSON
- FSM enum values were readable
- Source lines could be queried directly from the elaborated design
- Net drivers and loads could be checked without opening a schematic

This is especially useful for LLM-assisted debugging because the model can ask small, repeatable questions:

```text
Show all events on rx_valid_q and rx_data_q between 13 us and 17 us.
Map rx_data_q to the source line.
Show the driver/load relationship for rx_valid_q.
Search for the first X on state_q.
```

Instead of asking an LLM to reason from a giant waveform screenshot, we can give it a clean state trace and a few source snippets.

## How I Would Use It in a Real Debug Session

I would not start by building a large NPI framework. That is usually the wrong instinct. I would start from the existing VCS flow, keep the same filelists, and add only enough debug access to answer the next question.

For source and netlist debug, I compile with `-debug_access+all -kdb`. For waveform debug, I enable FSDB dumping only when I need it. This keeps the normal simulation path close to the original one, and it avoids turning every run into a heavy waveform run.

When the simulation fails, I try to make the first NPI query small:

- Show the FSM state changes around the failure.
- Show `valid`, `ready`, and data events in the failing time window.
- Map the suspicious signal back to the RTL source line.
- Check the driver and load list before assuming where a value came from.

That small-window habit is important. A giant waveform dump can make an LLM response worse, not better. A short JSON trace with exact signal names and times is much easier to reason about.

After NPI points to a likely bug, I still go back to the RTL and read the code like a normal engineer. The tool gives evidence; it does not replace understanding. In the I2C bugs above, the useful sequence was:

1. Use NPI to capture the failing signal behavior.
2. Use source mapping to jump to the relevant RTL.
3. Fix the handshake or buffering rule.
4. Re-run the simulation.
5. Add the corner case as a regression test.

That last step is the part I would not skip. If NPI helps find a real bug, the bug should become a normal regression, not just a nice debug story.

## Conclusion

Verdi Python NPI is a practical bridge between traditional RTL debug and automation. It lets us keep the normal VCS and Verdi flow, while exposing the important debug information in a scriptable way.

For human engineers, it reduces repetitive GUI work. For CI and LLM agents, it provides the missing handle: structured access to waveform, hierarchy, source, and netlist data.

The most useful pattern is simple:

```text
Run VCS -> dump FSDB -> query with NPI -> produce JSON -> debug from evidence
```

Once this loop exists, even small designs become much easier to debug systematically.

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
