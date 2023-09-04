# PD_ASIC
this is a repo that contains the notes and lab assignments for physical design for ASIC course
## RISCV
<details>
<summary>setting up the risc-v toolchain </summary>
1. the risc-v compiler version
![Screenshot from 2023-08-20 10-44-56]			(https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fc1fce85-47da-4347-b615-43a5367c0af2)

2. writing and compiling a c program

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

3. assembly program equivalent of the above c program we wrote using
the risc-v compiler
![Screenshot from 2023-08-20 11-10-01](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/f8eedd8f-0191-43fa-b1ee-50c693235359)
the -S flag tells the compiler to stop after assembly generation

![Screenshot from 2023-08-20 11-12-39](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fcab4a82-f31c-4278-a8e9-5f65051cc47f)
snippet of the first 25 lines  of the assembly code generated

4. spike simulation 
used to check whether the isntructions produced are right and we get the right output
![Screenshot from 2023-08-20 11-59-58](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1e6af2ce-e01d-4b9c-8b88-1f620d049ad2)

spike can also be used for debugging 
![Screenshot from 2023-08-20 12-01-59](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/a455e3fe-aab6-4a0d-b3ef-8519375702d6)
</details>
<details>
<summary>integer number representations</summary>

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

</details>

<details>
<summary> ABI INTERFACE </summary>

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

</details>

## SKY130

### intro to rtl design and synthesis
<details>
<summary>iverilog</summary>
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
</details>

<details>
<summary>yosys</summary>
1. yosys is a tool used to convert rtl to netlist it is called a synthesizer
2. we have a verilog design and have .lib file that contains the standard cells required for synthesis
given this to yosys it can generate a technology specific netlist
3. there is a read_verilog command to read design and read_liberty for the library file
and write_verilog to write out the netlist
4. now for checking whether the netlist generated is accurate we give the netlist and the testbench to iverilog
and we get a vcd file and use gtkwave to see the wave


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
![Screenshot from 2023-08-29 14-04-32](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fee9a105-6109-455b-85fb-fe418ed2a39e) 
2. now read the library using read_liberty -lib path
![Screenshot from 2023-08-29 14-14-40](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/a52b4805-d60f-4fdf-a54c-d246223a3933)
3. now read design file using read_verilog path
![Screenshot from 2023-08-29 14-18-20](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/0c30a230-1b97-4118-899c-7b958367f790)
4. synth -top module name to synthesize -top says that this is the top module
![Screenshot from 2023-08-29 14-22-00](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/d6ae5f34-9a6b-4b03-80d0-b10716aa86d0)
the output of the synthesis displays the number of wires used, number of standard cells used and the name of them
5. generating the netlist
![Screenshot from 2023-08-29 14-24-29](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/9b340817-44ce-47fd-9858-0d1483595bd1)
6. we can view the netlist by the show command
![Uploading Screenshot from 2023-08-29 14-25-00.png…]()
8. writing the netlist
![Screenshot from 2023-08-29 14-26-08](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/0a637a30-fff1-489a-a596-ad9055e24509)
9. viewing the netlist
![Screenshot from 2023-08-29 14-28-29](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/7c9f560a-4280-4e7e-8ab1-1689ba198d2f)

</details>

<details>
<summary>Timing libs</summary>
	
To view the contents inside the .lib file type the following command :
```
cd ASIC/sky130RTLDesignAndSynthesisWorkshop/lib/
gvim sky130_fd_sc_hd__tt_025C_1v80.lib
```
The name of the library file ("sky130_fd_sc_hd__tt_025C_1v80") means the following :

tt : indicates variations due to process and here it indicates Typical Process.
025C : indicates the variations due to temperatures where the silicon will be used.
1v80 : indicates the variations due to the voltage levels where the silicon will be incorporated.
One of the fundamental parameter stored within .lib files comprises PVT parameters, where P signifies Process, V represents Voltage, and T denotes Temperature. 

