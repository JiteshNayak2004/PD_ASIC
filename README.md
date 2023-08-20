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

# integer number representations

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




