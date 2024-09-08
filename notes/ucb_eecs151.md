Some notes on setup the environment are [here](https://github.com/eecsmap/logs/blob/main/eecs/ucb_eecs151.md)

FPGA labs cover common IO modules. They will be used in final project.

FPGA project leads to a RISC-V CPU (RV32I) implementation and tests.

###### Interface of FIFO

The Ready-Valid protocol can indeed be somewhat loose in certain scenarios because it allows both parties to have a degree of control over the data transfer, which might make it difficult to clearly specify who should initiate the communication.

For devices like FIFOs, designers often prefer to have a clear initiator in the communication process, which is typically the data producer or consumer. The FIFO user can query the FIFO's status using empty and full signals, which makes it more intuitive for the user to initiate the communication. The FIFO can provide its status, and the user (such as the data producer or consumer) can decide when to start the data transfer based on these status signals.

So for the writing side, we can have:
* input: data_in
* input: write_enable
* output: is_full

And for the reading side, we can have:
* input: read_enable
* output: data_out
* output: is_empty

# FPGA Project

(According to the [project spec](https://github.com/eecsmap/fpga_project_skeleton_fa22/blob/master/spec/EECS151_FPGA_Project_Fa22.pdf))

It is designed to familiarize students with the methods and tools of digital design.

We will implement a 3-stage pipelined RISC-V CPU.

* language: Verilog
* platform: Xilinx PYNQ-Z1 board with a Zynq 7000-seires FPGA.

Gain Experience:

* design with RTL descriptions
* resolve hazards
* build interfaces
* system-level optimization

> "EECS151/251A project could be the largest project you have faced so far at Berkeley."

I guess it is compared to cs61a/b/c?

Steps:

* Draw a schematic of your processor's datapath and pipeline stages. Write design documents (block diagram + write-up).
* Include IO-circuit Verilog modules you have implemented in the labs.
* Implement a fully functional RISC-V processor core in Verilog. It should be able to run the `mmult` demo.
* Implement a branch predictor in Verilog.
* Final processor optimization and checkoff.
* Final reports.

Tips

* Document your project as you go.
* Comment your Verilog and keep your diagrams up to date.

Prepare code repository.

Instead of

```
git clone https://github.com/EECS150/fpga_project_skeleton_fa22
cd fpga_project_skeleton_fa22
git submodule init
git submodule update
```

We should use
```
git clone --recusive-submodules https://github.com/EECS150/fpga_project_skeleton_fa22
```

Anyway, I have done the setup to create my [private repo](https://github.com/eecsmap/fpga_project_skeleton_fa22)

IO modules

* synchronizer
* debouncer
* edge detector
* fifo
* uart transmitter and receiver

Project structure:

* hardware/src
  * io_circuits
  * z1top.v
  * riscv_core/opcode.vh
  * riscv_core/cpu.v
* hardware/sim
  * cpu_tb.v
  * asm_tb.v
  * isa_tb.v
  * c_tests_tb.v
  * echo_tb.v
  * uart_parse_tb.v
  * bios_testbench.v
* software
  * bios
  * echo
  * asm
  * c_tests
  * riscv-isa-tests
  * mmult
 
Project is managed using `make`.

Instructions covered:
* R (funct7  |rs2|rs1|funct3|rd    |opcode5|11): add, sub, sll, slt, sltu, xor, srl, sra, or, and
* I/L (imm11-0     |rs1|funct3|rd    |opcode5|11): addi, slti, sltiu, xori, ori, andi, slli, srli, srai, *JALR*, lb, lh, lw, lbu, lhu
* S (imm11-5|rs2|base|funct3|imm4-0|opcode5|11): sb, sh, sw
* B (imm12,imm10-5|rs2|rs1|funct3|imm4-1,imm11|opcode5|11): beq, bne(not eq), blt, bge(not lt), bltu, bgeu
* U (imm31-12|rd|opcode5|11): auipc, lui
* J (imm20,imm10-1,imm11,imm19-12|rd|opcode5|11): jal

(J reuses the fields from B and U).
(rs1 is used as base in both load and store).

Following notes are based on the [2022 fall FPGA project](https://github.com/eecsmap/fpga_project_skeleton_fa22).

Pay attention to how the project is organized.

Start with [this doc](https://github.com/eecsmap/fpga_project_skeleton_fa22/blob/master/hardware/README.md).

I am using a `ubuntu 24.04.1 LTS`, `iverilog` could be installed with `sudo apt install iverilog`.

```bash
engineer@ubuntu:~/github/ucb_eecs151_2022_fall_fpga_project$ cd hardware
engineer@ubuntu:~/github/ucb_eecs151_2022_fall_fpga_project/hardware$ make -B sim/cpu_tb.fst
cd sim && iverilog -Ttyp\
 -D IVERILOG=1\
 -g2012\
 -gassertions\
 -Wall\
 -Wno-timescale\
 -D ABS_TOP=/home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware\
 -I /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/riscv_core\
 -I /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/sim\
 -o cpu_tb.tbi\
 cpu_tb.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/EECS151.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/synchronizer.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/uart_transmitter.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/uart_receiver.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/button_parser.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/fifo.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/edge_detector.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/uart.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/io_circuits/debouncer.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/riscv_core/cpu.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/riscv_core/reg_file.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/riscv_core/branch_prediction/branch_predictor.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/riscv_core/branch_prediction/bp_cache.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/riscv_core/branch_prediction/sat_updn.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/z1top.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/clocks.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/memories/imem.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/memories/dmem.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/src/memories/bios_mem.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/sim_models/glbl.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/sim_models/BUFG.v\
 /home/engineer/github/ucb_eecs151_2022_fall_fpga_project/hardware/sim_models/PLLE2_ADV.v
cd sim && vvp cpu_tb.tbi -fst |& tee cpu_tb.log
FST info: dumpfile cpu_tb.fst opened for output.
[Failed] Timeout at [         1] test                       R-Type ADD, expected_result = 00000064, got = 00000000
cpu_tb.v:106: $finish called at 450000 (1ps)
```


This [Makefile](https://github.com/eecsmap/fpga_project_skeleton_fa22/blob/8811dbcd32483b4203b0aa15698edf27e975af1f/hardware/Makefile#L33) is also a good example.

