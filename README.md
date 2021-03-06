# RT-LIFE: An Interface to easily attach Integrity Monitors to IoT-class processors

**Description**  
RT-LIFE is a framework, which standardizes an interface between a processor and an attached security monitor. Based on a unified behavior, its main goal is high portability for compatible security monitors. Existing interfaces are optimized for tracing [[1]](https://github.com/riscv/riscv-trace-spec), debugging [[2]](https://github.com/riscv/riscv-debug-spec) or formal verification [[3]](https://github.com/SymbioticEDA/riscv-formal) and focus on already commited instructions. As an alternative, RT-LIFE is fitted for minimal latency and focuses on uncommited instructions. For security monitors, an interface with minimal latency is crucial: Such monitors are typically most useful, if they can guarantee to prevent the core from executing malicious code ASAP with no or limited impact to the outside world. Depending on the individual core (its pipeline, its signal taps, its write latency...) and the intended security guarantees, even with RT-LIFE only 0 (combinatorial) - 2 clock cycles remain for the monitor's security evaluation. Of course, evaluation time can be extended via stall signals. 

How latency is reduced: 
- Plain signals, neither compression nor delta encoding
- Tapping of signals in early pipeline stages
- Two types of precise stalls
	- CF stall: Stalls the execute stage and later stages.
	- Stall on Store: Combinatorially stalls memory writes as late as possible.

By now RT-LIFE focuses on 32 bit IoT-class RISC-V cores, which are single-core, single-issue, and in-order type.
Supported cores are:
- [Flute](https://github.com/bluespec/Flute)
- [Orca (forked)](https://github.com/kingcard1131/orca), <s>[original source](https://github.com/VectorBlox/orca)</s>
- [Piccolo](https://github.com/bluespec/Piccolo)
- [PicoRV32](https://github.com/cliffordwolf/picorv32)
- [Taiga](https://gitlab.com/sfu-rcl/Taiga)
- [VexRiscv](https://github.com/SpinalHDL/VexRiscv)

**Architecture**  
| Tapasco Host | <--- 1 ---> | [SecurityMonitor <--- 2 ---> RT-LIFE enabled RISC-V Core] |
|--|--|--|

The [TaPaSCo](https://git.esa.informatik.tu-darmstadt.de/tapasco/tapasco) framework is used for rapid prototyping. A dummy security monitor is packed together with the chosen RISC-V core inside a TaPaSCo-PE. In the following, we differentiate between PE-external (1) and PE-internal (2) signals:



| Interface | Signals|
|--|--|
| --- 1 ---> | AXI for writing (DExIE and RISC-V core) memory contents |
| | AXI for controlling the TaPaSCo-PE  |
|<--- 1 --- | Reset, Interrupt via CTRL AXI|
|--- 2 ---> | Reset, AXI for forwarding (checked) RISC-V core memory contents|
|| Stalls: CF-Stall, Stall on Store, Continue Stall|
|<--- 2 --- | Control Flow: PC, Instruction, Next PC|
| |Memory Store: Valid, PC, Address, Size, Data, Stall Active |
| |Register Writeback: PC, Target Register (0=invalid), Data |



**Requirements:**   
Xilinx Vivado 2018.3 or newer  
TaPaSCo RISC-V https://github.com/esa-tu-darmstadt/tapasco-riscv  
TaPaSCo https://git.esa.informatik.tu-darmstadt.de/tapasco/tapasco  
Bluespec https://github.com/B-Lang-org/bsc  
RISC-V Toolchain https://github.com/riscv/riscv-gnu-toolchain  

**Project structure:**   
- Folder `core` contains RT-LIFE core wrappers and simplified Security Monitors.  
- Folder `ip` contains the RT-LIFE extended cores together with their individual diffs. These originate from the TaPaSCo RISC-V repository.  
- Folder `testPrograms` contains a number of small application examples.  
- The project's `root directory` contains the project packaging scripts.  
- After generation, folder `dexie_ip` contains the final resulting TaPaSCo-PE.  

**First steps:**  
- Run `make binaries`    
- Use `riscv32-unknown-elf-objdump -d`, to analyse the ELF object file of a sample code snippet    
- Sample ELF object files can be found in: `testPrograms/en_*/elf`    
- Have a closer look on control flow, memory write and register write instructions  
- Have a closer look on `core/DexieReg_Nested`, `core/DexieMem_Nested` and `core/DexieCF_Nested`
- Understand and edit the dummy Security Monitors in the middle of these files.
- `make taiga_pe` (or any other core)
- Using `tapasco-import`, `tapasco-compose` and `tapasco load-bitstream`, the design can be deloyed on any TaPaSCo-compatible FPGA board.

- Be aware, that the first 16 words (64 Bytes) are configuration input for the dummy Security Monitors (currently not interpreted). Only subsequent data is written into the instruction memory of the attached RISC-V core.

**Individual Licensing of the included cores:**  
The included files in /ip/... are foreign IP from foreign projects (RISC-V cores), which employ their own licensing. These licenses also apply to the patches, we implemented for each individual core.
- [Flute](https://github.com/bluespec/Flute/blob/master/LICENSE) - Apache 2.0
- [Orca](https://github.com/kingcard1131/orca/blob/master/LICENSE.txt) - Proprietary
- [Piccolo](https://github.com/bluespec/Piccolo/blob/master/LICENSE) - Apache 2.0
- [PicoRV32](https://en.wikipedia.org/wiki/ISC_license) - ISC License
- [Taiga](https://gitlab.com/sfu-rcl/Taiga/-/blob/master/LICENSE) - Apach 2.0
- [VexRiscv](https://github.com/SpinalHDL/VexRiscv/blob/master/LICENSE) - MIT License


**License RT-LIFE:**  
The following license applies to RT-LIFE's remaining components, which are core wrappers, packaging scripts, Makefiles, code examples, compiler scripts, AXI and Demo Security Monitors.

Copyright (c) 2019-2020 Embedded Systems and Applications, TU Darmstadt.
This file is part of RT-LIFE (see https://github.com/esa-tu-darmstadt/RT-LIFE).

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the "Software"),
to deal in the Software without restriction, including without limitation
the rights to use, copy, modify, merge, publish, distribute, sublicense,
and/or sell copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.  

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