The variations in these parameters can cause significant changes in the performance of circuits.

1. Process Variation: During the manufacturing process, there may be some deviations in the transistor characteristics, causing non-uniformity across the semiconductor wafer. Critical parameters like oxide thickness, dopant concentration, and transistor dimensions experience alterations.

2. Voltage Variation: Voltage regulators might exhibit variability in their output voltage over time, inducing fluctuations in current and impacting the operational speed of circuits. 

3. Temperature Variation: The functionality of a semiconductor device is sensitive to changes in temperature, particularly at the internal junctions of the chip. 

Further it contains the technology that is used is CMOS for which delay are modelled  through table lookup. This file also defines the units for parameters like voltage, power, current, capacitance, and resistance. Within the .lib library, each standard cell consists a  set of parameters specific to that cell's features.

Consider the a2111oi gate whose parameters and verilog files is shown below:

1. here a21110i means and first 2 ips and or it with other inputs
2. we can check the verilog model of the file to understand functionality
3. Within the .lib file, sevetral parameters specific to this particular standard cell is given,
   - including leakage power values for every possible input combination,
   - specifications regarding pin type and pin capacitances,
   - internal power metrics,
   -  timing-related particulars,
   -  as well as area measurements and power-related specifics for the standard cells
4. Similarly for all the standard cells the parameters above mentioned is listed in the .lib file.

Consider the different versions of the same logic gate shown below:


In all the three the logic inferred is same but the area is different. Wider cells consume more power but delay wise it is less. The leakage power in the wider cell is more compared to the narrow cell which is depicted in the image .
</details>

<details>
<summary>Hiererchical Synthesis and Flat Synthesis</summary>

1. Hierarchical synthesis is breaking a comples modules into smaller more manageable sub-modules or blocks. Each of these sub-modules can be synthesized or designed independently before being integrated into the larger system.
2. This approach allows for efficient design, optimization, and verification of individual components while maintaining a structured and organized design process.
3. An illustration of the hierarchical synthesis is shown below 

Consider the verilog file multiple module which is given in the verilog_files directory

 ```
module sub_module2 (input a, input b, output y);
	assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
	assign y = a&b;
endmodule


module multiple_modules (input a, input b, input c , output y);
	wire net1;
	sub_module1 u1(.a(a),.b(b),.y(net1));  //net1 = a&b
	sub_module2 u2(.a(net1),.b(c),.y(y));  //y = net1|c ,ie y = a&b + c;
endmodule
 ```

 In this case the module multiple_modules iinstantiates two sub_modules where the sub_module1 implements the AND gate and sub_module2 implemets the OR gate which are integrated in the multiple_modules.  Synthesis the multiple module using the sollowing commands:
 ```
 # Remove "#" if needed
 cd /home/jitesh/ASIC/sky130RTLDesignAndSynthesisWorkshop/verilog_files
 yosys
 read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
 read_verilog 
 read_verilog multiple_modules.v 
 synth -top multiple_modules
 abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
 show multiple_modules
 write_verilog multiple_modules_hier.v
 ``` 
 ___
 **Note:**</br>
 When using hierarchical design instead of enetering the ***show*** command to view the file ***show <module_name>*** must be otherwise yosys will generate the following error : "ERROR: For formats different than 'ps' or 'dot' only one module must be selected."
 ___
 
1. Yosys does not show the AND gate and OR gate in the synthesis instead it shows the submodule names the netlist also contains the AND and OR logic in separate submodules.
2. Some times yosys may optimize the design such that the OR gate will be created using NAND gates it is because the CMOS structure of the OR gate which is shown below has two pmos transistors stacked together the mobility of the holes is less than the mobility of the electrons
3. since mosfets are majority carrier devices and majority carrier of the pmos is holes it increases the delay hence it becomes a bad circuit. In NAND gate implementation only the nmos are stacked.


