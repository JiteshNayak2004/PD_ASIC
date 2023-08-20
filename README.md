# PD_ASIC
this is a repo that contains the notes and lab assignments for physical design for ASIC course

## Day 1 <setting up the risc-v compiler>
1.the risc-v compiler version
![Screenshot from 2023-08-20 10-44-56](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/fc1fce85-47da-4347-b615-43a5367c0af2)

2.writing and compiling a c program

'''c
#inclulde<stdio.h>

int main()
{
int 1,sum=0,n=15;
for (i=1;i<=n;++i)
{
sum=sum+i
}
printf("sum of numbers from i to %d is %d \n");
}


3.using the risc-v compiler to compile a simple hello world c program
and seeing the first few lines of the binary generated

![Screenshot from 2023-08-19 15-28-27](https://github.com/JiteshNayak2004/PD_ASIC/assets/117510555/482b273d-b45f-4d17-b5de-2354ed0a98e3)


