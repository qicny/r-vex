<h1>Quickstart Guide for XUP V2P FPGA Board</h1>

This is a quickstart guide to easily deploy ρ-VEX, mainly focused on the utilization
with a Xilinx University Program Virtex-II Pro FPGA board by Digilent. If
you want to use this guide together with another FPGA platform, you should first add
the definitions of your board to the workflow as described later.

<h2>Table of Contents</h2>


## Requirements ##
  * A [Xilinx University Program Virtex-II Pro board](http://www.digilentinc.com/Products/Detail.cfm?av1=Products&Nav2=Programmable&Prod=XUPV2P)
  * PC running Linux or Windows<sup>1</sup>
  * Xilinx ISE Suite (tested with 8.1.03i, should work with later versions too)

<sup>1</sup> When you want to make use of the Makefile method described below on a Windows machine, a Cygwin installation should be present with GNU Make. Xilinx EDK automatically installs a version of Cygwin. However, some GNU tools like `cat` are not included. This results in an error while bundling the log files after synthesis. This can be safely ignored.

## Quickstart - Deploying ρ-VEX on an FPGA ##
  1. Acquire the latest snapshot from the Downloads section at the ρ-VEX website, or checkout the latest code from the Subversion repository.
  1. Inside the r-VEX/src/ directory, synthesize ρ-VEX by entering the following command:
```
make v2p
```
  1. After the synthesis process has completed, the generated bit-file can be uploaded to the FPGA board by entering:
```
make fpga
```
  1. Connect a serial cable to the RS-232 interface on the XUP V2P board. Connect the other side of the cable to a PC, and start a terminal application, like Minicom (Linux), Putty or Hyperterminal (Windows). Connect using the following settings: _115200 bps transfer rate, 8 data bits, no parity_
  1. Press button SW2 on the XUP V2P board, which acts as the reset button. In the terminal application, you will see the contents of the first 16 data memory addresses, as well as a cycle counter.

By default, an application to calculate the 45th Fibonacci number is loaded and pre-synthesized. The VEX assembly source file of the default application is [fibonacci.s](https://github.com/tvanas/r-vex/blob/master/demos/fibonacci.s), which can be found in the [demos](https://github.com/tvanas/r-vex/tree/master/demos) directory. The output of ρ-VEX transmitted over the UART is the following:

```
r-VEX
-----
Cycles: 0x00000231

Data memory dump

addr | contents
-----+-----------
0x00 | 0x43A53F82
0x01 | 0x00000000
0x02 | 0x00000000
0x03 | 0x00000000
0x04 | 0x00000000
0x05 | 0x00000000
0x06 | 0x00000000
0x07 | 0x00000000
0x08 | 0x00000000
0x09 | 0x00000000
0x0A | 0x00000000
0x0B | 0x00000000
0x0C | 0x00000000
0x0D | 0x00000000
0x0E | 0x00000000
0x0F | 0x00000000
```

## Assembling and Running Other Code ##

The instruction memory ROM can be found in the file [i\_mem.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/i_mem.vhd). A new instruction ROM file can be generated by the ρ-ASM tool. This tool requires a UNIX operating system with the GNU C libraries.

  1. To compile ρ-ASM, go to the [r-ASM/src/](https://github.com/tvanas/r-vex/tree/master/r-ASM/src) directory, and enter the command
```
make
```
  1. To assemble a VEX assembly file, run ρ-ASM by entering the following command:
```
./rasm <source.s>
```
  1. By default, the resulting instruction ROM is written to `i_mem.vhd` in the current directory. Copy or move this file to the ρ-VEX source directory. When using the standard repository structure, this can be accomplished by entering
```
mv i_mem.vhd ../../r-VEX/src/
```
  1. Now, repeat the steps from the Quickstart described earlier.

To see more options of ρ-ASM (like enabling debug output) run the application without any arguments. Some demo applications with their corresponding outputs can be found in the [demos](https://github.com/tvanas/r-vex/tree/master/demos) directory.

## Using ρ-OPS ##
This guide on adding ρ-OPS to an ρ-VEX implementation is based on the example of adding the **FIB3** and **FIB4** operations, as used in our Fibonacci's Sequence benchmark application.

### Adapting ρ-VEX ###
  1. Determine what Functional Units will execute the operation. An unused ρ-OPS opcode can be optained from OperationsSemantics.
  1. In   [r-vex\_pkg.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/r-vex_pkg.vhd), constant definitions should be added for the new operations:
```
constant ALU_FIB4    : std_logic_vector(6 downto 0) := "1000010";
constant ALU_FIB3    : std_logic_vector(6 downto 0) := "1101100";
```
  1. In the determined Functional Unit, support for the new operations should be added in the corresponding VHDL file. In our benchmark, lines 9 - 12 are added in [alu.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/alu.vhd) from the code snippet below.
```
alu_control : process(clk, reset)
begin
	if (reset = '1') then
		out_valid <= '0';
		result_s <= (others => '0');
		cout_s <= '0';
	elsif (clk = '1' and clk'event) then
		...
		elsif std_match(aluop, ALU_FIB4) then
			result_s <= f_FIB4 (src1, src2);
		elsif std_match(aluop, ALU_FIB3) then
			result_s <= f_FIB3 (src1, src2);
		...
	end if;
end process alu_control;
```
  1. Functions and function prototypes should be added to the corresponding file. In our example, the code below was added to [alu\_operations.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/alu_operations.vhd).
```
function f_FIB4   ( s1, s2 : std_logic_vector(31 downto 0))
                    return std_logic_vector;

function f_FIB3   ( s1, s2 : std_logic_vector(31 downto 0))
                    return std_logic_vector;

...

function f_FIB4   ( s1, s2 : std_logic_vector(31 downto 0))
                    return std_logic_vector is
begin
	return (s1 + s2 + s2 + s1 + s2 + s2 + s1 + s2);
end function f_FIB4;

function f_FIB3   ( s1, s2 : std_logic_vector(31 downto 0))
                    return std_logic_vector is
begin
	return (s1 + s2 + s2 + s1 + s2);
end function f_FIB3;
```


### Adapting ρ-ASM ###
To adapt ρ-ASM to support the new ρ-OPS, two small additions should be made in [syllable.h](https://github.com/tvanas/r-vex/blob/master/r-ASM/src/syllable.h). A new opcode  define should be made, as well as an addition of the new opcode's mnemonic to the `operation_table` lookup table (lines 11 - 12).

```
#define FIB3   108
#define FIB4    66

...

static struct operation_t {
	const char *operation;
	int opcode;
} operation_table[] = {
	...
	{ "fib3",  FIB3   },
	{ "fib4",  FIB4   },
	...
};
```

## Running Modelsim Simulations ##

To run Modelsim simulations within the workflow, you should have installed a version of Mentor Graphics Modelsim (we simulated with the SE 6.3d edition). To run the simulations using the system wrapper testbench [r-VEX/testbenches/tb\_system.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/testbenches/tb_system.vhd) (which simulates the execution of the current instruction memory in [i\_mem.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/i_mem.vhd)), enter the following command in [r-VEX/src/](https://github.com/tvanas/r-vex/tree/master/r-VEX/src):
```
make modelsim
```

## Adding Support For Other FPGA Boards ##

To add support for other FPGA boards with Xilinx a FPGA in this workflow, the file [r-VEX/src/Makefile](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/Makefile) should be edited. The following changes should be applied:

  1. Choose a mnemonic for your new target. For example, we used `v2p` for the XUP V2P board containing a Virtex-II Pro FPGA. This mnemonic will be referred to as `<mnemonic>`. Check the Xilinx identifier for this FPGA. For example, the identifier for the Virtex-II Pro chip we used is _xc2vp30-ff896-7_. This identifier will be referred to as `<xilinx_identifier>`.
  1. Add the Xilinx FPGA identifier as a new variable. Use the following template:
```
XIL_PART_r-vex_<mnemonic> = <xilinx_identifier>
```
  1. Add a 'help' text to the `default` _Make_ target.
  1. Create a new _Make_ target according to this template:
```
<mnemonic>: r-vex_<mnemonic>.deps r-vex_<mnemonic>.bit log_<mnemonic>.txt
```
  1. Create a new _Make_ target `r-vex_<mnemonic>.ut` and let it write your preferred options to the `.ut` file.<sup>2</sup>
  1. Create a new _Make_ target `r-vex_<mnemonic>.xst` and let it write your preferred options to the XST configuration file.<sup>2</sup>
  1. Create a new _Make_ target `r-vex_<mnemonic>.ucf` and let it write your UCF user constraints to the `.ucf` file.<sup>2</sup>
  1. Create a new _Make_ target `r-vex_<mnemonic>.cmd` and let it write the commands for Xilinx IMPACT to program your FPGA.<sup>2</sup>
  1. Set the constant `ACTIVE_LOW` in [r-VEX/src/r-vex\_pkg.vhd](https://github.com/tvanas/r-vex/blob/master/r-VEX/src/r-vex_pkg.vhd#L41) (line 41) to `1` in case the board uses 'active low' logic (like the XUP V2P board) or to `0` in case the board uses 'active high' logic. **If this constant is defined wrong, your board won't do anything.**


<sup>2</sup>As a reference, you could use the _Make_ target with the mnemonic `v2p`.