Flattening the hierarchy means simplifying the hierarchical structure of a design by collapsing or merging lower-level modules or blocks into a single, unified representation. In yosys the flattening can be done with ***flat*** command. Yosys illustration of flattening the hiererchy.

```
 cd /home/kanish/ASIC/sky130RTLDesignAndSynthesisWorkshop/verilog_files
 yosys
 read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
 read_verilog 
 read_verilog multiple_modules.v 
 synth -top multiple_modules
 abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
 flatten
 show
 write_verilog multiple_modules_flat.v
```

The flatten command breaks the hierarchy and makes the design into a single module by creating AND and OR gates for the logics inferred by the submodule which is shown in the images above.

### **Synthesising a Submodule :**
Suppose a multiplier design needs to be used in numerous instances. Rather than undergoing synthesis six times independently, the preferred approach is to synthesize it once and then duplicate it within the primary module. Using module-level synthesis becomes advantageous when dealing with multiple occurrences of identical modules. Another reason for synthesizing submodule is to follow the principle of divide and conque for extensive designs that may not be optimized effectively, synthesizing the design module by module ensures that each module is effectively optimized.

**Steps to synthesis submodule :**

```
cd /home/kanish/ASIC/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog multiple_modules.v 
synth -top sub_module
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
```
</details>



<details>
<summary> flip flops </summary>

1. A flip-flop is a fundamental sequential synchronous electronic circuit that is capable of storing information a single flip-flop can store 1- bit of information and several flip-flops can be grouped together to form registers and memory that can store multiple bits of information.
2. There are several types of flip-flops like JK flip-flop, D flip-flop, T flip-flop and SR flip-flop but D flip-flop is widely and most commanly used since it transmits the input data to the output without performing any modifications.
3. A D flop-flop needs two inputs : data and clock. The flip-flop can be positive-edge triggered or negative-edge triggered i.e, the output makes transition during the rising edge of the clock pulse if it is positive-edge triggered and if the output makes transition during the falling edge of the clock pulse then it is said to be negative- edge triggered.

### **Need of flip-flops**</br>
In any electronic circuit there will always be an propagation delay. These delays may cause glitches in the output which may cause the output state to change when it is not supposed to. Glitches are unwanted transitions in the output. As an illustration consider the circuit shown below:

1. The propagation delay of the OR gate is 1ns and AND gate is 2ns. Initially a,b,c are 0,0,1 and the internal node i0 is 0 and the output Y is high.
2. At t=0ns there is change in the inputs a,b,c becomes 1,1,0. because of the propagation delays of the AND gate and OR gate at t=1ns the output node transits from high to low and since the input to the OR gate both i0 and c are 0.
3. At t=2ns the internal node i0 transists from 0 to 1 and  the inputs to the OR gate becomes 1 and 0. Since the propagation delay of the OR gate is 1ns the output Y becomes high at 3ns and remains stable. Between 1ns and 3ns the output made an unwanted change in the transition resulting in a glitch.  

In order to avoid the glitches a D flip-flop can be connected at the output so that the output will change only at the rising or falling edge of the clock. As mentioned earlier flip-flops generally needs two inputs: data and clock. But the problem is the initial state of the flip-flop is unknown. So in order to set the initial value of the flip-flop, two more inputs are provided : preset/set and reset. These additional inputs can be synchronous with clock or asynchronous with clock.

**Steps to simulate and generate the netlist for the below designs**

Simulation steps :
```
iverilog <rtl_name.v> <tb_name.v>
./a.out
gtkwave <dump_file_name.vcd>
```

Generating netlist steps :
```
# Remove "#" if needed
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib  
read_verilog <module_name.v> 
synth -top <top_module_name>
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
write_verilog -noattr <netlist_name.v>
```

___
***Note***:</br>
**dfflibmap** - technology mapping of flip-flops</br>
dfflibmap  -liberty - Maps internal flip-flop cells to the flip-flop cells in the technology library specified in the given liberty file.

