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
1. the rtl design is the implementation of a spec and we check the
functionality by simulating the design in a simulator
2. the simulator we'd be using is iverilog
3. the design is a set of verilog codes that has implemented the spec
like say full adder implemented with a lot of sub-blocks
4. a testbench is the setup  to apply some inputs and check whether 
the design is working as required
5. how does the simulator work it looks for changes in input and responds to them if there is no change in ip no change in op
6. we provide a design file and a testbench corresponding to the design file to iverilog and it generates a vcd file
(value change dump)
7. this vcd file cannot be directly viewwed and we use this other application called gtkwave to view the vcd file


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
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1e937c8a-25c7-401e-aee0-38c7b2a21a38)

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
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/31b66402-36be-4119-ac13-38d1d581cf22)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/662accaf-331f-4837-958b-546b6468fcf8)

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
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/da33fe3e-95ab-4a0a-8508-8c62b9341185)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/36a40662-9dc1-4ed2-b2a8-60de24e324ce)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/19dcab1b-4aa0-4f30-9a59-7c1118e550b3)



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
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4774add9-bc2c-4a23-8297-51daeec08046)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/8ab4b5d6-6422-4684-90b2-48567e2418d1)
![hierarchi_des](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/93b37b25-3098-4adb-a19f-2340212eb13a)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/ffafed20-1fc6-4ddf-bc62-18edad8b9907)

 
1. Yosys does not show the AND gate and OR gate in the synthesis instead it shows the submodule names the netlist also contains the AND and OR logic in separate submodules.
2. Some times yosys may optimize the design such that the OR gate will be created using NAND gates it is because the CMOS structure of the OR gate which is shown below has two pmos transistors stacked together the mobility of the holes is less than the mobility of the electrons
3. since mosfets are majority carrier devices and majority carrier of the pmos is holes it increases the delay hence it becomes a bad circuit. In NAND gate implementation only the nmos are stacked.

![CMOS_OR](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/b7f5ed53-23e7-4252-8914-1f709a934858)


Flattening the hierarchy means simplifying the hierarchical structure of a design by collapsing or merging lower-level modules or blocks into a single, unified representation. In yosys the flattening can be done with ***flat*** command. Yosys illustration of flattening the hiererchy.

```
 cd /home/jitesh/ASIC/sky130RTLDesignAndSynthesisWorkshop/verilog_files
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
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/db40caaf-453d-4436-9041-767c3e255ce0)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/bc81cab8-3b53-4920-9304-e8fef3e8a076)


### **Synthesising a Submodule :**
Suppose a multiplier design needs to be used in numerous instances. Rather than undergoing synthesis six times independently, the preferred approach is to synthesize it once and then duplicate it within the primary module. Using module-level synthesis becomes advantageous when dealing with multiple occurrences of identical modules. Another reason for synthesizing submodule is to follow the principle of divide and conque for extensive designs that may not be optimized effectively, synthesizing the design module by module ensures that each module is effectively optimized.

**Steps to synthesis submodule :**

```
cd /home/jitesh/ASIC/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog multiple_modules.v 
synth -top sub_module
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
```
![submodule_synth](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/52d31fc4-ac67-4ea5-a15c-5e5c7c84be27)
</details>


<details>
<summary> flip flops </summary>

1. A flip-flop is a fundamental sequential synchronous electronic circuit that is capable of storing information a single flip-flop can store 1- bit of information and several flip-flops can be grouped together to form registers and memory that can store multiple bits of information.
2. There are several types of flip-flops like JK flip-flop, D flip-flop, T flip-flop and SR flip-flop but D flip-flop is widely and most commanly used since it transmits the input data to the output without performing any modifications.
3. A D flop-flop needs two inputs : data and clock. The flip-flop can be positive-edge triggered or negative-edge triggered i.e, the output makes transition during the rising edge of the clock pulse if it is positive-edge triggered and if the output makes transition during the falling edge of the clock pulse then it is said to be negative- edge triggered.

### **Need of flip-flops**</br>
![glitch](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c0af23c2-daf0-4948-b1fe-ecf5eb63b9bd)
![glitch_plot](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/ebb7db29-4ddf-4967-949f-8c30c5b0fb69)

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
![wav1](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/0d92cd8d-e9a9-442a-a86c-33c3f6c43909)
![net1](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/93e8727f-9437-4fa0-9054-38567049a081)

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
![wav2](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/983435db-ab32-40fe-b10d-2d8eb5efdaa1)
![net2](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/335e152c-5636-412a-b127-b6a05b52feb7)


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
![wav3](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4892639c-beb1-48e1-a2ec-7ce9556c2141)
![net3](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/b193fd6b-3dba-44ce-8b11-f01782fda026)


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
![wav4](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fbde31fc-2861-4d6e-80ac-1d92cee58806)
![net4](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/cd20d35b-7c13-48f1-a352-4459485148a1)


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

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e8b8f7d6-9bb1-4c74-9f0f-dea09e948283)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/859f8794-1e32-4e78-893b-29dfe8a7c6e4)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4487910c-433c-481e-87b4-6791fcc297e6)



**2. Optimisation Example 2**

Consider the verilog design given below:
```
module mult8 (input [2:0] a , output [5:0] y);
	assign y = a * 9;
