# PD_ASIC
this is a repo that contains the notes and lab assignments for physical design for ASIC course

## Day 1 <setting up the risc-v toolchain>
1.the risc-v compiler version
![Screenshot from 2023-08-20 10-44-56](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fc1fce85-47da-4347-b615-43a5367c0af2)

2.writing and compiling a c program

~~~c
#include<stdio.h>

int main()
{
int i,sum=0,n=15;
for (i=1;i<=n;++i)
{
sum=sum+i;
}
printf("sum of numbers from i to %d is %d /n ",n,sum);
}
~~~
![Screenshot from 2023-08-20 11-01-39](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/76e1d4a6-ad34-495b-b14e-a543886a484a)

3.assembly program equivalent of the above c program we wrote using
the risc-v compiler
![Screenshot from 2023-08-20 11-10-01](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/f8eedd8f-0191-43fa-b1ee-50c693235359)
the -S flag tells the compiler to stop after assembly generation

![Screenshot from 2023-08-20 11-12-39](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fcab4a82-f31c-4278-a8e9-5f65051cc47f)
snippet of the first 25 lines  of the assembly code generated

4.spike simulation 
used to check whether the isntructions produced are right and we get the right output
![Screenshot from 2023-08-20 11-59-58](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1e6af2ce-e01d-4b9c-8b88-1f620d049ad2)

spike can also be used for debugging 
![Screenshot from 2023-08-20 12-01-59](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/a455e3fe-aab6-4a0d-b3ef-8519375702d6)

## integer number representations

let's write a c program that shows max and min length of an unsigned
64 bit integer
~~~
#include <stdio.h>
#include <math.h>

int main(){
	unsigned long long int max = (unsigned long long int) (pow(2,64) -1);
	unsigned long long int min = (unsigned long long int) (pow(2,64) *(-1));
	printf("lowest number represented by unsigned 64-bit integer is %llu\n",min);
	printf("highest number represented by unsigned 64-bit integer is %llu\n",max);
	return 0;
}
~~~
![Screenshot from 2023-08-20 12-08-52](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e5cf2d03-afaa-4465-9a64-825a18851ba6)

let's do the same for signed numbers

![Screenshot from 2023-08-20 12-17-19](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/7d44944a-453a-4264-a737-f47a1a716f71)

# DAY 2 (ABI INTERFACE)
## Application binary interface
ABI is a set of rules that tell us how binary code interacts with another binary code. 64 bit value can be loaded into the memory by 2 methods - little-endian and big-endian. Load instruction is used to transfer data from memory to a register. Store instruction is used to transfer data from register to memory. Add instruction performs addition operation on two registers. In RISC-V 64, we have 32 registers and their ABI names play a role in maintaining compatibility and facilitating communication between different software components
## Labwork using ABI function calls
c code for adding numbers 1 to 9 named prgm
~~~
#include <stdio.h>

extern int load(int x, int y);

int main(){
	int result = 0;
	int count = 9;
	result = load(0x0,count+1);
	printf("sum of numbers from 1 to 9 is %d \n",result);
}
~~~
the above c code in risc-v instruction set named load.s
~~~
.section .text
.global load
.type load, @function

load:
	add a4,a0,zero //initialize a4 with value 0x0
	add a2,a0,a1   //store value as 10 in a2, a1 has value 0xa from main function
	add a3,a0,zero //initialize a3 with value 0 
loop:   add a4,a3,a4   //incremental addition
	addi a3,a3,1   //increment a3 by 1
	blt a3,a2,loop //if a3 is lesser than a2 then pass through the loop again
	add a0,a4,zero //store final answer in a0
	ret
~~~
### compiling the c and assembly code
![Screenshot from 2023-08-22 15-15-22](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/d1fb169f-3401-44bb-ba8a-3413b9d95dc0)

# sky130 
## day 1
1.the rtl design is the implementation of a spec and we check the
functionality by simulating the design in a simulator
2.the simulator we'd be using is iverilog
3.the design is a set of verilog codes that has implemented the spec
like say full adder implemented with a lot of sub-blocks
4.a testbench is the setup  to apply some inputs and check whether 
the design is working as required
5.how does the simulator work it looks for changes in input and responds to them if there is no change in ip no change in op
6.we provide a design file and a testbench corresponding to the design file to iverilog and it generates a vcd file
(value change dump)
7.this vcd file cannot be directly viewwed and we use this other application called gtkwave to view the vcd file
## day 1_lab
1.running iverilog and gtkwave
~~~
iverilog good_mux.v tb_good_mux.v 
./a.out
gtkwave tb_good_mux.vcd
~~~
snapshots of the execution

![Screenshot from 2023-08-27 22-07-08](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/5a138dd9-bed7-4e78-9792-4cbac91ccc81)
![Screenshot from 2023-08-27 22-07-40](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/7d05390b-e7f4-449f-bfcd-3efcdf202dd5)

2.verilog design and testbench codes that we executed
~~~
module good_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
~~~

~~~
`timescale 1ns / 1ps
module tb_good_mux;
	// Inputs
	reg i0,i1,sel;
	// Outputs
	wire y;

        // Instantiate the Unit Under Test (UUT)
	good_mux uut (
		.sel(sel),
		.i0(i0),
		.i1(i1),
		.y(y)
	);

	initial begin
	$dumpfile("tb_good_mux.vcd");
	$dumpvars(0,tb_good_mux);
	// Initialize Inputs
	sel = 0;
	i0 = 0;
	i1 = 0;
	#300 $finish;
	end

always #75 sel = ~sel;
always #10 i0 = ~i0;
always #55 i1 = ~i1;
endmodule

~~~

## day 2
### class 1
1. yosys is a tool used to convert rtl to netlist it is called a synthesizer
2. we have a verilog design and have .lib file that contains the standard cells required for synthesis
given this to yosys it can generate a technology specific netlist
3. there is a read_verilog command to read design and read_liberty for the library file
and write_verilog to write out the netlist
4. now for checking whether the netlist generated is accurate we give the netlist and the testbench to iverilog
and we get a vcd file and use gtkwave to see the wave

### class 2
1. what is rtl design it is the  behavioural representation  of the required specs using a hdl

but what we need is a hardware not code ?
the rtl design  is converted into gates and the connection are made between the gates and given out 
in a file called netlist this is done by a logic synthesizer such as yosys

rtl --> synthesis --> netlist

2. what is .lib it is a collection  of logical modules including basic gates etc it may contain modifications of the same gate	
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/b3cdbf1f-2186-4571-a49d-d0b08f2b9612)



3. we'd need fast gates cuz we wanna reduce t_comb as that is the only parameter that we can modify and play around with in sequential ckts
as tpcq,tccq etc are fixed values
4. the slow cells are required to address hold time issues
load in digital ckts are capacitance
faster the charging/discharging of capacitance lesser the delay
->to charge/discharge the capacitances faster, we need transistors capable of sourcing more current higher w/l
->wider transistors low delay but more area and power
->narrow transistors high delay but less area and power
5. what kinda cells to use faster slower medium depends on the constraints we set the synthesizer optimizes the best soln
in all parameters according to these constraints
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4c7bb81a-c38e-4527-936e-77b0654f4fe9)

## yosys and logic synthesis
1. just typing out yosys in your shell will invoke yosys
   
3. now read the library using read_liberty -lib path
   
4. now read design file using read_verilog path
5. synth -top module name to synthesize -top says that this is the top module





