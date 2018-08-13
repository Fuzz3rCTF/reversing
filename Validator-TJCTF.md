# Validator | TJCTF 2018


Let's start by running the program.
```
mike@computer:~/Desktop$ ./validator 
Usage: ./validator <flag>
```
__Hmmm__ we now know we have to pass an argument.
We attach it on GDB and start looking around to get familiar with the flow of the execution.

We set a breakpoint on the __main__ function and
run the program using a random argument

```
gdb-peda$ b main
Breakpoint 1 at 0x8048509
gdb-peda$ r mike123
Starting program: /home/mike/Desktop/validator mike123

[----------------------------------registers-----------------------------------]
EAX: 0xf7fa8dbc --> 0xffffced0 --> 0xffffd0f9 ("LC_PAPER=el_GR.UTF-8")
EBX: 0x0 
ECX: 0xffffce30 --> 0x2 
EDX: 0xffffce54 --> 0x0 
ESI: 0xf7fa7000 --> 0x1b1db0 
EDI: 0xf7fa7000 --> 0x1b1db0 
EBP: 0xffffce18 --> 0x0 
ESP: 0xffffce14 --> 0xffffce30 --> 0x2 
EIP: 0x8048509 (<main+14>:	sub    esp,0x44)
EFLAGS: 0x282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x8048505 <main+10>:	push   ebp
   0x8048506 <main+11>:	mov    ebp,esp
   0x8048508 <main+13>:	push   ecx
=> 0x8048509 <main+14>:	sub    esp,0x44
   0x804850c <main+17>:	mov    eax,ecx
   0x804850e <main+19>:	mov    edx,DWORD PTR [eax+0x4]
   0x8048511 <main+22>:	mov    DWORD PTR [ebp-0x3c],edx
   0x8048514 <main+25>:	mov    ecx,DWORD PTR gs:0x14
[------------------------------------stack-------------------------------------]
0000| 0xffffce14 --> 0xffffce30 --> 0x2 
0004| 0xffffce18 --> 0x0 
0008| 0xffffce1c --> 0xf7e0d637 (<__libc_start_main+247>:	add    esp,0x10)
0012| 0xffffce20 --> 0xf7fa7000 --> 0x1b1db0 
0016| 0xffffce24 --> 0xf7fa7000 --> 0x1b1db0 
0020| 0xffffce28 --> 0x0 
0024| 0xffffce2c --> 0xf7e0d637 (<__libc_start_main+247>:	add    esp,0x10)
0028| 0xffffce30 --> 0x2 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x08048509 in main ()
gdb-peda$ 
```

We going instruction instruction until we find anything interesting.

We hit an instruction that checks if our input length is 43.
```
0x804858a <main+143>:	cmp    eax,0x2b
````
We accordingly set our input length to 43 so not to exit. 

```
gdb-peda$ set $eax=0x2b
```

We hit an interesting part again:
```
   0x0804858f <+148>:	mov    BYTE PTR [ebp-0x25],0x33
   0x08048593 <+152>:	mov    BYTE PTR [ebp-0x22],0x33
   0x08048597 <+156>:	mov    BYTE PTR [ebp-0x20],0x33
   0x0804859b <+160>:	mov    BYTE PTR [ebp-0x24],0x35
   0x0804859f <+164>:	mov    BYTE PTR [ebp-0x23],0x72
   0x080485a3 <+168>:	mov    BYTE PTR [ebp-0x1f],0x72
   0x080485a7 <+172>:	mov    BYTE PTR [ebp-0x21],0x76
 ```
 If we try to get the string off these addresses we will take some parts of the flag.
 So a bit of playing around the exact address where the flag is stored is:
 ```
 gdb-peda$ x $ebp-0x38
0xffffcde0:	"tjctf{ju57_c4ll_m3_35r3v3r_60d_fr0m_n0w_0n}"
```

Thanks for reading and sorry for the quality of the challenge. I had nothing more difficult to make a writeup :) <3 