endmodule
```
In this design the 3-bit input number "a" is multiplied by 9 i.e.,(a*9) which can be re-written as (a\*8) + a . The term (a\*8) is nothing but a left shifting the number a by three bits. Consider that a = a2 a1 a0. (a\*8) results in a2 a1 a0 0 0 0. (a\*9)=(a\*8)+a = a2 a1 a0 a2 a1 a0 = aa(in 6 bit format). Hence in this case no hardware realization is required. The synthesized netlist of this design is shown below:

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/60d884e7-5041-438b-b5b0-06f3a0b73a35)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/947a4a19-0ac0-4fc9-8981-ee29e21cf294)
![mult8](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/38b0705d-526d-4474-b9c9-88c2c8ea96f5)



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
![propag](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/40c5e2dd-5836-46c2-af2e-39123d88376c)


The boolean logic inferred is Y = ((AB)+C)'. If A is always tied to ground i.e., A = 0, then the expression will always evaluate to C'. In this case instead of having a AND gate and a NOR gate the circuit can be simplified by using a single NOT gate with C as its input. Even though both of then represent the same logic since the number of transistors used in the optimised design is less compared to that of the given circuit which shown in the above figure. The transistor level implementation of the given circuit and the optimised circuit is shown below :

![ckt1](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/a0379a57-2122-4310-8e06-2f37ebd508b9)
![ckt2](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e5f2a095-c3e5-4e55-aa7c-f95372dffa3d)

The circuit that is given is implemented in NAND logic in order to prevent the stacking of the pmos. The transistor implementation clearly demonstrates a reduction in the required number of transistors for designing, decreasing from 12 to 2 in the optimised design. This will result in reduced power consumption and occuppies less area.

#### **2. Boolean Logic Optimisation Illustration**
Consider the verilog statement below : 
```
assign y = a?(b?c:(c?a:0)):(!c);
```
The ternary operator **(?:)** will realize a mux upon synthesis. The combinational circuit that corresponds to the above statement is shown below:
![1](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/5c4804cc-ebc8-4032-911e-c2b1ae330d1f)

This circuit can be optimised by writing the equivalent expression (or function) in boolean variables and minimising the function that will result in more optimised design which is shown below:
![bl_opt](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/70501933-51cd-4fd7-80b4-640cb68e7362)

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
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/5de45f59-ba66-4b36-80f9-808b5099422a)


Since one of the inputs of the multiplexer is always connected to the ground it will infer an AND gate on optimisation.
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/30f1995e-cfc2-4be0-a8eb-95079c0f4a36)

The synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/363b1da1-6702-492d-8038-a0edd45920b0)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/112cd21c-5d60-4a98-be77-1ca7cb698b59)



#### **Example 2**
The verilog code for the example 2 is given below :
```
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```
The above code infers a multiplexer as shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4948b838-7b5b-4f82-aed6-d97dc8fe1d4e)


Since one of the inputs of the multiplexer is always connected to the logic 1 it will infer an OR gate on optimisation. The OR gate will be NAND implementation since NOR gate has stacked pmos while NAND implementation has stacked nmos.

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/6fa873cd-d884-44af-b21a-a70eb488cb93)

The synthesis result and the netlist are shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1f8d3b77-6ffe-45f5-b2d9-1fa15f997f0d)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/0025a3e6-3058-486b-9ee7-43655d65c8d8)



#### **Example 3**
The verilog code for the example 3 is given below :
```
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```
The above code infers two multiplexers as shown below : 

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/0cc86bfc-a118-427e-ada4-a526e82d4cbd)


On optimisation the above design becomes a 3 input AND gate as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/adcc49de-3659-49cb-9e1b-9ffe679965bf)


The synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/bacea75d-622c-408c-9cf1-8d26db2e778f)

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/8b85458b-8624-409a-89b0-6b9aa553a50a)


#### **Example 4**
The verilog code for the example 4 is given below :
```
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```
The above code infers two multiplexers as shown below : 

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c19e9f8e-a234-4a7a-b563-25230bffcf73)


On optimisation the above design becomes a 2 input XNOR gate as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/8ae4ace0-6bd7-4fb7-8b43-6bf27787d62e)


The synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e2b1289a-f3ae-407c-ab59-31b7100594c4)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/bfe24864-076a-44ed-b101-6e0862b26826)


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

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/0a54ce2e-3680-495c-acd8-3f4264d24341)


On optimisation the above design becomes a AND OR gate as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/8ade3b15-49f1-4445-ae10-7d6578a75eac)


The synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/136a733a-063b-427c-9830-d52745763d6c)


![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/8785a593-01a1-496d-8c8d-6fac408b3c4b)


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

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fbf80926-d64b-40e0-9dfb-3e5cd354a9b7)


On optimisation the above design becomes a direct connection of ground (logic 0) to output as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/9ce8f38c-3d52-4a0d-9c25-5c5fd4cdb9a8)


The synthesis result and the netlist are shown below :


![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/17731ff3-cb1e-48c5-94ef-8594c8db704f)

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fc3fe3ca-2197-400e-94f1-b3d2ffae947d)




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

[image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e27a0e31-da2d-4b29-ba60-b8c699bb0906)

The D flip-flop shown in the figure is positive edge triggered with asynchronous reset and the data input D is always tied to the ground (i.e, low state or logic 0). When reset is applied the output of the flop becomes low and if reset it deasserted the output of the flop still remains low. Hence one of the input to the NAND gate is always low resulting in the output Y to be always in high stae (logic 1 or VDD). Hence the optimised version of this circuit is connecting the output port Y directly to VDD i.e., the supply voltage.

___
***Note***: </br>
Consider the circuit shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/df66ca8b-0c5e-4a46-a90a-3ce5d015e81e)




This circuit is similar to the one that is discussed above except that it doesn't have asynchronous reset instead it has asynchronous set. When the set input is logic 1 then output of the flop i.e., Q becomes high otherwise Q follows D input which is logic 0. This circuit can't be optimised like the previous circuit discussed in the above section. Consider the waveform between timestamp 1 and timestamp 2, the set pin is deasserted before the rising edge of the clock. The output Q remains high until the next rising edge even though the set input is deasseretd. The output of thr flop Q makes transition only at timestamp2. Therefore set input must be considered as Q'. This circuit can't be optimised.
___

#### **2. State Optimisation**
State optimization refers to the process of minimizing the number of unused states in a digital circuit's state machine.

#### **3. Sequential Logic Cloning**
Sequential logic cloning is used to replicate or clone a portion of a sequential logic circuit while maintaining its functionality and behavior. The goal is to exploit the benefits of parallelism and redundancy while ensuring that the cloned circuit produces identical outputs to the original circuit for the same inputs.
This technique is commonly employed in various scenarios such as redundancy for fault tolerance, speed improvement, and power optimization. This technique is generally used when a physical aware synthesis is done.

Consider the circuit shown below : 
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/7f0a0a13-9b0c-491e-8b62-53948674b03d)



Consider flop A has large positive slack. The flops B and C are far from flop A. Hence there will be a large routing delay from A to B and A to C. To avoid this flop A and the combinational logic 2 is replicated  or cloned in the paths of B and C as shown in the figure below. Since flop A has large positive slack the delay introduced because of the cloning will be compensated and the further delay in the circuit is mainly depended on flop B and flop C.

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/8db3e6cb-0ee1-414a-a66b-1deb6d53adf8)


#### **4. Retiming**
Retiming  used to improve the performance interms of better timing characteristics by repositioning the registers (flip-flops) within the circuit without altering its functionality. In a digital circuit, registers (flip-flops) are used to store intermediate results and control the flow of data. The placement of these registers can significantly impact the circuit's overall performance, including its critical path delay, clock frequency, and power consumption. Retiming aims to optimize these factors by moving registers to appropriate locations within the circuit.

Consider the circuit shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/61633200-cd69-430a-973e-b5129674573b)


Consider the C-Q delay and set up time is 0ns. The combinational circuits have finite amount of the propagation delay. The maximum clock frequency with which the circuit operates depends on the propagation delay of the combinational logic. From flop A to B the propagation delay is 5ns and the maximum frequency with which this portion of circuit can be operated is 200MHz. Fom flop B to C the propagation delay is 2ns and the maximum frequency with which this portion of circuit can be operated is 500MHz. The effective frequency is minimum of the both which is 200MHz.

Suppose some part of the logic from combinational circuit between flop B and C is placed with the combinational circuit between the flop A and flop B in such a way that the propagation delay of the circuit between flop A and flop is reduced while propagation delay between flop B and flop C is increased by a small amount as show below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fe93e51e-ac8e-4a80-b568-4ea63fa4b111)


The maximum frequency with which the portion of circuit between A and B can be operated is 250MHz and the maximum frequency with which the portion of circuit between B and C can be operated is 333MHz. The effective frequency is minimum of the both which is 250MHz. Thus the effective maximum frequency has increased after performing the retiming.

### **Illustration of Sequential Optimizsation:**

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
# flatten # use if the multiple modules are present
opt_clean -purge
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
write_verilog -noattr <netlist_name.v>
```

