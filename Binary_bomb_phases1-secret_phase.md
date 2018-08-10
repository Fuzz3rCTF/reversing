#  CMU Binary Bomb

# PHASE1:
##### On the the phase_1 function you see it pushes a value on an address
```
gdb-peda$ disas phase_1
Dump of assembler code for function phase_1:
   0x08048b20 <+0>:	push   ebp
   0x08048b21 <+1>:	mov    ebp,esp
   0x08048b23 <+3>:	sub    esp,0x8
   0x08048b26 <+6>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048b29 <+9>:	add    esp,0xfffffff8
   0x08048b2c <+12>:	push   0x80497c0
   0x08048b31 <+17>:	push   eax
   0x08048b32 <+18>:	call   0x8049030 <strings_not_equal>
   0x08048b37 <+23>:	add    esp,0x10
   0x08048b3a <+26>:	test   eax,eax
   0x08048b3c <+28>:	je     0x8048b43 <phase_1+35>
   0x08048b3e <+30>:	call   0x80494fc <explode_bomb>
   0x08048b43 <+35>:	mov    esp,ebp
   0x08048b45 <+37>:	pop    ebp
   0x08048b46 <+38>:	ret    
End of assembler dump.
```
```
gdb-peda$ x/s 0x80497c0
0x80497c0:	"Public speaking is very easy."
```

##### Phase1 `flag` =>  Public speaking is very easy.

# PHASE2:
```
gdb-peda$ disas phase_2
Dump of assembler code for function phase_2:
   0x08048b48 <+0>:	push   ebp
   0x08048b49 <+1>:	mov    ebp,esp
   0x08048b4b <+3>:	sub    esp,0x20
   0x08048b4e <+6>:	push   esi
   0x08048b4f <+7>:	push   ebx
   0x08048b50 <+8>:	mov    edx,DWORD PTR [ebp+0x8]
   0x08048b53 <+11>:	add    esp,0xfffffff8
   0x08048b56 <+14>:	lea    eax,[ebp-0x18]
   0x08048b59 <+17>:	push   eax
   0x08048b5a <+18>:	push   edx
   0x08048b5b <+19>:	call   0x8048fd8 <read_six_numbers>
   0x08048b60 <+24>:	add    esp,0x10
   0x08048b63 <+27>:	cmp    DWORD PTR [ebp-0x18],0x1
   0x08048b67 <+31>:	je     0x8048b6e <phase_2+38>
   0x08048b69 <+33>:	call   0x80494fc <explode_bomb>
   0x08048b6e <+38>:	mov    ebx,0x1
   0x08048b73 <+43>:	lea    esi,[ebp-0x18]
   0x08048b76 <+46>:	lea    eax,[ebx+0x1]
   0x08048b79 <+49>:	imul   eax,DWORD PTR [esi+ebx*4-0x4]
   0x08048b7e <+54>:	cmp    DWORD PTR [esi+ebx*4],eax
   0x08048b81 <+57>:	je     0x8048b88 <phase_2+64>
   0x08048b83 <+59>:	call   0x80494fc <explode_bomb>
   0x08048b88 <+64>:	inc    ebx
   0x08048b89 <+65>:	cmp    ebx,0x5
   0x08048b8c <+68>:	jle    0x8048b76 <phase_2+46>
   0x08048b8e <+70>:	lea    esp,[ebp-0x28]
   0x08048b91 <+73>:	pop    ebx
   0x08048b92 <+74>:	pop    esi
   0x08048b93 <+75>:	mov    esp,ebp
   0x08048b95 <+77>:	pop    ebp
   0x08048b96 <+78>:	ret    
End of assembler dump.
```
> You can clearly see it needs 6 ints as an input
> You can also verify it by goinf in the `read_six_numbers` function
>  0x08049007 <+47>:	cmp    eax,0x5