Generally in the flow there will be a separate .lib file for the flip-flops which needs to be used with the dfflibmap command.
___

### **Illustration of Different types of Flip-flop** 
**1. D flip-flop with Synchronous reset**</br>
A D flip-flop with synchronous reset  combines the functionality of a D flip-flop with the ability to reset its state synchronously. This means that the flip-flop's stored value can be reset to 0 or low state based on a clock signal and a reset input, ensuring that the reset operation occurs when the clock signal transits.
The verilog code, simulation and synthesis results are shown below:
```
module dff_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk )
begin
	if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```



**2. D flip-flop with Asynchronous reset**</br>
A D flip-flop with asynchronous reset combines the functionality of a D flip-flop with the ability to reset its state asynchronously. This means that the flip-flop's stored value can be reset to 0 or low state regardless of the clock signal's state.
The verilog code, simulation and synthesis results are shown below:
```
module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```



**3. D flip-flop with Asynchronous set**</br>
A D flip-flop with asynchronous set combines the functionality of a D flip-flop with the ability to set its state asynchronously. This means that the flip-flop's stored value can be set to 1 or high state regardless of the clock signal's state.
The verilog code, simulation and synthesis results are shown below:

```
module dff_async_set ( input clk ,  input async_set , input d , output reg q );
always @ (posedge clk , posedge async_set)
begin
	if(async_set)
		q <= 1'b1;
	else	
		q <= d;
end
endmodule
```


**4. D flip-flop with Asynchronous and Synchronous reset**</br>
A D flip-flop with both asynchronous and synchronous reset that combines the features of a D flip-flop with the ability to reset its state using either an asynchronous reset input or a synchronous reset input. This provides flexibility in resetting the flip-flop's state under different conditions.

The verilog code, simulation and synthesis results are shown below:

```
module dff_asyncres_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```


### **Optimizations**
During synthesis yosys will perform optimisations based on the logic that is being designed. An illustration of the yosys optimization is given below:

**1. Optimisation Example 1**

Consider the verilog design given below:
```
module mul2 (input [2:0] a, output [3:0] y);
	assign y = a * 2;
endmodule
```
This code performs multiplication of the input number by 2. Since the input is 3-bit binary number all the input and output combinations are as follows:
| a2 a1 a0  |y3 y2 y1 y0   |
|:---:|:---:|
| 0 0 0 | 0 0 0 0  |
| 0 0 1 | 0 0 1 0  |
| 0 1 0 | 0 1 0 0  |
| 0 1 1 | 0 1 1 0  |
| 1 0 0 | 1 0 0 0  |
| 1 0 1 | 1 0 1 0  |
| 1 1 0 | 1 1 0 0  |
| 1 1 1 | 1 1 1 0  |

y0 is always 0 and the code doesn't need any hardware and it only needs the proper wiring of the input bits to the output and grounding the bit y0. The netlist of the design is shown below:


**2. Optimisation Example 2**

Consider the verilog design given below:
```
module mult8 (input [2:0] a , output [5:0] y);
	assign y = a * 9;
endmodule
```
In this design the 3-bit input number "a" is multiplied by 9 i.e.,(a*9) which can be re-written as (a\*8) + a . The term (a\*8) is nothing but a left shifting the number a by three bits. Consider that a = a2 a1 a0. (a\*8) results in a2 a1 a0 0 0 0. (a\*9)=(a\*8)+a = a2 a1 a0 a2 a1 a0 = aa(in 6 bit format). Hence in this case no hardware realization is required. The synthesized netlist of this design is shown below:

</details>

### combinational and sequential optimizations
<details>
<summary>combinational optimizations</summary>

### **Logic Optimisations**
In a broader context, Digital electronics encompasses two types of optimisations: Combinational and Sequential optimisations. These optimisations are done inorder to achieve designs that are efficient in terms of area, power, and performance.