#### **Example 1**
The verilog code for the example 1 is given below :
```
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end

endmodule
```
The above code infers the circuit as shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1d83f130-32f1-40f4-b5fd-5cfb32d56bdf)


Since this code doesn't need optimisation it will infer a D flip-flop with asynchronous reset as shown above.

The simulation, synthesis result and the netlist are shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/6c8d3b94-1234-4091-afe5-d1d84f247d31)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/9a76991c-b3bd-4518-8241-82a5db652b48)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1cc57676-806f-4c90-b0e2-7a49469acf7c)


All the standard cells by default have negative logic for reset and since in the code reset is mentioned as positive, an inverter is used for the reset signal. 



#### **Example 2**
The verilog code for the example 2 is given below :
```
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end
endmodule
```
The above code infers a D flip-flop with asynchronous set (reset signal is applied to set input) as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/2c7fe0c0-40e1-42f0-b785-0aeb07d7dce7)


The optimised design infers a direct connection of VDD (logic 1) to the output q as shown below:

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4856a19f-8f85-42a1-9ca4-cdf6cecd6fdf)


The simulation, synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/6ac2e91d-0e46-41b8-b7ab-f66f1c87c5c3)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/dd70c622-8877-44ae-8f25-28109c746db7)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/cc66a777-9eb5-4096-aca5-e9ad3a927db3)



