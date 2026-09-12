# Hardware Implementation of 32-Point FFT using Vitis HLS on PYNQ-Z1

## Overview

This project implements a **32-point Fast Fourier Transform (FFT)** as a
hardware accelerator for the **PYNQ-Z1** FPGA board using **Xilinx Vitis
HLS**.

The FFT algorithm was first developed in **C++**, synthesized into
hardware using Vitis HLS, exported as a custom IP, and integrated with a
**Zynq Processing System** in Vivado. The implementation was verified
through C simulation, HLS co-simulation, and hardware testing using a
Jupyter Notebook on the PYNQ-Z1.

The design follows a **radix-2 Decimation-in-Time (DIT)** FFT structure
with five butterfly stages.

## Project Information

-   **Project:** Hardware Implementation of 32-point FFT
-   **Target Board:** PYNQ-Z1
-   **Algorithm:** 32-point Radix-2 DIT FFT
-   **HLS Language:** C++
-   **HLS Tool:** Xilinx Vitis HLS 2021.1
-   **Hardware Integration:** Vivado 2021.1
-   **Processor:** Zynq Processing System
-   **Data Interface:** AXI4 Master
-   **Control Interface:** AXI4-Lite
-   **Fixed-point format:** `ap_fixed<16,4>`

## FFT Architecture

The 32-point FFT consists of five radix-2 butterfly stages.

``` text
Input Buffer
     |
     v
Bit Reversal
     |
     v
FFT Stage 1
16 groups of 2 points
     |
     v
FFT Stage 2
8 groups of 4 points
     |
     v
FFT Stage 3
4 groups of 8 points
     |
     v
FFT Stage 4
2 groups of 16 points
     |
     v
FFT Stage 5
1 group of 32 points
     |
     v
Output Buffer
     |
     v
Data Output
```

The input data is first reordered using bit reversal. The FFT
computation is then performed stage by stage using butterfly operations.
Intermediate results are stored in static arrays before being passed to
the next stage.

## Hardware Architecture

The synthesized FFT is exported from Vitis HLS as a custom IP core and
integrated into a Zynq-based Vivado design.

The main hardware components are:

-   **Zynq Processing System (PS)**
    -   HP ports configured for high-throughput data transfers
    -   AXI GP/control connectivity
-   **Custom FFT IP**
    -   Generated using Vitis HLS
    -   AXI4 Master ports for input/output data
    -   AXI4-Lite interface for control
-   **AXI Interconnect**
    -   Connects the Zynq PS with the FFT IP
-   **Clock and Reset**
    -   Used to synchronize the FFT accelerator with the processing
        system

The hardware block diagram is included in `Block_diagram.png`.

## Verification

The implementation was verified at multiple stages.

### C Simulation

C simulation was used to verify the functional correctness of the C++
FFT implementation.

-   Output was compared against expected FFT values.
-   A tolerance of **0.1** was used.
-   The self-checking testbench reported **zero errors**.

### HLS Co-Simulation

HLS co-simulation was performed to verify the functionality of the
synthesized RTL implementation.

-   Synthesized hardware behavior was compared with the software model.
-   The output was verified with minimal error.

### Hardware Testing

The generated bitstream was deployed to the PYNQ-Z1 and tested using a
Jupyter Notebook.

The hardware FFT output was compared with MATLAB-generated FFT values
for the same input.

-   Comparison tolerance: **0.1**
-   Errors detected: **0**

## HLS Optimizations

Four implementations were evaluated to study the effect of HLS
optimization directives.

### Latency

  Implementation      Latency (cycles)   Latency (µs)
  ----------------- ------------------ --------------
  Unoptimized                     1295         12.950
  Loops Pipelined                  374          3.740
  Inlining                         302          3.020
  Dataflow                         294          2.940

### Interval

  Implementation      Interval (cycles)
  ----------------- -------------------
  Unoptimized                      1296
  Loops Pipelined                   375
  Inlining                          303
  Dataflow                           75

### Resource Utilization

  Implementation      LUTs    FFs   DSP   BRAM (18K)
  ----------------- ------ ------ ----- ------------
  Unoptimized         3665   2289     8            8
  Loops Pipelined     4400   2890     8            8
  Inlining            5439   3392    16            8
  Dataflow            4923   3951    16           10

