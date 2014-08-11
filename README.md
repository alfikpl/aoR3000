### aoR3000

The aoR3000 is a MIPS R3000A compatible core capable of booting the Linux kernel version 3.16 in about 3 seconds and with a rating of 48.74 BogoMIPS. It features a compatible MMU, but no FPU.

### Current status
- 11 August 2014 - initial version 1.0.

### Features

The aoR3000 soft processor core has the following features:
- 5 stage pipeline (single-issue, in-order, forwarding, hazard detection);
- implements all MIPS I instructions (this includes the privileged coprocessor 0 instructions);
- the MMU in coprocessor 0 is compatible with the R3000A but has more micro-TLB entries: 64 TLB entries, 8 micro-TLB entries for data, 4 micro-TLB entries for instructions;
- 2 kB instruction cache (direct map);
- 2 kB data cache (direct map);
- all exceptions are implemented (bus error exceptions are not issued, exceptions in branch delays are supported);
- the core has one combined instruction/data Altera Avalon MM master interface (with burst and pipelined instruction reads, pipelined data reads);
- multiplication implemented in hardware using vendor-independent modules (Altera Quartus II infers embedded multipliers);
- division implemented in hardware using long division (33 clock cycles required);
- the core uses 7700 LE and runs at 50 MHz on a Altera Cyclone IV EP4CE115F29C7;
- the core is modeled in a vendor-independent subset of Verilog consistent with the requirements of Verilator (fully synchronous design, no vendor specific module instances);
- a Verilator generated executable C++ model is available;
- the core was tested and compared with the vmips software R3000 emulator;
- the core is chosen to be little-endian and can not be changed by software;

### Resource usage
The following table shows the result of synthesis using a balanced area/speed optimization: 

    +---------------------------------------------------------------------------------+
    ; Flow Summary                                                                    ;
    +------------------------------------+--------------------------------------------+
    ; Flow Status                        ; Successful - Mon Aug 11 21:46:48 2014      ;
    ; Quartus II 64-Bit Version          ; 14.0.0 Build 200 06/17/2014 SJ Web Edition ;
    ; Revision Name                      ; aoR3000                                    ;
    ; Top-level Entity Name              ; aoR3000                                    ;
    ; Family                             ; Cyclone IV E                               ;
    ; Device                             ; EP4CE115F29C7                              ;
    ; Timing Models                      ; Final                                      ;
    ; Total logic elements               ; 7,660 / 114,480 ( 7 % )                    ;
    ;     Total combinational functions  ; 7,174 / 114,480 ( 6 % )                    ;
    ;     Dedicated logic registers      ; 2,633 / 114,480 ( 2 % )                    ;
    ; Total registers                    ; 2633                                       ;
    ; Total pins                         ; 115 / 529 ( 22 % )                         ;
    ; Total virtual pins                 ; 0                                          ;
    ; Total memory bits                  ; 61,616 / 3,981,312 ( 2 % )                 ;
    ; Embedded Multiplier 9-bit elements ; 8 / 532 ( 2 % )                            ;
    ; Total PLLs                         ; 0 / 4 ( 0 % )                              ;
    +------------------------------------+--------------------------------------------+

    +-------------------------------------------------+
    ; Slow 1200mV 85C Model Fmax Summary              ;
    +-----------+-----------------+------------+------+
    ; Fmax      ; Restricted Fmax ; Clock Name ; Note ;
    +-----------+-----------------+------------+------+
    ; 52.58 MHz ; 52.58 MHz       ; clk        ;      ;
    +-----------+-----------------+------------+------+

### Implemented instructions