#### **Example 3**
The verilog code for the example 3 is given below :
```
module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```
The above code infers a two D flip-flop with asynchronous set and reset (reset signal is applied to set and reset input) as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/74539b84-c571-410f-b796-63d193bd0aae)


Since this code doesn't need optimisation it will infer two D flip-flop with asynchronous set and reset as shown above.

The simulation, synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/88aa9f25-1880-4321-a79a-859a16153ae2)

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/54df708e-4748-4968-b1fb-60adb6ad95bb)

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/eb10ce56-c9db-446c-b1c8-7ffab31e82a6)


At the timestamp 1550 the signal q1 changes from 0 to 1 but the output q transits from 1 to 0 for a clock cycle. It is because there will be a finite clock to q delay so the second flip-flop will sample the logic 0 at that rising edge of the clock. Hence there is a change in the output signal for one clock cycle. 




#### **Example 4**
The verilog code for the example 4 is given below :
```
module dff_const4(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b1;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```
The above code infers a two D flip-flop with asynchronous set(reset signal is applied to set input ) as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/9f7d572c-a497-450c-a94a-b3b3c4a6605b)


The optimised design infers a direct connection of VDD (logic 1) to the output q as shown below:
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/78e7c903-2d16-4bb9-8d60-9c84f3ac49c8)