### Optimization Trade-off

The unoptimized design provides the lowest LUT/resource usage but has
the highest latency and interval.

Applying loop pipelining, function inlining, and dataflow significantly
improves performance at the cost of additional hardware resources.

The **Dataflow implementation** provides the lowest reported latency and
interval, making it the best-performing configuration among the
evaluated implementations.

## Fixed-Point Implementation

The design uses:

``` text
ap_fixed<16,4>
```

The fixed-point representation was selected after profiling the dynamic
range of the FFT data.

Using fixed-point arithmetic reduces hardware cost compared with a
floating-point implementation and helps reduce LUT/DSP usage while
improving timing.

Twiddle factors are common sub-expressions in the FFT computation. They
are therefore pre-calculated and stored in a header file rather than
being repeatedly calculated during execution.

## Repository Contents

The project archive contains the following supporting files:

``` text
.
├── AXI-FFT.ipynb
├── Block_diagram.png
├── FFT.drawio
├── FFT.png
├── Result.png
├── error.png
├── fin.bit
├── fin.hwh
├── _csim.log
├── FFT_cosim.rpt
├── unoptimized.rpt
├── Loops_pipelined.rpt
├── inline_added.rpt
└── dataflow_added.rpt
```

### Important Files

  File                  Description
  --------------------- -----------------------------------------------------
  `AXI-FFT.ipynb`       Jupyter Notebook used for hardware testing
  `fin.bit`             Generated FPGA bitstream
  `fin.hwh`             Hardware handoff file
  `Block_diagram.png`   Vivado hardware architecture
  `FFT.png`             FFT architecture/diagram
  `FFT.drawio`          Editable FFT architecture diagram
  `Result.png`          Hardware FFT output/result
  `error.png`           Tool error encountered during development
  `*_added.rpt`         HLS synthesis reports for optimized implementations
  `_csim.log`           C simulation log
  `FFT_cosim.rpt`       HLS co-simulation report

## Implementation Flow

``` text
C++ FFT Algorithm
        |
        v
   C Simulation
        |
        v
    HLS Synthesis
        |
        v
 HLS Co-Simulation
        |
        v
    Export IP
        |
        v
 Vivado Integration
        |
        v
 Generate Bitstream
        |
        v
     PYNQ-Z1
        |
        v
 Jupyter Notebook
        |
        v
 Hardware FFT Output
```

## Tools and Technologies

-   C++
-   Xilinx Vitis HLS
-   Xilinx Vivado
-   High-Level Synthesis (HLS)
-   FPGA hardware acceleration
-   PYNQ-Z1
-   Zynq Processing System
-   AXI4
-   AXI4-Lite
-   Fixed-point arithmetic
-   Radix-2 FFT
-   Jupyter Notebook

## Key Results

The project demonstrates that HLS optimization directives can
substantially improve FFT accelerator performance.

Compared with the unoptimized implementation:

-   Latency was reduced from **1295 cycles to 294 cycles**.
-   Reported interval was reduced from **1296 cycles to 75 cycles**.
-   The optimized implementation requires additional LUT, FF, DSP, and
    BRAM resources.

This demonstrates the fundamental **performance-versus-resource
trade-off** involved in mapping signal-processing algorithms onto FPGA
architectures.

## Known Tool Issue

During development with **Vitis HLS/Vivado 2021.1**, an issue was
encountered while generating the IP. Based on troubleshooting through
the AMD/Xilinx forum, manually changing the system date to **2015 or
earlier** was reported to resolve the issue.

Additionally, the same C++ code and block design produced problems when
reading the hardware output using the 2021.1-generated files. Testing
the same design on a different machine using **Vitis HLS/Vivado 2023.2**
successfully generated the FFT output.

## References

1.  NPTEL lectures by Dr. Nitin Chandrachoodan:
    https://archive.nptel.ac.in/courses/108/106/108106149/

2.  Reference videos and documents provided during the course.

## Course

This project was completed as part of:

**EE5332 -- Mapping Signal Processing Algorithms to DSP Architectures**

Instructor: **Dr. Nitin Chandrachoodan**

------------------------------------------------------------------------

**Author:** Ranjith V. R.