### **Combinational Optimisations**
The techniques used for optimising the combinational Circuits are as follows:
1. Constant Propagation (Direct Optimisation)
2. Boolean Logic Optimisation (using K-Map or Quine McCluskey method)

#### **1. Constant Propagation Illustration**
Consider the combinational circuit shown below :

The boolean logic inferred is Y = ((AB)+C)'. If A is always tied to ground i.e., A = 0, then the expression will always evaluate to C'. In this case instead of having a AND gate and a NOR gate the circuit can be simplified by using a single NOT gate with C as its input. Even though both of then represent the same logic since the number of transistors used in the optimised design is less compared to that of the given circuit which shown in the above figure. The transistor level implementation of the given circuit and the optimised circuit is shown below :


The circuit that is given is implemented in NAND logic in order to prevent the stacking of the pmos. The transistor implementation clearly demonstrates a reduction in the required number of transistors for designing, decreasing from 12 to 2 in the optimised design. This will result in reduced power consumption and occuppies less area.

#### **2. Boolean Logic Optimisation Illustration**
Consider the verilog statement below : 
```
assign y = a?(b?c:(c?a:0)):(!c);
```
The ternary operator **(?:)** will realize a mux upon synthesis. The combinational circuit that corresponds to the above statement is shown below:


This circuit can be optimised by writing the equivalent expression (or function) in boolean variables and minimising the function that will result in more optimised design which is shown below:


### **Illustration of Combinational Optimizsation:**

**Steps to generate the netlist for the below designs**

Generating netlist steps :
```
# Remove "#" if needed
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib  
read_verilog <module_name.v> 
synth -top <top_module_name>
# flatten # Use if multiple modules are present
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
write_verilog -noattr <netlist_name.v>
```

___
**opt_clean** - remove unused cells and wires. The ***-purge*** switch removes internal nets if they have a public name. This command identifies wires and cells that are unused and removes them.  This command can be used to clean up after the commands that do the actual work.
___

#### **Example 1**
The verilog code for the example 1 is given below :
```
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```
The above code infers a multiplexer as shown below :

Since one of the inputs of the multiplexer is always connected to the ground it will infer an AND gate on optimisation.



The synthesis result and the netlist are shown below :



#### **Example 2**
The verilog code for the example 2 is given below :
```
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```
The above code infers a multiplexer as shown below :



Since one of the inputs of the multiplexer is always connected to the logic 1 it will infer an OR gate on optimisation. The OR gate will be NAND implementation since NOR gate has stacked pmos while NAND implementation has stacked nmos.



The synthesis result and the netlist are shown below :




#### **Example 3**
The verilog code for the example 3 is given below :
```
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```
The above code infers two multiplexers as shown below : 



On optimisation the above design becomes a 3 input AND gate as shown below :



The synthesis result and the netlist are shown below :



#### **Example 4**
The verilog code for the example 4 is given below :
```
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```
The above code infers two multiplexers as shown below : 



On optimisation the above design becomes a 2 input XNOR gate as shown below :



The synthesis result and the netlist are shown below :


#### **Example 5**
The verilog code for the example 5 is given below :
```
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule


module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule


module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));

assign y = c | (b & n1); 


endmodule
```

The circuit inferred by the code is shown below : 



On optimisation the above design becomes a AND OR gate as shown below :



The synthesis result and the netlist are shown below :




#### **Example 6**
The verilog code for the example 6 is given below :
```
module sub_module(input a , input b , output y);
 assign y = a & b;
endmodule



module multiple_module_opt2(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module U1 (.a(a) , .b(1'b0) , .y(n1));
sub_module U2 (.a(b), .b(c) , .y(n2));
sub_module U3 (.a(n2), .b(d) , .y(n3));
sub_module U4 (.a(n3), .b(n1) , .y(y));


endmodule
```

The circuit inferred by the code is shown below : 



On optimisation the above design becomes a direct connection of ground (logic 0) to output as shown below :



The synthesis result and the netlist are shown below :







</details>