The simulation, synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/914eb58d-5541-450f-b1ba-0abbb939d38e)

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c21e4ad2-4131-4ba3-ad14-a058fa84eaa6)

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/aef01b5f-90d7-41e0-a8b4-9d88ce5f0c9c)


#### **Example 5**
The verilog code for the example 5 is given below :
```
module dff_const5(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b0;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```


The above code infers a two D flip-flop with asynchronous reset  as shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c0dabdb4-5d5a-4b78-9d60-04333501a796)


Since this code doesn't need optimisation it will infer two D flip-flop with asynchronous reset as shown above.


The simulation, synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/03909b0b-2890-435e-8ade-d9353140b98d)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/1349295b-e885-4083-a4ee-b8826674eb0e)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/11c3c13e-316b-4fba-93a0-0ee6f52e4e34)


### **Optimisation of Unused States**

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
opt_clean -purge
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
write_verilog -noattr <netlist_name.v>
```


Consider the verilog code shown below :
```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```

This verilog code will infer a 3-bit counter with asynchronous reset.
The possible states of the counter are as follows :
| count[2] count[1] count[0]  |COUNT[2] COUNT[1] COUNT[0] |
|:---:|:---:|
| 0 0 0 | 0 0 1  |
| 0 0 1 | 0 1 0 |
| 0 1 0 | 0 1 1 |
| 0 1 1 | 1 0 0 |
| 1 0 0 | 1 0 1 |
| 1 0 1 | 1 1 0 |
| 1 1 0 | 1 1 1 |
| 1 1 1 | 0 0 0 |

where </br>
count - Previous count</br>
COUNT - Preset count

Since the output q is always assigned COUNT[0]. The other bits of the count are not used and not required. Instead of infering three flip-flops , on optimising the design it will infer a single D flip-flop and an inverter as shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/383c3192-2500-4a89-9713-2af9a841ccbd)

The simulation, synthesis result and the netlist are shown below :

![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/4e9729d7-92c8-42c5-815b-22d623d62d4b)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fcbfdf1b-abd7-475c-9edc-f965137f430d)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c61caa75-6cad-4e51-829b-d2d1db400d15)


Consider another verilog code shown below :
```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count==3'b100;

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```
In this case since q is asserted only when count == 3'b100, all the three flip-flops are used. Hence even after optimisation , the code will infer three flops.

The simulation, synthesis result and the netlist are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/95e4db9a-9489-49e2-bddd-d9d0e157c2a4)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c991f5d0-9a5f-4627-b4da-234953017346)


</details>

### gate level simulation
<details>

<summary>gate level simulation, blocking and non blocking assignments and synthesis-simulation mismatch</summary>


## Day - 4 : Gate Level Simulation (GLS), Blocking Vs Non-blocking assignment and Synthesis-Simulation Mismatch

### **Gate Level Simulation**
Gate Level Simulation helps ensure that the synthesized version of the design matches the specification both in terms of functionality and  timing. It helps identify mistakes and differences in the synthesised netlist and ensures that the final design functions as intended. Generally GLS is done to ensure that there is no synthesis-simulation mismatch. To perform the GLS the testbench that is used to verify the RTL is used. The GLS flow is similar to the testbench flow except that gate level verilog models are also used. It is necessary to mention the gatelevel verilog models  to iverilog to make the iverilog understand about the standard cell given in the library .GLS requires adding information about timing delays. Gate level Verilog models can be functional and timing aware. If the gate level models are delay annotated then it can used for timing validation. 
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e05c6043-9c54-43ed-954b-decb7f65d7d9)


### **Synthesis-Simulation Mismatch**
Synthesis-simulation mismatch refers to the differences between the behavior of a digital circuit as simulated at the Register Transfer Level (RTL) and its behavior after being synthesized to gate-level netlists. Synthesis-simulation mismatch can occur because of the following reasons:
1. Missing Sensitivity List
2. Blocking vs Non-blocking assignments
3. Non standard verilog coding

#### **1. Missing Sensitivity List**
Consider the verilog code and its corresponding graph shown below :
```
module mux(
	input i0,i1,s,
	output reg y
)
	always @(sel) begin
		if(sel)
			y = i1;
		else
			y = i0;
	end