You start the program on a debugger with a breakpoint on phase_2 function
You give the first flag you already know
It asks for a second input and you give six numbers ex. `1 2 3 4 5 6`
You hit the breakpoint you previously set :)
You go instruction-instruction to be sure you don't miss anything
You see a cmp instruction,
```
=> 0x8048b63 <phase_2+27>:	cmp    DWORD PTR [ebp-0x18],0x1
```
It checks if the first number you gave is 1.
We hold that in mind and continue instruction-instruction until we see something interesting
```
=> 0x8048b7e <phase_2+54>:	cmp    DWORD PTR [esi+ebx*4],eax
```

#### Hmmm
Let's see what the value of `eax` is.
```
gdb-peda$ p/x $eax
$2 = 0x2
```
> Nice, so our second input must be 2!
> We AGAIN continue instruction-instruction till we see something interesting 

We again see the same instruction
We do the same
It is clear now the we are on a loop :O
```
gdb-peda$ x/d $eax
```
With the same logic our third number should be 6
We continue
> But we set `esi+$ebx*4` to the value of `eax` we gave so we won't hit explode_bomb 
```
gdb-peda$ x/d $esi+$ebx*4
0xffffcde8:	3
gdb-peda$ set *0xffffcde8=0x6
```
> When we hit the cmp instruction the `EAX` is `0x18`
> `0x18 = 24`
> so our forth input must be 24 
> We again continue 
> But again we set `esi+ebx*4` to the value of `eax` so we don't hit on explode_bomb function
```
gdb-peda$ x/d $esi+$ebx*4
0xffffcdec:	4
gdb-peda$ set *0xffffcdec=0x18
```
> Continue(we could clearly understand the pattern but let's finish it the way i did it back then because i was dumb af)

we continue
`Eax` now becomes `0x78 = 120`
Our fifth number must be `120`
We again set our input to `120` so we won't hit explode_bomb function
```
gdb-peda$ x/d $esi+$ebx*4
0xffffcdf0:	5
gdb-peda$ set *0xffffcdf0=0x78
```
Continue.

Now `eax` is `0x2d0 = 720`
So our last number must be `720`

To sum up our input to bypass phase_2 must be:
`1 2 6 24 120 720`

# PHASE3:
#### Opening the function over IDA to be more clear 
You see the input must be 
<NUM> <CHAR> <NUM>
(%d %c %d)

You open the program on a debugger and give it an input that satisfies the format
ex.
`1 a 2`

> You have again set a breakpoint on phase_3 to see the instructions 
> You AGAIN go through intruction-intruction 
We see a `cmp` instruction
```
=> 0x8048bbf <phase_3+39>:	cmp    eax,0x2
```
BUT this doesnt mean your first input that is stored in eax(You can check it) must be 2
Over some tests you can see that IF your first int is 2 the program calls the explode _bomb function
So! Our first input must be an int < 2 
Let's say `1`
Ok we continue 

> We then see an intruction that checks if our SECOND number(third input) is `214`
```
=> 0x8048c02 <phase_3+106>:	cmp    DWORD PTR [ebp-0x4],0xd6
```
So we now know that our flag looks like this 
`1 <CHAR> 214`
We still need the character. 
We continue. 
We see a cmp instruction that checks if our character (ebp-0x5) is equal to `bl` variable.
```
=> 0x8048c8f <phase_3+247>:	cmp    bl,BYTE PTR [ebp-0x5]
```
> We just print out the `bl` variable. 
```
gdb-peda$ p/x $bl
$31 = 0x62
```
So our flag must be `1 b 214`

# PHASE4:
#### This part is by far the worst reversing challenge i have seen (i havent seen many:P)
We start again by setting a breakpoint on phase_4 

We have first seen from the disassemble of IDA that it need only one number as input
I wish i had bruteforced this...

You give an input let's say `7`

You hit the breakpoint
We see some junk checks(if our inut is 0 the bomb explodes)
We see a interesting function `func4`

If you go through it a looooooot of times you will notice that it removes 1 byte from your input and then 2 and then it add those numbers together.
After "some" googling i found that this is the relation of Fibonacci numbers. 
You must also have seen that there is a `cmp` instruction after func4 function.
that checks if it's 55. So you understand that what ever we gave as input it must become 55 
after func4's loop
There is a fibonacci table number list that giver you the source number for every fibonacci number. 
```
f(55) = 9
```
So `9` is the answer here.

# PHASE5:

#### Here we have to do with a cipher function that we have to find an input that when it's ciphered gives us the word `giants`

There is also a list that acccording to it the cipher occurs. 
Okay, our input must be 6 chars long so we give a good pattern.
`abcdef`
After the cipher loop you see the output and the list that was used.

list: ```isrveawhobpnutfg```
output of ````abcdef: srveaw````

It took me a long time to figure out what was goinf on but you can understand that because our first input was 0x61 (a)
it moved 1 character.

#### NOTE: computers start counting from 0 dummy!

So we must have an input that their hex looks like this:

```0xXf, 0xX0, 0xX5, 0xX5, 0xXb, 0xXd```

> Make you calculations and i got a weird string.
> There many possible string that can be inputted.
> Mine was `?05+=1`


# PHASE6:
#### An easy one but tricky
After opening it on a disassembler you see a weird variable called node1
`There must be the trick` a skilled hacker should have thought but it took me hours to figure this out.
> You print the node1 on the `GDB`
And you see it is a number!
You hold that.
After hours of brainfuck i thought
there must be other nodes too!
> I was lucky making the decission of checking for other nodes
After the loop function stops we will be able to print the value of node1
```
gdb-peda$ x/3x $esi
0x804b26c <node1>:	0x000000fd	0x00000001	0x0804b260
```
> BUT you can't just go through byte-byte to hit the the node2 node3 etc
> You must do a trick i found out.
> The trick goes as follows: 
```
gdb-peda$ x/3x *($esi+8)
0x804b260 <node2>:	0x000002d5	0x00000002	0x0804b254
```
> You do the same for the other `4` nodes 
> And you finally get all the nodes:
```
gdb-peda$ x/3x *(*($esi+8)+8)
0x804b254 <node3>:	0x0000012d	0x00000003	0x0804b248
```
```
gdb-peda$ x/3x *(*(*($esi+8)+8)+8)
0x804b248 <node4>:	0x000003e5	0x00000004	0x0804b23c
```
```
gdb-peda$ x/3x *(*(*(*($esi+8)+8)+8)+8)
0x804b23c <node5>:	0x000000d4	0x00000005	0x0804b230
```
```
gdb-peda$ x/3x *(*(*(*(*($esi+8)+8)+8)+8)+8)
0x804b230 <node6>:	0x000001b0	0x00000006	0x00000000
```
The values on the `nodes` go as follows:
```
node1 = 253
node2 = 725
node3 = 301
node4 = 997
node5 = 212
node6 = 432
```
If you are smart you could notice that the nodes represent the numbers of your input and you must input them from the bigger to the smaller.

##### So:
```
node4 = 997
node2 = 725
node6 = 432
node3 = 301
node1 = 253
node5 = 212
 ```
###### So you can immediately understand your input must be 4 2 6 3 1 5
##### flag => `4 2 6 3 1 5`




# SECRET PHASE:

##### Secret phase was easy.
You just had to notice that on the phase_defused function there was a check for secret_phase
You can see that the path you have to follow is to input a number and the string `austinpowers`
Where there is only one number?
Correct! The path `4`
So we input `9 austinpowers`
But on the end it asks us for a new input.
This was easy to 
You just had to notice that 
There is a value that our input must be equal to.
The number `1000`
But when debugging you could see that your input is stored on `EBX` and then `EAX = EBX - 1`
So your input must be `1001` so after the instruction it will be equal to `1000`

##### flag => `1001`

### Thanks for reading, your lovely skid,
## Fuzz3r <3
