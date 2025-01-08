---
layout: post
title: VCS Cheat Sheet
date: 2024-02-18 11:12 +0100
author: me
categories: [eda, vcs]
tags: [vcs]
---

Synopsys VCS is a popular Verilog simulator. Here are some useful commands:

```makefile

# Variables
TB_TOP ?= 
RTL_TOP ?= 
ASSERT_TOP ?= 
RTL_DIR ?= 
ASSERT_DIRS ?= 
INC_DIRS ?=

VLIBS ?=
DEFINES ?=


UVM_HOME ?= 

VERDI_HOME ?= 

SEARCH_PATHS = -y $(RTL_DIR) +libext++.v +libext+.sv
SEARCH_PATHS += -y $(ASSERT_DIRS) +libext+.vlib+.v+.vv+.sv

LD_LIBRARY_PATH := $(VERDI_HOME)/share/PLI/VCS/LINUX64

VCS = vcs
VCS_OPTS = -full64 -kdb -debug_acc+pp+f+dmptf -debug_region+cell+encrypt -sverilog -timescale=1ns/1ps
VCS_OPTS += -P $(LD_LIBRARY_PATH)/novas.tab $(LD_LIBRARY_PATH)/pli.a
VCS_OPTS += $(foreach dir, $(INC_DIRS), +incdir+$(dir))
VCS_OPTS += $(foreach def, $(DEFINES), +define+$(def))
VCS_OPTS += $(foreach lib, $(VLIBS), -v $(lib))

ifneq ($(UVM_HOME),)
# VCS_OPTS += -ntb_opts uvm-1.2
VCS_OPTS += +incdir+$(UVM_HOME)/src $(UVM_HOME)/src/uvm.sv
VCS_OPTS += $(UVM_HOME)/src/dpi/uvm_dpi.cc -CFLAGS -DVCS
endif

OUTPUT = simv
WAVES = +waves_on
FSDB_FILE = $(RTL_TOP_MODULE).fsdb

ASSERT_FILES = $(ASSERT_DIRS)/$(ASSERT_TOP)

# Default target
all: compile run

# Compile the design and testbench using search paths and file list
compile:
	$(VCS) $(VCS_OPTS) $(SEARCH_PATHS) $(TB_TOP) $(RTL_TOP) $(ASSERT_FILES) -o $(OUTPUT) -l compile.log

# Run the simulation
run:
	./$(OUTPUT) $(WAVES) +fsdb=$(FSDB_FILE) | tee sim.log

# Clean up the generated files
clean:
	rm -rf csrc $(OUTPUT)* ucli.key vc_hdrs.h DVEfiles/ *.vpd *.log verdiLog novas*

# verdi launch
verdi:
	verdi -simflow -simBin $(OUTPUT) -ssf $(FSDB_FILE) &


fsdb2saif:
	fsdb2saif $(FSDB_FILE) -o $(FSDB_FILE).saif


echo_var:
	@echo "RTL_TOP: $(RTL_TOP)"
	@echo "RTL_DIR: $(RTL_DIR)"
	@echo "SEARCH_PATHS: $(SEARCH_PATHS)"

.PHONY: all compile run clean verdi echo_var

``` 

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