<details>
<summary>sequential optimizations</summary>

 ### **Sequential Optimisations**
The sequential logic optimisations techniques are broadly classified into two categories :
1. Basic Techniques
	a. Sequential Constant Propagation
2. Advanced Techniques
	a. State Optimisation
	b. Retiming
	c. Sequential Logic Cloning (Floor aware Synthesis)

#### **1. Sequential Constant Propagation**
Consider the sequential circuit shown below :



The D flip-flop shown in the figure is positive edge triggered with asynchronous reset and the data input D is always tied to the ground (i.e, low state or logic 0). When reset is applied the output of the flop becomes low and if reset it deasserted the output of the flop still remains low. Hence one of the input to the NAND gate is always low resulting in the output Y to be always in high stae (logic 1 or VDD). Hence the optimised version of this circuit is connecting the output port Y directly to VDD i.e., the supply voltage.

___
***Note***: </br>
Consider the circuit shown below :


This circuit is similar to the one that is discussed above except that it doesn't have asynchronous reset instead it has asynchronous set. When the set input is logic 1 then output of the flop i.e., Q becomes high otherwise Q follows D input which is logic 0. This circuit can't be optimised like the previous circuit discussed in the above section. Consider the waveform between timestamp 1 and timestamp 2, the set pin is deasserted before the rising edge of the clock. The output Q remains high until the next rising edge even though the set input is deasseretd. The output of thr flop Q makes transition only at timestamp2. Therefore set input must be considered as Q'. This circuit can't be optimised.
___

#### **2. State Optimisation**
State optimization refers to the process of minimizing the number of unused states in a digital circuit's state machine.

#### **3. Sequential Logic Cloning**
Sequential logic cloning is used to replicate or clone a portion of a sequential logic circuit while maintaining its functionality and behavior. The goal is to exploit the benefits of parallelism and redundancy while ensuring that the cloned circuit produces identical outputs to the original circuit for the same inputs.
This technique is commonly employed in various scenarios such as redundancy for fault tolerance, speed improvement, and power optimization. This technique is generally used when a physical aware synthesis is done.

Consider the circuit shown below : 



Consider flop A has large positive slack. The flops B and C are far from flop A. Hence there will be a large routing delay from A to B and A to C. To avoid this flop A and the combinational logic 2 is replicated  or cloned in the paths of B and C as shown in the figure below. Since flop A has large positive slack the delay introduced because of the cloning will be compensated and the further delay in the circuit is mainly depended on flop B and flop C.



#### **4. Retiming**
Retiming  used to improve the performance interms of better timing characteristics by repositioning the registers (flip-flops) within the circuit without altering its functionality. In a digital circuit, registers (flip-flops) are used to store intermediate results and control the flow of data. The placement of these registers can significantly impact the circuit's overall performance, including its critical path delay, clock frequency, and power consumption. Retiming aims to optimize these factors by moving registers to appropriate locations within the circuit.

Consider the circuit shown below :


Consider the C-Q delay and set up time is 0ns. The combinational circuits have finite amount of the propagation delay. The maximum clock frequency with which the circuit operates depends on the propagation delay of the combinational logic. From flop A to B the propagation delay is 5ns and the maximum frequency with which this portion of circuit can be operated is 200MHz. Fom flop B to C the propagation delay is 2ns and the maximum frequency with which this portion of circuit can be operated is 500MHz. The effective frequency is minimum of the both which is 200MHz.

Suppose some part of the logic from combinational circuit between flop B and C is placed with the combinational circuit between the flop A and flop B in such a way that the propagation delay of the circuit between flop A and flop is reduced while propagation delay between flop B and flop C is increased by a small amount as show below :


The maximum frequency with which the portion of circuit between A and B can be operated is 250MHz and the maximum frequency with which the portion of circuit between B and C can be operated is 333MHz. The effective frequency is minimum of the both which is 250MHz. Thus the effective maximum frequency has increased after performing the retiming.

</details>