endmodule
```
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/550ece82-a607-4a1f-95b8-ce9602cb8559)


The "always" block is sensitive only to the "sel" signal. Whenever there's a modification in the "sel" output, it triggers a change in the output value. However, as this piece of code implies a multiplexer, the output should also change if the input changes. Since the sensitivity list includes only "sel," the output remains unaffected and it doesn't follow the input i0 when the sel is logic 0. Hence this a circuit behaves like a latch.

In order to solve the problem all the critical signals needed to be mentioned in the sensitivity list. So the corrected code is given below :
```
module mux(
	input i0,i1,s,
	output reg y
)
	always @(*) begin //* - It considers changes in all the input signals. So always is evaluated whenever any signal changes.
		if(sel)
			y = i1;
		else
			y = i0;
	end
endmodule
```
#### **2. Blocking and Non-Blocking Statements in Verilog**
Blocking and Non-Blocking statemnets are very important statements. They must used with utmost care so that intended logic is created. These statements are used inside the always block.
**1. Blocking Assignment:**
Blocking assignments are denoted using the "=" operator. When a blocking assignment is executed, it directly assigns the right-hand side value to the left-hand side variable immediately within the current simulation cycle. The subsequent statements in the procedural block  will wait for this assignment to complete before proceeding. Blocking assignments are sequentially executed.

**2. Non-blocking Assignment:**
Non-blocking assignments are denoted using the "<=" operator. When a non-blocking assignment is encountered, the right-hand side value is scheduled to be assigned to the left-hand side variable at the end of the current simulation cycle. This means that all non-blocking assignments within a procedural block are executed simultaneously, updating variables concurrently. The value changes take effect in the next simulation cycle. Non-Blocking assignments are executed in parallel.

#### **Caveats with Blocking Assignment**
**Example 1**
Consider the verilog code given below:
```
module code(
	input clk,reset,d,
	output reg q
)
	reg q0;
	always @(posedge clk, posedge reset) begin
		if(reset) begin
			q=1'b0;
			q0=1'b0;
		end
		else begin
			q = q0; //Line 1
			q0=d; // Line 2
		end
	end
endmodule
```
The inetent of this code is to create a 2-bit shift register. Since blocking assignmnet is used for Line 1 and Line 2 both the lines will be executed sequentially. First line 1 will be executed creating a flip-flop whose input is q0 and output is q. Then line 2 will be executed which creates a second flip-flop whose input is d and output is q0 thereby connecting two flip-flops and creating a 2-bit shift register shown below:
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/080f5f4c-90ca-4d32-af3d-e615464f4a09)


Consider the verilog code shown below :
```
module code(
	input clk,reset,d,
	output reg q
)
	reg q0;
	always @(posedge clk, posedge reset) begin
		if(reset) begin
			q=1'b0;
			q0=1'b0;
		end
		else begin
			q0 = d; //Line 1
			q=q0; // Line 2
		end
	end