All the MIPS I instructions are implemented:
- ADD, ADDI, ADDIU, ADDU, AND, ANDI, NOR, OR, ORI, SLL, SLLV, SLT, SLTI, SLTIU, SLTU, SRA, SRAV, SRL, SRLV, SUB, SUBU, XOR, XORI, LUI;
- DIV, DIVU, MULT, MULTU, MTHI, MTLO, MFHI, MFLO;
- BREAK, SYSCALL;
- CFCz, CTCz, LWCz, SWCz;
- MFCz, MTCz;
- COPz, RFE, TLBP, TLBR, TLBWI, TLBWR;
- BEQ, BGEZ, BGEZAL, BGTZ, BLEZ, BLTZ, BLTZAL, BNE, J, JAL, JALR, JR;
- LB, LBU, LH, LHU, LW, LWL, LWR, SB, SH, SW, SWL, SWR;

### Running the tests
The aoR3000 core can be converted by Verilator to a C++ executable model. This executable model was compared with the vmips software R3000 emulator (http://vmips.sourceforge.net/) in simulation.
All instructions were simulated with random register and memory contents. After every instruction the register contents are compared.

To run the test the following steps have to be taken:
- compile the vmips emulator in sim/vmips/ by 'make tester';
- compile the Verilator aoR3000 model in sim/aoR3000/ by 'make tester';
- choose the test to be run in sim/tester/main_tester.cpp by setting the pointer 'tst_t *tst_current =';
- compile the tester in sim/tester/ by 'make tester';
- run the tester in sim/tester/ by './main_tester';

### GNU toolchain
To run programs on the aoR3000 a standard GNU toolchain in required. I used the following options during compilation of the toolchain:
- for GNU binutils: './../binutils-2.24.51/configure --prefix=<path to install directory> --target=mipsel-unknown-linux-gnu';
- for GCC: './../gcc-4.9.1/configure --prefix=<path to install directory> --target=mipsel-unknown-linux-gnu --enable-languages=c --disable-threads --disable-shared --disable-libssp --disable-libquadmath --disable-libgomp --disable-libatomic';

The target 'mipsel-unknown-elf' can also be used, but more changes are required to the Linux kernel in that case.

### Building a minimal Linux kernel
In the directory linux/ there is a minimal set of files required to add the aoR3000 SoC platform to the kernel sources version 3.16. To compile the kernel with these changes just copy/overwrite the files in
a Linux source tree. After that 'make ARCH=mips CROSS_COMPILE=mipsel-unknown-linux-gnu- vmlinux.bin' to build the kernel.

### Booting the Linux kernel in simulation on a Verilator executable model of the aoR3000
The booting of the Linux kernel is performed in a similar method like the tests described above. The Verilator executable model of the aoR3000 is run together with the vmips R3000 software simulator to verify
the correctness of the aoR3000. Every write to memory is checked to confirm that both of the models write the same data at the same time and in the same order.

To boot the Linux kernel in simulation the following steps have to be taken:
- compile the vmips emulator in sim/vmips/ by 'make linux';
- compile the Verilator aoR3000 model in sim/aoR3000/ by 'make linux';
- compile the tester in sim/tester/ by 'make linux';
- compile the Linux kernel to get the 'vmlinux.bin' file in the arch/mips/boot/ directory;
- run the tester in sim/tester/ by './main_linux <path to vmlinux.bin file>';

The booting of the Linux kernel takes about 12 minutes on a modern PC (more than 46 milion instructions have to be executed/simulated).

The tester simulates the following hardware devices:
- an eary Linux console that outputs the data written to that console to the file 'tester/early_console.txt';
- a simple hardware time interrupt device that signals IRQ 0 every 10 miliseconds;
- an Altera JTAG UART connected to a pseudo terminal (Unix PTY). The terminal name is printed just after executing the tester (for example: 'slave pty: /dev/pts/9');

To connect to this terminal one can use the following command: 'picocom --nolock /dev/pts/9'. Data can be read and written to this terminal.

### Booting the Linux kernel on a FPGA with a synthesized aoR3000
Together with the aoR3000 core there is also a example SoC in the syn/soc/ directory. It consists of:
- the aoR3000 core;
- an Altera JTAG UART;
- an Altera SDRAM controller;
- a simple time interrupt device;
- onchip memory for the boot code of the aoR3000;
- a Altera JTAG to Avalon Master Bridge to upload the Linux kernel;

The SoC is designed for the Terasic DE2-115 board.

To compile the SoC the following steps have to be taken:
- open the Altera Quartus II project in syn/soc/;
- open the Qsys tool and generate the HDL;
- compile the project;
- program the FPGA with the compiled project;
- in the Altera Nios2 Command Shell enter the directory arch/mips/boot/ of the Linux kernel and run 'system-console -cli';
- load the Linux kernel binary using the system-console by entering the following TCL commands at the system-console prompt:

    set srv [claim_service "master" [lindex [get_service_paths "master"] 0] ""];
    master_write_from_file $srv vmlinux.bin 0;
    close_service master $srv;

- open the nios2-terminal in a Altera Nios2 Command Shell;
- verify that the following text is displayed in the terminal: 'Press any key to boot kernel...';
- press any key to boot the kernel;

The kernel is booted with the following output displayed on the nios2-terminal:

---
    Press any key to boot kernel...
    Booting kernel...
    console [ttyJ0] enabled
    bootconsole [early0] disabled
    Freeing unused kernel memory: 176K (80204000 - 80230000)
    # 
---

A shell is run as init in the early root-fs. The early root-fs is the compiled klibc project.

After running the command dmesg the following output is displayed:

---
    Linux version 3.16.0 (alek@duke) (gcc version 4.9.1 (GCC) ) #4 Mon Aug 11 23:49:07 CEST 2014
    bootconsole [early0] enabled
    CPU0 revision is: 00000230 (R3000A)
    Determined physical RAM map:
     memory: 08000000 @ 00000000 (usable)
    Initrd not found or empty - disabling initrd
    Zone ranges:
      Normal   [mem 0x00000000-0x07ffffff]
    Movable zone start for each node
    Early memory node ranges
      node   0: [mem 0x00000000-0x07ffffff]
    On node 0 totalpages: 32768
    free_area_init_node: node 0, pgdat 80203600, node_mem_map 81000000
      Normal zone: 256 pages used for memmap
      Normal zone: 0 pages reserved
      Normal zone: 32768 pages, LIFO batch:7
    Primary instruction cache 2kB, linesize 4 bytes.
    Primary data cache 2kB, linesize 4 bytes.
    pcpu-alloc: s0 r0 d32768 u32768 alloc=1*32768
    pcpu-alloc: [0] 0 
    Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 32512
    Kernel command line:  console=ttyJ0,115200
    PID hash table entries: 512 (order: -1, 2048 bytes)
    Dentry cache hash table entries: 16384 (order: 4, 65536 bytes)
    Inode-cache hash table entries: 8192 (order: 3, 32768 bytes)
    Memory: 127492K/131072K available (1741K kernel code, 101K rwdata, 212K rodata, 176K init, 171K bss, 3580K reserved)
    SLUB: HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
    NR_IRQS:128
    Console: colour dummy device 80x25
    Calibrating delay loop... 48.74 BogoMIPS (lpj=243712)
    pid_max: default: 32768 minimum: 301
    Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
    Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
    futex hash table entries: 256 (order: -1, 3072 bytes)
    io scheduler noop registered
    io scheduler deadline registered
    io scheduler cfq registered (default)
    ttyJ0 at MMIO 0x1ffffff0 (irq = 3, base_baud = 0) is a Altera JTAG UART
    console [ttyJ0] enabled
    bootconsole [early0] disabled
    Freeing unused kernel memory: 176K (80204000 - 80230000)
---

### License
Most of the files in this project are under the BSD license. All the Verilog code for the aoR3000 core is under the BSD license.

A few files are under the GPL license. These files are:
- all files in the sim/vmips/ directory;
- all files in the linux/ directory;

See the LICENSE file for details.