endmodule
```
This code looks similar to the previous one except that line 1 and line 2 are interchanged. Since , blocking assignment is used line 1 and line 2 will be executed sequentially. First line 1 will be executed which creates a D flip-flop with the input d and output q0, then line 2 is executed. Since q0 is already defined assigning q0 to q creates wire . Hence only flip-flop is inferred instead of two. The circuit corresponding to the code is shown below :


![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/ab617f50-66cd-4b25-a90b-417190e528cb)


**Example 2**
Consider the verilog code shown below :
```
module(
	input a,b,c,
	output reg y
)
	reg q0;
	always @(*) begin
		y = q0 & c; //Line 1
		q0 = a|b; //Line 2
	end
endmodule
```
In line 1 the output y is assigned with q0&c. But q0 is not mentioned anywhere before. Hence the previous value of the q0 will be taken and this will not infer a combinational circuit as expected instead a latch based circuit will be inferred. The corrected version of the code is shown below:
```
module(
	input a,b,c,
	output reg y
)
	reg q0;
	always @(*) begin
		q0 = a|b; //Line 1
		y = q0 & c; //Line 2
		
	end
endmodule
``` 

### **Illustration of GLS and Synthesis Simulation Mismatch**

**Steps to simulate, generate the netlist and to perform the GLS for the below designs**

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
# opt_clean -purge # If optimisation has to be done
# dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib # if sequential circuit is used 
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
show
write_verilog -noattr <netlist_name.v>
```

Steps to perform GLS:
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v <netlist_name.v> <tb_name.v>
./a.out
gtkwave <dump_file_name.vcd>
```

#### **Example 1**
Consider the verilog code shown below :
```
module ternary_operator_mux (input i0 , input i1 , input sel , output y);
	assign y = sel?i1:i0;
endmodule
```
In verilog ternary operator will realize  multiplexer upin synthesis. If the operand left of the ? is true then output follows the immediate operand right of  ? otherwise the ouput follows the immediate operand to the right of :.

The simulation, synthesis result , the netlist and the GLS are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/cc154fd0-0f15-430e-b255-4c0ef3945a00)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c1ba98ee-5fd2-434d-a165-e69e136b8687)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/92848a97-2730-49b4-996e-26531826de70)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/94b5c142-69bb-4aef-9b6f-88a79fc8eead)



In this case there is no synthesis and simulation mismatch.

#### **Example 2**
Consider the verilog code shown below :
```
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @ (sel)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```
This code only has sel signal in sensitivity list. Hence the RTL simulation output will not match the expected specification.

The simulation, synthesis result , the netlist and the GLS are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/a7302786-10fc-4fd0-a7cb-a975cbec01f7)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/e051425a-ae15-4ecf-88b5-66c7fa4e2d57)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/b22d9d0c-1933-46d5-897f-8ef82c31ef88)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/c1cf136d-9749-4556-9159-193db1707052)



In this case there is a synthesis and simulation mismatch. While performing synthesis yosys has corrected the sensitivity list error.


#### **Example 3**
Consider the verilog code shown below :
```
module blocking_caveat (input a , input b , input  c, output reg d); 
reg x;
always @ (*)
begin
	d = x & c; //Line 1
	x = a | b; //Line 2
end
endmodule
```
This code only has signal x in line which is not defined before. Hence the previous value of x will be taken and the expression will be evaluated. Hence the RTL simulation output will not match the expected specification and will infer a latch based circuit instead of the combinational circuit.

The simulation, synthesis result , the netlist and the GLS are shown below :
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/732d0d02-4c37-4a4f-b0c1-68a07ac51896)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/5854ca4f-7257-4cd9-a925-b1c98cb6aa51)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fa7e20d7-44db-44bf-97a7-ec6cc9db821e)
![image](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/22444184-55b4-419f-9f86-8a07aa8a6908)


In this case there is a synthesis and simulation mismatch. While performing synthesis yosys has corrected the latch error.


</details>















